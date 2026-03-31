# Kubernetes Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `08-kubernetes/` folder

---

# Table of Contents

1. [Cluster Architecture](#1-cluster-architecture)
2. [Components Interaction](#2-components-interaction)
3. [Kubernetes Services](#3-kubernetes-services)
4. [Pod IP Communication](#4-pod-ip-communication)
5. [Types of Services](#5-types-of-services)
6. [Labels and Selectors](#6-labels-and-selectors)
7. [NodePort vs LoadBalancer](#7-nodeport-vs-loadbalancer)
8. [Service Relates to Kube-Proxy](#8-service-relates-to-kube-proxy)
9. [Disadvantages of LoadBalancer Service](#9-disadvantages-of-loadbalancer-service)
10. [Headless Service](#10-headless-service)
11. [Pod Access Service in Different Namespace](#11-pod-access-service-in-different-namespace)
12. [Network Policies](#12-network-policies)
13. [Deployment Strategy](#13-deployment-strategy)
14. [Rollback Strategy](#14-rollback-strategy)
15. [Avoid Rollbacks](#15-avoid-rollbacks)
16. [All Deployment Strategies](#16-all-deployment-strategies)
17. [CoreDNS](#17-coredns)
18. [Taints and Tolerations](#18-taints-and-tolerations)
19. [CrashLoopBackOff](#19-crashloopbackoff)
20. [Liveness vs Readiness Probes](#20-liveness-vs-readiness-probes)
21. [LoadBalancer vs Ingress](#21-loadbalancer-vs-ingress)
22. [ClusterIP Works but Ingress Fails](#22-clusterip-works-but-ingress-fails)
23. [Ingress Controller vs Ingress](#23-ingress-controller-vs-ingress)
24. [Custom Ingress Controller](#24-custom-ingress-controller)
25. [Only One Replica Running](#25-only-one-replica-running)
26. [ConfigMap Changes Not Reflected](#26-configmap-changes-not-reflected)
27. [Node Affinity](#27-node-affinity)
28. [Node Affinity vs Node Label Selector](#28-node-affinity-vs-node-label-selector)
29. [Container Runtime](#29-container-runtime)

---

# 1. Cluster Architecture

## Question
Can you explain the architecture of a Kubernetes cluster and the components involved?

## ✅ Answer
A Kubernetes cluster consists of a **Control Plane** (API Server, Scheduler, Controller Manager, etcd) and multiple **Worker Nodes** (Kubelet, Kube Proxy, Container Runtime).

### 🧠 1. Control Plane — The Brain

| Component | Purpose |
|---|---|
| **kube-apiserver** | Entry point to the cluster. All communication goes through this REST API. |
| **etcd** | Distributed key-value store for all cluster data. |
| **kube-scheduler** | Assigns pods to nodes based on resources, taints/tolerations, affinities. |
| **controller-manager** | Runs controllers (Node, ReplicaSet, Job) to maintain desired state. |

### ⚙️ 2. Worker Nodes — Where Apps Run

| Component | Purpose |
|---|---|
| **kubelet** | Agent on each node, communicates with API server, ensures containers are running. |
| **kube-proxy** | Manages networking rules to route traffic to correct pod. |
| **Container Runtime** | Runs the containers (containerd, CRI-O). |

### 🔐 3. Add-Ons

| Add-on | Purpose |
|---|---|
| **CoreDNS** | Resolves service and pod names to IPs. |
| **Ingress Controller** | Manages HTTP/HTTPS access from outside. |
| **Metrics Server** | Collects metrics for autoscaling. |

### 🔗 Communication Flow
1. `kubectl apply -f deployment.yaml`
2. `kubectl` talks to `kube-apiserver`
3. API server stores config in `etcd`
4. `scheduler` finds the best node
5. `kubelet` on that node pulls image and starts container
6. `kube-proxy` and `service` route traffic to the pod

---

# 2. Components Interaction

## Question
What happens behind the scenes when you run `kubectl apply -f pod.yaml`?

## ✅ Answer

### Step-by-Step Flow:

**Step 1:** You run `kubectl apply -f pod.yaml` → sends REST request to API server.

**Step 2:** API Server authenticates, validates, and stores desired state in **etcd**.

**Step 3:** Scheduler watches for unscheduled pods, sees the new pod has no assigned node.

**Step 4:** Scheduler picks a suitable node based on resources, taints, affinities. Updates pod spec with `nodeName`.

**Step 5:** Kubelet on the selected node pulls the image and starts the container using the container runtime.

**Step 6:** Kube-proxy sets up networking rules to route traffic if the pod is part of a Service.

**Step 7:** Pod is now running. Verify:
```bash
kubectl get pods
kubectl describe pod myapp
```

### Summary Table

| Component | Role |
|---|---|
| `kubectl` | Sends request to API |
| `kube-apiserver` | Validates & stores the object |
| `etcd` | Stores desired state |
| `kube-scheduler` | Chooses node for pod |
| `kubelet` | Pulls image and starts container |
| `containerd` | Runs the actual container |
| `kube-proxy` | Sets up network rules |
| `CoreDNS` | Resolves internal DNS |

---

# 3. Kubernetes Services

## Question
What role does a Kubernetes Service play in a cluster?

## ✅ Answer
A Service provides a **stable network identity** (IP and DNS) to access dynamic, ephemeral pods. It enables load balancing across multiple pod replicas.

### 🧩 Key Features

| Feature | Explanation |
|---|---|
| **Stable IP/DNS** | Fixed ClusterIP and DNS name (`myapp.default.svc.cluster.local`) |
| **Load Balancing** | Routes requests to all healthy pods |
| **Pod Discovery** | Enables other services to locate backend pods |
| **Supports Selectors** | Uses labels to find and group pods |

### 🧪 Example
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

### Types of Services

| Type | Use Case |
|---|---|
| `ClusterIP` (default) | Internal-only communication |
| `NodePort` | Exposes on a port of each node |
| `LoadBalancer` | Cloud provider external LB |
| `ExternalName` | Maps to external DNS |
| `Headless Service` | No LB — direct pod DNS (StatefulSets) |

---

# 4. Pod IP Communication

## Question
Why should you avoid hardcoding pod IPs for inter-service communication?

## ✅ Answer
Pod IPs are **ephemeral**. Pods can restart, scale, or be rescheduled, resulting in new IPs. Use Kubernetes Services instead.

```python
# BAD
requests.post("http://10.244.1.17:5000/api")

# GOOD
requests.post("http://auth-service.default.svc.cluster.local:5000/api")
```

> **Key takeaway:** Pod IPs are temporary. Use Services to decouple applications from underlying pod infrastructure.

---

# 5. Types of Services

## Question
What are the different types of Kubernetes Services?

## ✅ Answer

### 📦 1. ClusterIP (Default) — Internal only
```yaml
spec:
  type: ClusterIP
```

### 🌐 2. NodePort — Exposes on node IP + port
```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```

### ☁️ 3. LoadBalancer — Cloud provider LB
```yaml
spec:
  type: LoadBalancer
```

### 🌍 4. ExternalName — Maps to external DNS
```yaml
spec:
  type: ExternalName
  externalName: db.mycompany.com
```

### 🧪 5. Headless Service — Direct pod IPs
```yaml
spec:
  clusterIP: None
```

### Summary Table

| Type | Exposed Outside | Use Case |
|---|---|---|
| ClusterIP | ❌ | Internal microservice communication |
| NodePort | ✅ (node IP:port) | Dev/test quick access |
| LoadBalancer | ✅ (public IP) | External user traffic in cloud |
| ExternalName | ✅ (DNS) | Connect to external services |
| Headless | ❌ (DNS only) | Stateful apps needing stable IDs |

---

# 6. Labels and Selectors

## Question
What are labels and selectors in Kubernetes?

## ✅ Answer
**Labels** are key-value metadata pairs. **Selectors** filter or group objects based on labels.

```yaml
# Pod labels
metadata:
  labels:
    app: frontend
    env: production

# Service selector
spec:
  selector:
    app: frontend
```

### 🎯 Why Useful

| Use Case | How Labels Help |
|---|---|
| Service-to-Pod communication | Selectors match pods |
| Rolling updates & scaling | Deployments use label selectors |
| Monitoring | Prometheus uses labels |
| Node affinity & scheduling | Pods match labels on nodes |

---

# 7. NodePort vs LoadBalancer

## Question
When exposing an app externally — NodePort or LoadBalancer?

## ✅ Answer
**LoadBalancer** for production. **NodePort** for development/testing.

| Use Case | Recommended |
|---|---|
| Local testing on Minikube | NodePort |
| Dev/staging in the cloud | NodePort (sometimes) |
| Production in the cloud | LoadBalancer ✅ |

---

# 8. Service Relates to Kube-Proxy

## Question
What is the relationship between Services and kube-proxy?

## ✅ Answer
Kube-proxy **implements the logic** of Services. It runs on each node and routes service traffic to correct backend pods using **iptables**, **ipvs**, or **eBPF** rules.

| Component | Role |
|---|---|
| Kubernetes Service | Defines virtual IP + selector |
| kube-proxy | Implements traffic routing logic |
| Endpoints/EndpointSlices | Lists pod IPs backing the service |
| iptables/ipvs/eBPF | Actual mechanism for forwarding packets |

---

# 9. Disadvantages of LoadBalancer Service

## Question
What are the limitations of `LoadBalancer` service type?

## ✅ Answer

1. **💰 Cost** — One cloud LB per service (expensive)
2. **🧱 Scalability** — Can't reuse same LB for multiple services
3. **🛠 Vendor Lock-In** — Only works in cloud providers
4. **🚦 No L7 Routing** — Only Layer 4 (TCP/UDP), no path-based routing

### Alternative: Use Ingress Controller
- One LB + multiple services
- Better routing, SSL termination, cost efficiency

| Feature | LoadBalancer | Ingress |
|---|---|---|
| Cost per service | High (1 LB each) | Low (1 LB shared) |
| HTTP routing | ❌ | ✅ |
| Works locally | ❌ | ✅ |

---

# 10. Headless Service

## Question
What is a headless service and when did you use it?

## ✅ Answer
A service with `clusterIP: None`. DNS returns **individual pod IPs** instead of a single ClusterIP.

```yaml
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
```

Pods in StatefulSet get stable DNS:
```
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
```

| Feature | ClusterIP Service | Headless Service |
|---|---|---|
| DNS returns | Single ClusterIP | Individual Pod IPs |
| Load balancing | Yes | No |
| Use with StatefulSet | ❌ Not ideal | ✅ Recommended |

---

# 11. Pod Access Service in Different Namespace

## Question
Can a pod access a Service in a different namespace?

## ✅ Answer
Yes. Use the **fully qualified service DNS name**:
```
<service-name>.<namespace>.svc.cluster.local
```

Example:
```bash
curl http://api.backend.svc.cluster.local:80
```

| Item | Notes |
|---|---|
| **NetworkPolicies** | Can restrict cross-namespace communication |
| **RBAC** | DNS reachability ≠ RBAC. API/secrets still need RBAC. |
| **Headless Services** | Same FQDN pattern applies |

---

# 12. Network Policies

## Question
How can you restrict access to a DB pod to only one app in the same namespace?

## ✅ Answer
Create a **NetworkPolicy** that allows **ingress** only from pods with a specific label.

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

Result: `app-0` → `db-0` on port 5432 ✅ | Any other pod → `db-0` ❌

---

# 13. Deployment Strategy

## Question
What deployment strategy does your organization follow?

## ✅ Answer
**Rolling Update** (default) + **Canary deployments** for critical services.

### 🔁 1. Rolling Update
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

### 🧪 2. Canary Deployment
Using Argo Rollouts / Flagger: `10% → 50% → 100%` with checks between steps.

### 🚦 3. Blue-Green (less frequent)
Deploy v2 alongside v1. Switch traffic via LB or Ingress.

### ⚙️ Tooling

| Tool | Purpose |
|---|---|
| **ArgoCD** | GitOps-based deployment |
| **Argo Rollouts** | Progressive delivery |
| **Prometheus** | Health and SLO monitoring |
| **Helm** | Templated deployments |

---

# 14. Rollback Strategy

## Question
How does your team handle rollbacks?

## ✅ Answer

### 🔁 GitOps Rollback (Primary)
```bash
git revert <bad-commit>
git push origin main
```
Argo CD picks up the change and rolls back automatically.

### 🧪 Canary Rollback
Argo Rollouts auto-pauses if success rate drops below threshold.

### ⚙️ Helm Rollback
```bash
helm rollback my-app 2
```

### 🔐 Safeguards

| Strategy | Purpose |
|---|---|
| Pre-deploy validation | Prevents broken YAML |
| Readiness & liveness probes | Catch bad pods early |
| SLO-based auto rollback | Canary rollouts monitored via metrics |
| Alerts on sync divergence | Argo CD notifies on drift |

---

# 15. Avoid Rollbacks

## Question
How would you design a strategy to minimize rollbacks?

## ✅ Answer

### ✅ 1. Pre-Deployment Safety Nets
- Automated testing (unit, integration, regression)
- Schema validation (`kubeval`, `kubeconform`, `opa`)
- Security scanning (`Trivy`, `Snyk`, `Checkov`)
- Static code analysis (SonarQube)

### 🚦 2. Progressive Delivery
- Canary deployments (Argo Rollouts / Flagger)
- Feature flags
- Blue-Green deployments

### 🔍 3. Observability + Quality Gates
Monitor latency, error rate, resource usage, logs. Auto-pause releases if metrics degrade.

### 🧠 4. GitOps
All deployments through Git. No manual `kubectl apply`.

---

# 16. All Deployment Strategies

## Question
Explain the various deployment strategies you've used.

## ✅ Answer

### 1. 🔁 Rolling Update (Most Common)
- Replaces old pods gradually with new ones
- Zero downtime
- Tools: Kubernetes Deployments, Argo CD

### 2. 🟦🟩 Blue-Green Deployment
- Deploy new version in parallel
- Switch traffic when validated
- Tools: AWS ALB + target groups, Helm, Jenkins

### 3. 🧪 Canary Deployment
- Gradually roll out to small % of users
- Observe metrics before increasing traffic
- Tools: Argo Rollouts, Flagger, Prometheus + Grafana

---

# 17. CoreDNS

## Question
What is the role of CoreDNS in a Kubernetes cluster?

## ✅ Answer
CoreDNS is the **default DNS server** for service discovery. It translates service names to Pod/Cluster IPs.

### 🔁 How It Works
1. Pod makes DNS request
2. Request goes to CoreDNS ClusterIP (`10.96.0.10`)
3. CoreDNS resolves via Kubernetes API
4. Returns Cluster IP or Pod IP (headless)

### 🔧 Configuration (ConfigMap)
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
        loop
        reload
        loadbalance
    }
```

---

# 18. Taints and Tolerations

## Question
A node is tainted with `NoSchedule`. Can you still schedule a pod?

## ✅ Answer
Yes, if the pod has a **matching toleration**.

```bash
kubectl taint nodes node-1 env=dev:NoSchedule
```

```yaml
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "dev"
      effect: "NoSchedule"
```

| Taint Effect | Behavior |
|---|---|
| NoSchedule | Won't place pods unless they tolerate |
| PreferNoSchedule | Tries to avoid, may still place |
| NoExecute | Evicts running pods unless they tolerate |

---

# 19. CrashLoopBackOff

## Question
Pod is stuck in `CrashLoopBackOff`. How do you troubleshoot?

## ✅ Answer

### Step-by-Step:

1. **Check Pod Status:** `kubectl describe pod <pod-name>`
2. **View Logs:** `kubectl logs <pod-name> --previous`
3. **Check ConfigMaps/Secrets:** Missing or misconfigured?
4. **Check Dependencies:** Database reachable? DNS resolving?
5. **Check Resources:** `OOMKilled`? Increase memory limits.
6. **Check Image/CMD:** Wrong image version? Missing binary?
7. **Debug:** `kubectl debug -it <pod-name> --image=busybox`

---

# 20. Liveness vs Readiness Probes

## Question
What is the difference between liveness and readiness probes?

## ✅ Answer

### 🔁 Liveness Probe — "Is the app alive?"
- Failure → container is **restarted**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

### 🟢 Readiness Probe — "Is the app ready to serve?"
- Failure → removed from **service endpoints** (not restarted)
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```

| Feature | Liveness | Readiness |
|---|---|---|
| Checks if app is | **Alive** | **Ready to serve** |
| Failure Action | Restarts container | Removes from routing |
| Affects traffic? | ❌ No | ✅ Yes |

---

# 21. LoadBalancer vs Ingress

## Question
What is the difference between Ingress and LoadBalancer service?

## ✅ Answer
**LoadBalancer** = one public IP per service. **Ingress** = single IP, routes based on host/path rules.

### Ingress Example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

| Feature | LoadBalancer | Ingress |
|---|---|---|
| External IP per service | ✅ Yes | ❌ Shared |
| HTTP routing rules | ❌ No | ✅ Yes |
| TLS termination | ❌ Manual | ✅ Built-in |
| Cost efficient | ❌ | ✅ |
| Handles non-HTTP | ✅ TCP/UDP | ❌ HTTP only |

---

# 22. ClusterIP Works but Ingress Fails

## Question
App works with ClusterIP but fails with Ingress. How do you troubleshoot?

## ✅ Answer

1. **Check Ingress Controller is running:** `kubectl get pods -n ingress-nginx`
2. **Check Ingress resource:** `kubectl describe ingress <name>` — rules, paths, service match
3. **Check annotations/class:** `kubernetes.io/ingress.class: nginx` or `ingressClassName: nginx`
4. **Check DNS/host header:** DNS must point to ingress external IP
5. **Check controller logs:** `kubectl logs -n ingress-nginx <controller-pod>`
6. **Check backend endpoints:** `kubectl get endpoints <service-name>`
7. **Check TLS config:** Valid secret, `tls` section present

---

# 23. Ingress Controller vs Ingress

## Question
Why is an Ingress Controller needed after creating an Ingress resource?

## ✅ Answer
The **Ingress resource** is just routing rules. The **Ingress Controller** is the actual proxy that processes them.

| Component | Role |
|---|---|
| Ingress Resource | Specifies rules (host/path/service) |
| Ingress Controller | Watches and implements rules via a proxy |
| Without Controller | Rules are never executed → traffic not routed |

Popular controllers: `nginx-ingress`, `traefik`, `aws-alb-ingress-controller`

> **Key takeaway:** Creating an Ingress resource is like writing a script but never running it.

---

# 24. Custom Ingress Controller

## Question
Can we use an in-house load balancer with Kubernetes Ingress?

## ✅ Answer
Yes, but the LB must **send traffic to a running Ingress Controller** inside the cluster.

```
Client --> Load Balancer --> Ingress Controller --> Services --> Pods
```

The in-house LB **cannot** interpret Kubernetes Ingress YAML directly. Only an Ingress Controller can.

---

# 25. Only One Replica Running

## Question
Deployment has `replicas: 3` but only 1 pod is running. What's wrong?

## ✅ Answer

1. **Check Pod Statuses:** `kubectl get pods -l app=my-app`
2. **Describe Deployment/Pods:** Look for events, errors
3. **Check Node Capacity:** `kubectl describe nodes` — insufficient CPU/memory?
4. **Check Affinity/Taints:** Restrictions preventing scheduling?
5. **Check Pod Crashes:** `kubectl logs <pod-name>` — OOMKilled?

---

# 26. ConfigMap Changes Not Reflected

## Question
Pod mounts a ConfigMap, but changes aren't reflected. Why?

## ✅ Answer

1. **App reads config only at startup** — needs restart
2. **Kubelet delay** — file update sync every 1-2 minutes
3. **Wrong ConfigMap** — editing a different resource
4. **ConfigMap used as env vars** — requires pod restart

### ✅ Fix
```bash
kubectl rollout restart deployment <name>
```

Or add checksum annotation for auto-rollout:
```yaml
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

---

# 27. Node Affinity

## Question
How does Node Affinity work and when to use it?

## ✅ Answer
Constrains which nodes a pod can be scheduled on based on **node labels**.

### Types
1. **requiredDuringScheduling** — Hard rule: pod won't schedule unless met
2. **preferredDuringScheduling** — Soft rule: prefers but can fall back

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

| Use Case | Why |
|---|---|
| GPU workloads | Run only on GPU nodes |
| Zone awareness | Pin to specific zone |
| Storage | Run on SSD-backed nodes |

---

# 28. Node Affinity vs Node Label Selector

## Question
What is the difference between `nodeSelector` and `nodeAffinity`?

## ✅ Answer

### nodeSelector — Simple
```yaml
spec:
  nodeSelector:
    disktype: ssd
```
Only supports **exact match** (key = value).

### nodeAffinity — Advanced
Supports `In`, `NotIn`, `Exists`, `DoesNotExist`. Has **required** and **preferred** rules.

| Feature | `nodeSelector` | `nodeAffinity` |
|---|---|---|
| Operators | Only `=` | `In`, `NotIn`, `Exists`, etc. |
| Rule type | Only required | Required or Preferred |
| Flexibility | Low | High |

---

# 29. Container Runtime

## Question
What is a Container Runtime in Kubernetes?

## ✅ Answer
The software responsible for **running containers** on each node. It pulls images, starts/stops containers, and reports status to the Kubelet.

### 🔗 Architecture
```
Kubelet → CRI → Container Runtime (containerd)
```

### 🎯 Popular Runtimes

| Runtime | Description |
|---|---|
| **containerd** | Lightweight, now default in most K8s distributions |
| **CRI-O** | Purpose-built for Kubernetes. Used by OpenShift. |
| **Docker** | Deprecated in Kubernetes since v1.20+ |

> Kubernetes uses `containerd` or `CRI-O` via the **Container Runtime Interface (CRI)**.

---
