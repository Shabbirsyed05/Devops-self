# DevOps / DevSecOps Interview Prep — Answer Guide
 
Use this as a study reference, not a script to memorize word-for-word. For every question, aim to answer in 3 parts: **concept → how it works → your experience (or how you'd approach it if you haven't done it directly)**. Interviewers trust honesty about gaps far more than a rehearsed answer that falls apart under a follow-up.
 
---
 
## 1. AWS & Cloud Infrastructure
 
**How would you design a highly available, multi-AZ architecture on AWS for a production service?**
Spread compute across at least 2–3 Availability Zones behind an Application Load Balancer, use an Auto Scaling Group so instances self-heal, put the database in Multi-AZ mode (e.g., RDS with a standby replica), and use S3/EBS with cross-AZ redundancy built in. Route 53 health checks add failover at the DNS layer if you're multi-region. The key idea to say out loud: no single AZ failure should take down the service, and nothing should be a single point of failure.
 
**Walk me through how IAM roles, policies, and SCPs work together to enforce least-privilege access.**
A **policy** is a JSON document defining permissions (allow/deny on specific actions/resources). A **role** is an identity that policies attach to, assumed temporarily by users, services, or applications (no long-lived credentials). **SCPs (Service Control Policies)** sit at the AWS Organizations level and set the *maximum* permissions boundary across accounts — they don't grant anything themselves, they only restrict. So even if an IAM policy allows an action, an SCP can override and block it account-wide. Together: SCP sets the ceiling, IAM policies grant within that ceiling, roles are how identities pick up those permissions temporarily.
 
**What's the difference between an IAM role and an IAM user, and when would you use each?**
A user is a persistent identity with long-term credentials (password/access keys), meant for a person. A role has no credentials of its own — it's assumed temporarily and issues short-lived tokens. Use roles for applications, EC2 instances, Lambda functions, and cross-account access; use users sparingly, mainly for people who need console/CLI access, and even then prefer federated SSO over IAM users where possible.
 
**How do you approach cloud cost optimization (FinOps)? What AWS tools have you used for this?**
Start with visibility: AWS Cost Explorer and Cost & Usage Reports to see where spend concentrates. Then act on the big levers — right-sizing instances (Compute Optimizer), moving predictable workloads to Reserved Instances or Savings Plans, using Spot for fault-tolerant batch work, cleaning up unattached EBS volumes/old snapshots, and setting budget alerts. If you haven't owned FinOps directly: be honest, but describe the framework above and mention any cost-awareness from your infra consolidation work (tie back to your VMware/patch consolidation numbers if relevant).
 
**Explain VPC peering vs. Transit Gateway — when would you use each?**
VPC Peering is a direct 1:1 connection between two VPCs — simple, no extra cost beyond data transfer, but it doesn't scale well (no transitive routing, so peering 10 VPCs together means many point-to-point connections). Transit Gateway is a hub-and-spoke model — one gateway that many VPCs (and on-prem via VPN/Direct Connect) attach to, with centralized route tables. Use peering for a couple of VPCs; use Transit Gateway once you're managing many VPCs or need transitive routing.
 
**How would you design a disaster recovery strategy with RPO 4hrs / RTO 2hrs on AWS?**
RPO of 4 hours means you can tolerate losing up to 4 hours of data — so snapshot/backup frequency needs to be at least every 4 hours (RDS automated backups + point-in-time recovery, or EBS snapshot schedules via AWS Backup). RTO of 2 hours means the service must be back up within 2 hours of a failure — that points toward a **warm standby** approach: infrastructure as code (Terraform) to redeploy fast, a scaled-down but running replica in a second region, and Route 53 failover routing to redirect traffic. Full active-active would beat RTO easily but costs more; pilot light might not hit 2 hours reliably. This maps directly to Veeam/DR experience — same concepts (backup frequency = RPO, recovery speed = RTO), just translated to cloud-native tooling.
 
---
 
## 2. Terraform / IaC
 
