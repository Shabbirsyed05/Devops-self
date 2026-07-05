# Kubernetes Interview Guide

> **Source:** [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `08-kubernetes/` folder
> **Purpose:** Master Kubernetes architecture, networking, scheduling, and troubleshooting for interviews

---

## Table of Contents

| # | Topic |
|---|-------|
| [1](#1-cluster-architecture) | Cluster Architecture |
| [2](#2-components-interaction) | Components Interaction |
| [3](#3-kubernetes-services) | Kubernetes Services |
| [4](#4-pod-ip-communication) | Pod IP Communication |
| [5](#5-types-of-services) | Types of Services |
| [6](#6-labels-and-selectors) | Labels and Selectors |
| [7](#7-nodeport-vs-loadbalancer) | NodePort vs LoadBalancer |
| [8](#8-service-and-kube-proxy) | Service and Kube-Proxy |
| [9](#9-disadvantages-of-loadbalancer-service) | Disadvantages of LoadBalancer Service |
| [10](#10-headless-service) | Headless Service |
| [11](#11-pod-access-service-in-different-namespace) | Pod Access Service in Different Namespace |
| [12](#12-network-policies) | Network Policies |
| [13](#13-deployment-strategy) | Deployment Strategy |
| [14](#14-rollback-strategy) | Rollback Strategy |
| [15](#15-avoid-rollbacks) | Avoid Rollbacks |
| [16](#16-all-deployment-strategies) | All Deployment Strategies |
| [17](#17-coredns) | CoreDNS |
| [18](#18-taints-and-tolerations) | Taints and Tolerations |
| [19](#19-crashloopbackoff) | CrashLoopBackOff |
| [20](#20-liveness-vs-readiness-probes) | Liveness vs Readiness Probes |
| [21](#21-loadbalancer-vs-ingress) | LoadBalancer vs Ingress |
| [22](#22-clusterip-works-but-ingress-fails) | ClusterIP Works but Ingress Fails |
| [23](#23-ingress-controller-vs-ingress) | Ingress Controller vs Ingress |
| [24](#24-custom-ingress-controller) | Custom Ingress Controller |
| [25](#25-only-one-replica-running) | Only One Replica Running |
| [26](#26-configmap-changes-not-reflected) | ConfigMap Changes Not Reflected |
| [27](#27-node-affinity) | Node Affinity |
| [28](#28-node-affinity-vs-nodeselector) | Node Affinity vs nodeSelector |
| [29](#29-container-runtime) | Container Runtime |

---

## 1. Cluster Architecture

> **Q: Can you explain the architecture of a Kubernetes cluster and the components involved?**

### Answer

A Kubernetes cluster consists of a **Control Plane** (the brain) and multiple **Worker Nodes** (where apps run).

---

### Control Plane Components

| Component | Purpose |
|---|---|
| **kube-apiserver** | Entry point to the cluster — all communication goes through this REST API |
| **etcd** | Distributed key-value store that holds all cluster state and configuration |
| **kube-scheduler** | Assigns pods to nodes based on resources, taints/tolerations, and affinities |
| **controller-manager** | Runs controllers (Node, ReplicaSet, Job) to maintain desired state |

### Worker Node Components

| Component | Purpose |
|---|---|
| **kubelet** | Agent on each node — communicates with API server and ensures containers are running |
| **kube-proxy** | Manages networking rules to route traffic to the correct pod |
| **Container Runtime** | Runs the containers (e.g. `containerd`, `CRI-O`) |

### Add-Ons

| Add-on | Purpose |
|---|---|
| **CoreDNS** | Resolves service and pod names to IPs |
| **Ingress Controller** | Manages HTTP/HTTPS access from outside the cluster |
| **Metrics Server** | Collects resource metrics for autoscaling |

### Communication Flow

```
kubectl apply -f deployment.yaml
    │
    ▼
kube-apiserver  ──stores──►  etcd
    │
    ▼
kube-scheduler  ──assigns node──►  kubelet (on chosen node)
    │
    ▼
Container Runtime  ──pulls image & starts container──►  Pod Running
    │
    ▼
kube-proxy  ──sets up routing rules──►  Service traffic works
```

---

## 2. Components Interaction

> **Q: What happens behind the scenes when you run `kubectl apply -f pod.yaml`?**

### Answer

| Step | Component | Action |
|---|---|---|
| 1 | `kubectl` | Sends REST request to kube-apiserver |
| 2 | `kube-apiserver` | Authenticates, validates, and persists desired state in etcd |
| 3 | `kube-scheduler` | Watches for unscheduled pods, picks the best node based on resources and constraints |
| 4 | `kube-scheduler` | Updates pod spec with `nodeName` |
| 5 | `kubelet` | Detects the pod assignment, pulls the image, starts the container |
| 6 | `containerd` | Runs the actual container process |
| 7 | `kube-proxy` | Sets up iptables/ipvs rules if the pod is part of a Service |
| 8 | `CoreDNS` | Resolves internal DNS so other pods can reach it by name |

### Verify

```bash
kubectl get pods
kubectl describe pod myapp
```

---

## 3. Kubernetes Services

> **Q: What role does a Kubernetes Service play in a cluster?**

### Answer

A **Service** provides a stable network identity (fixed IP and DNS name) to access dynamic, ephemeral pods. It load-balances traffic across all healthy pod replicas.

### Key Features

| Feature | Detail |
|---|---|
| **Stable IP/DNS** | Fixed ClusterIP and DNS: `myapp.default.svc.cluster.local` |
| **Load Balancing** | Distributes requests across all matching healthy pods |
| **Pod Discovery** | Enables other services to locate backend pods by label |
| **Decoupling** | Clients don't need to know pod IPs — they use the service name |

### Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

---

## 4. Pod IP Communication

> **Q: Why should you avoid hardcoding pod IPs for inter-service communication?**

### Answer

Pod IPs are **ephemeral**. When a pod restarts, scales, or gets rescheduled, it receives a new IP. Always use a **Service name** instead.

```python
# BAD — breaks on any pod restart or reschedule
requests.post("http://10.244.1.17:5000/api")

# GOOD — always resolves to the correct pod(s)
requests.post("http://auth-service.default.svc.cluster.local:5000/api")
```

> **Key takeaway:** Pod IPs are temporary. Use Services to decouple applications from the underlying pod lifecycle.

---

## 5. Types of Services

> **Q: What are the different types of Kubernetes Services?**

### Answer

| Type | Exposed Outside Cluster | Use Case |
|---|---|---|
| **ClusterIP** (default) | No | Internal microservice communication |
| **NodePort** | Yes — via node IP + static port | Dev/test quick access |
| **LoadBalancer** | Yes — via cloud public IP | Production external traffic |
| **ExternalName** | Yes — via DNS alias | Connect to external services |
| **Headless** | No — DNS returns pod IPs directly | StatefulSets needing stable pod identities |

### Examples

```yaml
# ClusterIP (default)
spec:
  type: ClusterIP

# NodePort
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080

# LoadBalancer
spec:
  type: LoadBalancer

# ExternalName
spec:
  type: ExternalName
  externalName: db.mycompany.com

# Headless
spec:
  clusterIP: None
```

---

## 6. Labels and Selectors

> **Q: What are labels and selectors in Kubernetes?**

### Answer

**Labels** are key-value metadata attached to objects. **Selectors** filter or group objects based on those labels. Services use selectors to find their target pods.

```yaml
# Pod — has labels
metadata:
  labels:
    app: frontend
    env: production

# Service — selects pods by label
spec:
  selector:
    app: frontend
```

### Why They Matter

| Use Case | How Labels Help |
|---|---|
| Service-to-Pod routing | Selector matches pods to receive traffic |
| Rolling updates & scaling | Deployments track pods by label |
| Monitoring | Prometheus scrapes targets by label |
| Scheduling | Node affinity uses node labels |

---

## 7. NodePort vs LoadBalancer

> **Q: When exposing an app externally — NodePort or LoadBalancer?**

### Answer

| Scenario | Recommended Type |
|---|---|
| Local testing on Minikube | **NodePort** |
| Dev/staging (quick access) | **NodePort** |
| Production in the cloud | **LoadBalancer** |
| Multiple services, one IP | **Ingress** (preferred over both) |

> **Key takeaway:** Use `LoadBalancer` for production external traffic. Use `NodePort` for dev/test. Use `Ingress` when you need routing rules or cost efficiency.

---

## 8. Service and Kube-Proxy

> **Q: What is the relationship between Services and kube-proxy?**

### Answer

A **Service** defines a virtual IP and routing rules. **kube-proxy** is the component that actually **implements** those rules on every node.

| Component | Role |
|---|---|
| **Kubernetes Service** | Declares virtual IP, port, and pod selector |
| **kube-proxy** | Watches Service/Endpoints objects and programs routing rules |
| **Endpoints / EndpointSlices** | Lists the real pod IPs backing the service |
| **iptables / ipvs / eBPF** | The actual mechanism that forwards packets on the node |

### How It Works

```
Client → Service ClusterIP → kube-proxy rules (iptables/ipvs) → Pod IP
```

---

## 9. Disadvantages of LoadBalancer Service

> **Q: What are the limitations of the `LoadBalancer` service type?**

### Answer

| Limitation | Detail |
|---|---|
| **Cost** | Provisions one cloud load balancer per service — expensive at scale |
| **Scalability** | Cannot reuse a single LB across multiple services |
| **Vendor lock-in** | Only works with cloud providers (AWS, GCP, Azure) — not local/on-prem |
| **No L7 routing** | Only Layer 4 (TCP/UDP) — no path-based or host-based routing |

### Solution: Use Ingress Instead

| Feature | LoadBalancer | Ingress |
|---|---|---|
| Cost per service | High (1 LB each) | Low (1 shared LB) |
| HTTP/path routing | No | Yes |
| TLS termination | Manual | Built-in |
| Works locally | No | Yes |

---

## 10. Headless Service

> **Q: What is a headless service and when would you use it?**

### Answer

A **headless service** sets `clusterIP: None`. Instead of returning a single virtual IP, DNS returns the **individual pod IPs** directly. Used primarily with **StatefulSets**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
```

Each StatefulSet pod gets a **stable, predictable DNS name**:

```
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
```

| Feature | ClusterIP Service | Headless Service |
|---|---|---|
| DNS returns | Single virtual IP | Individual pod IPs |
| Load balancing | Yes | No |
| Best for | Stateless apps | StatefulSets (databases, queues) |

---

## 11. Pod Access Service in Different Namespace

> **Q: Can a pod access a Service in a different namespace?**

### Answer

Yes. Use the **fully qualified domain name (FQDN)**:

```
<service-name>.<namespace>.svc.cluster.local
```

```bash
# Pod in namespace "frontend" accessing service in namespace "backend"
curl http://api.backend.svc.cluster.local:80
```

| Consideration | Notes |
|---|---|
| **NetworkPolicy** | Can restrict cross-namespace traffic — check policies |
| **RBAC** | DNS reachability ≠ RBAC access. Secrets/APIs still need RBAC. |
| **Headless services** | Same FQDN pattern applies for individual pod DNS |

---

## 12. Network Policies

> **Q: How can you restrict access to a DB pod to only one app in the same namespace?**

### Answer

Create a **NetworkPolicy** that allows ingress only from pods with a specific label.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-db
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: api
      ports:
        - protocol: TCP
          port: 5432
```

| Traffic | Result |
|---|---|
| `role: api` pod → `role: db` pod on port 5432 | Allowed |
| Any other pod → `role: db` pod | Blocked |

> **Note:** NetworkPolicies are enforced by the CNI plugin (Calico, Cilium, etc.). Without a CNI that supports them, policies have no effect.

---

## 13. Deployment Strategy

> **Q: What deployment strategy does your organization follow?**

### Answer

**Rolling Update** is the default for most services. **Canary deployments** are used for critical services.

### Rolling Update

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1         # max extra pods during update
    maxUnavailable: 1   # max pods that can be unavailable
```

### Canary Deployment

Route a small percentage of traffic to the new version, observe metrics, then gradually increase.

```
10% traffic → new version  →  metrics OK?  →  50%  →  100%
```

Tools: **Argo Rollouts**, **Flagger**

### Blue-Green Deployment

Run v1 (blue) and v2 (green) in parallel. Switch traffic via Ingress or LB once v2 is validated.

### Tooling

| Tool | Purpose |
|---|---|
| **ArgoCD** | GitOps-based deployment |
| **Argo Rollouts** | Progressive/canary delivery |
| **Prometheus** | Health and SLO monitoring |
| **Helm** | Templated, versioned deployments |

---

## 14. Rollback Strategy

> **Q: How does your team handle rollbacks?**

### Answer

### GitOps Rollback (Primary)

```bash
git revert <bad-commit>
git push origin main
# ArgoCD detects the change and rolls back automatically
```

### Kubectl Rollback

```bash
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2
kubectl rollout history deployment/my-app
```

### Helm Rollback

```bash
helm rollback my-app 2   # roll back to revision 2
```

### Canary Auto-Rollback

Argo Rollouts pauses and rolls back automatically if success rate drops below a defined threshold.

### Safeguards

| Practice | Purpose |
|---|---|
| Pre-deploy schema validation | Prevents broken YAML from reaching the cluster |
| Readiness and liveness probes | Catch bad pods before they receive traffic |
| SLO-based auto rollback | Canary rollouts abort when metrics degrade |
| ArgoCD drift alerts | Notifies when live state diverges from Git |

---

## 15. Avoid Rollbacks

> **Q: How would you design a strategy to minimize rollbacks?**

### Answer

### 1. Pre-Deployment Safety Nets

| Check | Tool |
|---|---|
| Unit / integration tests | Jest, pytest, JUnit |
| Schema validation | `kubeval`, `kubeconform`, OPA/Gatekeeper |
| Security scanning | Trivy, Snyk, Checkov |
| Static code analysis | SonarQube |

### 2. Progressive Delivery

- **Canary deployments** — Argo Rollouts / Flagger
- **Feature flags** — decouple deploy from release
- **Blue-Green** — zero-risk switch with instant rollback

### 3. Observability + Quality Gates

Monitor error rate, latency, resource usage, and logs. Auto-pause or abort the release if metrics degrade.

### 4. GitOps

All deployments flow through Git. No manual `kubectl apply` in production. Every change is traceable and revertible.

---

## 16. All Deployment Strategies

> **Q: Explain the various deployment strategies you've used.**

### Answer

| Strategy | How It Works | Downtime | Risk | Best For |
|---|---|---|---|---|
| **Rolling Update** | Replaces old pods gradually | None | Low-medium | Most services |
| **Blue-Green** | Deploy v2 alongside v1; switch traffic | None | Low | Critical services |
| **Canary** | Route small % to v2; expand gradually | None | Very low | High-traffic, critical services |
| **Recreate** | Shut down all old pods, start new | Yes | High | Dev/test only |

### Rolling Update
Kubernetes default. Controlled by `maxSurge` and `maxUnavailable`.

### Blue-Green
Two identical environments. Switch via Ingress annotation or LB target group. Instant rollback: just switch back.

### Canary
Argo Rollouts / Flagger route 5–10% of traffic to the new version. Prometheus metrics gate progression.

---

## 17. CoreDNS

> **Q: What is the role of CoreDNS in a Kubernetes cluster?**

### Answer

CoreDNS is the **default DNS server** for Kubernetes service discovery. It translates service names to Cluster IPs (or individual pod IPs for headless services).

### How It Works

```
Pod makes DNS request for "api-service.default.svc.cluster.local"
    │
    ▼
CoreDNS (ClusterIP: 10.96.0.10)
    │
    ▼
Queries Kubernetes API for the service
    │
    ▼
Returns ClusterIP (or pod IPs for headless)
```

### CoreDNS ConfigMap

```yaml
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        reload
        loadbalance
    }
```

> **Key takeaway:** Without CoreDNS, pods cannot resolve service names — all inter-service DNS would break.

---

## 18. Taints and Tolerations

> **Q: A node is tainted with `NoSchedule`. Can you still schedule a pod on it?**

### Answer

Yes — if the pod has a **matching toleration**.

```bash
# Taint a node
kubectl taint nodes node-1 env=dev:NoSchedule
```

```yaml
# Pod with matching toleration
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "dev"
      effect: "NoSchedule"
```

### Taint Effects

| Effect | Behavior |
|---|---|
| `NoSchedule` | New pods without toleration are not placed on the node |
| `PreferNoSchedule` | Scheduler tries to avoid, but may still place pods |
| `NoExecute` | New pods rejected AND existing pods without toleration are evicted |

### Common Use Cases

| Use Case | How |
| Dedicated GPU nodes | Taint GPU nodes; only ML workloads have toleration |
| Reserved control-plane nodes | Prevent user pods from landing on control-plane |
| Maintenance | Taint node with `NoExecute` to drain it gracefully |

---

## 19. CrashLoopBackOff

> **Q: A pod is stuck in `CrashLoopBackOff`. How do you troubleshoot?**

### Answer

`CrashLoopBackOff` means the container is starting, crashing, and Kubernetes keeps restarting it with exponential backoff.

### Step-by-Step Troubleshooting

```bash
# Step 1: Check pod events and status
kubectl describe pod <pod-name>

# Step 2: View logs from the crashed container
kubectl logs <pod-name> --previous

# Step 3: Open a debug shell
kubectl debug -it <pod-name> --image=busybox --copy-to=debug-pod
```

### Common Root Causes

| Root Cause | How to Identify | Fix |
|---|---|---|
| App startup error | Logs show exception/panic | Fix application code or config |
| Missing config/secret | Env var `<nil>` or mount error in events | Add ConfigMap or Secret |
| Wrong CMD/ENTRYPOINT | Container exits with code 127 | Fix Dockerfile |
| OOM killed | Exit code 137 in `describe` | Increase memory limits |
| Dependency not ready | Connection refused in logs | Add `initContainers` or retry logic |

---

## 20. Liveness vs Readiness Probes

> **Q: What is the difference between liveness and readiness probes?**

### Answer

| Feature | Liveness Probe | Readiness Probe |
|---|---|---|
| Question answered | "Is the app alive?" | "Is the app ready to serve traffic?" |
| Failure action | Kubernetes **restarts** the container | Kubernetes **removes pod from service endpoints** |
| Affects routing? | No | Yes |
| Use for | Detecting deadlocks/hangs | Waiting for startup, DB connections |

### Liveness Probe Example

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
```

### Readiness Probe Example

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
  failureThreshold: 2
```

> **Key takeaway:** Use both together. Liveness catches broken processes; readiness gates traffic until the app is truly ready.

---

## 21. LoadBalancer vs Ingress

> **Q: What is the difference between Ingress and a LoadBalancer service?**

### Answer

| Feature | LoadBalancer Service | Ingress |
|---|---|---|
| External IP per service | Yes (one LB per service) | Shared (one LB for all) |
| HTTP routing rules | No | Yes (host/path-based) |
| TLS termination | Manual | Built-in |
| Cost efficient | No | Yes |
| Handles non-HTTP | Yes (TCP/UDP) | No (HTTP/HTTPS only) |
| Requires controller | No | Yes (Ingress Controller) |

### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

---

## 22. ClusterIP Works but Ingress Fails

> **Q: The app works fine with ClusterIP but fails when accessed through Ingress. How do you troubleshoot?**

### Answer

### Checklist

```bash
# 1. Is the Ingress Controller running?
kubectl get pods -n ingress-nginx

# 2. Check the Ingress resource for misconfiguration
kubectl describe ingress <name>

# 3. Check the controller logs
kubectl logs -n ingress-nginx <controller-pod>

# 4. Verify backend service endpoints exist
kubectl get endpoints <service-name>
```

| Check | Command / Action |
|---|---|
| Ingress Controller running | `kubectl get pods -n ingress-nginx` |
| Correct `ingressClassName` | `ingressClassName: nginx` in spec |
| Correct path and host | `kubectl describe ingress <name>` |
| DNS → Ingress external IP | `nslookup app.example.com` |
| TLS secret valid | `kubectl get secret <tls-secret>` |
| Backend endpoints not empty | `kubectl get endpoints <service>` |

---

## 23. Ingress Controller vs Ingress

> **Q: Why is an Ingress Controller needed if I already created an Ingress resource?**

### Answer

| Component | What It Is | Role |
|---|---|---|
| **Ingress Resource** | Kubernetes object (YAML) | Declares routing rules (host, path, backend) |
| **Ingress Controller** | A running pod (e.g. nginx) | Watches Ingress resources and actually implements the rules |

Creating an Ingress resource without a controller is like writing a config file with no app to read it — **traffic is never routed**.

### Popular Ingress Controllers

| Controller | Best For |
|---|---|
| `ingress-nginx` | General purpose, most widely used |
| `traefik` | Dynamic config, microservices |
| `aws-alb-ingress-controller` | AWS ALB integration |
| `kong` | API gateway features |

> **Key takeaway:** The Ingress resource is the *definition*. The Ingress Controller is the *implementation*.

---

## 24. Custom Ingress Controller

> **Q: Can we use an in-house load balancer with Kubernetes Ingress?**

### Answer

Yes — but the in-house LB must forward traffic to a **running Ingress Controller** inside the cluster. The LB itself cannot read Kubernetes Ingress YAML.

```
Client
  │
  ▼
In-house Load Balancer  (routes to cluster NodePort/IP)
  │
  ▼
Ingress Controller Pod  (reads Ingress rules, routes to services)
  │
  ▼
Service → Pods
```

> **Key takeaway:** Any LB can sit in front of Kubernetes, but only an Ingress Controller understands and enforces Ingress routing rules.

---

## 25. Only One Replica Running

> **Q: A Deployment is set to `replicas: 3` but only 1 pod is running. What's wrong?**

### Answer

```bash
# Step 1: Check pod statuses
kubectl get pods -l app=my-app

# Step 2: Check events on the deployment and pods
kubectl describe deployment my-app
kubectl describe pod <pending-pod>

# Step 3: Check node resources
kubectl describe nodes | grep -A5 "Allocated resources"
```

### Common Causes

| Root Cause | How to Identify | Fix |
|---|---|---|
| Insufficient CPU/memory on nodes | Events: `Insufficient cpu` | Add nodes or reduce resource requests |
| Taints blocking scheduling | Events: `node(s) had taint` | Add toleration or remove taint |
| PVC not bound | Pod stuck in `Pending` | Fix PVC/StorageClass config |
| Pod anti-affinity too strict | Events: `didn't match pod affinity` | Relax affinity rules |
| Image pull failing | `ImagePullBackOff` status | Fix image name or registry credentials |

---

## 26. ConfigMap Changes Not Reflected

> **Q: A pod mounts a ConfigMap, but after updating the ConfigMap, the changes aren't visible. Why?**

### Answer

### Root Causes

| Cause | Explanation |
|---|---|
| **App reads config at startup only** | No live reload — needs pod restart |
| **Kubelet sync delay** | File-mounted ConfigMaps update every ~1–2 min |
| **Env var injection** | ConfigMap values used as `env:` require pod restart — never hot-reload |
| **Editing wrong ConfigMap** | Double-check name and namespace |

### Fix — Restart the Deployment

```bash
kubectl rollout restart deployment <name>
```

### Fix — Auto-rollout on ConfigMap Change (Helm)

```yaml
# Add a checksum annotation so pods restart when ConfigMap changes
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

> **Key takeaway:** File-mounted ConfigMaps do sync eventually, but env-var-injected ones never do. Always restart pods after a ConfigMap change to be safe.

---

## 27. Node Affinity

> **Q: How does Node Affinity work and when would you use it?**

### Answer

Node Affinity constrains which nodes a pod can be scheduled on, based on **node labels**. It is more expressive than `nodeSelector`.

### Types

| Type | Behavior |
|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | Hard rule — pod will **not** schedule if no node matches |
| `preferredDuringSchedulingIgnoredDuringExecution` | Soft rule — prefers matching node but can fall back |

### Example

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

### Use Cases

| Use Case | Why |
|---|---|
| GPU workloads | Schedule ML pods only on GPU-enabled nodes |
| Zone awareness | Pin stateful apps to a specific availability zone |
| Storage requirements | Run DB pods only on SSD-backed nodes |

---

## 28. Node Affinity vs nodeSelector

> **Q: What is the difference between `nodeSelector` and `nodeAffinity`?**

### Answer

| Feature | `nodeSelector` | `nodeAffinity` |
|---|---|---|
| Syntax | Simple key=value | Full expression with operators |
| Operators supported | Only `=` (exact match) | `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt` |
| Rule types | Required only | Required **or** Preferred |
| Flexibility | Low | High |

### `nodeSelector` — Simple

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### `nodeAffinity` — Advanced

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

> **Key takeaway:** Start with `nodeSelector` for simple needs. Switch to `nodeAffinity` when you need `NotIn`, `Exists`, or preferred (soft) scheduling rules.

---

## 29. Container Runtime

> **Q: What is a Container Runtime in Kubernetes?**

### Answer

The **Container Runtime** is the software on each worker node responsible for pulling images, starting/stopping containers, and reporting their status to the Kubelet.

### Architecture

```
Kubelet  →  CRI (Container Runtime Interface)  →  Container Runtime (containerd)
```

Kubernetes communicates with runtimes through the **CRI standard**, so any CRI-compliant runtime works.

### Popular Runtimes

| Runtime | Description |
|---|---|
| **containerd** | Lightweight, production-ready — now the default in most Kubernetes distributions |
| **CRI-O** | Purpose-built for Kubernetes — used by OpenShift |
| **Docker** | Deprecated and removed as a runtime in Kubernetes v1.24+ |

> **Key takeaway:** Kubernetes uses `containerd` or `CRI-O` via the CRI standard. Docker is no longer supported as a runtime — but Docker-built images still work fine since they follow the OCI image spec.

---

## Quick Reference Cheatsheet

```
=========================================================
         KUBERNETES INTERVIEW CHEATSHEET
=========================================================

  ARCHITECTURE
  ------------
  Control Plane: apiserver | etcd | scheduler | controller-manager
  Worker Node:   kubelet | kube-proxy | container runtime
  Add-ons:       CoreDNS | Ingress Controller | Metrics Server

  SERVICES
  --------
  ClusterIP   = internal only (default)
  NodePort    = node IP + static port (dev/test)
  LoadBalancer = cloud public IP (production)
  Headless    = clusterIP: None → returns pod IPs (StatefulSets)
  Cross-NS:   <svc>.<ns>.svc.cluster.local

  SCHEDULING
  ----------
  Labels + Selectors → how services find pods
  nodeSelector  → simple exact-match node filtering
  nodeAffinity  → advanced operators + required/preferred rules
  Taints        → repel pods from nodes
  Tolerations   → allow pods onto tainted nodes

  PROBES
  ------
  Liveness  → failure = restart container
  Readiness → failure = remove from service endpoints (no restart)

  INGRESS
  -------
  Ingress Resource = routing rules (YAML)
  Ingress Controller = implements the rules (nginx pod)
  Ingress >> LoadBalancer: shared IP, L7 routing, TLS, cheaper

  DEPLOYMENTS
  -----------
  Rolling Update  → default, maxSurge + maxUnavailable
  Blue-Green      → parallel envs, instant switch
  Canary          → % traffic split, metric-gated (Argo Rollouts)
  Rollback:  kubectl rollout undo | helm rollback | git revert

  DEBUGGING
  ---------
  CrashLoopBackOff → kubectl logs --previous | describe pod
  Exit 137         → OOMKilled → increase memory limit
  Pending pod      → insufficient resources or taint mismatch
  Ingress fails    → check controller running + endpoints not empty

=========================================================
```

---

## Interview Tips

| Do This | Not This |
|---|---|
| For architecture → walk through the full `kubectl apply` flow | List components without explaining interaction |
| For Services → explain *why* ClusterIP is the default | Just list the 4 service types |
| For Ingress → distinguish the *resource* from the *controller* | Say "just create an Ingress" |
| For CrashLoopBackOff → check `--previous` logs and exit codes | Just say "check the logs" |
| For probes → explain the different failure actions | Treat liveness and readiness as the same |
| For rolling update → mention `maxSurge` and `maxUnavailable` | Say "Kubernetes updates it automatically" |
| For ConfigMap → mention env-var vs file-mount difference | Just say "restart the pod" |

---

## Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/)
- [Argo Rollouts](https://argoproj.github.io/rollouts/)
- [CoreDNS for Kubernetes](https://coredns.io/plugins/kubernetes/)
- [Original Source: devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)

---

> Star this repo if it helped you prepare for your DevOps interview!
> Drop the next topic's raw notes and they will be formatted and added here.
