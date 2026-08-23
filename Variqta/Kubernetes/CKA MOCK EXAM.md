# ☸️ Certified Kubernetes Administrator (CKA) Master Mock Exam
> **Full-Length 20-Scenario Performance-Based Exam Simulation, Speed CLI Workflows, Complete Solutions & Verification Playbooks.**  
> *Targeted for CKA Candidates, Kubernetes Administrators, SREs, and Platform Engineers.*

---

## 📑 Table of Contents
1. [CKA Exam Blueprint, Time Strategy & Speed Setup](#1-cka-exam-blueprint-time-strategy--speed-setup)
2. [Exam Terminal Setup & Speed Aliases](#2-exam-terminal-setup--speed-aliases)
3. [Full-Length 20-Scenario CKA Mock Exam](#3-full-length-20-scenario-cka-mock-exam)
   - [Task 1: RBAC - ServiceAccount, Role & RoleBinding (4%)](#task-1-rbac---serviceaccount-role--rolebinding-4)
   - [Task 2: Control Plane & Worker Node Kubeadm Upgrade (8%)](#task-2-control-plane--worker-node-kubeadm-upgrade-8)
   - [Task 3: etcd Snapshot Backup & Disaster Recovery Restore (7%)](#task-3-etcd-snapshot-backup--disaster-recovery-restore-7)
   - [Task 4: Multi-Container Pod with Shared Volume & Sidecar (4%)](#task-4-multi-container-pod-with-shared-volume--sidecar-4)
   - [Task 5: Workload Scheduling with NodeAffinity & PodAntiAffinity (5%)](#task-5-workload-scheduling-with-nodeaffinity--podantiaffinity-5)
   - [Task 6: Node Maintenance, Cordon, Drain & Taints/Tolerations (4%)](#task-6-node-maintenance-cordon-drain--taintstolerations-4)
   - [Task 7: Persistent Volumes, Dynamic StorageClass & PVC Binding (4%)](#task-7-persistent-volumes-dynamic-storageclass--pvc-binding-4)
   - [Task 8: PVC Storage Expansion & Non-Root Pod Security Context (4%)](#task-8-pvc-storage-expansion--non-root-pod-security-context-4)
   - [Task 9: Ingress Routing with Path Rules & TLS Secret (6%)](#task-9-ingress-routing-with-path-rules--tls-secret-6)
   - [Task 10: Zero-Trust NetworkPolicy Whitelisting (6%)](#task-10-zero-trust-networkpolicy-whitelisting-6)
   - [Task 11: ClusterIP, NodePort Services & Endpoint Debugging (5%)](#task-11-clusterip-nodeport-services--endpoint-debugging-5)
   - [Task 12: CoreDNS Custom Upstream Forwarding & Service Discovery (4%)](#task-12-coredns-custom-upstream-forwarding--service-discovery-4)
   - [Task 13: Node NotReady Triage - Kubelet & Container Runtime Failure (7%)](#task-13-node-notready-triage---kubelet--container-runtime-failure-7)
   - [Task 14: Control Plane Triage - Broken Static Pod Manifest (7%)](#task-14-control-plane-triage---broken-static-pod-manifest-7)
   - [Task 15: Pod CrashLoopBackOff & OOMKilled (Exit 137) Triage (6%)](#task-15-pod-crashloopbackoff--oomkilled-exit-137-triage-6)
   - [Task 16: StatefulSet with Headless Service & PVC Templates (4%)](#task-16-statefulset-with-headless-service--pvc-templates-4)
   - [Task 17: Deployment Rolling Update, Pause, Resume & Rollback (4%)](#task-17-deployment-rolling-update-pause-resume--rollback-4)
   - [Task 18: Static Pod Deployment on Specific Worker Node (4%)](#task-18-static-pod-deployment-on-specific-worker-node-4)
   - [Task 19: JSONPath Extraction & Resource Sorting (4%)](#task-19-jsonpath-extraction--resource-sorting-4)
   - [Task 20: Pod Security Admission (PSA) & Namespace Quotas (3%)](#task-20-pod-security-admission-psa--namespace-quotas-3)
4. [Master Scoring Matrix & Evaluation Rubric](#4-master-scoring-matrix--evaluation-rubric)
5. [Top 10 High-Yield Exam Pitfalls & Prevention Strategies](#5-top-10-high-yield-exam-pitfalls--prevention-strategies)

---

## 1. CKA Exam Blueprint, Time Strategy & Speed Setup

### Official Exam Domain Breakdown
```
  ┌────────────────────────────────────────────────────────────────────────┐
  │                   CKA EXAM DOMAIN DISTRIBUTION                         │
  ├──────────────────────────────────────────────────────┬─────────────────┤
  │ Troubleshooting & Forensics                          │ 30% (Highest)   │
  │ Cluster Architecture, Installation & Configuration   │ 25%             │
  │ Services & Networking                                │ 20%             │
  │ Workloads & Scheduling                               │ 15%             │
  │ Storage Subsystem                                    │ 10%             │
  └──────────────────────────────────────────────────────┴─────────────────┘
```

### The 4-Minute Rule & Exam Time Management
* **Total Time:** 120 Minutes (2 Hours)
* **Total Questions:** 17 to 20 Performance-Based Scenarios
* **Passing Score:** 66%
* **Time Budget:** Allocate **4 to 6 minutes** per standard task. Allocate **8 to 10 minutes** for high-weight tasks (`kubeadm upgrade`, `etcd restore`, node troubleshooting).
* **Context Switching:** Always execute the context switch command shown at the very top of each question (`kubectl config use-context <name>`). Failure to switch clusters means 0 points!

---

## 2. Exam Terminal Setup & Speed Aliases

Execute these lines immediately upon starting the exam session:

```bash
# 1. Shell Aliases & Auto-completion
alias k=kubectl
alias kgp="kubectl get pods -o wide"
alias kgn="kubectl get nodes -o wide"
alias kgs="kubectl get svc -o wide"
alias kga="kubectl get all -A"
alias kd="kubectl describe"

# 2. Fast Dry-Run YAML Generation Aliases
export do="--dry-run=client -o yaml"
export now="--grace-period=0 --force"

# 3. Enable bash completion for alias k
source <(kubectl completion bash)
complete -o default -F __start_kubectl k

# 4. Vim Ergonomics (~/.vimrc)
cat <<EOF > ~/.vimrc
set tabstop=2
set shiftwidth=2
set expandtab
set number
set cursorline
syntax on
EOF
```

---

## 3. Full-Length 20-Scenario CKA Mock Exam

---

### Task 1: RBAC - ServiceAccount, Role & RoleBinding (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. In the `finance` namespace, create a new ServiceAccount named `fin-processor`.
2. Create a Role named `fin-reader` that allows `get`, `list`, and `watch` actions on `pods` and `pods/log` in the `finance` namespace.
3. Bind the Role `fin-reader` to the ServiceAccount `fin-processor` using a RoleBinding named `fin-reader-binding`.
4. Verify that the ServiceAccount can list pods in `finance` but **cannot** list secrets or pods in `default`.

#### Complete Solution
```bash
# 1. Ensure namespace exists & create ServiceAccount
kubectl create namespace finance --dry-run=client -o yaml | kubectl apply -f -
kubectl create sa fin-processor -n finance

# 2. Create Role imperatively
kubectl create role fin-reader \
  --namespace=finance \
  --verb=get,list,watch \
  --resource=pods,pods/log

# 3. Create RoleBinding imperatively
kubectl create rolebinding fin-reader-binding \
  --namespace=finance \
  --role=fin-reader \
  --serviceaccount=finance:fin-processor

# 4. Verify RBAC permissions
kubectl auth can-i list pods -n finance --as=system:serviceaccount:finance:fin-processor
# Expected: yes

kubectl auth can-i get pods/log -n finance --as=system:serviceaccount:finance:fin-processor
# Expected: yes

kubectl auth can-i get secrets -n finance --as=system:serviceaccount:finance:fin-processor
# Expected: no

kubectl auth can-i list pods -n default --as=system:serviceaccount:finance:fin-processor
# Expected: no
```

---

### Task 2: Control Plane & Worker Node Kubeadm Upgrade (8%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
Upgrade the Control Plane node `controlplane` and worker node `node01` from version `v1.30.0` to `v1.30.1`.
- Ensure all workloads are safely drained from `node01` before upgrading.
- Uncordon `node01` after the upgrade completes.

#### Complete Solution
```bash
# ==============================================================================
# PART A: UPGRADE CONTROL PLANE NODE
# ==============================================================================
# 1. Upgrade kubeadm on controlplane
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.30.1-1.1
sudo apt-mark hold kubeadm

# 2. Verify and apply cluster upgrade plan
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.30.1 -y

# 3. Drain control plane node (if not already cordoned)
kubectl drain controlplane --ignore-daemonsets --delete-emptydir-data

# 4. Upgrade kubelet and kubectl on controlplane
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.30.1-1.1 kubectl=1.30.1-1.1
sudo apt-mark hold kubelet kubectl

# 5. Restart kubelet daemon
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon controlplane

# ==============================================================================
# PART B: UPGRADE WORKER NODE (node01)
# ==============================================================================
# 1. From controlplane, drain worker node
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data --force

# 2. SSH into node01
ssh node01

# 3. Upgrade kubeadm on node01
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.30.1-1.1
sudo apt-mark hold kubeadm

# 4. Execute node upgrade
sudo kubeadm upgrade node

# 5. Upgrade kubelet and kubectl on node01
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.30.1-1.1 kubectl=1.30.1-1.1
sudo apt-mark hold kubelet kubectl

# 6. Restart kubelet on node01
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit

# 7. From controlplane, uncordon worker node and verify version
kubectl uncordon node01
kubectl get nodes -o wide
# Verify all nodes show Ready with v1.30.1
```

---

### Task 3: etcd Snapshot Backup & Disaster Recovery Restore (7%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Take a snapshot backup of the current running `etcd` instance and save it to `/var/lib/backup/etcd-snapshot.db`.
2. Inspect the static pod manifest `/etc/kubernetes/manifests/etcd.yaml` to identify the PKI certificates.
3. Restore the snapshot into a new data directory `/var/lib/etcd-restored`.
4. Update the `etcd.yaml` static pod manifest so that `etcd` starts with the restored dataset.

#### Complete Solution
```bash
# 1. Extract certificates and endpoints from etcd manifest
cat /etc/kubernetes/manifests/etcd.yaml | grep -E "cert-file|key-file|trusted-ca-file|endpoints"

# 2. Take Snapshot Backup
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /var/lib/backup/etcd-snapshot.db

# 3. Verify Snapshot Integrity
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /var/lib/backup/etcd-snapshot.db

# 4. Perform Snapshot Restore to new data directory
ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-restored snapshot restore /var/lib/backup/etcd-snapshot.db

# 5. Modify Static Pod Manifest /etc/kubernetes/manifests/etcd.yaml
# Edit the hostPath volume 'etcd-data' to point to /var/lib/etcd-restored:
```

```yaml
# In /etc/kubernetes/manifests/etcd.yaml:
spec:
  volumes:
  - name: etcd-data
    hostPath:
      path: /var/lib/etcd-restored    # <-- Update this line
      type: DirectoryOrCreate
```

```bash
# 6. Verify etcd and API Server recover cleanly
kubectl get pods -n kube-system
kubectl get nodes
```

---

### Task 4: Multi-Container Pod with Shared Volume & Sidecar (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
Create a multi-container Pod named `app-logger` in namespace `production`:
- **App Container:** Name `app-writer`, Image `busybox:1.36`, Command writes the current timestamp every 5 seconds into `/var/log/app.log`.
- **Sidecar Container:** Name `log-streamer`, Image `busybox:1.36`, Command runs `tail -f /var/log/app.log`.
- **Volume:** Use an `emptyDir` volume named `shared-logs` mounted at `/var/log` in both containers.

#### Complete Solution
```bash
# 1. Generate YAML template
k run app-logger -n production --image=busybox:1.36 $do > app-logger.yaml
```

```yaml
# app-logger.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-logger
  namespace: production
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}
  containers:
  - name: app-writer
    image: busybox:1.36
    command: ["/bin/sh", "-c", "while true; do date >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  - name: log-streamer
    image: busybox:1.36
    command: ["/bin/sh", "-c", "tail -f /var/log/app.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

```bash
# 2. Apply and Verify
kubectl apply -f app-logger.yaml
kubectl logs app-logger -c log-streamer -n production
# Verify live timestamps stream out
```

---

### Task 5: Workload Scheduling with NodeAffinity & PodAntiAffinity (5%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Label node `node01` with `environment=production` and `tier=frontend`.
2. Create a Deployment named `web-prod` in namespace `default`:
   - Replicas: `3`
   - Image: `nginx:1.25-alpine`
   - **Hard NodeAffinity:** Must run on nodes with `environment=production`.
   - **Soft PodAntiAffinity:** Prefer not to co-locate replicas on nodes with label `app=web-prod` on `topologyKey: kubernetes.io/hostname` (Weight: 100).

#### Complete Solution
```bash
# 1. Label Node
kubectl label node node01 environment=production tier=frontend --overwrite

# 2. Create Deployment manifest
k create deploy web-prod --image=nginx:1.25-alpine --replicas=3 $do > web-prod.yaml
```

```yaml
# web-prod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-prod
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-prod
  template:
    metadata:
      labels:
        app: web-prod
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - production
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web-prod
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx:1.25-alpine
        ports:
        - containerPort: 80
```

```bash
# 3. Apply and Verify Scheduling
kubectl apply -f web-prod.yaml
kubectl get pods -o wide -l app=web-prod
```

---

### Task 6: Node Maintenance, Cordon, Drain & Taints/Tolerations (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Apply a taint to `node02`: Key `workload`, Value `dedicated`, Effect `NoSchedule`.
2. Create a Pod named `priority-worker` in namespace `default`:
   - Image: `nginx:alpine`
   - Add a toleration so this pod can schedule on `node02`.
3. Cordon `node02` temporarily, verify no new pods schedule, then uncordon `node02`.

#### Complete Solution
```bash
# 1. Taint node02
kubectl taint nodes node02 workload=dedicated:NoSchedule

# 2. Generate pod manifest with toleration
k run priority-worker --image=nginx:alpine $do > priority-worker.yaml
```

```yaml
# priority-worker.yaml
apiVersion: v1
kind: Pod
metadata:
  name: priority-worker
  namespace: default
spec:
  tolerations:
  - key: "workload"
    operator: "Equal"
    value: "dedicated"
    effect: "NoSchedule"
  nodeName: node02   # Or allow scheduler via matching toleration
  containers:
  - name: priority-worker
    image: nginx:alpine
```

```bash
# 3. Apply and test cordon/uncordon
kubectl apply -f priority-worker.yaml
kubectl cordon node02
kubectl get nodes
# node02 should show 'Ready,SchedulingDisabled'

kubectl uncordon node02
```

---

### Task 7: Persistent Volumes, Dynamic StorageClass & PVC Binding (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Create a StorageClass named `local-fast` with `provisioner: kubernetes.io/no-provisioner` and `volumeBindingMode: WaitForFirstConsumer`.
2. Create a PersistentVolume named `pv-analytics`:
   - Capacity: `2Gi`, AccessMode: `ReadWriteOnce`, StorageClass: `local-fast`
   - `hostPath`: `/mnt/analytics-data`
3. Create a PersistentVolumeClaim named `pvc-analytics` in namespace `analytics`:
   - Request: `2Gi`, AccessMode: `ReadWriteOnce`, StorageClass: `local-fast`
4. Mount `pvc-analytics` into a Pod `analytics-processor` (Image: `busybox:1.36`) at `/data`.

#### Complete Solution
```yaml
# storage-setup.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-fast
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-fast
  hostPath:
    path: /mnt/analytics-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-analytics
  namespace: analytics
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-fast
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: analytics-processor
  namespace: analytics
spec:
  volumes:
  - name: data-vol
    persistentVolumeClaim:
      claimName: pvc-analytics
  containers:
  - name: processor
    image: busybox:1.36
    command: ["sleep", "3600"]
    volumeMounts:
    - name: data-vol
      mountPath: /data
```

```bash
# Apply and verify binding upon Pod scheduling
kubectl create ns analytics $do | kubectl apply -f -
kubectl apply -f storage-setup.yaml
kubectl get pvc -n analytics
# Status should switch from Pending to Bound once the pod is scheduled!
```

---

### Task 8: PVC Storage Expansion & Non-Root Pod Security Context (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. In namespace `security-lab`, create a Pod named `secure-vault` running `busybox:1.36` (`command: ["sleep", "3600"]`).
2. Configure `securityContext` on the Pod:
   - `runAsUser: 10001`
   - `runAsGroup: 10001`
   - `fsGroup: 20001`
3. Verify that files written by the pod have UID `10001` and GID `20001`.

#### Complete Solution
```yaml
# secure-vault.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-vault
  namespace: security-lab
spec:
  securityContext:
    runAsUser: 10001
    runAsGroup: 10001
    fsGroup: 20001
  volumes:
  - name: scratch-space
    emptyDir: {}
  containers:
  - name: vault
    image: busybox:1.36
    command: ["sleep", "3600"]
    volumeMounts:
    - name: scratch-space
      mountPath: /vault-data
```

```bash
# Verify UID/GID inside container
kubectl create ns security-lab $do | kubectl apply -f -
kubectl apply -f secure-vault.yaml
kubectl exec -it secure-vault -n security-lab -- id
# Output: uid=10001 gid=10001 groups=20001
```

---

### Task 9: Ingress Routing with Path Rules & TLS Secret (6%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Create a TLS Secret named `tls-prod-cert` in namespace `ecommerce` using key `/etc/certs/tls.key` and cert `/etc/certs/tls.crt`.
2. Create an Ingress resource named `ecommerce-ingress` in namespace `ecommerce`:
   - `ingressClassName: nginx`
   - Host: `shop.example.com`
   - Path `/orders` $\rightarrow$ Service `orders-svc` on port `8080` (PathType: `Prefix`)
   - Path `/catalog` $\rightarrow$ Service `catalog-svc` on port `9090` (PathType: `Prefix`)
   - Enable TLS referencing secret `tls-prod-cert`.

#### Complete Solution
```bash
# 1. Create TLS Secret
kubectl create secret tls tls-prod-cert \
  --cert=/etc/certs/tls.crt \
  --key=/etc/certs/tls.key \
  -n ecommerce
```

```yaml
# ecommerce-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - shop.example.com
    secretName: tls-prod-cert
  rules:
  - host: shop.example.com
    http:
      paths:
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: orders-svc
            port:
              number: 8080
      - path: /catalog
        pathType: Prefix
        backend:
          service:
            name: catalog-svc
            port:
              number: 9090
```

```bash
# Apply and verify
kubectl apply -f ecommerce-ingress.yaml
kubectl describe ingress ecommerce-ingress -n ecommerce
```

---

### Task 10: Zero-Trust NetworkPolicy Whitelisting (6%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
In namespace `backend-api`:
1. Lock down all Pods with label `app=database` to **Default Deny** for Ingress.
2. Allow incoming TCP connections on port `5432` **only** from Pods labeled `app=api-server` in the same namespace.
3. Deny all other inbound network traffic.

#### Complete Solution
```yaml
# db-netpol.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: backend-api
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
          app: api-server
    ports:
    - protocol: TCP
      port: 5432
```

```bash
# Apply and Verify
kubectl apply -f db-netpol.yaml
kubectl describe netpol allow-api-to-db -n backend-api
```

---

### Task 11: ClusterIP, NodePort Services & Endpoint Debugging (5%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
A Service named `frontend-svc` in namespace `web` has no endpoints and clients receive Connection Refused.
1. Diagnose why `frontend-svc` is not routing traffic to the backing Pods.
2. Fix the selector mismatch and expose the service externally as a `NodePort` on port `31080`.

#### Complete Solution
```bash
# 1. Investigate Service and backing Pod labels
kubectl get svc frontend-svc -n web -o yaml
kubectl get pods -n web --show-labels

# 2. Identify the label discrepancy (e.g. Service selector has 'app: front-end' while Pod has 'app: frontend')
# 3. Patch or update Service manifest:
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: web
spec:
  type: NodePort
  selector:
    app: frontend    # Fixed matching label
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31080
```

```bash
# 4. Verify endpoints are populated
kubectl get endpoints frontend-svc -n web
# Endpoints should now list Pod IPs!
```

---

### Task 12: CoreDNS Custom Upstream Forwarding & Service Discovery (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
Configure CoreDNS in namespace `kube-system` to forward all queries for domain `corp.internal` to upstream nameserver `10.200.0.10:53`.

#### Complete Solution
```bash
# 1. Edit the coredns ConfigMap
kubectl edit configmap coredns -n kube-system
```

```yaml
# Add this block inside the Corefile:
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
    corp.internal:53 {
        forward . 10.200.0.10
        cache 30
    }
```

```bash
# 2. Restart CoreDNS Deployment to pick up changes
kubectl rollout restart deployment coredns -n kube-system
```

---

### Task 13: Node NotReady Triage - Kubelet & Container Runtime Failure (7%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
Worker node `node02` is reporting `NotReady` status.
1. SSH into `node02` and diagnose why `kubelet` is failing.
2. Fix the underlying configuration or service issue and return `node02` to `Ready` status.

#### Complete Solution
```bash
# 1. Check Node conditions
kubectl describe node node02

# 2. SSH into node02
ssh node02

# 3. Check Kubelet Service status & journal logs
sudo systemctl status kubelet
sudo journalctl -u kubelet -n 50 --no-pager

# Common Scenario A: Kubelet stopped
sudo systemctl enable --now kubelet

# Common Scenario B: Container Runtime (containerd) is down
sudo systemctl restart containerd
sudo systemctl restart kubelet

# Common Scenario C: Misconfigured --container-runtime-endpoint in /var/lib/kubelet/kubeadm-flags.env
# Verify containerd socket exists at unix:///run/containerd/containerd.sock

exit
# 4. Verify from control plane
kubectl get nodes
# node02 should transition to Ready within 10-15 seconds!
```

---

### Task 14: Control Plane Triage - Broken Static Pod Manifest (7%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
`kubectl` commands fail with `The connection to the server 127.0.0.1:6443 was refused`.
1. Inspect the control plane static pod manifests in `/etc/kubernetes/manifests/`.
2. Fix the typo/misconfiguration in `kube-apiserver.yaml` and restore cluster functionality.

#### Complete Solution
```bash
# 1. Check container runtime to see if kube-apiserver container is crashing
sudo crictl ps -a | grep apiserver
sudo crictl logs <container-id>

# 2. Inspect /etc/kubernetes/manifests/kube-apiserver.yaml
# Look for common typos:
# - Wrong cert path (e.g. /etc/kubernetes/pki/apiserver-error.crt)
# - Misspelled flags (e.g. --etcd-servers instead of --etcd-servers=https://...)
# - Indentation errors in YAML

# 3. Fix the manifest file
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml

# 4. Kubelet automatically detects changes in /etc/kubernetes/manifests/ and restarts static pods
sleep 10
kubectl get nodes
```

---

### Task 15: Pod CrashLoopBackOff & OOMKilled (Exit 137) Triage (6%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
A Pod named `data-aggregator` in namespace `analytics` is trapped in `CrashLoopBackOff` and terminating with `Exit Code 137`.
1. Identify the root cause.
2. Update the Pod's memory limit to `512Mi` so it can stabilize.

#### Complete Solution
```bash
# 1. Inspect Pod status and exit code
kubectl describe pod data-aggregator -n analytics
# Look for: Last State: Terminated | Reason: OOMKilled | Exit Code: 137

# 2. Extract YAML, increase memory limit, and replace
kubectl get pod data-aggregator -n analytics -o yaml > data-aggregator.yaml

# Edit data-aggregator.yaml to set:
# resources:
#   requests:
#     memory: "256Mi"
#   limits:
#     memory: "512Mi"

kubectl replace --force -f data-aggregator.yaml
kubectl get pod data-aggregator -n analytics
# Verify pod enters Running state
```

---

### Task 16: StatefulSet with Headless Service & PVC Templates (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
Create a StatefulSet named `redis-cluster` in namespace `database`:
- Replicas: `3`, Image: `redis:7.0-alpine`
- Headless Service named `redis-headless` (`clusterIP: None`, port: `6379`).
- `volumeClaimTemplates` requesting `1Gi` storage with StorageClass `local-fast` mounted at `/data`.

#### Complete Solution
```yaml
# redis-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
  namespace: database
spec:
  clusterIP: None
  selector:
    app: redis-cluster
  ports:
  - port: 6379
    name: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: database
spec:
  serviceName: "redis-headless"
  replicas: 3
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:7.0-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-fast"
      resources:
        requests:
          storage: 1Gi
```

```bash
# Apply and verify sequential pod creation (redis-cluster-0 -> redis-cluster-1 -> redis-cluster-2)
kubectl create ns database $do | kubectl apply -f -
kubectl apply -f redis-statefulset.yaml
kubectl get statefulset,pods,pvc -n database
```

---

### Task 17: Deployment Rolling Update, Pause, Resume & Rollback (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Create a Deployment named `api-server` with `3` replicas running `nginx:1.24` in namespace `apps`.
2. Update the image to `nginx:1.25` and record the change.
3. Pause the rollout midway, verify status, then resume the rollout.
4. Roll back to the previous revision (`nginx:1.24`).

#### Complete Solution
```bash
# 1. Create Deployment
kubectl create deployment api-server --image=nginx:1.24 --replicas=3 -n apps

# 2. Update Image
kubectl set image deployment/api-server nginx=nginx:1.25 -n apps

# 3. Pause and Resume Rollout
kubectl rollout pause deployment/api-server -n apps
kubectl rollout status deployment/api-server -n apps
kubectl rollout resume deployment/api-server -n apps

# 4. Rollback to previous revision
kubectl rollout undo deployment/api-server -n apps
kubectl rollout history deployment/api-server -n apps
```

---

### Task 18: Static Pod Deployment on Specific Worker Node (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. SSH into worker node `node01`.
2. Create a Static Pod named `node-watchdog` running image `busybox:1.36` (`command: ["sleep", "3600"]`).
3. Ensure the pod survives node reboot and is managed directly by `kubelet`.

#### Complete Solution
```bash
# 1. SSH into node01 and identify staticPodPath
ssh node01
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# Default output: staticPodPath: /etc/kubernetes/manifests

# 2. Create Static Pod Manifest in that directory
sudo cat <<EOF > /etc/kubernetes/manifests/node-watchdog.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-watchdog
spec:
  containers:
  - name: watchdog
    image: busybox:1.36
    command: ["sleep", "3600"]
EOF

# 3. Exit and verify from control plane
exit
kubectl get pods -A | grep node-watchdog
# Should show: node-watchdog-node01 Running
```

---

### Task 19: JSONPath Extraction & Resource Sorting (4%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. Extract the names and IP addresses of all Nodes in the cluster, sorted by node name, formatted as `NAME:IP`.
2. Save the output to file `/root/node_ips.txt`.

#### Complete Solution
```bash
# JSONPath extraction with sorting
kubectl get nodes --sort-by=.metadata.name \
  -o jsonpath='{range .items[*]}{.metadata.name}{":"}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}' \
  > /root/node_ips.txt

# Verify output
cat /root/node_ips.txt
```

---

### Task 20: Pod Security Admission (PSA) & Namespace Quotas (3%)
**Context:** `kubectl config use-context k8s`

#### Problem Statement
1. In namespace `restricted-workloads`, configure Pod Security Admission to `enforce: restricted` and `warn: restricted` using Kubernetes labels.
2. Create a `ResourceQuota` named `compute-quota` that limits the namespace to `max 4 CPUs`, `8Gi Memory`, and `10 Pods`.

#### Complete Solution
```bash
# 1. Label namespace for Pod Security Admission (PSA)
kubectl create namespace restricted-workloads $do | kubectl apply -f -

kubectl label namespace restricted-workloads \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=latest \
  --overwrite

# 2. Create ResourceQuota
kubectl create quota compute-quota \
  --namespace=restricted-workloads \
  --hard=cpu=4,memory=8Gi,pods=10
```

```bash
# 3. Verify Quota
kubectl describe quota compute-quota -n restricted-workloads
```

---

## 4. Master Scoring Matrix & Evaluation Rubric

| Task # | Domain | Focus Area | Max Score | Min Passing Threshold |
|:---:|:---|:---|:---:|:---:|
| 1 | Cluster Architecture | RBAC SA, Role, RoleBinding | 4% | 4% |
| 2 | Cluster Architecture | `kubeadm` Upgrade (CP + Worker) | 8% | 8% |
| 3 | Cluster Architecture | `etcd` Backup & Restore | 7% | 7% |
| 4 | Workloads | Multi-Container Shared Volume Pod | 4% | 4% |
| 5 | Scheduling | NodeAffinity & PodAntiAffinity | 5% | 5% |
| 6 | Scheduling | Cordon/Drain & Taints/Tolerations | 4% | 4% |
| 7 | Storage | PV, StorageClass, PVC Binding | 4% | 4% |
| 8 | Storage | SecurityContext `runAsUser`/`fsGroup` | 4% | 4% |
| 9 | Services & Networking | Ingress Routing & TLS Secret | 6% | 6% |
| 10 | Services & Networking | Zero-Trust NetworkPolicy | 6% | 6% |
| 11 | Services & Networking | Service ClusterIP/NodePort & Endpoints | 5% | 5% |
| 12 | Services & Networking | CoreDNS Forwarding | 4% | 4% |
| 13 | Troubleshooting | Worker Node `NotReady` Kubelet Triage | 7% | 7% |
| 14 | Troubleshooting | Static Pod API Server Crash Triage | 7% | 7% |
| 15 | Troubleshooting | Pod `CrashLoopBackOff` OOMKilled Fix | 6% | 6% |
| 16 | Workloads | StatefulSet with Headless Service | 4% | 4% |
| 17 | Workloads | Deployment Rollout & Undo | 4% | 4% |
| 18 | Cluster Architecture | Static Pod on Worker Node | 4% | 4% |
| 19 | Cluster Architecture | JSONPath Extraction & Sorting | 4% | 4% |
| 20 | Cluster Architecture | Pod Security Admission & Quotas | 3% | 3% |
| **TOTAL** | | | **100%** | **66% (PASS)** |

---

## 5. Top 10 High-Yield Exam Pitfalls & Prevention Strategies

1. **Forgetting to Switch Context:** Always copy-paste the `kubectl config use-context <name>` command at the top of every question.
2. **Missing Namespaces:** Always append `-n <namespace>` or set context namespace: `kubectl config set-context --current --namespace=<ns>`.
3. **Overwriting Existing Manifests:** When editing static pods (`/etc/kubernetes/manifests/`), always create a quick backup: `cp etcd.yaml etcd.yaml.bak`.
4. **Editing Live Pods Directly:** Pod specs are largely immutable; use `kubectl get pod <name> -o yaml > pod.yaml`, edit, then `kubectl replace --force -f pod.yaml`.
5. **Node Drain Failures:** When draining nodes, always specify `--ignore-daemonsets --delete-emptydir-data` to prevent blocking errors.
6. **`etcdctl` Missing API Version:** Always prepend `ETCDCTL_API=3` before all `etcdctl` snapshot commands.
7. **Service Selector Label Mismatches:** If a Service has no endpoints (`kubectl get ep`), check that `spec.selector` matches the Pod labels character-for-character.
8. **NetworkPolicy Default Deny Trap:** Applying a NetworkPolicy isolates selected Pods immediately; ensure ingress rules explicitly allow port and pod selector.
9. **StorageClass `WaitForFirstConsumer` Confusion:** PVCs with `WaitForFirstConsumer` stay in `Pending` state until a consumer Pod is actually created and scheduled.
10. **Slow Typing:** Never write YAML from scratch! Use `kubectl create ... --dry-run=client -o yaml` or `kubectl run ... --dry-run=client -o yaml` and redirect to a file.
