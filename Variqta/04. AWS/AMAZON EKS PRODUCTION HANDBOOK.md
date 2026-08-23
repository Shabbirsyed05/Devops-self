# 🚢 Amazon EKS Production Master Handbook
> **Build, Deploy, Secure & Operate Enterprise Kubernetes on AWS.**  
> *20-Page Complete Curriculum: Architecture, VPC CNI, IRSA & Pod Identity, Karpenter, ALB Ingress, CSI Storage, Zero-Trust Hardening, Upgrades & Outage Playbooks.*

---

## 📑 Table of Contents
1. [Executive Mental Model: The Enterprise EKS Production Blueprint](#1-executive-mental-model-the-enterprise-eks-production-blueprint)
2. [EKS Production Architecture (Control Plane, VPC Design & Private Endpoints)](#2-eks-production-architecture)
3. [Building a Production EKS Cluster (Terraform, EKS Access Entries & Add-Ons)](#3-building-a-production-eks-cluster)
4. [EKS Networking Architecture (Amazon VPC CNI, ENIs & Prefix Delegation)](#4-eks-networking-architecture)
5. [EKS Identity & Access (IAM, RBAC & EKS Access Entries)](#5-eks-identity--access)
6. [IAM Roles for Service Accounts (IRSA) vs. EKS Pod Identity](#6-iam-roles-for-service-accounts-irsa-vs-eks-pod-identity)
7. [EKS Compute Strategy (Managed Node Groups, Fargate, Spot & Graviton)](#7-eks-compute-strategy)
8. [Next-Gen EKS Autoscaling: Karpenter vs. Cluster Autoscaler & HPA](#8-next-gen-eks-autoscaling-karpenter-vs-cluster-autoscaler--hpa)
9. [Application Deployment on EKS (Declarative YAML, Helm & GitOps)](#9-application-deployment-on-eks)
10. [EKS Load Balancing & Ingress (AWS Load Balancer Controller & IP-Mode Target Groups)](#10-eks-load-balancing--ingress)
11. [EKS Storage Architecture (EBS CSI gp3, EFS Shared Storage & FSx Lustre)](#11-eks-storage-architecture)
12. [EKS Security Hardening (PSA Restricted, Security Groups per Pod & KMS)](#12-eks-security-hardening)
13. [EKS Observability (CloudWatch Container Insights, AMP, AMG & OpenTelemetry)](#13-eks-observability)
14. [EKS Production Troubleshooting (Pending, CrashLoop, OOMKilled & Scheduling)](#14-eks-production-troubleshooting)
15. [EKS Networking Troubleshooting (IP Exhaustion, DNS & CNI Outages)](#15-eks-networking-troubleshooting)
16. [Zero-Downtime EKS Cluster Upgrades (Control Plane, Node Groups & Add-Ons)](#16-zero-downtime-eks-cluster-upgrades)
17. [High Availability & Resilience (Multi-AZ, TopologySpread & PDBs)](#17-high-availability--resilience)
18. [EKS CI/CD & GitOps Workflows (GitHub Actions, ECR & Argo CD)](#18-eks-cicd--gitops-workflows)
19. [EKS Cost & Performance Optimization (Karpenter Consolidation & Kubecost)](#19-eks-cost--performance-optimization)
20. [Complete End-to-End EKS Production Architecture Blueprint](#20-complete-end-to-end-eks-production-architecture-blueprint)

---

## 1. Executive Mental Model: The Enterprise EKS Production Blueprint

```
                              ENTERPRISE AMAZON EKS BLUEPRINT
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Control Plane  : Fully Managed Multi-AZ API Server + etcd (Private VPC Endpoint).   │
 │ 2. Data Plane     : Multi-AZ Managed Node Groups + Karpenter Auto-Provisioning (Spot). │
 │ 3. Networking     : Amazon VPC CNI with Prefix Delegation + AWS LB Controller (IP Mode)│
 │ 4. Identity & Auth: EKS Pod Identity / IRSA (Zero Static Keys) + EKS Access Entries.   │
 │ 5. Storage        : AWS EBS CSI (gp3) + AWS EFS CSI (RWX Shared) + StorageClasses.     │
 │ 6. Operations     : GitOps (Argo CD) + Amazon Managed Prometheus/Grafana + Kubecost.   │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. EKS Production Architecture

```
                                      AWS VPC (10.0.0.0/16)
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │  PUBLIC SUBNETS (10.0.1.0/24, 10.0.2.0/24)                                            │
 │  ├── Internet Gateway ──> Application Load Balancer (ALB)                              │
 │  └── NAT Gateways (AZ-A, AZ-B, AZ-C)                                                   │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │  MANAGED EKS CONTROL PLANE (Multi-AZ AWS Managed API Server & etcd Quorum)            │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │  PRIVATE SUBNETS (10.0.10.0/22, 10.0.20.0/22)                                          │
 │  ├── Managed Node Group AZ-A ├── Managed Node Group AZ-B ├── Managed Node Group AZ-C  │
 │  │   [ Worker Node 1 ]       │   [ Worker Node 2 ]       │   [ Worker Node 3 ]        │
 │  │   [ Pod A ] [ Pod B ]     │   [ Pod C ] [ Pod D ]     │   [ Pod E ] [ Pod F ]      │
 │  └── Karpenter Just-in-Time Auto-Provisioned Instances (Spot / Graviton)               │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │  DATA SUBNETS (10.0.100.0/24)                                                          │
 │  ├── Amazon Aurora PostgreSQL Multi-AZ ├── Amazon ElastiCache Redis Cluster            │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

### Public vs. Private API Server Endpoints
* **Public Endpoint:** Accessible from internet (Requires strict CIDR allow-lists).
* **Private Endpoint (Production Standard):** API server traffic remains entirely within your Amazon VPC; operators connect via corporate VPN, AWS Direct Connect, or SSM Bastion.

---

## 3. Building a Production EKS Cluster

```
                        MODERN EKS ACCESS MANAGEMENT EVOLUTION
  Legacy (Deprecated): aws-auth ConfigMap ──> Fragile, XML/YAML editing, risk of lockout.
  Modern (Standard)  : EKS Access Entries ──> Native AWS API, IAM mapped directly, auditable.
```

### Terraform Production Cluster Blueprint (`cluster.tf`)
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "production-eks-cluster"
  cluster_version = "1.30"

  cluster_endpoint_public_access  = false
  cluster_endpoint_private_access = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  # Enable modern Access Entries (No aws-auth ConfigMap)
  authentication_mode = "API_AND_CONFIG_MAP"

  # Core EKS Add-ons
  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent = true
      configuration_values = jsonencode({
        enableNetworkPolicy = "true"
        env = {
          ENABLE_PREFIX_DELEGATION = "true"
          WARM_PREFIX_TARGET       = "1"
        }
      })
    }
    aws-ebs-csi-driver = {
      most_recent = true
    }
  }

  eks_managed_node_groups = {
    general = {
      instance_types = ["m6i.large", "m6g.large"]
      min_size       = 3
      max_size       = 10
      desired_size   = 3
      capacity_type  = "ON_DEMAND"
    }
  }
}
```

---

## 4. EKS Networking Architecture

```
                       AMAZON VPC CNI WITH PREFIX DELEGATION
  Worker Node (EC2)
  ├── Primary ENI
  │   └── Secondary IPv4 Slots ──(Prefix Delegation: /28)──> 16 IPs per slot!
  │                                                          (Radically increases Pod density)
  └── Pods get native VPC Subnet IPs (10.0.10.x) ──> Zero NAT routing to AWS RDS/Services
```

### Prefix Delegation Benefits
* **Without Prefix Delegation:** A `t3.medium` can only run 17 Pods due to ENI/IP limits.
* **With Prefix Delegation (`ENABLE_PREFIX_DELEGATION=true`):** The same instance can run **110 Pods**, drastically reducing compute costs and preventing subnet IP fragmentation.

---

## 5. EKS Identity & Access

```
                          EKS ACCESS ENTRY AUTHENTICATION FLOW
  [ Operator / CI/CD ] ──(IAM Auth)──> [ EKS Access Entry ] ──(Maps to)──> [ K8s RBAC Group ]
                                                                                   │
                                                                                   ▼
                                                                       [ ClusterRole: Admin ]
```

### Predefined EKS Access Policies
1. `AmazonEKSClusterAdminPolicy`: Full cluster administrative privileges.
2. `AmazonEKSAdminPolicy`: Cluster admin without RBAC editing rights.
3. `AmazonEKSViewPolicy`: Read-only access to cluster resources.

---

## 6. IAM Roles for Service Accounts (IRSA) vs. EKS Pod Identity

```
                         IRSA VS. EKS POD IDENTITY COMPARISON
 ┌──────────────────────┬────────────────────────────────────────────────────────┬────────────────────────────────────────┐
 │ Feature              │ IRSA (IAM Roles for Service Accounts)                  │ EKS Pod Identity (Modern 2024+)        │
 ├──────────────────────┼────────────────────────────────────────────────────────┼────────────────────────────────────────┤
 │ **Mechanism**        │ OIDC Provider federation + JWT Mutating Webhook        │ EKS Auth DaemonSet + Node Agent API    │
 │ **Configuration**    │ Requires OIDC provider per cluster + IAM Trust Policy │ Single IAM Role linked to ServiceAcct  │
 │ **Cross-Cluster**    │ Complex trust policy with OIDC ARN                     │ Easy re-use across multiple clusters   │
 │ **Credential Scope** │ Temporary STS token mounted in Pod (`tmpfs`)           │ Temporary STS token provided by agent  │
 └──────────────────────┴────────────────────────────────────────────────────────┴────────────────────────────────────────┘
```

```bash
# EKS Pod Identity CLI Association Example
aws eks create-pod-identity-association \
  --cluster-name production-eks \
  --namespace production \
  --service-account payment-service-sa \
  --role-arn arn:aws:iam::123456789012:role/PaymentServiceRole
```

---

## 7. EKS Compute Strategy

```
                              EKS COMPUTE TRIAD
 ┌────────────────────────────────────┬────────────────────────────────────┬────────────────────────────────────┐
 │ 1. Managed Node Groups (MNG)       │ 2. AWS Fargate (Serverless)        │ 3. Karpenter Just-in-Time Nodes   │
 ├────────────────────────────────────┼────────────────────────────────────┼────────────────────────────────────┤
 │ AWS manages AMI lifecycle & drains │ VM-isolated per Pod; zero servers  │ Provisions right-sized EC2 fleets  │
 │ Best for: Core production tier     │ Best for: Batch jobs, PCI workloads│ Best for: Fast scaling & Spot save │
 └────────────────────────────────────┴────────────────────────────────────┴────────────────────────────────────┘
```

---

## 8. Next-Gen EKS Autoscaling: Karpenter vs. Cluster Autoscaler & HPA

```
                      KARPENTER JUST-IN-TIME NODE PROVISIONING
  [ Unscheduled Pending Pods (Requests: 3.5 CPU, 14GB RAM) ]
                               │
                               ▼
  [ Karpenter Controller (Evaluates NodePool Requirements) ]
                               │
                               ▼ (Direct EC2 Fleet API Call in ~30 seconds)
  [ Provisions 1x c6g.xlarge Spot Instance (4 vCPU, 16GB RAM) ]
                               │
                               ▼
  [ Pods Scheduled Immediately & Bin-Packed Optimally ]
```

### Karpenter NodePool Manifest (`nodepool.yaml`)
```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: general-purpose
spec:
  template:
    spec:
      requirements:
      - key: karpenter.sh/capacity-type
        operator: In
        values: ["spot", "on-demand"]
      - key: kubernetes.io/arch
        operator: In
        values: ["arm64", "amd64"]
      - key: karpenter.k8s.aws/instance-category
        operator: In
        values: ["c", "m", "r"]
      nodeClassRef:
        apiVersion: karpenter.k8s.aws/v1beta1
        kind: EC2NodeClass
        name: default
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: 720h # 30 days node rotation
```

---

## 9. Application Deployment on EKS

```
                           PRODUCTION EKS GITOPS LIFECYCLE
  [ Developer PR ] ──> [ Merge to Main ] ──> [ CI: Build & Push ECR ] ──> [ Update Git Manifests ]
                                                                                   │
                                                                                   ▼
                                                                        [ Argo CD Controller ]
                                                                                   │ (Auto-Reconcile)
                                                                                   ▼
                                                                        [ Multi-AZ EKS Cluster ]
```

---

## 10. EKS Load Balancing & Ingress

```
                        AWS LOAD BALANCER CONTROLLER (IP MODE)
  Internet Traffic ──> [ Application Load Balancer (ALB) ]
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼ (Direct VPC IP routing - Sub-millisecond) ▼
       [ Pod A (10.0.10.15:8080) ]               [ Pod B (10.0.20.22:8080) ]
```

### Hardened Ingress Manifest with AWS WAF & SSL Redirect
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/abc-123
    alb.ingress.kubernetes.io/wafv2-acl-arn: arn:aws:wafv2:us-east-1:123456789012:regional/webacl/prod-waf/xyz
    alb.ingress.kubernetes.io/healthcheck-path: /healthz
spec:
  rules:
  - host: api.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

---

## 11. EKS Storage Architecture

```
                          STORAGECLASS WITH WAITFORFIRSTCONSUMER
  [ PersistentVolumeClaim (50Gi gp3) ] ──> [ StorageClass (ebs.csi.aws.com) ]
                                                        │
                                    (Delays provisioning until Pod is placed!)
                                                        ▼
                            [ Bound in Exact AZ (e.g. us-east-1a) Where Pod Runs ]
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
```

---

## 12. EKS Security Hardening

```
                      EKS 4-TIER SECURITY HARDENING MODEL
 ┌───────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Private Cluster  : API server private endpoint only; bastion/VPN access.          │
 │ 2. Pod Security     : Enforce `Restricted` Pod Security Admission (PSA) standard.     │
 │ 3. Network Isolation: Default-deny NetworkPolicies + Security Groups per Pod.        │
 │ 4. Secret Security  : AWS Secrets Manager synced via External Secrets Operator (ESO). │
 └───────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. EKS Observability

```
                             OBSERVABILITY ARCHITECTURE
  EKS Cluster ──> [ AWS Distro for OpenTelemetry / Fluent Bit ]
                            │
         ┌──────────────────┼──────────────────┐
         ▼                  ▼                  ▼
  [ CloudWatch Logs ]  [ Amazon Managed ]  [ Amazon Managed ]
  (Audit & App Logs)   [ Prometheus (AMP) ][ Grafana (AMG)  ]
```

---

## 14. EKS Production Troubleshooting

```
Problem Detected on EKS
  │
  ├──> Pod is "Pending" ───────> Check Karpenter/CA logs & Subnet IP availability (`describe pod`)
  ├──> "CrashLoopBackOff" ────> Check IAM IRSA / Pod Identity permissions & `--previous` logs
  ├──> "OOMKilled" (Exit 137) ─> Container exceeded memory limit; right-size memory requests/limits
  └──> Node is "NotReady" ─────> Kubelet daemon crashed or node has disk/memory pressure
```

---

## 15. EKS Networking Troubleshooting

```bash
# 1. Check AWS VPC CNI DaemonSet status
kubectl get pods -n kube-system -l k8s-app=aws-node

# 2. Check available IP addresses per worker node
aws ec2 describe-network-interfaces --filters "Name=attachment.instance-id,Values=<instance-id>"

# 3. Verify CoreDNS resolution inside a pod
kubectl exec -it <pod-name> -n production -- nslookup kubernetes.default
```

---

## 16. Zero-Downtime EKS Cluster Upgrades

```
                         EKS UPGRADE 4-PHASE SEQUENCE
  [ 1. Review Lifecycle ] ──> [ 2. Upgrade Control Plane ] ──> [ 3. Upgrade Node Groups ] ──> [ 4. Upgrade Add-ons ]
```

### Upgrade Step-by-Step Runbook
1. **Validate Add-on Compatibility:** Ensure VPC CNI, CoreDNS, and kube-proxy support the target K8s version.
2. **Upgrade Control Plane:** Execute in AWS Console/Terraform (AWS upgrades API server with zero downtime).
3. **Upgrade Managed Node Groups:** Trigger rolling update with `maxUnavailable: 1` or launch new Node Group and gracefully drain old nodes (`kubectl drain <node> --ignore-daemonsets`).
4. **Upgrade Add-ons:** Update VPC CNI, CoreDNS, kube-proxy, and EBS CSI driver.

---

## 17. High Availability & Resilience

```yaml
# Enforce Multi-AZ Pod Distribution
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 6
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: api
```

---

## 18. EKS CI/CD & GitOps Workflows

* **Argo CD ApplicationSet:** Deploys identical microservices across multiple EKS clusters (Dev, Stage, Prod).
* **Automated Rollbacks:** If Prometheus/CloudWatch alerts fire during a canary rollout, Argo CD triggers instant rollback.

---

## 19. EKS Cost & Performance Optimization

```
                           EKS COST REDUCTION LEVERS
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Karpenter Consolidation : Automatically replaces underutilized nodes with smaller. │
 │ 2. Spot Instances (80-90%): Use Spot for stateless worker pods managed by Karpenter.  │
 │ 3. Graviton EC2 Instances  : Switch workloads from x86 to ARM64 for 20% cost savings.  │
 │ 4. Kubecost on EKS         : Track real-time Kubernetes cost allocation per namespace. │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 20. Complete End-to-End EKS Production Architecture Blueprint

```
                                COMPLETE EKS PRODUCTION FLOW
  [ User ] ──> [ Route 53 ] ──> [ AWS WAF ] ──> [ AWS ALB (IP Mode) ]
                                                       │
         ┌─────────────────────────────────────────────┴─────────────────────────────────────────────┐
         ▼                                                                                           ▼
  [ AZ-A Private Subnet ]                                                     [ AZ-B Private Subnet ]
  ├── Worker Node (Graviton Spot)                                             ├── Worker Node (Graviton Spot)
  │   ├── [ Frontend Pod ]                                                    │   ├── [ Frontend Pod ]
  │   └── [ Backend API Pod ] ──(EKS Pod Identity)──> [ S3 / SecretsManager ] │   └── [ Backend API Pod ]
  └── Persistent Storage: AWS EBS gp3 (WaitForFirstConsumer)                  └── Persistent Storage: AWS EBS gp3
         │                                                                                           │
         └─────────────────────────────────────────────┬─────────────────────────────────────────────┘
                                                       ▼
                                      [ Amazon Aurora PostgreSQL Multi-AZ ]
                                      [ Amazon ElastiCache Redis Cluster  ]
```
