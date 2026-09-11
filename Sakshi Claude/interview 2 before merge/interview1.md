# 🚀 DevOps End-to-End Project — Interview Notes

> A structured, simplified reference for explaining a real-world DevOps project in interviews.
> Covers 35+ topics spanning fresher to architect level: SCM/CI/CD/Deployment/Monitoring fundamentals, Kubernetes networking & troubleshooting, SonarQube, Terraform, Terraform+Ansible, career guidance, GitOps/ArgoCD, Cert-Manager, Linux/Shell/SQL, Ansible deep-dive, GitHub Actions (+Contexts), Helm, Docker, Jenkins scenarios, Blue-Green deployments, DevOps roadmap, domain-specific (Banking/Healthcare) questions, Cloud Migration, DevOps maturity/12-Factor/SRE concepts, Azure/AWS/Terraform/Kubernetes scenario-based Q&A (40+ real scenarios), SRE deep-dive, RBAC across Azure/AWS/K8s, OpenShift basics, and Architect-level Kubernetes internals

---

## 📌 Quick Overview: The 5-Step Project Narrative

| Step | Topic | What It Covers |
|------|-------|-----------------|
| 1 | Infrastructure Provisioning | Terraform (VPC, Subnets, EKS Cluster) |
| 2 | Source Code Management (SCM) | Branching strategy, team access, webhooks |
| 3 | CI/CD Pipeline | Jenkins Declarative Pipeline (build, test, scan, push) |
| 4 | Application Deployment | Kubernetes Manifests / Helm Charts on EKS |
| 5 | Monitoring & Logging | Prometheus, Grafana, EFK stack, alerting |

**Simple way to remember it:** *Build the land (Terraform) → Store the code (Git) → Automate the pipeline (Jenkins) → Ship the app (K8s/Helm) → Watch it (Prometheus/Grafana/EFK).*

---

## 🧩 Step 2: Fetch Code from SCM (Source Code Management)

### Repository Basics
```
https://github.com/mycompany-test/project.git
```
| Part | Meaning |
|------|---------|
| `github.com` | SCM (Source Code Management) platform |
| `mycompany-test` | Organization |
| `project.git` | Repository |

**In simple terms:** You need to know *where* the code lives, *who owns it* (org), and *what it's called* (repo). Teams are created in GitHub/GitLab and given specific access levels (Read, Triage, Write, Admin) based on role.

### 🌳 Branching Strategies (Very common interview question!)

#### Model A: Git Flow (best for large teams / scheduled releases)

| Branch | Purpose |
|--------|---------|
| `main` / `master` | Always production-ready code |
| `develop` | Integration branch — new features merge here first |
| `feature/*` | Built from `develop`, merged back via Pull Request after review |
| `release/*` | Cut from `develop` when preparing a release — final testing & bug fixes |
| `hotfix/*` | Branched from `main` to urgently fix production bugs, then merged into both `main` and `develop` |

**Simple analogy:** Think of `develop` as the kitchen where dishes (features) are prepared, `release` as final plating/QA, and `main` as what's actually served to customers. `hotfix` is the emergency fix when a served dish has a problem.

#### Model B: GitLab Flow / Environment-Based Branching (more flexible)

- **Long-lived branches** mapped to environments: `dev`, `staging/qa`, `production`
- **Short-lived branches** tied to issues/tasks: `issue-102-fix`, `feature/cart`
- These merge into environment branches through **automated promotion pipelines**

> 💬 **Interview tip:** If asked *"What branching strategy do you follow?"* — pick ONE model and describe it confidently. Git Flow = structured/enterprise. GitLab Flow = flexible/issue-driven.

### ⚙️ Handling Multi-Environment Deployments in Pipelines

**Parameterized Pipelines** — let you choose environment/branch at runtime instead of hardcoding it.

**Conditional stage execution using `when`:**
```groovy
stage('Deploy to Dev') {
    when { branch 'develop' }
    steps { ... }
}
stage('Deploy to Production') {
    when { branch 'main' }
    steps { ... }
}
```

**Continuous Delivery vs. Continuous Deployment** (commonly confused — know the difference!):

| Term | Meaning |
|------|---------|
| **Continuous Deployment** | Every passing build auto-deploys to production — no manual approval |
| **Continuous Delivery** (enterprise standard) | Auto-deploys up to staging/QA, but production needs a manual **Approval Gate** |

### 🔗 Webhook Integration (GitHub → Jenkins)

| Setting | Value |
|---------|-------|
| Payload URL | `http://<jenkins-url>/github-webhook/` |
| Content-Type | `application/json` |
| Trigger | "Just the push event" or select individual events (push, PR) |

> ⚠️ **Critical Tip:** Always include the trailing slash `/` in the webhook URL — missing it often causes HTTP 302/404 errors from Jenkins.

---

## 🛠️ Step 3: Setting Up the CI/CD Pipeline

### Example Parameterized Jenkins Pipeline (Groovy / Declarative DSL)
```groovy
pipeline {
    agent none

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        booleanParam(name: 'CLEAN', defaultValue: true, description: 'Clean before building')
        choice(name: 'ENVIRONMENT', choices: ['development', 'staging', 'production'], description: 'Choose deployment environment')
    }

    stages {
        stage('Checkout') {
            agent { label 'maven-builder' }
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/example/repo.git'
            }
        }

        stage('Clean') {
            when { expression { return params.CLEAN } }
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }

        stage('Build') {
            steps {
                echo "Building project for environment: ${params.ENVIRONMENT}"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT} environment..."
            }
        }
    }

    post {
        always {
            echo 'Cleaning up after build...'
        }
    }
}
```

### 🧠 Simple Breakdown of Parameters & Stages

| Parameter | Type | Purpose |
|-----------|------|---------|
| `BRANCH` | string | Which branch to build (default: `main`) |
| `CLEAN` | boolean | Whether to wipe the workspace before building |
| `ENVIRONMENT` | choice | Target environment: dev / staging / prod |

| Stage | Purpose |
|-------|---------|
| Checkout | Pulls code from the chosen branch |
| Clean | Deletes old workspace files if `CLEAN=true` |
| Build | Compiles/packages the app |
| Deploy | Ships it to the chosen environment |

### 🏗️ Full Production-Grade Pipeline Stages (real-world standard)

1. **Workspace Clean** — `cleanWs()` removes stale build artifacts
2. **Checkout SCM** — clone repo using branch + credentials
3. **Build & Package** — `mvn clean package` (Java) or `npm` (Node.js)
4. **Security & Quality Gates** — SonarQube (SAST) + OWASP Dependency-Check
5. **Containerization & Registry Push** — Docker image → Amazon ECR / Docker Hub
6. **Deployment & Verification** — deploy to Dev/QA/Staging/Prod on EKS/EC2
7. **Post Actions** — Slack/Teams/Email alerts, archive test reports

### 🧭 Node Allocation (Master-Agent Architecture)

> **Interview Q: "How do you ensure specific tasks run on designated nodes?"**

- `agent none` at the top level → prevents reserving an executor on the Jenkins **master/controller**
- Each stage binds to a specific **Agent Label**:

```groovy
pipeline {
    agent none
    stages {
        stage('Build & Test') {
            agent { label 'maven-builder' }
            steps { sh 'mvn clean package' }
        }
        stage('Security Scan') {
            agent { label 'security-agent' }
            steps { sh 'sonar-scanner' }
        }
    }
}
```

### 🔁 Pipeline Resilience Features

| Feature | Purpose |
|---------|---------|
| **Timeout** | `options { timeout(time: 30, unit: 'MINUTES') }` — kills stuck/zombie jobs |
| **Triggers** | `githubPush()` webhook or `pollSCM` scheduled polling |
| **Parallel Execution** | Run unit tests, integration tests, linting simultaneously to save time |
| **Masked Credentials** | `withCredentials([usernamePassword(...)])` keeps secrets out of console logs |

### 🏷️ Versioning Strategies

| Method | Example |
|--------|---------|
| Git Tags | `v1.4.0` |
| Commit Hash | `${GIT_COMMIT[0..7]}` |
| Build Number | `v1.0-${BUILD_NUMBER}` |
| Version File | `version.txt` read during build |

### 💾 Scheduled Backups (Jenkins)

1. Install backup plugin (e.g., ThinBackup)
2. Create backup directory: `/var/lib/jenkins/jenkinsbackup`
3. `chown -R jenkins:jenkins /backup-dir`
4. Schedule cron: weekly full backup + nightly differential backup (configs, jobs, credentials, plugin metadata)

---

## 📦 Step 4: Deploying the Application

### Method 1: Plain Kubernetes Manifest (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app-name
spec:
  replicas: 2
  selector:
    matchLabels:
      app: your-app-name
  template:
    metadata:
      labels:
        app: your-app-name
    spec:
      containers:
        - name: your-app-container
          image: your_ecr_repo/image_name:tag
          ports:
            - containerPort: 80
```

**Why Deployment instead of a raw Pod?**
> A raw Pod has no self-healing and no rolling update capability. A `Deployment` manages ReplicaSets, giving you **rolling updates, automated rollbacks, zero-downtime deploys, and scalability.**

Apply and verify:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/my-app
```

### Method 2: Helm Charts (Enterprise Standard)

**Why Helm?** Reusability, parameterization (no hardcoded values), and release tracking (easy rollback of an entire release).

**Setup:**
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm create my-app
```

**Chart structure:**
```
my-app/
├── Chart.yaml       # chart metadata (name, version, description)
├── values.yaml      # default configurable values
├── charts/          # dependency charts
├── templates/       # parameterized K8s manifests
└── .helmignore       # files to exclude when packaging
```

**`Chart.yaml`:**
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes to deploy my app
version: 1.0.0
```

**`values.yaml`:**
```yaml
replicaCount: 2

image:
  repository: your_ecr_repo/image_name
  pullPolicy: Always
  tag: latest

service:
  type: ClusterIP
  port: 80

resources: {}
```

**`templates/deployment.yaml`** (uses Helm templating to inject values):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: my-app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

**Deploy in CI/CD pipeline:**
```bash
helm upgrade --install my-app ./my-app \
  --namespace production \
  --set image.tag=${GIT_COMMIT[0..7]} \
  --wait
```

`helm upgrade --install` is important: it **creates** the release if it doesn't exist, or does an **incremental rolling upgrade** if it does.

**Verify:**
```bash
kubectl get pods -n production
```

### 🔀 Alternative Deployment Tools

| Platform | Tool |
|----------|------|
| AWS | CodeDeploy (uses `appspec.yml`, supports in-place or blue/green) |
| Azure | Azure App Services (via Azure Pipelines) |
| GitOps | ArgoCD — pull-based; continuously syncs cluster state with Git repo |

---

## 📊 Step 5: Monitoring and Logging

### Common Tools

| Category | Tools |
|----------|-------|
| Metrics + Dashboards | Prometheus + Grafana |
| Cloud-native | AWS CloudWatch, Azure Monitor, Google Stackdriver |
| Logging | ELK / EFK (Elasticsearch, Fluentd/Fluent Bit, Kibana) |
| SaaS platforms | Datadog, New Relic, Dynatrace, Nagios |

### 🆚 Metrics vs. Logs (know this distinction cold — common interview Q)

| | **Metrics** | **Logs** |
|---|---|---|
| Type | Time-series numeric data | Discrete, timestamped text events |
| Examples | CPU %, memory, pod restarts, HTTP request rate | API request hit, DB query, stack trace error |
| Best for | Trends, thresholds, dashboards | Root-cause / forensic debugging |

### a) Metrics: Prometheus + Grafana

**Install (Kubernetes via Helm):**
```bash
kubectl create ns monitoring
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```
> `kube-prometheus-stack` bundles the Prometheus Operator, node-exporter, and kube-state-metrics.

**Connect Grafana → Prometheus:**
- In Grafana: **Data Sources → Prometheus**
- Enter Prometheus server URL, e.g. `http://prometheus-server.monitoring.svc.cluster.local:9090`
- Build dashboards for CPU, memory, pod status, network throughput

### b) Logging: EFK Stack (Elasticsearch, Fluentd, Kibana)

```bash
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
```

- **Fluentd/Fluent Bit** runs as a **DaemonSet** on every node — tails `stdout`/`stderr` logs from `/var/log/containers/*.log`, enriches them with pod/namespace metadata, ships to Elasticsearch.

Sample Fluentd output config:
```
<match>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
  flush_interval 5s
</match>
```

- **Elasticsearch** indexes & stores the logs.
- **Kibana** is the UI to search/filter/visualize logs (query with Lucene/KQL, trace errors, inspect stack traces).

### c) Alerts & Notifications

**Prometheus alert rule example:**
```yaml
groups:
  - name: example-alerts
    rules:
      - alert: HighCPUUsage
        expr: sum(rate(container_cpu_usage_seconds_total{image!="", container!="POD"}[1m])) by (container) > 0.8
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container CPU usage is too high"
```

- **Alertmanager**: receives firing alerts from Prometheus, deduplicates/groups them, routes to **Slack, PagerDuty, or Email**.
- **Grafana Alerts**: can also be configured directly on dashboard panels.

### d) Architecture Diagram (simple mental model)
```
[ Target Pods / Nodes ]
        │ (exposes /metrics)
        ▼
[ Prometheus Server ] ──(scrapes)──► Stores in TSDB
        │
        ├──► [ Grafana ]        ──► Dashboards
        └──► [ Alertmanager ]   ──► Slack / PagerDuty / Email
```

### 🎯 Interview One-Liner Summary (Step 5)
> "We run `kube-prometheus-stack` in a dedicated `monitoring` namespace via Helm. Prometheus scrapes pod metrics, Grafana visualizes them, and Alertmanager routes high-severity alerts (e.g., CrashLoopBackOff) to Slack. For logs, Fluent Bit ships container logs to Elasticsearch/Kibana (or CloudWatch Container Insights), letting us debug without SSH access to nodes."

---

## 🕸️ Kubernetes Networking: Pod, Service & Headless Service Communication

### 1. Deployment vs. Pod (basics)

| Concept | Description |
|---------|--------------|
| **Deployment** | Manages the lifecycle of identical pods — replica count, update strategy, rollbacks |
| **Pod** | Smallest deployable unit; containers *inside the same pod* share a network namespace and can talk via `localhost` |

### 2. Kubernetes Service Types

| Type | Description |
|------|-------------|
| `ClusterIP` (default) | Exposes service **inside the cluster only** |
| `NodePort` | Exposes service on a fixed port on every node |
| `LoadBalancer` | Exposes service externally via cloud load balancer |
| `Headless` (`clusterIP: None`) | No virtual IP — DNS resolves directly to each individual pod |

### 3. 🎯 Headless Service — The Big Interview Topic

**The problem it solves:**
> A standard `ClusterIP` service load-balances traffic randomly/round-robin across pods via `kube-proxy`. This **breaks stateful systems** (databases, Kafka, etc.) where writes must go to a specific **primary/leader** pod, while reads can go to replicas. A normal service hides *which* pod you're hitting.

**The fix — Headless Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```
- No virtual IP is assigned.
- CoreDNS returns **direct A-records for each pod** instead of one shared ClusterIP.
- Best paired with a **StatefulSet**, which gives pods stable, predictable names (`db-0`, `db-1`, `db-2`) instead of random hashes.

**Resulting DNS pattern:**
```
<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local
```
Example:
- Primary: `db-0.my-db-headless.default.svc.cluster.local`
- Replica 1: `db-1.my-db-headless.default.svc.cluster.local`
- Replica 2: `db-2.my-db-headless.default.svc.cluster.local`

### 4. Full Example: StatefulSet DB + App Deployment

**Database (StatefulSet + Headless Service):**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-db
spec:
  serviceName: "my-db-headless"
  replicas: 3
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          ports:
            - containerPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: my-db
  ports:
    - port: 5432
```

**Application Deployment (connects to specific DB pod):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          env:
            - name: DATABASE_URL
              value: "postgresql://postgres@my-db-headless.default.svc.cluster.local:5432/mydb"
```

### 5. Standard Service vs. Headless Service — Comparison Table

| Feature | Standard Service (ClusterIP) | Headless Service (`clusterIP: None`) |
|---------|-------------------------------|----------------------------------------|
| Cluster IP | Allocated from service CIDR | None |
| Routing/Proxying | Handled by `kube-proxy` (iptables/IPVS) | Bypasses `kube-proxy` — client resolves DNS directly |
| DNS Resolution | Returns single virtual ClusterIP | Returns multiple A-records / per-pod FQDNs |
| Best for | Stateless web apps, REST APIs, microservices | Stateful workloads: DB leaders/replicas, Kafka brokers, Elasticsearch, Cassandra |

---

## 🌐 General Pod-to-Pod Communication & Traffic Routing

### Communication Scopes

| Scope | How They Talk |
|-------|----------------|
| **Same Pod** (container-to-container) | Directly via `localhost` (shared network namespace) |
| **Same Namespace** (pod-to-pod) | Via short service DNS: `http://backend-service:8080` |
| **Cross-Namespace** | Via full FQDN: `backend-service.backend.svc.cluster.local` |

### How Traffic Actually Gets Forwarded

1. A pod sends a request to a Service's DNS name (e.g., `backend-service.default.svc.cluster.local`)
2. **CoreDNS** resolves this to the Service's **ClusterIP**
3. The Service tracks matching pods via **Endpoints / EndpointSlice** (based on label selectors)
4. **kube-proxy** (on each node) intercepts traffic and forwards it to one of the healthy backend pods — typically **round-robin** via iptables/IPVS

```
Request 1 → Pod A
Request 2 → Pod B
Request 3 → Pod C
Request 4 → Pod A  (cycle repeats)
```

### Special Routing Modes

| Mode | Use Case |
|------|----------|
| **Headless Service** (`clusterIP: None`) | Direct, deterministic pod addressing — bypasses `kube-proxy` load balancing (stateful DBs) |
| **Ingress Controller** | Layer-7 routing for external HTTP/HTTPS traffic — host-based (`api.example.com`) or path-based (`example.com/api`) rules |
| **Egress Policies** | Network policies restricting traffic *leaving* the cluster (to external APIs, DBs, etc.) |

### 🎯 Interview One-Liner Summary (Networking)
> "Communication flows through a decoupled service abstraction. Containers in the same pod talk via `localhost`. Cross-pod calls target a Service's DNS name, which CoreDNS resolves to a ClusterIP; `kube-proxy` then round-robins the request to one of the matching pod IPs in the EndpointSlice. For stateful workloads needing direct pod targeting — like database replicas — we use a Headless Service (`clusterIP: None`) to bypass the proxy and expose deterministic per-pod DNS records."

---

## 🔍 SonarQube Interview Prep: Code Smells vs. Bugs vs. Vulnerabilities

### 1. The Core Comparison

| Category | Primary Impact | Definition | What Happens to the System |
|----------|------------------|------------|------------------------------|
| **Code Smell** | Maintainability & Readability | Sub-optimal design or poor practices that hurt long-term maintainability, reusability, extensibility | Program still works and gives correct output, but code is messy, fragile, hard to refactor |
| **Bug** | Reliability & Correctness | Flaws, runtime exceptions, or logic errors that stop code from working as designed | Program crashes, throws runtime exceptions, or produces wrong calculations |
| **Vulnerability** | Security & Integrity | Security flaws / exposed entry points that attackers can exploit | System may run fine, but sensitive data or access is exposed to exploitation |

**Simple way to remember it:**
> 🧹 Code Smell = "it works, but it's ugly and risky to maintain."
> 🐞 Bug = "it's broken."
> 🔓 Vulnerability = "it's a door left open for attackers."

---

### 2. 🧹 Code Smells — Real-World Examples & Fixes

