# 🚨 VERIQTA Kubernetes Production Outages Master Handbook (Volume 1)
> **Workload & Pod Failures: Root Causes, Forensics, Investigation Runbooks & Prevention Playbooks.**  
> *Engineered for Site Reliability Engineers (SREs), DevOps Engineers, Platform Architects, and Incident Responders.*

---

## 📑 Table of Contents
1. [Executive Mental Model: The 4-Layer Outage Correlation](#1-executive-mental-model-the-4-layer-outage-correlation)
2. [How Kubernetes Workload Outages Develop (The Failure Path)](#2-how-kubernetes-workload-outages-develop-the-failure-path)
3. [The Senior Engineer's 10-Step Pod Investigation Workflow](#3-the-senior-engineers-10-step-pod-investigation-workflow)
4. [CrashLoopBackOff: Root Causes & Exponential Backoff Mechanics](#4-crashloopbackoff-root-causes--exponential-backoff-mechanics)
5. [CrashLoopBackOff Forensics & Linux Exit Codes (137, 143, 127, 1)](#5-crashloopbackoff-forensics--linux-exit-codes-137-143-127-1)
6. [ImagePullBackOff & ErrImagePull Triage Matrix](#6-imagepullbackoff--errimagepull-triage-matrix)
7. [Containers Stuck in ContainerCreating: Node-Level Failure Modes](#7-containers-stuck-in-containercreating-node-level-failure-modes)
8. [Pods Stuck in Pending: Scheduler Predicates & Quota Exhaustion](#8-pods-stuck-in-pending-scheduler-predicates--quota-exhaustion)
9. [OOMKilled Containers: Linux Cgroup Limits & Memory Leaks (Exit 137)](#9-oomkilled-containers-linux-cgroup-limits--memory-leaks-exit-137)
10. [CPU Throttling Without Pod Failure: Silent Latency & CFS Quotas](#10-cpu-throttling-without-pod-failure-silent-latency--cfs-quotas)
11. [Liveness Probe Failures: The Cascading Restart Storm](#11-liveness-probe-failures-the-cascading-restart-storm)
12. [Readiness Probe Failures: The "Zero-Capacity" Service Blackhole](#12-readiness-probe-failures-the-zero-capacity-service-blackhole)
13. [Startup Probe Misconfiguration & Slow Initialization Protection](#13-startup-probe-misconfiguration--slow-initialization-protection)
14. [Init Container Failures: Blocking the Entire Application Lifecycle](#14-init-container-failures-blocking-the-entire-application-lifecycle)
15. [ConfigMap Changes That Break Production & In-Memory Drift](#15-configmap-changes-that-break-production--in-memory-drift)
16. [Secret Failures, Base64 Traps & External Secret Sync Outages](#16-secret-failures-base64-traps--external-secret-sync-outages)
17. [Deployment Rollout Failures & ProgressDeadlineExceeded](#17-deployment-rollout-failures--progressdeadlineexceeded)
18. [Failed Rollbacks & The Distributed State Schema Trap](#18-failed-rollbacks--the-distributed-state-schema-trap)
19. [High-Severity Workload Outage Incident Response Playbook](#19-high-severity-workload-outage-incident-response-playbook)
20. [Pre-Deployment & Active-Incident Production Readiness Scorecard](#20-pre-deployment--active-incident-production-readiness-scorecard)

---

## 1. Executive Mental Model: The 4-Layer Outage Correlation

```
                   THE 4-LAYER OUTAGE CORRELATION FRAMEWORK
 ┌────────────────────────────────────────────────────────────────────────┐
 │ 1. Kubernetes Infrastructure State (Nodes Up, Controllers Happy)       │
 │ 2. Pod & Container Health (Kubelet Probes, Cgroups, Exit Codes)       │
 │ 3. External Dependencies (Databases, Redis, Queues, DNS, IAM, Secrets) │
 │ 4. Real User Impact (HTTP 5xx Rates, P99 Latency, SLA/SLO Breaches)    │
 └────────────────────────────────────────────────────────────────────────┘
```

> [!CAUTION]
> **Production Axiom:** Kubernetes can keep the infrastructure 100% "healthy" (Nodes `Ready`, Pods `Running`) while your users experience a **100% catastrophic outage**. Never judge system health by pod status alone.

---

## 2. How Kubernetes Workload Outages Develop (The Failure Path)

```
                            THE WORKLOAD FAILURE PATH
┌────────────┐      ┌────────────┐      ┌────────────┐      ┌────────────┐      ┌─────────────┐
│ Deployment │ ───> │ ReplicaSet │ ───> │    Pod     │ ───> │ Container  │ ───> │ Application │
│ (Strategy) │      │ (Counts)   │      │ (Topology) │      │ (Cgroups)  │      │ (Code/DB)   │
└────────────┘      └────────────┘      └────────────┘      └────────────┘      └─────────────┘
      │                   │                   │                   │                    │
  Deadlock            Mismatch            Eviction            OOMKill             Exception
```

### Why a `Running` Pod $\neq$ a Healthy Application
1. **Internal Application Deadlock:** The process is alive inside the container PID namespace, but all worker threads are stuck in a mutex lock.
2. **Readiness Probe Passing Incorrectly:** Probe checks static `/healthz` returning HTTP 200 while database connection pool is completely exhausted.
3. **Downstream Dependency Outage:** PostgreSQL RDS or Redis ElastiCache is down; pods return HTTP 500 on all user requests.
4. **Service Misconfiguration:** Service selector labels do not match pod labels; endpoints list is empty (`0/N Endpoints`).
5. **CPU CFS Quota Throttling:** Pod is starved of CPU time slices, causing extreme latency and connection timeouts without crashing.

---

## 3. The Senior Engineer's 10-Step Pod Investigation Workflow

```
                        10-STEP INCIDENT TRIAGE WORKFLOW
  [01. Confirm User Impact] ──> [02. Check Deploy Status] ──> [03. Inspect Pod States]
                                                                        │
  [06. Inspect Logs & Prev] <── [05. Desired vs Available] <── [04. Review Events]
        │
        └──> [07. Check Probes/Limits] ──> [08. Review Recent Changes]
                                                    │
             [10. Recover & Monitor] <── [09. Validate Dependencies]
```

### Core CLI Diagnostic Commands
```bash
# 1. Broad view of deployments and pods
kubectl get deploy,rs,pods -n production -o wide

# 2. Chronological event log (Reveals scheduler, probe, and OOM events)
kubectl get events -n production --sort-by=.lastTimestamp

# 3. Deep forensic inspection of a failing pod
kubectl describe pod <pod-name> -n production

# 4. Read logs from the PREVIOUS crashed container instance
kubectl logs <pod-name> -n production -c <container-name> --previous

# 5. Live CPU and Memory utilization per pod
kubectl top pod -n production --containers
```

---

## 4. CrashLoopBackOff: Root Causes & Exponential Backoff Mechanics

```
                       EXPONENTIAL BACKOFF RESTART TIMELINE
   Crash 1        Crash 2         Crash 3         Crash 4           Crash 5+
  ┌────────┐     ┌────────┐      ┌────────┐      ┌────────┐      ┌───────────────┐
  │ Delay: │ ──> │ Delay: │ ───> │ Delay: │ ───> │ Delay: │ ───> │ Max Delay:    │
  │ 10 Sec │     │ 20 Sec │      │ 40 Sec │      │ 80 Sec │      │ 5 Mins (300s) │
  └────────┘     └────────┘      └────────┘      └────────┘      └───────────────┘
```

### What `CrashLoopBackOff` Actually Means
* `CrashLoopBackOff` is **not an error in itself**—it is Kubernetes rate-limiting container restart attempts to protect worker nodes and shared resources from CPU/I/O thrashing.

### Top 6 Root Causes
1. **Application Startup Failure:** Uncaught exceptions, syntax errors, missing runtime dependencies.
2. **Missing Configuration:** Non-existent ConfigMap key or Secret referenced in `env` or `volumeMounts`.
3. **Port Conflict:** Application attempts to bind to a port already occupied inside the Pod container.
4. **Volume Permission Denied:** Container runs as non-root UID `10001` attempting to write to a volume owned by root UID `0`.
5. **Failing Liveness Probe:** Aggressive probe intervals killing the app before it completes initialization.
6. **Failed Downstream Connection:** Database connection timeout or DNS resolution failure on startup.

---

## 5. CrashLoopBackOff Forensics & Linux Exit Codes (137, 143, 127, 1)

```
                            LINUX CONTAINER EXIT CODE MATRIX
 ┌───────────┬──────────────────────────┬────────────────────────────────────────────────────────┐
 │ Exit Code │ Standard Linux Meaning   │ Common Kubernetes Root Cause & Forensic Action         │
 ├───────────┼──────────────────────────┼────────────────────────────────────────────────────────┤
 │ 0         │ Success                  │ Batch task finished (or process exited unexpectedly). │
 │ 1         │ Application Error        │ Uncaught language exception, bad config, fatal crash. │
 │ 2         │ Shell Built-in Misuse    │ Invalid arguments in container command/entrypoint.    │
 │ 126       │ Command Not Executable   │ Permissions error on binary (chmod +x missing).       │
 │ 127       │ Command Not Found        │ Binary path missing in container image or bad PATH.   │
 │ 137       │ OOMKilled (SIGKILL 9)    │ Container exceeded cgroup memory limit (128 + 9).     │
 │ 143       │ Graceful Exit (SIGTERM)  │ Kubernetes shutting down pod during scale/upgrade.    │
 │ 255       │ Exit Out of Range        │ Fatal process exit or initialization failure.          │
 └───────────┴──────────────────────────┴────────────────────────────────────────────────────────┘
```

### Forensic Inspection Command
```bash
# Extract exact termination exit code, reason, and finished timestamp
kubectl get pod <pod-name> -n production -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\tLastState: "}{.lastState.terminated.reason}{"\tExitCode: "}{.lastState.terminated.exitCode}{"\n"}{end}'
```

---

## 6. ImagePullBackOff & ErrImagePull Triage Matrix

```
                      IMAGE PULL INVESTIGATION WORKFLOW
  Check Pod Events ──> Verify Registry URL ──> Validate Auth Secret ──> Verify Node Arch
       (describe)          (Image Tag)             (imagePullSecrets)        (ARM64 vs AMD64)
```

### Common Failure Scenarios & Solutions
1. **Typo in Image Tag:** Image `webapp:v1.2.0` typed as `webapp:v1.2.O`. *Fix:* Correct manifest tag.
2. **Missing `imagePullSecrets`:** Private DockerHub/ECR registry requires credentials in the workload namespace.
3. **DockerHub Rate Limiting:** Anonymous pulls exceed 100 requests per 6 hours. *Fix:* Configure authenticated secret.
4. **Architecture Mismatch:** Container built for Apple Silicon (`linux/arm64`) deployed on `linux/amd64` EC2 nodes (causes `exec format error`).
5. **Private Registry DNS Failure:** Worker node cannot resolve internal registry hostname.

---

## 7. Containers Stuck in ContainerCreating: Node-Level Failure Modes

```
                       CONTAINERCREATING STAGES ON NODE
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ 1. CNI Network  │ ──> │ 2. CSI Volume   │ ──> │ 3. Config/Secret│ ──> │ 4. Container    │
│ IP Allocation   │     │ Attach & Mount  │     │ Ingestion       │     │ Runtime Launch  │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Distinction: `Pending` vs. `ContainerCreating`
* **`Pending`:** The Kubernetes **Scheduler** has not assigned the pod to any node (Cluster compute/scheduling issue).
* **`ContainerCreating`:** The Pod **has been assigned** to a node; the node's **Kubelet** is preparing sandbox, volumes, and networking.

### Root Causes
* **CNI IP Exhaustion:** Subnet has zero available IPs for the node's ENI.
* **Volume Attachment Lock:** Cloud volume (AWS EBS) is still locked to an old terminating node (`Multi-Attach error`).
* **Disk Pressure:** Node `/var/lib/containerd` is at 100% capacity; cannot unpack container image layers.

---

## 8. Pods Stuck in Pending: Scheduler Predicates & Quota Exhaustion

```
                     SCHEDULER 2-STAGE DECISION PIPELINE
                        [ Unscheduled Pod ]
                                 │
                                 ▼
                     Stage 1: Filtering (Predicates)
        • Node has sufficient CPU/Memory requests?
        • Node matches nodeSelector / NodeAffinity?
        • Node has matching tolerations for taints?
        • PVC is bound in the matching Availability Zone?
                                 │
                                 ▼
                     Stage 2: Scoring (Priorities)
        • Ranks remaining candidate nodes (1-100)
                                 │
                                 ▼
                     [ Bound to Optimal Node ]
```

### Troubleshooting Checklist
```bash
# Check why scheduler failed to place pod
kubectl describe pod <pod-name> -n production | grep -A 10 Events

# Check namespace ResourceQuota consumption
kubectl describe resourcequota -n production

# Check cluster node allocatable capacity
kubectl describe nodes | grep -A 8 "Allocated resources"
```

---

## 9. OOMKilled Containers: Linux Cgroup Limits & Memory Leaks (Exit 137)

```
                       LINUX CGROUP MEMORY ENFORCEMENT
  App Allocates Memory ──> Exceeds `limits.memory` ──> Linux OOM-Killer Kills Process
                                                               │
                                                               ▼
                                                    Exit Code 137 (SIGKILL 9)
```

### Why Increasing Memory Limit is NOT Always the Fix
* **Traffic-Driven Scale:** If memory increases linearly with active user sessions, increase limits or scale horizontally via HPA.
* **Memory Leak:** If memory climbs continuously even with zero traffic, increasing limits only delays the crash and wastes money. The application code must be fixed!

---

## 10. CPU Throttling Without Pod Failure: Silent Latency & CFS Quotas

```
                     LINUX COMPLETELY FAIR SCHEDULER (CFS)
   CPU Demand
   ┌────────────────────────────────────────────────────────┐
   │ ▲ Application needs CPU bursts                         │
   │ ┼ - - - - - - - - - - - - - - - - - - - - - - - - - - -│ CPU Limit (Quota)
   │ │ [ CPU Execution ] ░░░░░ THROTTLED ░░░░░ [ Execution ]│
   └────────────────────────────────────────────────────────┘
     0ms                 100ms (CFS Period)               200ms
```

### The Danger of Hard CPU Limits
* Exceeding CPU limit does **NOT kill the pod** (unlike memory).
* Linux CFS periodically throttles container CPU cycles, causing **severe latency spikes (P99), connection timeouts, and slow health probes** while CPU graphs look flat and healthy.

> [!TIP]
> **Modern SRE Best Practice:** Set explicit CPU **requests** for scheduler bin-packing, but consider **omitting CPU limits** (or setting high ceilings) to avoid CFS throttling on bursty workloads.

---

## 11. Liveness Probe Failures: The Cascading Restart Storm

```
                         THE RESTART STORM CYCLE
  Slow App / Latency ──> Liveness Probe Times Out ──> Kubelet Kills Container
          ▲                                                    │
          │                                                    ▼
   Probe Fails Again <── App Takes 60s to Warm Up <── Cold Container Restart
```

### Rules for Bulletproof Liveness Probes
1. **Never check external dependencies:** Do NOT connect to databases or caches in a liveness probe. If the DB has a 5-second latency spike, all your app pods restart simultaneously, causing a full cluster crash!
2. **Use Liveness as a Scalpel:** Liveness probes should only answer: *"Is the process deadlocked/frozen?"*
3. **Set generous timeouts:** Avoid `timeoutSeconds: 1` on JVM/Node apps; allow 3–5 seconds.

---

## 12. Readiness Probe Failures: The "Zero-Capacity" Service Blackhole

```
                        READINESS FAILURE TRAFFIC DROP
  Downstream DB Outage ──> Readiness Probes Fail ──> Pods Removed from Endpoints
                                                              │
                                                              ▼
                                                   Service Endpoints: 0/N
                                                   (100% User Traffic Drops!)
```

### Liveness vs. Readiness Failure Behavior
* **Liveness Failure:** Kubelet kills and **restarts the container**.
* **Readiness Failure:** Kubelet leaves container running but **removes its IP from the Service EndpointSlice**, isolating it from user traffic.

---

## 13. Startup Probe Misconfiguration & Slow Initialization Protection

```
                         PROBE EXECUTION TIMELINE
  Container Starts ──> [ Startup Probe Running ] ──> Success!
                                                          │
                               ┌──────────────────────────┴──────────────────────────┐
                               ▼                                                     ▼
                     [ Liveness Probe Active ]                             [ Readiness Probe Active ]
```

* **Purpose:** Shields slow-starting applications (JVM Spring Boot class loading, cache warming, database migrations) from premature liveness restarts.
* **Calculation:** $\text{Max Startup Window} = \text{periodSeconds} \times \text{failureThreshold}$. (e.g., $10\text{s} \times 30 = 300\text{s}$ safe startup buffer).

---

## 14. Init Container Failures: Blocking the Entire Application Lifecycle

```
                       INIT CONTAINER EXECUTION ORDER
  Pod Scheduled ──> [ Init-1: DB Migrate ] ──> [ Init-2: Config Gen ] ──> [ App Containers ]
                           │
                         Crash! (Init:Error)
                           │
                    App Never Starts!
```

### Common Pitfalls
1. **Non-Idempotent Migrations:** If an init container runs a SQL script that fails on duplicate table creation upon pod restart.
2. **Missing Network Connectivity:** Init container cannot resolve database DNS before CoreDNS or CNI is ready.

---

## 15. ConfigMap Changes That Break Production & In-Memory Drift

```
                       CONFIGMAP MOUNT UPDATE REALITY
 ┌────────────────────────────────────────┬────────────────────────────────────────┐
 │ Mounted as Volume File (`/etc/config`)  │ Injected as Environment Variable (`env`)│
 ├────────────────────────────────────────┼────────────────────────────────────────┤
 │ File updates automatically on disk    │ Value is STATIC. App NEVER sees updates│
 │ within 60-120 seconds.                 │ until pod is killed and recreated.     │
 └────────────────────────────────────────┴────────────────────────────────────────┘
```

> [!WARNING]
> If application code reads config files only once at startup, disk updates will create **configuration drift** between pods without triggering any alert or pod restart!

---

## 16. Secret Failures, Base64 Traps & External Secret Sync Outages

### Top Secret Outage Scenarios
1. **Base64 Double-Encoding Trap:** Running `echo "password" | base64` includes a hidden newline `\n`. Always use `echo -n "password" | base64`.
2. **External Secrets Operator Sync Failure:** HashiCorp Vault token expires; ESO fails to sync updated database credentials to the Kubernetes Secret.
3. **IRSA / IAM Workload Identity Failure:** Missing trust relationship on AWS IAM role prevents pod from fetching cloud secrets.

---

## 17. Deployment Rollout Failures & ProgressDeadlineExceeded

```
                      PROGRESSDEADLINEEXCEEDED TIMELINE
  Rollout Started ──> New Pods Crash/Pending ──> 600s Timer Expires (Default)
                                                          │
                                                          ▼
                                          Deployment Condition:
                                          Type: Progressing, Status: False
                                          Reason: ProgressDeadlineExceeded
```

### Key Deployment Safety Parameters
* `spec.progressDeadlineSeconds: 600` (Flags failed rollouts after 10 minutes).
* `spec.minReadySeconds: 30` (Ensures new pods remain healthy for 30s before considering them available).
* `spec.strategy.rollingUpdate.maxUnavailable: 0` (Prevents dropping existing capacity during deployments).

---

## 18. Failed Rollbacks & The Distributed State Schema Trap

```
                        THE FAILED ROLLBACK TRAP
  v1 App ──> v2 App Deployed (DB Schema Altered) ──> v2 Fails ──> Rollback to v1 App
                                                                         │
                                                                         ▼
                                                             v1 App Crashes!
                                                             (Cannot read new DB schema)
```

### The Expand-Contract (Parallel Change) Pattern
1. **Phase 1 (Expand):** Add new database columns without removing old ones. (Both v1 and v2 app work).
2. **Phase 2 (Deploy v2):** Write to both, read from new column.
3. **Phase 3 (Contract):** After v2 stabilizes in production, run cleanup migration to drop legacy v1 columns.

---

## 19. High-Severity Workload Outage Incident Response Playbook

```
                         INCIDENT RESPONSE WORKFLOW
  [01. Alert] ──> [02. Verify Impact] ──> [03. Freeze Changes] ──> [04. Inspect Rollout]
                                                                          │
  [07. Review Logs] <── [06. Check Events] <── [05. Inspect Pods] <───────┘
        │
        └──> [08. Validate Resources] ──> [09. Validate Dependencies]
                                                     │
              [11. Post-Mortem] <── [10. Execute Recovery Decision]
```

### Emergency Recovery Decision Tree
* **Scenario A: Bad Code Release $\rightarrow$** `kubectl rollout undo deployment/<name> -n <ns>`
* **Scenario B: Capacity Starvation $\rightarrow$** Scale up node group or scale down non-critical batch workloads.
* **Scenario C: Misconfigured Liveness Probe $\rightarrow$** Temporarily patch deployment to disable probe while diagnosing.
* **Scenario D: ConfigMap Typo $\rightarrow$** Revert ConfigMap and execute `kubectl rollout restart deployment/<name>`.

---

## 20. Pre-Deployment & Active-Incident Production Readiness Scorecard

| Category | Verification Item | Production Status |
| :--- | :--- | :---: |
| **Pre-Deploy** | Manifest validated via `kubectl apply --dry-run=server` and `kubectl diff` | [ ] |
| **Pre-Deploy** | Immutable image tags pinned (no `:latest` or mutable branch tags) | [ ] |
| **Pre-Deploy** | CPU/RAM `requests` and `limits` defined on all containers | [ ] |
| **Pre-Deploy** | Liveness probe does NOT check downstream external dependencies | [ ] |
| **Pre-Deploy** | Startup probe configured for slow-initializing frameworks (JVM/Spring) | [ ] |
| **Pre-Deploy** | PodDisruptionBudget (PDB) configured for multi-replica services | [ ] |
| **In-Incident**| Events inspected via `kubectl get events --sort-by=.lastTimestamp` | [ ] |
| **In-Incident**| Previous container logs captured via `kubectl logs <pod> --previous` | [ ] |
| **In-Incident**| Exit codes checked (137 OOMKill vs 1 Application Error vs 143 SIGTERM) | [ ] |
| **In-Incident**| Database backward compatibility verified before executing rollback | [ ] |
| **Post-Incident**| Root-cause documented, prevention action items created in issue tracker | [ ] |
