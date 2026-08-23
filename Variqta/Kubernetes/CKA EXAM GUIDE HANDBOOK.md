# ☸️ Certified Kubernetes Administrator (CKA) Master Exam & Production Handbook
> **Comprehensive 60-Topic Deep-Dive, Speed-Optimized CLI Workflows, YAML Blueprints, Control Plane Upgrades, etcd Disaster Recovery & SRE Incident Runbooks.**  
> *Engineered for CKA/CKAD Candidates, Kubernetes Cluster Administrators, DevOps Engineers, and SRE Platform Leads.*

---

## 📑 Master Table of Contents
1. [CKA Exam Strategy, Time Management & Speed Optimizations](#1-cka-exam-strategy-time-management--speed-optimizations)
2. [Cluster Architecture & Control Plane Internals](#2-cluster-architecture--control-plane-internals)
3. [kubectl Speed Mastery, JSONPath & Imperative CLI Techniques](#3-kubectl-speed-mastery-jsonpath--imperative-cli-techniques)
4. [YAML Manifest Generation & Rapid Editing Strategies](#4-yaml-manifest-generation--rapid-editing-strategies)
5. [Cluster Lifecycle: `kubeadm` Upgrades & `etcd` Backup / Restore](#5-cluster-lifecycle-kubeadm-upgrades--etcd-backup--restore)
6. [Node Management: Cordon, Drain, Labels, Taints & Tolerations](#6-node-management-cordon-drain-labels-taints--tolerations)
7. [Namespaces, Contexts & Multi-Cluster `kubeconfig` Switching](#7-namespaces-contexts--multi-cluster-kubeconfig-switching)
8. [Resource Governance: `ResourceQuotas` & `LimitRanges`](#8-resource-governance-resourcequotas--limitranges)
9. [Workload Controllers: Deployments, StatefulSets, DaemonSets & Jobs](#9-workload-controllers-deployments-statefulsets-daemonsets--jobs)
10. [Pod Placement & Advanced Scheduling (Affinity, Topology & Priority)](#10-pod-placement--advanced-scheduling-affinity-topology--priority)
11. [Cluster Networking & Services (ClusterIP, NodePort, Headless & CoreDNS)](#11-cluster-networking--services-clusterip-nodeport-headless--coredns)
12. [Ingress Controllers, L7 Routing, Annotations & TLS Secrets](#12-ingress-controllers-l7-routing-annotations--tls-secrets)
13. [Zero-Trust Network Policies & Pod Microsegmentation](#13-zero-trust-network-policies--pod-microsegmentation)
14. [Storage Subsystem: Volumes, PVs, PVCs & Dynamic StorageClasses](#14-storage-subsystem-volumes-pvs-pvcs--dynamic-storageclasses)
15. [Cluster Security: RBAC, ServiceAccounts & Hardened `securityContext`](#15-cluster-security-rbac-serviceaccounts--hardened-securitycontext)
16. [Pod Security Admission (PSA) & Admission Controllers](#16-pod-security-admission-psa--admission-controllers)
17. [Production Incident Triage & High-Yield Diagnostic Flowcharts](#17-production-incident-triage--high-yield-diagnostic-flowcharts)
18. [High-Frequency CKA Exam Simulation Scenarios & Real-World Fixes](#18-high-frequency-cka-exam-simulation-scenarios--real-world-fixes)
19. [Top 20 Senior Kubernetes Administrator Interview Q&A](#19-top-20-senior-kubernetes-administrator-interview-qa)

---

## 1. CKA Exam Strategy, Time Management & Speed Optimizations

```
                           CKA DOMAIN WEIGHTINGS (2026)
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. Troubleshooting (Cluster, Nodes, Pods, Network, Storage)        : 30%    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. Cluster Architecture, Installation & Configuration (kubeadm/etcd): 25%   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 3. Services & Networking (Services, Ingress, NetworkPolicies, DNS) : 20%    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 4. Workloads & Scheduling (Deployments, StatefulSets, Affinity)    : 15%    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 5. Storage (StorageClasses, PVs, PVCs, VolumeMounts)               : 10%    │
 └─────────────────────────────────────────────────────────────────────────────┘
```

### ⏱️ Time Budgeting Formula (120 Minutes Total)
* **Tier 1 (Easy 1–2 mins):** Imperative Pods, ConfigMaps, Secrets, manual scaling (`kubectl scale`).
* **Tier 2 (Medium 3–5 mins):** Ingress routing, RBAC Roles/Bindings, PVC mounts, NetworkPolicies.
* **Tier 3 (Hard 6–8 mins):** `kubeadm` version upgrades, broken `kubelet` systemd triage, `etcd` TLS snapshot restore.
* **The 4-Minute Rule:** If stuck on a task for $>4$ minutes, add a flag in the UI, move on immediately, and revisit during the final 15-minute review window.

---

## 2. Cluster Architecture & Control Plane Internals

```
+===================================================================================+
|                              KUBERNETES CONTROL PLANE                             |
|                                                                                   |
|  +--------------------+      +---------------------+      +--------------------+  |
|  |   kube-apiserver   | <──> |      etcd v3        |      |   kube-scheduler   |  |
|  | (REST API Gateway) |      | (State Database)    |      | (Placement Engine) |  |
|  +--------------------+      +---------------------+      +--------------------+  |
|            │                                                        │             |
|            ▼                                                        │             |
|  +--------------------+                                             │             |
|  |  kube-controller-  | (Reconciliation Control Loops)              │             |
|  |      manager       |                                             │             |
|  +--------------------+                                             │             |
+===================================================================================+
            │                                                         │
            ├───────────────────────────┬─────────────────────────────┘
            ▼                           ▼
+===========================+===========================+
|       WORKER NODE 1       |       WORKER NODE 2       |
|  +---------------------+  |  +---------------------+  |
|  |       kubelet       |  |  |       kubelet       |  |
|  | (Node Agent Daemon) |  |  | (Node Agent Daemon) |  |
|  +---------------------+  |  +---------------------+  |
|  +---------------------+  |  +---------------------+  |
|  |     kube-proxy      |  |  |     kube-proxy      |  |
|  | (IPVS / iptables)   |  |  | (IPVS / iptables)   |  |
|  +---------------------+  |  +---------------------+  |
|  +---------------------+  |  +---------------------+  |
|  | containerd / CRI    |  |  | containerd / CRI    |  |
|  +---------------------+  |  +---------------------+  |
|    [Pod] [Pod] [Pod]      |    [Pod] [Pod] [Pod]      |
+===========================+===========================+
```

### Component Roles & Failure Impact
* **`kube-apiserver`:** Exposes the REST API, authenticates tokens/certs, executes admission controllers, and communicates with `etcd`. *If down: `kubectl` commands fail; existing pods keep running.*
* **`etcd`:** Consistent distributed B-tree key-value store (Port 2379). *If down: Cluster becomes read-only and no state updates can occur.*
* **`kube-scheduler`:** 2-Stage placement engine: **Filtering** (Predicates) ➔ **Scoring** (Priority weights 1–100).
* **`kubelet`:** Node-level supervisor daemon running via systemd. Talks to CRI (`containerd`) to launch pods and monitors liveness/readiness probes.
* **`kube-proxy`:** Manages IPVS hash tables or iptables NAT rules to direct Service VIP traffic to Pod IPs.

---

## 3. kubectl Speed Mastery, JSONPath & Imperative CLI Techniques

### 🚀 Exam-Ready Environment Setup
```bash
# Add to ~/.bashrc or execute immediately
alias k='kubectl'
alias kgp='kubectl get pods -o wide'
alias kgn='kubectl get nodes -o wide'
alias kgs='kubectl get svc -o wide'
alias kd='kubectl describe'
alias kdel='kubectl delete --grace-period=0 --force'
export do="--dry-run=client -o yaml"
export now="--grace-period=0 --force"

# Enable instant autocompletion
source <(kubectl completion bash)
complete -o default -F __start_kubectl k
```

### 🎯 High-Yield JSONPath Expressions
```bash
# 1. Get all Pod names in a namespace
kubectl get pods -n prod -o jsonpath='{.items[*].metadata.name}'

# 2. Extract Node Names and their Internal IPs
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'

# 3. List all container images across all Deployments
kubectl get deploy -A -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.containers[*].image}{"\n"}{end}'

# 4. Sort Pods by creation timestamp
kubectl get pods -A --sort-by='.metadata.creationTimestamp'
```

---

## 4. YAML Manifest Generation & Rapid Editing Strategies

```bash
# Generate Deployment manifest
kubectl create deploy web-portal --image=nginx:1.25 --replicas=3 $do > deploy.yaml

# Generate ClusterIP Service
kubectl expose deploy web-portal --port=80 --target-port=8080 --name=web-svc $do > svc.yaml

# Generate NodePort Service
kubectl create service nodeport web-portal --tcp=80:8080 --node-port=30080 $do > nodeport.yaml

# Generate ConfigMap from literal
kubectl create configmap app-cfg --from-literal=ENV=prod --from-literal=DB_PORT=5432 $do > cm.yaml

# Generate Secret
kubectl create secret generic db-pass --from-literal=password=SuperPass123 $do > secret.yaml

# Generate Batch Job
kubectl create job db-migrate --image=busybox $do -- sh -c "echo migrating; sleep 5" > job.yaml
```

---

## 5. Cluster Lifecycle: `kubeadm` Upgrades & `etcd` Backup / Restore

```
                       KUBEADM UPGRADE SEQUENCE
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. Upgrade Control Plane Node:                                              │
 │    • `kubectl cordon <cp-node>` && `kubectl drain <cp-node> --ignore-daemonsets`│
 │    • `apt-get update && apt-get install -y kubeadm=1.29.0-00`               │
 │    • `kubeadm upgrade plan` ──▶ `kubeadm upgrade apply v1.29.0`             │
 │    • `apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00`               │
 │    • `systemctl daemon-reload && systemctl restart kubelet`                 │
 │    • `kubectl uncordon <cp-node>`                                           │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. Upgrade Worker Nodes (Sequentially):                                     │
 │    • `kubectl cordon <worker>` && `kubectl drain <worker> --ignore-daemonsets`│
 │    • `apt-get update && apt-get install -y kubeadm=1.29.0-00`               │
 │    • `kubeadm upgrade node`                                                 │
 │    • `apt-get install -y kubelet=1.29.0-00 kubectl=1.29.0-00`               │
 │    • `systemctl daemon-reload && systemctl restart kubelet`                 │
 │    • `kubectl uncordon <worker>`                                            │
 └─────────────────────────────────────────────────────────────────────────────┘
```

### 💾 `etcd` Snapshot Backup and Disaster Recovery
```bash
# 1. Take etcd Snapshot
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/etcd-backup.db

# 2. Verify Snapshot
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/etcd-backup.db

# 3. Restore to New Data Directory
ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-restored \
  snapshot restore /opt/etcd-backup.db

# 4. Update Static Pod Manifest (/etc/kubernetes/manifests/etcd.yaml)
# Change hostPath.path for 'etcd-data' volume from /var/lib/etcd to /var/lib/etcd-restored
```

---

## 6. Node Management: Cordon, Drain, Labels, Taints & Tolerations

```bash
# Labeling
kubectl label node worker-1 disktype=ssd tier=frontend --overwrite

# Node Maintenance
kubectl cordon worker-1                                        # Prevent new scheduling
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data --force # Evict pods
kubectl uncordon worker-1                                      # Mark active again
```

### Taints and Tolerations
* **`NoSchedule`:** Prevents new pods from placing on the node unless toleration matches.
* **`PreferNoSchedule`:** Soft constraint; scheduler avoids if possible.
* **`NoExecute`:** Evicts existing running pods immediately if they lack matching toleration.

```bash
# Add taint
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule

# Remove taint (trailing minus)
kubectl taint nodes worker-1 dedicated=gpu:NoSchedule-
```

```yaml
# Pod Toleration
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
```

---

## 7. Namespaces, Contexts & Multi-Cluster `kubeconfig` Switching

```bash
# Switch default namespace for active context
kubectl config set-context --current --namespace=production

# Multi-Cluster Context Management
kubectl config get-contexts
kubectl config use-context k8s-prod-cluster

# View active minified configuration
kubectl config view --minify --flatten
```

---

## 8. Resource Governance: `ResourceQuotas` & `LimitRanges`

```yaml
# ResourceQuota: Caps aggregate usage for the entire namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
---
# LimitRange: Injects default requests/limits into containers lacking them
apiVersion: v1
kind: LimitRange
metadata:
  name: container-defaults
  namespace: dev
spec:
  limits:
    - type: Container
      default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 250m
        memory: 256Mi
      max:
        cpu: 2000m
        memory: 2Gi
      min:
        cpu: 100m
        memory: 64Mi
```

---

## 9. Workload Controllers: Deployments, StatefulSets, DaemonSets & Jobs

```
 ┌───────────────┬─────────────────────────────────────────────────────────────┐
 │ Controller    │ Primary Production Purpose & Scheduling Characteristic      │
 ├───────────────┼─────────────────────────────────────────────────────────────┤
 │ Deployment    │ Stateless workloads, replica scaling, zero-downtime rollouts│
 │ StatefulSet   │ Stable network IDs (db-0, db-1), dedicated volume templates │
 │ DaemonSet     │ Exactly 1 Pod per eligible node (CNI, Fluent Bit, Prom Node)│
 │ Job / CronJob │ Batch run-to-completion tasks (`ttlSecondsAfterFinished`)   │
 └───────────────┴─────────────────────────────────────────────────────────────┘
```

### StatefulSet Blueprint (`statefulset.yaml`)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: "redis-headless"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7.0
          ports:
            - containerPort: 6379
              name: redis
          volumeMounts:
            - name: redis-data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: redis-data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "gp3"
        resources:
          requests:
            storage: 10Gi
```

---

## 10. Pod Placement & Advanced Scheduling (Affinity, Topology & Priority)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-service
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values: ["us-east-1a", "us-east-1b"]
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values: ["api-service"]
            topologyKey: "kubernetes.io/hostname"
  containers:
    - name: api
      image: api:v2
```

---

## 11. Cluster Networking & Services (ClusterIP, NodePort, Headless & CoreDNS)

```
                            KUBERNETES SERVICE ROUTING
   [Client Query: api-svc.prod.svc.cluster.local] ──▶ Resolves to CoreDNS ClusterIP
                                                               │
                                                               ▼
   [Service: ClusterIP VIP 10.96.0.50]
        │
        ├─────── kube-proxy (iptables DNAT / IPVS Hash Table)
        ▼
   [Endpoints / EndpointSlice] (10.244.1.15:8080, 10.244.2.22:8080)
        │
        ├───────────────────────────────┐
        ▼                               ▼
   [Pod 1 (10.244.1.15)]           [Pod 2 (10.244.2.22)]
```

### CoreDNS Standard Record Format
$$\text{Query} = \text{service-name}.\text{namespace}.\text{svc}.\text{cluster.local}$$
* Example: `payment-db.default.svc.cluster.local`

---

## 12. Ingress Controllers, L7 Routing, Annotations & TLS Secrets

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - portal.enterprise.com
      secretName: portal-tls-secret
  rules:
    - host: portal.enterprise.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-frontend
                port:
                  number: 80
```

---

## 13. Zero-Trust Network Policies & Pod Microsegmentation

```yaml
# 1. Default Deny All Ingress and Egress in Namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: finance
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
# 2. Allow Ingress on DB port 5432 ONLY from App Pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-db
  namespace: finance
spec:
  podSelector:
    matchLabels:
      role: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: backend
      ports:
        - protocol: TCP
          port: 5432
```

---

## 14. Storage Subsystem: Volumes, PVs, PVCs & Dynamic StorageClasses

```yaml
# Dynamic StorageClass (AWS EBS CSI Provisioner)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer  # Delays binding until Pod is scheduled
allowVolumeExpansion: true
reclaimPolicy: Retain
---
# PersistentVolumeClaim referencing StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-gp3-sc
  resources:
    requests:
      storage: 50Gi
```

---

## 15. Cluster Security: RBAC, ServiceAccounts & Hardened `securityContext`

```yaml
# 1. Dedicated ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ci-deployer
  namespace: staging
---
# 2. Granular Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: staging
rules:
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
# 3. RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-ci-deployer
  namespace: staging
subjects:
  - kind: ServiceAccount
    name: ci-deployer
    namespace: staging
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Verify RBAC with kubectl auth can-i
kubectl auth can-i create deployments --as=system:serviceaccount:staging:ci-deployer -n staging
# Returns: yes
```

---

## 16. Pod Security Admission (PSA) & Admission Controllers

```bash
# Apply Restricted Pod Security Standard to namespace
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=baseline
```

```yaml
# Hardened PodSpec compliant with PSA Restricted
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: app:v1
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
```

---

## 17. Production Incident Triage & High-Yield Diagnostic Flowcharts

```
                             [Pod In Broken / Unhealthy State]
                                             │
                             kubectl get pods -o wide
                                             │
                 ┌───────────────────────────┴───────────────────────────┐
                 ▼                                                       ▼
         [Status: Pending]                                  [Status: CrashLoopBackOff]
                 │                                                       │
       kubectl describe pod                                    kubectl logs <pod> --previous
                 │                                                       │
   ┌─────────────┴─────────────┐                           ┌─────────────┴─────────────┐
   ▼                           ▼                           ▼                           ▼
[Scheduler Filter Failure] [PVC Unbound]            [Application Code Error]    [OOMKilled Error]
- NodeSelector mismatch    - Missing StorageClass   - Bad startup arguments     - Exit Code 137
- Taint without toleration - Storage capacity       - Missing DB connection     - Memory limit too low
- Insufficient CPU/Memory  - WaitForFirstConsumer   - Missing ConfigMap/Secret  - Increase memory limit
```

### Worker Node `NotReady` Investigation Runbook
```bash
# 1. Check Node Status & Conditions
kubectl describe node worker-1

# 2. SSH into Node & Check kubelet service
sudo systemctl status kubelet
sudo journalctl -u kubelet -n 100 --no-pager

# 3. Check Container Runtime (containerd)
sudo systemctl status containerd
sudo crictl ps -a
sudo crictl info

# 4. Check Disk Space and Inodes
df -h
df -i
```

---

## 18. High-Frequency CKA Exam Simulation Scenarios & Real-World Fixes

#### Scenario 1: Service Has No Active Endpoints (`<none>`)
* **Forensics:** Run `kubectl get ep <svc-name>` and `kubectl get pods --show-labels`.
* **Fix:** Correct the typo in `spec.selector` of the Service manifest to match the Pod `metadata.labels`.

#### Scenario 2: PVC Stuck in `Pending` Indefinitely
* **Forensics:** Run `kubectl describe pvc <pvc-name>`.
* **Cause 1:** StorageClass has `volumeBindingMode: WaitForFirstConsumer`. *(Behavior is normal; PVC binds once a Pod referencing it is scheduled).*
* **Cause 2:** Requested storage size exceeds available PV capacity or `accessModes` mismatch (`ReadWriteOnce` vs `ReadWriteMany`).

#### Scenario 3: Pod Stuck in `ImagePullBackOff` / `ErrImagePull`
* **Forensics:** Run `kubectl describe pod <pod-name>` and check the **Events** section.
* **Fix:** Fix image tag typo or create and link an `imagePullSecrets` Docker registry secret.

#### Scenario 4: Pod Stuck in `ContainerCreating`
* **Forensics:** Run `kubectl describe pod <pod-name>`.
* **Fix:** Look for missing ConfigMaps, Secrets, or unattached CSI volumes.

---

## 19. Top 20 Senior Kubernetes Administrator Interview Q&A

| # | High-Yield CKA / Senior Interview Question | Expert Platform Lead Model Answer |
|---|---|---|
| 1 | **What is the exact sequence executed by `kube-scheduler`?** | *The scheduler runs a 2-stage pipeline: **1. Filtering (Predicates):** Filters out nodes that do not satisfy hard requirements (resource requests, nodeSelectors, hard nodeAffinity, taints). **2. Scoring (Priorities):** Ranks remaining candidate nodes with weights from 1 to 100 based on soft constraints (least requested resources, topology spread, soft anti-affinity), selecting the node with the highest aggregate score.* |
| 2 | **How do you safely backup and restore `etcd` in a production cluster?** | *Use `etcdctl snapshot save <path>` providing the CA certificate (`--cacert`), server certificate (`--cert`), and server private key (`--key`). To restore, execute `etcdctl snapshot restore <path> --data-dir=<new-dir>`, and update the static pod manifest `/etc/kubernetes/manifests/etcd.yaml` to point the `etcd-data` hostPath volume to the newly restored directory.* |
| 3 | **What is the difference between `WaitForFirstConsumer` and `Immediate` volume binding?** | *`Immediate` allocates the storage volume as soon as the PVC is created without knowing which worker node will host the pod, potentially binding to an EBS volume in AZ `us-east-1a` while the pod gets scheduled in AZ `us-east-1b`. `WaitForFirstConsumer` delays volume provisioning until the pod is actually scheduled, guaranteeing storage placement in the matching Availability Zone.* |
| 4 | **How does `kubelet` determine if a container is dead vs. unready?** | *• **Liveness Probe:** Detects deadlocks and unrecoverable hangs. If it fails, kubelet kills the container and restarts it according to `restartPolicy`.<br>• **Readiness Probe:** Detects if the container is ready to accept incoming user traffic. If it fails, kubelet removes the Pod IP from the Service's active Endpoints without restarting the container.* |
| 5 | **What happens under the hood when a Node enters `NotReady` state?** | *The node's `kubelet` fails to send heartbeats (NodeStatus lease) to `kube-apiserver` within `node-monitor-grace-period` (default 40s). The NodeLifecycleController marks the node `NotReady` and, after `pod-eviction-timeout` (default 5m), begins scheduling replacement pods on healthy nodes.* |

---
*Created for CKA Exam Preparation, Practical Cluster Operations & Senior Kubernetes Engineering.*