| Smell | Problem | Fix |
|-------|---------|-----|
| **Hardcoded Values & Endpoints** | Static values/URLs/config baked directly into source code — any change needs a recompile, CI rebuild, and redeploy | Externalize into environment variables, config files (`application.properties` / `config.yaml`), or parameter stores |
| **Deeply Nested Control Logic** (Arrow Anti-Pattern) | Multiple nested `if/else` layers hurt readability and make testing harder | Use guard clauses, early returns, or polymorphism |
| **Bloated Functions** (violates Single Responsibility) | One function doing too much — parsing, business logic, DB persistence, notifications all in one place | Break into small, modular, reusable helper functions |
| **Duplicated Code** (copy-paste anti-pattern) | Same logic repeated in multiple places — bug fixes need to happen in every copy | Extract shared logic into reusable methods / shared library modules |

---

### 3. 🐞 Bugs — Real-World Examples

| Bug Type | Example |
|----------|---------|
| **Off-by-One / Index Out of Bounds** | Array of size 4 (valid indices `0`–`3`) accessed at index `4` → `ArrayIndexOutOfBoundsException` |
| **Divide-by-Zero** | Performing division without checking if the denominator is `0` |
| **Scope & Uninitialized Variables** | Declaring a variable inside an inner block, then referencing it outside that block → compilation failure or `NullPointerException` |

---

### 4. 🔓 Vulnerabilities — Real-World Examples & Remediations

| Vulnerability | Example | Remediation |
|----------------|---------|--------------|
| **Hardcoded Credentials & DB Secrets** | Passing credentials in cleartext (`-u username -p password`) in scripts/config files | Inject secrets at runtime using HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault |
| **SQL Injection (SQLi)** | Building SQL queries via string concatenation with unsanitized input: `SELECT * FROM users WHERE id = ' + input + '` | Use parameterized queries / `PreparedStatement`s and ORM frameworks with proper input sanitization |
| **Broken Authentication & Input Validation** | Processing sensitive API requests without verifying requester identity/domain (e.g., accepting any email/domain without token check) | Enforce OAuth2/JWT verification, strict RBAC, and origin whitelisting on every endpoint |

---

### 5. 🎯 Interview One-Liner: How Do You Enforce This Across Teams?

> "In our CI/CD pipeline, every Pull Request triggers a Maven build coupled with a SonarQube scan. We configure strict **Quality Gates**: any PR that introduces Blocker/Critical Bugs, Security Vulnerabilities, or drops Code Coverage below 80% automatically **fails the Quality Gate**. This blocks the pipeline from generating container artifacts or advancing to staging — forcing developers to resolve issues before merging into `develop`."

---

## 🧱 Terraform Modules — Interview Prep

### 1. Why Use Terraform Modules?

| Reason | Simple Explanation |
|--------|----------------------|
| **Reusability & DRY** | Define infrastructure patterns (VPC, EKS, etc.) **once**, reuse them across Dev/QA/Staging/Prod instead of copy-pasting HCL code |
| **Separation of Concerns** | Keep networking (VPC), cluster control-plane (EKS), and compute (worker nodes) in separate, decoupled directories |

### 2. Standard Enterprise Directory Layout

```
terraform-root/
├── main.tf                 # Root orchestration calling child modules
├── variables.tf            # Global root input variables
├── outputs.tf              # Aggregated root outputs
├── providers.tf            # AWS provider and Terraform version constraints
└── modules/
    ├── vpc/                # Networking module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── eks/                # EKS Cluster control-plane module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── ec2_node/           # Managed Node Groups / Worker compute module
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 3. Child Module Construction & Dependencies

**A. VPC Module (`modules/vpc`)**
- Resources: `aws_vpc` (DNS hostnames/support enabled), public/private subnets across multiple AZs, `aws_internet_gateway`, `aws_nat_gateway` with Elastic IPs
- Exports its IDs so parent modules can use them:
```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

**B. EKS Cluster Module (`modules/eks`)**
- Resources: IAM Cluster Role (with `AmazonEKSClusterPolicy`) + `aws_eks_cluster`
- **Inputs:** accepts `vpc_id` and `subnet_ids` instead of hardcoding them
- **Outputs:** `cluster_name`, `cluster_endpoint`, `cluster_certificate_authority_data`

**C. Worker Node Group Module (`modules/ec2_node`)**
- Resources: Node IAM roles (`AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`) + `aws_eks_node_group`
- **Dynamic Scaling:** controlled via `min_size`, `max_size`, `desired_size` input variables — not fixed counts

### 4. Root Module: Sourcing & Output Chaining

```hcl
# 1. Instantiate VPC Module
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  env    = var.environment
}

# 2. Instantiate EKS Module using VPC Outputs
module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id              # Inter-module reference
  subnet_ids = module.vpc.private_subnet_ids  # Inter-module reference
}

# 3. Instantiate Worker Node Module using EKS & VPC Outputs
module "ec2_node" {
  source       = "./modules/ec2_node"
  cluster_name = module.eks.cluster_name
  subnet_ids   = module.vpc.private_subnet_ids
  node_count   = var.worker_node_count
}
```

**Module source types:**

| Type | Example | When to Use |
|------|---------|-------------|
| Relative local path | `source = "./modules/vpc"` | Modules live in the same repo |
| Remote Git repo | `source = "git::https://github.com/org/terraform-aws-vpc.git?ref=v1.2.0"` | Enterprise standard — version pinning & cross-repo sharing |

### 5. Essential Interview Q&A

**Q: "How do you pass data between modules?"**
> A child module's internal resources are private by default. It must explicitly expose an attribute via `outputs.tf`. The root module then references it as `module.<module_name>.<output_name>`.

**Q: "Why avoid hardcoding values inside child modules?"**
> Hardcoding breaks modularity and reusability across environments. All environment-specific values (instance types, CIDRs, replica counts) must be surfaced through `variables.tf`.

---

## 🔗 Terraform + Ansible: How They Work Together

**Simple mental model:** *Terraform builds the house (infrastructure). Ansible furnishes and configures it (software/services).*

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure-as-Code — provisions VMs, VPCs, databases declaratively (HCL) |
| **Ansible** | Configuration management — connects via SSH/WinRM to install packages, configure services, deploy apps |

### Typical Workflow
1. **Terraform provisions infrastructure** (EC2s, VPCs, DBs) on any cloud
2. **Ansible configures the software** on that infrastructure (web server install, DB setup, etc.)
3. **Linking them:** Terraform `outputs` (IPs, instance IDs) feed into Ansible's inventory so it knows *what* to configure

### Example Repository Structure
```
project/
├── terraform/
│   ├── main.tf               # Provisioning config
│   ├── variables.tf
│   ├── outputs.tf             # e.g., IP addresses, instance IDs
│   ├── provider.tf
│   └── terraform.tfvars       # Environment-specific values
├── ansible/
│   ├── inventory/
│   │   ├── prod.yaml
│   │   ├── dev.yaml
│   │   └── staging.yaml
│   ├── playbooks/
│   │   ├── setup.yaml
│   │   └── deploy.yaml
│   ├── roles/
│   │   ├── webserver/
│   │   ├── db/
│   │   └── app/
│   └── ansible.cfg
└── scripts/
    └── deploy.sh              # Orchestrates Terraform + Ansible
```

### How Ansible Gets Data From Terraform

**Option A — Manual output passing:**
```bash
terraform init
terraform apply

export instance_ip=$(terraform output -raw instance_ip)
ansible-playbook -i ${instance_ip}, playbooks/setup.yaml
```

**Option B — Dynamic Inventory (preferred at scale):**
A script (Python/Shell) queries Terraform's state file and auto-generates a live inventory for Ansible — no manual copy-pasting of IPs.

```python
#!/usr/bin/env python
import json, subprocess

def get_terraform_output():
    output = subprocess.check_output(['terraform', 'output', '-json'], universal_newlines=True)
    return json.loads(output)

def main():
    inventory = {'all': {'hosts': []}}
    tf_output = get_terraform_output()
    for resource in tf_output['instances']['value']:
        inventory['all']['hosts'].append(resource['public_ip'])
    print(json.dumps(inventory, indent=2))

if __name__ == "__main__":
    main()
```

### Handling Multiple Environments

- **Terraform side:** use separate **workspaces** (`prod`, `staging`, `dev`)
- **Ansible side:** use separate **inventory files** (`prod.yaml`, `dev.yaml`, `staging.yaml`) — or dynamic scripts pulling from each workspace's state

```bash
terraform workspace select prod
terraform apply

ansible-playbook -i inventory/prod.yaml playbooks/setup.yaml
```

### Running It All in a CI/CD Pipeline

```yaml
stages:
  - terraform
  - ansible
  - deploy

variables:
  TF_VAR_environment: "dev"

terraform:
  stage: terraform
  script:
    - terraform init
    - terraform plan
    - terraform apply -auto-approve

ansible:
  stage: ansible
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/setup.yaml

deploy:
  stage: deploy
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/deploy.yaml
```

**How often do these run?**
| Tool | Frequency |
|------|-----------|
| **Terraform** | Weekly, or whenever infrastructure changes are needed |
| **Ansible** | On every commit / pull request — to deploy & configure services |

---

## 🌱 Career Transition Guide: Switching Into DevOps

### Two Practical Routes

| Route | Approach |
|-------|----------|
| **A. Internal Transition** (fastest legitimacy) | Ask your manager for release/partial allocation to an internal DevOps/Cloud project. Even 2–4 months of cross-skilling gives you real, defensible production context for your resume |
| **B. Self-Learning / External Switch** | If internal mobility is blocked, don't wait indefinitely. Use hands-on labs (e.g., KodeKloud), AWS/Azure free tier, and self-host tools (EC2, Jenkins, Docker, Minikube/kind/EKS) to build end-to-end projects |

### The 3 Core Interview Questions Career-Switchers Must Nail

1. **"What are your exact roles and responsibilities?"**
   → Anchor answers around tools you're actually solid in (pipelines, infra provisioning, or scripting)
2. **"What does a typical day look like for you?"**
   → Structure a realistic day: standup → monitoring alerts → root-cause investigation → building automation
3. **"What tools and cloud technologies do you own?"**
   → Be specific about your stack, don't generalize

### High-Priority Tools to Focus On

| Area | Tools |
|------|-------|
| Kubernetes Orchestration | EKS/AKS, Pods, Deployments, Services, Helm |
| CI/CD & Containerization | Jenkins / GitHub Actions / GitLab CI, multi-stage Dockerfiles, ECR |
| DevSecOps | SonarQube (SAST), OWASP Dependency-Check (SCA), HashiCorp Vault / AWS Secrets Manager |
| IaC & Scripting | Modular Terraform, Shell/Python automation |

### Overcoming Imposter Syndrome
> Every engineer faces a ramp-up curve on a new codebase/infra/team. The effort you put into labs and troubleshooting translates directly to job performance. After **30–60 days**, the rhythm becomes natural.

---

## ⚡ Rapid-Fire Interview Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | How do you rollback to a previous build in Jenkins? | Go to job → Build History → select previous build → click **Rebuild** (or download/redeploy that build's artifact) |
| 2 | How do you connect to a Kubernetes cluster? | Install `kubectl`; ensure `~/.kube/config` is present and configured |
| 3 | What's needed in an Ingress to route traffic? | Install an Ingress Controller → create Ingress resource (host-based or path-based rules) → create a DNS record pointing to the controller's IP |
| 4 | How do you troubleshoot High CPU Utilization? | Use `top` to find the offending process; kill it if stuck; check background processes |
| 5 | How many Jenkins slave nodes? | Example answer: 1 master, 3 slave nodes |
| 6 | How many services run in your project? | Clarify: Kubernetes Services vs. Docker services |
| 7 | What Jenkins pipeline types have you used? | Scripted pipelines and Multibranch pipelines (e.g., "5–6 pipeline scripts created and maintained") |
| 8 | How to run a command in the background? | `nohup command &` |
| 9 | How do you import a manually-created resource into Terraform? | `terraform import aws_instance.example i-0abcd1234efgh5678` (after defining the matching resource block) |
| 10 | Difference between load balancer types? | **Layer 4** = TCP/UDP | **Layer 7** = HTTP/HTTPS |

---

## 📦 Artifact Promotion: How Deployment to Production Really Works

**The interview dilemma:** *Is the exact same tested artifact promoted to production, or does a new pipeline rebuild it from `main`?*

### Approach A: Rebuild Per Target Pipeline
- Artifact is built & tested during Dev/QA pipeline runs
- On merge to `main`, a **separate production pipeline** rebuilds the artifact from that branch (same version tag/commit SHA)
- Before touching prod, it's deployed to a staging/QA-mirroring environment for smoke/regression tests
- A **manual approval gate** (sign-off) is required before going live

### Approach B: "Build Once, Promote Everywhere" (12-Factor Standard) ✅ Industry preferred
- An **immutable artifact** (Docker image / `.jar`) is built **only once**, from the commit SHA
- The **same image digest** flows through Dev → QA/Staging → Production — **never rebuilt**
- Environment differences are handled purely through **externalized config** (ConfigMaps, Secrets, env vars)

| | Rebuild Per Pipeline | Build Once, Promote Everywhere |
|---|---|---|
| Risk | Possible drift between builds | No drift — same binary everywhere |
| Speed | Slower (redundant builds) | Faster |
| Industry standard? | Sometimes used | ✅ Preferred / 12-Factor standard |

**QA/Pre-Prod parity matters:** Staging must mirror production (networking, DB engines, ingress rules) — otherwise config drift can hide environment-specific failures.

### 🎯 Interview One-Liner
> "We build the Docker image once in CI, tag it with the immutable Git commit SHA, and push it to ECR. That exact image digest is deployed to QA, passes integration tests, and is promoted to Production behind a manual approval gate — only ConfigMaps and Secrets change between environments."

---

## ⚙️ Event-Driven Architecture in DevOps

**Definition:** Actions/workflows triggered automatically by *events* rather than manual steps or fixed schedules.

### DevOps Use Cases

| Trigger Event | Automated Response |
|----------------|----------------------|
| Code pushed to Git | CI/CD pipeline starts automatically (test → scan → package → deploy) |
| High traffic load | Auto-Scaling Group / Kubernetes HPA launches new instances/pods |
| Failed deployment | Automatic rollback to the previous stable version |
| System outage / high error rate | Alerts fire + automated remediation (e.g., service restart) |

### How to Defend "Event-Driven Architecture" on Your Resume
| Mechanism | Example |
|-----------|---------|
| CI/CD Triggers | Git webhook → Jenkins/GitHub Actions run instantly on push/PR |
| Dynamic Autoscaling | CloudWatch/Prometheus threshold → triggers ASG or K8s HPA/Karpenter |
| Self-Healing & Rollback | Health-check failure → automated rollback, container restart, or traffic shift at LB level |

---

## 🧪 QA Pipeline Ownership & Multi-Environment Pipeline Design

**Q: Who builds the QA CI/CD pipeline — QA testers or DevOps engineers?**
> The **DevOps engineer** designs, provisions, and maintains the pipeline infrastructure/code. The **QA team** contributes automated test scripts (Selenium/Cypress/Playwright) that run *inside* a pipeline stage.

**Standard QA Pipeline Flow:**
1. Git checkout of testable branch/tag
2. Build/compile the app
3. Static analysis & security scan (SonarQube)
4. Automated integration/end-to-end tests
5. Publish test reports & notify build status

**Q: Do we rebuild the Docker image in QA and Prod, or reuse it?**
> **Reuse it.** Build once, tag with commit SHA, push to registry (ECR/Nexus/Harbor), and deploy that exact same digest everywhere. Rebuilding risks dependency/OS drift.

**Q: In a single parameterized pipeline, do all stages run for every environment?**
> No — use the `when` directive to conditionally run/skip stages based on the environment parameter:
```groovy
stage('Deploy to Production') {
    when {
        expression { params.ENVIRONMENT == 'Production' }
    }
    steps {
        input message: 'Approve production deployment?'
        sh 'helm upgrade --install prod-release ./charts/app'
    }
}
```

### Real-World Linux Automation You Should Be Ready to Discuss
- **Cost optimization:** scripts to shut down non-prod servers outside business hours, restart before workday
- **Mass maintenance:** rolling restarts across many servers, verifying health checks before moving to the next node
- **Log rotation / disk space:** cron jobs detecting `/var/log` usage > 85%, archiving to S3, purging stale caches
- **CrashLoopBackOff / OOM remediation:** scripts that triage stuck pods, extract exit codes (e.g., `OOMKilled` / Exit Code 137), dump logs, notify on-call

---

## 📜 Most Important Git Commands (Cheat Sheet)

| # | Command | What It Does |
|---|---------|----------------|
| 1 | `git init` | Creates 3 areas: working, staging, local repo |
| 2 | `git status` | Shows file status |
| 3 | `git add .` | Stages all files from working area |
| 4 | `git add filename1 filename2` | Stages specific files |
| 5 | `git add *.sh` | Stages all `.sh` files |
| 6 | `git commit` | Moves staged files into local repo |
| 7 | `git commit -m "My commit"` | Commit with a message |
| 8 | `git config --global user.name ""` | Set global username |
| 9 | `git config --global user.email ""` | Set global email |
| 10 | `git commit -m "My Commit" filename` | Commit a specific file |
| 11 | `git remote add aliasname "https://github.com/..."` | Add a remote repo |
| 12 | `git remote -v` | List all mapped remotes |
| 13 | `git push aliasname master` | Push a branch to remote |
| 14 | `git log -2` | Show last 2 commit IDs |
| 15 | `git show commit_id` | Show files changed in a commit |
| 16 | `git reset` | Move files from staging back to working area |
| 17 | `git revert` | Revert the last commit (locally) |
| 18 | `git pull aliasname master` | Pull changes into working area |
| 19 | `git clean` | Remove newly created untracked files |
| 20 | `git branch branchname` | Create a branch |
| 21 | `git checkout branchname` | Switch to a branch |
| 22 | `git branch -a` | List all branches |
| 23 | `git merge branchname` | Merge a branch into current branch |
| 24 | `git diff branchname` | Show changes vs. a branch |
| 25 | `git push aliasname --all` | Push all branches |
| 26 | `git tag tagname` | Create a tag |
| 27 | `git push aliasname tag tagname` | Push a tag to remote |
| 28 | `git stash apply stash@{1}` | Apply a specific stashed change |
| 29 | `git cherry-pick commit_id` | Apply a specific commit onto current branch |
| 30 | `git fetch aliasname branchname` | Fetch changes into local repo (no merge) |
| 31 | `git clone "https://github.com/..."` | Clone an entire repo |
| 32 | `git rebase branchname` | Reapply commits on top of another branch |

---

## 🐙 ArgoCD Interview Prep

### 1. What is ArgoCD & GitOps?
**GitOps core principle:** Git is the **single source of truth** for both app code and Kubernetes infra state.

