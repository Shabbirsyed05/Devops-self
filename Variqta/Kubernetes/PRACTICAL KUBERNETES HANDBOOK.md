# 🛠️ VERIQTA Practical Kubernetes Master Handbook
> **From Local Cluster Labs to Production Operations: Hands-On Blueprints, Troubleshooting Labs, Security Hardening & Capstone Projects.**  
> *Learn by Doing. Build by Practicing. Master by Solving. Designed for DevOps Engineers, SREs, and Platform Operators.*

---

## 📑 Table of Contents
1. [Executive Summary & Core Practical Competency Matrix](#1-executive-summary--core-practical-competency-matrix)
2. [Page 1: Build Your Kubernetes Lab (Minikube, Kubectl & Docker)](#page-1-build-your-kubernetes-lab)
3. [Page 2: Deploy Your First Application (Deployment $\rightarrow$ RS $\rightarrow$ Pod $\rightarrow$ Service)](#page-2-deploy-your-first-application)
4. [Page 3: Work with Kubernetes Declarative YAML Manifests](#page-3-work-with-kubernetes-declarative-yaml-manifests)
5. [Page 4: Expose Applications with Services (ClusterIP, NodePort & Endpoints)](#page-4-expose-applications-with-services)
6. [Page 5: Labels, Selectors, and Namespaces (Isolation & Querying)](#page-5-labels-selectors-and-namespaces)
7. [Page 6: Configure Applications with ConfigMaps (Env Vars & Volume Mounts)](#page-6-configure-applications-with-configmaps)
8. [Page 7: Manage Sensitive Configuration with Secrets & Base64 Security](#page-7-manage-sensitive-configuration-with-secrets)
9. [Page 8: Persistent Storage Architecture (PV, PVC, StorageClasses & Lifecycle)](#page-8-persistent-storage-architecture)
10. [Page 9: Resource Requests, Limits & Cgroups (CPU Throttling vs. OOMKill)](#page-9-resource-requests-limits--cgroups)
11. [Page 10: Health Checks & Self-Healing (Liveness, Readiness & Startup Probes)](#page-10-health-checks--self-healing)
12. [Page 11: Scaling Applications (Manual Scaling & Horizontal Pod Autoscaler - HPA)](#page-11-scaling-applications)
13. [Page 12: Rolling Updates, MaxSurge, MaxUnavailable & Instant Rollbacks](#page-12-rolling-updates--rollbacks)
14. [Page 13: Ingress Controllers & Path/Host Application Routing](#page-13-ingress--application-routing)
15. [Page 14: Kubernetes Networking Lab (Pod-to-Pod, CoreDNS & Service IPVS)](#page-14-kubernetes-networking-lab)
16. [Page 15: RBAC & ServiceAccounts (Least Privilege & Permission Testing)](#page-15-rbac--serviceaccounts)
17. [Page 16: Practical Kubernetes Troubleshooting (8 Controlled Failure Scenarios)](#page-16-practical-kubernetes-troubleshooting)
18. [Page 17: Debug a Broken Kubernetes Application (Outside-In Investigation)](#page-17-debug-a-broken-kubernetes-application)
19. [Page 18: Production Deployment Best Practices Blueprint](#page-18-production-deployment-best-practices-blueprint)
20. [Page 19: Failure Simulation & Chaos Recovery Lab Matrix](#page-19-failure-simulation--chaos-recovery-lab-matrix)
21. [Page 20: Final Capstone Project: End-to-End Production-Style Application](#page-20-final-capstone-project)

---

## 1. Executive Summary & Core Practical Competency Matrix

```
                        KUBERNETES HANDS-ON LEARNING CYCLE
 ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
 │ 1. Build Lab    │ ──> │ 2. Deploy Apps  │ ──> │ 3. Break Apps   │ ──> │ 4. Recover &    │
 │ (Minikube/Kind) │     │ (Declarative)   │     │ (Chaos / Labs)  │     │ Harden (Prod)   │
 └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Core Practical Competencies
* **Manifest Mastery:** Crafting clean, declarative multi-tier YAML manifests with proper indentation and schema validation.
* **Service Discovery:** Connecting Frontend $\rightarrow$ Backend $\rightarrow$ Database using stable ClusterIP VIPs and CoreDNS.
* **Zero-Downtime Operations:** Executing progressive rolling updates, automated HPA scaling, and instant rollbacks.
* **Forensic Troubleshooting:** Triaging `CrashLoopBackOff`, `OOMKilled`, `ImagePullBackOff`, and empty endpoints using structured evidence collection.

---

## Page 1: Build Your Kubernetes Lab

```
                           MINIKUBE SINGLE-NODE LAB
 ┌────────────────────────────────────────────────────────────────────────┐
 │ Host Laptop / Dev Machine                                              │
 │   │                                                                    │
 │   ▼                                                                    │
 │ [ Minikube Virtual Machine / Docker Container ]                        │
 │   ├── Control Plane : kube-apiserver, etcd, scheduler, controller-mgr │
 │   ├── Worker Engine : kubelet, containerd runtime, kube-proxy          │
 │   └── Storage/Net   : Local path provisioner, CNI bridge               │
 └────────────────────────────────────────────────────────────────────────┘
```

### Lab Setup Workflow
```bash
# 1. Start Minikube cluster
minikube start --driver=docker --cpus=4 --memory=8192

# 2. Check cluster status
minikube status
kubectl cluster-info

# 3. Verify control plane and node health
kubectl get nodes -o wide
kubectl get pods -A
```

---

## Page 2: Deploy Your First Application

```
                     WORKLOAD RELATIONSHIP HIERARCHY
  Deployment (Desired State: 3 Replicas)
     │
     └──> ReplicaSet (Supervises Pod Count)
             │
             ├──> Pod 1 (nginx-pod-1) ──┐
             ├──> Pod 2 (nginx-pod-2) ──┼──> [ Service (ClusterIP: 10.96.0.10) ]
             └──> Pod 3 (nginx-pod-3) ──┘
```

### Essential Workload Commands
```bash
# Imperative creation (Fast testing only)
kubectl create deployment nginx-demo --image=nginx:1.25 --replicas=3

# Inspect hierarchy
kubectl get deployments
kubectl get rs
kubectl get pods -o wide

# Inspect pod lifecycle events
kubectl describe deployment nginx-demo
kubectl describe pod <pod-name>
```

---

## Page 3: Work with Kubernetes Declarative YAML Manifests

### Production Deployment Blueprint (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    tier: frontend
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
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
```

```bash
# Apply declaratively
kubectl apply -f deployment.yaml

# Local validation before applying
kubectl apply --dry-run=client -f deployment.yaml
```

---

## Page 4: Expose Applications with Services

```
                           SERVICE & ENDPOINTS FLOW
  [ Client Pod ] ──> [ Service: nginx-service (10.96.0.10:80) ]
                                    │
        ┌───────────────────────────┴───────────────────────────┐
        ▼                                                       ▼
  Endpoint 1: 10.244.1.2:80                               Endpoint 2: 10.244.1.3:80
  (nginx-pod-1)                                           (nginx-pod-2)
```

### Service Manifest (`service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

```bash
# Port forward for local testing
kubectl port-forward svc/nginx-service 8080:80

# Test endpoint
curl http://localhost:8080
```

---

## Page 5: Labels, Selectors, and Namespaces

```
                         NAMESPACE RESOURCE ISOLATION
  Namespace: dev (Labels: env=dev)       Namespace: prod (Labels: env=prod)
┌─────────────────────────────────┐    ┌─────────────────────────────────┐
│ [ Pod: app-dev-1 ]              │    │ [ Pod: app-prod-1 ]             │
│ [ Service: app-dev-svc ]        │    │ [ Service: app-prod-svc ]       │
└─────────────────────────────────┘    └─────────────────────────────────┘
```

### Essential Namespace & Label Commands
```bash
# Create environments
kubectl create namespace dev
kubectl create namespace prod

# Label management
kubectl label pod <pod-name> -n dev environment=dev tier=backend --overwrite

# Querying via label selectors
kubectl get pods -n dev -l environment=dev,tier=backend
kubectl get all -n dev --show-labels
```

---

## Page 6: Configure Applications with ConfigMaps

```
                        CONFIGMAP INGESTION METHODS
 ┌────────────────────────────────────────────────────────────────────────┐
 │ ConfigMap: app-config (LOG_LEVEL=debug, DB_HOST=db.prod.svc)           │
 └──────────────────────────────────┬─────────────────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
  Method 1: Environment Variables                       Method 2: Volume Mount
  env:                                                  volumeMounts:
  - name: LOG_LEVEL                                     - name: cfg-vol
    valueFrom:                                            mountPath: /etc/config
      configMapKeyRef:                                    readOnly: true
        name: app-config
        key: LOG_LEVEL
```

---

## Page 7: Manage Sensitive Configuration with Secrets

```
                           KUBERNETES SECRETS
  Your Password: 'MySecret123!' ──(Base64)──> In Manifest: 'TXlTZWNyZXQxMjMh'
                                                     │
                                                     ▼
                                      Stored in etcd (Key-Value)
```

> [!WARNING]
> **Base64 is NOT Encryption:** Base64 is simple encoding reversible with `echo "..." | base64 -d`. Always enable `etcd` KMS encryption-at-rest and use RBAC to restrict Secret access.

```bash
# Create secret from CLI
kubectl create secret generic db-secret \
  --from-literal=username=dbadmin \
  --from-literal=password='ComplexPass!2026'

# View secret keys (Masked values)
kubectl describe secret db-secret
```

---

## Page 8: Persistent Storage Architecture

```
                          STORAGE DECOUPLING ARCHITECTURE
  [ Pod / StatefulSet ] ──> [ PersistentVolumeClaim (PVC) ] ──> [ StorageClass ] ──> [ PersistentVolume (PV) ]
```

### PVC Manifest (`pvc.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

---

## Page 9: Resource Requests, Limits & Cgroups

```
                         CPU & MEMORY ENFORCEMENT
 ┌────────────────────────────────────┬────────────────────────────────────┐
 │ CPU Requests & Limits              │ Memory Requests & Limits           │
 ├────────────────────────────────────┼────────────────────────────────────┤
 │ Requests: Used by Scheduler.       │ Requests: Guaranteed memory floor. │
 │ Limits: Enforced by Linux CFS.     │ Limits: Hard cgroup boundary.      │
 │ Exceeded $\rightarrow$ **Throttled (Slow)** │ Exceeded $\rightarrow$ **OOMKilled (Exit 137)**   │
 └────────────────────────────────────┴────────────────────────────────────┘
```

---

## Page 10: Health Checks & Self-Healing

```
                          THE THREE PROBE TYPES
 ┌───────────────────┬────────────────────────────────────────────────────────┐
 │ Probe Type        │ Failure Action & Purpose                               │
 ├───────────────────┼────────────────────────────────────────────────────────┤
 │ **Startup Probe** │ Shields slow JVM/DB initialization. Pauses other checks│
 │ **Liveness Probe**│ Restarts container if process is deadlocked / crashed. │
 │ **Readiness Probe**│ Removes pod IP from Service endpoints if unready.     │
 └───────────────────┴────────────────────────────────────────────────────────┘
```

### Production Probe Blueprint
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 5
  failureThreshold: 30
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 3
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
```

---

## Page 11: Scaling Applications (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## Page 12: Rolling Updates & Rollbacks

```bash
# 1. Trigger rolling update
kubectl set image deployment/web-deployment web=nginx:1.27

# 2. Watch rollout in real time
kubectl rollout status deployment/web-deployment

# 3. Check history
kubectl rollout history deployment/web-deployment

# 4. Instant rollback on failure
kubectl rollout undo deployment/web-deployment
```

---

## Page 13: Ingress & Application Routing

```
                         PATH-BASED INGRESS ROUTING
  Internet User ──> [ NGINX Ingress Controller ]
                            ├── /app1 ──> [ Service App-1 (Port 80) ]
                            └── /app2 ──> [ Service App-2 (Port 80) ]
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: demo.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

---

## Page 14: Kubernetes Networking Lab

```
                   DNS RESOLUTION TO SERVICE POD TRAFFIC
  1. Client Pod resolves `my-app-svc` via CoreDNS (10.96.0.10)
  2. CoreDNS returns Service ClusterIP (10.96.10.20)
  3. Kube-proxy IPVS table routes request to healthy Backend Pod (10.244.2.7)
```

---

## Page 15: RBAC & ServiceAccounts

```
                           RBAC BUILDING BLOCKS
  [ ServiceAccount: app-sa ] ──> [ RoleBinding: read-pods-binding ] ──> [ Role: read-pods ]
                                                                               │
                                                                   Rules: get, list, watch
                                                                   Resources: pods
```

```bash
# Test authorization
kubectl auth can-i get pods --as=system:serviceaccount:default:app-sa
# Output: yes

kubectl auth can-i delete pods --as=system:serviceaccount:default:app-sa
# Output: no
```

---

## Page 16: Practical Troubleshooting (8 Failure Labs)

| Lab Failure Scenario | Root Cause Injected | First Diagnostic Command | SRE Remediation Action |
| :--- | :--- | :--- | :--- |
| **1. CrashLoopBackOff** | Bad application entrypoint | `kubectl logs <pod> --previous` | Fix syntax error in config/command. |
| **2. ImagePullBackOff** | Typo in container image tag | `kubectl describe pod <pod>` | Correct tag or upload `imagePullSecret`. |
| **3. Pending** | CPU request exceeds node capacity | `kubectl describe pod <pod>` | Right-size requests or add node. |
| **4. OOMKilled (137)** | App memory leak exceeds limit | `kubectl top pod`, `describe pod` | Increase memory limit & fix code leak. |
| **5. Failed Probes** | Wrong probe path `/health` (404) | `kubectl describe pod <pod>` | Correct path to `/healthz` in Pod spec. |
| **6. Empty Endpoints** | Selector mismatch (`app=web` vs `app=api`) | `kubectl get endpoints` | Align Service selector with Pod labels. |
| **7. Missing ConfigMap**| Pod references non-existent config | `kubectl get events` | Create missing ConfigMap resource. |
| **8. Missing Secret** | DB password secret deleted | `kubectl describe pod <pod>` | Restore Secret in target namespace. |

---

## Page 17: Debug a Broken Kubernetes Application

```
                   OUTSIDE-IN 9-STEP TROUBLESHOOTING PATH
  1. External Traffic ──> 2. Ingress ──> 3. Service ──> 4. Endpoints ──> 5. Pod
                                                                             │
  9. Network Conn <── 8. Config/Secrets <── 7. Logs & Events <── 6. Health <─┘
```

---

## Page 18: Production Deployment Best Practices Blueprint

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prod-workload
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: prod-workload
  template:
    metadata:
      labels:
        app: prod-workload
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: ["prod-workload"]
              topologyKey: "topology.kubernetes.io/zone"
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
      containers:
      - name: app
        image: company/app:v1.2.0
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

---

## Page 19: Failure Simulation & Chaos Recovery Lab Matrix

```
                        CHAOS RECOVERY DRILL
  Intentionally Break System ──> Observe Kubelet/Controller Response ──> Document Recovery
```

* **Self-Healing Automation:** Kubernetes automatically recovers deleted pods and process crashes (Liveness probe).
* **Requires Engineer Intervention:** Image typos, missing ConfigMaps/Secrets, DB connection outages, and selector mismatches require manual engineer fix.

---

## Page 20: Final Capstone Project: Production-Style Kubernetes Application

```
                           CAPSTONE ARCHITECTURE
                               [ Ingress TLS ]
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼                                         ▼
      [ Frontend Service ]                      [ Backend API Service ]
      (ClusterIP + 3 Replicas)                  (ClusterIP + 3 Replicas)
                 │                                         │
                 │ (REST API)                              ▼
                 └───────────────────────────────> [ Database StatefulSet ]
                                                   (PostgreSQL + PVC 10Gi)
```

### 10 Capstone Validation Steps
1. Deploy dedicated namespace: `kubectl create ns capstone-prod`
2. Create PostgreSQL StatefulSet with `volumeClaimTemplates` (Persistent Storage).
3. Create Backend API Deployment with ConfigMaps, Secrets, Probes, and HPA.
4. Create Frontend React Deployment with NGINX.
5. Apply Ingress resource routing `/` to Frontend and `/api` to Backend.
6. Verify RBAC least privilege for application ServiceAccount.
7. Perform rolling update on Backend API with zero downtime.
8. Simulate database network partition and observe readiness probe de-registering pods.
9. Verify HPA autoscaling under load (`kubectl top pods`).
10. Clean up all resources: `kubectl delete ns capstone-prod`.