**How do you manage Terraform state in a team environment? What is a remote backend and why does it matter?**
Never use local state in a team — it causes conflicts and risks loss. Use a remote backend like S3 (with DynamoDB for state locking) or Terraform Cloud. The backend stores state centrally so everyone works off the same source of truth, and locking prevents two people from running `apply` simultaneously and corrupting state.
 
**What's the difference between `terraform plan`, `apply`, and `refresh`?**
`plan` shows what changes Terraform *would* make without applying them — a dry run. `apply` executes those changes against real infrastructure. `refresh` (now largely folded into plan/apply automatically) reconciles Terraform's state file with the actual current state of infrastructure, in case something changed outside of Terraform.
 
**How do you handle secrets in Terraform without hardcoding them?**
Never put secrets in `.tf` files or commit them to state in plaintext. Use a secrets manager (AWS Secrets Manager, HashiCorp Vault) and reference secrets via data sources at apply time, or pass sensitive variables through environment variables / a `.tfvars` file that's gitignored and encrypted at rest. Mark variables `sensitive = true` so they're redacted from CLI output/logs.
 
**Have you dealt with Terraform state drift? How do you detect and fix it?**
Drift happens when someone changes infrastructure manually (console click) outside of Terraform, so the state file no longer matches reality. Detect it by running `terraform plan` regularly — it'll show unexpected diffs. Fix it either by updating the Terraform config to match reality (if the manual change was intentional and should stay) or running `apply` to force infrastructure back to match code (if the manual change was a mistake). Best prevention: lock down console access so changes only happen through IaC.
 
**Explain modules in Terraform — how do you structure reusable IaC across environments (dev/stage/prod)?**
A module is a reusable, self-contained group of Terraform resources (e.g., a "VPC module" or "EKS cluster module"). Structure environments by keeping one set of modules and separate `.tfvars` files (or separate directories) per environment that call the same modules with different variable values — this avoids duplicating code across dev/stage/prod while keeping each environment's actual state isolated.
 
---
 
## 3. Kubernetes / EKS
 
**Difference between a Deployment, ReplicaSet, and Pod.**
A **Pod** is the smallest deployable unit — one or more containers sharing network/storage. A **ReplicaSet** ensures a specified number of identical Pod replicas are running at all times. A **Deployment** manages ReplicaSets on top of that, adding rolling updates, rollback history, and declarative version control — in practice you almost always work with Deployments, not ReplicaSets directly.
 
**Walk me through what happens when a Pod goes into CrashLoopBackOff — how do you debug it?**
It means the container is starting, crashing, and Kubernetes keeps restarting it with exponential backoff. Debug steps: `kubectl describe pod <name>` to see events (OOMKilled, failed probes, image pull errors), `kubectl logs <pod> --previous` to see the crashed container's last logs, check resource limits (is it getting OOM killed?), and check the container's entrypoint/command and any missing config/secrets it depends on at startup.
 
**Blue-green vs. canary deployment — which have you implemented?**
Blue-green runs two full environments (old "blue," new "green"), and you switch all traffic over at once — fast rollback (just switch back) but needs double the resources briefly. Canary rolls the new version out to a small % of traffic first, monitors, then gradually increases — safer for catching issues early but slower and needs traffic-splitting infrastructure (service mesh or ingress support). Speak to whichever you've actually done; if neither directly, describe the concepts and which you'd choose for a given risk tolerance.
 
**How do Helm charts work, and why use them over raw YAML manifests?**
Helm packages a set of Kubernetes manifests as a templated, versioned "chart" with configurable values (`values.yaml`). Instead of maintaining separate raw YAML per environment, you parameterize one chart and override values per environment. It also gives you release history and easy rollback (`helm rollback`), which raw `kubectl apply` doesn't track natively.
 
