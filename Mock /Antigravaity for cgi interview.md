# 🛡️ DevSecOps Interview Handbook
> **By Syed Shabbir** | 6 Years Infrastructure + DevOps + DevSecOps  
> *Designed for long-term retention and interview preparation*

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| 1 | [Tell Me About Yourself](#1-tell-me-about-yourself) |
| 2 | [CI/CD Pipeline Workflow](#2-cicd-pipeline-workflow) |
| 3 | [Real Incident — OOMKilled / CrashLoopBackOff](#3-real-incident--oomkilled--crashloopbackoff) |
| 4 | [SAST vs DAST](#4-sast-vs-dast) |
| 5 | [Terraform State Drift](#5-terraform-state-drift) |
| 6 | [Kubernetes RBAC — Read-Only Service Account](#6-kubernetes-rbac--read-only-service-account) |
| 7 | [On-Call at 3AM — Production is Down](#7-on-call-at-3am--production-is-down) |
| 8 | [Disaster Recovery — RPO & RTO](#8-disaster-recovery--rpo--rto) |
| 9 | [Prometheus & Grafana — Monitoring & Alerting](#9-prometheus--grafana--monitoring--alerting) |
| 10 | [GitOps & ArgoCD](#10-gitops--argocd) |
| 11 | [Conflict with a Developer](#11-conflict-with-a-developer) |
| 12 | [Monolith to Microservices Migration](#12-monolith-to-microservices-migration) |
| 13 | [Kubernetes Security in Financial Services](#13-kubernetes-security-in-financial-services) |
| 14 | [Cross-Functional Remote Team Collaboration](#14-cross-functional-remote-team-collaboration) |
| 15 | [Career Goals — 3 to 5 Years](#15-career-goals--3-to-5-years) |
| 16 | [Secrets Management in Kubernetes](#16-secrets-management-in-kubernetes) |

---

## 1. Tell Me About Yourself

### 🧠 Simple Summary
> *"I went from systems admin → DevOps → DevSecOps. Now I want to make security part of the pipeline, not a bolt-on at the end."*

### 📌 Key Points to Remember

| Phase | Duration | What I Did |
|-------|----------|------------|
| **Systems Admin** | 5 years | VMware, Windows, Active Directory, GPO, PowerShell |
| **DevOps** | 1 year | Jenkins CI/CD, Terraform IaC, Ansible, EKS/Kubernetes |
| **Goal** | Now → Future | DevSecOps — security **inside** the pipeline, not after it |

### 💡 One-Line Memory Hook
> **"5 years admin → 1 year DevOps → going deep into DevSecOps"**

---

## 2. CI/CD Pipeline Workflow

### 🧠 Simple Summary
> *"Code push triggers Jenkins. Code is built, tested, scanned for quality, packaged, scanned for container vulnerabilities, pushed to registry, and deployed to Kubernetes via ArgoCD — with monitoring and notifications throughout."*

### 🔄 Pipeline Flow (Java Project)

```
Developer Push to GitHub
         │
         ▼ (Webhook triggers Jenkins)
  ┌──────────────────────────────────────────────────────────┐
  │  STAGE 1: BUILD                                          │
  │  Maven compiles code → catches syntax errors early       │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  STAGE 2: UNIT TESTS                                     │
  │  Verify functionality works as expected                  │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  STAGE 3: SONARQUBE (SAST)                               │
  │  Static code analysis → bugs, smells, vulnerabilities    │
  │  ❌ Quality Gate FAIL → Pipeline STOPS                   │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  STAGE 4: PACKAGE & ARTIFACT                             │
  │  Maven packages → pushed to JFrog Artifactory            │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  STAGE 5: DOCKER BUILD + TRIVY SCAN                      │
  │  Build Docker image → scan for CVEs                      │
  │  ❌ Critical vuln found → Pipeline STOPS                 │
  │  ✅ Clean → Push image to Docker Hub / ECR               │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  STAGE 6: GITOPS DEPLOYMENT                              │
  │  Update image tag in GitHub manifest repo                │
  │  ArgoCD detects change → auto-syncs to Kubernetes        │
  └──────────────────────────┬───────────────────────────────┘
                             │
  ┌──────────────────────────▼───────────────────────────────┐
  │  MONITORING: Prometheus + Grafana                        │
  │  NOTIFICATIONS: Email / Slack on success or failure      │
  └──────────────────────────────────────────────────────────┘
```

### 💡 Memory Hooks
- **SonarQube** = Code quality guard (before packaging)
- **Trivy** = Container vulnerability guard (after image build)
- **JFrog Artifactory** = Versioned artifact storage
- **ArgoCD** = GitOps auto-deploy to Kubernetes

---

## 3. Real Incident — OOMKilled / CrashLoopBackOff

### 🧠 Simple Summary
> *"We updated the app but forgot to update memory limits. Pods crashed in a loop. I fixed it immediately by raising limits, then prevented it permanently by adding a load test to the pipeline."*

### 🔍 What Happened

```
Problem:   App memory usage increased after update
           But Kubernetes resource limits were NOT updated

Result:    Pods hit memory limit → OS killed them (OOMKilled)
           Kubernetes restarted them → they crashed again → loop

Symptom:   CrashLoopBackOff
```

### 🛠️ How I Fixed It

| Step | Action | Tool |
|------|--------|------|
| **Detect** | Got alert, saw CrashLoopBackOff | Monitoring alert |
| **Diagnose** | Confirmed OOMKilled error | `kubectl describe pod` |
| **Immediate fix** | Increased memory limits → pods stabilized | kubectl / manifest update |
| **Long-term fix** | Added load/performance test in CI pipeline | Jenkins pipeline stage |

### 💡 Key Commands to Remember
```bash
kubectl describe pod <pod-name>     # See why a pod crashed
kubectl get pods                    # See pod status
kubectl top pods                    # See memory/CPU usage
```

---

## 4. SAST vs DAST

### 🧠 Simple Summary
> *"SAST = scan the code (early). DAST = attack the running app (late). Both catch different bugs — use both."*

### 📊 Comparison Table

| Feature | SAST | DAST |
|---------|------|------|
| **Full Name** | Static Application Security Testing | Dynamic Application Security Testing |
| **What it scans** | Source code (not running) | Running application |
| **When in pipeline** | Early — after build/unit tests | Late — after deployment to staging |
| **What it catches** | Insecure code patterns, hardcoded secrets, bad dependencies | Injection attacks, auth bypasses, runtime misconfigs |
| **Tool example** | SonarQube | OWASP ZAP, Burp Suite |
| **Analogy** | Reading a recipe for mistakes | Actually cooking and tasting for problems |

### 📍 Where They Fit in MY Pipeline

```
Source Code → [SAST: SonarQube] → Build → Docker Image → [Trivy: Container Scan] → Deploy → [DAST: against staging]
```

> **Note:** Trivy is neither SAST nor DAST — it's **Software Composition Analysis (SCA)** — scans the Docker image and dependencies for known CVEs.

### 💡 Memory Hook
> **"SAST = Static = Source. DAST = Dynamic = Deployed."**

---

## 5. Terraform State Drift

### 🧠 Simple Summary
> *"Drift = reality doesn't match Terraform's state file. Usually caused by someone making manual changes in the AWS console. Detect with `terraform plan`, fix by either reapplying Terraform or updating your code to match the manual change."*

### 🔍 What is State Drift?

```
Terraform State File says:  Security Group = port 443 open
Someone manually changed:   Security Group = port 80 AND 443 open

→ Drift! Terraform's record is now out of sync with real infrastructure
```

### 🛠️ How to Handle Drift

```bash
# Step 1: Detect drift
terraform plan
# → Terraform shows "unexpected" changes it wants to make
# → These are the drifted resources

# Step 2A: Drift was a MISTAKE → force back to code
terraform apply
# → Infrastructure goes back to what code defines

# Step 2B: Drift was INTENTIONAL (e.g. emergency fix) → update the code
# → Edit your .tf file to match what was manually changed
# → terraform plan should now show no changes
```

### 🚫 How to PREVENT Drift

> **Restrict console/manual access to production.**
> All changes MUST go through Terraform. No exceptions.
> Use IAM policies to block direct console modifications in prod.

### 💡 Memory Hook
> **"Plan detects, Apply fixes, Policy prevents."**

---

## 6. Kubernetes RBAC — Read-Only Service Account

### 🧠 Simple Summary
> *"Create a ServiceAccount → Create a Role (not ClusterRole) with only read verbs → Bind them with a RoleBinding. That's it."*

### 🔑 The 3-Step Recipe

```yaml
# STEP 1: ServiceAccount (scoped to namespace)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-readonly-sa
  namespace: staging

---

# STEP 2: Role (only get/list/watch — NO create/update/delete)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: readonly-role
  namespace: staging
rules:
  - apiGroups: [""]
    resources: ["pods", "deployments", "services"]
    verbs: ["get", "list", "watch"]   # Read-only verbs only!

---

# STEP 3: RoleBinding (connects Role → ServiceAccount)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: readonly-binding
  namespace: staging
subjects:
  - kind: ServiceAccount
    name: monitoring-readonly-sa
    namespace: staging
roleRef:
  kind: Role
  name: readonly-role
  apiGroup: rbac.authorization.k8s.io
```

### 📌 Key Differences to Remember

| | **Role** | **ClusterRole** |
|--|----------|-----------------|
| **Scope** | One namespace only ✅ | Entire cluster |
| **Use when** | Limiting access to single namespace | Cross-namespace or cluster-wide access |

> **RoleBinding** = The glue. The Role alone does nothing — it must be **bound** to an identity.

### 💡 Memory Hook
> **"SA + Role + RoleBinding = Locked-down access. Role without Binding = Nothing."**

---

## 7. On-Call at 3AM — Production is Down

### 🧠 Simple Summary
> *"Don't panic. Assess → Check recent changes → Communicate → Rollback or fix → Verify → Write RCA."*

### 📋 Step-by-Step Playbook

```
⏰ Page received at 3AM
│
├── STEP 1: ACKNOWLEDGE & ASSESS (Don't touch anything yet!)
│   └── Check Grafana/CloudWatch → How many services affected? All users or some?
│
├── STEP 2: CHECK WHAT CHANGED RECENTLY
│   └── Jenkins/ArgoCD pipeline history → Any recent deployments?
│   └── Terraform changes? Config updates? Cloud provider status?
│
├── STEP 3: COMMUNICATE EARLY (Even without root cause)
│   └── Post in Slack: "Investigating [service] errors, update in 10 min"
│   └── Silence during incidents = worse than incomplete updates
│
├── STEP 4: TRIAGE — ROLLBACK vs FIX FORWARD
│   ├── If deployment-related → ROLLBACK FIRST (restore service, investigate later)
│   │   └── kubectl rollout undo  OR  ArgoCD rollback to previous version
│   └── If resource issue → kubectl describe pod, check CPU/memory metrics
│
├── STEP 5: VERIFY RECOVERY
│   └── Watch error rate + health checks for a few minutes — don't assume fixed
│
└── STEP 6: COMMUNICATE RESOLUTION + RCA
    └── Post resolution update → Next day write proper RCA
    └── RCA = Timeline + Root cause + Impact + Action items (to prevent recurrence)
```

### 💡 Memory Hook
> **"Assess → Communicate → Rollback → Verify → RCA"**
> Restoring service is priority #1. Root cause investigation is priority #2.

---

## 8. Disaster Recovery — RPO & RTO

### 🧠 Simple Summary
> *"RPO = how much data you can lose (backup frequency). RTO = how long you can be down (recovery speed). For 4hr RPO + 2hr RTO: backup every 4hrs + warm standby in another region."*

### 📖 Definitions (Never Forget)

| Term | Full Name | Simple Meaning | Given Value |
|------|-----------|----------------|-------------|
| **RPO** | Recovery Point Objective | How much **data loss** is acceptable | 4 hours → backup every ≤4 hours |
| **RTO** | Recovery Time Objective | How long you can be **offline** | 2 hours → must restore within 2 hours |

### 🏗️ Strategy for RPO=4hr / RTO=2hr

```
┌─────────────────────────────────────────────────────┐
│  MEETING RPO (4 hours) — Backup Frequency           │
│                                                     │
│  • RDS automated backups + point-in-time recovery   │
│  • Continuous replication to standby region         │
│  • Snapshots every 2-3 hours (margin of safety)     │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  MEETING RTO (2 hours) — Recovery Speed             │
│                                                     │
│  • CANNOT do cold rebuild (takes too long)          │
│  • USE: Warm Standby in secondary AWS region        │
│    - Infrastructure already running at low capacity │
│    - Defined in Terraform (identical to primary)    │
│    - Scale up fast when needed                      │
│  • Database continuously replicating to standby     │
│  • Route 53 failover routing → redirects traffic    │
└─────────────────────────────────────────────────────┘
```

### 📊 DR Strategy Types (Bonus Knowledge)

| Strategy | RTO | Cost | How it works |
|----------|-----|------|--------------|
| **Backup & Restore** | Hours-Days | 💲 Cheapest | Restore from backups |
| **Pilot Light** | ~Hours | 💲💲 | Minimal core running, scale up on disaster |
| **Warm Standby** | ~Minutes-2hrs | 💲💲💲 | Scaled-down replica always running |
| **Multi-Site Active-Active** | ~Seconds | 💲💲💲💲 | Full duplicate, always live |

### 💡 Memory Hook
> **"RPO = Point (data). RTO = Time (downtime). Warm standby + Route53 = meet both."**

---

## 9. Prometheus & Grafana — Monitoring & Alerting

### 🧠 Simple Summary
> *"Prometheus pulls metrics from apps every 15-30 seconds via /metrics endpoint. Alert only on things that need a human RIGHT NOW. Everything else = just a dashboard. Avoid alert fatigue."*

### ⚙️ How Prometheus Works

```
Prometheus (pull model)
│
├── Scrapes targets every 15-30 seconds
├── Each target exposes a /metrics endpoint
│   ├── App native support → built-in /metrics
│   ├── Node Exporter → system metrics (CPU, memory, disk)
│   └── JMX Exporter → Java app metrics
│
├── Stores data as time series (value + timestamp + labels)
│   └── Labels: environment, service_name, pod_name, namespace
│
└── Grafana queries Prometheus → visualizes as dashboards
```

> **Pull model** = Prometheus goes OUT to collect data. Apps don't push to Prometheus.

### 🚨 Alert vs Dashboard Decision Framework

| **ALERT ON (needs human NOW)** | **DASHBOARD ONLY (check during work hours)** |
|-------------------------------|----------------------------------------------|
| ❌ Service completely down | 📊 CPU elevated but not critical |
| ❌ Error rate > 5% for 2+ min | 📊 Memory usage trends over time |
| ❌ Response latency > p99 2s | 📊 Request volume / traffic patterns |
| ❌ Pod in CrashLoopBackOff | 📊 Build times and pipeline durations |
| ❌ Disk > 85% on prod node | 📊 Non-critical warning thresholds |
| ❌ Certificate expiring in 30 days | |

### ⚠️ Alert Fatigue — Real Story
> **Problem:** Alert firing on CPU > 70% — fired constantly. Team ignored it.
> **Fix:** Raised threshold to CPU > 85% **for 5+ minutes sustained** + tied to error rate spike.
> **Result:** Alert volume dropped. Team started taking alerts seriously again.

### 💡 Memory Hook
> **"Alert = needs human at 3AM. Dashboard = check at 9AM."**
> **"5 meaningful alerts > 50 ignored alerts."**

---

## 10. GitOps & ArgoCD

### 🧠 Simple Summary
> *"GitOps = Git is the source of truth. ArgoCD watches your Git repo and keeps the cluster in sync with it. If someone changes the manifest without review, ArgoCD catches it (especially with manual sync on prod)."*

### 🔁 How ArgoCD Works

```
Developer pushes image tag update to Git manifest repo
                    │
                    ▼
         ArgoCD watches repo continuously
                    │
         Detects: Git ≠ Cluster (out of sync)
                    │
         ┌──────────┴──────────┐
         │ Auto-sync enabled   │ Manual sync (production)
         │ → Deploys instantly │ → Flags for human review
         └─────────────────────┘
```

### 🎯 Dashboard States to Know

| Color | Meaning |
|-------|---------|
| 🟢 Green | In sync + healthy |
| 🟡 Yellow | Out of sync (change detected, not applied) |
| 🔴 Red | Degraded / unhealthy |

### 🛡️ Real Scenario — How ArgoCD Prevented a Problem
> **What happened:** Developer accidentally pushed change reducing replicas from 3 → 1 in production manifest (without proper review).
> **Why it was caught:** Production was set to **manual sync** — ArgoCD flagged it as out-of-sync instead of auto-applying.
> **Action taken:** Reviewed the diff in ArgoCD dashboard, saw the replica count change, rejected the sync.
> **Lesson:** Auto-sync on staging ✅ | Manual sync on production ✅

### 🏆 Biggest Benefit of GitOps
> **Audit trail** — every production change is a Git commit with timestamp, author, and diff.
> At 3AM during an incident: "What changed?" → just check the Git log.

### 💡 Memory Hook
> **"Git is truth. ArgoCD is the enforcer. Manual sync on prod = human safety net."**

---

## 11. Conflict with a Developer

### 🧠 Simple Summary
> *"Developer wanted to skip SonarQube gate due to deadline. Instead of saying no, I sat with them, fixed the 2 real vulnerabilities together, and updated the quality gate to reduce noise. Everyone won."*

### 📖 Story Structure (STAR Format)

| Part | Detail |
|------|--------|
| **Situation** | Developer wanted to bypass SonarQube gate — deadline pressure |
| **Task** | Maintain security standards without blocking the release |
| **Action** | Reviewed flagged issues together → fixed 2 real vulns → raised ticket for noisy low-severity rules → updated gate config |
| **Result** | Release shipped same day, gate improved, developer thanked me |

### 🧠 Key Insight to Mention
> **"Security gates only work if developers trust them."**
> If they feel like arbitrary blockers, people route around them.
> Making the gate smarter AND working collaboratively = better long-term outcome.

### 💡 Memory Hook
> **"Not 'no' — but 'let's fix the real ones together and improve the gate.'"**

---

## 12. Monolith to Microservices Migration

### 🧠 Simple Summary
> *"Don't migrate all at once. Use Strangler Fig pattern — extract one service at a time. Each service gets its own pipeline. Use canary deployments for new services. Observability FIRST, not later. Database split is the hardest part."*

### 🗺️ Migration Phases

```
PHASE 1: UNDERSTAND THE MONOLITH
├── Map service boundaries (what talks to what?)
├── Identify least-coupled components to extract first
└── Document current deployment process + rollback story

PHASE 2: CI/CD PIPELINE (per microservice)
├── Each service = its own independent Jenkins pipeline
├── Pipeline: Build → Test → SonarQube → Trivy → Docker → ECR → GitOps → ArgoCD
└── Key: Services deploy INDEPENDENTLY (change to service A ≠ redeploy service B)

PHASE 3: STRANGLER FIG PATTERN
├── Extract services one at a time — don't rewrite everything
├── Route some traffic to new service, rest stays on monolith
└── Monolith shrinks over time until it's gone

PHASE 4: DEPLOYMENT STRATEGIES
├── New services → Canary deployment (10% → 25% → 50% → 100%)
│   └── Monitor error rates + latency at each step
└── Established services → Rolling deployments

PHASE 5: OBSERVABILITY (from day 1, not later!)
├── Prometheus metrics endpoint required before prod
├── Structured logging required
└── Distributed tracing (Jaeger/AWS X-Ray) required
    └── With 20 microservices, you NEED tracing to follow requests
```

### ⚠️ Hardest Part — The Database
> Monolith = one big shared database
> Microservices = each service owns its own data
> **Solution:** Keep shared database temporarily while services mature, then gradually separate data ownership.

### 🏷️ Image Tagging Rule
> **NEVER use `latest` tag in production.**
> Always tag with **Git commit SHA** → rollback = point ArgoCD at previous SHA.

### 💡 Memory Hook
> **"Strangler Fig = extract one by one. Canary = 10 → 100% gradually. Observability first. Database last."**

---

## 13. Kubernetes Security in Financial Services

### 🧠 Simple Summary
> *"5-layer security: Harden the cluster → RBAC least privilege → Network policies + mTLS → Image scanning + admission control → Runtime detection + compliance reporting."*

### 🏰 The 5 Security Layers

#### 🔒 Layer 1 — Cluster Hardening
- API server **NOT publicly accessible** — only reachable from within VPC or through VPN
- Enable **audit logging** on API server (full audit trail for compliance)
- Worker nodes on **latest patched AMIs** (use managed node groups)
- **Disable SSH** to nodes → use AWS Systems Manager Session Manager instead (auditable, no open ports)

#### 👤 Layer 2 — RBAC & Least Privilege
- Every service account, user, CI/CD tool = **minimum permissions only**
- No wildcard permissions, no cluster-admin for service accounts
- Run **kube-bench** or **Polaris** to flag overly permissive configs automatically
- Human access: **EKS + AWS IAM + MFA enforced** (no long-lived kubeconfigs on laptops)

#### 🌐 Layer 3 — Network Security
- **Default deny** all pod-to-pod communication via `NetworkPolicy`
- Only explicitly allowed pods can communicate
- **Istio service mesh** with **mTLS** → encrypt ALL service-to-service traffic inside the cluster
- In financial services, internal traffic encryption is as important as external

#### 📦 Layer 4 — Image & Supply Chain Security
- **Trivy** in CI pipeline = hard gate (critical CVE = pipeline fails)
- **OPA Gatekeeper / Kyverno** admission controller = cluster rejects unscanned images even from kubectl
- **Image signing** = only signed images from internal ECR allowed in prod
- **Distroless or Alpine** base images = minimal attack surface

#### 🔍 Layer 5 — Runtime Security & Compliance
- **Falco** = runtime anomaly detection (shell spawned in container? Falco alerts immediately)
- **kube-bench** = checks cluster against CIS Kubernetes Benchmark controls
- **Compliance SLAs:** Critical findings → fix in 24hrs | High → fix in 7 days
- Generate compliance reports for auditors

### 🎯 One-Line Summary of the 5 Layers

```
PREVENT bad things getting in  → Image scanning + admission control
LIMIT blast radius             → RBAC + Network policies
ENCRYPT everything in transit  → mTLS (Istio)
DETECT anomalies at runtime    → Falco
PROVE it all to auditors       → Audit logs + compliance reports
```

### 💡 Memory Hook
> **"Harden → RBAC → Network → Images → Runtime."**
> In financial services, the AUDIT TRAIL is as important as the control itself.

---

## 14. Cross-Functional Remote Team Collaboration

### 🧠 Simple Summary
> *"For remote/cross-timezone teams: single source of truth (Confluence) + async daily standup + explicit handoffs with named owners + shared deployment calendar + short weekly sync."*

### 📖 Story Structure (STAR Format)

| Part | Detail |
|------|--------|
| **Situation** | On-prem to AWS migration — dev, security, and ops teams in different timezones |
| **Task** | Keep project moving with competing priorities (dev=speed, security=controls, ops=stability) |
| **Action** | Confluence single source of truth, Slack channels per workstream, async standups, explicit handoffs, Jira tracking, shared deployment calendar |
| **Result** | Migration completed on schedule, security team signed off without last-minute blockers |

### 🛠️ Communication Tools & Rules

| Tool / Practice | Purpose |
|-----------------|---------|
| **Confluence** | Single source of truth for all decisions + diagrams |
| **Slack channels per workstream** | Organized, searchable conversations |
| **No important decisions in DMs** | Everyone has context, nothing lost |
| **Async daily standup** | 3 lines at end of day: Did / Blocked / Need from someone |
| **Jira tickets** | Every action item = owner + due date before meeting ends |
| **Weekly 30-min sync** | Review open items, unblock, reprioritize |
| **Shared deployment calendar** | All teams see when deployments are planned |

### 💡 Key Insight
> **"In remote cross-functional work, the biggest risk isn't technical — it's information gaps."**
> People making decisions without full context, or assuming someone else is handling something.

### 💡 Memory Hook
> **"Confluence = brain. Slack channels = conversations. Async standup = daily heartbeat. Explicit handoff = named owner."**

---

## 15. Career Goals — 3 to 5 Years

### 🧠 Simple Summary
> *"Go deep in DevSecOps — design secure-by-default systems, lead security conversations, mentor others. CGI's financial services client (strict compliance) is the right environment to develop this rigor fast."*

### 🎯 The Arc

```
NOW:                          3 YEARS:                    5 YEARS:
Hands-on DevSecOps       →   Subject matter expert    →  Technical Lead /
(Jenkins, Terraform,          in secure pipelines          Solutions Architect
 K8s, ArgoCD, Trivy,          + compliance                leading DevSecOps
 SonarQube)                   environments                 practices for a
                                                           program/team
```

### 🏦 Why CGI + Financial Services?
- **PCI DSS, SOC 2, strict audit requirements** = most demanding security environment
- **Real consequences** for getting security wrong = forces development of rigor
- **CGI's global teams** = develop consulting + communication skills (not just technical)
- This is a **depth move**, not a lateral move

### 💡 Memory Hook
> **"I want to make security invisible to developers — baked in, not bolted on. CGI's FS client is where that rigor gets built."**

---

## 16. Secrets Management in Kubernetes

### 🧠 Simple Summary
> *"Secrets never touch Git. Store in AWS Secrets Manager or HashiCorp Vault. Inject as mounted files (NOT env vars). Scrub logs. Rotate automatically. Prove it all with audit logs."*

### 🔐 4-Layer Approach

#### 🚫 Layer 1 — Prevent Secrets Entering Git
```
Developer laptop:  git-secrets / detect-secrets (pre-commit hook)
                   → Blocks commit if secret pattern detected

CI Pipeline:       Gitleaks / TruffleHog
                   → Scans every commit, fails build if secret found

Retroactive:       Run Gitleaks against entire Git history on new repos
                   → Secrets deleted from files are STILL in Git history!
```

#### 🏦 Layer 2 — Centralized Secrets Storage

| Tool | Use Case | Key Feature |
|------|----------|-------------|
| **AWS Secrets Manager** | EKS/AWS environments | KMS encryption + CloudTrail audit log + auto-rotation |
| **HashiCorp Vault** | Multi-cloud / cloud-agnostic | Dynamic secrets (expire automatically) + fine-grained access |

> **Key Principle:** Applications get **short-lived credentials** that expire automatically — never long-lived static credentials.

#### 💉 Layer 3 — Inject Secrets Safely into Pods

```
❌ WRONG:  Environment variables
           → Appear in kubectl describe pod output
           → Captured in crash dumps
           → Accidentally logged by developers

✅ RIGHT:  Mounted files via CSI Driver or Vault Agent Injector
           → Secret written to in-memory volume only the pod can read
           → Never touches Kubernetes API server or etcd
```

**Tools:**
- **AWS Secrets Manager:** AWS Secrets and Config Provider (ASCP) + Kubernetes Secrets Store CSI Driver
- **HashiCorp Vault:** Vault Agent Injector (sidecar) → authenticates via K8s service account → mounts secret as file

#### 📋 Layer 4 — Prevent Secrets in Logs

| Control | How |
|---------|-----|
| **Application-level** | Configure logging framework to mask/redact sensitive fields |
| **Log pipeline scrubbing** | ELK/Splunk filter stage — redact secret patterns before indexing |
| **K8s audit logging** | Monitor `get`/`list` on Secret objects — unusual = potential exfil |
| **Continuous scanning** | AWS Macie / Nightfall — scans data stores + logs for sensitive patterns |

### 🚨 Real Incident Story
> **What happened:** Developer accidentally committed AWS access key to feature branch during late-night debugging.
> **Caught by:** Gitleaks in CI pipeline — within seconds of the push.
> **Response:** Immediately rotated the key → checked CloudTrail → confirmed it wasn't used.
> **Follow-up:** Added pre-commit hooks on all developer machines + pushed for IAM roles instead of access keys.

### 💡 Memory Hook
> **"Pre-commit hooks → Vault/Secrets Manager → Mounted files NOT env vars → Log scrubbing → Auto-rotate."**

---

## 🧠 Master Cheat Sheet — Quick Recall

| Topic | 5-Word Summary |
|-------|----------------|
| **Tell Me About Yourself** | Admin → DevOps → DevSecOps |
| **CI/CD Pipeline** | Push → Build → Scan → Deploy |
| **OOMKilled Incident** | Forgot memory limits, pods crashed |
| **SAST vs DAST** | Static=early, Dynamic=deployed |
| **Terraform Drift** | Plan detects, Apply fixes, Policy prevents |
| **K8s RBAC** | SA + Role + RoleBinding = access |
| **3AM Incident** | Assess → Communicate → Rollback → RCA |
| **RPO / RTO** | RPO=data loss, RTO=downtime duration |
| **Prometheus** | Pull model, /metrics, alert fatigue |
| **GitOps / ArgoCD** | Git is truth, ArgoCD enforces it |
| **Conflict Resolution** | Fix real issues, improve the gate |
| **Microservices Migration** | Strangler Fig, one service at a time |
| **K8s Security** | Harden→RBAC→Network→Images→Runtime |
| **Remote Collaboration** | Single source of truth + async standup |
| **Career Goals** | Deep DevSecOps, security baked in |
| **Secrets Management** | Never Git, never env vars, always rotate |

---

## 🔧 Key Tools Reference

| Tool | Category | What It Does |
|------|----------|--------------|
| **Jenkins** | CI/CD | Pipeline orchestration |
| **Maven** | Build | Java build + test + package |
| **SonarQube** | SAST | Code quality + security analysis |
| **JFrog Artifactory** | Artifact Registry | Versioned artifact storage |
| **Trivy** | Container Security | CVE scanning of Docker images |
| **Docker / ECR** | Containers | Build + store container images |
| **ArgoCD** | GitOps / CD | Syncs Git manifests to Kubernetes |
| **Terraform** | IaC | Infrastructure as Code (AWS) |
| **Ansible** | Config Mgmt | Configuration management |
| **Kubernetes / EKS** | Orchestration | Container workload management |
| **Prometheus** | Monitoring | Metrics collection (pull model) |
| **Grafana** | Visualization | Dashboards + alerting |
| **Falco** | Runtime Security | Anomaly detection in containers |
| **Vault / AWS Secrets Manager** | Secrets | Centralized secrets management |
| **OPA Gatekeeper / Kyverno** | Admission Control | Enforce policies in cluster |
| **Istio** | Service Mesh | mTLS + traffic management |
| **Gitleaks / TruffleHog** | Secret Scanning | Find secrets in Git repos |
| **kube-bench** | Compliance | CIS Benchmark checks for K8s |
| **Route 53** | DNS / Failover | Traffic routing + DR failover |

---

*Last updated: August 2026 | Role Target: DevSecOps Engineer | Client: Financial Services / CGI*
