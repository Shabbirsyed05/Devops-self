# ☸️ Kubernetes From Scratch & Production Master Handbook
> **Comprehensive Enterprise Platform Reference, Architecture Blueprint, Production Runbooks & Technical Interview Masterclass.**  
> *Engineered for Site Reliability Engineers (SREs), DevOps Practitioners, Cloud Platform Architects, and Kubernetes Administrators.*

---

## 📑 Table of Contents
1. [Core Conceptual Mental Models for Jobs & Interviews](#1-core-conceptual-mental-models-for-jobs--interviews)
2. [Kubernetes Architecture: Control Plane vs. Worker Nodes](#2-kubernetes-architecture-control-plane-vs-worker-nodes)
3. [Cluster Setup, Bootstrap Tools & Kubectl Mastery](#3-cluster-setup-bootstrap-tools--kubectl-mastery)
4. [Workloads & Lifecycle Management](#4-workloads--lifecycle-management)
   - [Pods, Sidecars & Init Containers](#pods-sidecars--init-containers)
   - [Deployments, ReplicaSets & Zero-Downtime Rollouts](#deployments-replicasets--zero-downtime-rollouts)
   - [DaemonSets & StatefulSets](#daemonsets--statefulsets)
   - [Jobs & CronJobs](#jobs--cronjobs)
5. [Configuration & Secret Management](#5-configuration--secret-management)
6. [Persistent Storage Architecture (PV, PVC, StorageClass)](#6-persistent-storage-architecture-pv-pvc-storageclass)
7. [Resource Management, Probes & Scheduling Intelligence](#7-resource-management-probes--scheduling-intelligence)
8. [Networking, CoreDNS, Ingress & NetworkPolicies](#8-networking-coredns-ingress--networkpolicies)
9. [Autoscaling & High Availability Engineering](#9-autoscaling--high-availability-engineering)
10. [Observability: Monitoring & Centralized Logging](#10-observability-monitoring--centralized-logging)
11. [Cluster & Pod Security Hardening (4C Security Model)](#11-cluster--pod-security-hardening-4c-security-model)
12. [Modern CI/CD, GitOps & Helm](#12-modern-cicd-gitops--helm)
13. [AWS EKS Production Architecture](#13-aws-eks-production-architecture)
14. [Troubleshooting & Common Production Incident Matrix](#14-troubleshooting--common-production-incident-matrix)
15. [Senior Interview System Design Masterclass](#15-senior-interview-system-design-masterclass)
16. [13-Point Production Readiness Checklist & High-Yield Q&A](#16-13-point-production-readiness-checklist--high-yield-qa)

---

## 1. Core Conceptual Mental Models for Jobs & Interviews

```
                                  KUBERNETES AT A GLANCE
┌────────────────────────────────────────────────────────────────────────────────────────┐
│  • Control Plane  : API Server (Gateway) + etcd (DB) + Scheduler (Plan) + Controllers   │
│  • Worker Nodes   : Kubelet (Agent) + kube-proxy (Networking) + Runtime (containerd)    │
│  • Workloads      : Pod (Atomic) -> ReplicaSet (Count) -> Deployment (Rollout/Rollback) │
│  • Networking     : CNI (Pod IP) -> Service (ClusterIP/VIP) -> Ingress (Host/Path L7)   │
│  • Storage        : StorageClass (Driver) -> PVC (Claim) -> PV (Physical Disk)         │
│  • Security       : 4C Model (Cloud, Cluster, Container, Code) + RBAC + NetworkPolicies │
│  • Scaling        : HPA (Pod count) + VPA (Pod sizing) + Cluster Autoscaler / Karpenter │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### High-Yield Interview Domain Summary
| Domain | Key Mechanics to Master | Senior Interview Takeaway |
| :--- | :--- | :--- |
| **Control Plane** | `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`. | Only the API server communicates with `etcd`. `etcd` is the single source of truth and requires an odd quorum ($N=3, 5$). |
| **Worker Nodes** | `kubelet`, `kube-proxy`, Container Runtime (`containerd`). | `kubelet` enforces pod specs, probes, and static pods; `kube-proxy` configures Linux kernel `iptables`/`IPVS` rules. |
| **Workloads** | Deployments, StatefulSets, DaemonSets, Jobs/CronJobs. | StatefulSets require a Headless Service (`clusterIP: None`) and `volumeClaimTemplates` for stable network identities and storage. |
| **Storage** | StorageClass, PVC, PV. Access modes: `RWO`, `ROX`, `RWX`, `RWOP`. | Use `WaitForFirstConsumer` in StorageClasses to delay volume provisioning until the scheduler picks a node in the correct AZ. |
| **Scheduling** | Taints & Tolerations, NodeAffinity, PodAntiAffinity. | Taints repel pods; tolerations allow scheduling; NodeAffinity attracts pods; PodAntiAffinity distributes replicas across AZs. |
| **Networking** | ClusterIP, NodePort, LoadBalancer, Ingress, NetworkPolicies. | Applying a NetworkPolicy isolates selected pods into **Default Deny** mode for ingress/egress. |
| **Security** | RBAC, ServiceAccounts, PSA Restricted, `securityContext`. | Base64 Secrets are encoded, not encrypted. True security requires KMS envelope encryption-at-rest and Vault/ESO. |
| **Incident Triage**| `Pending` $\rightarrow$ describe (events); `CrashLoop` $\rightarrow$ logs `--previous`; `OOMKilled` $\rightarrow$ Exit 137. | Distinguish Exit 137 (OOMKill by Linux kernel cgroup limit) from Exit 1/143 (application exception / SIGTERM). |

---

## 2. Kubernetes Architecture: Control Plane vs. Worker Nodes

```
+-----------------------------------------------------------------------+
|                             CONTROL PLANE                             |
|                                                                       |
|  +--------------------+     +-------------------+     +------------+  |
|  |     kube-apiserver |<--->|   kube-scheduler  |     |   etcd     |  |
|  | (Frontend Gateway) |     | (Node Placement)  |     | (Database) |  |
|  +---------^----------+     +-------------------+     +-----^------+  |
|            |                         |                      |         |
|            |          +--------------v----------------+     |         |
|            +--------->| kube-controller-manager       |<----+         |
|                       | (Node, Endpoint, Replica Ctrl)|               |
|                       +-------------------------------+               |
+--------------------------------------^--------------------------------+
                                       |
                   +-------------------+-------------------+
                   |                                       |
+------------------v--------------------+ +----------------v--------------------+
|             WORKER NODE 1             | |             WORKER NODE 2             |
|  +---------------------------------+  | |  +---------------------------------+  |
|  |             kubelet             |  | |  |             kubelet             |  |
|  |  (Agent, runs Pods, health)     |  | |  |  (Agent, runs Pods, health)     |  |
|  +---------------------------------+  | |  +---------------------------------+  |
|  |           kube-proxy            |  | |  |           kube-proxy            |  |
|  |  (Network rules, iptables/IPVS) |  | |  |  (Network rules, iptables/IPVS) |  |
|  +---------------------------------+  | |  +---------------------------------+  |
|  |        Container Runtime        |  | |  |        Container Runtime        |  |
|  |      (containerd / CRI-O)       |  | |  |      (containerd / CRI-O)       |  |
|  +---------------------------------+  | |  +---------------------------------+  |
|    [Pod A]   [Pod B]   [Pod C]        | |    [Pod D]   [Pod E]   [Pod F]        |
+---------------------------------------+ +---------------------------------------+
```

### Control Plane Components
1. **`kube-apiserver` (Port 6443):**
   - The central REST gateway. Authenticates, authorizes (RBAC), validates, mutates (Admission Controllers), and persists resource manifests into `etcd`.
2. **`etcd` (Port 2379 / 2380):**
   - Highly consistent, distributed B-tree key-value database based on the Raft consensus algorithm.
   - Backup strategy: Snapshot saves via `etcdctl` with PKI certificates (`--cacert`, `--cert`, `--key`).
3. **`kube-scheduler`:**
   - Evaluates unscheduled Pods via a 2-stage pipeline:
     1. **Filtering (Predicates):** Eliminates nodes lacking CPU/RAM requests, missing taints, or failing nodeSelectors.
     2. **Scoring (Priorities):** Ranks remaining candidate nodes from 1 to 100 based on soft constraints (PodAntiAffinity, ImageLocality).
4. **`kube-controller-manager`:**
   - Runs reconciliation control loops: Node Controller, ReplicaSet Controller, EndpointSlice Controller, ServiceAccount Controller.
   - Ensures: $\text{Current Cluster State} \equiv \text{Desired Manifest State}$.

### Worker Node Components
1. **`kubelet` (Port 10250):**
   - Node agent that talks to the Container Runtime via CRI (Container Runtime Interface) gRPC sockets.
   - Executes Startup, Liveness, and Readiness probes; manages Static Pods in `/etc/kubernetes/manifests/`.
2. **`kube-proxy`:**
   - Manages Layer 4 load-balancing rules for ClusterIP, NodePort, and LoadBalancer services via `iptables` or `IPVS` hash tables.
3. **Container Runtime (`containerd` / `CRI-O`):**
   - Pulls images, configures Linux kernel namespaces (PID, NET, MNT, IPC, UTS) and `cgroups` (CPU/Memory resource isolation).

---

## 3. Cluster Setup, Bootstrap Tools & Kubectl Mastery

### Installation Ecosystem
* **Local & CI Testing:** `Kind` (Kubernetes in Docker), `Minikube`.
* **Bare-Metal & Self-Managed:** `kubeadm` (`kubeadm init` $\rightarrow$ `kubeadm join`).
* **Enterprise Managed Cloud:** AWS EKS, GCP GKE, Azure AKS.

### Essential `kubectl` Speed Cheatsheet
```bash
# Cluster Inspection
kubectl cluster-info
kubectl get nodes -o wide
kubectl top nodes                  # CPU/Memory usage (requires metrics-server)

# Context & Namespace Switching
kubectl config get-contexts
kubectl config set-context --current --namespace=production

# Resource Operations
kubectl get pods -A -o wide        # List all pods across all namespaces
kubectl describe pod <pod-name>    # Inspect events and lifecycle states
kubectl logs -f <pod-name> -c <container> # Stream logs
kubectl exec -it <pod-name> -- /bin/sh    # Interactive debugging shell

# Fast Declarative Dry-Run Generation
export do="--dry-run=client -o yaml"
kubectl create deployment web --image=nginx $do > deploy.yaml
kubectl run debug-pod --image=busybox:1.36 --command $do -- sleep 3600 > pod.yaml
kubectl expose deploy web --port=80 --target-port=80 $do > svc.yaml
```

---

## 4. Workloads & Lifecycle Management

### Pods, Sidecars & Init Containers
A **Pod** is the atomic deployable unit encapsulating one or more tightly coupled containers.

```
       +-----------------------------------------+
       |               POD INSTANCE              |
       |                                         |
       |  +----------------+  +---------------+  |
       |  | App Container  |  | Sidecar Proxy |  |
       |  |  (Nginx/Node)  |  |  (Log/Envoy)  |  |
       |  +-------+--------+  +-------+-------+  |
       |          |                   |          |
       |          +---------+---------+          |
       |                    |                    |
       |             [Shared Volume]             |
       |         Shared Network (localhost)      |
       +-----------------------------------------+
```

* **Pod Phases:** `Pending` $\rightarrow$ `Running` $\rightarrow$ `Succeeded` / `Failed` (or `Unknown`).
* **Init Containers:** Execute to completion sequentially *before* application containers start (ideal for DB schema migrations or waiting on external dependencies).
* **Sidecar Containers:** Run alongside the primary container to enhance capabilities (e.g., Fluent Bit log shipper, Envoy service mesh proxy, Vault Agent secret injector).

---

### Deployments, ReplicaSets & Zero-Downtime Rollouts
* **ReplicaSet:** Guarantees a specified number of identical Pod replicas are active.
* **Deployment:** High-level controller managing ReplicaSet versions to enable declarative updates.

```
Deployment (v2 Update)
   │
   ├──> ReplicaSet v1 (Scaling Down: 3 -> 2 -> 1 -> 0)
   └──> ReplicaSet v2 (Scaling Up:   0 -> 1 -> 2 -> 3)
```

#### Deployment Strategies
* **RollingUpdate (Default):** Incrementally replaces old Pods with new ones. Controlled via `maxSurge` (e.g., 25%) and `maxUnavailable` (e.g., 0% for zero downtime).
* **Recreate:** Terminates all v1 pods before creating v2 pods. Incurs downtime; used when dual-version database schema concurrency is impossible.
* **Canary / Blue-Green:** Traffic splitting via Ingress / Service routing (e.g., Argo Rollouts, Istio).

```bash
# Rollout Lifecycle Management
kubectl rollout status deployment/webapp
kubectl rollout history deployment/webapp
kubectl rollout pause deployment/webapp
kubectl rollout resume deployment/webapp
kubectl rollout undo deployment/webapp --to-revision=2
```

---

### DaemonSets & StatefulSets
* **DaemonSet:** Ensures that **every** (or selected) worker node runs exactly one copy of a Pod.
  - *Use Cases:* Node log shippers (`Fluent Bit`, `Vector`), metric collectors (`node-exporter`), CNI agents (`Calico`, `Cilium`).
* **StatefulSet:** Designed for stateful, distributed database and messaging workloads (PostgreSQL, Kafka, Elasticsearch).
  - *Key Characteristics:*
    1. Stable, deterministic hostnames: `redis-0`, `redis-1`, `redis-2`.
    2. Ordered startup (`0 -> 1 -> 2`) and graceful teardown (`2 -> 1 -> 0`).
    3. Dedicated volume provisioning via `volumeClaimTemplates` (storage persists across pod restarts and rescheduling).
    4. Requires a **Headless Service** (`clusterIP: None`) for peer-to-peer DNS discovery (`redis-0.redis-headless.database.svc.cluster.local`).

---

### Jobs & CronJobs
* **Job:** Supervises batch workloads until a specified number of successful completions (`completions`, `parallelism`, `backoffLimit`).
* **CronJob:** Executes Jobs on a recurring schedule using standard cron syntax (`*/15 * * * *`).
  - *Best Practice:* Ensure batch jobs are **idempotent**; configure `ttlSecondsAfterFinished: 100` for automatic pod and resource cleanup.

---

## 5. Configuration & Secret Management

| Feature | ConfigMap | Secret |
| :--- | :--- | :--- |
| **Data Type** | Non-sensitive plaintext (URLs, log levels, flags) | Sensitive credentials (passwords, tokens, TLS certs) |
| **Encoding** | Plain text | Base64 encoded by default (NOT encrypted by default!) |
| **Injection Methods** | Environment variables or Mounted volume files | Environment variables or Mounted volume files |
| **Production Rule** | Version ConfigMaps (`app-cfg-v1`) | Encrypt `etcd` at rest + integrate AWS Secrets Manager / Vault |

```yaml
# ConfigMap and Secret Volume Mount Blueprint
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  template:
    spec:
      containers:
      - name: api
        image: api:v1
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: db-credentials
        volumeMounts:
        - name: tls-vol
          mountPath: /etc/tls
          readOnly: true
      volumes:
      - name: tls-vol
        secret:
          secretName: api-tls-cert
```

---

## 6. Persistent Storage Architecture (PV, PVC, StorageClass)

Kubernetes decouples infrastructure storage provisioning from application logic:

```
[ Pod / StatefulSet ] ──> [ PersistentVolumeClaim (PVC) ] ──> [ StorageClass ] ──> [ PersistentVolume (PV) / Cloud EBS ]
```

1. **PersistentVolume (PV):** Cluster-scoped physical storage volume provisioned statically or dynamically.
2. **PersistentVolumeClaim (PVC):** Namespace-scoped request for storage specifying capacity and access mode.
3. **StorageClass:** Defines dynamic provisioner plugin (`ebs.csi.aws.com`), storage tier (`gp3`), IOPS, and binding mode (`WaitForFirstConsumer`).

### Access Modes Comparison
| Access Mode | Code | Description | Cloud Example |
| :--- | :---: | :--- | :--- |
| **ReadWriteOnce** | `RWO` | Mounted as read-write by a single cluster node | AWS EBS, GCP Persistent Disk |
| **ReadOnlyMany** | `ROX` | Mounted as read-only by multiple cluster nodes | Shared Read-Only NFS |
| **ReadWriteMany** | `RWX` | Mounted as read-write simultaneously by many nodes | AWS EFS, Azure Files, NFS |
| **ReadWriteOncePod**| `RWOP`| Mounted as read-write by a single Pod across the entire cluster | CSI Volume Guard |

---

## 7. Resource Management, Probes & Scheduling Intelligence

### Requests vs. Limits & QoS Classes
* **Requests (Minimum Guaranteed):** Used exclusively by `kube-scheduler` to find candidate nodes.
* **Limits (Hard Maximum):** Enforced by the container runtime:
  - Exceeding CPU Limit $\rightarrow$ Container is **throttled** (CFS quota penalty; performance degrades).
  - Exceeding Memory Limit $\rightarrow$ Container is **OOMKilled** (`Exit Code 137` by Linux kernel cgroup).

```
Quality of Service (QoS) Eviction Hierarchy:
┌──────────────────────────────────────────────────────────────┐
│ 1. BestEffort  (No requests, no limits)   ──> Evicted FIRST │
│ 2. Burstable   (Requests < Limits)        ──> Evicted SECOND│
│ 3. Guaranteed  (Requests == Limits)       ──> Evicted LAST  │
└──────────────────────────────────────────────────────────────┘
```

---

### Container Probes (Health Checks)
| Probe Type | Primary Responsibility | Failure Remediation |
| :--- | :--- | :--- |
| **Liveness Probe** | "Is the application deadlocked or in an unrecoverable state?" | Kubelet kills the container and restarts it (`restartPolicy`). |
| **Readiness Probe**| "Is the application ready to accept incoming traffic?" | Kubelet removes Pod IP from Service Endpoints (stops traffic). |
| **Startup Probe**  | "Is slow legacy/JVM application still initializing?" | Disables liveness/readiness probes until startup passes. |

---

### Scheduling Rules: Taints, Tolerations & Affinity
* **Taints (Node Property):** Repels Pods (`kubectl taint nodes node1 dedicated=gpu:NoSchedule`).
* **Tolerations (Pod Property):** Allows Pods with matching keys/effects to schedule on tainted nodes.
* **Node Affinity:** Attracts Pods to specific node labels (`requiredDuringScheduling...` = Hard; `preferredDuringScheduling...` = Soft).
* **Pod Anti-Affinity:** Distributes replica Pods across distinct failure domains (`topologyKey: topology.kubernetes.io/zone`).

---

## 8. Networking, CoreDNS, Ingress & NetworkPolicies

### End-to-End Traffic Flow

```
User (Internet)
      │
      ▼
[ Cloud Load Balancer / ALB / NLB ]
      │
      ▼
[ Ingress Controller (TLS Termination & Routing) ]
      │
      ▼
[ Kubernetes Service (ClusterIP Virtual IP) ]
      │ (kube-proxy / iptables / IPVS Load Balancing)
      ▼
[ Target Application Pod (10.244.x.x) ]
```

### Service Types
1. **ClusterIP (Default):** Virtual stable cluster-internal IP; only accessible inside the cluster.
2. **NodePort:** Exposes the service on every node's IP at a static port (`30000–32767`).
3. **LoadBalancer:** Provisions an external cloud load balancer (AWS NLB/ALB) forwarding to NodePorts/Pods.
4. **ExternalName:** Maps service queries to an external DNS CNAME (e.g., `db.production.rds.amazonaws.com`).

### CoreDNS Resolution Architecture
* Standard FQDN lookup format:
  ```
  <service-name>.<namespace>.svc.cluster.local
  ```

### Zero-Trust NetworkPolicies (Pod Firewall)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-db-tier
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend-api
    ports:
    - protocol: TCP
      port: 5432
```
* Applying a NetworkPolicy transitions matching Pods into **Default Deny** mode for unlisted traffic.

---

## 9. Autoscaling & High Availability Engineering

### The Autoscaling Triad
1. **Horizontal Pod Autoscaler (HPA):** Adjusts replica counts based on CPU, memory, or custom Prometheus metrics.
2. **Vertical Pod Autoscaler (VPA):** Right-sizes container CPU/Memory request baselines over time.
3. **Cluster Autoscaler & Karpenter:** Dynamically provisions and terminates cloud compute nodes when Pods enter `Pending` status.

### High Availability (HA) Cluster Blueprint
* **Control Plane:** 3 Control Plane nodes across 3 separate Availability Zones (AZs) maintaining an odd `etcd` quorum ($N=3, 5$).
* **PodDisruptionBudgets (PDB):** Enforces minimum available replicas during voluntary node drains and upgrades:
  ```yaml
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: web-pdb
  spec:
    minAvailable: 2
    selector:
      matchLabels:
        app: web
  ```

---

## 10. Observability: Monitoring & Centralized Logging

```
                          OBSERVABILITY STACK
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
   [ METRICS ]                                         [ LOGS ]
• Prometheus (Scrapes metrics)                  • Fluent Bit (Node agent)
• kube-state-metrics (Cluster state)            • Grafana Loki / Elasticsearch
• node-exporter (Host CPU/RAM/Disk)             • Centralized JSON indexing
• Grafana (Visual Dashboards)                   • Grafana / Kibana (Search)
• Alertmanager (Slack/PagerDuty)
```

* **Three Pillars:** Metrics (*What is happening*), Logs (*What happened*), Traces (*Where latency occurred via OpenTelemetry*).

---

## 11. Cluster & Pod Security Hardening (4C Security Model)

```
        +---------------------------------------------------+
        |                       CLOUD                       |
        |  (IAM Roles, Security Groups, KMS, Node Hardening)|
        +-------------------------+-------------------------+
                                  │
        +-------------------------v-------------------------+
        |                      CLUSTER                      |
        |  (RBAC, NetworkPolicies, PSA, Admission Webhooks) |
        +-------------------------+-------------------------+
                                  │
        +-------------------------v-------------------------+
        |                     CONTAINER                     |
        |  (Non-Root UID, Read-Only FS, Drop Capabilities)  |
        +-------------------------+-------------------------+
                                  │
        +-------------------------v-------------------------+
        |                       CODE                        |
        |  (Static Analysis, Dependency Scans, TLS Enforce) |
        +---------------------------------------------------+
```

### Hardened Production Container Security Context
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  runAsGroup: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL
```

---

## 12. Modern CI/CD, GitOps & Helm

### Helm (Package Manager)
* Packages Kubernetes manifests into reusable **Charts** parameterized dynamically via `values.yaml`.
* Provides release versioning and atomic rollbacks: `helm rollback my-release 1`.

### GitOps with Argo CD
* **Core Tenet:** Git is the **Single Source of Truth** for desired cluster state.
* The in-cluster controller pulls manifests from Git and reconciles live cluster state automatically.
* Eliminates manual `kubectl apply` commands in production and prevents configuration drift.

---

## 13. AWS EKS Production Architecture

```
                                AWS CLOUD REGION
┌─────────────────────────────────────────────────────────────────────────────────┐
│  Amazon EKS Managed Control Plane (Multi-AZ API Server + etcd Quorum)          │
└────────────────────────────────────────┬────────────────────────────────────────┘
                                         │
                 ┌───────────────────────┴───────────────────────┐
                 ▼                                               ▼
   [ Managed Node Group: General ]                 [ Managed Node Group: Compute ]
   • m5.large (Multi-AZ)                           • c5.xlarge (Spot + On-Demand)
   • Storage: EBS gp3 via CSI                      • IAM Roles for ServiceAccounts (IRSA)
   • Ingress: AWS Load Balancer Controller (ALB)   • Observability: CloudWatch / Prometheus
```

* **IRSA (IAM Roles for Service Accounts):** Binds AWS IAM roles directly to Kubernetes ServiceAccounts via OIDC federation, eliminating hardcoded IAM credentials.

---

## 14. Troubleshooting & Common Production Incident Matrix

```
Problem Detected
  │
  ├──> Pod is "Pending" ───────> kubectl describe pod <name>
  │                              (Check Events: Insufficient CPU/Memory, Taints, Unbound PVC)
  │
  ├──> "CrashLoopBackOff" ────> kubectl logs <name> --previous
  │                              (Check Application exit codes, missing config/env, failed DB conn)
  │
  ├──> "ImagePullBackOff" ────> kubectl describe pod <name>
  │                              (Check Image tag spelling, ImagePullSecret, Registry auth)
  │
  ├──> "OOMKilled" (Exit 137) ─> kubectl describe pod <name> / kubectl top pod
  │                              (Increase memory limits, fix app memory leak)
  │
  └──> Node is "NotReady" ─────> kubectl describe node <node> / SSH to node
                                 (Check Kubelet daemon, disk space pressure, memory pressure)
```

---

## 15. Senior Interview System Design Masterclass

When asked: *"Design a Production-Ready, Multi-Tenant, Highly Available Kubernetes Platform"*:

1. **Requirements & Scale:** Compute sizing, stateless vs stateful workloads, RTO/RPO SLAs, multi-tenancy model.
2. **Infrastructure & HA:** Multi-AZ managed control plane (EKS/GKE), split node groups (Spot for batch, On-Demand for core), Karpenter autoscaling.
3. **Networking & Ingress:** Cilium eBPF CNI, AWS Load Balancer Controller with ALB Ingress, TLS termination via cert-manager, CoreDNS autoscaling.
4. **Security & Governance:** Namespace isolation per team, RBAC + IRSA, NetworkPolicies default deny, Pod Security Standards (Restricted), Trivy container scanning.
5. **Observability & Operations:** Prometheus/Grafana metrics, Loki/Fluent Bit logging, Argo CD GitOps pipelines, automated etcd backup snapshots.

---

## 16. 13-Point Production Readiness Checklist & High-Yield Q&A

### Production Readiness Checklist
| Category | Verification Item | Status |
| :--- | :--- | :---: |
| **Architecture** | Control plane and worker nodes span $\ge 3$ Availability Zones | [ ] |
| **Resilience** | Liveness, Readiness, and Startup probes defined for all services | [ ] |
| **Resilience** | `PodDisruptionBudgets` (PDB) configured for critical workloads | [ ] |
| **Resources** | CPU & Memory `requests` and `limits` set on every container | [ ] |
| **Scaling** | HPA and Cluster Autoscaler / Karpenter configured and tested | [ ] |
| **Storage** | Dynamic CSI `StorageClass` configured; daily snapshot backups verified | [ ] |
| **Security** | RBAC least-privilege enforced; no default ServiceAccount automount | [ ] |
| **Security** | Containers run as non-root with read-only root filesystems | [ ] |
| **Security** | NetworkPolicies applied to enforce namespace and pod boundary isolation | [ ] |
| **Observability**| Prometheus scraping cluster components + Grafana dashboards active | [ ] |
| **Observability**| Centralized log pipeline indexing JSON logs with alerts configured | [ ] |
| **Operations** | `etcd` automated daily backup snapshots tested with restore drills | [ ] |
| **Delivery** | GitOps (Argo CD) implemented; automated rollbacks configured | [ ] |

---

### High-Yield Senior Interview Q&A
1. **Q: What is the exact difference between `requests` and `limits` in Kubernetes?**  
   *Answer:* `requests` are used exclusively by `kube-scheduler` during node filtering to guarantee minimum allocated compute resources. `limits` are enforced at runtime by the Linux kernel `cgroups`; exceeding CPU limits causes CPU throttling, whereas exceeding Memory limits triggers an immediate `OOMKilled` termination (`Exit Code 137`).
2. **Q: Why are Kubernetes Secrets not secure by default?**  
   *Answer:* Kubernetes Secrets are only Base64-encoded strings stored in plaintext within `etcd`. To secure secrets in production, you must enable `etcd` encryption-at-rest with KMS keys and integrate external secret management solutions (HashiCorp Vault, AWS Secrets Manager via External Secrets Operator).
3. **Q: What is the difference between a Liveness Probe and a Readiness Probe?**  
   *Answer:* A failing Liveness Probe causes `kubelet` to terminate and restart the container. A failing Readiness Probe removes the Pod's IP address from the Service's active Endpoints without restarting the container, preventing user traffic from reaching an unhealthy or initializing instance.
4. **Q: How does `WaitForFirstConsumer` solve storage scheduling issues?**  
   *Answer:* In multi-AZ cloud clusters, `Immediate` volume binding can provision an EBS volume in AZ-A before the Pod is scheduled, while the scheduler later places the Pod in AZ-B (causing volume mount failure). `WaitForFirstConsumer` delays volume provisioning until the Pod is placed on a node, guaranteeing storage is provisioned in the identical Availability Zone.