**Reconciliation Loop:**
- ArgoCD continuously compares the **Desired State** (Git manifests/Helm/Kustomize) with the **Live State** (what's running in the cluster)
- A mismatch = **OutOfSync**

| Sync Mode | Behavior |
|-----------|----------|
| **Automated Sync** | Auto-applies changes to restore parity |
| **Manual Sync** | Flags drift in UI/CLI, waits for operator approval |

**CI/CD split of labor:**
- **CI tools** (Jenkins/GitHub Actions) → build, test, scan, update the image tag in Git
- **ArgoCD** → pull-based **CD**, deploying inside the cluster without exposing cluster credentials to the CI server

### 2. ArgoCD Architecture

```
                  ┌──────────────────────┐
                  │    Git Repository     │
                  │ (Helm/Kustomize/YAML) │
                  └──────────▲───────────┘
                             │
                  ┌──────────┴───────────┐
                  │   Repository Server   │ ── Parses manifests & renders templates
                  └──────────▲───────────┘
                             │
┌──────────────┐  ┌──────────┴───────────┐  ┌──────────────┐
│  Web UI/CLI  │◄─┤      API Server       │◄─┤  Redis Cache │
└──────────────┘  └──────────┬───────────┘  └──────────────┘
                             │
                  ┌──────────▼───────────┐
                  │ Application Controller│ ── Compares Live vs. Desired state & syncs
                  └──────────┬───────────┘
                             ▼
                  [ Target K8s Cluster ]
```

| Component | Role |
|-----------|------|
| **API Server** | Handles UI/CLI/CI requests, auth, RBAC |
| **Repository Server** | Clones Git repos, renders manifests (YAML/Helm/Kustomize) |
| **Application Controller** | Continuously reconciles live vs. desired state, performs sync |
| **Redis Cache** | Speeds up state comparisons & session management |
| **Web UI / CLI** | Visualize topologies, sync status, logs, manual approvals |

### 3. Rollbacks & Versioning
| Method | How It Works |
|--------|----------------|
| **Git-native rollback** (preferred) | `git revert <commit-hash>` → ArgoCD auto-detects and rolls the cluster back |
| **UI/CLI rollback** | `argocd app rollback <app-name>` — revert to a previous deployment revision |
| **Self-healing & auto-pruning** | Reverts manual `kubectl edit` drift; prunes resources removed from Git |

### 4. Helm & Kustomize Support
- **Helm:** ArgoCD tracks a Git folder with `Chart.yaml`/`values.yaml` or a Helm repo, renders templates with value overrides
- **Kustomize:** native support for environment overlays (`overlays/dev`, `overlays/prod`) — no separate chart packaging needed

### 5. ApplicationSets & Multi-Cluster
- **ApplicationSets:** dynamically generate many ArgoCD `Application` resources from one template — e.g., deploy the same app across 50 regional clusters or multiple stages without repetitive YAML
- **Multi-cluster setup:**
  1. `argocd cluster add <kubecontext>` to register a remote cluster
  2. Specify target cluster in the Application manifest's `spec.destination`

### 6. Security Best Practices
- RBAC (Role-Based Access Control)
- SAML/OIDC Authentication
- Secure Git access
- Application Secrets management
- Enable Audit Logs

### 7. ArgoCD vs. Flux — Quick Comparison

| Dimension | ArgoCD | Flux CD |
|-----------|--------|---------|
| Architecture | Centralized server, rich Web UI, SSO, granular RBAC | Decentralized, lightweight Kubernetes controllers |
| User Interface | Rich interactive Web UI with live resource trees | CLI-centric; relies on third-party UIs (e.g., Weave GitOps) |
| Best Fit | Complex enterprise setups, multi-tenant teams | Minimalist, headless environments |
| Helm Support | Advanced, built-in | Requires more setup (Helm Controller) |

---

## 🌐 Ingress vs. Load Balancer, CronJobs, Multi-Cloud & Migration

### 1. Ingress vs. Cloud Load Balancer — Which to Use?

| Option | Cost/Behavior | Best For |
|--------|-----------------|----------|
| `Service type: LoadBalancer` | Allocates a **dedicated** cloud LB per exposed service → cost scales linearly | Raw TCP/UDP, non-HTTP protocols, low-latency websockets |
| **Ingress Controller** | **One** entry point (one ALB/VIP) multiplexes traffic across many services via path-based (`/api`) or host-based (`app.domain.com`) rules | Standard HTTP/HTTPS web & API routing — much more cost-effective |

### 2. Kubernetes CronJobs

Runs temporary batch pods on a recurring schedule (`Minute | Hour | Day-of-Month | Month | Day-of-Week`):

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup-cronjob
spec:
  schedule: "0 2 * * *"   # Daily at 02:00 UTC
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup-task
              image: backup-utility:1.2
```
**Use cases:** DB snapshot exports, cert-manager validation runs, log archiving, cache warming.

### 3. Multi-Cloud CI/CD Challenges

| Pain Point | Why It's Hard |
|------------|-----------------|
| Incompatible Cloud APIs & IAM | AWS IAM Roles vs. Azure Managed Identities vs. GCP Service Accounts all differ |
| Complex cross-cloud networking | Needs IPsec VPNs, transit gateways, or private interconnects (Direct Connect / ExpressRoute) |
| Tooling fragmentation | Pipelines must juggle `aws-cli`, `az`, `gcloud`, and different secret/storage backends |

**Recommended pattern:** Use **GitOps (ArgoCD)** running *inside* each target cluster, pulling from a centralized Git repo — avoids brittle direct cross-cloud API calls from the CI server.

### 4. Cloud Migration Strategy (e.g., Azure → AWS)

1. **Assessment:** inventory stateful data, DBs, compute dependencies
2. **Database migration:** use managed replication (e.g., **AWS DMS**) for continuous schema/data sync with minimal downtime
3. **Compute/storage replication:** use **AWS MGN** — install replication agents on source VMs for continuous block-level sync
4. **Validation & cutover:** validate data parity, smoke test in staging, shift traffic via **Route 53 weighted records**, decommission legacy instances

### 5. Common Real-World Jenkins Failures

| Failure | Root Cause | Fix |
|---------|------------|-----|
| **Agent Offline** | SSH credential expiry, network/firewall drop, JVM/Java version mismatch | Renew credentials, check connectivity, align Java versions |
| **Hanging/Stuck Builds** | Thread deadlocks, exhausted executors, unhandled interactive prompts (e.g., missing `-auto-approve`), no timeout | Add `options { timeout(...) }`, use `cleanWs()` |
| **Disk Full** | Stale Maven `~/.m2` deps, old logs, untagged Docker images | Schedule `docker system prune -af`, enable workspace discards |
| **Broken Plugins/Dependencies** | Jenkins core upgraded without checking plugin compatibility, Docker Hub rate limits | Check compatibility matrix before upgrading; use authenticated pulls |

---

## 🗓️ A Day in the Life of a DevOps Engineer (Interview-Ready Answer)

### Teams a DevOps Engineer Interfaces With
- Core Product Development teams (feature devs, CI consumers)
- Internal Platform/Automation teams (tooling, alert automation)
- Dev & QA Infrastructure teams
- Customer & Production Operations teams
- SRE / Support (incident response, on-call, telemetry)

### Sample Daily Schedule
```
[ 09:00 - 09:30 ]  ──► System checks, monitoring review, email alerts, Jira backlog
[ 09:30 - 10:00 ]  ──► Daily Standup (yesterday / today / blockers)
[ 10:00 - 13:00 ]  ──► P0/P1 priority work (pipeline debugging, hotfixes, unblocking devs)
[ 14:00 - 17:00 ]  ──► Core project execution (Terraform modules, CI/CD refactoring, scripts)
[ 17:00 - 18:00 ]  ──► Cross-team syncs, documentation, runbook updates
```

### Role Rotation in Mature Teams
| Role | Focus |
|------|-------|
| Pipeline Engineers | Jenkinsfiles/GitHub Actions, build agents, Docker cache, quality gates |
| Infra/Automation Engineers | Terraform, Helm charts, Python/Bash automation |
| Production Support | Monitoring live deployments, CrashLoopBackOff/OOM triage, assisting releases |
| Knowledge Transfer | Peer reviews, pairing, shadowing for coverage during leave |

### The "Automate Recurring Issues" Principle
> **Rule of thumb:** If an issue happens once, document it. If it happens twice, **automate the fix** (e.g., automated pod restarts, disk cleanup jobs, log archiving).

### 🎯 Interview One-Liner
> "My day starts with reviewing monitoring alerts and our Jira board for high-priority blockers. During standup, I align with developers on sprint deliverables. The core of my day splits between project automation — writing Terraform modules, optimizing CI/CD stages, refining Helm templates — and platform maintenance, like investigating failed pipelines and rotating cluster secrets. For recurring incidents, I turn manual fixes into automated scripts and update our runbooks."

---

## 🔒 Cert-Manager: Automating SSL/TLS in Kubernetes

**What it does:** Integrates with Kubernetes to **create, manage, and auto-renew** SSL/TLS certificates.

### How It Works
1. **Install Cert-Manager** into the cluster (deploys controllers + CRDs)
2. **Issuer / ClusterIssuer:** defines *how* certificates are obtained (Let's Encrypt, Vault, internal CA)
3. **Certificate resource:** defines *what* certificate you need (domain names, which Secret to store it in)
4. **Cert-Manager requests the cert**, completes the CA's challenge (e.g., HTTP-01/DNS-01 for Let's Encrypt), and stores the result in a **Kubernetes Secret**
5. **Automatic Renewal:** Cert-Manager watches expiry and renews certs automatically — no manual steps

### Step-by-Step Setup

**1. Install via Helm:**
```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.10.0
```

**2. Create an Issuer:**
```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-issuer
  namespace: default
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/...
    privateKeySecretRef:
      name: letsencrypt-private-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

**3. Create a Certificate resource:**
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-tls
  namespace: default
spec:
  secretName: example-tls-secret
  issuerRef:
    name: letsencrypt-issuer
  dnsNames:
    - example.com
    - www.example.com
```
- `secretName` → where the cert/key will be stored
- `dnsNames` → domains the cert should cover

**4. Attach it to an Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
  tls:
    - hosts:
        - example.com
      secretName: example-tls-secret
```
> The `tls` block references the Secret created by Cert-Manager — this is what actually secures the connection.

**5. Renewal:** fully automatic — Cert-Manager checks expiry and renews before the certificate lapses.

---

## 🧱 Terraform Modules — Interview Prep

### 1. Why Use Terraform Modules?

| Reason | Simple Explanation |
|--------|----------------------|
| **Reusability (DRY)** | Define infra once as a reusable "child module" instead of copy-pasting HCL for Dev, QA, Staging, Prod |
| **Separation of Concerns** | Keep networking (VPC), cluster control-plane (EKS), and compute (worker nodes) in separate, decoupled folders |

### 2. Standard Enterprise Directory Layout

```
terraform-root/
├── main.tf                 # Root orchestration calling child modules
├── variables.tf            # Global root input variables
├── outputs.tf              # Aggregated root outputs
├── providers.tf            # AWS provider + Terraform version constraints
└── modules/
    ├── vpc/                # Networking module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── eks/                # EKS cluster control-plane module
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── ec2_node/           # Worker node group / compute module
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 3. What Each Child Module Contains

**A. VPC Module** — creates `aws_vpc` (DNS enabled), public/private subnets across AZs, `aws_internet_gateway`, `aws_nat_gateway` with Elastic IPs. Exports IDs so parent modules can use them:
```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

**B. EKS Module** — creates IAM Cluster Role (`AmazonEKSClusterPolicy`) + `aws_eks_cluster`. Takes `vpc_id` and `subnet_ids` as **inputs** (never hardcoded). Exports `cluster_name`, `cluster_endpoint`, `cluster_certificate_authority_data`.

**C. Worker Node Module (`ec2_node`)** — creates node IAM roles (`AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`) + `aws_eks_node_group`. Scaling controlled via `min_size` / `max_size` / `desired_size` variables — never fixed static counts.

### 4. Root Module — Wiring the Modules Together

```hcl
# 1. Instantiate VPC Module
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  env    = var.environment
}

# 2. Instantiate EKS Module using VPC Outputs
module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id              # inter-module reference
  subnet_ids = module.vpc.private_subnet_ids  # inter-module reference
}

# 3. Instantiate Worker Node Module using EKS & VPC Outputs
module "ec2_node" {
  source       = "./modules/ec2_node"
  cluster_name = module.eks.cluster_name
  subnet_ids   = module.vpc.private_subnet_ids
  node_count   = var.worker_node_count
}
```

**Module source types:**

| Type | Example | When to Use |
|------|---------|-------------|
| Relative local path | `source = "./modules/vpc"` | Modules live in the same repo |
| Remote Git repo | `source = "git::https://github.com/org/terraform-aws-vpc.git?ref=v1.2.0"` | Enterprise standard — version pinning, sharing across repos |

### 5. 🎯 Key Interview Q&A

> **"How do you pass data between modules?"**
> A child module's internal resources can't be read directly — it must expose the value in its own `outputs.tf`. The root module references it as: `module.<module_name>.<output_name>`

> **"Why avoid hardcoding values inside child modules?"**
> Hardcoding breaks reusability across environments. All environment-specific values (instance types, CIDRs, replica counts) must flow through `variables.tf`.

---

## 🤝 Terraform + Ansible: Working Together in Real Projects

### 1. The Division of Labor

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure-as-Code — provisions VMs, VPCs, databases (declarative, HCL) |
| **Ansible** | Configuration management — installs packages, configures services, deploys apps (via SSH/WinRM) |

**Simple way to remember it:** *Terraform builds the house. Ansible furnishes and maintains it.*

**Workflow:**
1. Terraform provisions infrastructure (EC2, VPC, DB, etc.)
2. Ansible configures the software on top (web server, DB setup, app deploy)
3. Terraform **outputs** (IPs, instance IDs) feed into Ansible's inventory so it knows *what* to configure

### 2. Typical Repository Structure

```
project/
├── terraform/
│   ├── main.tf               # Resource provisioning
│   ├── variables.tf
│   ├── outputs.tf             # IPs, instance IDs, etc.
│   ├── provider.tf
│   └── terraform.tfvars       # Environment-specific values
├── ansible/
│   ├── inventory/
│   │   ├── prod.yaml
│   │   ├── dev.yaml
│   │   └── staging.yaml
│   ├── playbooks/
│   │   ├── setup.yaml         # Configure instances
│   │   └── deploy.yaml        # Deploy applications
│   ├── roles/
│   │   ├── webserver/
│   │   ├── db/
│   │   └── app/
│   └── ansible.cfg
└── scripts/
    └── deploy.sh               # Orchestrates Terraform + Ansible
```

### 3. How Ansible Talks to Terraform-Created Resources

**Option A — Pass Terraform output directly to Ansible:**
```bash
terraform init
terraform apply

# Grab the IP from Terraform output
export instance_ip=$(terraform output -raw instance_ip)

# Feed it to Ansible
ansible-playbook -i ${instance_ip}, playbooks/setup.yaml
```

**Option B — Dynamic Inventory Script (preferred at scale):**
Instead of manually copying IPs, a script queries Terraform's state and auto-generates a live inventory.

```python
#!/usr/bin/env python
import json
import subprocess

def get_terraform_output():
    terraform_output = subprocess.check_output(
        ['terraform', 'output', '-json'],
        universal_newlines=True
    )
    return json.loads(terraform_output)

def main():
    inventory = {'all': {'hosts': []}}
    terraform_output = get_terraform_output()
    for resource in terraform_output['instances']['value']:
        inventory['all']['hosts'].append(resource['public_ip'])
    print(json.dumps(inventory, indent=2))

if __name__ == "__main__":
    main()
```

> **Simple explanation:** This script asks Terraform "what did you just build?", turns the answer into a list of IPs, and Ansible uses that list as its target hosts — no manual copy-pasting needed.

### 4. Dynamic Inventory Across Multiple Environments

- Use **separate Terraform workspaces** (`prod`, `staging`, `dev`) to isolate environments
- Use **separate Ansible inventory files** (`prod.yaml`, `dev.yaml`, `staging.yaml`) — or generate them dynamically per environment

```bash
terraform workspace select prod
terraform apply

ansible-playbook -i inventory/prod.yaml playbooks/setup.yaml
```

### 5. Running It All Through a CI/CD Pipeline (GitLab CI example)

```yaml
stages:
  - terraform
  - ansible
  - deploy

variables:
  TF_VAR_environment: "dev"

terraform:
  stage: terraform
  script:
    - terraform init
    - terraform plan
    - terraform apply -auto-approve

ansible:
  stage: ansible
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/setup.yaml

deploy:
  stage: deploy
  script:
    - ansible-playbook -i dynamic_inventory.py playbooks/deploy.yaml
```

**How often do these run?**

| Tool | Frequency |
|------|-----------|
| **Terraform** | Weekly, or only when infrastructure changes (new resources) |
| **Ansible** | On every commit / pull request — to deploy and configure services |

---

## 🌱 Career Transition Guide: Switching to DevOps

### 1. Two Practical Routes

| Route | Description |
|-------|-------------|
| **A. Internal Transition (fastest legitimacy)** | Ask your manager for release/partial allocation to an internal DevOps/Cloud project. Even 2–4 months of cross-skilling gives you real, defensible production experience |
| **B. Self-Learning / External Switch** | If internal mobility is blocked, don't wait indefinitely. Use hands-on labs (KodeKloud), AWS/Azure free tier, or self-host tools (EC2, Jenkins, Docker, Minikube/kind/EKS) to build real end-to-end projects |

### 2. The Three Core Interview Questions for Career Switchers

| Question | Strategy |
|----------|----------|
| **"What are your exact roles and responsibilities?"** | Anchor answers around what you're actually strong in — pipelines, infra (Terraform), or scripting (Bash/Python) |
| **"What does a typical day look like for you?"** | Structure it: standup → check alerts/failed builds → root-cause investigation → infra/pipeline development |
| **"What tools and cloud tech do you own?"** | Be specific about your stack — don't generalize |

### 3. High-Priority Tools to Learn

- **Kubernetes:** EKS/AKS architecture, Pods, Deployments, Services, Helm
- **CI/CD & Containers:** Declarative pipelines (Jenkins/GitHub Actions/GitLab CI), multi-stage Dockerfiles, ECR
- **DevSecOps:** SAST (SonarQube), SCA (OWASP Dependency-Check), secrets management (Vault/AWS Secrets Manager)
- **IaC & Scripting:** Modular Terraform, Shell/Python automation

### 4. Overcoming Imposter Syndrome

> Every engineer has a ramp-up curve on a new codebase and infra topology. The effort put into labs, reading logs, and root-cause troubleshooting **does** translate directly to job performance. After ~30–60 days, it becomes natural.

---

## ⚡ Quick-Fire Practical Interview Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | How do you rollback to a previous build in Jenkins? | Go to job → Build History → select the previous build → click **Rebuild** or **Build Now**. Or, download the artifact from that build and redeploy it manually |
| 2 | How do you connect to a Kubernetes cluster? | Install `kubectl`; ensure `~/.kube/config` is present and points to the cluster |
| 3 | What's needed in an Ingress resource to route traffic? | Install an Ingress Controller → create the Ingress resource (host-based or path-based rules) → create a DNS record pointing to the controller's IP |
| 4 | How do you troubleshoot high CPU utilization? | Use `top` to identify the offending process; kill it if stuck; check for runaway background processes |
| 5 | How many slave nodes in your Jenkins setup? | Example answer: 1 master node, 3 slave nodes |
| 6 | How many services are running in your project? | Clarify: Kubernetes Services vs. Docker services — answer accordingly |
| 7 | How many Jenkins pipelines have you worked on? | Example: Scripted pipeline + Multibranch pipeline; 5–6 pipeline scripts created/maintained |
| 8 | How to run a command in the background? | `nohup <command> &` |
| 9 | How do you import a manually created resource into Terraform? | Define the resource block, then run: `terraform import aws_instance.example i-0abcd1234efgh5678` |
| 10 | Difference between load balancer types? | **Layer 4** = TCP/UDP; **Layer 7** = HTTP/HTTPS (application-aware routing) |

---

## 📦 Artifact Promotion: How Deployment Actually Happens in Production

### The Core Question
> "Is the exact same artifact tested in QA promoted to production, or does a separate pipeline build a fresh artifact from `main`?"

### Approach A: Rebuild Per Target Pipeline
- Artifact is built & tested in Dev/QA pipeline runs.
- On merge to `main`, a **separate production release pipeline** rebuilds the artifact from that branch (same version tag/commit SHA).
- Before touching prod, it's deployed to a staging/QA-mirror environment for smoke/regression tests.
- A **mandatory manual approval gate** (email, interactive input, or release sign-off) precedes the actual production push.

### Approach B: "Build Once, Promote Everywhere" (12-Factor Standard) ⭐
- An **immutable artifact** (Docker image / `.jar`) is built **only once**, from the commit SHA.
- The exact same image digest is tested in Dev, promoted to QA/Staging, then to Production — **never recompiled**.
- Environment differences are handled purely via **externalized config** (ConfigMaps, Secrets, env vars) — never baked into the image.

> 💡 **Industry standard/recommended answer:** Don't rebuild. Build once, tag with the Git commit SHA, push to a registry (ECR/Nexus/Harbor), and **promote the same image digest** across all environments — only swapping runtime config.

### QA / Pre-Prod Parity
Staging must mirror production as closely as possible (networking, DB engines, ingress rules) — otherwise config drift can hide environment-specific bugs that only show up in prod.

### 🎯 Interview One-Liner
> "In enterprise environments there are two common patterns: Single Artifact Promotion (build once, tag with commit SHA, push to ECR, promote the same digest through QA → Prod behind an approval gate — only ConfigMaps/Secrets change) or Branch-Triggered Pipeline (merging to main rebuilds the artifact fresh from main, verified in staging, then promoted after approval). I generally favor Build Once, Promote Everywhere for consistency and to eliminate rebuild drift."

---

## ⚡ Event-Driven Architecture in DevOps

**Definition (simple):** Instead of manually triggering actions, the system reacts automatically to *events* (a code push, a traffic spike, a failed health check).

### Real-World DevOps Examples

| Trigger Event | Automated Reaction |
|----------------|----------------------|
| Code pushed to Git | CI/CD pipeline auto-starts (test, scan, package, deploy) |
| Traffic spike | Auto-scaling launches new instances/pods; scales down when traffic drops |
| Deployment failure | Automatic rollback to the previous stable version |
| System outage / high error rate | Alerts fire, remediation actions trigger (e.g., service restart), team gets notified |

### 🎯 How to Defend "Event-Driven Architecture" on Your Resume
Don't just say the buzzword — name the actual mechanism:
- **CI/CD Triggers:** Git webhook → Jenkins/GitHub Actions workflow fires immediately
- **Dynamic Autoscaling:** CloudWatch/Prometheus threshold breach → triggers ASG or Kubernetes HPA/Karpenter
- **Self-Healing/Rollback:** Health check failure → automated rollback, container restart, or traffic shift at the load balancer

---

## 🧪 QA Pipeline Ownership & Environment-Specific Stage Control

### Who Builds the QA Pipeline?
> The **DevOps engineer** designs, provisions, and maintains the CI/CD pipeline. The **QA team** contributes automated test scripts (Selenium, Cypress, Playwright) that run *inside* a pipeline stage.

**Standard QA pipeline flow:**
1. Git checkout of the testable branch/tag
2. Build/compile the application
3. Static analysis & security scan (SonarQube)
4. Automated integration/end-to-end tests
5. Publish test reports & build status notifications

### Do You Rebuild the Image for Each Environment?
**No.** Build the Docker image once in CI, tag it immutably (commit SHA or semver), push to a registry (ECR/Nexus/Harbor). Deploy that **same image digest** to Dev, QA, Staging, and Prod — only runtime env vars/secrets change via ConfigMaps/Secrets.

### Controlling Which Stages Run Per Environment (Single Parameterized Pipeline)
Use the Jenkins `when` directive to conditionally run/skip stages based on the environment parameter:

```groovy
stage('Deploy to Production') {
    when {
        expression { params.ENVIRONMENT == 'Production' }
    }
    steps {
        input message: 'Approve production deployment?'
        sh 'helm upgrade --install prod-release ./charts/app'
    }
}
```

---

## 🖥️ Real-World Linux Automation Use Cases (Interview Favorites)

| Use Case | What It Does |
|----------|----------------|
| **Cost Optimization Scripts** | Auto-shutdown non-prod Dev/QA instances outside business hours; restart before workday starts |
| **Mass Maintenance / Coordinated Restarts** | Rolling restarts across many servers, verifying health checks/ports before moving to the next node |
| **Log Rotation & Disk Space Remediation** | Cron/event-driven scripts detect high disk usage (`/var/log > 85%`), archive logs to S3, purge stale cache |
| **CrashLoopBackOff / OOM Remediation** | Scripts triage stuck pods, extract exit codes (OOMKilled / Exit 137), dump logs, notify on-call |

---

## 🔧 Essential Git Commands Cheat Sheet

| Command | What It Does |
|---------|----------------|
| `git init` | Creates 3 areas: working area, staging area, local repo |
| `git status` | Shows file status |
| `git add .` | Adds all files from working → staging area |
| `git add file1 file2` | Adds only specified files |
| `git add *.sh` | Adds all `.sh` files |
| `git commit` | Moves files from staging → local repo |
| `git commit -m "msg"` | Commit with a message |
| `git config --global user.name ""` | Set global username |
| `git config --global user.email ""` | Set global email |
| `git commit -m "msg" filename` | Commit a specific file |
| `git remote add aliasname "url"` | Add a remote repo |
| `git remote -v` | Show all mapped remotes |
| `git push aliasname master` | Push a branch to remote |
| `git log -2` | Show last 2 commit IDs |
| `git show commit_id` | Show files changed in a commit |
| `git reset` | Move files from staging back to working area |
| `git revert` | Revert the last commit (locally) |
| `git pull aliasname master` | Pull changes into working area |
| `git clean` | Remove newly created untracked files |
| `git branch branchname` | Create a branch |
| `git checkout branchname` | Switch to a branch |
| `git branch -a` | List all branches |
| `git merge branchname` | Merge a branch into current branch |
| `git diff branchname` | Show differences |
| `git push aliasname --all` | Push all branches |
| `git tag tagname` | Create a tag |
| `git push aliasname tag tagname` | Push a tag to remote |
| `git stash apply stash@{1}` | Apply a specific stashed change |
| `git cherry-pick commit_id` | Merge a specific commit |
| `git fetch aliasname branchname` | Fetch changes into local repo (no merge) |
| `git clone "url"` | Clone a full new repo |
| `git rebase branchname` | Rebase current branch onto another |

---

## 🔄 ArgoCD — GitOps Interview Prep

### 1. What is ArgoCD / GitOps?
> **GitOps core principle:** Git is the single source of truth for both application code and Kubernetes infrastructure state.

**Reconciliation loop:** ArgoCD continuously compares the **Desired State** (Git: manifests/Helm/Kustomize) with the **Live State** (actual cluster). A mismatch = `OutOfSync`.

| Sync Mode | Behavior |
|-----------|----------|
| **Automated Sync** | Auto-applies changes to restore parity |
| **Manual Sync** | Flags drift in UI/CLI, waits for operator approval |

**CI/CD split:** Traditional CI (Jenkins/GitHub Actions) handles build/test/scan + updates the image tag in Git. **ArgoCD handles CD** — pulling changes into the cluster without exposing cluster credentials to the CI server.

### 2. ArgoCD Architecture

```
                  ┌──────────────────────┐
                  │    Git Repository    │
                  │ (Helm/Kustomize/YAML)│
                  └──────────▲───────────┘
                             │
                  ┌──────────┴───────────┐
                  │   Repository Server  │ ── Clones repo, renders manifests
                  └──────────▲───────────┘
                             │
┌──────────────┐  ┌──────────┴───────────┐  ┌──────────────┐
│  Web UI/CLI  │◄─┤      API Server      │◄─┤  Redis Cache │ ── Caches state/tokens
└──────────────┘  └──────────┬───────────┘  └──────────────┘
                             │
                  ┌──────────▼───────────┐
                  │Application Controller│ ── Compares Live vs Desired, syncs
                  └──────────┬───────────┘
                             ▼
                  [ Target K8s Cluster ]
```

| Component | Role |
|-----------|------|
| **API Server** | Handles UI/CLI/CI requests, auth, RBAC |
| **Repository Server** | Clones Git repos, renders manifests (raw YAML/Helm/Kustomize) |
| **Application Controller** | Core reconciliation loop — compares & syncs live vs. desired state |
| **Redis Cache** | Fast in-memory cache for state comparisons & sessions |
| **Web UI / CLI** | Visualize topologies, sync status, manual approvals |

### 3. Rollback & Versioning

| Method | How |
|--------|-----|
| **Git-native rollback (preferred)** | `git revert <commit-hash>` → ArgoCD auto-detects and rolls the cluster back |
| **UI/CLI rollback** | `argocd app rollback <app-name>` |
| **Self-healing / auto-pruning** | Reverts manual `kubectl edit` drift; prunes resources deleted from Git |

### 4. Helm & Kustomize Support
- **Helm:** Tracks a Helm repo or a Git folder with `Chart.yaml`/`values.yaml`, renders on the fly with value overrides.
- **Kustomize:** Native support for environment overlays (`overlays/dev`, `overlays/prod`) without separate charts.

### 5. ApplicationSets & Multi-Cluster
- **ApplicationSets:** Dynamically generate multiple ArgoCD `Application` resources from one template — great for deploying the same app across many clusters/environments without repetitive YAML.
- **Multi-cluster:** Register remote clusters (`argocd cluster add <kubecontext>`), then specify the target cluster in `spec.destination` of the Application manifest.

### 6. ArgoCD vs. Flux CD

| Dimension | ArgoCD | Flux CD |
|-----------|--------|---------|
| Architecture | Centralized server, rich built-in Web UI, SSO, RBAC | Decentralized, lightweight GitOps Toolkit controllers |
| UI | Rich interactive Web UI with live resource trees | CLI-centric; relies on 3rd-party UIs (e.g., Weave GitOps) |
| Best Fit | Complex enterprise, multi-tenant, developer self-service | Minimalist, headless environments |

### 7. Security Best Practices
RBAC, SAML/OIDC authentication, secure Git access, application secrets management, audit logging.

---

## 🌐 Ingress vs. Load Balancer, CronJobs, Multi-Cloud & Migration

### 1. Ingress vs. Cloud Load Balancer — Which to Use?

| Factor | `type: LoadBalancer` | Ingress Controller |
|--------|------------------------|----------------------|
| Cost | A **dedicated** cloud LB per exposed service → costs scale linearly | **One** entry point (single ALB/VIP) multiplexes traffic across many services |
| Routing | Basic L4 | L7 path-based (`/api`, `/web`) or host-based (`app.domain.com`) rules |
| Best for | Raw TCP/UDP, low-latency websockets, non-HTTP protocols | Standard HTTP/HTTPS web & API traffic |

> **Rule of thumb:** Use Ingress for HTTP/HTTPS (cost-effective, flexible routing). Use a Load Balancer (NLB) only when you need raw TCP/UDP or non-HTTP protocols.

### 2. Kubernetes CronJobs

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup-cronjob
spec:
  schedule: "0 2 * * *"   # Daily at 02:00 UTC
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup-task
              image: backup-utility:1.2
```

Schedule format: `Minute | Hour | Day-of-Month | Month | Day-of-Week`

**Common use cases:** DB snapshot exports, cert-manager validation runs, log archiving, cache warming.

### 3. Multi-Cloud CI/CD Challenges

| Pain Point | Why It's Hard |
|-------------|-----------------|
| Incompatible IAM models | AWS IAM Roles vs. Azure Managed Identities vs. GCP Service Accounts all differ |
| Complex cross-cloud networking | Needs IPsec VPN tunnels, transit gateways, or private interconnects (Direct Connect / ExpressRoute) |
| Tooling fragmentation | Scripts must juggle `aws-cli`, `az`, `gcloud` + different secret/storage backends |

> **Recommended pattern:** Use GitOps (ArgoCD) running *inside* each target cluster, pulling from a centralized Git repo — avoids brittle direct cross-cloud API calls from the CI server.

### 4. Cloud-to-Cloud Migration Strategy (e.g., Azure → AWS)

1. **Assessment & Inventory** — identify stateful data, active DBs, compute dependencies
2. **Database Migration** — use managed replication (e.g., AWS DMS) for continuous schema/data sync with minimal downtime
3. **Compute & Storage Replication** — use AWS MGN (replication agents on source VMs) for continuous block-level sync
4. **Validation & Cutover** — validate data parity, run staging smoke tests, shift traffic via Route 53 weighted records, decommission legacy instances

### 5. Practical Jenkins Failures (Real Troubleshooting Experience)

| Failure | Root Cause | Fix |
|---------|------------|-----|
| **Agent Offline / Not Ready** | SSH credential expiry, network/firewall drops, JVM/Java version mismatch | Refresh credentials, check connectivity, align Java versions |
| **Hanging/Stuck Builds** | Thread deadlocks, exhausted executors, unhandled interactive prompts (`terraform apply` without `-auto-approve`), missing timeouts | Add `options { timeout(...) }` and `cleanWs()` |
| **Disk Full** | Stale Maven deps (`~/.m2`), old logs, untagged Docker images | Daily `docker system prune -af` cron + workspace discard policy |
| **Broken Plugin/Dependency Conflicts** | Upgrading Jenkins core without checking plugin compatibility, Docker Hub rate limits on base image pulls | Check compatibility matrix before upgrades; use a mirror/registry cache |

---

## 🗓️ A Day in the Life of a DevOps Engineer

### Cross-Team Interactions
A DevOps/Platform engineer typically interfaces with:
- **Core Product Dev Teams** — feature developers consuming CI pipelines
- **Internal Platform/Automation Teams** — tooling & alert automation
- **Dev & QA Infra Teams** — lower/staging environments
- **Production Operations Teams** — live customer-facing infra
- **SRE/On-Call** — incident response, telemetry

### Realistic Daily Schedule

```
09:00–09:30 ── System checks, monitoring review, alerts, Jira backlog
09:30–10:00 ── Daily Standup (yesterday / today / blockers)
10:00–13:00 ── P0/P1 priority work (pipeline debugging, hotfixes, unblocking devs)
14:00–17:00 ── Core project execution (Terraform modules, CI/CD refactoring, scripts)
17:00–18:00 ── Cross-team syncs, documentation, runbook updates
```

### Role Rotation (Prevents Single Points of Failure)

| Role | Focus |
|------|-------|
| Pipeline Engineers | Jenkinsfiles/GitHub Actions, build agents, Docker build caching, quality gates |
| Infra/Automation Engineers | Terraform, Helm charts, Python/Bash automation |
| Production Support | Monitoring live deployments, CrashLoopBackOff/OOM investigation |
| Knowledge Transfer | Peer reviews, pairing, shadowing for coverage continuity |

### The "Automate Recurring Issues" Principle
> If an issue happens **once**, document the fix. If it happens **twice**, **automate the remediation** (e.g., automated pod restarts, disk cleanup jobs, log archiving).

### 🎯 Interview One-Liner
> "My day starts by reviewing monitoring alerts and our Jira board for high-priority blockers. During standup, I align with developers on sprint deliverables. The core of my day splits between project automation (Terraform modules, CI/CD refinements, Helm templates) and platform maintenance (investigating failed pipelines, helping devs with environment issues, rotating secrets). For recurring incidents, I turn manual fixes into automated scripts and keep our runbooks updated."

---

## 🔐 Cert-Manager: SSL/TLS Certificates in Kubernetes

### How It Works (Simple Explanation)
Cert-Manager automatically **creates, attaches, and renews** SSL/TLS certificates for apps running in Kubernetes — no manual cert management needed.

**Flow:**
1. Install Cert-Manager (controllers + CRDs) into the cluster
2. Define an **Issuer/ClusterIssuer** — tells Cert-Manager *how* to get certs (Let's Encrypt, Vault, internal CA)
3. Define a **Certificate** resource — specifies *what* domains need a cert and where to store it (a Secret)
4. Cert-Manager completes the CA's challenge (e.g., HTTP-01/DNS-01) automatically and stores the resulting cert in a **Kubernetes Secret**
5. Cert-Manager **auto-renews** the cert before it expires — no manual intervention

### Step-by-Step Setup

**1. Install Cert-Manager (via Helm):**
```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace --version v1.10.0
```

**2. Create an Issuer:**
```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-issuer
  namespace: default
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/...
    privateKeySecretRef:
      name: letsencrypt-private-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

**3. Create a Certificate resource:**
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-tls
  namespace: default
spec:
  secretName: example-tls-secret
  issuerRef:
    name: letsencrypt-issuer
  dnsNames:
    - example.com
    - www.example.com
```

**4. Attach the cert to an Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
  tls:
    - hosts:
        - example.com
      secretName: example-tls-secret
```

**5. Automatic renewal:** Cert-Manager periodically checks expiration and renews before the cert goes stale — fully hands-off.

---

## 🐚 Shell Scripting & SQL — Real Client Interview Questions

### 1. SQL: Find the 7th Highest Marks

**Step-by-step approach:**
1. Order records by marks descending: `ORDER BY marks DESC`
2. Limit to the top 7 rows: `LIMIT 7`
3. Wrap that as a subquery and sort **ascending**: `ORDER BY marks ASC`
4. Take the first row of that result: `LIMIT 1` → this is the 7th highest

```sql
SELECT marks FROM (
  SELECT marks FROM students
  ORDER BY marks DESC
  LIMIT 7
) AS top7
ORDER BY marks ASC
LIMIT 1;
```
> **Simple logic:** Grab the top 7 → flip their order → the first one of the flipped list is the 7th highest.

### 2. Shell Script: Count Files Containing a Specific Word

**Task:** Count how many files (including subdirectories) contain a given keyword.

```bash
#!/bin/bash

word="username"
dir="/path/to/search"

count=$(grep -rl "$word" "$dir" | wc -l)

echo "Number of files containing the word: $count"
```

| Part | Purpose |
|------|---------|
| `#!/bin/bash` | Shebang — tells the OS which interpreter runs this script |
| `word=`, `dir=` | Reusable variables |
| `grep -rl "$word" "$dir"` | `-r` recursive search, `-l` list matching filenames only |
| `wc -l` | Counts the number of matched files |

### 3. 🎯 Interview Strategy Tips
- **Think out loud** — never code in silence; narrate your logic as you go
- **Demonstrate core logic** — even imperfect syntax under pressure is fine if your reasoning (why `grep`, why `-r`, why `wc -l`) is sound; it shows problem-solving skill and lets the interviewer guide you

---

## 🏢 Real Interview Experiences (Company-Specific Prep)

### Globant — DevOps Engineer (1–1.5 hr technical round)

| Domain | Topics Asked |
|--------|----------------|
| **Linux** | File hierarchy, `ip addr`/`ifconfig`, `who`/`w`, `kill`/`kill -9`/`pkill`, `chmod`/`chown` |
| **Git** | `git fetch` vs `git pull`, purpose & mechanics of `git cherry-pick` |
| **Jenkins** | Pipeline lifecycle/stages, common plugins, trigger mechanisms (poll SCM, webhooks, cron) |
| **Docker** | `FROM`, `CMD` vs `ENTRYPOINT`, `COPY` vs `ADD`, Docker Swarm vs services/nodes, Compose, networking, volumes |
| **Kubernetes** | Control plane vs worker nodes, what happens to workloads if master is unreachable, headless services, ReplicaSet vs ReplicationController, Taints & Tolerations, Ingress, Helm, `kubectl --dry-run=client -f <file.yaml>` |
| **Terraform/Ansible** | `terraform apply -auto-approve`, Ansible syntax + `--syntax-check`, custom modules, roles, Tower/AWX, `ansible-vault` |

> **Key takeaway:** Even if your background is AWS-heavy, be ready for Azure DevOps-flavored questions if the role leans that way.

### BMC Software — DevOps / Automation Focus

Heavy emphasis on **live Bash coding**, not just theory:
- Purpose of the Shebang (`#!/bin/bash`)
- Taking dynamic user input (`read`)
- Writing test conditions (e.g., check if a file exists and is writable: `-w`)
- File automation: moving/renaming files by appending a dynamic date/timestamp

> **Key takeaway:** Prepare to write real Bash scripts live, not just explain concepts.

---

## 🧰 Linux Commands Interview Cheat Sheet

### File Inspection & Logs
| Command | Purpose |
|---------|---------|
| `head -n <N> <file>` / `tail -n <N> <file>` | Read first/last N lines |
| `tail -f <file>` | Live-stream newly appended lines (great for debugging logs) |

### Text Processing
| Tool | Purpose |
|------|---------|
| `sed` | Inline substitutions/deletions without opening an editor |
| `awk` | Record/field-based text manipulation (splitting by delimiter, isolating columns) |
| `grep -i` | Case-insensitive search |
| `grep -r` | Recursive search through subdirectories |
| `grep -c` | Returns count of matches, not the lines themselves |

### Users, System Status & Networking
| Command | Purpose |
|---------|---------|
| `whoami` | Current logged-in user |
| `w` / `users` | Active users & their processes |
| `uptime` | Server uptime, load averages, active users |
| `last` | Recent reboot/login history |
| `ifconfig` / `hostname -I` | Internal/interface IP addresses |

### Service & Process Management
| Command | Purpose |
|---------|---------|
| `systemctl start/restart/status <service>` | Manage services (Jenkins, Docker, Filebeat, etc.) |
| `ps -ef \| grep <process_name>` | Find a running process and its PID |
| `top` | Interactive real-time CPU/process monitoring |
| `free -m` / `free -g` | RAM & swap usage (MB/GB) |

### Archiving, Compression & Transfer
| Command | Purpose |
|---------|---------|
| `zip -r <archive.zip> <folder>` | Recursively compress a folder |
| `unzip <archive.zip>` | Extract a zip archive |
| `tar -cvf <name.tar> <folder>` | Create a tar archive (c=create, v=verbose, f=filename) |
| `tar -xvf <name.tar>` | Extract a tar archive |
| `scp /local/path user@remote:/dest/path` | Securely copy files between servers over SSH |

---

## 🤖 Ansible Interview Prep — Part 1: Fundamentals & Architecture

### 1. Configuration Management vs. Provisioning

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure provisioning (VMs, VPCs, subnets, storage) |
| **Ansible** | Configuration management — installs packages, manages config files, starts services post-provisioning |

### 2. Push vs. Pull Architecture

| Tool | Model | Agent Required? |
|------|-------|-------------------|
| **Ansible** | Push-based | ❌ Agentless — control node pushes changes over SSH |
| **Puppet / Chef** | Pull-based | ✅ Local agents periodically pull config from a master server |

### 3. OS Support
- **Linux nodes:** connected via **SSH**
- **Windows nodes:** connected via **WinRM**
- ⚠️ The Ansible **control node itself cannot run natively on Windows**, but it *can* manage/configure Windows target nodes

### 4. Inventories
| Type | Description |
|------|--------------|
| **Static Inventory** | Manually defined hosts/groups/IPs (default location: `/etc/ansible/hosts`) |
| **Dynamic Inventory** | Scripts (Python/cloud plugins) that query cloud providers or Terraform outputs for live host IPs |

### 5. Modules
| Type | Description |
|------|--------------|
| **Core Modules** | Built-in (e.g., `yum`, `apt`, `copy`, `service`, `win_copy`) |
| **Custom Modules** | User-written (commonly Python) for bespoke automation |

### 6. Execution Commands
```bash
# Run a playbook (verbose)
ansible-playbook -i <inventory_file> <playbook.yml> -v

# Syntax check only
ansible-playbook <playbook.yml> --syntax-check

# Ad-hoc one-liner command (no playbook needed)
ansible all -i <inventory> -m shell -a "date"
```

### 7. Roles — Modularizing Playbooks

| Directory | Purpose |
|-----------|---------|
| `tasks/` | Main execution steps |
| `handlers/` | Conditional tasks — run only when notified by a change |
| `vars/` & `defaults/` | Variable definitions |
| `templates/` & `files/` | Config files & Jinja2 templates |
| `meta/` | Role metadata & dependencies |

### 8. Ansible Vault (Secrets Encryption)
```bash
ansible-vault encrypt <secrets_file.yml>
```
Used to securely encrypt passwords, tokens, and keys inside YAML files.

---

## 🤖 Ansible Interview Prep — Part 2: Playbook Syntax & Structure

### 1. Core Play Directives
```yaml
- hosts: all          # Target servers: all, localhost, or an inventory group
  become: true         # Run tasks with sudo/root privileges
  tasks:
    - name: Install git
      yum:
        name: git
        state: present

    - name: Copy file
      copy:
        src: /local/path/file.txt
        dest: /remote/path/file.txt
```
> Each task starts with `- name:` — this is a YAML list item. Indentation defines the hierarchy, so it must be exact.

### 2. Handlers — Run Only When Notified
```yaml
tasks:
  - name: Configure Apache
    template:
      src: httpd.conf.j2
      dest: /etc/httpd/conf/httpd.conf
    notify: restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```
> ⚠️ The `notify:` string must **exactly match** the handler's `name:`. Handlers run once at the end of a play, only if triggered.

### 3. Loops — Avoid Repetitive Tasks
```yaml
- name: Install list of packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - wget
    - vim
    - zip
```

### 4. Variables
```yaml
vars:
  package_name: httpd

tasks:
  - name: Install web server
    yum:
      name: "{{ package_name }}"
      state: present
```

### 5. Tags — Run Only Specific Parts of a Playbook
```yaml
tasks:
  - name: Install packages
    yum:
      name: git
      state: present
    tags:
      - install
```
```bash
ansible-playbook playbook.yml --tags "install"
```

### 6. 🎯 Interview Advice
> Focus on understanding the **overall architecture and syntax flow** (`hosts`, `become`, `tasks`, module structure, YAML indentation). Forgetting a minor module parameter under pressure is fine — as long as your logical execution flow is correct.

---

## ⚖️ DevOps vs. GitOps

| Aspect | DevOps | GitOps |
|--------|--------|--------|
| **Scope** | Broad org culture & practices — full lifecycle: code, test, deliver, manage infra | Focused specifically on deployment automation & continuous delivery |
| **Center of Gravity** | Tool-agnostic — mixes CI/CD tools, scripts, cloud platforms | Git-centric — Git is the single source of truth for all environment states |
| **Configuration Model** | Declarative OR imperative (scripts, CLI, UI triggers) | Strictly declarative (K8s YAML, Helm, Kustomize stored in Git) |
| **Reconciliation** | Often push-based — CI server pushes deployments when triggered | Pull-based — in-cluster operator (ArgoCD/Flux) continuously reconciles drift against Git |

### How GitOps Actually Works
1. An operator/agent (ArgoCD or Flux) runs **inside** the Kubernetes cluster
2. It continuously compares Git against the live cluster state
3. When changes are merged into Git, the agent auto-syncs the cluster to match
4. If someone manually edits cluster resources (drift), the operator detects and **remediates** it automatically
5. A single GitOps engine can manage **multiple clusters** simultaneously

> **Simple way to remember it:** DevOps = the whole philosophy/culture. GitOps = a specific *implementation* of the "CD" part of DevOps, using Git as the control mechanism.

---

## 🏗️ Hands-On IaC Project: Terraform + Jenkins → EC2

A simple, presentable end-to-end mini-project for interviews.

### Prerequisites
- Jenkins installed (with Terraform + Git plugins)
- GitHub repo containing Terraform code
- AWS CLI credentials configured in Jenkins
- IAM permissions for EC2

### Sample Terraform Code (`main.tf`)
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # valid AMI for your region
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform-EC2"
  }
}
```
Also include: `variables.tf` (optional), `terraform.tfvars` (optional), `.terraform.lock.hcl` (auto-generated), `.gitignore`.

### Jenkinsfile (Pipeline Script)
```groovy
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }
        stage('Init Terraform') {
            steps { sh 'terraform init' }
        }
        stage('Plan Terraform') {
            steps { sh 'terraform plan' }
        }
        stage('Apply Terraform') {
            steps {
                input "Proceed with apply?"   // Manual approval
                sh 'terraform apply -auto-approve'
            }
        }
    }
}
```
> 🔐 **Never hardcode AWS credentials.** Store them under **Manage Jenkins → Credentials**.

### Setting Up the Jenkins Job
1. Open Jenkins → **New Item → Pipeline**
2. Under "Pipeline script from SCM," connect your GitHub repo
3. Save and Build

**Result:** Terraform code in GitHub → Jenkins pulls & applies it → EC2 instance launched on AWS.

---

## 💬 Behavioral & Scenario-Based Interview Questions (By Category)

### CI/CD Tools
1. Tell me about a time you set up a CI/CD pipeline from scratch — tools used, challenges faced?
2. A pipeline failed unexpectedly — how did you identify and resolve it?
3. Describe a time optimizing your CI/CD pipeline meaningfully improved team productivity.

### Cloud Platforms
4. Describe migrating infrastructure to the cloud — your role and tools used?
5. Describe troubleshooting a production issue in the cloud — steps taken?
6. Experience with IaC tools (Terraform/CloudFormation) — how did you implement/manage changes?

### Containers & Orchestration
7. Describe containerizing an application — benefits and unexpected challenges?
8. Walk through debugging a Kubernetes cluster issue in production.
9. Describe configuring Kubernetes for scaling or load balancing.

### Monitoring & Security
10. Describe proactively catching a performance issue through monitoring.
11. Describe responding to a discovered security vulnerability in CI/CD or infra.
12. Describe implementing logging & alerting from scratch — tools chosen and why?

### Collaboration & Culture
13. Describe bridging the gap between Dev and Ops teams.
14. Describe a high-pressure production incident — how did the team handle it, what did you learn?
15. Describe a process improvement you introduced — how was it received?

---

## 🎬 Sample Behavioral Answers (Real Scenarios to Reuse)

### CI/CD Setup & Failures
- **Setting up CI/CD from scratch:** Migrating from Jenkins to AWS-native tools (CodePipeline/CodeBuild). Challenges: granular IAM roles, cross-service permissions (S3 bucket policies), repo integration tokens, secure webhook triggering.
- **Unexpected pipeline failure:** Root causes — missing build dependencies, agent/runner outages, broken webhook secrets. Resolution — check build logs, inspect executor availability, add automated retry logic.
- **Productivity improvement:** Added Git pre-commit hooks for coding/security standards; shifted feedback left by auto-running tests on push instead of manual dev deployments.

### Cloud Infrastructure & Troubleshooting
- **Cloud migration:** Multi-cloud migration (Azure → AWS) using AWS DMS for large-scale data transfer.
- **Production outage (502 Bad Gateway):** Isolated backend pods crashing/OOMKilled → checked container runtime logs, inspected resource limits, collaborated with dev team on root cause.
- **IaC change management:** Managed Terraform state/drift; enforced environment promotion gates (dev/staging first, then peer-reviewed plans before prod).

### Containers & Kubernetes
- **Containerization benefits:** Consistent runtime environments, simpler dependency management, predictable scaling. Challenges: deconstructing monolithic dependencies, secure config/secret injection.
- **Cluster outage escalation:** Assess multi-namespace impact if control plane/worker nodes fail; escalate to managed cloud support (AWS/Azure) for control-plane issues; follow with RCA.

### Monitoring & Security
- **Proactive detection:** Caught CPU climbing above 80–90% or disk saturation from unrotated logs, resolved before SLA breach.
- **Security remediation:** Fixed overly permissive IAM roles and unauthenticated endpoints; enforced centralized identity (AD/Azure AD/Entra ID SSO) and DR/snapshot strategies.

### Culture & Collaboration
- **Bridging Dev & Ops:** Wrote self-service deployment scripts and standardized pipeline templates so developers could deploy safely without waiting on Ops; built shared observability dashboards to cut MTTR.

### 🧩 Detailed Real-World Scenario Answers

**1. Database Saturation & CrashLoopBackOff:**
> Pods hit OOM errors and CrashLoopBackOff due to connection exhaustion from unoptimized, massive DB tables.
> - **Short-term fix:** Purged historical records older than 3 months, added DB indexes to cut query latency.
> - **Long-term fix:** Scheduled automation jobs to archive/clean historical data periodically.

**2. Post-Deployment Data Inconsistencies:**
> Runtime errors after a release due to missing DB column values / config drift.
> - **Process:** Isolated pod logs → replicated & verified fixes in lower environments first → applied hotfix to prod.

**3. Terraform: Ad-hoc Scripts → Modular Code:**
> Early infra was provisioned via non-standardized scripts. As the fleet grew, migrated to reusable Terraform modules to eliminate boilerplate duplication.
> - **State management:** Fixed concurrency/drift issues by moving state to a **remote backend with locking** (S3 + DynamoDB), storing code in Git, running plan/apply via CI/CD with mandatory PR review.
> - **Why Terraform over CloudFormation:** Unified syntax across multi-cloud (Azure + AWS).

**4. VM → Kubernetes Containerization:**
> Traditional VM setups had silent weekend outages only caught Monday morning.
> - **Benefit:** Kubernetes gave automated self-healing, auto-restarts, predictable resource isolation — drastically cut recurring downtime.
> - **Challenge:** Steep learning curve on K8s architecture and container networking debugging; more operational complexity than monolithic VMs.

**5. DR Drill Failure & Recovery:**
> After a DR drill restore, workloads across multiple namespaces failed.
> - **Debug steps:** `kubectl get pods -A` to isolate failing pods → `kubectl describe pod` to check events/timestamps → found mismatched env vars & broken ConfigMaps from the restore.
> - **Fix:** Rolled back to the previous stable snapshot/release to restore SLA uptime, then did post-incident RCA to fix the restoration automation scripts.

**6. Auto-Scaling & Load Balancing:**
> - **Auto-scaling:** HPA scales pod replicas on CPU/memory thresholds; Cluster Autoscaler adds/removes worker nodes when pending pods exceed capacity.
> - **Traffic routing:** AWS ALB (via AWS Load Balancer Controller/Ingress) for L7 path-based routing into K8s target groups/microservices.

---

## ⚙️ GitHub Actions Basics

### Key Concepts
| Concept | Description |
|---------|--------------|
| **Workflows** | Defined in `.github/workflows/*.yml`; triggered by GitHub events (push, PR, issue) |
| **Jobs** | Each workflow has one or more jobs; each runs on a fresh VM (Ubuntu/Windows/macOS) |
| **Steps** | Ordered actions within a job — a shell command or a reusable action |
| **Actions** | Reusable plugins/scripts (checkout code, set up a language env, deploy, etc.) |

### Example: Simple Node.js CI Workflow
```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
```

### Common Use Cases
- Running tests on pull requests
- Linting and formatting checks
- Deploying to AWS, Vercel, Netlify, Firebase, etc.
- Releasing packages to npm or PyPI
- Automated code reviews / issue triaging

---

## 🏦 Banking Sector — Domain-Specific Interview Questions

### Security & Compliance
1. How would you implement IaC while meeting compliance standards like PCI-DSS or SOX?
2. How do you manage secrets securely across environments?
3. What tools/practices ensure auditability and traceability in CI/CD pipelines?

### CI/CD & Automation
4. What CI/CD pipeline would you design for a banking platform with multiple microservices?
5. How would you prevent unauthorized code changes or deployments in production?
6. Which CI/CD tools do you prefer and why (Jenkins, GitLab, Azure DevOps)?
7. How do you implement automated rollback strategies in CI/CD pipelines?

### Containers & Infrastructure
8. How would you secure a Docker image handling sensitive financial transactions?
9. Horizontal vs. vertical scaling — when would you use each in banking apps?
10. How do you handle zero-downtime deployments in Kubernetes for critical financial systems?

---

## 🏥 Healthcare & Insurance Domain — DevOps Considerations

### Why Domain-Specific Rounds Happen
Large enterprises run account/vertical-specific rounds (Healthcare, Insurance, Banking, Automobile) that go beyond general tool proficiency — they test your understanding of **regulatory compliance, strict SLAs, and data handling constraints**.

### Core Themes

**A. Data Retention & Fast Historical Access**
- Patient records/claims history must be retained for years or decades (regulatory requirement)
- Must be quickly retrievable during doctor visits or claim assessments — no latency issues
- **DevOps focus:** Tiered storage (hot/warm/cold — e.g., S3 Standard vs. Glacier), automated lifecycle policies, efficient DB indexing

**B. Data Security, Privacy & Pipeline Governance**
- Handles PII (Personally Identifiable Information) and PHI (Protected Health Information)
- Pipeline compliance: credentials/connection strings/tokens must never be logged or exposed during build/test
- Data integrity: deployments and schema migrations must never corrupt historical records in transit

**C. High Availability, Zero Downtime & Multi-Region DR**
- Outages directly impact patient care and real-time claim authorizations
- Key questions: zero-downtime deployment strategy (blue/green, canary), automated rollback on failed health checks, multi-region DR to survive region outages

### Typical Interview Questions
- How do you meet HIPAA/GDPR/SOC2 alignment while architecting CI/CD pipelines?
- What database engines, storage tiers, and load balancing patterns balance speed, encryption (at rest/in transit), and long-term retention?
- How do you guarantee CD scripts and schema migrations maintain data integrity without loss or drift?
- How do you structure multi-region deployments, automated failover, and verified backup/restore drills for zero-data-loss SLAs?

---

## ☁️ Cloud Migration Strategy — The 6 Rs

| R | Meaning | Example |
|---|---------|---------|
| **Rehost** (Lift & Shift) | Move workloads as-is, no redesign | Azure VM → AWS EC2 with matching vCPU/memory/OS |
| **Replatform** (Lift, Tinker & Shift) | Minor optimizations using cloud-managed services, no core code changes | Self-hosted DB → Amazon Aurora/RDS |
| **Refactor / Re-architect** | Re-engineer into cloud-native architecture | Monolith → microservices on containers/serverless |
| **Repurchase** | Replace custom software with off-the-shelf SaaS | — |
| **Retain** | Keep non-migratable/compliance-bound components on-prem (hybrid) | — |
| **Retire** | Decommission obsolete servers/services no longer needed | — |

### Cross-Cloud Service Mapping: Azure → AWS

| Azure | AWS Equivalent | Notes |
|-------|------------------|-------|
| Azure Kubernetes Service (AKS) | Amazon EKS | Node groups, CNI networking, pod manifests |
| Azure PostgreSQL Flexible Server | Amazon Aurora / RDS | Aurora for high performance + managed scaling |
| Management Groups & Resource Groups | AWS Organizations & Member Accounts | Azure = Resource Groups; AWS = separate Accounts + IAM boundaries |
| Network Security Groups (NSGs) | Security Groups / NACLs | Subnet routing, VPC peering, endpoint access |

### Practical Migration Workflow (e.g., Java Microservices: Azure → AWS)

1. **Pre-Migration Backups** — full snapshots of source databases and storage volumes
2. **Database Migration via AWS DMS:**
   - Set up an AWS DMS replication instance
   - Ensure network reachability between source Azure DB and target AWS RDS/Aurora
   - Define table mapping rules, schema replication, run full-load + CDC (Change Data Capture) tasks
3. **Application & Cluster Provisioning:**
   - Provision target EKS clusters, VPCs, subnets, node groups via IaC
   - Deploy application manifests/Helm charts, validate pod health
4. **Staging & Production Cutover:**
   - Run the full migration runbook in a lower/sandbox environment first to measure replication duration and catch errors
   - Schedule the production maintenance window, apply final delta syncs, re-point DNS/load balancer to AWS, monitor live telemetry

---

## 📝 More Practical DevOps Interview Questions (Practice List)

1. How do you check open ports in a Linux system?
2. What are the benefits of using a firewall?
3. Write a simple Terraform script to create a VM/EC2 instance.
4. Write a manifest file (`pod.yaml`) for a single container (database service).
5. Difference between Pod and Deployment — write a `deployment.yaml` for a database service with 3 replicas.
6. Difference between NodePort, ClusterIP, and LoadBalancer — where do you use each?
7. What is Helm? Explain its components.
8. Write a Dockerfile for a Node.js application.
9. What is a base image in a Dockerfile? How do you choose one?
10. What are liveness and readiness probes?
11. Difference between Deployment, ReplicaSet, and ReplicationController?
12. How do you do port forwarding in Docker and Kubernetes?

---

## 🧭 Essential `kubectl` Commands Cheat Sheet

### Declarative vs. Imperative
| Approach | Description |
|----------|--------------|
| **Imperative** | Direct CLI commands: `kubectl create ns test`, `kubectl run nginx --image=nginx`, `kubectl scale deployment ...` |
| **Declarative** | Define desired state in YAML/JSON, then: `kubectl apply -f manifest.yaml` |

### Inspecting Resources
```bash
kubectl get ns                          # List namespaces (default: default, kube-system, kube-public)
kubectl get pods -n <namespace_name>    # List pods in a namespace
kubectl describe pod <pod_name>         # Detailed info: events, restart reasons, image pull status, IP, node
kubectl get pods -o wide                # Extended output — pod IP + hosting node
```

### Resource Creation
```bash
# Imperative
kubectl create ns my-app
kubectl run my-pod --image=nginx -n my-app

# Declarative
kubectl apply -f ingress-gateway.yaml -n <namespace>
```

### Debugging & Logging
```bash
kubectl logs <pod_name>                       # View container logs
kubectl exec -it <pod_name> -- /bin/bash      # Interactive shell (use /bin/sh if bash unavailable)
kubectl get pods --watch                      # Live-stream pod status transitions
```

### Scaling & Deletion
```bash
kubectl scale deployment <deployment_name> --replicas=2
kubectl delete pod <pod_name>
kubectl delete ns <namespace_name>
```

---

## ⏪ Rollback Strategies in DevOps (GitOps + Jenkins)

### The Interview Problem
> Vague answer: *"We just deploy the previous image."* Interviewers actually want to know: **how is the rollback systematically automated** in the CI/CD + GitOps workflow?

### Base GitOps Deployment Workflow
1. Build & push microservice container images to a registry (Docker Hub / ECR / Nexus)
2. Update the Kubernetes YAML manifest repo with the new image tag
3. A GitOps controller (**Flux CD** or **ArgoCD**) detects the Git commit and syncs the new desired state to the cluster

```
Docker Image → Docker Hub → Archive current manifest as artifact →
Update K8s Manifest (GitHub Repo) → Flux CD → Deployment
```

### Strategy 1: Parameterized Rollback Pipeline via Registry API

- Keep rollback as a **separate, dedicated pipeline** (don't complicate the main build pipeline)
- Accept `service_name` as a parameterized input
- **Query the previous stable tag** via the container registry API (e.g., Docker Hub API)
- Use `jq` on the Jenkins agent to parse the JSON response and extract the tag just before the latest build
- **Commit the older image tag** back into the K8s manifest repo → Flux CD automatically reconciles and rolls back

> ⚠️ `jq` must be installed on the Jenkins agent running this pipeline.

### Strategy 2: Jenkins Archive Artifacts Snapshot

Avoids querying the external registry entirely — keeps the previous state within CI build history:

1. **In the main pipeline:** Before patching the K8s manifest with the new image tag, **back up the existing manifest file** and save it via Jenkins' `archiveArtifacts` step
2. **On rollback trigger:** The rollback pipeline pulls the archived manifest from the last successful build and commits that known-good manifest back to Git — letting the GitOps engine restore the previous state automatically, with no manual edits

### 🎯 Interview One-Liner
> "Our main pipeline builds and pushes the image, then updates the Kubernetes manifest repo, which Flux CD syncs to the cluster. For rollbacks, I maintain a separate parameterized pipeline that either queries the registry API for the previous stable tag using `jq`, or restores a manifest snapshot I archived as a Jenkins build artifact — both approaches commit the known-good state back to Git so GitOps handles the actual rollback."

---

## 🗺️ DevOps Learning Roadmap (Step-by-Step Sequence)

> A structured path to avoid gaps/missing prerequisites when learning DevOps from scratch:

1. **Linux Fundamentals** — filesystem hierarchy, disk mount points, permissions; troubleshooting CPU/memory/uptime/process management; basic networking (`ping`, checking open ports)
2. **Shell Scripting** — bash scripts with conditionals/loops/arrays; parsing CLI output with `grep`, `awk`, `sed`, `cut`
3. **Git & GitHub** — working dir/staging/local repo/remote branches; `add`/`commit`/`push`/`pull`/merge, `.gitignore`, Personal Access Tokens (PAT)
4. **CI/CD Pipelines** (Jenkins/GitHub Actions/GitLab CI) — checkout, build (Maven/npm), push artifacts/images, secrets management, parameterized builds, webhook triggers
5. **Cloud Infrastructure & Networking** — VPCs/VNets, public/private subnets, CIDR blocks, Internet/NAT Gateways, 3-tier design, DNS (Route 53), ALBs
6. **Containerization & Orchestration** — Dockerfile directives, image building, registries; Kubernetes Pods/Deployments/Services/Ingress
7. **IaC & Config Management** — Terraform for cloud provisioning; Ansible as a secondary tool for config management
8. **GitOps & Advanced Automation** — ArgoCD/Flux CD; Python for cloud automation scripts; DevSecOps scanning (SonarQube, Trivy)

---

## ⚙️ GitHub Actions — Interview Questions & Answers

### Core Questions to Prepare For
1. How do you implement a CI/CD pipeline using GitHub Actions?
2. How do you handle rollback in a deployment pipeline?
3. How would you integrate GitHub Actions with Slack, AWS, DockerHub, or Jira?
4. What are self-hosted runners and when should you use them?
5. How do you manage deployment to multiple environments (staging, prod)?
6. How do you create and publish a custom GitHub Action?
7. How would you debug a failing workflow?
8. What are the security best practices?

### Detailed Answers

**Workflow Architecture & Triggers**
- Workflows live in `.github/workflows/*.yml`
- Trigger events: `push`, `pull_request`, scheduled (`schedule` — cron), or manual (`workflow_dispatch`)
- Hierarchy: **Workflows → Jobs → Steps/Actions**

**Rollback Strategies**
- Revert the commit/tag, or re-deploy the prior immutable image/release artifact

**External Integrations**
- Slack notifications, Jira ticket updates, AWS IAM OIDC auth for secretless cloud access

**GitHub-Hosted vs. Self-Hosted Runners**

| Type | Use When |
|------|-----------|
| **GitHub-Hosted** | Standard, ephemeral, no special requirements |
| **Self-Hosted** | Private VPC access needed, proprietary build tools, specialized hardware/caching |

**Multi-Environment Deployments**
- Configure **GitHub Environments** (Dev, Staging, Prod) with required protection rules, environment-specific secrets, and manual reviewer approvals

**Custom GitHub Actions**
- Build reusable **Composite Actions**, **Docker container actions**, or **JavaScript actions**; publish for internal/public reuse

**Workflow Debugging**
- Review live step logs; enable `ACTIONS_STEP_DEBUG` for verbose runner logs; re-run failed jobs

**Security Best Practices**
- Store tokens strictly in **GitHub Actions Secrets** — never in code
- Limit `GITHUB_TOKEN` permissions via explicit `permissions:` blocks (Principle of Least Privilege)
- Pin action versions to **specific commit SHAs**, not mutable tags (e.g., `@v3` can change; a SHA can't)

---

## 🧩 GitHub Actions Contexts — Deep Dive

### What Are Contexts?
Built-in objects containing metadata about a workflow run, the runner environment, secrets, and the triggering event. Accessed via:
```yaml
${{ <context>.<property> }}
# e.g. ${{ github.ref }}, ${{ secrets.DOCKER_TOKEN }}
```

### Common Context Objects

| Context | Purpose | Example Use |
|---------|---------|--------------|
| `github` | Run & git metadata (`github.actor`, `github.event`, `github.repository`, `github.ref`, `github.run_number`) | Branch checks, event inspection, audit trail |
| `secrets` | Secure vault values (`secrets.MY_TOKEN`) | Masked API keys, registry tokens, SSH keys |
| `env` | Environment variables at workflow/job/step level | Passing custom config to steps |
| `runner` | Runner metadata (`runner.os`, `runner.arch`, `runner.temp`) | OS-conditional logic (Linux vs. Windows) |
| `job` / `steps` | Status & outputs of current jobs/steps | Inter-job dependencies, execution tracking |

### Conditional Execution
```yaml
if: github.ref == 'refs/heads/main'
```

### Accessing Commit Messages
```yaml
${{ github.event.head_commit.message }}
```

### ⚠️ Security: Script Injection Precaution
Interpolating context values directly into inline bash is dangerous:
```yaml
# ❌ Risky — direct interpolation
run: echo "${{ github.event.issue.title }}"

# ✅ Safe — assign to an env var first
env:
  TITLE: ${{ github.event.issue.title }}
run: echo "$TITLE"
```

### Debugging Contexts at Runtime
```yaml
- name: Dump GitHub Context
  run: echo '${{ toJSON(github) }}'
```

### Dynamic Artifact Naming
Append `${{ github.run_number }}` or `${{ github.sha }}` for unique, deterministic artifact/release names.

### Real-World Scenario Answers

| Scenario | Solution |
|----------|----------|
| Deploy only when PR has a specific label | `if: github.event.pull_request && contains(github.event.pull_request.labels.*.name, 'deploy')` |
| Set target URL dynamically by branch | Evaluate `${{ github.ref }}` — route `refs/heads/main` to prod, feature branches to preview/lower envs |
| Restrict execution to specific users | `if: contains(fromJSON('["lead-admin", "authorized-dev"]'), github.actor)` |

---

## 🔵🟢 Blue-Green Deployment — Explained Simply

### Core Concept
| Environment | Role |
|-------------|------|
| **Blue** | Active — currently serving all live production traffic (older version) |
| **Green** | Idle — new version deployed and thoroughly tested under production-like conditions |

**Zero-downtime cutover:** Once Green passes validation, the router/load balancer switches traffic from Blue to Green **instantly**. Blue stays on standby for immediate rollback, or is decommissioned later.

### How Traffic Routing Actually Happens (Ingress/Service Level)
Rather than replacing running pods in-place, redirection happens at the networking layer:
- **Ingress backend switch:** Update `backend.service.name` from `blue-service` → `green-service`
- **Service selector switch:** Change the Service's label selector (e.g., `version: v1.0` → `version: v2.0`) to point traffic to Green pods
- The DNS/URL stays **identical** for end users throughout

### Lab Practice vs. Real Enterprise Reality

| Context | Approach |
|---------|----------|
| **Local Lab (Minikube)** | Simulate with multiple profiles: `minikube start -p green --driver=docker` |
| **Enterprise Myth** | "We provision two entire duplicate Kubernetes clusters" — too expensive in practice |
| **Enterprise Reality** | Two **namespaces** in the same cluster, shifting Ingress/mesh traffic between them; OR side-by-side `app-blue`/`app-green` Deployments in the **same namespace**, switching the Service selector or Service Mesh (Istio/Linkerd) traffic weight; OR dedicated node pools via node selectors/taints |

---

## 💼 DevOps Interview Questions for Experienced Candidates (2+ Years)

> 💡 **Mindset note:** Don't let early rejections shake your confidence in later rounds — like the "tied elephant" that stops trying to break free after early failures despite having grown stronger. Trust your growth.

### 1. CI/CD & Build Automation

**Core CI/CD pipeline stages:**
```
Source checkout → Unit tests/Build → Static Code Analysis (SonarQube)
→ Container image build → Security scan (Trivy) → Push to registry → Deployment trigger
```

**Explaining your tool choices:** Be ready to justify tools based on business domain (Fintech, AI/Automation) and platform constraints, and describe your full stack: Version Control → CI Engine → Container Registry → Orchestrator → Observability.

**Declarative vs. Scripted Jenkins Pipelines:**

| Type | Description |
|------|--------------|
| **Scripted** | Groovy-based, imperative, more flexible but more complex |
| **Declarative** | Modern, structured syntax: `pipeline { agent any stages { ... } }` |

**Automating deployments via Jenkins:** Distinguish CI tasks (build/test) from CD workflows (triggering GitOps controllers like ArgoCD/Flux, or invoking deployment webhooks/scripts).

### 2. Version Control & Virtualization

**Resolving Git merge conflicts:** Explain how conflicts arise between feature and base branches → identify conflict markers → manually reconcile in editor → run tests to validate → complete the merge commit.

**Containers vs. VMs:**

| | Containers | Virtual Machines |
|---|---|---|
| Virtualization level | OS-level, share host kernel | Hardware-level, hypervisor-based |
| Footprint | Lightweight | Full dedicated guest OS |

**Docker Networking:** Network drivers (`bridge`, `host`, `overlay`, `none`), container-to-container communication, port mapping (`-p`), and port exposure (`EXPOSE`).

### 3. Orchestration & Cloud Infrastructure

**Kubernetes Rolling Updates:** The Deployment controller manages phased pod replacement using `maxSurge` and `maxUnavailable` to prevent service interruption.

**Structuring your cloud experience for interviews:**
```
Networking (VPCs, Subnets, Gateways) → Compute (EC2, Node Groups)
→ Databases (RDS, Aurora) → IAM & Observability
```

**Prometheus & Grafana:** Prometheus scrapes metrics and stores them as time-series data; Grafana visualizes them on dashboards and handles alerting rules.

### 4. Behavioral & Scenario-Based Questions

**Handling a failed deployment (use the STAR method):**
1. What failed & the business impact
2. How you tracked it down (APM traces, pod logs, error status codes)
3. Resolution steps (immediate rollback vs. hotfix)
4. MTTR (Mean Time to Recovery) and preventive actions taken afterward

**Ensuring zero-downtime releases:** Progressive delivery patterns — Canary releases, Blue-Green deployments, readiness probes.

**Managing staging vs. production parity:** Handling configuration drift, per-environment secrets/variables, approval gates, compliance auditing.

---

## 💻 Hands-On Practice Prompts (Common Live-Coding Asks)

### Shell Scripting
1. Check if a file exists at a given path and whether it's writable
2. Print a list of names read from an input file
3. Create a file named with today's date in `yyyy-MM-dd` format
4. Mount a disk on a given filesystem
5. Word count, grep a specific word, first/last 10 lines, count lines with `sed`

### Jenkins
- Write a basic declarative pipeline script

### Docker
- Write a Dockerfile to containerize a Python app

### Kubernetes
- Write a manifest for a Persistent Volume
- Write a manifest template for a Pod or Deployment

### Terraform
- Write a script to create an EC2 instance including VPC and subnet — and explain how to retrieve outputs

### Ansible
- Write a sample playbook to install `git`

---

## ⛵ Helm — Interview Questions & Answers

### What is Helm?
> Helm is **the package manager for Kubernetes** — the same role `apt` plays for Ubuntu/Debian or `yum`/`dnf` plays for RHEL/CentOS.

**Architecture note:** Helm 2 required an in-cluster component called **Tiller** (a security risk). **Helm 3 removed Tiller entirely**, using the user's kubeconfig and RBAC directly.

### Helm Chart Structure
```
mychart/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml          # Default config values injected into templates
├── templates/            # Deployments, Services, Ingress (Go template engine)
│   └── _helpers.tpl      # Reusable template snippets/naming conventions
└── charts/               # Sub-chart dependencies
```

### Installing a Chart
```bash
helm install <release-name> <chart-name-or-path>

# From an official repo:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx
```

### `helm install` vs. `helm upgrade`

| Command | Behavior |
|---------|----------|
| `helm install` | Deploys a new release — **fails** if that release name already exists |
| `helm upgrade` | Updates an existing release (new version/values), increments the revision |

> **Best practice:** `helm upgrade --install <release> <chart>` — installs if new, upgrades if it already exists.

### Viewing Releases
```bash
helm list          # current namespace
helm list -A        # all namespaces
helm history <release-name>
```

### Purpose of `values.yaml`
Defines default config variables (image tags, replica counts, resource limits, ports, ingress hosts) that parameterize the templates.

### Overriding Default Values
```bash
# Method 1: custom YAML file (recommended for GitOps/multi-env)
helm upgrade --install my-app ./my-chart -f values-prod.yaml

# Method 2: CLI flags
helm upgrade --install my-app ./my-chart --set replicaCount=3 --set image.tag="v2.1.0"
```

### What is a Helm Release?
A specific **running instance** of a chart deployed to a cluster. The same chart can be deployed multiple times under different release names (`app-dev`, `app-staging`).

### Uninstalling a Release
```bash
helm uninstall <release-name> -n <namespace>
```

---

## 🔧 Jenkins — Real-World Scenario-Based Interview Questions

### 1. Multi-Environment Deployments (Dev → Staging → Prod)
- **Parameterized pipelines:** build parameters (`choice`, `string`, active choices) or config files to select the target environment
- **Conditional execution:** `when { expression { ... } }` blocks for environment-specific logic
- **Folder/scope-level configs:** store env credentials/variables at the Jenkins folder level to avoid cross-contamination

### 2. Handling Pipeline Timeout & Partial Failure
- **State tracking/idempotency:** treat configs like IaC state — verify applied state rather than assume it
- **Retry mechanisms:** wrap transient ops in `retry(count) { ... }` with `timeout(time: 10, unit: 'MINUTES')`
- **Restart from failed stage:** re-trigger from the specific failed stage instead of rebuilding from scratch

### 3. Production Deployment Gates & Approvals
```groovy
input message: 'Approve Prod Deployment?', submitter: 'qa-leads'
```
- **Enterprise auditing:** integrate with ServiceNow/Jira — pipeline queries the change request state and only proceeds once it's **Approved**

### 4. Secret & Credential Handling
- Use **Jenkins Credentials Provider** + `withCredentials` binding (`usernamePassword`, `amazonWebServicesCredentials`, `string`) — ensures secrets are masked in console logs

### 5. Automated Rollbacks on Failure
```groovy
post {
    failure {
        // fetch previously archived known-good artifact
        // or trigger a rollback workflow to redeploy the prior stable tag
    }
}
```

### 6. Unique Docker Image Tagging
- Tag with Git commit SHA (`GIT_COMMIT`) or semver + build number (`v1.2.0-${BUILD_NUMBER}`)
- **Avoid mutable tags like `latest`** — enables exact traceability back to source

### 7. Optimizing Long-Running Test Suites
- Use the `parallel` block to distribute independent test suites (unit, functional, linting, security scans) across multiple agents/executors simultaneously

---

## 🎯 Career Advice: Why Keep Interviewing Even With an Offer in Hand

### Core Reasons to Keep Looking
- **Compensation gap** — the offer doesn't meet your financial expectations
- **Lack of project clarity** — accepting without visibility into the actual tech stack/project can stall your resume growth
- **Location & work-life fit** — commute, remote/hybrid arrangements, personal preferences

### How to Handle New Recruiters While Holding an Offer
- **Be upfront:** disclose immediately that you have an existing offer, and state your compensation/project expectations clearly
- **Respect everyone's time:** transparency avoids wasting time if they can't meet your benchmark or notice period

### Managing Multiple Pipelines
- Most companies need **2–3 rounds** (technical, managerial, HR) — this takes weeks from first screen to final offer letter
- Run **parallel interview pipelines** since offers can fall through — this maximizes the odds of landing something that truly fits before your notice period ends

### Key Takeaways
- **Know your value** — have a firm, realistic compensation/responsibility expectation
- **Demand project & team clarity** before accepting — ask about the account, tech stack, team culture
- **Do your due diligence** — reach out to current/former employees (e.g., on LinkedIn) to verify real working conditions before committing

---

## 🚢 Kubernetes Practical Q&A

**1. How do you deploy an application to Kubernetes?**
1. Containerize the app (Docker)
2. Push the image to a container registry
3. Write Kubernetes manifests (YAML)
4. Deploy the manifests to the cluster
5. Expose the application (optional)

**2. YAML manifest for a 3-replica nginx Deployment:**
```yaml
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

**3. How do you connect a pod to a Persistent Volume using a PVC?**
1. Create a Persistent Volume (PV)
2. Create a Persistent Volume Claim (PVC)
3. Mount the PVC inside your Pod spec

**4. How do you perform a rolling update?**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

**5. How do you check pod logs?**
```bash
kubectl logs pod-name -n namespace-name
```

**6. How do you debug a service that's not accessible?**
```bash
kubectl get svc service-name -o yaml
kubectl get pods --show-labels
kubectl get svc service-name
```

---

## 🐳 Docker — Interview Questions by Difficulty Level

### Beginner Level

| Question | Simple Answer |
|----------|-----------------|
| What is a Dockerfile? | A text manifest of instructions packaging an app + dependencies into an immutable, platform-independent image |
| `ADD` vs. `COPY` | `COPY` copies local files/dirs plainly. `ADD` also auto-extracts local tar archives and can fetch remote URLs |
| `ENTRYPOINT` vs. `CMD` | `ENTRYPOINT` = the immutable base executable. `CMD` = default arguments, overridable at `docker run` |
| `WORKDIR` | Sets the working directory for subsequent `RUN`/`CMD`/`ENTRYPOINT`/`COPY` |
| `EXPOSE` | Documents which ports the container listens on (informational — `-p` is still needed at runtime to actually map ports) |
| Multiple `FROM` instructions | Allowed and standard — used for **multi-stage builds** to discard build-time dependencies |
| Reducing image layers | Combine shell commands with `&&` in a single `RUN` (e.g., `apt-get update && apt-get install -y ... && rm -rf /var/lib/apt/lists/*`) |

### Intermediate Level

| Question | Simple Answer |
|----------|-----------------|
| `RUN` vs. `CMD` vs. `ENTRYPOINT` | `RUN` executes at **build time**, commits a new layer. `CMD`/`ENTRYPOINT` execute at **container startup** |
| `ARG` vs. `ENV` | `ARG` only exists during build (`docker build --build-arg`); `ENV` persists into the running container |
| Multi-stage builds | Compile in a "builder" image (Maven/Go/Node), then copy only the compiled output into a lightweight runtime image (Alpine/Distroless) — dramatically smaller final image |
| Risks of `COPY . .` in CI/CD | Blindly copies test artifacts, `.git` history, cache folders, and potential secrets — always pair with a strict `.dockerignore` |
| Image versioning/metadata | Use `LABEL` (maintainer, Git commit SHA, build version) to trace image provenance |

### Advanced Level

| Question | Simple Answer |
|----------|-----------------|
| Handling secrets in builds | Never bake secrets into `ENV`/`ARG`. Inject at runtime via env vars/secret stores, or use BuildKit secret mounts (`--mount=type=secret`) |
| Container starts and immediately exits | Check `docker logs <container_id>`; inspect the `CMD`/`ENTRYPOINT` foreground process — container exits when PID 1 finishes/crashes |
| Risk of `:latest` tag | Breaks reproducibility, silently pulls in unexpected base image changes, hurts caching. **Always pin specific version tags or SHA digests** |
| Docker build cache mechanics | Cache invalidates from the **first modified layer downward**. Put infrequently-changing instructions (e.g., `package.json`/`pom.xml`) **before** copying volatile source code |
| Deterministic builds | Pin explicit base image tags/digests, lock dependencies (`package-lock.json`), avoid mutable downloads |
| Security hardening | Run as non-root (`USER <non-root-uid>`), use minimal base images (Alpine/scratch/Distroless), scan images (Trivy/Grype), keep `.dockerignore` current |

---

## 🔟 Final DevOps Interview Revision — 10 Key Questions

### 1. What kinds of scans go into a CI/CD pipeline?

| Scan Type | Purpose |
|-----------|---------|
| Unit Testing / Code Quality | SonarQube — static analysis, code smells, coverage thresholds |
| **SAST** (Static App Security Testing) | Scans source code for security flaws before build |
| **DAST** (Dynamic App Security Testing) | Tests running applications/endpoints for runtime/API vulnerabilities |
| **SCA** (Software Composition Analysis) | Scans dependencies/third-party libraries (e.g., Snyk) |
| Container Image Scanning | Scans layers/OS packages for CVEs before registry push (Trivy, Aqua) |

### 2. What are Kubernetes Services and why are they useful?
Service types: `ClusterIP` (internal default), `NodePort`, `LoadBalancer`, and Ingress controllers.

> ⚠️ **Interview trap:** Interviewers rarely ask "what is a Service?" directly. They ask scenario questions like *"How do two pods find each other across nodes?"* or *"How does a backend talk to a DB pod without hardcoded IPs?"* — the answer to **all of these** is: **a Kubernetes Service** (stable virtual IP, DNS resolution, internal load balancing).

### 3. How does a CI/CD pipeline get triggered?
- **Push/Pull webhooks** — auto-trigger on `push`/`pull_request`
- **Branch-specific strategy:** feature/dev branches trigger automatically for fast feedback; main/release branches often need manual trigger (`workflow_dispatch`) or gated release tags to prevent accidental prod deploys
- **Scheduled runs** — nightly regression/security scans via cron

### 4. What is your Git branching strategy? (GitFlow)
| Branch | Purpose |
|--------|---------|
| `main`/`master` | Production-ready releases |
| `develop` | Integration branch for incoming features |
| `feature/*` | Branched off `develop`, merged back via PR |
| `release/*` | Branched from `develop` for release prep — bug fixes/metadata, then merges into both `main` and `develop` |
| `hotfix/*` | Branched from `main` for critical prod fixes, merged into both `main` and `develop` |

### 5 & 6. Ansible + Windows
| Question | Answer |
|----------|--------|
| Can the Ansible **control node** run on Windows? | **No** — requires a POSIX-compliant environment (Linux/Unix). On Windows, it can only run inside **WSL** |
| Can Ansible **manage** Windows targets? | **Yes** — via the **WinRM** connection plugin |

### 7. How do you manage secrets?
- Never hardcode tokens/passwords in pipeline scripts or repos
- Centralize in vaults: **HashiCorp Vault, AWS Secrets Manager, Azure Key Vault**
- Pipelines authenticate via short-lived tokens, IAM roles, or OIDC — fetching secrets dynamically at runtime

### 8. What scans should you run on Docker images?
Container/CVE scanning (Trivy, Aqua Security) on image layers and OS packages before pushing to a registry.

### 9. Your CI/CD pipeline is too slow — how do you optimize it?
- **Set explicit timeouts** (`timeout: 10m`) to prevent hung jobs blocking runners
- **Identify bottlenecks** — check if delay is in dependency downloads, long test suites, or slow container builds
- **Remediation:** parallel test execution, build/Docker layer caching, dedicated runner agents

### 10. `git fetch` vs. `git pull`

| Command | What It Does |
|---------|----------------|
| `git fetch` | Downloads new commits/branches/tags to local Git metadata — **does not touch your working directory** |
| `git pull` | `git fetch` + `git merge` (or rebase) — actively integrates remote changes into your current branch |

> **Formula:** `git pull` = `git fetch` + `git merge`

---

## 🏛️ DevOps Culture, Maturity & SDLC Frameworks (Senior/Architect Level)

> 💡 **Note:** In interviews, reference sheets give concise bullets — but you must **elaborate on real-world mechanics and architecture patterns**, not just recite one-liners.

### 1. DevOps Maturity Model
Evaluates organizational transformation across **automation, cross-team collaboration, continuous delivery, and observability**. Higher maturity = faster release cadence with minimal manual intervention.

### 2. 12-Factor App Methodology
Principles for building cloud-native SaaS apps:
- Strict config/code separation
- Stateless processes
- Backing service abstraction
- Disposability (fast startup/graceful shutdown)
- Dev/prod parity

### 3. SRE vs. DevOps
> **SRE implements DevOps** through concrete engineering metrics:
- **SLIs** (Service Level Indicators) — what you measure
- **SLOs** (Service Level Objectives) — the target for that measurement
- **Error Budgets** — how much unreliability is acceptable before you must slow down feature work and focus on reliability

---

## 🔒 Microservices, CI/CD & DevSecOps (Senior Level)

### Pipelines for Microservices
- Design **independent, decoupled pipelines** per service
- Build **immutable container artifacts** tagged with Git commit SHAs
- Use **progressive rollouts** (Canary / Blue-Green)

### Pipeline Hardening
- Avoid embedded credentials — use **dynamic secret injection** (Vault/AWS Secrets Manager)
- **Artifact signing**
- **Vulnerability scanning** (SAST/DAST/Trivy)
- **Audit logging**

### Artifact Management
Store versioned binaries/image layers in centralized registries: **JFrog Artifactory, Sonatype Nexus, or Amazon ECR**.

---

## ☸️ Kubernetes & Cloud-Native Architecture (Senior Level)

| Concept | Explanation |
|---------|--------------|
| **Cluster Troubleshooting** | Systematic triage of failing pods (`CrashLoopBackOff`, `ImagePullBackOff`, `OOMKilled`) using `kubectl describe`, event timelines, container logs |
| **Sidecar Pattern** | Auxiliary container deployed alongside the main app container — for logging, proxying, or secret sync |
| **Service Mesh** | Manages service-to-service communication, **mTLS encryption**, traffic shifting, observability (Istio, Linkerd) |
| **GitOps** | Reconciles cluster state automatically against a declarative Git repo (Flux CD / ArgoCD) |

---

## 🏗️ IaC & Cloud Operations at Scale (Senior Level)

### Terraform State Management
- Store `.tfstate` in **remote backends with state locking** (e.g., S3 + DynamoDB) to prevent concurrent overwrites and detect drift

### Structuring IaC at Scale
- Reusable **modules**, remote **workspaces**, environment-specific variable files (`dev.tfvars`, `prod.tfvars`)

### High Availability & Disaster Recovery
- **Multi-AZ / Multi-Region** failover
- Autoscaling groups
- **Layer-4 (NLB)** vs. **Layer-7 (ALB)** load balancers

---

## 🔭 Observability & Reliability (Senior Level)

| Concept | Explanation |
|---------|--------------|
| **Distributed Tracing** | Tracks end-to-end request journeys across microservices (Jaeger, Zipkin, AWS X-Ray) |
| **Production Incident RCA** | Log aggregation → metric spikes → post-incident review → permanent remediation plans |

---

## 🎯 DevOps Interview Questions — 2-3 Years Experience

### 1. CI/CD & Pipeline Design
- **End-to-end pipeline:** code checkout → build/package (Maven for Java, pip/npm for Python/Node) → security scans → container image generation → deployment
- **Rollback strategy:** automated pipeline rollbacks, GitOps reconciliation, image tag updates, or rollback scripts
- **Pipeline challenges:** inter-tool auth/credential integration, runner capacity/timeouts, scan-stage bottlenecks, pipeline security hardening

### 2. Docker & Container Management
- **Reduce image size:** minimal base images (Alpine, Distroless), multi-stage builds, chained `RUN` commands, avoid unnecessary deps
- **Cleanup:** `docker system prune -a` to remove dangling/unused images, containers, volumes

### 3. Kubernetes
- **Exposing services externally:** `NodePort` vs. cloud `LoadBalancer` vs. Ingress controllers (L7 path/host-based routing)

### 4. Linux Administration
- **Load average:** the 1-, 5-, and 15-minute load metrics (`uptime`, `top`) relative to available CPU cores
- **Log rotation:** use `logrotate` to prevent disk saturation
- **Long-running processes:** `ps -eo pid,etime,cmd`, `top`, `htop`

### 5. Cloud Networking
- **VPN tunneling:** secure site-to-site connectivity via VPN or Direct Connect/ExpressRoute
- **Gateways:** Internet Gateways, NAT Gateways/routers, private subnets
- **Load balancer types:** Layer-4 NLB (TCP/UDP, ultra-low latency) vs. Layer-7 ALB (HTTP/HTTPS routing, SSL termination)
- **Cloud security:** network segmentation (Security Groups/NACLs), IAM least-privilege, encryption at rest/in transit

---

## ☁️ Azure + Terraform + Kubernetes + Shell — Deep Practical Q&A

### Kubernetes Troubleshooting

| Scenario | Root Cause / Fix |
|----------|---------------------|
| **Deployment deleted but pods still running** | Orphaned ReplicaSets, pods spawned by another controller (StatefulSet/DaemonSet/standalone), finalizer hooks blocking cleanup, or controller-manager API lag |
| **Image updated in YAML but pod still uses old image** | Check `imagePullPolicy` — using mutable `:latest` tag **without** `imagePullPolicy: Always` prevents kubelet from pulling the new image |
| **HPA configured, threshold exceeded, but pod count stays at 1** | Missing Metrics Server, missing `resources.requests` (HPA needs a baseline to calculate % utilization), or min/max replica constraints |
| **OOMKilled with no resource limits set** | Without limits, a container can consume as much node memory as available. Memory spikes → Linux OOM killer terminates the process (or a namespace `LimitRange` blocks/defaults it) |

### Live-Coding: Full Deployment YAML (probes + PVC + resources)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: my-app-pvc
```

### Shell Script: Directory Check + File Count
```bash
#!/bin/bash
path="$1"

if [ -d "$path" ]; then
  count=$(find "$path" -maxdepth 1 -type f | wc -l)
  echo "Directory exists at: $path — File count: $count"
else
  echo "Not a valid directory: $path"
  exit 1
fi
```

### Terraform: Conditional Resource + Lifecycle + Dynamic Block
```hcl
resource "aws_instance" "example" {
  count = var.environment == "dev" ? 1 : 0

  lifecycle {
    create_before_destroy = true
    prevent_destroy        = false
    ignore_changes          = [tags]
  }

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port = ingress.value.from_port
      to_port   = ingress.value.to_port
      protocol  = ingress.value.protocol
    }
  }
}
```

### Azure Cloud Architecture Q&A

| Question | Answer |
|----------|--------|
| Can an existing subnet be extended from `/24` to `/23`? | **No** — you cannot change the IP range of an existing Azure subnet while resources/NICs are attached to it |
| Managed Identity vs. Service Principal | **Service Principal:** app registration needing manual credential/secret rotation. **Managed Identity:** Azure-managed credential, auto-rotated by Entra ID (System-assigned vs. User-assigned) |
| Application Gateway backend health is down | Mismatched health probe path/port, backend NSG blocking App Gateway subnet traffic, SSL cert mismatch, or backend listening on `localhost` instead of `0.0.0.0` |
| Private Endpoint not reachable from on-prem | DNS resolution failure (private DNS zone not forwarding/resolvable via Azure Private DNS Resolver), missing route tables, VPN/ExpressRoute gateway routing omissions |
| VNet peering — VNet B can't reach VNet A | Non-transitive peering limitations, asymmetric/missing peering (must be set up on **both** sides), overlapping CIDRs, restrictive NSG rules |
| VM creation "denied by policy" | Check Azure Policy definitions and their **effects**: `Deny`, `Audit`, `Modify`, `DeployIfNotExists` |
| Accessing Azure Storage from AKS in a different region | Works across regions but adds latency — verify network/firewall rules and consider using Private Endpoints |
| On-prem → Azure connectivity options | Site-to-Site VPN, Point-to-Site VPN, dedicated private line via **Azure ExpressRoute** |
| Load Balancer vs. Front Door vs. Traffic Manager vs. App Gateway | **LB** = L4 regional. **App Gateway** = L7 regional (path/host routing, WAF). **Front Door** = L7 **global**, edge-based routing/failover. **Traffic Manager** = DNS-based **global** routing |
| Can a Load Balancer work as an Ingress? | No — a standard cloud Load Balancer is L4; **Ingress** requires L7 (HTTP path/host routing), handled by an Ingress Controller |
| Terraform state file has secrets in plaintext | Encrypt the remote backend, use ephemeral outputs, delegate secret retrieval to KMS/Vault at runtime instead of storing raw secrets in state |
| How to recover a deleted Terraform state file | Restore via remote backend **object versioning** (S3/Azure Blob), or rebuild mappings using `terraform import` |
| Find log files older than 7 days | `find /var/log -type f -name "*.log" -mtime +7` |
| What are Private Endpoints vs. Service Endpoints? | **Private Endpoint:** gives the PaaS resource a private IP inside your VNet. **Service Endpoint:** extends VNet identity to the service over the Azure backbone (still uses the service's public IP internally) |
| Types of Managed Identities | **System-assigned** (tied to one resource's lifecycle) and **User-assigned** (standalone, reusable across resources) |
| OAuth for AKS | Configure OIDC/Workload Identity federation so pods can authenticate to Azure AD without storing credentials |
| HTTP Status Codes: 500 / 502 / 503 / 504 | **500** = generic server error. **502** = Bad Gateway (upstream returned invalid response). **503** = Service Unavailable (overloaded/down for maintenance). **504** = Gateway Timeout (upstream didn't respond in time) |
| What are Azure Functions / Web Apps / Logic Apps? | **Functions:** serverless event-driven compute. **Web Apps:** managed PaaS for hosting web applications. **Logic Apps:** low-code workflow automation/integration service |
| Log correlation / Log Analytics for App sets | Configure a shared **Log Analytics Workspace**, use correlation IDs/trace IDs across services, query via KQL |

---

## 🚨 SRE Interview Questions & Answers

### 1. Handling a Midnight Production Outage
1. **Priority 1 — Fast mitigation over deep analysis:** capture diagnostic snapshots, then mitigate fast (rollback latest release, failover to backup region/cluster, restart degraded pods)
2. **Priority 2 — Incident communication:** open an incident war-room/channel, notify on-call leads/stakeholders, provide regular status updates
3. **Priority 3 — Post-recovery RCA:** document the exact timeline, correlate telemetry spikes, preserve logs for RCA

### 2. Reducing MTTR
- **Granular observability:** high-resolution metrics (Prometheus/Grafana) at short scrape intervals
- **Automated runbooks/playbooks:** standardized responses to frequent failures, self-healing scripts (auto-restart hung workers, circuit tripping)
- **Smaller, incremental deployments:** small batch releases + canary rollouts → smaller, faster rollback blast radius

### 3. Troubleshooting High Latency
- **Layer-by-layer diagnostics:** Network/DNS → Ingress/LB → Application logic → Downstream DBs/caches → External 3rd-party APIs
- **Distributed tracing:** Jaeger, Zipkin, OpenTelemetry, AWS X-Ray to pinpoint the slow span (DB query, thread deadlock, blocking network call)

### 4. Debugging Intermittent Microservice Failures
- Inspect APM/logs for error spikes, retry storms, socket exhaustion, timeouts
- **Resilience patterns:** Circuit Breakers (Resilience4j, Envoy/Istio) to stop cascading failures; Exponential Backoff with Jitter on retries
- **Reproduction:** elevate debug logs, simulate peak traffic in staging to safely reproduce race conditions

### 5. Designing for High Availability
- Multi-AZ / Multi-Region, active-active or active-passive failover
- **Stateless service design** — decouple compute from persistent storage
- Enforce **strict timeouts/deadlines** to prevent thread pool exhaustion
- Data layer protection: async read replicas, multi-region replication, automated snapshots

### 6. Observability, Alert Fatigue & RCA
- **Three Pillars of Observability:** Metrics (health/trends), Logs (discrete events), Traces (request flow)
- **Reduce alert fatigue:** deprecate noisy non-actionable alerts; alert on **user-facing SLO symptoms** (elevated error rate, latency breach), not raw CPU spikes that self-resolve
- **Blameless post-mortem:** link metric timestamps to trigger events, identify systemic contributing factors, track preventative action items

---

## ☁️ AWS & Linux Practical Q&A

| Question | Answer |
|----------|--------|
| **Stop communication between pods in different namespaces?** | Define a `NetworkPolicy` restricting ingress/egress — use `namespaceSelector`/`podSelector` to only allow specific traffic; by default all else is denied once a policy exists (CNI-dependent) |
| **Connect a resource in AWS Account A to a resource in Account B?** | **VPC Peering** — create request from Requester, accept in Accepter account, update route tables, adjust security groups/NACLs. Alternatives: **Transit Gateway** (multi-VPC scale), **PrivateLink** (expose specific services), **VPN/Direct Connect** (hybrid) |
| **Increase disk space on a Linux server?** | Two steps: **expand the underlying storage** (EBS volume resize) then **resize the filesystem** to use the new space (`growpart` + `resize2fs`/`xfs_growfs`) |
| **Restrict access to specific S3 objects?** | Use **S3 Bucket Policies** or **IAM policies** with object-level permissions |
| **Install software on EC2 automatically at launch?** | Use **User Data** (bootstrap script executed on first boot) |
| **Where should a NAT Gateway live?** | In a **public subnet**, associated with an Elastic IP, with a route to an Internet Gateway — it lets private subnet instances reach the internet without being exposed to inbound traffic |

---

## 🔟 Top 10 Terraform Scenario-Based Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | State file corrupted (multiple devs applying at once) | Use a **remote backend with locking**: S3 (state) + DynamoDB (lock table) |
| 2 | Drift — someone manually changed a resource | `terraform plan` detects it → `terraform apply` reverts, or `terraform import` if untracked. Enforce IaC discipline to avoid manual changes |
| 3 | Managing Dev/Prod with same infra, different configs | Use **Workspaces** or separate state files + `-var-file=dev.tfvars` / `-var-file=prod.tfvars`; separate backend configs per env |
| 4 | Reusing VPC setup across projects | Create a **Terraform Module** (`module "vpc" { source = "./modules/vpc" ... }`) — promotes DRY & consistency |
| 5 | DB passwords exposed in code | Mark `sensitive = true`, use env vars or a secret manager (AWS Secrets Manager) |
| 6 | Zero-downtime EC2 update | `lifecycle { create_before_destroy = true }` combined with a load balancer for smooth switchover |
| 7 | Resource fails — dependency not ready | Terraform auto-handles most dependencies; enforce explicitly with `depends_on = [aws_instance.app]` when implicit detection fails |
| 8 | Managing hundreds of similar resources | Use `count` or `for_each` (e.g., `for_each = toset(["dev","prod"])`) |
| 9 | Apply failed mid-way | Terraform isn't fully transactional — fix the issue and re-run `terraform apply`; use version control for code rollback; consider blue/green for infra changes |
| 10 | Bring existing infra under Terraform control | `terraform import aws_instance.example i-123456`, then write matching `.tf` config, then `terraform plan` to verify sync |

---

## 🔟 Top 10 Kubernetes Scenario-Based Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | Pod stuck in `CrashLoopBackOff` | `kubectl logs`/`kubectl describe pod` — check for wrong entrypoint, missing env/config, failing liveness probe, or resource limits too low |
| 2 | Service not reachable | `kubectl get svc` (correct type?), `kubectl describe svc` (endpoints populated?), check label selector mismatch, pod readiness, network policies |
| 3 | Pod stuck `Pending` | `kubectl describe pod` — insufficient cluster resources, node selector/affinity mismatch, taints without tolerations, PVC not bound |
| 4 | High CPU usage cluster-wide | `kubectl top pods`, check requests/limits, use HPA, investigate inefficient code/memory leaks, consider cluster autoscaling |
| 5 | Rolling update failure, app down | `kubectl rollout status` → `kubectl rollout undo deployment <name>`; investigate probes, image issues, config errors |
| 6 | ConfigMap updated but pods didn't pick it up | ConfigMaps don't auto-restart pods — run `kubectl rollout restart deployment <name>` |
| 7 | App can't read Secrets | Verify with `kubectl get secrets`, check mount method (env var vs. volume), confirm key names and RBAC permissions |
| 8 | Intermittent pod-to-pod communication failure | Check network policies, DNS (CoreDNS) issues, service misconfig, pod restarts/scaling — debug with `kubectl exec` + `curl`/`nslookup` |
| 9 | Data disappears after pod restart | Pod was using **ephemeral storage** — use a **PV + PVC** with the correct StorageClass |
| 10 | Node shows `NotReady` | `kubectl describe node` — check kubelet status, disk/memory pressure, network issues; SSH in, restart kubelet, check `journalctl -u kubelet` |

---

## 🔟 Top 10 AWS Cloud Scenario-Based Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | Traffic spikes 10x on a single EC2 app | Move to HA/scalable architecture: ELB + Auto Scaling Groups, static content on S3 + CloudFront, RDS Multi-AZ, optional ElastiCache |
| 2 | AWS bill doubled | Analyze with Cost Explorer, resize/stop underutilized EC2, use Reserved Instances/Savings Plans, enable Auto Scaling, move cold S3 data to Glacier, right-size RDS |
| 3 | Region failure — DR strategy? | Multi-region architecture, cross-region replication (S3/RDS), Route 53 failover routing; choose based on RTO/RPO: Backup & Restore, Pilot Light, Warm Standby, or Active-Active |
| 4 | Store sensitive data in S3 securely | Encryption at rest (SSE-S3/SSE-KMS) + in transit (HTTPS), strict least-privilege IAM, block public access, versioning + MFA delete, CloudTrail monitoring |
| 5 | Microservices on ECS — how do they communicate? | Service discovery (AWS Cloud Map), internal load balancer for sync calls, SQS (decoupling)/SNS (fan-out) for async, IAM roles + VPC networking for security |
| 6 | Logs scattered, app fails randomly | Centralize with CloudWatch Logs + Log Insights, set alarms (CPU/memory/error rate), enable X-Ray tracing, build dashboards |
| 7 | RDS slow due to heavy reads | Add Read Replicas, use ElastiCache, optimize queries/indexing, consider Aurora, use connection pooling |
| 8 | Automate deployment (CI/CD design) | CodeCommit (repo) → CodeBuild (build/test) → CodeDeploy (deploy) → CodePipeline (orchestration); add approval steps + rollback strategy |
| 9 | Migrate EC2 monolith to serverless | Break into microservices, use Lambda for compute, API Gateway for APIs, DynamoDB/S3 for data, event-driven architecture (SQS/SNS) |
| 10 | Different teams need different access levels | IAM roles/policies with least privilege, IAM groups for team-based access, MFA, AWS Organizations for multi-account governance |

---

## 🔟 Top 10 Azure DevOps Scenario-Based Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | Deployment failed in production | Stop further deployments → analyze logs (release pipeline, app monitoring, K8s/VM) → rollback (previous stable artifact or slot swap) → RCA → add pre-deployment validation, smoke tests, health probes, auto-rollback conditions |
| 2 | Build pipeline suddenly slow | Identify the slow stage via pipeline analytics → enable dependency caching, parallel jobs, incremental builds, reduce unnecessary tests, optimize Docker layers, use self-hosted agents for heavy workloads |
| 3 | Devs pushing directly to `main`, unstable releases | Branch policies in Azure Repos: mandatory PRs, min reviewers, successful build validation, work item linking, restrict direct pushes; enforce CI validation before merge; consider GitFlow/trunk-based branching |
| 4 | Design CI/CD for microservices | Reusable YAML templates; stages: checkout → static analysis → unit tests → Docker build → push to ACR → deploy to AKS via Helm → smoke tests → prod approval gate. Use env-specific variable groups, Key Vault secrets, rollback strategy, blue-green/canary |
| 5 | Secret accidentally exposed in a pipeline | Immediately revoke/rotate the secret, identify exposure scope, audit logs/access → move secrets to Key Vault, use secret variables (never hardcode), restrict pipeline permissions, enable log masking, review RBAC |
| 6 | K8s deployment works in staging, fails in prod | Compare config differences, secrets/ConfigMaps, resource limits, ingress rules, namespace policies; check pod logs, `kubectl describe`, Helm release history, probes — prefer IaC + consistent Helm charts across envs |
| 7 | Ensure zero-downtime deployments | Blue-green, rolling updates, canary deployments; readiness probes ensure traffic only reaches healthy pods; deployment slots (Azure App Service) for slot-swap; automated health checks + rollback |
| 8 | Pipeline failing intermittently | Usually flaky tests, infra instability, concurrency issues, network dependency problems — analyze failure patterns, isolate flaky tests, rerun with debug logs, check agent health, consider dedicated self-hosted agents, add retry logic |
| 9 | Manage IaC in Azure DevOps | Terraform / Bicep / ARM templates; pipeline: Validate → Plan → Approval → Apply; state files in Azure Storage with locking; separate environments via workspaces/state files; PR validation before infra changes |
| 10 | Secure an Azure DevOps pipeline | RBAC + least privilege, minimal-permission service connections, Key Vault integration, branch protection, signed artifacts, approval gates, dependency vulnerability scanning, disable public agent access, audit permissions regularly |

---

## 🔐 RBAC: Azure RBAC vs. AWS IAM vs. Kubernetes RBAC

| Layer | Controls |
|-------|-----------|
| **Azure RBAC** | Access to Azure cloud resources (subscriptions, resource groups, VMs, AKS **infrastructure**) |
| **AWS IAM** | Identities & permissions for AWS resources (EC2, S3, EKS) |
| **Kubernetes RBAC** | Permissions **inside** the cluster (namespaces, pods, deployments, secrets) |

> **Simple way to remember it:** Cloud RBAC secures the *infrastructure* layer. Kubernetes RBAC secures the *workload* layer inside the cluster. You need **both** — layered security, least privilege, namespace isolation, better governance.

### Common Governance Interview Q&A

| Question | Answer |
|----------|--------|
| **How do you ensure least-privilege access?** | RBAC across Azure/AWS/K8s; minimum permissions per role/namespace; avoid cluster-admin; use Just-In-Time (JIT) elevated access; use groups not direct grants; review access regularly; revoke immediately on offboarding |
| **How do you do user audit & logging?** | Centralized logging: Azure Activity Logs/Monitor/Sentinel, AWS CloudTrail/CloudWatch/GuardDuty, Kubernetes API server audit logs — all feeding into a central SIEM (Sentinel, Splunk, ELK) |
| **How do you handle access removal when a dev switches teams?** | Centralized IAM (Entra ID / AWS IAM Identity Center) with **group-based** RBAC — never assign directly to users. Move the user between groups → auto-updates Azure RBAC, AWS IAM roles, and K8s RoleBindings. Validate removal via audit logs and periodic reviews |
| **How do you ensure cluster/control-plane isolation?** | Separate Dev/QA/Prod into different clusters; use managed services (AKS/EKS) to secure the control plane; namespace isolation, RBAC, NetworkPolicies, node isolation, Pod Security Standards; restrict API server access via private networking/VPN; enable audit logging |

### Sample Kubernetes Role Manifest
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: payments-dev
  name: developer-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
```

---

## 🎩 OpenShift Basics

### What is OpenShift?
> **Red Hat OpenShift** is an enterprise container platform built **on top of Kubernetes**. It adds developer tools, built-in CI/CD, enhanced security, monitoring, an image registry, web console, and an Operator framework.

**Simple way to remember it:** *OpenShift = Kubernetes + enterprise features + automation.*

### Core Concepts

| Concept | Description |
|---------|--------------|
| **Container** | Packages app code, runtime, libraries, dependencies — runs identically everywhere (Docker is the common engine) |
| **Pod** | Smallest deployable unit — one or more containers running together (`oc get pods`) |
| **Project** | A logical namespace for organizing resources and controlling access (`oc new-project demo-project`) |
| **CRC (OpenShift Local)** | Lets developers run OpenShift locally on a laptop |

### Architecture
- **Control plane nodes:** API server, scheduler, etcd
- **Worker nodes:** run the actual workloads
- **Ingress router:** handles external traffic routing

### Essential `oc` CLI Commands
```bash
oc login
oc get pods
oc get svc
oc describe pod
oc logs
oc exec
```
> `oc` is the OpenShift CLI — the equivalent of `kubectl`, with additional OpenShift-specific commands.

---

## 🧠 Advanced DevOps Interview Questions (Architect/Principal Level)

### 1. Zero-Downtime Microservices Deployment on Kubernetes
- **Strategies:** Rolling Updates, Blue-Green, or Canary (via Argo Rollouts, Istio traffic routing, feature flags)
- **Key config:**
  - Accurate `readinessProbe`, `livenessProbe`, `startupProbe`
  - `maxUnavailable: 0`, `maxSurge: 1` (or `25%`) — old pods only terminate **after** new pods pass readiness checks
- **Canary analogy:** Shift traffic gradually — 5% → 20% → 50% → 100% — while watching error rates and latency before full cutover

### 2. Worker Node Enters `NotReady`
- **Control plane reaction:** `node-lifecycle-controller` detects missed heartbeats, marks node `NotReady`
- **Scheduling:** scheduler stops assigning new pods to it; existing pods remain until the **eviction timeout** (default 5 min), then get rescheduled elsewhere
- **Constraints to mention:**
  - **PodDisruptionBudgets (PDB):** guarantees minimum available instances during voluntary disruptions
  - **Stateless vs. StatefulSets:** stateless pods recreate easily; StatefulSets need CSI driver unmount/remount of PVs to the new node

### 3. Disaster Recovery: What Happens if etcd Crashes?
> **etcd** is the consistent key-value store holding the entire cluster state (nodes, pods, secrets, configs).

- **Impact:** `kube-apiserver` loses read/write capability and becomes unresponsive. **Running workloads keep running** (kubelet + container runtime continue executing already-running containers) — but control plane operations **freeze**: no new scheduling, scaling, secret retrieval, or deployment updates.
- **Mitigation:** run multi-node HA clusters (odd quorum: 3 or 5 nodes) across failure zones; restore from automated snapshots via `etcdctl snapshot restore`

### 4. Production Outage Troubleshooting Protocol
> **Structured triage, never panic-reboot random services.**

1. **Telemetry & Logs:** observe CPU, memory, IOPS, connection pools, error rate surges
2. **Layer Isolation:** determine if the fault is Network/DNS, Ingress/LB, Application logic, or downstream Databases
3. **Fast Mitigation:** if unresolved in 5–10 minutes, roll back to the last known-good deployment — save crash logs for post-mortem

### 5. Kubernetes Networking: Pod-to-Internet Path
```
Pod Network Namespace → veth pair → Linux Bridge/CNI Plugin (Calico, Cilium, Flannel)
   → Node routing tables → iptables/eBPF (SNAT)
   → Node's physical NIC (eth0) → VPC Router/NAT Gateway → Internet Gateway → External Web
```

### 6. Kubernetes Cluster Hardening (Defense-in-Depth)
| Layer | Practices |
|-------|-----------|
| **Admission Control** | Admission webhooks — OPA Gatekeeper, Kyverno |
| **Identity & Isolation** | Strict least-privilege RBAC, namespace boundaries, NetworkPolicies (block flat pod-to-pod access) |
| **Runtime & Supply Chain** | Sign images (Sigstore/Cosign), scan base images (Trivy), runtime threat detection (Falco), KMS encryption for etcd secrets |

### 7. Control Plane Lifecycle: End-to-End Flow of `kubectl apply -f deployment.yaml`
1. `kubectl` sends an HTTP request to `kube-apiserver`
2. API Server runs **Authentication → Authorization (RBAC) → Mutating/Validating Admission Controllers**
3. Desired state is persisted into **etcd**
4. **Deployment/ReplicaSet Controller** detects the state diff and creates Pod objects
5. **kube-scheduler** assigns pods to worker nodes based on resource filters and affinities
6. **kubelet** on the target node watches the API server, instructs the container runtime (containerd/CRI-O) to pull images and start containers, and reports pod status back

---

## 🎤 Full Interview Summary Script (Steps 1–4 Combined)

Use this as a cohesive narrative when asked to *"walk me through your project end-to-end"*:

1. **Infrastructure:** *"First, I provisioned the underlying AWS VPC, multi-AZ subnets, security groups, and EKS cluster using modular Terraform configurations stored in remote S3 with DynamoDB state locking."*
2. **SCM & Branching:** *"Our development teams follow a GitFlow model. Code commits trigger automated CI webhooks to our Jenkins server."*
3. **CI/CD Pipeline:** *"The Jenkins Declarative Pipeline checks out the code, compiles it with Maven, enforces SonarQube quality gates and OWASP dependency checks, builds an immutable Docker container tagged with the Git commit hash, and pushes it to private Amazon ECR."*
4. **Deployment:** *"Finally, the pipeline triggers an automated Helm upgrade (`helm upgrade --install`) on our EKS cluster, dynamically injecting the new image tag into `values.yaml` and performing a zero-downtime rolling update verified by `kubectl rollout status` checks."*
5. **Monitoring:** *"For observability, we run `kube-prometheus-stack` for metrics/dashboards, Alertmanager for proactive Slack alerts, and EFK/CloudWatch for centralized log aggregation and debugging."*

---

## 🧠 Quick Recall Cheat Sheet

- **Git Flow branches:** `main` → `develop` → `feature/*` → `release/*` → `hotfix/*`
- **Webhook URL must end with:** `/github-webhook/`
- **Continuous Delivery** = manual approval before prod | **Continuous Deployment** = fully automatic
- **`agent none`** at pipeline top level = no default executor reserved on master
- **`helm upgrade --install`** = creates release if new, upgrades if exists
- **Metrics** = numeric trends (Prometheus) | **Logs** = discrete text events (EFK)
- **Headless Service** = `clusterIP: None` → direct per-pod DNS, used with StatefulSets for databases
- **DNS pattern for pod-to-pod (cross-namespace):** `service-name.namespace.svc.cluster.local`
- **Traffic load-balancing:** CoreDNS resolves name → ClusterIP → kube-proxy round-robins to pod IPs
- **Code Smell** = messy but works | **Bug** = broken/crashes | **Vulnerability** = security hole
- **SonarQube Quality Gate** = blocks PR merge if Blocker/Critical bugs, vulnerabilities, or coverage < 80%
- **SQLi fix:** parameterized queries / PreparedStatements | **Secrets fix:** Vault / Secrets Manager, never hardcoded
- **Terraform modules:** child module outputs must be exported to be used — `module.<name>.<output>`
- **Terraform + Ansible:** Terraform builds infra → Ansible configures it → dynamic inventory bridges the two
- **Build Once, Promote Everywhere:** build image once, tag with commit SHA, promote the *same digest* through all environments — only config changes
- **ArgoCD = pull-based CD** (cluster pulls from Git); Jenkins/CI = push-based build/test/scan
- **ArgoCD rollback:** `git revert <hash>` (preferred) or `argocd app rollback <app-name>`
- **Ingress** = cost-effective L7 HTTP routing | **LoadBalancer** = 1-per-service, use for raw TCP/UDP
- **Automate recurring issues:** happens once → document it; happens twice → automate the fix
- **Cert-Manager:** Issuer (how to get certs) → Certificate (what domains) → stored in a Secret → auto-renewed
- **7th highest marks SQL:** top-7 DESC → subquery ASC → LIMIT 1
- **Ansible:** push-based & agentless (SSH/WinRM) vs. Puppet/Chef (pull-based, agent required)
- **DevOps vs GitOps:** DevOps = broad culture/full lifecycle | GitOps = Git-centric, declarative, pull-based CD specifically
- **6 Rs of migration:** Rehost, Replatform, Refactor, Repurchase, Retain, Retire
- **Rollback (GitOps):** query previous stable tag via registry API + `jq`, OR restore an archived manifest snapshot — commit back to Git, let Flux/ArgoCD reconcile
- **kubectl basics:** `apply -f` (declarative) vs `create`/`run` (imperative); `describe`, `logs`, `exec -it`, `--watch` for debugging
- **Blue-Green:** Blue = live, Green = idle/new. Cutover = switch Ingress backend or Service selector — same DNS/URL throughout
- **Helm:** `helm install` fails if release exists; `helm upgrade --install` is the safe pattern (creates OR updates)
- **`ARG` vs `ENV`:** ARG = build-time only; ENV = persists into the running container
- **Docker cache:** invalidates from the first changed layer downward — put stable deps (package.json) before volatile source code
- **`git fetch`** = download only (no merge) | **`git pull`** = fetch + merge
- **Ansible control node:** cannot run natively on Windows (needs WSL); but CAN manage Windows targets via WinRM
- **CI/CD scan types:** SAST (source code) → SCA (dependencies) → Container scan (image CVEs) → DAST (running app)
- **SLI/SLO/Error Budget:** SLI = what you measure, SLO = the target, Error Budget = allowed unreliability before you must prioritize reliability over features
- **etcd crash:** running pods keep running (kubelet/runtime continue); but NO new scheduling/scaling/secrets — control plane freezes
- **NotReady node:** pods evicted only after ~5 min timeout; PDBs guarantee minimum availability during the shuffle
- **Zero-downtime K8s deploy:** `maxUnavailable: 0`, `maxSurge: 1` + accurate readiness probes = old pods die only after new ones are ready
- **Cloud RBAC vs K8s RBAC:** Cloud RBAC = infrastructure layer access; K8s RBAC = workload layer access inside the cluster — use both
- **502 vs 503 vs 504:** 502 = bad response from upstream | 503 = service overloaded/down | 504 = upstream timed out
- **Terraform state recovery:** remote backend versioning (S3/Blob) OR `terraform import` to rebuild
- **OpenShift = Kubernetes + enterprise features** (CI/CD, security, web console, Operators) — `oc` is its CLI
