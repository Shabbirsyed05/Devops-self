# ☸️ Kubernetes Advanced Architecture & Platform Engineering Master Handbook (Volume 3)
> **From Production Incidents to Platform Excellence — Designing, Securing, Scaling, and Operating Enterprise-Grade Kubernetes Platforms**  
> *Engineered for Principal DevOps Engineers, Lead Site Reliability Engineers (SREs), Platform Architects, and Enterprise Cloud Leaders.*

---

## 📑 Master Table of Contents
1. [Core Platform Architecture Mental Models](#1-core-platform-architecture-mental-models)
2. [20-Module Advanced Platform Engineering Curriculum Deep Dive](#2-20-module-advanced-platform-engineering-curriculum-deep-dive)
   - [Module 1: Ingress Architecture & Host/Path-Based L7 Routing](#module-1-ingress-architecture--hostpath-based-l7-routing-page-1)
   - [Module 2: Ingress Controller Ecosystem (NGINX, Traefik, HAProxy, AWS ALB)](#module-2-ingress-controller-ecosystem-nginx-traefik-haproxy-aws-alb-page-2)
   - [Module 3: Gateway API: The Role-Oriented Evolution of Kubernetes Networking](#module-3-gateway-api-the-role-oriented-evolution-of-kubernetes-networking-page-3)
   - [Module 4: Zero-Trust Network Policies & eBPF Microsegmentation (Calico / Cilium)](#module-4-zero-trust-network-policies--ebpf-microsegmentation-calico--cilium-page-4)
   - [Module 5: Role-Based Access Control (RBAC): Roles, ClusterRoles & Least Privilege](#module-5-role-based-access-control-rbac-roles-clusterroles--least-privilege-page-5)
   - [Module 6: Workload Identity & Projected ServiceAccount Tokens (IRSA)](#module-6-workload-identity--projected-serviceaccount-tokens-irsa-page-6)
   - [Module 7: Admission Controllers: Mutating, Validating, OPA Gatekeeper & Kyverno](#module-7-admission-controllers-mutating-validating-opa-gatekeeper--kyverno-page-7)
   - [Module 8: Pod Security Standards (PSS): Privileged, Baseline & Restricted](#module-8-pod-security-standards-pss-privileged-baseline--restricted-page-8)
   - [Module 9: Multi-Tenant Governance: `ResourceQuotas` & `LimitRanges`](#module-9-multi-tenant-governance-resourcequotas--limitranges-page-9)
   - [Module 10: PriorityClasses & Workload Preemption Under Cluster Pressure](#module-10-priorityclasses--workload-preemption-under-cluster-pressure-page-10)
   - [Module 11: Enterprise Packaging with Helm: Templates, Values & Release Lifecycles](#module-11-enterprise-packaging-with-helm-templates-values--release-lifecycles-page-11)
   - [Module 12: Declarative Configuration with Kustomize: Base & Overlays](#module-12-declarative-configuration-with-kustomize-base--overlays-page-12)
   - [Module 13: Kubernetes Operator Pattern: Custom Controllers & Day-2 Automation](#module-13-kubernetes-operator-pattern-custom-controllers--day-2-automation-page-13)
   - [Module 14: Custom Resource Definitions (CRDs): Extending the Kubernetes API](#module-14-custom-resource-definitions-crds-extending-the-kubernetes-api-page-14)
   - [Module 15: Service Mesh Deep Dive: Istio vs. Linkerd (mTLS, Canary, Tracing)](#module-15-service-mesh-deep-dive-istio-vs-linkerd-mtls-canary-tracing-page-15)
   - [Module 16: Multi-Cluster Kubernetes: Federation, Cilium Mesh & Global Services](#module-16-multi-cluster-kubernetes-federation-cilium-mesh--global-services-page-16)
   - [Module 17: Enterprise GitOps: Argo CD & Flux CD (Drift Detection & Reconciliation)](#module-17-enterprise-gitops-argo-cd--flux-cd-drift-detection--reconciliation-page-17)
   - [Module 18: Platform Engineering & Internal Developer Platforms (IDP / Backstage)](#module-18-platform-engineering--internal-developer-platforms-idp--backstage-page-18)
   - [Module 19: Advanced Kubernetes CLI Mastery & Operator Cheat Sheet](#module-19-advanced-kubernetes-cli-mastery--operator-cheat-sheet-page-19)
   - [Module 20: Enterprise Platform Readiness & Governance Audit Scorecard](#module-20-enterprise-platform-readiness--governance-audit-scorecard-page-20)
3. [Senior Platform Architect Interview Q&A (Enterprise Level)](#3-senior-platform-architect-interview-qa-enterprise-level)

---

## 1. Core Platform Architecture Mental Models

```
                    THE 7 GOLDEN RULES OF PLATFORM ENGINEERING
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ 1. Ingress vs. Gateway API: Role-Oriented Separation of Concerns            │
 │    • Ingress combines all L7 rules into a single monolith object.           │
 │    • Gateway API decouples Infra (`GatewayClass`), Ops (`Gateway`), and     │
 │      Devs (`HTTPRoute`), enabling safe multi-tenant routing at scale.       │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 2. Default is Open; Your Policy is Your Perimeter                           │
 │    • Without NetworkPolicies, all Pods talk to all Pods across namespaces.  │
 │    • Always enforce a `Default Deny Ingress & Egress` policy on day one.    │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 3. Never Embed Static IAM Keys; Use Projected Workload Identity             │
 │    • Bind Kubernetes ServiceAccounts directly to Cloud IAM Roles (IRSA).    │
 │    • Projected short-lived tokens eliminate permanent credential leakage.   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 4. Admission Controllers Enforce Security Before etcd Persistence           │
 │    • Mutating Webhooks modify objects (Inject sidecars, set safe defaults). │
 │    • Validating Webhooks (Kyverno / OPA) reject non-compliant manifests.   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 5. LimitRanges Set Default Boundaries; ResourceQuotas Cap Aggregates        │
 │    • `LimitRange` prevents individual runaway containers in a namespace.    │
 │    • `ResourceQuota` prevents team A from exhausting total cluster CPU/RAM. │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 6. Operators Encode Human SRE Runbooks into Continuous Reconcile Loops      │
 │    • Operators = Custom Controller + Custom Resource Definition (CRD).      │
 │    • Automates Day-2 ops: backups, failovers, schema migrations, scaling.   │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ 7. Git is the Single Source of Truth (GitOps Drift Reconciliation)          │
 │    • No manual `kubectl apply` in production. Argo CD / Flux CD detects     │
 │      cluster drift and continuously reconciles state back to Git.           │
 └─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 20-Module Advanced Platform Engineering Curriculum Deep Dive

---

### Module 1: Ingress Architecture & Host/Path-Based L7 Routing (Page 1)

```
                            INGRESS TRAFFIC ROUTING
                               [ External Users ]
                                       │
                                       ▼
                       [ Cloud Load Balancer (AWS ALB) ]
                                       │ (Port 80 / 443)
                                       ▼
                     [ Ingress Controller (NGINX Ingress) ]
                                       │
             ┌─────────────────────────┴─────────────────────────┐
  (Host: api.corp.com / Path: /v1)            (Host: api.corp.com / Path: /v2)
             ▼                                                   ▼
     [ Service: api-v1 ]                                 [ Service: api-v2 ]
             │                                                   │
             ▼                                                   ▼
      [ Pods (v1.0) ]                                     [ Pods (v2.0) ]
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: core-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts: [api.corp.com]
      secretName: api-tls-cert
  rules:
    - host: api.corp.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: api-v1-svc
                port:
                  number: 8080
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: api-v2-svc
                port:
                  number: 8080
```

---

### Module 2: Ingress Controller Ecosystem (NGINX, Traefik, HAProxy, AWS ALB) (Page 2)

| Feature | NGINX Ingress Controller | Traefik | HAProxy Ingress | AWS Load Balancer Controller |
| :--- | :--- | :--- | :--- | :--- |
| **Operating Layer** | Layer 7 (HTTP/HTTPS) | Layer 7 (Modern Microservices)| Layer 4 & Layer 7 | Layer 7 (Native AWS ALB) |
| **Architecture** | In-cluster pod reverse proxy | Native dynamic config reload | Ultra-high performance C engine | Provisions external AWS ALB |
| **Auto TLS** | Via `cert-manager` | Native Let's Encrypt engine | Via `cert-manager` | Native AWS Certificate Manager |
| **Best Use Case** | Enterprise standard & custom regex | Dynamic multi-tenant microservices | Ultra-low latency & TCP proxying | AWS-native cloud integrations |

---

### Module 3: Gateway API: The Role-Oriented Evolution of Kubernetes Networking (Page 3)

```
                            GATEWAY API ROLE MODEL
 ┌───────────────────────┬───────────────────────────┬─────────────────────────┐
 │ Role Persona          │ Gateway API Resource      │ Scope & Responsibility  │
 ├───────────────────────┼───────────────────────────┼─────────────────────────┤
 │ 🏢 Infra Provider     │ `GatewayClass`            │ Cluster-wide controller │
 │                       │ (e.g., istio, cilium, alb)│ definitions (AWS, Envoy)│
 ├───────────────────────┼───────────────────────────┼─────────────────────────┤
 │ ⚙️ Cluster Operator   │ `Gateway`                 │ Declares network ports, │
 │ (Platform Team)       │ (Port 80/443, VIP, TLS)   │ IP bindings & hostnames │
 ├───────────────────────┼───────────────────────────┼─────────────────────────┤
 │ 👩‍💻 Application Dev    │ `HTTPRoute` / `GRPCRoute` │ Path matching, traffic  │
 │ (Service Teams)       │ (Routes traffic to Svcs)  │ splitting, canary rules │
 └───────────────────────┴───────────────────────────┴─────────────────────────┘
```

#### 🛠️ Gateway API Specification Example
```yaml
# 1. Gateway (Platform Team)
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-gateway
  namespace: platform
spec:
  gatewayClassName: istio
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        certificateRefs:
          - name: prod-wildcard-tls

---
# 2. HTTPRoute with Canary Traffic Split (App Team)
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: payment-route
  namespace: payments
spec:
  parentRefs:
    - name: prod-gateway
      namespace: platform
  hostnames:
    - "payments.corp.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /checkout
      backendRefs:
        - name: payment-v1-svc
          port: 8080
          weight: 90               # 90% traffic to stable v1
        - name: payment-v2-svc
          port: 8080
          weight: 10               # 10% traffic to canary v2
```

---

### Module 4: Zero-Trust Network Policies & eBPF Microsegmentation (Calico / Cilium) (Page 4)

```
                       ZERO-TRUST PERIMETER ENFORCEMENT
   [ Frontend Pod ] ──────── Ingress Port 8080 ────────▶ [ Payment Backend ]
          │                                                       │
          │ (Egress Denied!)                                      │ Egress TCP 5432
          ▼                                                       ▼
   [ Unauthorized Pod ] ──❌ (Dropped)                      [ PostgreSQL Database ]
```

```yaml
# Enforce Default Deny on Namespace AND explicitly allow trusted paths
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-zero-trust
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: payment-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend-web
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres-db
      ports:
        - protocol: TCP
          port: 5432
    # Mandatory: DNS Resolution
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

---

### Module 5: Role-Based Access Control (RBAC): Roles, ClusterRoles & Least Privilege (Page 5)

```
                            RBAC BINDING ARCHITECTURE
 ┌──────────────────────┐          RoleBinding          ┌──────────────────────┐
 │ Subject              │──────────────────────────────▶│ Role / ClusterRole   │
 │ • User (alice)       │    (Binds Subject to Role)    │ • API Groups: [""]   │
 │ • Group (developers) │                               │ • Resources: ["pods"]│
 │ • ServiceAccount (sa)│                               │ • Verbs: [get, list] │
 └──────────────────────┘                               └──────────────────────┘
```

```yaml
# 1. Scoped Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: payments
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]

---
# 2. RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: payments
subjects:
  - kind: User
    name: dev-user@corp.com
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### Module 6: Workload Identity & Projected ServiceAccount Tokens (IRSA) (Page 6)

```
                       AWS IAM ROLES FOR SERVICE ACCOUNTS (IRSA)
 ┌─────────────────┐      OIDC Token Exchange       ┌───────────────────────────┐
 │ Pod with SA     │───────────────────────────────▶│ AWS STS (AssumeRoleWithWeb)│
 │ (payment-sa)    │                                │ Validates K8s OIDC Issuer │
 └─────────────────┘                                └─────────────┬─────────────┘
          ▲                                                       │ (Issues Short-Lived
          └───────────── Temporary AWS IAM Credentials ───────────┘  Session Keys)
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payment-sa
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/PaymentDynamoDBAccessRole
```

---

### Module 7: Admission Controllers: Mutating, Validating, OPA Gatekeeper & Kyverno (Page 7)

```
                          ADMISSION CONTROLLER PIPELINE
 ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
 │ 1. API Req   │───▶│ 2. Mutating  │───▶│ 3. Validating│───▶│ 4. Persist   │
 │ Authenticate │    │ Webhook      │    │ Webhook      │    │ to etcd      │
 │ & Authorize  │    │ (Inject/Mod) │    │ (Allow/Deny) │    │ Database     │
 └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

#### 🛡️ Kyverno Policy: Block Containers Running as Root
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-run-as-non-root
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-runAsNonRoot
      match:
        resources:
          kinds: [Pod]
      validate:
        message: "Running as root is strictly forbidden in production clusters!"
        pattern:
          spec:
            securityContext:
              runAsNonRoot: true
```

---

### Module 8: Pod Security Standards (PSS): Privileged, Baseline & Restricted (Page 8)

```
                        POD SECURITY STANDARDS (PSS) TIERS
 ┌──────────────────────┬──────────────────────────────────────────────────────┐
 │ PSS Profile          │ Security Characteristics & Restrictions              │
 ├──────────────────────┼──────────────────────────────────────────────────────┤
 │ 🔴 Privileged        │ Completely open. Allows root, hostPID, hostNetwork.  │
 │                      │ Reserved strictly for CNIs, storage drivers, logging.│
 ├──────────────────────┼──────────────────────────────────────────────────────┤
 │ 🟡 Baseline          │ Prevents known privilege escalations. Blocks         │
 │                      │ hostNetwork, hostPID, and raw kernel capabilities.   │
 ├──────────────────────┼──────────────────────────────────────────────────────┤
 │ 🟢 Restricted        │ Hardened enterprise standard. Enforces non-root,     │
 │                      │ drops ALL capabilities, enforces read-only root fs.  │
 └──────────────────────┴──────────────────────────────────────────────────────┘
```

```bash
# Enforce Restricted PSS at Namespace Level via Labels
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted
```

---

### Module 9: Multi-Tenant Governance: `ResourceQuotas` & `LimitRanges` (Page 9)

```yaml
# 1. LimitRange (Guarantees every container has sane defaults)
apiVersion: v1
kind: LimitRange
metadata:
  name: default-container-limits
  namespace: dev-team
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
      max:
        cpu: "2"
        memory: "2Gi"

---
# 2. ResourceQuota (Caps entire namespace aggregate consumption)
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "50"
    services.loadbalancers: "2"
```

---

### Module 10: PriorityClasses & Workload Preemption Under Cluster Pressure (Page 10)

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: mission-critical
value: 1000000                       # High integer = Higher priority
globalDefault: false
preemptionPolicy: PreemptLowerPriority # Evicts lower-priority pods when nodes are full
description: "Used strictly for payment gateways and order processing engines."
```

---

### Module 11: Enterprise Packaging with Helm: Templates, Values & Release Lifecycles (Page 11)

```bash
# Production Helm Workflow
helm create myapp-chart
helm lint myapp-chart/
helm template myapp-release ./myapp-chart -f values-prod.yaml

# Safe Atomic Upgrade (Rolls back automatically if health checks fail!)
helm upgrade --install payment-api ./myapp-chart \
  --namespace production \
  --values values-prod.yaml \
  --atomic \
  --timeout 5m
```

---

### Module 12: Declarative Configuration with Kustomize: Base & Overlays (Page 12)

```
                            KUSTOMIZE DIRECTORY LAYOUT
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ base/                                                                       │
 │ ├── deployment.yaml                                                         │
 │ ├── service.yaml                                                            │
 │ └── kustomization.yaml                                                      │
 ├── overlays/                                                                 │
 │   ├── dev/                                                                  │
 │   │   ├── replica_patch.yaml (Replicas: 1)                                  │
 │   │   └── kustomization.yaml                                                │
 │   └── prod/                                                                 │
 │       ├── replica_patch.yaml (Replicas: 10)                                 │
 │       ├── resources_patch.yaml (High CPU/RAM)                               │
 │       └── kustomization.yaml                                                │
 └─────────────────────────────────────────────────────────────────────────────┘
```

```bash
# Render and apply production overlay without template engines
kubectl kustomize overlays/prod/ | kubectl apply -f -
# Or native kubectl:
kubectl apply -k overlays/prod/
```

---

### Module 13: Kubernetes Operator Pattern: Custom Controllers & Day-2 Automation (Page 13)

```
                        THE OPERATOR RECONCILIATION LOOP
 ┌──────────────┐     Watches CRD      ┌───────────────────────────┐
 │ Custom       │─────────────────────▶│ Operator Controller       │
 │ Resource (CR)│                      │ (Reconciliation Loop)     │
 └──────────────┘                      └─────────────┬─────────────┘
        ▲                                            │ (Compares Actual vs Desired)
        │                                            ▼
        │                              ┌───────────────────────────┐
        └────── Status Updated ────────│ Manages StatefulSets,     │
                                       │ Backups, Failovers, PVCs  │
                                       └───────────────────────────┘
```

* **The Formula:** $\text{Operator} = \text{Custom Resource Definition (CRD)} + \text{Custom Controller (Reconcile Loop)} + \text{Domain Knowledge}$.
* **Why Operators Matter:** Replaces manual database failover runbooks with automated state reconciliation code.

---

### Module 14: Custom Resource Definitions (CRDs): Extending the Kubernetes API (Page 14)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.databases.corp.io
spec:
  group: databases.corp.io
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames: [db]
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine: { type: string, enum: [postgres, mysql] }
                storageSize: { type: string }
                replicas: { type: integer, minimum: 1 }
```

---

### Module 15: Service Mesh Deep Dive: Istio vs. Linkerd (mTLS, Canary, Tracing) (Page 15)

```
                         SERVICE MESH mTLS ENCRYPTION
 [ Pod A (payments) ]                              [ Pod B (orders) ]
 ┌─────────────────┐                               ┌─────────────────┐
 │ App Container   │                               │ App Container   │
 └────────┬────────┘                               └────────▲────────┘
          │ Localhost                                       │ Localhost
          ▼                                                 │
 ┌─────────────────┐       Mutual TLS (mTLS)       ┌────────┴────────┐
 │ Envoy Sidecar   │==============================>│ Envoy Sidecar   │
 └─────────────────┘      (Strict Zero-Trust)      └─────────────────┘
```

* **Core Benefits:**
  1. **Transparent mTLS:** Encrypts and cryptographically authenticates all service-to-service communication.
  2. **Traffic Shifting:** L7 canary releases and circuit breaking without altering application code.
  3. **Observability:** Distributed tracing (OpenTelemetry/Jaeger) and service graph topology (Kiali).

---

### Module 16: Multi-Cluster Kubernetes: Federation, Cilium Mesh & Global Services (Page 16)

```
+========================================================================================+
|                        GLOBAL MULTI-CLUSTER INFRASTRUCTURE                             |
|                                                                                        |
|  [ Global Anycast DNS / Cloudflare ] ──▶ Routes user to closest healthy region         |
|                                                                                        |
|  +--------------------------------+          +--------------------------------------+  |
|  | Cluster US-East (Primary)      |          | Cluster EU-West (Failover)           |  |
|  | - Cilium Cluster Mesh          |◀────────▶| - Cilium Cluster Mesh                |  |
|  | - Pod-to-Pod Encrypted Overlay | (IPsec)  | - Pod-to-Pod Encrypted Overlay       |  |
|  +--------------------------------+          +--------------------------------------+  |
+========================================================================================+
```

---

### Module 17: Enterprise GitOps: Argo CD & Flux CD (Drift Detection & Reconciliation) (Page 17)

```
                          ARGO CD GITOPS RECONCILIATION
 ┌─────────────────┐      Polls / Webhook      ┌───────────────────────────┐
 │ Git Repository  │──────────────────────────▶│ Argo CD Application Sync    │
 │ (Single Source) │                           │ (Compares Git vs Cluster) │
 └─────────────────┘                           └─────────────┬─────────────┘
                                                             │
                                                             ▼ (Auto-Sync / Self-Heal)
                                               ┌───────────────────────────┐
                                               │ Kubernetes Cluster        │
                                               │ (Reconciles Drift)        │
                                               └───────────────────────────┘
```

```bash
# Argo CD CLI Sync & Status
argocd app get payment-service
argocd app sync payment-service --prune
```

---

### Module 18: Platform Engineering & Internal Developer Platforms (IDP / Backstage) (Page 18)

```
                       INTERNAL DEVELOPER PLATFORM (IDP)
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ Developer Experience Layer (Backstage Portal, Golden Path CLI, Service Cat) │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ Platform Orchestration Layer (Crossplane, Argo CD, Terraform, Helm)         │
 ├─────────────────────────────────────────────────────────────────────────────┤
 │ Infrastructure Layer (AWS EKS, GCP GKE, Cilium CNI, Vault, Prometheus)      │
 └─────────────────────────────────────────────────────────────────────────────┘
```

* **The Golden Path:** Provides pre-approved, secure, automated templates for microservice creation, eliminating developer cognitive overload while maintaining strict enterprise compliance.

---

### Module 19: Advanced Kubernetes CLI Mastery & Operator Cheat Sheet (Page 19)

```bash
# 1. Output All Installed CRDs
kubectl get crds

# 2. Inspect Custom Resources Defined by Operators
kubectl get databases.databases.corp.io -A

# 3. Impersonate a ServiceAccount to Test RBAC Permissions
kubectl auth can-i create deployments --as=system:serviceaccount:payments:payment-sa -n payments

# 4. View Raw API Latency from API Server
kubectl get --raw /metrics | grep apiserver_request_duration_seconds

# 5. Server-Side Diff Against Cluster State
kubectl diff -k overlays/prod/
```

---

### Module 20: Enterprise Platform Readiness & Governance Audit Scorecard (Page 20)

```
                    ENTERPRISE PLATFORM READINESS SCORECARD
 ┌──┬─────────────────────────────┬────────────────────────────────────────────┐
 │  │ Platform Capability         │ Enterprise Maturity Standard               │
 ├──┼─────────────────────────────┼────────────────────────────────────────────┤
 │☐ │ Zero-Trust Networking       │ Default Deny NetworkPolicies + mTLS Mesh   │
 │☐ │ GitOps Deployment           │ 100% workloads managed via Argo CD / Flux  │
 │☐ │ Policy as Code              │ Kyverno / OPA Gatekeeper enforcing non-root│
 │☐ │ Workload Identity           │ Projected SA tokens (AWS IRSA / GCP WIF)   │
 │☐ │ Multi-Tenancy               │ Strict ResourceQuotas & LimitRanges active │
 │☐ │ Golden Path IDP             │ Backstage software templates for devs      │
 │☐ │ Disaster Recovery           │ Automated multi-region backup & failover   │
 └──┴─────────────────────────────┴────────────────────────────────────────────┘
```

---

## 3. Senior Platform Architect Interview Q&A (Enterprise Level)

| # | High-Frequency Architecture Question | Senior Platform Architect Model Answer |
|---|---|---|
| 1 | **What is the difference between Kubernetes Ingress and the Gateway API?** | *Ingress is an early monolithic L7 specification that forced vendor-specific annotations for basic features (canary, rewrites) and lacked role separation. The **Gateway API** is a modern, expressive standard that decouples infrastructure concerns across 3 personas: `GatewayClass` (Infra Provider), `Gateway` (Platform Operator), and `HTTPRoute` (Application Developer), providing native multi-tenant traffic routing.* |
| 2 | **How does the Kubernetes Operator Pattern work under the hood?** | *An Operator combines a **Custom Resource Definition (CRD)** with a **Custom Controller** running an infinite reconciliation loop. The controller continuously watches the Kubernetes API server for events, compares the desired state declared in the CRD instance with the actual runtime state of the system, and executes automated idempotent mutations to heal state.* |
| 3 | **Why are Projected ServiceAccount Tokens preferred over legacy static tokens?** | *Legacy ServiceAccount tokens were static, non-expiring JWT secrets stored directly in etcd secrets, creating permanent exfiltration risks. **Projected Tokens** are generated dynamically by `kube-apiserver` with short lifespans (e.g., 1 hour), bound to specific audiences (OIDC), and automatically rotated by `kubelet` directly in tmpfs mounts.* |
| 4 | **How does a Service Mesh enforce Mutual TLS (mTLS) without modifying application code?** | *The Service Mesh injects an Envoy sidecar proxy into each application Pod using a Mutating Admission Webhook. iptables rules intercept all incoming and outgoing TCP traffic, redirecting it through the local Envoy proxy. Envoy negotiates TLS handshakes, verifies SPIFFE cryptographic identities, and encrypts traffic transparently.* |
| 5 | **What is the difference between GitOps Push vs. Pull models?** | *• **Push Model (CI-driven):** CI server (Jenkins/GitHub Actions) holds cluster administrative credentials and pushes changes using `kubectl apply`. High security blast radius if CI is breached.<br>• **Pull Model (Argo CD/Flux):** An in-cluster agent pulls desired state from Git and reconciles differences. Zero external cluster credentials required; enables automated drift detection and self-healing.* |

---
*Created for Enterprise Platform Engineering, Kubernetes Governance & Principal Systems Interviews.*