**Explain Kubernetes RBAC — how would you restrict a service account to read-only access on one namespace?**
RBAC controls who/what can do what, on which resources, in which namespace, via **Roles** (namespace-scoped) or **ClusterRoles** (cluster-wide), bound to identities via **RoleBindings**/**ClusterRoleBindings**. To restrict a service account to read-only on one namespace: create a Role in that namespace with verbs limited to `get`, `list`, `watch`, then a RoleBinding attaching that Role to the specific ServiceAccount — scoped to that namespace only, so it has zero visibility or permissions outside it.
 
**How does EKS differ from self-managed Kubernetes in terms of operational responsibility?**
EKS manages the control plane for you (API server, etcd, scheduler, upgrades, HA) — AWS's responsibility. You're still responsible for worker nodes (or use Fargate to offload that too), networking (VPC/CNI), IAM integration, and workload-level security. Self-managed Kubernetes (kubeadm, on-prem) means you own the control plane too — upgrades, etcd backups, HA setup, everything. EKS trades some control for significantly less operational overhead.
 
---
 
## 4. CI/CD (Jenkins)
 
**Walk me through a Jenkins pipeline you built end-to-end — stages, triggers, and failure handling.**
*(Use your own real example here — this is the one question that must be personal.)* Structure your answer as: trigger (webhook on push / scheduled / manual), stages (checkout → build → test → security scan if applicable → deploy to staging → approval gate → deploy to prod), and failure handling (pipeline fails fast on any stage, sends a Slack/email alert, and you either auto-rollback or hold at the failed stage for manual intervention).
 
**Declarative vs. scripted Jenkins pipelines.**
Declarative uses a structured, opinionated syntax (`pipeline { stages { ... } }`) — easier to read, built-in validation, preferred for most teams. Scripted uses raw Groovy with full programmatic flexibility — more powerful but harder to maintain and easier to write fragile pipelines with. Default to declarative unless you need scripted's flexibility for complex conditional logic.
 
**How do you handle secrets/credentials securely within a Jenkins pipeline?**
Use the Jenkins Credentials plugin/store (never hardcode in Jenkinsfile) and reference them via `credentials()` binding or `withCredentials` blocks, which inject them as environment variables scoped only to that step and masked in logs. For more mature setups, integrate Jenkins with Vault or AWS Secrets Manager so secrets aren't stored in Jenkins at all.
 
**How would you design a pipeline to support multiple environments with different approval gates?**
Use parameterized/multi-branch pipelines: dev deploys automatically on every commit to a feature branch, staging deploys on merge to main (maybe with automated smoke tests as the gate), and production requires a manual `input` step (approval) plus possibly a change-ticket check. Keep environment-specific config (URLs, credentials) externalized so it's the same pipeline logic, different parameters per environment.
 
**What's your strategy for rolling back a failed deployment?**
Prefer automated rollback triggers on failed health checks (e.g., Kubernetes rollout automatically halts and you `kubectl rollout undo`, or blue-green lets you just flip traffic back). Keep every deployment versioned/tagged so "rollback" is just redeploying the last known-good artifact, not reconstructing it. The core principle: rollback should be as automated and fast as the deploy itself, not a manual scramble.
 
---
 
## 5. Monitoring & Observability
 
**Difference between monitoring and observability.**
Monitoring is watching predefined metrics/dashboards for known failure modes — "is CPU high, is the service up." Observability is having enough data (metrics, logs, traces) to answer questions you *didn't* predict in advance — debugging novel issues by exploring the system's actual internal state, not just checking a dashboard someone built ahead of time.
 
**How do you decide what to alert on vs. just log/dashboard?**
Alert only on things that need a human to act *right now* — symptoms that affect users (error rate spike, latency breach, service down), not every anomaly. Everything else goes to dashboards/logs for investigation later. Alert fatigue happens when low-signal alerts fire constantly and people start ignoring all of them — so alerts should map to actual SLO/SLA breaches, not raw thresholds on every metric.
 
**Walk me through how Prometheus scrapes and stores metrics — what is a time-series database?**
Prometheus pulls (scrapes) metrics from configured targets over HTTP at a set interval, storing each metric as a time series — a sequence of timestamped values with labels (key-value pairs) for filtering/grouping. A time-series database is optimized for this write-heavy, timestamp-ordered pattern and for fast range queries ("what was CPU over the last hour"), unlike a relational DB.
 
**"We use VictoriaMetrics — have you worked with it or something similar?"**
Be honest: *"I've worked with Prometheus and Grafana directly, not VictoriaMetrics specifically — but VictoriaMetrics is Prometheus-compatible, using the same data model, scrape config format, and PromQL query language, so the concepts transfer directly. I'd expect to be productive with it quickly given that overlap."*
 
**What's the ELK stack used for, and how does it differ from a metrics-based stack like Prometheus?**
ELK (Elasticsearch, Logstash, Kibana) is built for searching and analyzing unstructured **log** data — full-text search across log lines. Prometheus is built for structured **numeric time-series metrics** — efficient at aggregation and alerting on trends over time, not full-text log search. In practice, teams run both: metrics for "is something wrong," logs for "why is it wrong."
 
---
 
## 6. Incident Management & RCA (high-priority area)
 
**Walk me through an incident you resolved — detection to resolution.**
*(Use a real L2/L3 example, structured as:)* Detection (how you found out — alert, ticket, user report) → Triage (assessed impact/severity) → Investigation (what you checked, logs/metrics you pulled) → Fix (what you actually did) → Verification (confirmed it was resolved) → Follow-up (any prevention steps). Have 1–2 of these ready in detail.
 
**How do you write an RCA report? What sections does a good RCA include?**
Standard structure: **Summary** (what happened, in 2–3 sentences) → **Impact** (who/what was affected, duration, severity) → **Timeline** (detection through resolution, timestamped) → **Root Cause** (the actual underlying cause, not just the trigger) → **Contributing Factors** (things that made it worse or harder to catch) → **Resolution** (what fixed it) → **Action Items / Prevention** (specific, owned, dated follow-ups to prevent recurrence).
 
**Difference between a root cause and a contributing factor.**
The root cause is the fundamental reason the failure was *possible* — if you fixed only this, the incident wouldn't recur. Contributing factors made the incident worse, harder to detect, or more likely, but weren't the core cause. Example: root cause = a missing input validation allowed a bad config to deploy; contributing factor = no automated rollback meant it stayed live for 40 minutes instead of 2.
 
**How do you prevent the same incident from recurring after an RCA?**
Turn findings into concrete, owned action items with deadlines — not vague "we'll be more careful." Examples: add a specific alert that would've caught it sooner, add a validation/guardrail that makes the failure mode impossible, add a test case, or automate a manual step that was the actual trigger. Track these to completion, don't just close the RCA doc and move on.
 
**Prep note for you specifically:** since your resume doesn't show formal RCA ownership, take 1–2 real L2/L3 support incidents and reframe them explicitly using this structure (what happened → impact → root cause → fix → prevention) even if you didn't call it an "RCA" at the time. Interviewers care about the thinking process more than the paperwork title.
 
---
 
## 7. Linux & Scripting
 
**How would you troubleshoot a server running out of disk space in production?**
`df -h` to see which filesystem is full, then `du -sh /*` (or narrower) to find which directory is consuming space. Common culprits: log files that were never rotated, old core dumps, Docker images/containers piling up, or a runaway process writing to disk. Once identified, clear or archive the space safely, then fix the root cause — add log rotation, disk usage alerting, or automated cleanup so it doesn't recur.
 
**Difference between a hard link and a symbolic link.**
A hard link is a second directory entry pointing to the same inode/data on disk — if you delete the original file, the hard link still works because it *is* the data, just referenced twice. A symbolic link is a separate file that just contains a path pointing to another file — if the original is deleted or moved, the symlink breaks ("dangling link"). Hard links can't cross filesystems or point to directories; symlinks can do both.
 
**Bash one-liner to find and delete log files older than 7 days.**
```bash
find /var/log -name "*.log" -type f -mtime +7 -delete
```
Walk through it live: `find` searches the path, `-name "*.log"` filters by extension, `-type f` ensures only files (not directories), `-mtime +7` means modified more than 7 days ago, `-delete` removes matches. Good practice to mention: run it without `-delete` first (or with `-print`) to verify the match list before actually deleting anything in production.
 
**How does systemd manage services, and how do you debug a failed service start?**
systemd manages services via unit files (`.service` files) defining how to start/stop/restart a process, its dependencies, and restart policy. To debug a failed start: `systemctl status <service>` for a quick error summary, `journalctl -u <service>` for full logs, and check the unit file for misconfigured `ExecStart` paths, missing environment variables, or permission issues on files it needs.
 
---
 
## 8. Security
 
**Difference between SAST and DAST.**
SAST (Static Application Security Testing) analyzes source code without running it — catches issues like hardcoded secrets, insecure code patterns, injection vulnerabilities, early in development (in the pipeline, pre-build). DAST (Dynamic Application Security Testing) tests a running application from the outside, like an attacker would — catches runtime issues (auth bypass, misconfigurations) that only show up when the app is actually executing. Good pipelines use both: SAST early/fast, DAST later against a deployed environment.
 
**How would you manage secrets for an application running on Kubernetes?**
Kubernetes has built-in Secrets objects, but they're only base64-encoded (not encrypted) by default, so for anything sensitive you'd want them encrypted at rest (enable etcd encryption) or better, use an external secrets manager — HashiCorp Vault or AWS Secrets Manager — with a tool like the External Secrets Operator to sync secrets into the cluster at runtime rather than storing them statically. *If you haven't used Vault/Secrets Manager directly, say so plainly: "I understand the concept and the gap between raw K8s Secrets and a proper secrets manager — I haven't implemented Vault hands-on yet, but I'd approach learning it by starting with their basic secrets engine and the K8s auth method for pod-level access."*
 
**What is the principle of least privilege, and how have you applied it in IAM or Security Group design?**
Grant only the minimum access needed to do a job, nothing more — reduces blast radius if credentials are compromised. In IAM: scope policies to specific actions/resources instead of `*`, use roles instead of long-lived user keys, time-bound access where possible. In Security Groups: only open the specific ports/protocols needed, restrict source IPs/CIDR ranges instead of `0.0.0.0/0`, and separate rules per tier (web, app, db) so a compromised web server can't directly reach the database on an open rule.
 
---
 
## 9. Behavioral / Scenario-Based
 
**Tell me about a time a deployment failed in production — what did you do?**
Use the STAR structure: Situation (what was being deployed), Task (your role), Action (how you noticed, triaged, and fixed/rolled back — be specific about commands/steps), Result (downtime avoided or minimized, and what you changed afterward to prevent recurrence).
 
**Describe a time you had to learn a new tool quickly for a project. How did you approach it?**
Pick a real example (could be Terraform, EKS, Jenkins — your DevOps transition year is full of these). Structure: what forced the fast learning curve, how you approached it (docs, sandboxing in a non-prod environment, small experiments before touching anything real), and the outcome — ideally something concrete you shipped with it.
 
**How do you prioritize when multiple production issues come in at once?**
Triage by actual user/business impact first — is something fully down vs. degraded vs. a minor edge case. Fully-down, customer-facing issues take priority over internal or cosmetic ones. Communicate status/ETA to stakeholders early even before it's fully resolved, and don't context-switch chaotically between issues — stabilize the highest-impact one first, then move to the next.
 
**Tell me about a decision you made that improved a process.**
This is your best spot to use the **patch management / vulnerability backlog** story (or the 30% VMware consolidation number) — structure it as: what the process looked like before (manual, slow, or risky), the decision you made, the specific action you took, and the measurable result (the % number). Numbers are what make this answer memorable versus a generic "I improved things."
 
---
 
## How to use this before the interview
 
1. **Fill in your real numbers and stories** wherever this guide says "your example" — vague answers are the #1 thing that makes technical interviewers dig deeper and lose confidence.
2. **Don't memorize verbatim** — know the structure of each answer so you can speak it naturally and handle follow-up questions.
3. **Practice out loud**, especially the Jenkins pipeline walkthrough, the incident/RCA story, and the "tell me about a time..." questions — these are the ones interviewers follow up on most.
4. **It's fine to say "I haven't done X hands-on, but here's how I understand it / would approach learning it"** — especially for Vault, VictoriaMetrics, and formal RCA ownership. Honesty here builds more trust than a strained claim of expertise.
 
