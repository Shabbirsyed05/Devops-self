# ☸️ Junior Kubernetes Administrator Projects Master Handbook
> **From Kubernetes Fundamentals to Practical Cluster Administration — 18 Hands-On Projects, Troubleshooting Labs & Capstone Incident Recoveries**  
> *Engineered for Aspiring Kubernetes Administrators, Junior DevOps Engineers, Cloud Practitioners, and CKA/CKAD Candidates.*

---

## 📑 Master Table of Contents
1. [Core Kubernetes Administrator Mental Models](#1-core-kubernetes-administrator-mental-models)
2. [18 Hands-On Administrator Projects Deep Dive](#2-18-hands-on-administrator-projects-deep-dive)
   - [Project 1: Build Your First Kubernetes Cluster (Minikube / Kind)](#project-1-build-your-first-kubernetes-cluster-minikube--kind-page-2)
   - [Project 2: Deploy Your First Application (Deployment ➔ ReplicaSet ➔ Pods)](#project-2-deploy-your-first-application-deployment--replicaset--pods-page-3)
   - [Project 3: Expose an Application with a Service (ClusterIP & Endpoints)](#project-3-expose-an-application-with-a-service-clusterip--endpoints-page-4)
   - [Project 4: Manage Applications with Namespaces (Multi-Tenant Isolation)](#project-4-manage-applications-with-namespaces-multi-tenant-isolation-page-5)
   - [Project 5: Configure an Application with ConfigMaps (Env & Volumes)](#project-5-configure-an-application-with-configmaps-env--volumes-page-6)
   - [Project 6: Manage Application Secrets (Credentials & Base64 Best Practices)](#project-6-manage-application-secrets-credentials--base64-best-practices-page-7)
   - [Project 7: Perform a Zero-Downtime Rolling Update & Safe Rollback](#project-7-perform-a-zero-downtime-rolling-update--safe-rollback-page-8)
   - [Project 8: Configure Application Health Checks (Startup, Liveness & Readiness)](#project-8-configure-application-health-checks-startup-liveness--readiness-page-9)
   - [Project 9: Control CPU and Memory (Requests, Limits & OOMKill Lab)](#project-9-control-cpu-and-memory-requests-limits--oomkill-lab-page-10)
   - [Project 10: Create Persistent Application Storage (PV, PVC & Data Survival)](#project-10-create-persistent-application-storage-pv-pvc--data-survival-page-11)
   - [Project 11: Run Batch Jobs & Scheduled Tasks (CronJobs & Auto-Cleanup)](#project-11-run-batch-jobs--scheduled-tasks-cronjobs--auto-cleanup-page-12)
   - [Project 12: Control Workload Placement (NodeSelector, NodeAffinity & Taints)](#project-12-control-workload-placement-nodeselector-nodeaffinity--taints-page-13)
   - [Project 13: Build a Multi-Tier Application (React ➔ Node.js API ➔ PostgreSQL)](#project-13-build-a-multi-tier-application-react--nodejs-api--postgresql-page-14)
   - [Project 14: Troubleshooting Lab: Diagnose & Fix `CrashLoopBackOff`](#project-14-troubleshooting-lab-diagnose--fix-crashloopbackoff-page-15)
   - [Project 15: Troubleshooting Lab: Fix `ImagePullBackOff` & `Pending` Pods](#project-15-troubleshooting-lab-fix-imagepullbackoff--pending-pods-page-16)
   - [Project 16: Troubleshooting Lab: Restore Broken Kubernetes Networking](#project-16-troubleshooting-lab-restore-broken-kubernetes-networking-page-17)
   - [Project 17: Implement Basic RBAC Security (ServiceAccount, Role & Binding)](#project-17-implement-basic-rbac-security-serviceaccount-role--binding-page-18)
   - [Project 18: Perform a Junior Administrator Cluster Health Check](#project-18-perform-a-junior-administrator-cluster-health-check-page-19)
3. [Final Capstone Project: Recover a Broken Multi-Tier Environment (10 Failures)](#3-final-capstone-project-recover-a-broken-multi-tier-environment-10-failures-page-20)
4. [Junior Kubernetes Administrator Master Command Reference](#4-junior-kubernetes-administrator-master-command-reference)
5. [High-Frequency Junior/Mid Kubernetes Interview Q&A](#5-high-frequency-juniormid-kubernetes-interview-qa)

---

## 1. Core Kubernetes Administrator Mental Models

```
                      THE 7 CORE PRINCIPLES OF K8S ADMINS
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. Always Read Events & Logs Before Restarting Workloads                    │
 │    • `kubectl describe pod <pod>` shows Kubernetes lifecycle events.        │
 │    • `kubectl logs <pod> --previous` shows why the application died.        │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. Pods are Ephemeral; Persistent Data Belongs in PVCs                      │
 │    • Containers are wiped on restart. Persistent data must live in PVCs.    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 3. Services Route Traffic ONLY to Healthy (Ready) Pods                      │
 │    • If a Pod fails its Readiness Probe, it is removed from Service         │
 │      Endpoints (`kubectl get ep`).                                          │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 4. ConfigMaps for Settings; Secrets for Sensitive Credentials               │
 │    • Environment variables freeze at startup; volume mounts auto-sync.      │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 5. Requests Guarantee Scheduling; Limits Protect the Node                   │
 │    • Kube-scheduler uses `requests` to find nodes. Kubelet uses `limits`    │
 │      to throttle CPU or OOMKill runaway memory consumers.                   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 6. Least Privilege RBAC: Never Grant `cluster-admin` for Daily Workloads    │
 │    • Restrict ServiceAccounts to specific namespaces, resources, and verbs. │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 7. Test Changes Safely with `--dry-run=client -o yaml`                      │
 │    • Generate and review declarative YAML before applying to the cluster.   │
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 18 Hands-On Administrator Projects Deep Dive

---

### Project 1: Build Your First Kubernetes Cluster (Minikube / Kind) (Page 2)

```bash
# 1. Start a multi-node local cluster with Minikube
minikube start --nodes 3 --driver=docker

# 2. Verify Control Plane and Worker Nodes
kubectl cluster-info
kubectl get nodes -o wide

# 3. Explore Core System Namespaces & System Pods
kubectl get namespaces
kubectl get pods -n kube-system
```

---

### Project 2: Deploy Your First Application (Deployment ➔ ReplicaSet ➔ Pods) (Page 3)

```yaml
# nginx-deployment.yaml
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
          image: nginx:1.25
          ports:
            - containerPort: 80
```

```bash
# Apply and verify object hierarchy
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods -l app=nginx
```

---

### Project 3: Expose an Application with a Service (ClusterIP & Endpoints) (Page 4)

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx                        # Must match Deployment pod labels!
  ports:
    - protocol: TCP
      port: 80                        # Port exposed by Service
      targetPort: 80                  # Port NGINX listens on
```

```bash
# Verify Service Endpoints & Internal Connectivity
kubectl apply -f nginx-service.yaml
kubectl get svc nginx-service
kubectl get endpoints nginx-service

# Test connectivity from inside cluster
kubectl run curl-test --rm -it --image=curlimages/curl -- curl http://nginx-service.default.svc.cluster.local
```

---

### Project 4: Manage Applications with Namespaces (Multi-Tenant Isolation) (Page 5)

```bash
# 1. Create isolated environment namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# 2. Deploy into specific namespace
kubectl apply -f nginx-deployment.yaml -n dev
kubectl apply -f nginx-deployment.yaml -n prod

# 3. Set default working namespace for current context
kubectl config set-context --current --namespace=dev
kubectl get pods                      # Automatically lists dev pods
```

---

### Project 5: Configure an Application with ConfigMaps (Env & Volumes) (Page 6)

```bash
# 1. Create ConfigMap imperatively
kubectl create configmap app-config \
  --from-literal=APP_NAME="Production-Portal" \
  --from-literal=APP_PORT="8080" \
  --from-literal=ENV="production"
```

```yaml
# Injecting ConfigMap as Environment Variables and Volume Mounts
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: app
          image: nginx
          env:
            - name: PORT
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_PORT
          volumeMounts:
            - name: config-vol
              mountPath: /etc/config
      volumes:
        - name: config-vol
          configMap:
            name: app-config
```

---

### Project 6: Manage Application Secrets (Credentials & Base64 Best Practices) (Page 7)

```bash
# Create Secret with database credentials
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=SuperSecretPassword123 \
  --from-literal=DB_HOST=postgres-service
```

```yaml
# Injecting Secret into Application Pod
spec:
  containers:
    - name: api
      image: myapp:v1
      env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

---

### Project 7: Perform a Zero-Downtime Rolling Update & Safe Rollback (Page 8)

```bash
# 1. Trigger Rolling Update by updating image
kubectl set image deployment/nginx-deployment nginx=nginx:1.26 --record

# 2. Watch rollout progress in real-time
kubectl rollout status deployment/nginx-deployment

# 3. View Revision History
kubectl rollout history deployment/nginx-deployment

# 4. Instant Emergency Rollback to Previous Version
kubectl rollout undo deployment/nginx-deployment
```

---

### Project 8: Configure Application Health Checks (Startup, Liveness & Readiness) (Page 9)

```yaml
spec:
  containers:
    - name: api
      image: myapp:v2
      # 1. Startup Probe: Disables liveness/readiness until app finishes warmup
      startupProbe:
        httpGet:
          path: /health/startup
          port: 8080
        failureThreshold: 30
        periodSeconds: 10
      # 2. Liveness Probe: Restarts container if deadlock occurs
      livenessProbe:
        httpGet:
          path: /health/live
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      # 3. Readiness Probe: Removes pod from Service Endpoints if overloaded
      readinessProbe:
        httpGet:
          path: /health/ready
          port: 8080
        periodSeconds: 5
```

---

### Project 9: Control CPU and Memory (Requests, Limits & OOMKill Lab) (Page 10)

```yaml
resources:
  requests:
    cpu: "250m"         # 0.25 CPU Core guaranteed
    memory: "256Mi"     # 256 MB RAM guaranteed
  limits:
    cpu: "500m"         # Max 0.5 CPU Core (Throttled if exceeded)
    memory: "512Mi"     # Max 512 MB RAM (OOMKilled if exceeded!)
```

```bash
# Stress-testing memory limit to trigger Exit Code 137 (OOMKilled)
kubectl run memory-stress --rm -it --image=polinux/stress -- \
  stress --vm 1 --vm-bytes 600M --timeout 10s

# Inspect OOMKill termination event
kubectl describe pod memory-stress | grep -E "Reason:|Exit Code:"
# Last State: Terminated, Reason: OOMKilled, Exit Code: 137
```

---

### Project 10: Create Persistent Application Storage (PV, PVC & Data Survival) (Page 11)

```yaml
# 1. PersistentVolume (Storage Pool)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
---
# 2. PersistentVolumeClaim (Storage Request)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

```bash
# Verify binding
kubectl get pv
kubectl get pvc
# STATUS should transition from Pending -> Bound
```

---

### Project 11: Run Batch Jobs & Scheduled Tasks (CronJobs & Auto-Cleanup) (Page 12)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hourly-cleanup
spec:
  schedule: "0 * * * *"              # Runs at minute 0 of every hour
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 300   # Auto-clean Job and Pods 5 minutes after completion
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: busybox
              command: ["sh", "-c", "echo 'Running log cleanup'; rm -rf /tmp/logs/*"]
```

---

### Project 12: Control Workload Placement (NodeSelector, NodeAffinity & Taints) (Page 13)

```bash
# 1. Add Label to Node
kubectl label nodes worker-node-02 disktype=ssd env=production

# 2. Add Taint to Dedicated Node
kubectl taint nodes worker-node-02 dedicated=backend:NoSchedule
```

```yaml
# Workload with Matching NodeSelector & Toleration
spec:
  nodeSelector:
    disktype: ssd
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "backend"
      effect: "NoSchedule"
```

---

### Project 13: Build a Multi-Tier Application (React ➔ Node.js API ➔ PostgreSQL) (Page 14)

```
                            3-TIER APPLICATION ARCHITECTURE
 ┌─────────────────┐      HTTP Port 80       ┌──────────────────┐     TCP Port 5432     ┌─────────────────┐
 │ Frontend (React)│────────────────────────▶│ Backend API      │──────────────────────▶│ PostgreSQL      │
 │ Service:        │                         │ Service:         │                       │ Service:        │
 │ frontend-svc:80 │                         │ backend-svc:3000 │                       │ database-svc    │
 └─────────────────┘                         └──────────────────┘                       └─────────────────┘
```

---

### Project 14: Troubleshooting Lab: Diagnose & Fix `CrashLoopBackOff` (Page 15)

```bash
# Step-by-Step Triage Workflow
# 1. Check Pod State & Restart Count
kubectl get pods

# 2. Pull crash logs from previous container run
kubectl logs <pod-name> --previous

# 3. Inspect Pod lifecycle events
kubectl describe pod <pod-name>

# 4. Common Fix: Correct environment variable typos or missing config files
```

---

### Project 15: Troubleshooting Lab: Fix `ImagePullBackOff` & `Pending` Pods (Page 16)

```
 ┌────┬────────────────────────┬──────────────────────────────────┬────────────────────────────────────────────┐
 │ #  │ Failure Scenario       │ Diagnostic Indicator             │ Root Cause Fix                             │
 ├────┼────────────────────────┼──────────────────────────────────┼────────────────────────────────────────────┤
 │ 1  │ Invalid Image Name/Tag │ `ErrImagePull / ImagePullBackOff`│ Correct repository path or image tag.      │
 │ 2  │ Missing Auth Secret    │ `401 Unauthorized` in events     │ Create & link `imagePullSecrets`.          │
 │ 3  │ Saturated Node Memory  │ Pod stuck in `Pending`           │ Lower `requests.memory` or scale cluster.  │
 │ 4  │ NodeSelector Mismatch  │ `0/3 nodes available` in events  │ Update `nodeSelector` or label nodes.      │
 └────┴────────────────────────┴──────────────────────────────────┴────────────────────────────────────────────┘
```

---

### Project 16: Troubleshooting Lab: Restore Broken Kubernetes Networking (Page 17)

```bash
# 1. Verify Service Endpoints (If empty, selector does not match pod labels!)
kubectl get endpoints <service-name>

# 2. Test DNS Resolution inside cluster
kubectl run dns-test --rm -it --image=busybox:1.28 -- nslookup backend-svc.default.svc.cluster.local

# 3. Test HTTP connectivity directly
kubectl run curl-test --rm -it --image=curlimages/curl -- curl -v http://backend-svc:3000/health
```

---

### Project 17: Implement Basic RBAC Security (ServiceAccount, Role & Binding) (Page 18)

```yaml
# 1. ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader-sa
  namespace: demo
---
# 2. Role (Read-only on Pods and Services)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-service-reader
  namespace: demo
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
---
# 3. RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-app-reader
  namespace: demo
subjects:
  - kind: ServiceAccount
    name: app-reader-sa
    namespace: demo
roleRef:
  kind: Role
  name: pod-service-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Test permissions with kubectl auth can-i
kubectl auth can-i list pods --as=system:serviceaccount:demo:app-reader-sa -n demo
# Output: yes

kubectl auth can-i delete pods --as=system:serviceaccount:demo:app-reader-sa -n demo
# Output: no
```

---

### Project 18: Perform a Junior Administrator Cluster Health Check (Page 19)

```bash
# Cluster Health Audit Script
kubectl get nodes -o wide                      # Verify all nodes are 'Ready'
kubectl get pods -A --field-selector status.phase!=Running # Find broken pods
kubectl top nodes && kubectl top pods -A      # Check resource saturation
kubectl get events -A --sort-by='.lastTimestamp' | tail -n 20 # Inspect recent errors
```

---

## 3. Final Capstone Project: Recover a Broken Multi-Tier Environment (10 Failures) (Page 20)

```
                            CAPSTONE INCIDENT TRIAGE MATRIX
 ┌────┬────────────────────────────┬─────────────────────────────────┬─────────────────────────────────────────┐
 │ #  │ Hidden Failure             │ Observed Symptom                │ Administrator Forensic Fix              │
 ├────┼────────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────┤
 │ 1  │ Frontend Bad Image Tag     │ `ImagePullBackOff`              │ Update tag to `nginx:1.25-alpine`.      │
 │ 2  │ Service Selector Typo      │ Endpoints `<none>`              │ Change selector `app: web` to `frontend`│
 │ 3  │ Missing ConfigMap          │ Pod stuck `CreateContainerError`│ Apply `app-config` with `API_URL`.      │
 │ 4  │ Secret Key Mismatch        │ Backend fails DB connection     │ Align Secret key `db_pass` in YAML.     │
 │ 5  │ Broken Readiness Probe     │ Pod Running (0/1 Ready)         │ Correct probe path from `/redy` to `/ready`│
 │ 6  │ CPU Limit Too Low (10m)    │ Backend Throttled & Laggy       │ Increase `limits.cpu` to `500m`.        │
 │ 7  │ PVC Unbound (Wrong Class)  │ Database Pod `Pending`          │ Match `storageClassName: standard`.     │
 │ 8  │ App Crash on Port Conflict │ Backend `CrashLoopBackOff`      │ Change listening port from 80 to 3000.  │
 │ 9  │ NetworkPolicy Blocking DB  │ Backend DB connection timeout   │ Allow ingress on port 5432 in netpol.   │
 │ 10 │ RBAC Permission Denied     │ App cannot list ConfigMaps      │ Grant `get, list` verbs in Role.        │
 └────┴────────────────────────────┴─────────────────────────────────┴─────────────────────────────────────────┘
```

---

## 4. Junior Kubernetes Administrator Master Command Reference

```bash
# Generate YAML without applying (Dry-Run)
kubectl create deployment web --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment web --port=80 --target-port=80 --dry-run=client -o yaml > svc.yaml

# Interactive Debugging Shell
kubectl exec -it <pod-name> -- /bin/sh

# Fast Pod Restart (Rollout)
kubectl rollout restart deployment <deployment-name>
```

---

## 5. High-Frequency Junior/Mid Kubernetes Interview Q&A

| # | Interview Question | Junior / Mid Administrator Model Answer |
|---|---|---|
| 1 | **What is the difference between `kubectl apply` and `kubectl create`?** | *`kubectl create` is imperative; it creates a new resource and fails if the resource already exists. `kubectl apply` is declarative; it creates or updates the resource by comparing the local YAML with the live cluster configuration using `last-applied-configuration` annotations.* |
| 2 | **Why is my Service not forwarding traffic to my Pods?** | *1. The Service `spec.selector` does not match the Pod `metadata.labels`.<br>2. The Pods failed their Readiness Probes and were removed from Service Endpoints.<br>3. The Service `targetPort` does not match the port the application is listening on.* |
| 3 | **What is the difference between a ConfigMap and a Secret?** | *ConfigMaps store non-sensitive plain text configuration data (environment variables, config files). Secrets store sensitive credentials (passwords, tokens, TLS certificates) and are Base64 encoded (and can be encrypted at rest in etcd).* |
| 4 | **What happens if a Pod exceeds its Memory Limit vs. CPU Limit?** | *If a Pod exceeds its CPU limit, Kubernetes throttles the CPU (application runs slower, but does not crash). If a Pod exceeds its Memory limit, the Linux kernel terminates the container immediately with **Exit Code 137 (OOMKilled)**.* |
| 5 | **How do you safely drain a node for maintenance without causing downtime?** | *Run `kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data`. This cordons the node to prevent new pods and evicts running pods gracefully according to PodDisruptionBudgets.* |

---
*Created for Hands-On Kubernetes Administration, Practical Lab Mastery & Junior/Mid Systems Interviews.*
