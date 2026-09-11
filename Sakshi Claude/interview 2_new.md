# 🚀 DevOps Interview Master Guide v2 (Fully Tool-Wise)

> Consolidated, long-term-memory reference merging all DevOps interview material — organized **by tool/technology**, not by video/session. Nothing is removed from the source material; everything is regrouped so related concepts live together.

## 📌 Quick Overview: The 5-Step Project Narrative

| Step | Topic | What It Covers |
|------|-------|-----------------|
| 1 | Infrastructure Provisioning | Terraform (VPC, Subnets, EKS Cluster) |
| 2 | Source Code Management (SCM) | Branching strategy, team access, webhooks |
| 3 | CI/CD Pipeline | Jenkins Declarative Pipeline (build, test, scan, push) |
| 4 | Application Deployment | Kubernetes Manifests / Helm Charts on EKS |
| 5 | Monitoring & Logging | Prometheus, Grafana, EFK stack, alerting |

**Simple way to remember it:** *Build the land (Terraform) → Store the code (Git) → Automate the pipeline (Jenkins) → Ship the app (K8s/Helm) → Watch it (Prometheus/Grafana/EFK).*

## Table of Contents

1. [Kubernetes](#1-kubernetes)
2. [Docker](#2-docker)
3. [Terraform / Infrastructure as Code](#3-terraform--infrastructure-as-code)
4. [Ansible](#4-ansible)
5. [AWS](#5-aws)
6. [Azure](#6-azure)
7. [Linux, Shell Scripting & SQL](#7-linux-shell-scripting--sql)
8. [CI/CD — Jenkins & GitHub Actions](#8-cicd--jenkins--github-actions)
9. [GitOps — Helm, ArgoCD & Flux](#9-gitops--helm-argocd--flux)
10. [Monitoring, Logging & SRE](#10-monitoring-logging--sre)
11. [Code Quality & DevSecOps (SonarQube, Scans)](#11-code-quality--devsecops-sonarqube-scans)
12. [RBAC & Security (Cloud + Kubernetes)](#12-rbac--security-cloud--kubernetes)
13. [OpenShift](#13-openshift)
14. [Real-World Migration & Disaster Recovery Scenarios](#14-real-world-migration--disaster-recovery-scenarios)
15. [DevOps Culture, Maturity & SDLC](#15-devops-culture-maturity--sdlc)
16. [End-to-End Project Narrative (Interview Script)](#16-end-to-end-project-narrative-interview-script)
17. [Career, Behavioral & Company-Specific Prep](#17-career-behavioral--company-specific-prep)
18. [Quick Recall Cheat Sheet](#18-quick-recall-cheat-sheet)

---

## 1. Kubernetes

### 1.1 Core Concepts & Manifests

**Deployment vs. Pod**

| Concept | Description |
|---------|--------------|
| **Deployment** | Manages the lifecycle of identical pods — replica count, update strategy, rollbacks |
| **Pod** | Smallest deployable unit; containers *inside the same pod* share a network namespace, talk via `localhost` |

**Why Deployment instead of a raw Pod?**
> A raw Pod has no self-healing and no rolling update capability. A `Deployment` manages ReplicaSets, giving you rolling updates, automated rollbacks, zero-downtime deploys, and scalability.

**Basic Deployment manifest:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app-name
spec:
  replicas: 2
  selector:
    matchLabels:
      app: your-app-name
  template:
    metadata:
      labels:
        app: your-app-name
    spec:
      containers:
        - name: your-app-container
          image: your_ecr_repo/image_name:tag
          ports:
            - containerPort: 80
```
```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/my-app
```

**Full Deployment YAML with probes + resources + PVC (common live-coding ask):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: my-app-pvc
```

**How to connect a Pod to a PVC (step-by-step):**
1. Create a Persistent Volume (PV)
2. Create a Persistent Volume Claim (PVC)
3. Mount the PVC inside your Pod spec

**Rolling update strategy:**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

**Basic deployment workflow:**
1. Containerize the app (Docker)
2. Push the image to a container registry
3. Write Kubernetes manifests (YAML)
4. Deploy the manifests to the cluster
5. Expose the application (optional)

**3-replica nginx Deployment example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

### 1.2 Service Types & Networking

| Type | Description |
|------|-------------|
| `ClusterIP` (default) | Exposes service **inside the cluster only** |
| `NodePort` | Exposes service on a fixed port on every node |
| `LoadBalancer` | Exposes service externally via cloud load balancer |
| `Headless` (`clusterIP: None`) | No virtual IP — DNS resolves directly to each individual pod |

**🎯 Headless Service — the big interview topic**

*The problem it solves:* A standard `ClusterIP` service load-balances traffic randomly/round-robin across pods via `kube-proxy`. This **breaks stateful systems** (databases, Kafka) where writes must go to a specific **primary/leader** pod while reads can go to replicas. A normal service hides *which* pod you're hitting.

*The fix:*
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```
- No virtual IP is assigned.
- CoreDNS returns **direct A-records for each pod** instead of one shared ClusterIP.
- Best paired with a **StatefulSet**, giving pods stable, predictable names (`db-0`, `db-1`, `db-2`).

**Resulting DNS pattern:**
```
<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local
```
Example: `db-0.my-db-headless.default.svc.cluster.local`

**Full example — StatefulSet DB + App:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-db
spec:
  serviceName: "my-db-headless"
  replicas: 3
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          ports:
            - containerPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: my-db
  ports:
    - port: 5432
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          env:
            - name: DATABASE_URL
              value: "postgresql://postgres@my-db-headless.default.svc.cluster.local:5432/mydb"
```

**Standard vs. Headless Service — comparison**

| Feature | Standard Service (ClusterIP) | Headless Service (`clusterIP: None`) |
|---------|-------------------------------|----------------------------------------|
| Cluster IP | Allocated from service CIDR | None |
| Routing/Proxying | Handled by `kube-proxy` (iptables/IPVS) | Bypasses `kube-proxy` — client resolves DNS directly |
| DNS Resolution | Returns single virtual ClusterIP | Returns multiple A-records / per-pod FQDNs |
| Best for | Stateless web apps, REST APIs | Stateful workloads: DB leaders/replicas, Kafka, Elasticsearch, Cassandra |

**General pod-to-pod communication**

| Scope | How They Talk |
|-------|----------------|
| Same Pod (container-to-container) | Directly via `localhost` (shared network namespace) |
| Same Namespace (pod-to-pod) | Short service DNS: `http://backend-service:8080` |
| Cross-Namespace | Full FQDN: `backend-service.backend.svc.cluster.local` |

**How traffic is actually forwarded:**
1. Pod sends a request to a Service's DNS name
2. **CoreDNS** resolves it to the Service's **ClusterIP**
3. The Service tracks matching pods via **Endpoints/EndpointSlice** (label selectors)
4. **kube-proxy** (per node) intercepts traffic and forwards it — typically round-robin via iptables/IPVS

```
Request 1 → Pod A
Request 2 → Pod B
Request 3 → Pod C
Request 4 → Pod A  (cycle repeats)
```

**Special routing modes**

| Mode | Use Case |
|------|----------|
| Headless Service | Direct, deterministic pod addressing — bypasses kube-proxy load balancing (stateful DBs) |
| Ingress Controller | Layer-7 routing for external HTTP/HTTPS — host-based (`api.example.com`) or path-based (`example.com/api`) |
| Egress Policies | Network policies restricting traffic *leaving* the cluster |

**🎯 Interview one-liner:**
> "Communication flows through a decoupled service abstraction. Containers in the same pod talk via `localhost`. Cross-pod calls target a Service's DNS name, which CoreDNS resolves to a ClusterIP; `kube-proxy` round-robins the request to a matching pod IP. For stateful workloads needing direct pod targeting, we use a Headless Service to bypass the proxy and expose deterministic per-pod DNS records."

**Ingress vs. Cloud Load Balancer — which to use?**

| Option | Cost/Behavior | Best For |
|--------|-----------------|----------|
| `Service type: LoadBalancer` | Allocates a **dedicated** cloud LB per exposed service → cost scales linearly | Raw TCP/UDP, non-HTTP, low-latency websockets |
| **Ingress Controller** | **One** entry point (ALB/VIP) multiplexes traffic across many services via path/host rules | Standard HTTP/HTTPS — much more cost-effective |

> **Can a Load Balancer work as an Ingress?** No — a standard L4 Load Balancer cannot do host/path-based HTTP routing like an Ingress controller.

**Kubernetes CronJobs**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup-cronjob
spec:
  schedule: "0 2 * * *"   # Daily at 02:00 UTC
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup-task
              image: backup-utility:1.2
```
Schedule format: `Minute | Hour | Day-of-Month | Month | Day-of-Week`. Use cases: DB snapshot exports, cert-manager validation runs, log archiving, cache warming.

### 1.3 Troubleshooting Scenarios (Consolidated)

| # | Scenario | Root Cause / Fix |
|---|----------|---------------------|
| 1 | Pod stuck in `CrashLoopBackOff` | `kubectl logs`/`kubectl describe pod` — wrong entrypoint, missing env/config, failing liveness probe, resource limits too low |
| 2 | Service not reachable | `kubectl get svc`/`describe svc` — check type, endpoints populated?, label mismatch, pod readiness, network policies |
| 3 | Pod stuck `Pending` | Insufficient cluster resources, node selector/affinity mismatch, taints without tolerations, PVC not bound |
| 4 | High CPU usage cluster-wide | `kubectl top pods`, check requests/limits, use HPA, investigate memory leaks, consider cluster autoscaling |
| 5 | Rolling update failure | `kubectl rollout status` → `kubectl rollout undo deployment <name>`; check probes, image, config |
| 6 | ConfigMap updated but pod didn't pick it up | ConfigMaps don't auto-restart pods — `kubectl rollout restart deployment <name>` |
| 7 | App can't read Secrets | Verify `kubectl get secrets`, check mount method, key names, RBAC permissions |
| 8 | Intermittent pod-to-pod comms failure | Network policies, DNS (CoreDNS) issues, service misconfig, pod restarts — debug via `kubectl exec` + curl/nslookup |
| 9 | Data disappears after pod restart | Pod using ephemeral storage — use PV + PVC with correct StorageClass |
| 10 | Node shows `NotReady` | `kubectl describe node` — kubelet status, disk/memory pressure, network; SSH in, restart kubelet, `journalctl -u kubelet` |
| 11 | Deployment deleted but pods still running | Orphaned ReplicaSets, pods from another controller (StatefulSet/DaemonSet), finalizer hooks, controller-manager API lag |
| 12 | Image updated in YAML, pod still uses old image | `imagePullPolicy` — mutable `:latest` tag **without** `imagePullPolicy: Always` prevents kubelet from pulling the new image |
| 13 | HPA configured, threshold exceeded, pod count stays 1 | Missing Metrics Server, missing `resources.requests` (HPA needs a baseline), or min/max replica constraints |
| 14 | OOMKilled with no resource limits | Without limits, a container can consume all node memory; memory spikes → Linux OOM killer terminates it (or namespace `LimitRange` blocks/defaults it) |
| 15 | Pod can't be scheduled | `kubectl describe pod/node`, `get events` — check CPU/memory, taints, tolerations, node selectors, affinity, resource quotas, PVC, availability |
| 16 | Debugging a service not accessible | `kubectl get svc -o yaml`, `get pods --show-labels`, `get svc <name>` — verify selector matches pod labels |

### 1.4 Advanced Kubernetes Internals

**Zero-Downtime Microservices Deployment**
- Combine Rolling Updates, Blue-Green, or Canary (Argo Rollouts, Istio traffic routing, feature flags)
- Accurate `readinessProbe`, `livenessProbe`, `startupProbe`
- `maxUnavailable: 0`, `maxSurge: 1` (or `25%`) — old pods only terminate after new pods pass readiness checks
- Canary analogy: shift traffic 5% → 20% → 50% → 100% while watching error rates/latency

**Worker Node Enters `NotReady`**
- `node-lifecycle-controller` detects missed heartbeats, marks node `NotReady`
- Scheduler stops assigning new pods; existing pods remain until eviction timeout (default 5 min via `pod-eviction-timeout`/`node.kubernetes.io/unreachable` tolerations), then rescheduled elsewhere
- **PodDisruptionBudgets (PDB):** guarantee minimum available instances
- **Stateless vs. StatefulSets:** stateless recreate easily; StatefulSets need CSI driver unmount/remount of PVs to the new node

**Disaster Recovery: etcd Crash**
- etcd = consistent key-value store holding entire cluster state (nodes, pods, secrets, configs)
- Impact: `kube-apiserver` loses read/write, becomes unresponsive. **Running workloads keep running** (kubelet + container runtime continue existing containers). Control plane freezes — no new scheduling/scaling/secrets/deployments.
- Mitigation: multi-node HA clusters (odd quorum: 3 or 5) across failure zones; restore via `etcdctl snapshot restore`

**Production Outage Troubleshooting Protocol**
1. Telemetry & Logs — CPU, memory, IOPS, connection pools, error rate surges
2. Layer Isolation — Network/DNS, Ingress/LB, Application logic, downstream DBs
3. Fast Mitigation — if unresolved in 5–10 min, rollback to last known-good deployment; save crash logs for post-mortem

**Pod-to-Internet Networking Path**
```
Pod Network Namespace → veth pair → Linux Bridge/CNI (Calico, Cilium, Flannel)
   → Node routing tables → iptables/eBPF (SNAT)
   → Node's physical NIC (eth0) → VPC Router/NAT Gateway → Internet Gateway → External Web
```

**Cluster Hardening (Defense-in-Depth)**

| Layer | Practices |
|-------|-----------|
| Admission Control | Admission webhooks — OPA Gatekeeper, Kyverno |
| Identity & Isolation | Strict least-privilege RBAC, namespace boundaries, NetworkPolicies |
| Runtime & Supply Chain | Sign images (Sigstore/Cosign), scan images (Trivy), runtime threat detection (Falco), KMS encryption for etcd secrets |

**Control Plane Lifecycle — `kubectl apply -f deployment.yaml` end-to-end**
1. `kubectl` sends HTTP request to `kube-apiserver`
2. API Server: Authentication → Authorization (RBAC) → Mutating/Validating Admission Controllers
3. Desired state persisted into **etcd**
4. Deployment/ReplicaSet Controller detects diff, creates Pod objects
5. `kube-scheduler` assigns pods to nodes based on resource filters/affinities
6. kubelet on target node instructs container runtime (containerd/CRI-O) to pull images and start containers, reports status back

**Other patterns:**
- **Sidecar Pattern** — auxiliary container alongside main app for logging, proxying, secret sync
- **Service Mesh** — service-to-service communication, mTLS, traffic shifting, observability (Istio, Linkerd)
- **GitOps** — reconciling cluster state against declarative Git repos (Flux CD / ArgoCD) — see [Section 9](#9-gitops--helm-argocd--flux)

### 1.5 kubectl Cheat Sheet

**Declarative vs. imperative**

| Approach | Description |
|----------|--------------|
| **Imperative** | `kubectl create ns test`, `kubectl run nginx --image=nginx`, `kubectl scale deployment ...` |
| **Declarative** | Define desired state in YAML, then `kubectl apply -f manifest.yaml` |

**Inspecting**
```bash
kubectl get ns
kubectl get pods -n <namespace_name>
kubectl describe pod <pod_name>
kubectl get pods -o wide
```

**Creating**
```bash
# Imperative
kubectl create ns my-app
kubectl run my-pod --image=nginx -n my-app
# Declarative
kubectl apply -f ingress-gateway.yaml -n <namespace>
```

**Debugging & logging**
```bash
kubectl logs <pod_name>
kubectl exec -it <pod_name> -- /bin/bash
kubectl get pods --watch
```

**Scaling & deletion**
```bash
kubectl scale deployment <deployment_name> --replicas=2
kubectl delete pod <pod_name>
kubectl delete ns <namespace_name>
```

**Checking pod logs**
```bash
kubectl logs pod-name -n namespace-name
```

### 1.6 Blue-Green Deployment — Explained Simply

| Environment | Role |
|-------------|------|
| **Blue** | Active — currently serving all live production traffic (older version) |
| **Green** | Idle — new version deployed and tested under production-like conditions |

**Zero-downtime cutover:** once Green passes validation, the router/load balancer switches traffic from Blue to Green **instantly**. Blue stays on standby for immediate rollback or is decommissioned later.

**Traffic routing mechanism (Ingress/Service level):**
- **Ingress backend switch:** update `backend.service.name` from `blue-service` → `green-service`
- **Service selector switch:** change the Service's label selector (`version: v1.0` → `v2.0`)
- DNS/URL stays identical throughout

**Alternating roles reminder:** Blue/Green are **not fixed labels** — they alternate every release. After v1→v2 cutover, Green is now active (serving v2), and Blue becomes the new staging target for v3.

**Lab practice vs. enterprise reality**

| Context | Approach |
|---------|----------|
| Local Lab (Minikube) | Simulate with multiple profiles: `minikube start -p green --driver=docker` |
| Enterprise Myth | "Two entire duplicate Kubernetes clusters" — too expensive in practice |
| Enterprise Reality | Two **namespaces** in the same cluster shifting Ingress/mesh traffic; OR side-by-side `app-blue`/`app-green` Deployments in the same namespace, switching Service selector or Service Mesh (Istio/Linkerd) traffic weight; OR dedicated node pools via selectors/taints |

**Implementation patterns for EKS specifically**
- **Dual In-Cluster Deployments** (cost-effective): two Deployments (`app-blue`, `app-green`) in the same cluster; switch ALB listener rule target-group weights (0/100 split) or the Service selector label
- **Dual-Cluster Blue/Green** (high isolation, critical banking): two identical EKS clusters, traffic switching via Route 53 (weighted/failover routing)

**Full production scenario:** financial app, ~1M transactions/day, 30-min downtime per release because updates happen in-place → goal: zero downtime + instant rollback. See [Section 14.8](#14-real-world-migration--disaster-recovery-scenarios) for the full breakdown.

### 1.7 AKS (Azure Kubernetes Service) Specific

**What is AKS?** Microsoft's managed Kubernetes service — Azure manages the control plane; you manage applications, node pools, networking, RBAC, security, scaling, observability.

```
                    Azure
                      |
                     AKS
                      |
          +-----------+-----------+
          |                       |
    Control Plane             Node Pools
                                  |
                       +----------+----------+
                       |          |          |
                     Node       Node       Node
                       |          |          |
                      Pod        Pod        Pod
```

**Node Pool:** group of nodes with a particular VM config — **System Node Pool** (core K8s workloads) vs. **User Node Pool** (application workloads).

**Scaling**
- **HPA:** changes **Pod count** (3 → 10) based on CPU/memory/custom metrics
- **Cluster Autoscaler:** changes **Node count** (3 → 6)
- **KEDA:** scales workloads based on event-driven metrics (queues)

**AKS pulling images from ACR**
```
AKS → Identity → Azure RBAC → ACR → Pull Image
```
Common permission: `AcrPull`. Don't give AKS full `Contributor` access to ACR when it only needs to pull images.

**AKS Cluster Upgrade (v1.27 → v1.30) — full scenario** in [Section 14.2](#14-real-world-migration--disaster-recovery-scenarios).

**AKS Disaster Recovery (Velero-based)** in [Section 14.7](#14-real-world-migration--disaster-recovery-scenarios).

**Windows Containers / .NET on AKS Modernization** in [Section 14.10](#14-real-world-migration--disaster-recovery-scenarios).

---

## 2. Docker

### 2.1 Image Optimization & Cleanup

- **Reduce image size:** minimal base images (Alpine, Distroless), multi-stage builds, chain `RUN` commands to reduce layer count, avoid unnecessary dependencies
- **Clean unused containers/images:**
  ```bash
  docker system prune -a
  ```

### 2.2 Docker Interview Questions by Difficulty

**Beginner**

| Question | Simple Answer |
|----------|-----------------|
| What is a Dockerfile? | A text manifest of instructions packaging an app + dependencies into an immutable, platform-independent image |
| `ADD` vs. `COPY` | `COPY` copies local files/dirs plainly. `ADD` also auto-extracts local tar archives and can fetch remote URLs |
| `ENTRYPOINT` vs. `CMD` | `ENTRYPOINT` = immutable base executable. `CMD` = default arguments, overridable at `docker run` |
| `WORKDIR` | Sets the working directory for subsequent `RUN`/`CMD`/`ENTRYPOINT`/`COPY` |
| `EXPOSE` | Documents which ports the container listens on (informational — `-p` still needed to actually map ports) |
| Multiple `FROM` instructions | Allowed/standard — used for **multi-stage builds** to discard build-time dependencies |
| Reducing image layers | Combine shell commands with `&&` in a single `RUN` (e.g., `apt-get update && apt-get install -y ... && rm -rf /var/lib/apt/lists/*`) |

**Intermediate**

| Question | Simple Answer |
|----------|-----------------|
| `RUN` vs. `CMD` vs. `ENTRYPOINT` | `RUN` executes at **build time**, commits a new layer. `CMD`/`ENTRYPOINT` execute at **container startup** |
| `ARG` vs. `ENV` | `ARG` only exists during build (`docker build --build-arg`); `ENV` persists into the running container |
| Multi-stage builds | Compile in a "builder" image (Maven/Go/Node), copy only the compiled output into a lightweight runtime image (Alpine/Distroless) — dramatically smaller final image |
| Risks of `COPY . .` in CI/CD | Blindly copies test artifacts, `.git` history, cache folders, potential secrets — always pair with a strict `.dockerignore` |
| Image versioning/metadata | Use `LABEL` (maintainer, Git commit SHA, build version) for traceability |

**Advanced**

| Question | Simple Answer |
|----------|-----------------|
| Handling secrets in builds | Never bake secrets into `ENV`/`ARG`. Inject at runtime via env vars/secret stores, or use BuildKit secret mounts (`--mount=type=secret`) |
| Container starts and immediately exits | Check `docker logs <container_id>`; inspect the `CMD`/`ENTRYPOINT` foreground process — container exits when PID 1 finishes/crashes |
| Risk of `:latest` tag | Breaks reproducibility, silently pulls unexpected base image changes, hurts caching. **Always pin specific version tags or SHA digests** |
| Docker build cache mechanics | Cache invalidates from the **first modified layer downward**. Put infrequently-changing instructions (e.g., `package.json`/`pom.xml`) **before** copying volatile source code |
| Deterministic builds | Pin explicit base image tags/digests, lock dependencies (`package-lock.json`), avoid mutable downloads |
| Security hardening | Run as non-root (`USER <non-root-uid>`), use minimal base images (Alpine/scratch/Distroless), scan images (Trivy/Grype), keep `.dockerignore` current |

**Docker Networking:** network drivers (`bridge`, `host`, `overlay`, `none`), container-to-container communication, port mapping (`-p`), port exposure (`EXPOSE`).

**Containers vs. VMs**

| | Containers | Virtual Machines |
|---|---|---|
| Virtualization level | OS-level, share host kernel | Hardware-level, hypervisor-based |
| Footprint | Lightweight | Full dedicated guest OS |

**Practice ask:** write a Dockerfile for a Python/Node.js application.

---

## 3. Terraform / Infrastructure as Code

### 3.1 State Management

**State File Conflict in Team** — multiple devs applying simultaneously corrupts state.
> Use a remote backend with locking:
```hcl
backend "s3" {
  bucket         = "my-tf-state"
  key            = "prod/terraform.tfstate"
  region         = "ap-south-1"
  dynamodb_table = "tf-lock"
}
```
On Azure: Azure Blob Storage with native lease-based locking.

**Handling concurrent state execution:** when multiple engineers/pipelines trigger Terraform simultaneously, state locking (DynamoDB or Azure Blob lease) ensures only one process acquires the lock, preventing race conditions and corrupted state.

**Recovering a deleted state file:** restore via remote backend versioning (S3/Azure Blob object versioning), or rebuild mappings using `terraform import`.

**Secrets visible in Terraform state:** encrypt remote backends, use ephemeral outputs, delegate secret retrieval to KMS/Vault at runtime. Mark variables `sensitive = true` to redact console/log output — but this does **not** remove them from the raw state file, so backend encryption still matters.

**Drift between infra and code:** `terraform plan` detects drift → `terraform apply` to revert, OR `terraform import` if untracked. Avoid manual console changes; enforce IaC discipline.

### 3.2 Structuring for Multiple Environments

**Approach A — Module Directory Pattern (most popular in enterprise)**
- Core infra (vpc, compute, database, storage) as reusable versioned modules under `/modules`
- Separate environment directories: `/environments/dev`, `/environments/qa`, `/environments/prod`
- Each env directory has its own `backend.tf`, `main.tf` (calling shared modules), `terraform.tfvars`, outputs

**Approach B — Workspace-Based Pattern**
- Single config, switch environments with `terraform workspace select <env>`
- **Interview note:** module-per-directory with separate state backends is typically preferred in production — workspaces share the same backend storage and risk cross-environment blast radius

**Also common:**
```bash
terraform apply -var-file=dev.tfvars
terraform apply -var-file=prod.tfvars
```
Best practice: separate backend configs per environment.

### 3.3 Modules — Deep Dive

**Why modules?**

| Reason | Simple Explanation |
|--------|----------------------|
| Reusability (DRY) | Define infra once, reuse across Dev/QA/Staging/Prod |
| Separation of Concerns | Keep networking (VPC), control-plane (EKS), compute (nodes) in separate directories |

**Standard enterprise layout:**
```
terraform-root/
├── main.tf                 # Root orchestration calling child modules
├── variables.tf            # Global root input variables
├── outputs.tf              # Aggregated root outputs
├── providers.tf            # Provider + version constraints
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── eks/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── ec2_node/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

**VPC Module** — creates `aws_vpc`, public/private subnets across AZs, `aws_internet_gateway`, `aws_nat_gateway` with EIPs. Exports IDs:
```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}
output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

**EKS Module** — IAM Cluster Role (`AmazonEKSClusterPolicy`) + `aws_eks_cluster`. Takes `vpc_id`/`subnet_ids` as **inputs** (never hardcoded). Exports `cluster_name`, `cluster_endpoint`, `cluster_certificate_authority_data`.

**Worker Node Module** — node IAM roles (`AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`) + `aws_eks_node_group`. Scaling via `min_size`/`max_size`/`desired_size` variables.

**Root module wiring:**
```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  env    = var.environment
}
module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
module "ec2_node" {
  source       = "./modules/ec2_node"
  cluster_name = module.eks.cluster_name
  subnet_ids   = module.vpc.private_subnet_ids
  node_count   = var.worker_node_count
}
```

**Module source types**

| Type | Example | When to Use |
|------|---------|-------------|
| Relative local path | `source = "./modules/vpc"` | Same repo |
| Remote Git repo | `source = "git::https://github.com/org/terraform-aws-vpc.git?ref=v2.1.0"` | Enterprise standard — version pinning |

**Key Q&A:**
> **"How do you pass data between modules?"** A child module's internal resources are private by default — it must expose the value via `outputs.tf`. The root module references it as `module.<module_name>.<output_name>`.
> **"Why avoid hardcoding in child modules?"** Hardcoding breaks reusability across environments — all environment-specific values must flow through `variables.tf`.

### 3.4 Unexpected Resource Destruction

- **Never run `apply` blindly** — inspect the plan for what triggered replacement (immutable argument change or identifier rename)
- **Renamed resource:** e.g. `aws_db_instance.db1` → `db2` — Terraform sees `db1` deleted, `db2` new
  - Fix with `moved` block (1.1+):
    ```hcl
    moved {
      from = aws_db_instance.db1
      to   = aws_db_instance.db2
    }
    ```
- **Destruction safeguard:** `lifecycle { prevent_destroy = true }` on critical stateful resources so `apply` fails rather than destroying data

### 3.5 Sensitive Data, Lifecycle, Dependencies

```hcl
variable "db_password" {
  sensitive = true
}
```
Use environment variables and secret managers (AWS Secrets Manager, Azure Key Vault, Vault).

**Zero downtime resource replacement:**
```hcl
lifecycle {
  create_before_destroy = true
}
```

**Dependencies:**
```hcl
depends_on = [aws_instance.app]
```
- **Implicit:** Terraform infers ordering from attribute references
- **Explicit:** manually specified via `depends_on` when no direct reference exists but ordering matters (e.g., IAM role before EKS cluster)

### 3.6 Scaling: `count` vs. `for_each`

| Scenario | Preferred Construct | Why |
|---|---|---|
| Identical config, distinct names | `count = 20` | Index addressing — simple for homogeneous arrays |
| Heterogeneous configs (VM sizes, subnets, OS) | `for_each = var.instances_map` | Map keys — removing an item by key doesn't re-index the rest |

```hcl
resource "aws_instance" "servers" {
  count = 3
}
```
OR
```hcl
for_each = toset(["dev","prod"])
```

**Targeted deletion pitfall with `count`:** deleting a specific index can cause Terraform to destroy/recreate all subsequent indexed items.
```bash
terraform destroy -target="azurerm_linux_virtual_machine.vm[1]"
```
**Best practice:** use `for_each` with unique map keys.

### 3.7 Conditional Creation & Advanced HCL Blocks

```hcl
count = var.environment == "dev" ? 1 : 0
```

```hcl
resource "aws_instance" "example" {
  count = var.environment == "dev" ? 1 : 0

  lifecycle {
    create_before_destroy = true
    prevent_destroy        = false
    ignore_changes          = [tags]
  }

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port = ingress.value.from_port
      to_port   = ingress.value.to_port
      protocol  = ingress.value.protocol
    }
  }
}
```
- `lifecycle` blocks: `create_before_destroy`, `prevent_destroy`, `ignore_changes`
- `dynamic` blocks: for repeating nested arguments (e.g., security group rules)

### 3.8 Rollback, Import & Data Sources

**Rollback of a failed apply:** Terraform is not fully transactional. Fix the issue, re-run `apply`. Use version control for code rollback; consider blue/green for infra changes.

**Importing existing infrastructure:**
```bash
terraform import aws_instance.example i-123456
```
Then write matching `.tf` config, run `terraform plan` to sync. (Terraform 1.5+: declarative `import {}` blocks auto-generate config.)

**Data sources vs. resources:**
- `resource`: declares infra Terraform creates/manages
- `data`: read-only queries for existing infra outside the current root module (e.g., pre-existing subnet ID, latest AMI)

### 3.9 Meta-Arguments, Sentinel & Delivery Models

- **Meta-arguments:** `depends_on`, `count`, `for_each`, `provider`, `lifecycle`
- **Terraform Sentinel** (Policy-as-Code, Cloud/Enterprise): guardrails before applying — *Soft Mandatory* (warn/override, e.g. missing cost-center tag) vs. *Hard Mandatory* (blocks apply, e.g. port 22 open to `0.0.0.0/0`)

**Delivery models:**
- **Terraform CLI in CI/CD:** runs on pipeline runners (Azure DevOps agents, GitHub Actions runners, Jenkins nodes) with CLI installed. State/locking via S3+DynamoDB or Azure Blob lease. Credentials via OIDC/federated workload identities or IAM instance profiles — avoiding long-lived API keys.
- **Terraform Cloud/Enterprise:** managed remote state, VCS-driven workspaces, audit logs, private module registries.

### 3.10 Terraform + Azure Integration

```
Terraform → Azure Provider → Microsoft Entra Authentication → Azure Subscription
    → VNet / AKS / ACR
```
```bash
terraform init
terraform validate
terraform plan
terraform apply
```
State stored in Azure Storage Account → Blob Container; plan for access control, encryption, locking, versioning/recovery.

### 3.11 Hands-On: Terraform + Jenkins → EC2 (Mini Project)

**Prerequisites:** Jenkins with Terraform + Git plugins, GitHub repo with Terraform code, AWS CLI credentials in Jenkins, IAM permissions for EC2.

**`main.tf`:**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform-EC2"
  }
}
```

**Jenkinsfile:**
```groovy
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout Code') {
            steps { git 'https://github.com/your-username/your-repo.git' }
        }
        stage('Init Terraform') {
            steps { sh 'terraform init' }
        }
        stage('Plan Terraform') {
            steps { sh 'terraform plan' }
        }
        stage('Apply Terraform') {
            steps {
                input "Proceed with apply?"
                sh 'terraform apply -auto-approve'
            }
        }
    }
}
```
> 🔐 Never hardcode AWS credentials — store under **Manage Jenkins → Credentials**.

**Job setup:** New Item → Pipeline → "Pipeline script from SCM" → connect GitHub repo → Save & Build.

**Practice ask:** write a Terraform script to create an EC2 instance including VPC/subnet, and explain how to retrieve outputs.

---

## 4. Ansible

### 4.1 Fundamentals & Architecture

**Configuration management vs. provisioning**

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure provisioning (VMs, VPCs, subnets, storage) |
| **Ansible** | Configuration management — installs packages, manages config files, starts services post-provisioning |

**Push vs. pull architecture**

| Tool | Model | Agent Required? |
|------|-------|-------------------|
| **Ansible** | Push-based | ❌ Agentless — control node pushes over SSH |
| **Puppet / Chef** | Pull-based | ✅ Local agents periodically pull config from a master server |

**OS support:** Linux nodes via **SSH**; Windows nodes via **WinRM**. ⚠️ The Ansible control node itself **cannot run natively on Windows** (needs WSL), but *can* manage/configure Windows target nodes.

**Inventories**

| Type | Description |
|------|--------------|
| **Static** | Manually defined hosts/groups/IPs (default: `/etc/ansible/hosts`) |
| **Dynamic** | Scripts (Python/cloud plugins) querying cloud providers or Terraform outputs for live host IPs |

**Modules**

| Type | Description |
|------|--------------|
| **Core** | Built-in (`yum`, `apt`, `copy`, `service`, `win_copy`) |
| **Custom** | User-written (commonly Python) for bespoke automation |

**Execution commands:**
```bash
ansible-playbook -i <inventory_file> <playbook.yml> -v
ansible-playbook <playbook.yml> --syntax-check
ansible all -i <inventory> -m shell -a "date"
```

**Roles — modularizing playbooks**

| Directory | Purpose |
|-----------|---------|
| `tasks/` | Main execution steps |
| `handlers/` | Conditional tasks — run only when notified |
| `vars/` & `defaults/` | Variable definitions |
| `templates/` & `files/` | Config files & Jinja2 templates |
| `meta/` | Role metadata & dependencies |

**Ansible Vault (secrets encryption):**
```bash
ansible-vault encrypt <secrets_file.yml>
```

### 4.2 Playbook Syntax & Structure

**Core play directives:**
```yaml
- hosts: all
  become: true
  tasks:
    - name: Install git
      yum:
        name: git
        state: present

    - name: Copy file
      copy:
        src: /local/path/file.txt
        dest: /remote/path/file.txt
```

**Handlers — run only when notified:**
```yaml
tasks:
  - name: Configure Apache
    template:
      src: httpd.conf.j2
      dest: /etc/httpd/conf/httpd.conf
    notify: restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```
⚠️ `notify:` must **exactly match** the handler's `name:`. Handlers run once at the end of a play, only if triggered.

**Loops:**
```yaml
- name: Install list of packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - wget
    - vim
    - zip
```

**Variables:**
```yaml
vars:
  package_name: httpd

tasks:
  - name: Install web server
    yum:
      name: "{{ package_name }}"
      state: present
```

**Tags:**
```yaml
tasks:
  - name: Install packages
    yum:
      name: git
      state: present
    tags:
      - install
```
```bash
ansible-playbook playbook.yml --tags "install"
```

> **Interview advice:** Focus on understanding the overall architecture/syntax flow (`hosts`, `become`, `tasks`, module structure, YAML indentation). Forgetting a minor module parameter under pressure is fine as long as the logical execution flow is correct.

**Practice ask:** write a sample playbook to install `git`.

### 4.3 Terraform + Ansible — Working Together

**Division of labor:** *Terraform builds the house. Ansible furnishes and maintains it.*

**Workflow:**
1. Terraform provisions infra (EC2, VPC, DB)
2. Ansible configures software on top (web server, DB setup, app deploy)
3. Terraform **outputs** (IPs, instance IDs) feed into Ansible's inventory

**Typical repo structure:**
```
project/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── provider.tf
│   └── terraform.tfvars
├── ansible/
│   ├── inventory/
│   │   ├── prod.yaml
│   │   ├── dev.yaml
│   │   └── staging.yaml
│   ├── playbooks/
│   │   ├── setup.yaml
│   │   └── deploy.yaml
│   ├── roles/
│   │   ├── webserver/
│   │   ├── db/
│   │   └── app/
│   └── ansible.cfg
└── scripts/
    └── deploy.sh
```

**Option A — manual output passing:**
```bash
terraform init
terraform apply
export instance_ip=$(terraform output -raw instance_ip)
ansible-playbook -i ${instance_ip}, playbooks/setup.yaml
```

**Option B — dynamic inventory (preferred at scale):**
```python
#!/usr/bin/env python
import json, subprocess

def get_terraform_output():
    output = subprocess.check_output(['terraform', 'output', '-json'], universal_newlines=True)
    return json.loads(output)

def main():
    inventory = {'all': {'hosts': []}}
    tf_output = get_terraform_output()
    for resource in tf_output['instances']['value']:
        inventory['all']['hosts'].append(resource['public_ip'])
    print(json.dumps(inventory, indent=2))

if __name__ == "__main__":
    main()
```
> **Simple explanation:** the script asks Terraform "what did you just build?", turns the answer into a list of IPs, and Ansible uses that as its target hosts.

**Multiple environments:**
```bash
terraform workspace select prod
terraform apply
ansible-playbook -i inventory/prod.yaml playbooks/setup.yaml
```

**CI/CD pipeline (GitLab CI example):**
```yaml
stages:
  - terraform
  - ansible
  - deploy

variables:
  TF_VAR_environment: "dev"

terraform:
  stage: terraform
  script:
    - terraform init
    - terraform plan
    - terraform apply -auto-approve

ansible:
  stage: ansible
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/setup.yaml

deploy:
  stage: deploy
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/deploy.yaml
```

**How often do these run?**

| Tool | Frequency |
|------|-----------|
| Terraform | Weekly, or only when infra changes |
| Ansible | On every commit/PR — to deploy & configure services |

---

## 5. AWS

### 5.1 Core Networking & Resource Questions

| Question | Answer |
|----------|--------|
| Increase disk space on a Linux server? | Two-step: expand the storage (EBS resize) + resize the filesystem (`growpart` + `resize2fs`/`xfs_growfs`) |
| Restrict access to a specific S3 object? | S3 Bucket Policies or IAM Policies with object-level permissions |
| Install software on EC2 automatically at launch? | **User Data** (bootstrap script on first boot) |
| Where to place a NAT Gateway? | **Public Subnet** — must have an Elastic IP, requires a route to an Internet Gateway; lets private subnet instances reach the internet without inbound exposure |
| Connect a resource in AWS Account A to Account B? | **VPC Peering** — create request (Requester), accept (Accepter), update route tables, adjust SGs/NACLs. Alternatives: Transit Gateway (multi-VPC scale), PrivateLink (expose specific services), VPN/Direct Connect (hybrid) |
| Stop communication between pods in different namespaces (K8s)? | Define a `NetworkPolicy` restricting ingress/egress — `namespaceSelector`/`podSelector`; unmatched traffic denied by default once a policy exists (CNI-dependent) |

### 5.2 Top 10 AWS Cloud Scenario Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | Traffic spikes 10x on single EC2 | ELB + Auto Scaling Groups, static content on S3 + CloudFront, RDS Multi-AZ, optional ElastiCache |
| 2 | AWS bill doubled | Cost Explorer analysis, resize/stop underutilized EC2, Reserved Instances/Savings Plans, Auto Scaling, S3 lifecycle to Glacier, right-size RDS |
| 3 | Region failure — DR strategy | Multi-region architecture, cross-region replication (S3/RDS), Route 53 failover; choose by RTO/RPO: Backup & Restore → Pilot Light → Warm Standby → Active-Active |
| 4 | Secure data storage in S3 | SSE-S3/SSE-KMS encryption + HTTPS, least-privilege IAM, block public access, versioning + MFA delete, CloudTrail |
| 5 | Microservices on ECS communication | Service discovery (Cloud Map), internal LB, SQS (decouple)/SNS (fan-out), IAM roles + VPC networking |
| 6 | Scattered logs, random failures | Centralize with CloudWatch Logs + Insights, alarms (CPU/memory/errors), X-Ray tracing, dashboards |
| 7 | RDS slow due to heavy reads | Read Replicas, ElastiCache, query/index optimization, Aurora, connection pooling |
| 8 | CI/CD pipeline design | CodeCommit → CodeBuild → CodeDeploy → CodePipeline; approval steps + rollback |
| 9 | EC2 monolith → serverless | Microservices + Lambda + API Gateway + DynamoDB/S3 + event-driven (SQS/SNS) |
| 10 | Different teams, different access | IAM roles/policies (least privilege), IAM groups, MFA, AWS Organizations |

---

## 6. Azure

### 6.1 Foundations

**What is Microsoft Azure?** Cloud platform for Compute, Networking, Storage, Databases, Containers, Kubernetes, Identity, Security, Monitoring, DevOps, AI/analytics. Key DevOps services: **AKS, ACR, Virtual Network, Key Vault, Managed Identity, Azure Monitor, Log Analytics, Storage, Azure RBAC**.

**Resource Group:** logical container for resources sharing the same lifecycle/administration boundary — a **management boundary**, not necessarily a network boundary.

**Subscription hierarchy:**
```
Azure Tenant → Management Groups → Subscriptions → Resource Groups → Resources
```
Provides billing boundary, resource boundary, access-control boundary, quota boundary.

### 6.2 Identity, RBAC & Managed Identity

**Microsoft Entra ID** (formerly Azure AD): identity and access management — Users, Groups, Applications, Service Principals, Managed Identities, Authentication, Authorization.

**RBAC vs. Entra ID:** Entra ID → identity/authentication (**AuthN**). Azure RBAC → what an identity is authorized to do (**AuthZ**).
```
GitHub Actions → OIDC authentication → Entra ID → Azure RBAC → ACR / AKS / Storage
```

**Azure RBAC roles:** `Reader`, `Contributor`, `Owner`, `AcrPush`, `AcrPull`, `Virtual Machine Contributor`. Use least privilege — don't grant `Contributor`/`Owner` when a specific role suffices.

**Managed Identity — very important**
```
AKS → Managed Identity → Azure Key Vault
```
- **System-assigned:** created with the resource; deleted when the resource is deleted
- **User-assigned:** created separately, assignable to multiple resources

**Why better than storing credentials?** Avoids manually storing/rotating `CLIENT_ID`/`CLIENT_SECRET`/`PASSWORD`.
```
Azure Resource → Managed Identity → Entra ID → Azure Resource
```

**AKS app accessing Key Vault:**
```
Application Pod → AKS Workload Identity → Microsoft Entra ID → Azure RBAC → Key Vault
```

**Managed Identity vs. Service Principal:**
- **Service Principal:** app registration, manual credential/secret rotation
- **Managed Identity:** Azure-managed credential, auto-rotated by Entra ID

**Types of Managed Identities:** System-assigned, User-assigned.

**OAuth for AKS:** configure OIDC/Workload Identity federation so pods authenticate to Azure AD without storing credentials.

### 6.3 Key Vault & Secrets

Stores Secrets, Encryption keys, Certificates (DB password, API key, TLS cert). Avoid putting secrets in Git, Docker images, Terraform code, or K8s YAML — use Key Vault + identity/access mechanism.

### 6.4 Networking

**VNet:**
```
VNet
 +---- Subnet A
 +---- Subnet B
 +---- Subnet C
```

**Subnet extension pitfall:** Can an existing subnet be extended from `/24` to `/23`? **No** — you cannot change the IP range of an existing Azure subnet while resources/NICs are attached.

**NSG:** controls inbound/outbound traffic via rules (source, destination, port, protocol, direction). **NSG vs. Azure Firewall:** NSG = basic subnet/NIC filtering; Firewall = managed, centralized, advanced.

**Load Balancer types:**

| Service | Layer | Scope | Notes |
|---------|-------|-------|-------|
| Azure Load Balancer | L4 (TCP/UDP) | Regional | Basic traffic distribution |
| Application Gateway | L7 (HTTP/HTTPS) | Regional | URL/host routing, TLS termination, WAF |
| Traffic Manager | DNS-level | Global | Routes at DNS layer, no data path involvement |
| Front Door | L7 | Global | Edge routing + CDN + WAF, actively proxies traffic |

**Can a Load Balancer work as an Ingress?** No — L4 LB can't do HTTP host/path routing.

**Private Endpoint vs. Service Endpoint**
- **Private Endpoint:** private IP in your VNet for a PaaS service — not exposed publicly for that access path
- **Service Endpoint:** extends VNet identity to Azure services over the Azure backbone (service still has a public IP internally)

**Azure DNS:** hosting/management; **Private DNS Zones** matter for private resources.

**Troubleshooting:**
- **VNet Peering failure:** non-transitive peering, asymmetric/missing peering (must be set up on both sides), overlapping CIDRs, restrictive NSG rules
- **Private Endpoint unreachable from on-prem:** DNS resolution failure (private DNS zone not forwarding/resolvable via Azure Private DNS Resolver), missing route tables, VPN/ExpressRoute routing omissions
- **On-prem to Azure connectivity options:** Site-to-Site VPN, Point-to-Site VPN, dedicated private line via Azure ExpressRoute

### 6.5 Compute & Storage

**Managed Disk tiers:** Standard HDD, Standard SSD, Premium SSD, Ultra Disk.

**VM Scale Sets (VMSS):** scaling, HA, load balancing, automated updates — AKS node pools can be backed by VMSS infra.

**Storage types:** Blob (object storage), Azure Files (managed shares), Queue (messaging), Table (NoSQL key-value).

**Storage Account:**
```
Storage Account
 +-- Blob
 +-- File
 +-- Queue
 +-- Table
```
Security: Entra ID, RBAC, SAS, access keys, private endpoints, encryption — prefer identity-based access over distributing storage keys.

**AKS accessing storage in a different region:** works but adds latency — use Blob CSI driver/SDK with Managed/Workload Identity; verify network path (private endpoint/VNet peering if private).

### 6.6 AKS Specific — see [Section 1.7](#17-aks-azure-kubernetes-service-specific)

**Azure Functions / Web Apps / Logic Apps:**
- **Functions:** serverless, event-driven compute
- **Web Apps (App Service):** managed hosting with deployment slots
- **Logic Apps:** low-code workflow automation/orchestration

**Log correlation:** shared correlation/trace ID across services; configure Log Analytics workspaces to ingest logs from multiple app resources, query via KQL.

**Application Gateway backend health down:** mismatched probe path/port, backend NSG blocking App Gateway subnet, SSL cert mismatch, backend listening on `localhost` instead of `0.0.0.0`.

### 6.7 Azure Policy & Governance

- **"Denied by policy" on VM creation:** check policy definitions and effects
- **Types of Effects:** `Deny`, `Audit`, `Modify`, `DeployIfNotExists`
- **Azure Policy:** enforces standards/compliance (required tags, approved regions, required configs)
- **Policy vs. RBAC:** RBAC = who can do what; Policy = what configurations are allowed/required
- **Azure Landing Zone:** structured foundation addressing Identity, Networking, Governance, Security, Subscriptions, Policies, Logging, Management

### 6.8 HTTP Status Codes

| Code | Meaning |
|------|---------|
| **500** | Internal Server Error — unhandled error in the app |
| **502** | Bad Gateway — upstream sent an invalid response to the gateway/proxy |
| **503** | Service Unavailable — overloaded/maintenance |
| **504** | Gateway Timeout — upstream didn't respond in time |

### 6.9 CI/CD in Azure

```
GitHub → GitHub Actions → Tests + Security → Docker Build → ACR → GitOps (Flux) → AKS → Pods
```
Supporting services: Key Vault → Secrets, Azure Monitor → Monitoring, Log Analytics → Logs, Entra ID → Identity, RBAC → Authorization.

**OIDC federation (preferred over client secrets):**
```yaml
- name: Azure Login
  uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```
```
GitHub Actions → OIDC Token → Microsoft Entra ID → Federated Credential → Azure RBAC → Azure Resources
```
> **Why OIDC?** A client secret is long-lived — if leaked, standing access. OIDC provides short-lived federated authentication.

**Azure DevOps vs. GitHub Actions:** Azure DevOps = Repos, Pipelines, Boards, Artifacts, Test Plans. GitHub Actions = native GitHub CI/CD. Concepts transfer easily between them.

**Azure DevOps Pipeline architecture:**
```
Git → Azure DevOps Pipeline → Build → Test → Security Scan → Docker Build → ACR → AKS
```
With GitOps: `Pipeline → Build image → ACR → Update Git deployment config → Flux → AKS`

### 6.10 Monitoring in Azure

**Azure Monitor vs. Log Analytics:** Monitor = broad platform (Metrics, Logs, Alerts, Telemetry). Log Analytics = workspace for querying logs via **KQL**.
```kql
AzureActivity
| where TimeGenerated > ago(1h)
| summarize count() by ResourceGroup
```

**Application Insights:** APM within Azure Monitor — Requests, Response times, Exceptions, Dependencies, Availability.
```
User → Application → Application Insights → Azure Monitor
```

**Monitoring AKS:**
- K8s-level: `kubectl get pods`, `kubectl top pods`, `kubectl top nodes`
- Azure-level: Azure Monitor, Container Insights, Log Analytics, Application Insights
- Metrics: CPU, memory, pod restarts, node health, API latency, error rates, request volume, disk, network, exceptions

**End-to-end troubleshooting flow:**
```
User → DNS → Front Door / Application Gateway → Load Balancer → Kubernetes Service → Pod → Application → Database/external dependency
```
```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get svc
kubectl get ingress
kubectl get endpoints
kubectl get events
```
Then Azure-side: Azure Monitor, Log Analytics, Application Insights, NSG, Load Balancer/App Gateway, network connectivity, Key Vault.

### 6.11 Common Azure Troubleshooting Scenarios

| Scenario | Approach |
|----------|----------|
| App suddenly unavailable | Establish scope/impact → recent changes, health, Resource Health → for AKS: ingress/App Gateway, services, endpoints, pods, events, logs → dependencies (Key Vault, DB, DNS, networking) → rollback if deployment-related → RCA |
| AKS can't access Key Vault | `Pod → Workload/Managed Identity → Entra ID → Azure RBAC → Key Vault`. Check: correct identity used? federated correctly? has Key Vault permissions? network accessible? DNS resolving? secret name correct? audit logs? recent changes? |
| AKS can't pull image from ACR | `AKS → Identity → RBAC → ACR → Image`. `kubectl describe pod` → look for `ImagePullBackOff`/`ErrImagePull` → check image name/tag, ACR availability, identity, `AcrPull` permission, network, private endpoint/DNS |
| Terraform pipeline can't authenticate to Azure | `Pipeline → Authentication → Entra ID → Service Principal/Workload Identity → Azure RBAC → Subscription`. Verify OIDC permission, federated credential, Client/Tenant/Subscription ID, repo/branch/environment conditions, RBAC permissions. `az account show` for first validation |
| App slow but infra looks healthy | Don't just scale. Check: `App metrics → Latency → Error rate → CPU/Memory → Database → External APIs → Network → Recent deployments`. Causes: DB query perf, connection pool exhaustion, external API latency, CPU throttling, memory pressure, GC, network latency, code changes |

### 6.12 Resource Hierarchy & Disaster Recovery Concepts

**4-Tier Hierarchy:** `Management Groups → Subscriptions → Resource Groups → Resources`

**DR concepts:** multiple AZs, backups, DB replication, geo-redundancy, IaC, automated deployment, recovery procedures, RTO, RPO, regular DR testing.
- **RPO:** how much data loss is acceptable
- **RTO:** how quickly the system must be restored
- Example: RPO = 15 min, RTO = 1 hour

**Region vs. Availability Zone:**
```
Azure Region
 ├── Zone 1
 ├── Zone 2
 └── Zone 3
```
Region = geographic location; AZ = physically separate datacenter within a region.

### 6.13 Azure Security Architecture (End-to-End)

```
Identity → Entra ID → RBAC → Network → VNet/NSG/Firewall → Application → AKS security → Secrets → Key Vault → Monitoring → Azure Monitor/Sentinel
```

**Securing an AKS cluster:** Entra ID + K8s RBAC + Azure RBAC, private networking where appropriate, managed/workload identities instead of static credentials, Key Vault, network policies, image scanning, least privilege, patching, monitoring/audit logging.

**Healthcare environment security:** defense in depth — Entra ID + RBAC, managed identities, Key Vault, private networking + NSGs, encryption at rest/in transit, centralized logging/monitoring, vulnerability scanning, Azure Policy, strict least privilege, compliance/change-management adherence.

---

## 7. Linux, Shell Scripting & SQL

### 7.1 Fundamentals

**Load average:** 1-, 5-, 15-minute load metrics (`uptime`/`top`) relative to available CPU cores.

**Log rotation:** `logrotate` prevents disk saturation/node pressure.

**Long-running processes:**
```bash
ps -eo pid,etime,cmd
top
htop
```

**Directory verification & file count script:**
```bash
#!/bin/bash
path="$1"

if [ -d "$path" ]; then
  count=$(find "$path" -maxdepth 1 -type f | wc -l)
  echo "Directory exists at: $path — File count: $count"
else
  echo "Not a valid directory: $path"
  exit 1
fi
```

**Finding log files older than 7 days:**
```bash
find /var/log -type f -name "*.log" -mtime +7
```

**Count files containing a specific word:**
```bash
#!/bin/bash
word="username"
dir="/path/to/search"
count=$(grep -rl "$word" "$dir" | wc -l)
echo "Number of files containing the word: $count"
```

| Part | Purpose |
|------|---------|
| `#!/bin/bash` | Shebang — which interpreter runs this script |
| `word=`, `dir=` | Reusable variables |
| `grep -rl "$word" "$dir"` | `-r` recursive, `-l` list matching filenames only |
| `wc -l` | Counts matched files |

### 7.2 SQL: Find the 7th Highest Marks

```sql
SELECT marks FROM (
  SELECT marks FROM students
  ORDER BY marks DESC
  LIMIT 7
) AS top7
ORDER BY marks ASC
LIMIT 1;
```
> **Simple logic:** grab the top 7 → flip their order → the first of the flipped list is the 7th highest.

### 7.3 Interview Strategy Tips

- Think out loud — never code in silence; narrate your logic as you go
- Demonstrate core logic — even imperfect syntax under pressure is fine if reasoning (why `grep`, why `-r`, why `wc -l`) is sound

### 7.4 Linux Commands Cheat Sheet

**File inspection & logs**

| Command | Purpose |
|---------|---------|
| `head -n <N> <file>` / `tail -n <N> <file>` | Read first/last N lines |
| `tail -f <file>` | Live-stream newly appended lines |

**Text processing**

| Tool | Purpose |
|------|---------|
| `sed` | Inline substitutions/deletions |
| `awk` | Record/field-based text manipulation |
| `grep -i` | Case-insensitive search |
| `grep -r` | Recursive search |
| `grep -c` | Count of matches |

**Users, system status & networking**

| Command | Purpose |
|---------|---------|
| `whoami` | Current logged-in user |
| `w` / `users` | Active users & processes |
| `uptime` | Server uptime, load averages |
| `last` | Recent reboot/login history |
| `ifconfig` / `hostname -I` | Interface IPs |

**Service & process management**

| Command | Purpose |
|---------|---------|
| `systemctl start/restart/status <service>` | Manage services |
| `ps -ef \| grep <process_name>` | Find a running process |
| `top` | Interactive real-time monitoring |
| `free -m` / `free -g` | RAM & swap usage |

**Archiving, compression & transfer**

| Command | Purpose |
|---------|---------|
| `zip -r <archive.zip> <folder>` | Recursively compress |
| `unzip <archive.zip>` | Extract a zip |
| `tar -cvf <name.tar> <folder>` | Create a tar archive |
| `tar -xvf <name.tar>` | Extract a tar archive |
| `scp /local/path user@remote:/dest/path` | Secure copy over SSH |

**Other:**
```bash
nohup command &            # Run a command in the background
```

### 7.5 Real-World Linux Automation Use Cases

| Use Case | What It Does |
|----------|----------------|
| **Cost optimization scripts** | Auto-shutdown non-prod instances outside business hours; restart before workday |
| **Mass maintenance / coordinated restarts** | Rolling restarts across many servers, verifying health checks before moving to the next node |
| **Log rotation / disk space remediation** | Cron/event scripts detect `/var/log` usage > 85%, archive to S3, purge stale caches |
| **CrashLoopBackOff / OOM remediation** | Scripts triage stuck pods, extract exit codes (OOMKilled/Exit 137), dump logs, notify on-call |

---

## 8. CI/CD — Jenkins & GitHub Actions

### 8.1 Pipeline Fundamentals

**End-to-end pipeline stages:** code checkout → build/packaging (Maven/pip/npm) → security scans → container image generation → deployment.

**Full production-grade pipeline (real-world standard):**
1. **Workspace Clean** — `cleanWs()`
2. **Checkout SCM** — clone repo with branch + credentials
3. **Build & Package** — `mvn clean package` (Java) or `npm` (Node.js)
4. **Security & Quality Gates** — SonarQube (SAST) + OWASP Dependency-Check
5. **Containerization & Registry Push** — Docker image → ECR/Docker Hub
6. **Deployment & Verification** — Dev/QA/Staging/Prod on EKS/EC2
7. **Post Actions** — Slack/Teams/Email alerts, archive test reports

**Rollback strategy:** automated pipeline rollbacks, GitOps reconciliations, image tag updates, or rollback scripts.

**Pipeline challenges:** inter-tool auth/credential integration, runner capacity/timeouts, scan-stage bottlenecks, pipeline security hardening.

**Pipelines for microservices:** independent decoupled pipelines per service; immutable container artifacts tagged with Git commit SHAs; progressive rollouts (Canary/Blue-Green).

**Pipeline hardening (DevSecOps):** avoid embedded credentials — dynamic secret injection (Vault/Secrets Manager); artifact signing; vulnerability scanning (SAST/DAST/Trivy); audit logging.

**Artifact management:** versioned binaries/image layers in centralized registries — JFrog Artifactory, Sonatype Nexus, Amazon ECR.

### 8.2 Example Parameterized Jenkins Pipeline

```groovy
pipeline {
    agent none

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        booleanParam(name: 'CLEAN', defaultValue: true, description: 'Clean before building')
        choice(name: 'ENVIRONMENT', choices: ['development', 'staging', 'production'], description: 'Choose deployment environment')
    }

    stages {
        stage('Checkout') {
            agent { label 'maven-builder' }
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/example/repo.git'
            }
        }
        stage('Clean') {
            when { expression { return params.CLEAN } }
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }
        stage('Build') {
            steps { echo "Building project for environment: ${params.ENVIRONMENT}" }
        }
        stage('Deploy') {
            steps { echo "Deploying to ${params.ENVIRONMENT} environment..." }
        }
    }

    post {
        always { echo 'Cleaning up after build...' }
    }
}
```

**Parameters/stages breakdown:**

| Parameter | Type | Purpose |
|-----------|------|---------|
| `BRANCH` | string | Which branch to build |
| `CLEAN` | boolean | Whether to wipe workspace |
| `ENVIRONMENT` | choice | Target: dev/staging/prod |

| Stage | Purpose |
|-------|---------|
| Checkout | Pulls code from the chosen branch |
| Clean | Deletes old workspace files if `CLEAN=true` |
| Build | Compiles/packages the app |
| Deploy | Ships to the chosen environment |

### 8.3 Node Allocation (Master-Agent Architecture)

> **Q: "How do you ensure specific tasks run on designated nodes?"**
- `agent none` at top level → prevents reserving an executor on the master/controller
- Each stage binds to a specific **Agent Label**:
```groovy
pipeline {
    agent none
    stages {
        stage('Build & Test') {
            agent { label 'maven-builder' }
            steps { sh 'mvn clean package' }
        }
        stage('Security Scan') {
            agent { label 'security-agent' }
            steps { sh 'sonar-scanner' }
        }
    }
}
```

### 8.4 Pipeline Resilience & Versioning

| Feature | Purpose |
|---------|---------|
| **Timeout** | `options { timeout(time: 30, unit: 'MINUTES') }` — kills stuck jobs |
| **Triggers** | `githubPush()` webhook or `pollSCM` scheduled polling |
| **Parallel Execution** | Run unit/integration/lint tests simultaneously |
| **Masked Credentials** | `withCredentials([usernamePassword(...)])` keeps secrets out of logs |

**Versioning strategies:**

| Method | Example |
|--------|---------|
| Git Tags | `v1.4.0` |
| Commit Hash | `${GIT_COMMIT[0..7]}` |
| Build Number | `v1.0-${BUILD_NUMBER}` |
| Version File | `version.txt` read during build |

**Scheduled backups (Jenkins):**
1. Install backup plugin (ThinBackup)
2. Create backup directory: `/var/lib/jenkins/jenkinsbackup`
3. `chown -R jenkins:jenkins /backup-dir`
4. Schedule cron: weekly full backup + nightly differential

### 8.5 Multi-Environment Pipelines

**Conditional stage execution:**
```groovy
stage('Deploy to Dev') {
    when { branch 'develop' }
    steps { ... }
}
stage('Deploy to Production') {
    when { branch 'main' }
    steps { ... }
}
```
```groovy
stage('Deploy to Production') {
    when {
        expression { params.ENVIRONMENT == 'Production' }
    }
    steps {
        input message: 'Approve production deployment?'
        sh 'helm upgrade --install prod-release ./charts/app'
    }
}
```

**Continuous Delivery vs. Continuous Deployment**

| Term | Meaning |
|------|---------|
| **Continuous Deployment** | Every passing build auto-deploys to production — no manual approval |
| **Continuous Delivery** (enterprise standard) | Auto-deploys to staging/QA, production needs a manual **Approval Gate** |

### 8.6 Webhook Integration (GitHub → Jenkins)

| Setting | Value |
|---------|-------|
| Payload URL | `http://<jenkins-url>/github-webhook/` |
| Content-Type | `application/json` |
| Trigger | "Just the push event" or select individual events |

> ⚠️ Always include the trailing slash `/` — missing it often causes HTTP 302/404 errors.

### 8.7 Real-World Jenkins Troubleshooting

**A deployment failed in production:** stop further deployments → analyze logs (release pipeline, app monitoring, K8s/VM) → rollback (previous stable artifact or slot swap) → RCA → add pre-deployment validation, smoke tests, health probes, auto-rollback conditions.

**Build pipeline suddenly slow:** identify the slow stage via analytics → dependency caching, parallel jobs, incremental builds, reduce unnecessary tests, optimize Docker layers, self-hosted agents.

**Devs pushing directly to `main`:** branch policies — mandatory PRs, min reviewers, successful build validation, work item linking, restricted direct pushes; enforce CI validation before merge; GitFlow/trunk-based branching.

**Secret exposed in a pipeline:** revoke/rotate immediately, identify exposure scope, audit logs/access → move secrets to Key Vault/Vault, use secret variables (never hardcode), restrict pipeline permissions, enable log masking, review RBAC.

**K8s deployment works in staging but fails in prod:** compare config differences, secrets/ConfigMaps, resource limits, ingress rules, namespace policies; check pod logs, `kubectl describe`, Helm release history, probes — prefer IaC + consistent Helm charts across envs.

**Common Jenkins failures (real troubleshooting)**

| Failure | Root Cause | Fix |
|---------|------------|-----|
| Agent Offline | SSH credential expiry, network/firewall drop, JVM/Java version mismatch | Renew credentials, check connectivity, align Java versions |
| Hanging/Stuck Builds | Thread deadlocks, exhausted executors, unhandled interactive prompts (e.g., missing `-auto-approve`), no timeout | Add `options { timeout(...) }`, use `cleanWs()` |
| Disk Full | Stale Maven `~/.m2` deps, old logs, untagged Docker images | Schedule `docker system prune -af`, enable workspace discards |
| Broken Plugins/Dependencies | Jenkins core upgraded without checking plugin compatibility, Docker Hub rate limits | Check compatibility matrix before upgrading; use authenticated pulls/mirror |

### 8.8 Jenkins Scenario-Based Q&A (Deep)

**Multi-environment deployments (Dev→Staging→Prod):** parameterized pipelines (choice/string/active choices), `when { expression { ... } }` blocks, folder/scope-level configs to avoid cross-contamination.

**Handling timeout & partial failure:** state tracking/idempotency (verify applied state), `retry(count) { ... }` with `timeout(time: 10, unit: 'MINUTES')`, restart from the specific failed stage.

**Production deployment gates:**
```groovy
input message: 'Approve Prod Deployment?', submitter: 'qa-leads'
```
Enterprise auditing: integrate with ServiceNow/Jira — pipeline queries change request state, proceeds only when **Approved**.

**Secret & credential handling:** Jenkins Credentials Provider + `withCredentials` binding (`usernamePassword`, `amazonWebServicesCredentials`, `string`) — masks secrets in console logs.

**Automated rollback on failure:**
```groovy
post {
    failure {
        // fetch previously archived known-good artifact
        // or trigger a rollback workflow to redeploy the prior stable tag
    }
}
```

**Unique Docker image tagging:** Git commit SHA or semver + build number (`v1.2.0-${BUILD_NUMBER}`); avoid mutable `latest` tags.

**Optimizing long test suites:** `parallel` block to distribute independent test suites across agents/executors.

### 8.9 Rollback Strategies (GitOps + Jenkins Deep-Dive)

**The interview problem:** vague answer "we just deploy the previous image" isn't enough. Interviewers want to know **how rollback is systematically automated** in CI/CD + GitOps.

**Base GitOps deployment workflow:**
```
Docker Image → Docker Hub → Archive current manifest as artifact →
Update K8s Manifest (GitHub Repo) → Flux CD → Deployment
```

**Strategy 1 — Parameterized Rollback Pipeline via Registry API**
- Keep rollback as a **separate, dedicated pipeline**
- Accept `service_name` as parameterized input
- Query the previous stable tag via container registry API (Docker Hub API)
- Use `jq` on the Jenkins agent to parse JSON, extract the tag before the latest build
- Commit the older image tag into the K8s manifest repo → Flux CD reconciles and rolls back
> ⚠️ `jq` must be installed on the Jenkins agent.

**Strategy 2 — Jenkins Archive Artifacts Snapshot**
1. In the main pipeline: before patching the K8s manifest, back up the existing manifest and save via `archiveArtifacts`
2. On rollback trigger: pull the archived manifest from the last successful build, commit that known-good manifest back to Git — GitOps engine restores the previous state automatically

**🎯 Interview one-liner:**
> "Our main pipeline builds and pushes the image, then updates the K8s manifest repo, which Flux CD syncs to the cluster. For rollbacks, I maintain a separate parameterized pipeline that either queries the registry API for the previous stable tag using `jq`, or restores a manifest snapshot archived as a Jenkins build artifact — both approaches commit the known-good state back to Git so GitOps handles the actual rollback."

### 8.10 GitHub Actions — Fundamentals

| Concept | Description |
|---------|--------------|
| **Workflows** | `.github/workflows/*.yml`; triggered by GitHub events (push, PR, issue) |
| **Jobs** | One or more per workflow; each runs on a fresh VM |
| **Steps** | Ordered actions within a job — a shell command or reusable action |
| **Actions** | Reusable plugins/scripts (checkout, setup env, deploy, etc.) |

**Simple Node.js CI workflow:**
```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
```

**Common use cases:** running tests on PRs, linting/formatting checks, deploying to AWS/Vercel/Netlify/Firebase, releasing packages (npm/PyPI), automated code review/issue triaging.

### 8.11 GitHub Actions — Deep Interview Q&A

**Rollback strategies:** revert the commit/tag, or re-deploy the prior immutable image/release artifact.

**External integrations:** Slack notifications, Jira ticket updates, AWS IAM OIDC auth for secretless cloud access.

**GitHub-Hosted vs. Self-Hosted Runners**

| Type | Use When |
|------|-----------|
| GitHub-Hosted | Standard, ephemeral, no special requirements |
| Self-Hosted | Private VPC access, proprietary build tools, specialized hardware/caching |

**Multi-environment deployments:** GitHub Environments (Dev/Staging/Prod) with protection rules, environment-specific secrets, manual reviewer approvals.

**Custom Actions:** Composite Actions, Docker container actions, JavaScript actions — publish for internal/public reuse.

**Workflow debugging:** review live step logs; enable `ACTIONS_STEP_DEBUG` for verbose runner logs; re-run failed jobs.

**Security best practices:**
- Secrets only in GitHub Actions Secrets — never in code
- Limit `GITHUB_TOKEN` permissions via explicit `permissions:` blocks (least privilege)
- Pin action versions to **specific commit SHAs**, not mutable tags

### 8.12 GitHub Actions Contexts — Deep Dive

Built-in objects containing metadata about a run, runner, secrets, triggering event:
```yaml
${{ <context>.<property> }}
# e.g. ${{ github.ref }}, ${{ secrets.DOCKER_TOKEN }}
```

| Context | Purpose | Example Use |
|---------|---------|--------------|
| `github` | Run & git metadata (`github.actor`, `github.event`, `github.repository`, `github.ref`, `github.run_number`) | Branch checks, event inspection, audit trail |
| `secrets` | Secure vault values (`secrets.MY_TOKEN`) | Masked API keys, registry tokens, SSH keys |
| `env` | Env vars at workflow/job/step level | Custom config to steps |
| `runner` | Runner metadata (`runner.os`, `runner.arch`, `runner.temp`) | OS-conditional logic |
| `job` / `steps` | Status & outputs of current jobs/steps | Inter-job dependencies |

**Conditional execution:**
```yaml
if: github.ref == 'refs/heads/main'
```

**Accessing commit messages:**
```yaml
${{ github.event.head_commit.message }}
```

**⚠️ Security — script injection precaution:**
```yaml
# ❌ Risky — direct interpolation
run: echo "${{ github.event.issue.title }}"

# ✅ Safe — assign to an env var first
env:
  TITLE: ${{ github.event.issue.title }}
run: echo "$TITLE"
```

**Debugging contexts:**
```yaml
- name: Dump GitHub Context
  run: echo '${{ toJSON(github) }}'
```

**Dynamic artifact naming:** append `${{ github.run_number }}` or `${{ github.sha }}` for unique names.

**Real-world scenario answers**

| Scenario | Solution |
|----------|----------|
| Deploy only when PR has a specific label | `if: github.event.pull_request && contains(github.event.pull_request.labels.*.name, 'deploy')` |
| Set target URL dynamically by branch | Evaluate `${{ github.ref }}` — route `refs/heads/main` to prod, feature branches to preview |
| Restrict execution to specific users | `if: contains(fromJSON('["lead-admin", "authorized-dev"]'), github.actor)` |

### 8.13 Artifact Promotion Strategy

**The interview dilemma:** is the exact same tested artifact promoted to production, or does a new pipeline rebuild from `main`?

| | Rebuild Per Pipeline | Build Once, Promote Everywhere |
|---|---|---|
| Risk | Possible drift between builds | No drift — same binary everywhere |
| Speed | Slower (redundant builds) | Faster |
| Industry standard? | Sometimes used | ✅ Preferred / 12-Factor standard |

**Approach A — Rebuild Per Target Pipeline:** artifact built/tested in Dev/QA; on merge to `main`, a separate production pipeline rebuilds from that branch (same version/SHA); deployed to a staging/QA-mirror first; manual approval gate before prod.

**Approach B — "Build Once, Promote Everywhere" (12-Factor) ⭐:** immutable artifact built **once** from the commit SHA; same image digest flows through Dev→QA→Prod, never rebuilt; environment differences handled purely via externalized config.

**QA/pre-prod parity:** staging must mirror production (networking, DB engines, ingress rules) — otherwise config drift hides environment-specific failures.

**🎯 Interview one-liner:**
> "We build the Docker image once in CI, tag it with the immutable Git commit SHA, push to ECR. That exact image digest is deployed to QA, passes integration tests, and is promoted to Production behind a manual approval gate — only ConfigMaps and Secrets change between environments."

### 8.14 QA Pipeline Ownership

> **Who builds the QA pipeline?** The DevOps engineer designs/provisions/maintains it. The QA team contributes automated test scripts (Selenium/Cypress/Playwright) that run *inside* a pipeline stage.

**Standard QA flow:** Git checkout → build/compile → static analysis/security scan (SonarQube) → automated integration/e2e tests → publish reports & notify.

**Do you rebuild per environment?** No — build once, tag immutably, deploy the same digest everywhere; only env vars/secrets change.

### 8.15 Event-Driven Architecture in DevOps

**Definition:** actions/workflows triggered automatically by *events* rather than manual steps or fixed schedules.

| Trigger Event | Automated Response |
|----------------|----------------------|
| Code pushed to Git | CI/CD pipeline starts automatically |
| High traffic load | ASG / Kubernetes HPA launches new instances/pods |
| Failed deployment | Automatic rollback to previous stable version |
| System outage / high error rate | Alerts fire + automated remediation (service restart) |

**How to defend this on your resume:**

| Mechanism | Example |
|-----------|---------|
| CI/CD Triggers | Git webhook → Jenkins/GitHub Actions run instantly |
| Dynamic Autoscaling | CloudWatch/Prometheus threshold → ASG or K8s HPA/Karpenter |
| Self-Healing & Rollback | Health-check failure → automated rollback/restart/traffic shift |

---

## 9. GitOps — Helm, ArgoCD & Flux

### 9.1 Helm

**What is Helm?** The **package manager for Kubernetes** — the role `apt` plays for Ubuntu or `yum`/`dnf` plays for RHEL. Helm 2 required an in-cluster **Tiller** component (a security risk); **Helm 3 removed Tiller entirely**, using the user's kubeconfig and RBAC directly.

**Chart structure:**
```
mychart/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml          # Default config values injected into templates
├── templates/            # Deployments, Services, Ingress (Go template engine)
│   └── _helpers.tpl      # Reusable template snippets/naming conventions
└── charts/               # Sub-chart dependencies
```

**Full chart setup:**
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm create my-app
```

**`Chart.yaml`:**
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes to deploy my app
version: 1.0.0
```

**`values.yaml`:**
```yaml
replicaCount: 2

image:
  repository: your_ecr_repo/image_name
  pullPolicy: Always
  tag: latest

service:
  type: ClusterIP
  port: 80

resources: {}
```

**`templates/deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: my-app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

**Installing a chart:**
```bash
helm install <release-name> <chart-name-or-path>

# From an official repo:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx
```

**`helm install` vs. `helm upgrade`**

| Command | Behavior |
|---------|----------|
| `helm install` | Deploys a new release — **fails** if the release name already exists |
| `helm upgrade` | Updates an existing release, increments revision |

> **Best practice:** `helm upgrade --install <release> <chart>` — installs if new, upgrades if it exists.

```bash
helm upgrade --install my-app ./my-app \
  --namespace production \
  --set image.tag=${GIT_COMMIT[0..7]} \
  --wait
```

**Viewing releases:**
```bash
helm list
helm list -A
helm history <release-name>
```

**Overriding default values:**
```bash
# Custom YAML (recommended for GitOps/multi-env)
helm upgrade --install my-app ./my-chart -f values-prod.yaml
# CLI flags
helm upgrade --install my-app ./my-chart --set replicaCount=3 --set image.tag="v2.1.0"
```

**What is a Helm Release?** A specific **running instance** of a chart — the same chart can be deployed multiple times under different release names (`app-dev`, `app-staging`).

**Uninstalling:**
```bash
helm uninstall <release-name> -n <namespace>
```

**Verify deployment:**
```bash
kubectl get pods -n production
```

### 9.2 ArgoCD — GitOps Interview Prep

**What is ArgoCD/GitOps?**
> **GitOps core principle:** Git is the **single source of truth** for both app code and Kubernetes infrastructure state.

**Reconciliation loop:** ArgoCD continuously compares the **Desired State** (Git: manifests/Helm/Kustomize) with the **Live State** (actual cluster). A mismatch = `OutOfSync`.

| Sync Mode | Behavior |
|-----------|----------|
| **Automated Sync** | Auto-applies changes to restore parity |
| **Manual Sync** | Flags drift in UI/CLI, waits for operator approval |

**CI/CD split of labor:** CI tools (Jenkins/GitHub Actions) build/test/scan and update the image tag in Git; **ArgoCD handles CD** — pulling changes into the cluster without exposing cluster credentials to the CI server.

**Architecture:**
```
                  ┌──────────────────────┐
                  │    Git Repository     │
                  │ (Helm/Kustomize/YAML) │
                  └──────────▲───────────┘
                             │
                  ┌──────────┴───────────┐
                  │   Repository Server   │ ── Clones repo, renders manifests
                  └──────────▲───────────┘
                             │
┌──────────────┐  ┌──────────┴───────────┐  ┌──────────────┐
│  Web UI/CLI  │◄─┤      API Server       │◄─┤  Redis Cache │
└──────────────┘  └──────────┬───────────┘  └──────────────┘
                             │
                  ┌──────────▼───────────┐
                  │ Application Controller│ ── Compares Live vs Desired, syncs
                  └──────────┬───────────┘
                             ▼
                  [ Target K8s Cluster ]
```

| Component | Role |
|-----------|------|
| **API Server** | Handles UI/CLI/CI requests, auth, RBAC |
| **Repository Server** | Clones Git repos, renders manifests (YAML/Helm/Kustomize) |
| **Application Controller** | Continuously reconciles live vs. desired state, performs sync |
| **Redis Cache** | Speeds up state comparisons & session management |
| **Web UI / CLI** | Visualize topologies, sync status, logs, manual approvals |

**Rollback & versioning**

| Method | How |
|--------|-----|
| **Git-native rollback (preferred)** | `git revert <commit-hash>` → ArgoCD auto-detects and rolls back |
| **UI/CLI rollback** | `argocd app rollback <app-name>` |
| **Self-healing/auto-pruning** | Reverts manual `kubectl edit` drift; prunes resources removed from Git |

**Helm & Kustomize support:**
- **Helm:** tracks a Git folder with `Chart.yaml`/`values.yaml` or a Helm repo, renders with value overrides
- **Kustomize:** native environment overlays (`overlays/dev`, `overlays/prod`) — no separate chart packaging needed

**ApplicationSets & Multi-Cluster:**
- **ApplicationSets:** dynamically generate many ArgoCD `Application` resources from one template — deploy the same app across many clusters/environments without repetitive YAML
- **Multi-cluster setup:** `argocd cluster add <kubecontext>` to register a remote cluster; specify target cluster in `spec.destination`

**Security best practices:** RBAC, SAML/OIDC authentication, secure Git access, application secrets management, audit logs.

**ArgoCD vs. Flux CD**

| Dimension | ArgoCD | Flux CD |
|-----------|--------|---------|
| Architecture | Centralized server, rich Web UI, SSO, granular RBAC | Decentralized, lightweight Kubernetes controllers |
| UI | Rich interactive Web UI with live resource trees | CLI-centric; relies on 3rd-party UIs (e.g., Weave GitOps) |
| Best Fit | Complex enterprise, multi-tenant teams | Minimalist, headless environments |
| Helm Support | Advanced, built-in | Requires more setup (Helm Controller) |

### 9.3 DevOps vs. GitOps

| Aspect | DevOps | GitOps |
|--------|--------|--------|
| **Scope** | Broad org culture & practices — full lifecycle | Focused specifically on deployment automation & CD |
| **Center of Gravity** | Tool-agnostic | Git-centric — single source of truth |
| **Configuration Model** | Declarative OR imperative | Strictly declarative |
| **Reconciliation** | Often push-based (CI server pushes) | Pull-based (in-cluster operator reconciles drift) |

**How GitOps actually works:**
1. Operator/agent (ArgoCD or Flux) runs **inside** the K8s cluster
2. Continuously compares Git against live cluster state
3. Changes merged into Git → agent auto-syncs the cluster
4. Manual `kubectl` drift → operator detects and remediates automatically
5. A single GitOps engine can manage **multiple clusters** simultaneously

> **Simple way to remember it:** DevOps = the whole philosophy/culture. GitOps = a specific *implementation* of the "CD" part of DevOps, using Git as the control mechanism.

---

## 10. Monitoring, Logging & SRE

### 10.1 Monitoring Tools Landscape

| Category | Tools |
|----------|-------|
| Metrics + Dashboards | Prometheus + Grafana |
| Cloud-native | AWS CloudWatch, Azure Monitor, Google Stackdriver |
| Logging | ELK / EFK (Elasticsearch, Fluentd/Fluent Bit, Kibana) |
| SaaS platforms | Datadog, New Relic, Dynatrace, Nagios |

**Metrics vs. Logs — know this cold**

| | **Metrics** | **Logs** |
|---|---|---|
| Type | Time-series numeric data | Discrete, timestamped text events |
| Examples | CPU %, memory, pod restarts, HTTP request rate | API request hit, DB query, stack trace error |
| Best for | Trends, thresholds, dashboards | Root-cause / forensic debugging |

### 10.2 Prometheus + Grafana

**Install (Kubernetes via Helm):**
```bash
kubectl create ns monitoring
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```
> `kube-prometheus-stack` bundles the Prometheus Operator, node-exporter, kube-state-metrics.

**Connect Grafana → Prometheus:** Data Sources → Prometheus → `http://prometheus-server.monitoring.svc.cluster.local:9090` → build dashboards for CPU, memory, pod status, network throughput.

### 10.3 EFK Stack (Elasticsearch, Fluentd, Kibana)

```bash
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
```
- **Fluentd/Fluent Bit** runs as a **DaemonSet** on every node — tails `stdout`/`stderr` from `/var/log/containers/*.log`, enriches with pod/namespace metadata, ships to Elasticsearch.

```
<match>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
  flush_interval 5s
</match>
```
- **Elasticsearch** indexes/stores logs.
- **Kibana** is the UI to search/filter/visualize (Lucene/KQL, trace errors, inspect stack traces).

### 10.4 Alerts & Notifications

**Prometheus alert rule example:**
```yaml
groups:
  - name: example-alerts
    rules:
      - alert: HighCPUUsage
        expr: sum(rate(container_cpu_usage_seconds_total{image!="", container!="POD"}[1m])) by (container) > 0.8
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container CPU usage is too high"
```
- **Alertmanager:** receives firing alerts, deduplicates/groups them, routes to Slack, PagerDuty, or Email
- **Grafana Alerts:** can also be configured directly on dashboard panels

**Architecture diagram:**
```
[ Target Pods / Nodes ]
        │ (exposes /metrics)
        ▼
[ Prometheus Server ] ──(scrapes)──► Stores in TSDB
        │
        ├──► [ Grafana ]        ──► Dashboards
        └──► [ Alertmanager ]   ──► Slack / PagerDuty / Email
```

**🎯 Interview one-liner:**
> "We run `kube-prometheus-stack` in a dedicated `monitoring` namespace via Helm. Prometheus scrapes pod metrics, Grafana visualizes them, and Alertmanager routes high-severity alerts (e.g., CrashLoopBackOff) to Slack. For logs, Fluent Bit ships container logs to Elasticsearch/Kibana (or CloudWatch Container Insights), letting us debug without SSH access to nodes."

### 10.5 SRE Fundamentals

**SRE vs. DevOps:** SRE implements DevOps through concrete engineering metrics: **SLIs** (what you measure), **SLOs** (the target), **Error Budgets** (allowed unreliability before you must slow down features and focus on reliability).

**Handling a midnight production outage**
1. **Priority 1 — Fast mitigation over deep analysis:** capture diagnostics, then mitigate fast (rollback latest release, failover to backup region/cluster, restart degraded pods)
2. **Priority 2 — Incident communication:** war-room/channel, notify on-call leads/stakeholders, regular status updates
3. **Priority 3 — Post-recovery RCA:** document exact timeline, correlate telemetry spikes, preserve logs

**Reducing MTTR**
- **Granular observability:** high-resolution metrics (Prometheus/Grafana) at short scrape intervals
- **Automated runbooks/playbooks:** standardized responses, self-healing scripts
- **Smaller, incremental deployments:** small batch releases + canary rollouts

**Troubleshooting high latency**
- **Layer-by-layer diagnostics:** Network/DNS → Ingress/LB → Application logic → Downstream DBs/caches → External APIs
- **Distributed tracing:** Jaeger, Zipkin, OpenTelemetry, AWS X-Ray to pinpoint slow spans

**Debugging intermittent microservice failures**
- Inspect APM/logs for error spikes, retry storms, socket exhaustion, timeouts
- **Resilience patterns:** Circuit Breakers (Resilience4j, Envoy/Istio), Exponential Backoff with Jitter
- **Reproduction in staging:** elevate debug logs, simulate peak traffic

**Designing for HA**
- Multi-AZ/Multi-Region, active-active or active-passive failover
- Stateless service design — decouple compute from persistent storage
- Strict timeouts/deadlines to prevent thread pool exhaustion
- Data layer protection: async read replicas, multi-region replication, automated snapshots

**Observability, alert fatigue & RCA**
- **Three Pillars:** Metrics (trends), Logs (discrete events), Traces (request flow)
- **Reduce alert fatigue:** deprecate noisy alerts; alert on user-facing SLO symptoms, not raw CPU spikes that self-resolve
- **Blameless post-mortem:** link metric timestamps to trigger events, identify systemic factors, track action items

---

## 11. Code Quality & DevSecOps (SonarQube, Scans)

### 11.1 Code Smells vs. Bugs vs. Vulnerabilities

| Category | Primary Impact | Definition | What Happens to the System |
|----------|------------------|------------|------------------------------|
| **Code Smell** | Maintainability & Readability | Sub-optimal design/poor practices hurting long-term maintainability, reusability, extensibility | Works and gives correct output, but code is messy, fragile, hard to refactor |
| **Bug** | Reliability & Correctness | Flaws, runtime exceptions, logic errors preventing correct operation | Crashes, throws exceptions, or produces wrong calculations |
| **Vulnerability** | Security & Integrity | Security flaws/exposed entry points attackers can exploit | Runs fine, but sensitive data/access is exposed to exploitation |

**Simple way to remember it:**
> 🧹 Code Smell = "it works, but it's ugly and risky to maintain." 🐞 Bug = "it's broken." 🔓 Vulnerability = "it's a door left open for attackers."

**🧹 Code Smells**

| Smell | Problem | Fix |
|-------|---------|-----|
| Hardcoded Values & Endpoints | Static values/URLs baked into source — any change needs recompile/rebuild/redeploy | Externalize into env vars, config files, or parameter stores |
| Deeply Nested Control Logic (Arrow Anti-Pattern) | Multiple nested `if/else` hurts readability/testability | Guard clauses, early returns, polymorphism |
| Bloated Functions (violates SRP) | One function doing too much | Break into small, modular helper functions |
| Duplicated Code | Same logic repeated — bug fixes must happen everywhere | Extract shared logic into reusable methods/libraries |

**🐞 Bugs**

| Bug Type | Example |
|----------|---------|
| Off-by-One / Index Out of Bounds | Array of size 4 accessed at index 4 → `ArrayIndexOutOfBoundsException` |
| Divide-by-Zero | Division without checking the denominator |
| Scope & Uninitialized Variables | Variable declared in inner block, referenced outside → compile error/`NullPointerException` |

**🔓 Vulnerabilities**

| Vulnerability | Example | Remediation |
|----------------|---------|--------------|
| Hardcoded Credentials & DB Secrets | Passing credentials in cleartext in scripts/config | Inject at runtime via Vault/Secrets Manager/Key Vault |
| SQL Injection | Building queries via string concatenation | Parameterized queries/`PreparedStatement`s, ORM with sanitization |
| Broken Authentication & Input Validation | Processing sensitive requests without verifying identity | Enforce OAuth2/JWT verification, strict RBAC, origin whitelisting |

**🎯 Interview one-liner:**
> "In our CI/CD pipeline, every Pull Request triggers a Maven build coupled with a SonarQube scan. We configure strict **Quality Gates**: any PR introducing Blocker/Critical Bugs, Security Vulnerabilities, or dropping Code Coverage below 80% automatically **fails the Quality Gate**, blocking artifact generation or promotion to staging."

### 11.2 CI/CD Scan Types (Summary)

| Scan Type | Purpose |
|-----------|---------|
| Unit Testing / Code Quality | SonarQube — static analysis, code smells, coverage thresholds |
| **SAST** (Static App Security Testing) | Scans source code for security flaws before build |
| **DAST** (Dynamic App Security Testing) | Tests running applications/endpoints for runtime/API vulnerabilities |
| **SCA** (Software Composition Analysis) | Scans dependencies/third-party libraries (e.g., Snyk) |
| Container Image Scanning | Scans layers/OS packages for CVEs before registry push (Trivy, Aqua) |

**CI/CD scan flow to remember:** SAST (source code) → SCA (dependencies) → Container scan (image CVEs) → DAST (running app).

---

## 12. RBAC & Security (Cloud + Kubernetes)

### 12.1 Conceptual Differences

| Layer | Controls |
|-------|-----------|
| **Azure RBAC** | Access to Azure cloud resources (subscriptions, resource groups, VMs, AKS **infrastructure**) |
| **AWS IAM** | Identities & permissions for AWS resources (EC2, S3, EKS) |
| **Kubernetes RBAC** | Permissions **inside** the cluster (namespaces, pods, deployments, secrets) |

> **Simple way to remember it:** Cloud RBAC secures infrastructure-level access; K8s RBAC secures workload-level access inside the cluster. You need **both** — layered security, least privilege, namespace isolation, better governance.

### 12.2 Practical Implementation

**Least privilege access:** RBAC across Azure/AWS/K8s; minimum permissions per role/namespace; avoid cluster-admin; JIT elevated access; groups instead of direct grants; regular reviews; immediate revocation on offboarding/team changes.

**User audit and logging:** centralized logging — Azure Activity Logs/Monitor/Sentinel, AWS CloudTrail/CloudWatch/GuardDuty, Kubernetes API server audit logs — all feeding a SIEM (Sentinel, Splunk, ELK).

**Access removal when a developer moves teams:** centralized IAM (Entra ID/AWS IAM Identity Center) with group-based RBAC — never assign directly to users. Remove from old group, add to new → auto-updates Azure RBAC, AWS IAM roles, K8s RoleBindings. Validate via audit logging and periodic access reviews.

**Cluster plane isolation:** separate Dev/QA/Prod into different clusters; managed services (AKS/EKS) secure the control plane; namespace isolation, RBAC, network policies, node isolation, Pod Security Standards; restrict API server access via private networking/VPN; enable audit logging.

### 12.3 Example Kubernetes Role YAML

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: payments-dev
  name: developer-role

rules:
  apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get","list","watch"]
```

---

## 13. OpenShift

**What is OpenShift?** Red Hat's enterprise container platform built **on top of Kubernetes** — adds developer tools, built-in CI/CD, enhanced security, monitoring, image registry, web console, Operator framework.

> **Simple way to remember it:** OpenShift = Kubernetes + enterprise features + automation.

**Core Concepts**

| Concept | Description |
|---------|--------------|
| **Container** | Packages code, runtime, libraries, dependencies — runs identically everywhere |
| **Pod** | Smallest deployable unit — `oc get pods` |
| **Project** | Logical namespace for organizing resources/access — `oc new-project demo-project` |
| **CRC (OpenShift Local)** | Run OpenShift locally on a laptop |

**Architecture:** Control plane nodes (API server, scheduler, etcd), Worker nodes, Ingress router.

**Essential `oc` CLI commands:**
```bash
oc login
oc get pods
oc get svc
oc describe pod
oc logs
oc exec
```
> `oc` is the OpenShift CLI — equivalent of `kubectl` with additional OpenShift-specific commands.

---

## 14. Real-World Migration & Disaster Recovery Scenarios

### 14.1 Kubernetes Storage Migration (AWS EBS gp2 → gp3)

**Scenario:** fintech payment app on EKS, 60 microservices, ~10M daily transactions. Critical PostgreSQL StatefulSet on EBS gp2 PVC faces I/O bottlenecks. Must migrate to gp3 with **max 2 minutes downtime**.

1. **Pre-migration safety:** snapshot via AWS Backup/EBS Snapshots; pre-flight checks (row counts, schema checksums, disk metrics)
2. **Data copy & delta sync:** provision new gp3 volume via new StorageClass/PVC; asynchronous replication (rsync/logical replication) while DB stays online, minimizing final delta
3. **2-minute cutover:** stop write traffic (or read-only mode) → shut down Postgres pod → run final delta sync → update PVC/StatefulSet volume binding to gp3 → restart pod
4. **Validation:** automated health queries (row counts, read/write transactions), monitor CloudWatch EBS IOPS/queue metrics
5. **Rollback:** **don't delete gp2 immediately** — if checks fail during cutover, re-point PVC/mount back to gp2 and restart the pod

> **Production tip:** AWS EBS Elastic Volumes can modify volume type gp2→gp3 on-the-fly without detaching, or use CSI volume expansion — potentially avoiding a manual cutover window.

### 14.2 AKS Cluster Upgrade (v1.27 → v1.30, End-of-Life)

**Scenario:** 200+ microservices, stateless Deployments + StatefulSets on Azure Managed Disks; integrated with App Gateway, Key Vault, Azure Monitor, ACR. Zero customer downtime required.

- **API & deprecation assessment:** review release notes for deprecated APIs; audit Helm charts/manifests with static checkers (`pluto`, `kubent`); validate CSI drivers, Azure CNI, AGIC, Key Vault CSI compatibility
- **Validation in dev/staging:** replicate the full workload, test control plane and node pool upgrades end-to-end
- **Blue-green cluster rollout:** spin up a new "green" AKS cluster on v1.30+ via Terraform/Bicep; deploy identical manifests; shift traffic progressively at App Gateway/Front Door (5%→25%→100%)
- **StatefulSets & rollback:** synchronize persistent disk snapshots/DB replicas from blue to green before cutover; switch routing back to blue immediately if 5xx errors/degradation occur

### 14.3 Entire Data Center Migration to AWS (400 VMs)

**Scenario:** 400 on-prem VMs — Java apps, Windows IIS apps, Oracle & SQL Server DBs, file servers, AD, Jenkins, monitoring tools. 6-month deadline; dedicated Direct Connect already provisioned. Mixed criticality (dev to mission-critical banking).

```
Step 1: Landing Zone Setup → Step 2: Discovery & Dependency Mapping → Step 3: Phased Wave Execution → Step 4: DNS Cutover & Fallback
(VPC, Transit GW, IAM, AD)     (App-to-DB dependency mapping)          (Dev → UAT → Prod Waves)         (Route 53 TTL → Decommission)
```

1. **Landing zone prep:** multi-account setup (AWS Organizations), VPCs, subnets, Internet/NAT Gateways, SGs, baseline IAM roles; extend on-prem network via Direct Connect, deploy domain controllers in AWS
2. **Discovery & dependency mapping:** map app-to-DB connections (IIS→SQL Server, Java→Oracle)
3. **Migration waves:** Wave 1 (Dev/Test) → Wave 2 (Staging/UAT) → Wave 3 (Mission-critical Production)
   - **Compute:** AWS MGN for block-level VM replication to EC2
   - **Databases:** AWS DMS + SCT for CDC replication from Oracle/SQL Server to RDS/Aurora
   - **File Storage:** AWS DataSync over Direct Connect to FSx or EFS
4. **Validation, cutover & rollback:** test EC2 instances from replicated volumes in an isolated subnet; lower DNS TTL, stop on-prem writes, final CDC sync, update Route 53 to AWS ALBs; **keep on-prem warm/read-only-standby** for an agreed soak period before decommissioning

### 14.4 Cross-Cloud Migration (AWS → Azure)

**Scenario:** production platform on AWS for 5 years, migrating to Azure over 12 months (enterprise licensing). Stack: EKS, RDS, S3, Route 53, CloudWatch, IAM, Jenkins, Terraform. **Zero downtime required.**

| Domain | AWS Service | Azure Equivalent | Migration Consideration |
|---|---|---|---|
| Compute/Containers | Amazon EKS | Azure Kubernetes Service (AKS) | Standardize manifests via Helm; redeploy to AKS |
| Object Storage | Amazon S3 | Azure Blob Storage | Replicate via AzCopy or Azure Data Factory |
| Databases | RDS/Aurora | Azure DB for PostgreSQL/MySQL/SQL | Dual-write or continuous replication via Azure DMS |
| DNS & Routing | Route 53 | Azure DNS / Traffic Manager | Weighted DNS routing to split traffic progressively |
| Observability | CloudWatch | Azure Monitor & Log Analytics | Migrate Fluent Bit shippers to Log Analytics |
| Secrets & Identity | Secrets Manager/IAM | Key Vault & Entra ID | IAM roles → Managed Identities + Secrets CSI drivers |
| IaC | AWS-provider Terraform | AzureRM-provider Terraform/Bicep | Re-architect modules for Azure providers |

**Zero-downtime process:** discovery & landing zone → workload deployment to AKS + DB/blob replication → phased traffic shift (5%→25%→100%) → fallback safety net (keep AWS passive until Azure confirmed stable)

### 14.5 Database Migration with Near-Zero Downtime (MySQL → Aurora)

**Scenario:** 25 TB MySQL on EC2, ~8,000 writes/sec, migrating to Aurora MySQL. No data loss allowed; **5-minute maintenance window**.

- Use **AWS DMS** for schema conversion + continuous CDC replication of ongoing writes
- During the 5-minute window: pause/queue writes → apply final CDC delta → switch connection strings to Aurora → resume traffic
- **Validation:** compare row counts/checksums, verify zero replication lag before cutover, run smoke tests
- **Rollback:** revert connection strings to the original MySQL-on-EC2 endpoint (kept warm, receiving no writes) if Aurora shows performance issues

### 14.6 CI/CD Tool Migration (Jenkins → Azure DevOps)

**Scenario:** ~700 Jenkins pipelines, shared libraries, self-hosted Linux/Windows agents, SonarQube, Artifactory, K8s deployments — migrate to Azure DevOps Pipelines with zero release interruption.

Approach: migrate pipelines, credentials, agents, artifacts, deployment strategies, approvals, secrets, rollback mechanisms incrementally; run **both CI/CD systems in parallel** before decommissioning Jenkins, validating each migrated pipeline against its Jenkins counterpart.

### 14.7 Kubernetes Disaster Recovery (Velero-based)

**Scenario:** EKS cluster unavailable due to accidental deletion of control-plane resources, affecting 150+ apps. RTO < 30 min, RPO < 5 min. Backups via Velero; DBs replicated cross-region.

**How Velero works:** runs as an in-cluster controller, periodically captures cluster state — K8s manifests (Deployments, ConfigMaps, Secrets, Services, Ingress, RBAC) as YAML in S3, plus native EBS VolumeSnapshots via the CSI driver.

**Recovery flow:**
1. **Cluster rebuilding:** fast `terraform apply` against version-controlled IaC to spin up a replacement control plane + node groups. *(If IaC is missing, a 30-min RTO is practically impossible.)*
2. **Addons & CSI setup:** reinstall VPC CNI, CoreDNS, kube-proxy, EBS CSI driver + StorageClass; install Velero pointed at the backup bucket
3. **Velero restore:**
   ```bash
   velero restore create --from-backup prod-cluster-backup-latest --wait
   ```
   Velero reconstitutes namespaces, reattaches new EBS volumes from snapshots to PVCs, reconciles secrets/ConfigMaps/deployments
4. **DB & DNS cutover:** update app config to point to the warm cross-region DB replica (meets <5 min RPO); restore Load Balancer Controller/Ingress; update Route 53 to new load balancers
5. **Smoke validation:** `kubectl get pods -A` for zero CrashLoopBackOff; synthetic checkout/login transactions; validate DB write integrity/replication lag; confirm Prometheus/Grafana/Fluent Bit emitting metrics

### 14.8 Enterprise Blue-Green Deployment (Full Scenario)

**Scenario:** financial app, ~1M transactions/day, 30-min downtime per release (updates applied in-place). Goal: zero downtime + instant rollback.

- **Initial release (v1→v2):** Blue (active) serves 100% traffic on v1; Green (idle) provisioned with v2, smoke-tested without affecting live users; traffic cutover via ALB target group switch or Route 53 update; Blue stays on standby for instant rollback
- **Next cycle (v2→v3):** Blue/Green **alternate roles** — Green (now active with v2) stays live while Blue becomes the new staging target for v3
- **Implementation patterns:**
  - *Dual in-cluster* (cost-effective): two Deployments in the same cluster, switch ALB listener weights or Service selector
  - *Dual-cluster* (high isolation, critical banking): two identical EKS clusters, switching orchestrated at Route 53

### 14.9 Multi-Region Disaster Recovery (AWS)

**Scenario:** company operates exclusively out of AWS Mumbai (`ap-south-1`). Stack: EKS, RDS, Redis, S3, CloudFront, Route 53, CI/CD. Requirement: survive a **complete regional outage**.

```
                          Route 53 DNS (Failover / Health Checks)
                                      │
            ┌─────────────────────────┴─────────────────────────┐
            ▼                                                   ▼
 Primary Region (Mumbai - ap-south-1)          Secondary DR Region (e.g., ap-southeast-1 / us-east-1)
 ├── CloudFront Edge Caching                   ├── Standby / Active CloudFront
 ├── EKS Cluster (Active Workloads)            ├── Replicated EKS Cluster (via Terraform/IaC)
 ├── Amazon RDS (Primary Read/Write) ─────────►├── Amazon RDS (Cross-Region Read Replica)
 ├── Amazon S3 (Primary Bucket) ──────────────►├── Amazon S3 (Cross-Region Replication - CRR)
 └── Regional Redis Cache (Independent)        └── Regional Redis Cache (Independent Warm Cache)
```
- **IaC replicability:** parameterized Terraform/CloudFormation modules deploy identical topologies in the secondary region
- **DNS failover:** Route 53 Failover Routing + Health Checks; CloudFront origin failover groups for edge caching
- **DB/storage sync:** RDS/Aurora async Cross-Region Read Replica (WAL streaming) — promote on failover; S3 CRR with versioning; Redis runs **independent** caches per region (avoid WAN sync latency)
- **Session continuity:** avoid sticky sessions — use stateless JWTs or distributed session state
- **CI/CD strategy:** Active-Active (deploy to both simultaneously) vs. Active-Passive (secondary pre-provisioned, spun up on incident)

### 14.10 Legacy .NET Modernization to AKS

**Scenario:** 300+ legacy .NET Framework apps on Windows VMs, coupled to SMB file shares and SQL Server, migrating to AKS with zero disruption.

- **Assessment:** separate stateless web APIs from services bound to SMB/SQL; migrate low-risk internal services first
- **Containerization:** .NET Framework needs a Windows kernel base image:
  ```dockerfile
  FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2022
  ```
  Push to ACR
- **AKS architecture:** hybrid node pools — Linux (system services) + Windows Server (containerized .NET apps); Azure Files CSI driver (SMB) for Windows file share compatibility
- **Secrets & DB connectivity:** externalize connection strings to Key Vault via Secrets Store CSI Driver; secure AKS-to-SQL via Private Endpoints/VNet integration
- **Safe cutover:** automated pipelines build/test/push/deploy via Helm; keep legacy VMs running in parallel, decommission only after full production burn-in

### 14.11 Cloud Migration Strategy — The 6 Rs

| R | Meaning | Example |
|---|---------|---------|
| **Rehost** (Lift & Shift) | Move workloads as-is | Azure VM → AWS EC2 matching vCPU/memory/OS |
| **Replatform** (Lift, Tinker & Shift) | Minor optimizations using managed services | Self-hosted DB → Aurora/RDS |
| **Refactor / Re-architect** | Re-engineer into cloud-native | Monolith → microservices on containers/serverless |
| **Repurchase** | Replace custom software with SaaS | — |
| **Retain** | Keep non-migratable/compliance-bound components on-prem | — |
| **Retire** | Decommission obsolete servers/services | — |

**Cross-cloud mapping (Azure → AWS):**

| Azure | AWS Equivalent | Notes |
|-------|------------------|-------|
| AKS | Amazon EKS | Node groups, CNI networking, pod manifests |
| Azure PostgreSQL Flexible Server | Aurora/RDS | Aurora for high performance + managed scaling |
| Mgmt Groups & Resource Groups | AWS Organizations & Member Accounts | Azure = RGs; AWS = separate accounts + IAM boundaries |
| NSGs | Security Groups/NACLs | Subnet routing, VPC peering, endpoint access |

**Practical migration workflow (Java microservices Azure→AWS):**
1. **Pre-migration backups** — full snapshots of source DBs and storage volumes
2. **DB migration via AWS DMS:** replication instance, network reachability, table mapping, full-load + CDC tasks
3. **App & cluster provisioning:** target EKS clusters/VPCs/node groups via IaC; deploy manifests/Helm, validate pod health
4. **Staging & production cutover:** dry-run in sandbox first, then schedule maintenance window, final delta sync, re-point DNS/LB, monitor telemetry

---

## 15. DevOps Culture, Maturity & SDLC

**DevOps Maturity Model:** evaluates organizational transformation across automation, cross-team collaboration, continuous delivery, and observability. Higher maturity → faster release cadence, minimal manual intervention.

**12-Factor App Methodology:** strict config/code separation, stateless processes, backing service abstraction, disposability (fast startup/graceful shutdown), dev/prod parity.

**SRE vs. DevOps:** SRE implements DevOps via SLIs, SLOs, Error Budgets (see [Section 10.5](#105-sre-fundamentals)).

**General interview guidance:** reference sheets give concise bullets — but candidates must elaborate on real-world mechanics and architecture patterns, not just recite one-liners. Interviewers want to see you narrate *why* something fails, *how* you'd verify it, and *how* you'd fix/prevent it — like a real incident, not a cheat sheet.

**Domain-specific interview rounds (Banking/Healthcare/Insurance):** enterprises run vertical-specific rounds testing regulatory compliance, strict SLAs, and data handling constraints beyond general tool proficiency.

**Banking:**
- IaC while meeting PCI-DSS/SOX; secure secrets management across environments; auditability/traceability in CI/CD; CI/CD design for microservices; preventing unauthorized prod changes; automated rollback strategies; securing Docker images handling financial transactions; horizontal vs. vertical scaling; zero-downtime K8s deployments for critical financial systems.

**Healthcare & Insurance:**
- **Data retention & fast access:** tiered storage (hot/warm/cold, S3 Standard vs. Glacier), automated lifecycle policies, efficient DB indexing for records that must be retained for years but retrieved instantly
- **Data security/privacy/pipeline governance:** PII/PHI handling; credentials never logged/exposed during build/test; schema migrations must never corrupt historical records
- **HA, zero downtime & multi-region DR:** outages impact patient care/claims — blue/green or canary deployments, automated rollback on failed health checks, multi-region DR for zero-data-loss SLAs
- Typical questions: HIPAA/GDPR/SOC2 alignment in CI/CD; DB engines/storage tiers/LB patterns balancing speed/encryption/retention; guaranteeing schema migration data integrity; multi-region deployment/failover/backup-restore drills

---

## 16. End-to-End Project Narrative (Interview Script)

Use this as a cohesive answer when asked *"walk me through your project end-to-end"*:

1. **Infrastructure:** *"First, I provisioned the underlying AWS VPC, multi-AZ subnets, security groups, and EKS cluster using modular Terraform configurations stored in remote S3 with DynamoDB state locking."*
2. **SCM & Branching:** *"Our development teams follow a GitFlow model. Code commits trigger automated CI webhooks to our Jenkins server."*
3. **CI/CD Pipeline:** *"The Jenkins Declarative Pipeline checks out the code, compiles it with Maven, enforces SonarQube quality gates and OWASP dependency checks, builds an immutable Docker container tagged with the Git commit hash, and pushes it to private Amazon ECR."*
4. **Deployment:** *"Finally, the pipeline triggers an automated Helm upgrade (`helm upgrade --install`) on our EKS cluster, dynamically injecting the new image tag into `values.yaml` and performing a zero-downtime rolling update verified by `kubectl rollout status` checks."*
5. **Monitoring:** *"For observability, we run `kube-prometheus-stack` for metrics/dashboards, Alertmanager for proactive Slack alerts, and EFK/CloudWatch for centralized log aggregation and debugging."*

### 16.1 SCM & Branching Deep Dive

**Repository basics:**
```
https://github.com/mycompany-test/project.git
```
| Part | Meaning |
|------|---------|
| `github.com` | SCM platform |
| `mycompany-test` | Organization |
| `project.git` | Repository |

Teams get specific access levels (Read, Triage, Write, Admin) based on role.

**Model A: Git Flow (large teams/scheduled releases)**

| Branch | Purpose |
|--------|---------|
| `main`/`master` | Always production-ready |
| `develop` | Integration branch — new features merge here first |
| `feature/*` | Built from `develop`, merged back via PR after review |
| `release/*` | Cut from `develop` for release prep — final testing/fixes |
| `hotfix/*` | Branched from `main` for urgent prod fixes, merged into both `main` and `develop` |

**Analogy:** `develop` = kitchen where dishes (features) are prepared, `release` = final plating/QA, `main` = what's served to customers. `hotfix` = the emergency fix when a served dish has a problem.

**Model B: GitLab Flow / Environment-Based Branching**
- Long-lived branches mapped to environments: `dev`, `staging/qa`, `production`
- Short-lived branches tied to issues/tasks: `issue-102-fix`, `feature/cart`
- These merge into environment branches through automated promotion pipelines

> **Interview tip:** pick ONE model and describe it confidently. Git Flow = structured/enterprise. GitLab Flow = flexible/issue-driven.

### 16.2 Deployment Method Comparison

| Platform | Tool |
|----------|------|
| AWS | CodeDeploy (`appspec.yml`, in-place or blue/green) |
| Azure | Azure App Services (via Azure Pipelines) |
| GitOps | ArgoCD — pull-based; continuously syncs cluster state with Git repo |

---

## 17. Career, Behavioral & Company-Specific Prep

### 17.1 Career Transition Guide — Switching Into DevOps

**Two practical routes**

| Route | Approach |
|-------|----------|
| **A. Internal Transition** (fastest legitimacy) | Ask your manager for release/partial allocation to an internal DevOps/Cloud project. Even 2–4 months of cross-skilling gives real, defensible production context |
| **B. Self-Learning / External Switch** | If internal mobility is blocked, use hands-on labs (KodeKloud), AWS/Azure free tier, self-host tools (EC2, Jenkins, Docker, Minikube/kind/EKS) to build end-to-end projects |

**The 3 core interview questions career-switchers must nail**
1. *"What are your exact roles and responsibilities?"* → anchor around tools you're actually solid in
2. *"What does a typical day look like for you?"* → standup → monitoring alerts → root-cause → automation
3. *"What tools and cloud technologies do you own?"* → be specific, don't generalize

**High-priority tools**

| Area | Tools |
|------|-------|
| Kubernetes Orchestration | EKS/AKS, Pods, Deployments, Services, Helm |
| CI/CD & Containerization | Jenkins/GitHub Actions/GitLab CI, multi-stage Dockerfiles, ECR |
| DevSecOps | SonarQube (SAST), OWASP Dependency-Check (SCA), Vault/Secrets Manager |
| IaC & Scripting | Modular Terraform, Shell/Python automation |

**Overcoming imposter syndrome:** every engineer faces a ramp-up curve. Effort in labs/troubleshooting translates directly to job performance. After ~30–60 days, the rhythm becomes natural.

### 17.2 A Day in the Life of a DevOps Engineer

**Cross-team interactions:** Core Product Dev teams, Internal Platform/Automation teams, Dev & QA Infra teams, Production Operations teams, SRE/On-Call.

**Realistic daily schedule:**
```
09:00–09:30 ── System checks, monitoring review, alerts, Jira backlog
09:30–10:00 ── Daily Standup (yesterday / today / blockers)
10:00–13:00 ── P0/P1 priority work (pipeline debugging, hotfixes, unblocking devs)
14:00–17:00 ── Core project execution (Terraform modules, CI/CD refactoring, scripts)
17:00–18:00 ── Cross-team syncs, documentation, runbook updates
```

**Role rotation (prevents single points of failure)**

| Role | Focus |
|------|-------|
| Pipeline Engineers | Jenkinsfiles/GitHub Actions, build agents, Docker cache, quality gates |
| Infra/Automation Engineers | Terraform, Helm charts, Python/Bash automation |
| Production Support | Monitoring live deployments, CrashLoopBackOff/OOM triage |
| Knowledge Transfer | Peer reviews, pairing, shadowing for coverage |

**The "Automate Recurring Issues" principle:** if an issue happens once, document it. If it happens twice, **automate the fix**.

**🎯 Interview one-liner:**
> "My day starts with reviewing monitoring alerts and our Jira board for high-priority blockers. During standup, I align with developers on sprint deliverables. The core of my day splits between project automation — writing Terraform modules, optimizing CI/CD stages, refining Helm templates — and platform maintenance, like investigating failed pipelines and rotating cluster secrets. For recurring incidents, I turn manual fixes into automated scripts and update our runbooks."

### 17.3 Behavioral & Scenario-Based Questions (By Category)

**CI/CD Tools**
1. Tell me about a time you set up a CI/CD pipeline from scratch — tools used, challenges faced?
2. A pipeline failed unexpectedly — how did you identify and resolve it?
3. Describe a time optimizing your CI/CD pipeline meaningfully improved team productivity.

**Cloud Platforms**
4. Describe migrating infrastructure to the cloud — your role and tools used?
5. Describe troubleshooting a production issue in the cloud — steps taken?
6. Experience with IaC tools — how did you implement/manage changes?

**Containers & Orchestration**
7. Describe containerizing an application — benefits and unexpected challenges?
8. Walk through debugging a Kubernetes cluster issue in production.
9. Describe configuring Kubernetes for scaling or load balancing.

**Monitoring & Security**
10. Describe proactively catching a performance issue through monitoring.
11. Describe responding to a discovered security vulnerability in CI/CD or infra.
12. Describe implementing logging & alerting from scratch — tools chosen and why?

**Collaboration & Culture**
13. Describe bridging the gap between Dev and Ops teams.
14. Describe a high-pressure production incident — how did the team handle it, what did you learn?
15. Describe a process improvement you introduced — how was it received?

### 17.4 Sample Behavioral Answers (Reusable Scenarios)

**CI/CD Setup & Failures**
- **Setting up CI/CD from scratch:** Migrating Jenkins → AWS-native tools (CodePipeline/CodeBuild). Challenges: granular IAM roles, cross-service permissions, repo integration tokens, secure webhook triggering.
- **Unexpected pipeline failure:** Root causes — missing build deps, agent/runner outages, broken webhook secrets. Resolution — check logs, executor availability, add automated retry.
- **Productivity improvement:** added Git pre-commit hooks; shifted feedback left by auto-running tests on push.

**Cloud Infrastructure & Troubleshooting**
- **Cloud migration:** Azure→AWS using AWS DMS for large-scale data transfer.
- **502 Bad Gateway outage:** backend pods crashing/OOMKilled → checked container runtime logs, resource limits, collaborated with dev on root cause.
- **IaC change management:** managed Terraform state/drift, enforced environment promotion gates (dev/staging first, peer-reviewed plans before prod).

**Containers & Kubernetes**
- **Containerization benefits:** consistent environments, simpler dependency management, predictable scaling. Challenges: deconstructing monolithic dependencies, secure config/secret injection.
- **Cluster outage escalation:** assess multi-namespace impact, escalate control-plane issues to managed cloud support, follow with RCA.

**Monitoring & Security**
- **Proactive detection:** caught CPU climbing 80–90% or disk saturation from unrotated logs before SLA breach.
- **Security remediation:** fixed overly permissive IAM roles/unauthenticated endpoints; enforced centralized identity (SSO) and DR/snapshot strategies.

**Culture & Collaboration**
- **Bridging Dev & Ops:** wrote self-service deployment scripts and standardized pipeline templates; built shared observability dashboards to cut MTTR.

### 17.5 Detailed Real-World Scenario Answers

**1. Database saturation & CrashLoopBackOff:** Pods OOM and CrashLoopBackOff due to connection exhaustion from unoptimized, massive DB tables. **Short-term:** purged records older than 3 months, added DB indexes. **Long-term:** automated periodic archive/cleanup jobs.

**2. Post-deployment data inconsistencies:** runtime errors from missing DB columns/config drift. Isolated pod logs → replicated/verified fixes in lower environments first → applied hotfix to prod.

**3. Terraform: ad-hoc scripts → modular code:** migrated non-standardized provisioning scripts to reusable Terraform modules. Fixed concurrency/drift by moving state to remote backend with locking (S3+DynamoDB), storing code in Git, running plan/apply via CI/CD with mandatory PR review. Chose Terraform over CloudFormation for unified multi-cloud syntax.

**4. VM → Kubernetes containerization:** VM setups had silent weekend outages caught only Monday. K8s gave automated self-healing, auto-restarts, predictable resource isolation, cutting recurring downtime. Challenge: steep K8s learning curve and networking debugging complexity.

**5. DR drill failure & recovery:** after a DR restore, workloads across multiple namespaces failed. Debug: `kubectl get pods -A` → `kubectl describe pod` for events/timestamps → found mismatched env vars/broken ConfigMaps from restore. Fix: rolled back to previous stable snapshot to restore SLA uptime, then RCA to fix the restoration automation scripts.

**6. Auto-scaling & load balancing:** HPA scales pod replicas on CPU/memory; Cluster Autoscaler adds/removes nodes when pending pods exceed capacity. AWS ALB (via Load Balancer Controller/Ingress) for L7 path-based routing into K8s target groups.

### 17.6 Company-Specific Real Interview Experiences

**Globant — DevOps Engineer (1–1.5 hr technical round)**

| Domain | Topics Asked |
|--------|----------------|
| Linux | File hierarchy, `ip addr`/`ifconfig`, `who`/`w`, `kill`/`kill -9`/`pkill`, `chmod`/`chown` |
| Git | `git fetch` vs `git pull`, purpose/mechanics of `git cherry-pick` |
| Jenkins | Pipeline lifecycle/stages, common plugins, trigger mechanisms (poll SCM, webhooks, cron) |
| Docker | `FROM`, `CMD` vs `ENTRYPOINT`, `COPY` vs `ADD`, Swarm vs services/nodes, Compose, networking, volumes |
| Kubernetes | Control plane vs worker nodes, master unreachable behavior, headless services, ReplicaSet vs ReplicationController, Taints & Tolerations, Ingress, Helm, `kubectl --dry-run=client -f <file.yaml>` |
| Terraform/Ansible | `terraform apply -auto-approve`, Ansible syntax + `--syntax-check`, custom modules, roles, Tower/AWX, `ansible-vault` |

> **Key takeaway:** even AWS-heavy backgrounds should be ready for Azure DevOps-flavored questions.

**BMC Software — DevOps/Automation Focus**

Heavy emphasis on **live Bash coding**, not just theory:
- Purpose of the shebang (`#!/bin/bash`)
- Taking dynamic user input (`read`)
- Writing test conditions (e.g., file exists and is writable: `-w`)
- File automation: moving/renaming files by appending dynamic date/timestamp

> **Key takeaway:** prepare to write real Bash scripts live, not just explain concepts.

### 17.7 Career Advice — Keep Interviewing With an Offer in Hand

**Reasons to keep looking:** compensation gap, lack of project clarity, location/work-life fit.

**How to handle new recruiters while holding an offer:** be upfront about the existing offer and stated compensation/project expectations — respects everyone's time.

**Managing multiple pipelines:** most companies need 2–3 rounds (technical, managerial, HR) taking weeks — run parallel interview pipelines since offers can fall through, maximizing odds of landing the right fit before notice period ends.

**Key takeaways:** know your value; demand project/team clarity before accepting; do due diligence (reach out to current/former employees to verify working conditions).

> 💡 **Mindset note:** don't let early rejections shake confidence in later rounds — trust your growth.

### 17.8 Practice Prompts (Common Live-Coding Asks)

**Shell Scripting**
1. Check if a file exists and whether it's writable
2. Print a list of names read from an input file
3. Create a file named with today's date in `yyyy-MM-dd` format
4. Mount a disk on a given filesystem
5. Word count, grep a specific word, first/last 10 lines, count lines with `sed`

**Jenkins:** write a basic declarative pipeline script
**Docker:** write a Dockerfile to containerize a Python app
**Kubernetes:** write a manifest for a Persistent Volume; write a Pod/Deployment template
**Terraform:** write a script to create an EC2 instance including VPC/subnet, explain output retrieval
**Ansible:** write a sample playbook to install `git`

### 17.9 More Practical DevOps Interview Questions (Practice List)

1. How do you check open ports in a Linux system?
2. What are the benefits of using a firewall?
3. Write a simple Terraform script to create a VM/EC2 instance.
4. Write a manifest file (`pod.yaml`) for a single container (database service).
5. Difference between Pod and Deployment — write a `deployment.yaml` for a database service with 3 replicas.
6. Difference between NodePort, ClusterIP, and LoadBalancer — where do you use each?
7. What is Helm? Explain its components.
8. Write a Dockerfile for a Node.js application.
9. What is a base image in a Dockerfile? How do you choose one?
10. What are liveness and readiness probes?
11. Difference between Deployment, ReplicaSet, and ReplicationController?
12. How do you do port forwarding in Docker and Kubernetes?

### 17.10 DevOps Learning Roadmap (Step-by-Step Sequence)

1. **Linux Fundamentals** — filesystem hierarchy, disk mount points, permissions; troubleshooting CPU/memory/uptime/process management; basic networking
2. **Shell Scripting** — bash with conditionals/loops/arrays; parsing CLI output with `grep`/`awk`/`sed`/`cut`
3. **Git & GitHub** — working dir/staging/local repo/remote branches; add/commit/push/pull/merge, `.gitignore`, PAT
4. **CI/CD Pipelines** (Jenkins/GitHub Actions/GitLab CI) — checkout, build, push artifacts/images, secrets, parameterized builds, webhook triggers
5. **Cloud Infrastructure & Networking** — VPCs/VNets, subnets, CIDR blocks, Internet/NAT Gateways, 3-tier design, DNS, ALBs
6. **Containerization & Orchestration** — Dockerfile directives, image building, registries; K8s Pods/Deployments/Services/Ingress
7. **IaC & Config Management** — Terraform for provisioning; Ansible for config management
8. **GitOps & Advanced Automation** — ArgoCD/Flux CD; Python for cloud automation; DevSecOps scanning (SonarQube, Trivy)

### 17.11 Rapid-Fire Interview Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | How do you rollback to a previous build in Jenkins? | Job → Build History → select previous build → **Rebuild** (or download/redeploy that artifact) |
| 2 | How do you connect to a Kubernetes cluster? | Install `kubectl`; ensure `~/.kube/config` is present/configured |
| 3 | What's needed in an Ingress to route traffic? | Install an Ingress Controller → create Ingress resource (host/path rules) → DNS record pointing to the controller's IP |
| 4 | How do you troubleshoot High CPU Utilization? | `top` to find the offending process; kill if stuck; check background processes |
| 5 | How many Jenkins slave nodes? | Example: 1 master, 3 slave nodes |
| 6 | How many services run in your project? | Clarify: K8s Services vs. Docker services |
| 7 | What Jenkins pipeline types have you used? | Scripted and Multibranch pipelines |
| 8 | How to run a command in the background? | `nohup command &` |
| 9 | How do you import a manually-created resource into Terraform? | `terraform import aws_instance.example i-0abcd1234efgh5678` (after defining the resource block) |
| 10 | Difference between load balancer types? | **Layer 4** = TCP/UDP; **Layer 7** = HTTP/HTTPS |
| 11 | `git fetch` vs `git pull` | `git fetch` downloads new commits/branches without touching your working dir. `git pull` = `git fetch` + `git merge` |

---

## 18. Quick Recall Cheat Sheet

- **Git Flow branches:** `main` → `develop` → `feature/*` → `release/*` → `hotfix/*`
- **Webhook URL must end with:** `/github-webhook/`
- **Continuous Delivery** = manual approval before prod | **Continuous Deployment** = fully automatic
- **`agent none`** at pipeline top level = no default executor reserved on master
- **`helm upgrade --install`** = creates release if new, upgrades if exists
- **Metrics** = numeric trends (Prometheus) | **Logs** = discrete text events (EFK)
- **Headless Service** = `clusterIP: None` → direct per-pod DNS, used with StatefulSets for databases
- **DNS pattern for pod-to-pod (cross-namespace):** `service-name.namespace.svc.cluster.local`
- **Traffic load-balancing:** CoreDNS resolves name → ClusterIP → kube-proxy round-robins to pod IPs
- **Code Smell** = messy but works | **Bug** = broken/crashes | **Vulnerability** = security hole
- **SonarQube Quality Gate** = blocks PR merge if Blocker/Critical bugs, vulnerabilities, or coverage < 80%
- **SQLi fix:** parameterized queries/PreparedStatements | **Secrets fix:** Vault/Secrets Manager, never hardcoded
- **Terraform modules:** child module outputs must be exported to be used — `module.<name>.<output>`
- **Terraform + Ansible:** Terraform builds infra → Ansible configures it → dynamic inventory bridges the two
- **Build Once, Promote Everywhere:** build image once, tag with commit SHA, promote the *same digest* through all environments — only config changes
- **ArgoCD = pull-based CD** (cluster pulls from Git); Jenkins/CI = push-based build/test/scan
- **ArgoCD rollback:** `git revert <hash>` (preferred) or `argocd app rollback <app-name>`
- **Ingress** = cost-effective L7 HTTP routing | **LoadBalancer** = 1-per-service, use for raw TCP/UDP
- **Automate recurring issues:** happens once → document it; happens twice → automate the fix
- **Cert-Manager:** Issuer (how to get certs) → Certificate (what domains) → stored in a Secret → auto-renewed
- **7th highest marks SQL:** top-7 DESC → subquery ASC → LIMIT 1
- **Ansible:** push-based & agentless (SSH/WinRM) vs. Puppet/Chef (pull-based, agent required)
- **DevOps vs GitOps:** DevOps = broad culture/full lifecycle | GitOps = Git-centric, declarative, pull-based CD specifically
- **6 Rs of migration:** Rehost, Replatform, Refactor, Repurchase, Retain, Retire
- **Rollback (GitOps):** query previous stable tag via registry API + `jq`, OR restore an archived manifest snapshot — commit back to Git, let Flux/ArgoCD reconcile
- **kubectl basics:** `apply -f` (declarative) vs `create`/`run` (imperative); `describe`, `logs`, `exec -it`, `--watch` for debugging
- **Blue-Green:** Blue = live, Green = idle/new. Cutover = switch Ingress backend or Service selector — same DNS/URL throughout
- **Helm:** `helm install` fails if release exists; `helm upgrade --install` is the safe pattern
- **`ARG` vs `ENV`:** ARG = build-time only; ENV = persists into the running container
- **Docker cache:** invalidates from the first changed layer downward — put stable deps before volatile source code
- **`git fetch`** = download only (no merge) | **`git pull`** = fetch + merge
- **Ansible control node:** cannot run natively on Windows (needs WSL); but CAN manage Windows targets via WinRM
- **CI/CD scan types:** SAST (source code) → SCA (dependencies) → Container scan (image CVEs) → DAST (running app)
- **SLI/SLO/Error Budget:** SLI = what you measure, SLO = the target, Error Budget = allowed unreliability before you must prioritize reliability over features
- **etcd crash:** running pods keep running (kubelet/runtime continue); but NO new scheduling/scaling/secrets — control plane freezes
- **NotReady node:** pods evicted only after ~5 min timeout; PDBs guarantee minimum availability during the shuffle
- **Zero-downtime K8s deploy:** `maxUnavailable: 0`, `maxSurge: 1` + accurate readiness probes = old pods die only after new ones are ready
- **Cloud RBAC vs K8s RBAC:** Cloud RBAC = infrastructure layer access; K8s RBAC = workload layer access inside the cluster — use both
- **502 vs 503 vs 504:** 502 = bad response from upstream | 503 = service overloaded/down | 504 = upstream timed out
- **Terraform state recovery:** remote backend versioning (S3/Blob) OR `terraform import` to rebuild
- **OpenShift = Kubernetes + enterprise features** (CI/CD, security, web console, Operators) — `oc` is its CLI
- **Managed Identity vs Service Principal:** Managed Identity = Azure-rotated, passwordless; Service Principal = manual credential rotation
- **AKS pulling from ACR:** needs `AcrPull` role via Managed Identity, not full Contributor
- **Prometheus + Grafana:** Prometheus scrapes/stores metrics (TSDB); Grafana visualizes; Alertmanager routes alerts to Slack/PagerDuty
- **EFK stack:** Fluentd/Fluent Bit (DaemonSet, tails logs) → Elasticsearch (stores/indexes) → Kibana (search/visualize)
- **Velero DR:** backs up K8s manifests to S3 + EBS snapshots via CSI driver; `velero restore create --from-backup <name>` to recover
