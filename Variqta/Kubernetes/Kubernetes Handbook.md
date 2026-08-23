# ☸️ VERIQTA Kubernetes Visual Master Handbook
> **The Ultimate 20-Page Quick Reference Guide: From Core Concepts to Enterprise Production & Interviews.**  
> *Practical. Visual. Interview-Ready. Designed for DevOps Engineers, SREs, Cloud Architects, and CKA/CKAD Candidates.*

---

## 📑 Table of Contents
1. [Executive Summary & High-Yield Mnemonics](#1-executive-summary--high-yield-mnemonics)
2. [Page 1: Introduction to Kubernetes (Origins, Why & Borg)](#page-1-introduction-to-kubernetes)
3. [Page 2: Kubernetes Architecture Overview (Brain vs. Hands & Legs)](#page-2-kubernetes-architecture-overview)
4. [Page 3: Control Plane Components Deep Dive (`apiserver`, `etcd`, `scheduler`, `controller-manager`)](#page-3-control-plane-components-deep-dive)
5. [Page 4: Worker Node Components Deep Dive (`kubelet`, `kube-proxy`, Container Runtime)](#page-4-worker-node-components-deep-dive)
6. [Page 5: Pods Explained (Lifecycle, Multi-Container, Sidecars)](#page-5-pods-explained)
7. [Page 6: ReplicaSets (Desired State & Self-Healing Engine)](#page-6-replicasets)
8. [Page 7: Deployments (Zero-Downtime Rollouts & Instant Rollbacks)](#page-7-deployments)
9. [Page 8: Services (ClusterIP, NodePort, LoadBalancer, ExternalName)](#page-8-services)
10. [Page 9: Namespaces (Multi-Tenancy & Resource Isolation)](#page-9-namespaces)
11. [Page 10: ConfigMaps & Secrets (Configuration Management & Etcd Security)](#page-10-configmaps--secrets)
12. [Page 11: Labels & Selectors (Equality-Based vs. Set-Based Targeting)](#page-11-labels--selectors)
13. [Page 12: Ingress (Layer 7 Host & Path Routing, TLS Termination)](#page-12-ingress)
14. [Page 13: StatefulSets (Stable Network Identity, Headless Service & PVCs)](#page-13-statefulsets)
15. [Page 14: DaemonSets (Node-Level Agents & Infrastructure Daemons)](#page-14-daemonsets)
16. [Page 15: Jobs & CronJobs (Finite Batch Workloads & Scheduled Automation)](#page-15-jobs--cronjobs)
17. [Page 16: Kubernetes Networking (Flat CNI Model & NetworkPolicies)](#page-16-kubernetes-networking)
18. [Page 17: Scaling in Kubernetes (HPA, VPA, Karpenter & Cluster Autoscaler)](#page-17-scaling-in-kubernetes)
19. [Page 18: Kubernetes Security & Hardening (RBAC, PSA & SecurityContext)](#page-18-kubernetes-security--hardening)
20. [Page 19: Kubernetes + Docker + CI/CD (From Source Code to Cloud)](#page-19-kubernetes--docker--cicd)
21. [Page 20: Master Interview Questions, Must-Know CLI Cheat Sheet & Scorecard](#page-20-master-interview-questions--cli-cheat-sheet)

---

## 1. Executive Summary & High-Yield Mnemonics

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

### High-Yield Mnemonics for Job Interviews
* **P.O.D.S.** $\rightarrow$ **P**ortable, **O**perable, **D**isposable, **S**hare nothing.
* **K.I.S.S.** $\rightarrow$ **K**eep **I**t **S**imple (Use Deployments for stateless apps; don't manage bare ReplicaSets or Pods).
* **D.R.Y.** $\rightarrow$ **D**on't **R**epeat **Y**ourself (Use Helm Charts and Kustomize overlays for multi-environment deployments).
* **H.P.A.** $\rightarrow$ **H**andle **P**eak **A**utomatically (Horizontal Pod Autoscaling based on metrics).
* **S.E.C.U.R.E.** $\rightarrow$ **S**ervice Accounts, **E**ncrypt secrets at rest, **C**lusterRoles least privilege, **U**ser isolation (Namespaces), **R**unAsNonRoot, **E**nable Audit Logs.

---

## Page 1: Introduction to Kubernetes

```
                   THE EVOLUTION OF CONTAINER ORCHESTRATION
  2014                     2015                     2016                   TODAY
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐   ┌──────────────────┐
│ Google open-     │ ──> │ CNCF Founded     │ ──> │ 1st CNCF         │ ─>│ De Facto Cloud   │
│ sources K8s      │     │ Kubernetes 1.0   │     │ Graduated Project│   │ Operating System │
│ (Borg Heritage)  │     │ launched         │     │ Production Ready │   │ Everywhere       │
└──────────────────┘     └──────────────────┘     └──────────────────┘   └──────────────────┘
```

### 1. What is Kubernetes (K8s)?
* **Definition:** An open-source container orchestration platform designed to automate the deployment, scaling, healing, and operational lifecycle of containerized applications.
* **The "8" in K8s:** An abbreviation representing the 8 letters between "K" and "s" in "Kubernetes" (Greek for *helmsman* or *pilot*).

### 2. Problems Before Kubernetes vs. Solutions Provided
| Problem Before Kubernetes | How Kubernetes Solves It |
| :--- | :--- |
| **Manual deployments & script sprawl** | Declarative YAML configuration (`kubectl apply -f manifest.yaml`). |
| **Difficult scaling during traffic spikes** | Automated horizontal pod autoscaling (HPA) and node provisioning. |
| **Poor server resource utilization** | Bin-packing scheduler places workloads based on CPU/RAM requests. |
| **Unplanned downtime on node failure** | Automated self-healing: detects dead nodes and reschedules pods elsewhere. |
| **Inconsistent environments (dev vs prod)** | Identical container runtime standards across on-prem, cloud, and hybrid. |

### 3. Google Borg Heritage
* **Borg:** Google's internal cluster management system running billions of containers weekly.
* **Kubernetes:** The open-source evolution of Borg, written in Go, incorporating 10+ years of Google production lessons learned.

### 4. 6 Core Pillars of Kubernetes
1. **Automation:** Eliminates manual sysadmin tasks.
2. **Scalability:** Scale up/down seamlessly in seconds.
3. **High Availability:** Self-healing, automated failover, and rolling updates.
4. **Efficiency:** Optimal compute packing and resource quotas.
5. **Portability:** Runs on AWS, GCP, Azure, bare-metal, or local laptops.
6. **Extensibility:** Custom Resource Definitions (CRDs) and Operator ecosystem.

---

## Page 2: Kubernetes Architecture Overview

```
+-----------------------------------------------------------------------+
|                       CONTROL PLANE (The Brain)                       |
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
|          WORKER NODE 1 (Hands/Legs)   | |          WORKER NODE 2 (Hands/Legs)   |
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

### High-Level Responsibility Split
* **Control Plane (The Brain):** Makes global decisions (scheduling, replication, scaling, upgrades) and reacts to cluster events. *Does not run user application containers.*
* **Worker Nodes (Hands & Legs):** The physical or virtual machines that execute the actual workloads inside Pods and report health back to the control plane.

### End-to-End User Request Flow
```
1. User Request  ──> 2. Ingress / LoadBalancer  ──> 3. kube-proxy (iptables)  ──> 4. Pod Container  ──> 5. HTTP Response
```

---

## Page 3: Control Plane Components Deep Dive

```
                               CONTROL PLANE ARCHITECTURE
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │                                                                                        │
 │  kubectl / Clients ──> [ kube-apiserver ] <──> [ etcd ] (Port 2379)                    │
 │                              │                                                         │
 │                    ┌─────────┴─────────┐                                               │
 │                    ▼                   ▼                                               │
 │          [ kube-scheduler ]  [ kube-controller-manager ]                               │
 │                                                                                        │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

| Component | Port | Primary Role | Key Responsibilities | Communicates With |
| :--- | :---: | :--- | :--- | :--- |
| **`kube-apiserver`** | `6443` | Frontend Gateway | Authenticates, authorizes (RBAC), validates requests, and reads/writes to `etcd`. | All components, `kubectl`, clients, `kubelet`. |
| **`kube-scheduler`** | `10259` | Node Placement | Evaluates resource requests, taints, affinity rules, and binds pods to nodes. | `kube-apiserver` |
| **`kube-controller-manager`** | `10257` | Desired State Reconciler | Runs Node Controller, ReplicaSet Controller, EndpointSlice Controller, Service Controller. | `kube-apiserver` |
| **`etcd`** | `2379 / 2380` | Distributed Key-Value Store | Stores entire cluster state, configs, secrets, and metadata. Raft consensus. | **Only `kube-apiserver`** |

> **Interview Golden Rule:** No component talks directly to `etcd` except `kube-apiserver`. If `etcd` fails, the cluster cannot read or write any configuration (existing running pods remain running, but zero orchestration occurs).

---

## Page 4: Worker Node Components Deep Dive

```
                             WORKER NODE ARCHITECTURE
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │                                                                                        │
 │  kube-apiserver (HTTPS) <──> [ kubelet ] (Port 10250)                                  │
 │                                    │                                                   │
 │               ┌────────────────────┴────────────────────┐                              │
 │               ▼                                         ▼                              │
 │      [ Container Runtime ] (CRI)               [ kube-proxy ] (Networking)             │
 │      (containerd / CRI-O)                      (iptables / IPVS rules)                 │
 │               │                                         │                              │
 │               ▼                                         ▼                              │
 │       ┌───────────────┐                         ┌───────────────┐                      │
 │       │ [ Pod A ]     │                         │ Node Ports    │                      │
 │       │ [ Pod B ]     │                         │ ClusterIP VIPs│                      │
 │       └───────────────┘                         └───────────────┘                      │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

### Worker Components Breakdown
1. **`kubelet`:**
   - The primary node agent. Receives `PodSpecs` from the API Server and instructs the container runtime to launch containers.
   - Executes Startup, Liveness, and Readiness health probes. Reports node and pod health back to the control plane.
2. **`kube-proxy`:**
   - Manages Layer 4 network routing on each node by translating virtual ClusterIPs and NodePorts into actual Pod IP addresses using Linux kernel `iptables` or `IPVS` tables.
3. **Container Runtime:**
   - Implements the Container Runtime Interface (CRI) to pull container images, run containers, and enforce Linux `cgroups` (CPU/RAM limits) and namespaces (network/PID isolation).
   - *Note on Docker:* Dockershim was deprecated in K8s 1.20 and removed in 1.24+. Production clusters use **`containerd`** or **`CRI-O`**.

---

## Page 5: Pods Explained

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

### 1. What is a Pod?
* The **smallest atomic deployable unit** in Kubernetes.
* A Pod wraps one or more tightly coupled containers that share:
  1. **Network Namespace:** Shared IP address and `localhost` port space.
  2. **Storage Volumes:** Shared filesystem mount points.
  3. **Lifecycle:** Scheduled, started, and destroyed together.

### 2. Pod Lifecycle & States
```
[ Pending ] ──> [ Scheduled ] ──> [ ContainerCreating (Pulling) ] ──> [ Running ] ──> [ Succeeded / Failed ]
```

| Pod State | Technical Meaning | Typical Root Cause / Action |
| :--- | :--- | :--- |
| **`Pending`** | Pod accepted by API server but not placed on a node. | Insufficient CPU/RAM, node taints, or unbound PVC. |
| **`Running`** | Pod bound to a node and all containers created. | At least one container is executing or restarting. |
| **`Succeeded`** | All containers terminated successfully (Exit Code 0). | Batch Job completed its execution. |
| **`Failed`** | All containers terminated; at least one failed (Exit Code $\neq 0$). | Application crash, syntax error, or out-of-memory. |
| **`CrashLoopBackOff`** | Container starts, crashes, and Kubelet restarts it with backoff. | Missing environment variable, bad config, failing liveness probe. |
| **`Unknown`** | Node cannot report status to API Server. | Worker node lost network connectivity to control plane. |

---

## Page 6: ReplicaSets

```
                        REPLICASET RECONCILIATION LOOP
 ┌────────────────────────────────────────────────────────────────────────┐
 │                                                                        │
 │  Desired Replicas: 3 <─── [ ReplicaSet Controller ] ───> Live Pods: 2  │
 │                                    │                                   │
 │                           (Action: Create 1 Pod)                       │
 │                                    ▼                                   │
 │                         [Pod 1]  [Pod 2]  [Pod 3]                      │
 │                                                                        │
 └────────────────────────────────────────────────────────────────────────┘
```

### Core Purpose & Mechanics
* **Purpose:** Ensures a specified number of identical Pod replicas are running at all times (guarantees availability and self-healing).
* **Key Fields:**
  - `spec.replicas`: Desired replica count.
  - `spec.selector.matchLabels`: Labels used to discover and manage pods.
  - `spec.template`: The blueprint for creating new pods.
* **Production Rule:** Never create bare ReplicaSets manually in production; always manage them via **Deployments**.

---

## Page 7: Deployments

```
                          DEPLOYMENT ROLLING UPDATE
Deployment (v2 Image Update)
   │
   ├──> ReplicaSet v1 (Scaling Down: 3 -> 2 -> 1 -> 0) [Old Pods Terminating]
   └──> ReplicaSet v2 (Scaling Up:   0 -> 1 -> 2 -> 3) [New Pods Starting]
```

### Deployment Strategies
1. **`RollingUpdate` (Default):** Incrementally launches new version pods while draining old ones. Zero application downtime.
   - `maxSurge`: Max extra pods created during update (e.g., `25%`).
   - `maxUnavailable`: Max pods that can be down during update (e.g., `0` for strict zero downtime).
2. **`Recreate`:** Kills all v1 pods before starting v2 pods. Incurs downtime; used when database schema conflicts prevent running v1 and v2 simultaneously.
3. **Canary / Blue-Green:** Traffic splitting via Ingress controllers or Service routing.

### Essential Rollout Commands
```bash
# Update Image
kubectl set image deployment/webapp nginx=nginx:1.25

# Check Rollout Status
kubectl rollout status deployment/webapp

# Inspect Rollout History
kubectl rollout history deployment/webapp

# Instant Rollback to Previous Revision
kubectl rollout undo deployment/webapp

# Rollback to Specific Revision
kubectl rollout undo deployment/webapp --to-revision=2
```

---

## Page 8: Services

```
                           SERVICE NETWORKING OVERVIEW
                                 [ Client / User ]
                                         │
                                         ▼
                            [ Service (ClusterIP VIP) ]
                               (Stable IP + DNS Name)
                                         │
                     ┌───────────────────┴───────────────────┐
                     ▼                                       ▼
            [ Pod 1 (10.244.1.5) ]                  [ Pod 2 (10.244.2.8) ]
```

### Why Services are Required
Pods are **ephemeral**; their IP addresses change every time they restart or reschedule. A **Service** provides a permanent, stable Virtual IP (VIP) and DNS record that load-balances traffic across dynamic backing pods.

### 4 Core Service Types
| Service Type | How It Works | Use Case | Accessibility |
| :--- | :--- | :--- | :--- |
| **`ClusterIP`** (Default) | Allocates an internal cluster IP address. | Microservice-to-microservice internal communication. | Only within the cluster. |
| **`NodePort`** | Exposes the service on each node's IP at a static port (`30000–32767`). | Development, bare-metal testing, legacy edge proxies. | Outside cluster via `<NodeIP>:<NodePort>`. |
| **`LoadBalancer`** | Automatically provisions a Cloud Load Balancer (AWS NLB/ALB, GCP LB). | Public-facing production internet traffic. | Public Internet / External VPC. |
| **`ExternalName`** | Maps service DNS to an external CNAME (e.g., `db.aws.rds.com`). | Directing internal pods to external databases. | Internal DNS resolution. |

---

## Page 9: Namespaces

```
                            KUBERNETES CLUSTER
 ┌────────────────────────────────────────────────────────────────────────┐
 │                                                                        │
 │   ┌──────────────────────┐  ┌──────────────────────┐  ┌─────────────┐  │
 │   │ Namespace: dev       │  │ Namespace: staging   │  │ Namespace:  │  │
 │   │ [Pods] [Services]    │  │ [Pods] [Services]    │  │ prod        │  │
 │   │ [ConfigMaps/Secrets] │  │ [ConfigMaps/Secrets] │  │ [Pods/Svcs] │  │
 │   └──────────────────────┘  └──────────────────────┘  └─────────────┘  │
 │                                                                        │
 └────────────────────────────────────────────────────────────────────────┘
```

### Purpose & Default Namespaces
* **Virtual Partitioning:** Divides a single physical cluster into multiple isolated virtual environments for multi-tenancy, environment isolation (`dev`, `stage`, `prod`), and RBAC permission scoping.
* **Default Namespaces:**
  1. `default`: Default namespace for user workloads without an explicit namespace.
  2. `kube-system`: Reserved for Kubernetes control plane components, CoreDNS, and CNI plugins.
  3. `kube-public`: Auto-generated, publicly readable cluster information.
  4. `kube-node-lease`: Holds node heartbeat lease objects to determine node health.

---

## Page 10: ConfigMaps & Secrets

| Feature | ConfigMap | Secret |
| :--- | :--- | :--- |
| **Primary Purpose** | Store non-sensitive configuration data | Store sensitive credentials, tokens, TLS certs |
| **Data Storage** | Plain text | Base64 encoded by default (NOT encrypted!) |
| **Typical Data** | URLs, log levels, flags, JSON/YAML configs | Database passwords, API keys, private SSH/TLS keys |
| **Injection Methods** | Environment variables (`envFrom`) or volume file mounts | Environment variables (`secretKeyRef`) or volume file mounts |
| **Production Best Practice** | Version ConfigMaps (`config-v1`) | Encrypt `etcd` at rest with KMS + use HashiCorp Vault / External Secrets Operator |

```bash
# Imperative Creation Examples
kubectl create configmap app-cfg --from-literal=LOG_LEVEL=debug --from-literal=DB_PORT=5432
kubectl create secret generic db-pass --from-literal=password='SuperSecretPass123!'
```

---

## Page 11: Labels & Selectors

```
                       LABELS (Metadata) vs. SELECTORS (Queries)
  ┌─────────────────────────────────┐         ┌─────────────────────────────────┐
  │         Pod Manifest            │         │        Service Manifest         │
  │ metadata:                       │         │ spec:                           │
  │   labels:                       │ <────── │   selector:                     │
  │     app: ecommerce              │         │     app: ecommerce              │
  │     env: production             │         │     env: production             │
  │     tier: backend               │         └─────────────────────────────────┘
  └─────────────────────────────────┘
```

### Selector Types
1. **Equality-Based Selectors:** Evaluates exact key-value matches (`app = frontend`, `env != stage`). Used in basic Services and ReplicationControllers.
2. **Set-Based Selectors:** Evaluates membership expressions (`env in (production, staging)`, `tier notin (legacy)`, `version exists`). Used in Deployments, StatefulSets, and NetworkPolicies.

---

## Page 12: Ingress

```
                           INGRESS ARCHITECTURE
  Internet / Users
         │
         ▼
[ Ingress Controller (NGINX / AWS ALB) ] ──> Evaluates Host & Path Rules
         │
         ├──> Host: shop.example.com/orders  ──> [ Orders Service (Port 8080) ]
         └──> Host: shop.example.com/catalog ──> [ Catalog Service (Port 9090) ]
```

### What is Ingress?
* **Ingress Resource:** A declarative Kubernetes manifest defining Layer 7 HTTP/HTTPS routing rules based on hostnames and URL paths.
* **Ingress Controller:** The actual reverse proxy engine (e.g., NGINX Ingress Controller, Traefik, AWS Load Balancer Controller) that reads Ingress resources and configures load-balancing rules.
* **Key Features:** Single IP for multiple services, TLS/SSL certificate termination, URL path rewrites (`rewrite-target`), and rate limiting.

---

## Page 13: StatefulSets

```
                       STATEFULSET DEPLOYMENT ORDER
  ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
  │     mysql-0     │ ───>  │     mysql-1     │ ───>  │     mysql-2     │
  │ [Dedicated PVC] │       │ [Dedicated PVC] │       │ [Dedicated PVC] │
  └─────────────────┘       └─────────────────┘       └─────────────────┘
```

### Core Characteristics of StatefulSets
1. **Predictable Identity:** Pods receive sticky, deterministic ordinal names (`mysql-0`, `mysql-1`, `mysql-2`).
2. **Ordered Startup & Teardown:** Pods start sequentially ($0 \rightarrow 1 \rightarrow 2$) and terminate in reverse order ($2 \rightarrow 1 \rightarrow 0$).
3. **Dedicated Persistent Storage:** Uses `volumeClaimTemplates` to provision a separate PersistentVolumeClaim for each pod that persists across restarts and rescheduling.
4. **Headless Service (`clusterIP: None`):** Creates direct DNS A-records for each pod for peer-to-peer clustering (`mysql-0.mysql-headless.database.svc.cluster.local`).

---

## Page 14: DaemonSets

```
                          DAEMONSET ARCHITECTURE
  Worker Node 1                 Worker Node 2                 Worker Node 3
┌──────────────────────┐      ┌──────────────────────┐      ┌──────────────────────┐
│ [ Fluent Bit Agent ] │      │ [ Fluent Bit Agent ] │      │ [ Fluent Bit Agent ] │
│ [ App Workload Pod ] │      │ [ App Workload Pod ] │      │ [ App Workload Pod ] │
└──────────────────────┘      └──────────────────────┘      └──────────────────────┘
```

* **Definition:** Ensures that **every** (or selected) worker node runs exactly one copy of a Pod.
* **Primary Use Cases:**
  1. Cluster Log Collection: `Fluent Bit`, `Fluentd`, `Vector`.
  2. Node Monitoring Exporters: `Prometheus node-exporter`.
  3. Security & Runtime Monitoring: `Falco`, `Wiz Agent`.
  4. Cluster Networking: `Calico CNI`, `Cilium`, `kube-proxy`.

---

## Page 15: Jobs & CronJobs

```
  Job (Batch)     ──> Runs finite container tasks ──> Completed (Exit 0) / Retried on failure
  CronJob (Sched) ──> Runs Jobs on a recurring schedule (e.g., '0 2 * * *' = 2 AM daily)
```

### Concurrency Policies for CronJobs
* **`Allow` (Default):** Multiple jobs can execute concurrently.
* **`Forbid`:** If a previous job is still running, the new job execution is skipped.
* **`Replace`:** Cancels the currently running job and starts a new one.

---

## Page 16: Kubernetes Networking

```
                             FLAT CNI NETWORK MODEL
┌────────────────────────────────────────────────────────────────────────────────────────┐
│  • Rule 1: All Pods can communicate with all other Pods without NAT.                   │
│  • Rule 2: All Nodes can communicate with all Pods without NAT.                        │
│  • Rule 3: The IP a Pod sees for itself is the same IP others see for it.              │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### CoreDNS Resolution Blueprint
* Standard internal DNS format:
  ```
  <service-name>.<namespace>.svc.cluster.local
  ```

### Zero-Trust NetworkPolicies
* By default, Kubernetes networking is **flat and open** (any pod can reach any pod).
* When a `NetworkPolicy` selects a pod, that pod enters **Default Deny** mode for unlisted traffic.

---

## Page 17: Scaling in Kubernetes

```
                              THE SCALING TRIAD
 ┌───────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Horizontal Pod Autoscaler (HPA) : Scales Pod REPLICAS based on CPU/RAM/Prometheus. │
 │ 2. Vertical Pod Autoscaler (VPA)   : Adjusts Pod CPU/RAM REQUESTS & LIMITS baseline.  │
 │ 3. Cluster Autoscaler / Karpenter  : Adds/terminates physical CLOUD VM NODES.         │
 └───────────────────────────────────────────────────────────────────────────────────────┘
```

### Scaling Comparison Matrix
| Feature | HPA | VPA | Cluster Autoscaler / Karpenter |
| :--- | :--- | :--- | :--- |
| **What it scales** | Number of Pod replicas | CPU / Memory resource sizes | Number of cloud VM worker nodes |
| **Scope** | Application Deployment | Container resource requests | Kubernetes Cluster compute pool |
| **Trigger Metric** | CPU/Memory utilization % | Historical usage trends | Pods trapped in `Pending` state |
| **Requires Restart** | No (adds/removes pods) | Yes (recreates pod with new sizing) | No |

---

## Page 18: Kubernetes Security & Hardening

```
                      THE 4C SECURITY MODEL FOR CLOUD-NATIVE
 ┌───────────────────────────────────────────────────────────────────────────────────────┐
 │ CLOUD      : IAM least privilege, VPC Security Groups, KMS envelope encryption.       │
 │ CLUSTER    : RBAC (Roles/RoleBindings), NetworkPolicies default-deny, Audit Logs.     │
 │ CONTAINER  : Non-root user (UID 10001), read-only root filesystem, drop ALL caps.    │
 │ CODE       : Static security analysis (SAST), image vulnerability scanning (Trivy).   │
 └───────────────────────────────────────────────────────────────────────────────────────┘
```

### Production Security Context Blueprint
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  runAsGroup: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

---

## Page 19: Kubernetes + Docker + CI/CD

```
                             FROM CODE TO PRODUCTION CLOUD
 ┌──────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
 │ Code PR  │ ───> │ CI Build  │ ───> │ Registry  │ ───> │ GitOps CD │ ───> │ Amazon    │
 │ (GitHub) │      │ (Actions) │      │ (ECR/GCR) │      │ (Argo CD) │      │ EKS Prod  │
 └──────────┘      └───────────┘      └───────────┘      └───────────┘      └───────────┘
```

### Helm Packaging & Release Management
* **Helm:** The package manager for Kubernetes.
* **Chart Structure:** `Chart.yaml` (metadata), `values.yaml` (default configurations), `templates/` (parameterized YAML manifests).
* **Rollbacks:** `helm rollback <release-name> <revision-number>`.

---

## Page 20: Master Interview Questions & CLI Cheat Sheet

### Top 10 High-Yield Kubernetes Interview Questions & Answers
1. **Q: What is the difference between a Deployment and a StatefulSet?**  
   *Answer:* Deployments manage interchangeable stateless pods with random hostnames and shared storage. StatefulSets manage stateful workloads requiring stable ordinal identities (`app-0`), ordered startup, dedicated volume bindings via `volumeClaimTemplates`, and a Headless Service (`clusterIP: None`).
2. **Q: What happens when a container exceeds its CPU limit vs. its Memory limit?**  
   *Answer:* Exceeding CPU limit causes CPU throttling via CFS quotas (application slows down). Exceeding Memory limit causes immediate container termination by the Linux kernel OOM-killer with `Exit Code 137`.
3. **Q: Why are Kubernetes Secrets not secure out-of-the-box?**  
   *Answer:* Secrets are only Base64-encoded strings stored in plaintext within `etcd`. Production security requires enabling `etcd` KMS encryption-at-rest and using external secret managers (HashiCorp Vault, AWS Secrets Manager via External Secrets Operator).
4. **Q: What is the difference between Liveness and Readiness Probes?**  
   *Answer:* Liveness probe failure restarts the container. Readiness probe failure removes the Pod's IP address from Service endpoints to stop incoming traffic without restarting the container.
5. **Q: What is `WaitForFirstConsumer` in a StorageClass?**  
   *Answer:* Delays volume provisioning until the Pod is placed on a node, guaranteeing storage is provisioned in the identical Availability Zone where the pod is scheduled.

---

### Must-Know `kubectl` Speed Reference
| Operational Task | Speed `kubectl` Command |
| :--- | :--- |
| **List all Pods across cluster** | `kubectl get pods -A -o wide` |
| **Inspect events & lifecycle errors** | `kubectl describe pod <pod-name> -n <ns>` |
| **Stream previous crashed logs** | `kubectl logs <pod-name> --previous -n <ns>` |
| **Instant force-delete a stuck pod** | `kubectl delete pod <pod-name> --grace-period=0 --force` |
| **Fast dry-run manifest generator** | `kubectl create deploy web --image=nginx --dry-run=client -o yaml > deploy.yaml` |
| **Switch active namespace** | `kubectl config set-context --current --namespace=<ns>` |
| **Inspect node resource consumption** | `kubectl top nodes` |
| **Check RBAC authorization** | `kubectl auth can-i create pods -n production --as=developer` |
| **Rollback a deployment** | `kubectl rollout undo deployment/webapp` |
