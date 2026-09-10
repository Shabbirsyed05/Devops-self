# 🚀 DevOps Interview Preparation Notes

> Source: Compiled from Sakshi Gautam's YouTube video series on DevOps interview questions.
> Format: Simplified explanations for quick revision + original detail preserved for depth.
> 📌 **This file is a living document — new topics will keep getting added below.**

---

## 📑 Table of Contents

1. [Interview Structure & Resume Prep](#1-interview-structure--resume-prep)
2. [Linux & Shell Scripting](#2-linux--shell-scripting)
3. [Git](#3-git)
4. [Maven](#4-maven)
5. [Jenkins (CI/CD)](#5-jenkins-cicd)
6. [Docker & Containerization](#6-docker--containerization)
7. [Kubernetes (K8s)](#7-kubernetes-k8s)
8. [Ansible (Configuration Management)](#8-ansible-configuration-management)
9. [AWS (Cloud)](#9-aws-cloud)
10. [Terraform (IaC)](#10-terraform-iac)
11. [Linux Power Commands: xargs & visudo](#11-linux-power-commands-xargs--visudo)
12. [AWS DevOps CI/CD Services](#12-aws-devops-cicd-services)
13. [Ansible — Advanced & Scenario-Based](#13-ansible--advanced--scenario-based)
14. [Docker Deep-Dive & Docker Swarm](#14-docker-deep-dive--docker-swarm)
15. [Kubernetes — Scenario-Based Troubleshooting (Round 2)](#15-kubernetes--scenario-based-troubleshooting-round-2)
16. [Kubernetes Deployment Strategies](#16-kubernetes-deployment-strategies)
17. [Load Balancing, Ingress, DR Strategy & Node Maintenance](#17-load-balancing-ingress-dr-strategy--node-maintenance)
18. ["What Are Your Daily Roles & Responsibilities?" — The Big Interview Question](#18-what-are-your-daily-roles--responsibilities--the-big-interview-question)
19. [Shell Scripting — Text Processing Commands](#19-shell-scripting--text-processing-commands)
20. [Networking & Security Fundamentals](#20-networking--security-fundamentals)
21. [Monitoring & Observability Tools](#21-monitoring--observability-tools)
22. [Jenkins CI/CD Pipeline — Declarative Deep-Dive](#22-jenkins-cicd-pipeline--declarative-deep-dive)
23. [Ansible — Hands-On Playbook, Roles & Looping](#23-ansible--hands-on-playbook-roles--looping)
24. [AWS Site-to-Site VPN](#24-aws-site-to-site-vpn)
25. [Cloud Cost Optimization (FinOps)](#25-cloud-cost-optimization-finops)
26. [AWS Storage: EFS (Elastic File System)](#26-aws-storage-efs-elastic-file-system)
27. [AWS Storage: S3 (Simple Storage Service)](#27-aws-storage-s3-simple-storage-service)
28. [AWS Storage: EBS (Elastic Block Store)](#28-aws-storage-ebs-elastic-block-store)
29. [EBS Quick-Fire Q&A](#29-ebs-quick-fire-qa)
30. [Common Errors & Solutions — Round 1](#30-common-errors--solutions--round-1)
31. [Common Errors & Solutions — Round 2](#31-common-errors--solutions--round-2)
32. [Real-World Architecture & Best Practices](#32-real-world-architecture--best-practices)
33. [Azure DevOps Interview Preparation](#33-azure-devops-interview-preparation)
34. [Shell Scripting — 6 Scenario Scripts](#34-shell-scripting--6-scenario-scripts)
35. [Ansible Integration, Ingress Setup, Error Handling, DR & EKS Upgrade (Q&A)](#35-ansible-integration-ingress-setup-error-handling-dr--eks-upgrade-qa)
36. [S3 Tiering, Spot Instances, Lost SSH Keys, Git SSH Auth & Jenkins Security (Q&A)](#36-s3-tiering-spot-instances-lost-ssh-keys-git-ssh-auth--jenkins-security-qa)
37. [Jenkins Multi-Environment & Multi-Branch Pipelines (Deep-Dive)](#37-jenkins-multi-environment--multi-branch-pipelines-deep-dive)
38. [System Design Scenarios: API Security, Data Loss, Leaked Secrets & Multi-Cloud HA](#38-system-design-scenarios-api-security-data-loss-leaked-secrets--multi-cloud-ha)
39. [Introduce Yourself & Daily Roles — The Complete Framework](#39-introduce-yourself--daily-roles--the-complete-framework)
40. [Terraform in CI/CD & Advanced Concepts](#40-terraform-in-cicd--advanced-concepts)
41. [Service Mesh](#41-service-mesh)
42. [Real-World Scenarios: API Gateway vs LB, VPC Peering, ASG & Monitoring](#42-real-world-scenarios-api-gateway-vs-lb-vpc-peering-asg--monitoring)
43. [Git Advanced: Merge vs Rebase, Submodules, Hooks, Branching & Versioning](#43-git-advanced-merge-vs-rebase-submodules-hooks-branching--versioning)
44. [Managerial Round — Behavioral Answers](#44-managerial-round--behavioral-answers)
45. [Building Your DevOps Resume/CV](#45-building-your-devops-resumecv)
46. [Linux Interview Questions — Complete Answers](#46-linux-interview-questions--complete-answers)
47. [Kubernetes Architecture & Request Lifecycle (Deep-Dive)](#47-kubernetes-architecture--request-lifecycle-deep-dive)
48. [Kubernetes Security Hardening (Deep-Dive)](#48-kubernetes-security-hardening-deep-dive)
49. [Python Scripting for DevOps](#49-python-scripting-for-devops)
50. [Quick-Fire Practice Questions: Git & Jenkins](#50-quick-fire-practice-questions-git--jenkins)
51. [System Design for DevOps/SRE Interviews](#51-system-design-for-devopssre-interviews)
52. [SonarQube — SAST & Quality Gates](#52-sonarqube--sast--quality-gates)
53. [OWASP Tools — DAST & SCA in CI/CD](#53-owasp-tools--dast--sca-in-cicd)
54. [Terraform: Provisioning EKS Infrastructure (Full Walkthrough)](#54-terraform-provisioning-eks-infrastructure-full-walkthrough)
55. [DevOps Project Explanation Framework](#55-devops-project-explanation-framework)
56. [Project Deep-Dive — Step 2: Fetching Code from SCM](#56-project-deep-dive--step-2-fetching-code-from-scm)
57. [Project Deep-Dive — Step 3: Setting Up the CI/CD Pipeline](#57-project-deep-dive--step-3-setting-up-the-cicd-pipeline)
58. [Project Deep-Dive — Step 4: Deploying the Application](#58-project-deep-dive--step-4-deploying-the-application)
59. [Project Deep-Dive — Step 5: Monitoring & Logging Setup](#59-project-deep-dive--step-5-monitoring--logging-setup)
60. [Kubernetes Headless Services & Pod-to-Pod Communication (Deep-Dive)](#60-kubernetes-headless-services--pod-to-pod-communication-deep-dive)
56. [Project Step 2: Fetch Code from SCM — Branching & Webhooks](#56-project-step-2-fetch-code-from-scm--branching--webhooks)
57. [Project Step 3: Setting Up the CI/CD Pipeline (Full Script)](#57-project-step-3-setting-up-the-cicd-pipeline-full-script)
58. [Project Step 4: Deploying the Application (Manifests & Helm)](#58-project-step-4-deploying-the-application-manifests--helm)
59. [Project Step 5: Monitoring & Logging Setup](#59-project-step-5-monitoring--logging-setup)
60. [Headless Services — Deep Dive](#60-headless-services--deep-dive)
61. [Pod-to-Pod Communication & Traffic Routing](#61-pod-to-pod-communication--traffic-routing)

---

## 1. Interview Structure & Resume Prep

**Simple takeaway:** DevOps interviews usually happen in 3 stages, and getting past the resume screen is 50% of the battle.

### Typical Interview Flow
| Stage | What Happens |
|---|---|
| **Resume Shortlisting** | ATS (Applicant Tracking System) or HR filters resumes using keyword match against the Job Description (JD) |
| **Technical Round 1 (Foundations)** | Basic syntax & fundamentals — Linux, Git, Shell, Docker, K8s, Jenkins, Terraform |
| **Technical Round 2 (Project Deep-Dive)** | Real project experience, day-to-day responsibilities, AWS scenarios, cloud security & cost optimization |
| **HR Round** | Culture fit + salary negotiation |

> 💡 Some companies merge Round 1 & 2 into a single technical round.

### Beating ATS & Resume Screening
- **Keyword Matching:** Most companies use AI-based ATS to match your resume against the JD — so **mirror the exact tool names/keywords** from the JD in your resume.
- **Tailor Every Application:** Don't send one generic resume — adjust the summary & bullet points per role.

### Recommended Resume Structure (2-Column Layout)
**Right Column — Work History & Impact**
- Professional Experience (roles, responsibilities, tools used, promotions)

**Left Column — Credentials & Foundations**
- Core Skills (grouped: Linux, CI/CD, Containers, IaC, Cloud)
- Certifications (AWS, Azure, CKA/CKAD, etc.)
- Achievements & Recognition
- Education

---

## 2. Linux & Shell Scripting

**Simple takeaway:** Know your way around the filesystem, basic commands, and be ready to explain a real script you've written.

### Directory Structure
| Directory | Purpose |
|---|---|
| `/bin` | Essential user binaries/commands |
| `/sbin` | System admin binaries |
| `/etc` | Configuration files |
| `/opt` | Optional/third-party software |

### Must-Know Commands
- `ls` → list files
- `touch` → create empty file
- `uname` → system info
- `users` → currently logged-in users
- `ifconfig` → check IP address
- `vi` → edit text files
- `df` / `du` → disk space usage (filesystem-level vs directory-level)

### Shell Scripting
- Be ready to discuss a **real script** you wrote at work (automation, backups, monitoring, etc.)
- **Shebang (`#!`)**: first line of a script (e.g. `#!/bin/bash`) that tells the OS which interpreter to use to run the script.

---

## 3. Git

**Simple takeaway:** Git is a *distributed* version control system — everyone has a full copy of the repo locally. Interviewers love comparing commands (`reset` vs `revert`, `merge` vs `rebase`, etc.)

### Architecture & Fundamentals
- **Why SCM (Source Code Management) matters:** enables multiple developers to collaborate, track history, work in parallel, and connect onshore/offshore teams.
- **Git vs SVN:**
  - SVN = **centralized** (one central server holds the code)
  - Git = **distributed** (every developer has a full local repo → can work offline, commit fast, branch cheaply)

### The 4 Git Areas
1. **Working Directory** — where you edit files
2. **Staging Area (Index)** — snapshot of files ready to commit
3. **Local Repository** — committed history stored on your machine
4. **Remote Repository** (GitHub/GitLab) — shared central copy

### Bare vs Non-Bare Repo
- **Bare repo:** no working directory (used on servers like GitHub)
- **Non-bare repo:** has an active working directory (used on your laptop)

### Branching Strategy
- `master/main` → stable production branch
- `development` → long-lived integration branch
- `feature/*` → individual feature branches
- `hotfix/*` → urgent fixes cut directly from master

### Branch vs Tag
| | Branch | Tag |
|---|---|---|
| Mutability | Moves forward with new commits | Fixed/immutable |
| Use case | Active development | Marking releases (`v1.0.0`) |

### Key Commands & Differences (Very common interview Qs)
| Command | Meaning |
|---|---|
| `git stash` | Temporarily shelve uncommitted changes to switch context, restore later with `stash apply`/`pop` |
| `git reset` | Moves HEAD back / unstages files — **rewrites local history** |
| `git revert` | Creates a **new commit** that undoes a previous commit — safe for shared history |
| `git clone` | Creates a brand-new local repo from remote |
| `git pull` | Updates existing repo (= `fetch` + `merge`) |
| `git fetch` | Downloads remote changes **without** touching your working directory |
| `git merge` | Combines branches, keeps full history (creates merge commit) |
| `git rebase` | Replays your commits on top of another branch → clean, linear history |
| `git cherry-pick` | Applies one specific commit (by hash) onto another branch |
| `git squash` | Combines multiple small commits into one clean commit |
| `git bisect` | Binary search through commit history to find the commit that introduced a bug |

### Git Hooks
Scripts triggered automatically on events:
- `pre-commit` → enforce commit message rules / run linters
- `post-commit` → trigger notifications/automation

### Scenario-Based Questions
- **Keep secrets out of Git:** use `.gitignore`
- **Shallow clone for CI/CD** (faster, less bandwidth):
  ```bash
  git clone --single-branch --branch <branch-name> --depth 1 <repo-url>
  ```
- **Check remote URLs:** `git remote -v`
- **Submodules:** embed another Git repo inside your project without merging source trees

### ⚠️ Correction Note
OpenShift ≠ Jenkins. **OpenShift** = enterprise Kubernetes platform. **Jenkins** = CI/CD automation tool.

---

## 4. Maven

**Simple takeaway:** Maven = build automation tool for Java. It follows "convention over configuration" so you don't write build steps manually like in Ant.

### What Maven Does
Automates **build → test → package** into deployable artifacts (JAR, WAR, EAR).

### Ant vs Maven
| | Ant | Maven |
|---|---|---|
| Config file | `build.xml` | `pom.xml` |
| Style | Imperative (you write every step) | Declarative (follows convention) |
| Dependency management | Manual | Automatic |

### Prerequisites
- Java (JDK) installed
- `JAVA_HOME` and `PATH` environment variables configured

### pom.xml (Project Object Model)
Contains: project identity, dependencies, plugins, build config, resources, profiles.

**Minimum required elements:**
- `<project>` root
- `<modelVersion>` (usually `4.0.0`)
- `<groupId>` (organization/namespace)
- `<artifactId>` (project name)
- `<version>`

- **`<packaging>`** tag defines output type: `jar`, `war`, `pom`
- **Super POM:** the parent POM every Maven project inherits — sets default plugin bindings & directory layout
- **settings.xml:** user/system-level config — proxies, credentials, custom local repo path (`~/.m2/repository`)

### Repositories
1. **Local** — cached on your machine (`~/.m2/repository`)
2. **Central** — public default Maven repo
3. **Remote/Private** — org-hosted (Nexus, JFrog Artifactory)

### Dependencies
- **Transitive Dependency:** If A depends on B, and B depends on C → Maven auto-pulls C.
- **Dependency Exclusion:** use `<exclusions>` to block unwanted transitive dependencies.

### Lifecycles & Phases
- **Clean:** wipes `target/` directory
- **Default (Build):** `validate → compile → test → package → install → deploy`
- **Site:** generates documentation/reports

### Useful Commands
```bash
# Generate a new project skeleton
mvn archetype:generate

# Build while skipping unit tests
mvn clean package -DskipTests=true
```

---

## 5. Jenkins (CI/CD)

**Simple takeaway:** Jenkins automates build/test/deploy. Know pipeline types, controller-agent architecture, and be ready for troubleshooting scenarios.

### Fundamentals
- **CI/CD Benefits:** faster bug detection, quicker releases, automated integration
- **Prerequisite:** Java (JRE/JDK)
- **Jenkins Home:** `/var/lib/jenkins`
- **Logs:** `/var/log/jenkins/jenkins.log`
- **Written in:** Java

### Architecture
- **Controller-Agent model** (formerly Master-Slave): controller orchestrates, agents execute the actual build workloads — helps scale across many jobs.
- **Shared Libraries** (`vars`, `src`, `resources`): reusable pipeline code shared across teams — keeps pipelines DRY.

### Pipeline Types
- Freestyle
- Pipeline (**Declarative** vs **Scripted** syntax)
- Multibranch Pipeline (auto-detects Git branches & runs matching `Jenkinsfile`)
- Maven project

**Basic Declarative Pipeline Structure:**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') { steps { /* checkout code */ } }
        stage('Build')    { steps { /* build */ } }
        stage('Test')     { steps { /* test */ } }
        stage('Deploy')   { steps { /* deploy */ } }
    }
    post { /* always/success/failure blocks */ }
}
```

### Build Triggers
| Trigger | Description |
|---|---|
| GitHub Webhook | Instant build on push (recommended) |
| Poll SCM | Periodically checks for new commits |
| Build Periodically | Cron-like scheduled builds |

- **Active Choices Plugin:** dynamic/cascading/reactive dropdown parameters

### Security, Backups & Maintenance
- **Access Control:** Matrix-based security, Project-based Matrix authorization, Role Strategy Plugin (RBAC)
- **Backups:** back up `/var/lib/jenkins` (config XMLs, job configs) or use backup plugins
- **Disk Management:**
  - Enable "Discard Old Builds" (e.g., keep only last 5 builds)
  - Use Workspace Cleanup Plugin to wipe temp files post-build

### Troubleshooting Scenarios (Common in Interviews!)
| Problem | Likely Cause |
|---|---|
| Build stuck in queue | No available executor matching label/node, or agent offline |
| Stuck on "Preparing Jenkins" at boot | Corrupted config XML or bad/incompatible plugin — check logs |
| "Works locally, fails in Jenkins" | Environment mismatch — missing tools, wrong PATH, version differences on agent |
| Need to fail build early on a keyword in logs | Use Text-finder / Log Parser plugin or a Groovy script check |

---

## 6. Docker & Containerization

**Simple takeaway:** Know Dockerfile basics, the ENTRYPOINT vs CMD distinction, and how Docker differs from Kubernetes.

### Core Concepts
- **Dockerfile syntax:** instructions to build an image (FROM, RUN, COPY, CMD, ENTRYPOINT, etc.)
- **ENTRYPOINT vs CMD:**
  - `ENTRYPOINT` → fixed command that always runs
  - `CMD` → default arguments, can be overridden at runtime
- **Image tagging:** versioning images (e.g., `myapp:1.0`)

### Management & Composition
- `docker system prune` → cleans up unused containers/images/networks
- **Overriding environment variables:** pass `-e KEY=VALUE` at runtime or via `docker-compose.yml`
- `docker-compose` → run multi-container applications together
- **Docker vs Kubernetes:** Docker runs/builds containers on a single host; Kubernetes orchestrates containers **across multiple hosts/nodes** (scaling, self-healing, load balancing).

---

## 7. Kubernetes (K8s)

**Simple takeaway:** Understand control plane vs worker node roles, what happens on failure, and the difference between Pods, Containers, and workload types.

### Architecture & Failure Scenarios
- **Control Plane Components:** API Server, Scheduler, Controller Manager, etcd
- **If Master Node Fails:**
  - Worker nodes **keep running existing workloads**
  - BUT new scheduling, auto-scaling, and control-plane operations **stop working**
- **etcd:** the cluster's key-value store — holds all cluster state/configuration. Critical component.

### Workloads & Pods
- **Init Containers:** run before the main app container starts (setup tasks)
- **Sidecar Containers:** run alongside the main container (e.g., logging agent)
- **Ephemeral Containers:** temporary containers for debugging a running pod
- **Pod vs Container:**
  - Container = single running process/app
  - Pod = smallest deployable unit in K8s, can hold **one or more containers** that share network/storage
- **StatefulSets:** manage stateful apps needing stable identities & storage (e.g., databases)
- **Headless Services:** used with StatefulSets to give each pod a stable DNS name (no load-balanced single IP)

### Manifest Spec Components
- `selector` → matches pods to a controller (e.g., Deployment)
- `replicas` → number of pod copies to run
- `template` → pod metadata + spec (defines what each pod looks like)

---

## 8. Ansible (Configuration Management)

**Simple takeaway:** Ansible = agentless configuration management tool, control node must be Linux/WSL.

### Components
- **Roles:** reusable, organized sets of tasks
- **Ansible Galaxy:** community hub to share/download roles
- **Ad-hoc commands:** quick one-off commands without writing a full playbook

### Playbook Structure
- Define `hosts` (target machines) and `tasks` (actions to perform)
- Run with: `ansible-playbook <playbook>.yml`
- Validate syntax before running: `ansible-playbook <playbook>.yml --syntax-check`

### OS Support
- **Control node** (where Ansible runs from) **requires Linux or WSL** — cannot run natively on Windows.
- **Managed nodes** (targets) can be Windows or Linux.

---

## 9. AWS (Cloud)

**Simple takeaway:** Focus on Load Balancer types and EBS backup strategy — these come up frequently in scenario rounds.

### Elastic Load Balancers
- **ALB (Application Load Balancer):** Layer 7, routes based on URL/host/path — good for HTTP/HTTPS microservices
- **NLB (Network Load Balancer):** Layer 4, handles extreme performance/low-latency TCP traffic

### EBS (Elastic Block Store)
- **Snapshots:** point-in-time backups of EBS volumes, stored in S3
- **Backup Strategy:** automate snapshots (e.g., via AWS Backup or Lifecycle Manager), store across regions for disaster recovery

---

## 10. Terraform (IaC)

**Simple takeaway:** Know the core workflow (`init → plan → apply`), how state is managed/secured, and modules vs workspaces.

### Essential Workflow & Commands
| Command | Purpose |
|---|---|
| `terraform init` | Initializes directory, downloads provider plugins, sets up backend |
| `terraform plan` | Compares `.tf` code vs `terraform.tfstate` vs real infra → shows execution plan |
| `terraform apply` | Provisions/modifies resources per the plan |
| `terraform destroy` | Tears down resources |
| `terraform show` | Displays current state |
| `terraform validate` | Checks config syntax |

**Automation flags:**
```bash
# Skip approval prompt
terraform apply --auto-approve
terraform destroy --auto-approve

# Target a specific resource only
terraform apply --target="resource_type.resource_name"
```

**Bring existing cloud resources under Terraform management:**
```bash
terraform import <resource_type.resource_name> <resource_id>
```

### State File Management (`terraform.tfstate`)
- **Role:** maps your `.tf` config to real-world cloud resources; tracks metadata & drift
- **Disaster Recovery Best Practices:**
  - Store remotely (e.g., AWS S3 with **versioning** enabled + strict IAM access control)
  - Use **State Locking** (e.g., DynamoDB) to prevent concurrent writes/race conditions
  - Maintain scheduled automated backups

### Multi-Environment Strategy
| | Modules | Workspaces |
|---|---|---|
| Purpose | Reusable, parameterized "blueprints" to avoid repeating code | Manage separate state files (dev/uat/prod) using the **same** code |

### Providers vs Provisioners
- **Providers:** plugins that talk to APIs of cloud/platforms (AWS, Azure, GCP)
- **Provisioners** (last resort — run scripts during create/destroy):
  - `local-exec` → runs script locally (on the machine running Terraform)
  - `remote-exec` → SSH/WinRM into the new resource to run commands
  - `file` → copies files to the target resource

### Advanced Configuration
- **Secrets:** never hardcode — use environment variables (`TF_VAR_*`) or a secret store like **HashiCorp Vault**
- **Loops:** `count` (simple repetition) or `for_each` (complex maps/sets)
- **Explicit Dependencies:** `depends_on` — used when Terraform can't auto-detect resource order
- **Lifecycle Rules:** `create_before_destroy`, `prevent_destroy` — protect critical resources
- **Variables & Outputs:**
  - Define inputs → `variables.tf`
  - Assign values → `terraform.tfvars`
  - Expose values → `output` blocks

### Terraform vs Ansible
| | Terraform | Ansible |
|---|---|---|
| Purpose | **Provisioning infrastructure** (VPCs, subnets, VMs, load balancers) | **Configuration management** (installing software, configuring services on existing servers) |

---

## 11. Linux Power Commands: xargs & visudo

**Simple takeaway:** These two commands signal "real-world CLI fluency" in interviews — small commands, big impact.

### `xargs`
- **Purpose:** builds and executes commands using input piped from another command — turns output into arguments.
- **Example (find + delete in one line):**
  ```bash
  find . -type f -name "*.txt" | xargs rm
  ```
- **Why it matters in interviews:** shows you can write efficient one-liners instead of clunky multi-step scripts/loops.

### `visudo`
- **What sudo does:** grants temporary admin/root privileges to regular users, controlled via `/etc/sudoers`.
- **Why NOT to edit `/etc/sudoers` directly with `vi`/`nano`:**
  1. **Syntax validation:** `visudo` checks syntax before saving — a typo in a raw edit can **break sudo entirely and lock out admins**.
  2. **File locking:** prevents concurrent edits that could cause race conditions or file corruption.

---

## 12. AWS DevOps CI/CD Services

**Simple takeaway:** AWS has its own native CI/CD suite that mirrors the open-source tools you already know — map each AWS service to its "equivalent" to memorize fast.

### AWS CI/CD Tools vs Open-Source Equivalents
| AWS Service | What It Does | Equivalent To |
|---|---|---|
| **CodeCommit** | Managed Git source code repository | GitHub / GitLab |
| **CodePipeline** | Orchestrates & automates release phases (continuous delivery) | Jenkins Pipelines |
| **CodeBuild** | Fully managed build/compile service. Stores artifacts in **S3**, pushes container images to **ECR** | Maven / Docker build agents |
| **CodeDeploy** | Automated deployment to EC2, ECS, EKS, or on-prem servers | Ansible/Deployment scripts |

### Pipeline Configuration Files
- **IAM Permissions:** every AWS DevOps service works on **least-privilege IAM roles/policies** — this is the #1 thing to mention in interviews.
- **`buildspec.yml`** (used by CodeBuild) — defines build phases:
  - `install` → set up runtime & install tools
  - `pre_build` / `build` → compile code & run tests
  - `post_build` → package artifacts, push image to ECR/S3
- **`appspec.yml`** (used by CodeDeploy) — controls deployment lifecycle hooks:
  - `ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService`
- **CodeDeploy Agent:** must be installed & running on target EC2/on-prem instances so they can **poll** for deployment commands.

### Real-World Best Practices (Common Interview Scenarios)
- **Container Build Issues:**
  - Never run containers as **root** in a Dockerfile
  - Use lightweight official base images (e.g., **Alpine**) → smaller attack surface, faster builds
- **Secrets Management:** never hardcode secrets in repo/pipeline configs — pull them from **AWS Secrets Manager** or **SSM Parameter Store**
- **Most Common CodeDeploy Failure:** CodeDeploy agent stopped/missing, or missing IAM role permissions on the target EC2 instance
- **Notifications:** use **Amazon SNS** + CodePipeline/EventBridge for build status & approval notifications

### Supporting AWS Operational Services
| AWS Service | DevOps Role | Equivalent To |
|---|---|---|
| **CloudFormation** | Native IaC using JSON/YAML stacks | Terraform |
| **OpsWorks** | Managed Chef/Puppet config management | Ansible |
| **CloudWatch** | Log aggregation, metrics, alarms, dashboards | Prometheus/Grafana |
| **CloudTrail** | Audits API calls & admin activity account-wide | Audit logging |
| **X-Ray** | Distributed tracing & latency bottleneck analysis | Jaeger/Zipkin |
| **Lambda** | Serverless triggers (e.g., auto-remediation when an alarm fires) | Event-driven automation |

---

## 13. Ansible — Advanced & Scenario-Based

**Simple takeaway:** This builds on Section 8 — focus on *why* Ansible is chosen over Chef/Puppet, and how it's used in real production scenarios (bastion hosts, scaling, Terraform integration).

### Ansible vs Terraform (again — commonly asked together)
- **Terraform** → provisions infrastructure (IaC)
- **Ansible** → configures/updates/deploys software on servers that **already exist**

### Why Ansible over Chef / Puppet
| | Ansible | Chef / Puppet |
|---|---|---|
| Agent required? | **Agentless** — connects via SSH (Linux) / WinRM (Windows) | Requires an agent daemon on each node |
| Model | **Push** — control node pushes config out | **Pull** — nodes poll a central master server |

### Prerequisites & OS Support
- Requires **Python** installed on both control node & target nodes
- Control node **cannot be native Windows** (needs Linux/WSL) — but Ansible **can manage** Windows targets via WinRM

### Inventories
- Defines managed nodes, grouped by environment/role
- Default location: `/etc/ansible/hosts`
- Supports **static** and **dynamic** inventories
- **Terraform → Ansible integration:** After Terraform provisions instances, their IPs are passed into Ansible inventory (via dynamic inventory scripts or `local-exec`)

### Ad-Hoc Commands
Quick one-liner modules run without a full playbook:
```bash
ansible all -m shell -a "uptime"
```

### Playbook Core Directives
- `hosts` → target inventory group or `localhost`
- `become: true` → escalate privileges via sudo
- `tasks` → list of actions mapped to modules (`yum`/`apt`, `copy`, `systemd`, `win_package`, etc.)

```bash
# Syntax check before running
ansible-playbook --syntax-check playbook.yml

# Run with a custom inventory file
ansible-playbook -i custom_hosts playbook.yml
```

- **Loops:** `loop` or `with_items` → iterate over lists (packages, users, etc.)
- **Conditionals:** `when` → run tasks conditionally, e.g. `when: ansible_distribution == 'CentOS'`
- **Handlers & `notify`:** tasks that fire only when another task reports a **changed** state (e.g., restart Nginx only if its config file changed)
- **Ansible Facts:** system metadata auto-gathered at play start (IPs, OS version, disk partitions)

### Roles, Modularity & Security
- **Roles folder structure:** `tasks/`, `handlers/`, `templates/`, `files/`, `vars/`, `defaults/`, `meta/`
- **Ansible Galaxy:** public hub for pre-built roles → `ansible-galaxy init <role_name>`
- **Ansible Tower / AWX:** web UI for RBAC, scheduling, centralized logging
- **Ansible Vault:** built-in encryption for sensitive variables/passwords/files

### Advanced Operational Scenarios
- **Connecting via a Bastion/Jump Host** (2 approaches):
  1. Define `ProxyJump` in `~/.ssh/config`
  2. Set `ansible_ssh_common_args: '-o ProxyJump=user@jumphost'` in inventory file or `ansible.cfg`
- **`ansible.cfg`:** global config — default paths, SSH timeouts, inventory location, privilege escalation rules
- **Performance Optimization for Large Fleets:** increase parallelism by raising `forks` in `ansible.cfg` (default = 5) or pass `-f <number>` via CLI

---

## 14. Docker Deep-Dive & Docker Swarm

**Simple takeaway:** This expands Section 6 — go deeper into Dockerfile internals, container lifecycle commands, networking, and Docker Swarm as a lightweight alternative to Kubernetes.

### Core Components
- **Docker Client:** CLI tool used to interact with Docker
- **Docker Daemon (`dockerd`):** background service managing images, containers, networks, volumes
- **Registry vs Repository:**
  - **Repository** = collection of related images with different tags/versions
  - **Registry** (Docker Hub, AWS ECR, ACR) = hosts multiple repositories
- **Image vs Container:**
  - **Image** = immutable, read-only template (code + dependencies)
  - **Container** = runnable, isolated instance of that image

### Dockerfile Optimization Best Practices
- Use **multi-stage builds** → discard compiler/build tools, shrink final image size
- Use lightweight official base images (**Alpine**)
- Prefer **exec form** `["executable", "param1"]` over shell form for `CMD`/`ENTRYPOINT` → ensures signals like `SIGTERM` pass through cleanly

### Key Instruction Comparisons
| Instruction | Meaning |
|---|---|
| `RUN` | Executes at **build time** |
| `CMD` | Default command/args, **overridable at runtime** |
| `ENTRYPOINT` | Fixed container entry executable |
| `COPY` | Moves local files into the container |
| `ADD` | Like COPY, but can also fetch remote URLs & auto-unpack local tar archives |
| `ARG` | Build-time variable, passed via `--build-arg` |
| `ENV` | Persistent environment variable inside the running container |

```bash
# Use a custom Dockerfile path/name
docker build -f <custom-dockerfile-path> .
```

### Container Lifecycle, CLI & Troubleshooting
- **`docker stop` vs `docker kill`:** `stop` sends `SIGTERM` (graceful) then `SIGKILL` if it times out; `kill` sends `SIGKILL` immediately
- **`docker run` vs `docker create`:** `create` preps the container in a stopped state; `run` creates **and** starts it immediately

**Moving an image without a registry:**
```bash
docker save -o image.tar <image_name>        # archive to tarball
scp image.tar user@remote-host:/path         # transfer file
docker load -i image.tar                      # unpack on target host
```

- **Garbage Collection:** `docker system prune` → removes unused/dangling resources
- **Troubleshooting:**
  ```bash
  docker logs <container_id>              # view logs
  docker exec -it <container_id> /bin/sh  # interactive shell
  ```

### Storage & Networking
- **Bind Mounts:** maps a specific host directory into the container
- **Named Volumes:** managed by Docker, decoupled from container lifecycle
- **Network Drivers:**
  | Driver | Behavior |
  |---|---|
  | `bridge` | Default network for standalone containers on one host |
  | `host` | Bypasses network isolation, uses host's network stack directly |
  | `none` | Disables networking entirely |
  | `overlay` | Multi-host networking — containers on different hosts/nodes communicate securely |
- **Container-to-container discovery:** containers on the same custom bridge/overlay network can resolve each other by **container/service name**

### Docker Compose
- Declarative YAML defining multi-container apps (`version`, `services`, `networks`, `volumes`)
- `depends_on` → enforces startup sequence between services

### Docker Swarm
- **Architecture:** built-in clustering — **Manager nodes** (orchestration/state) + **Worker nodes** (run tasks)
```bash
# On manager
docker swarm init

# On each worker
docker swarm join --token <token> <manager-ip>
```
- **Service Deployment Modes:**
  - **Replicated Mode:** fixed number of identical replica tasks across the swarm
  - **Global Mode:** exactly one task per node (great for monitoring agents/log collectors)
- **Swarm vs Kubernetes:** Swarm is simpler to set up but lacks native advanced auto-scaling, complex traffic routing, and the rich ecosystem that makes K8s the industry standard
- **Zero-Downtime Node Maintenance:**
  ```bash
  docker node update --availability drain <node>
  ```
  Migrates running tasks off the node before OS updates.

---

## 15. Kubernetes — Scenario-Based Troubleshooting (Round 2)

**Simple takeaway:** Round 2 interviews test *how you'd actually debug production issues*, not just definitions. Memorize the **command → cause → fix** flow for each scenario.

### Scenario: Pod in CrashLoopBackOff or High Resource Usage
**Diagnose:**
```bash
kubectl get pods
kubectl describe pod <pod_name>
kubectl logs <pod_name>          # add -p to see PREVIOUS crashed container's logs
```
**Common Root Causes:**
- **OOM / Resource Limits:** container hits memory limit → fine-tune `requests`/`limits` in the manifest
- **Downstream Dependency Failure:** DB or upstream service connection failing
- **DNS/Network Errors:** misconfigured CoreDNS or network policies blocking service discovery
- **Config Mismatches:** invalid image tag, broken env vars, missing ConfigMaps/Secrets
- **Prevention:** set up Prometheus/Grafana alerting to catch spikes *before* outages

### Scenario: Worker Node in `NotReady` State
```bash
kubectl get nodes
kubectl describe node <node-name>
```
Then **SSH into the node** and check:
- Kubelet daemon logs: `journalctl -u kubelet`
- Container runtime status (`containerd`/Docker)
- Host-level networking/disk pressure issues

### Scenario: High CPU / Resource Spikes
```bash
kubectl top pods
kubectl top nodes
```
- Enforce explicit `requests` and `limits` in pod spec
- Implement **Horizontal Pod Autoscaler (HPA)** based on CPU/memory thresholds

### Scenario: Migrating Monolith → Microservices on Kubernetes
Key pillars to mention:
- **High Availability:** deploy across **multi-AZ** worker nodes
- **Auto-scaling:** HPA + cluster autoscaler for traffic spikes
- **Pod Disruption Budgets (PDB):** guarantee minimum replicas stay up during node drains/updates
- **Security:** manage secrets via **AWS Secrets Manager / HashiCorp Vault** + strict **RBAC** (never plain-text in manifests)
- **Health Probes:**
  - **Liveness Probe** → restarts dead/unresponsive containers
  - **Readiness Probe** → only routes traffic once the pod is truly ready
- Decouple config/secrets via **ConfigMaps** + external secret stores
- Set up cluster-wide monitoring (Prometheus/Grafana)

### Scenario: Pod Deleted/Crashed — Who Fixes It?
- The **ReplicaSet controller** detects drift between desired vs current state → immediately provisions a replacement pod
- The **kube-scheduler** (control plane) evaluates node capacity, taints/tolerations, and affinity rules to place the new pod

### Scenario: Failed Deployment → Rollback
```bash
kubectl rollout undo deployment/<deployment_name>
kubectl rollout undo deployment/<deployment_name> --to-revision=<n>
```
Then: isolate the issue, check logs/events, fix root cause **before** redeploying.

### Disaster Recovery (DR) for Kubernetes
- Deploy clusters across **multi-region/multi-AZ**
- Use cross-region replication for stateful backends (e.g., AWS RDS primary + read replicas + automated backups)

### Workload Scheduling
- **Targeted Pod Placement:** `nodeSelector`, Node Affinity/Anti-Affinity, Pod Affinity/Anti-Affinity
- **Why regular pods avoid Master/Control Plane nodes:** control plane nodes carry a default **Taint** (`node-role.kubernetes.io/control-plane:NoSchedule`) — only system pods with matching **Tolerations** (kube-proxy, CoreDNS, CNI DaemonSets) can run there

### Storage Concepts
- **PersistentVolume (PV) & PersistentVolumeClaim (PVC):** request & bind persistent storage dynamically
- **StorageClasses & Reclaim Policies:** dynamic provisioning + policies (`Retain`, `Delete`, `Recycle`)

### EKS vs ECS (Why Teams Prefer EKS)
- Avoids vendor lock-in — uses open-source Kubernetes APIs & tooling (Helm, ArgoCD, Prometheus)
- Greater granular control over networking (CNI), pod scheduling, and multi-tenant architecture

---

## 16. Kubernetes Deployment Strategies

**Simple takeaway:** Don't just recite definitions — tie your answer to **business constraints** (downtime tolerance, budget, storage type). This is what separates hands-on candidates from textbook ones.

### How to Frame Your Answer
Ask yourself (and say out loud in the interview):
- Can the application tolerate downtime?
- What's the infrastructure/compute budget?
- Are there persistent storage constraints (e.g., `ReadWriteOnce` volumes)?

### 1️⃣ RollingUpdate (Default, Zero-Downtime)
Progressively replaces old pods with new ones while keeping the service available.

**Manifest params (`spec.strategy.rollingUpdate`):**
- `maxUnavailable` → max number/% of pods allowed down during update (e.g., `25%` or `1`)
- `maxSurge` → max number of *extra* pods allowed above desired replica count during rollout

> ⚠️ Must be paired with **Readiness Probes** + proper `initialDelaySeconds` so traffic isn't sent to a not-yet-ready pod.

### 2️⃣ Recreate
Terminates **all** old pods before starting new ones → causes brief downtime.

**When to use it:**
- Dev/staging environments with tight compute limits (can't afford surge pods)
- **Critical real scenario — `ReadWriteOnce` (RWO) PVCs:** when a stateful app (like Grafana or a DB) binds to a disk that only **one pod at a time** can mount. In a rolling update, the new pod can't mount the volume until the old one releases it → **deadlock**. Recreate avoids this.

### 3️⃣ Blue/Green Deployment
Two identical environments running side-by-side:
- **Blue** → live environment serving real traffic
- **Green** → idle/testing environment running the new release (validated by QA)

**How it's done in K8s:**
- Two separate Deployments labeled `version: blue` and `version: green`
- Traffic cutover = instantly updating the **Service selector** from `version: blue` → `version: green`

### 4️⃣ Canary Deployment
Gradually rolls out changes to a small % of users (e.g., 5%) while the rest (95%) stay on the stable version — monitor telemetry before full rollout.

**Native K8s implementation:**
- Stable Deployment (e.g., 9 replicas) + Canary Deployment (e.g., 1 replica), both matched to the **same Service selector**
- Gradually shift replica counts until canary = 100%, then delete the old deployment

> 💡 For **true percentage-based** traffic splitting (not tied to pod ratios), mention **Istio Service Mesh** or **Flagger**.

---

## 17. Load Balancing, Ingress, DR Strategy & Node Maintenance

**Simple takeaway:** This section covers cross-cutting "systems design" style questions — load balancing math, DR cost/speed tradeoffs, and the safe way to patch a node.

### Load Balancing & Autoscaling
- **Load Balancing:** distributes incoming traffic across backend servers → optimizes resource use, throughput, and latency
- **Round-Robin:** sequential distribution across the server pool
- **HPA Evaluation Logic (tricky interview Q!):**
  - Q: *If one pod is maxed out and another is idle, does HPA scale immediately?*
  - **A: No.** HPA calculates the **average metric utilization across all target pods** (`currentMetricValue / desiredMetricValue`) and only scales out when the **overall average** crosses the threshold.

### Kubernetes Ingress
- **Ingress** = Layer 7 HTTP/HTTPS routing, managed by an **Ingress Controller** (NGINX, HAProxy, Contour)
- **Routing types:**
  - **Path-based:** `/api` vs `/web`
  - **Host-based:** `app1.domain.com` vs `app2.domain.com`
- **TLS Termination:** centralize SSL/TLS certs at the Ingress level instead of configuring every pod backend

### Disaster Recovery (DR) Strategies & Metrics
- **RTO (Recovery Time Objective):** max acceptable **downtime** before systems are back online
- **RPO (Recovery Point Objective):** max acceptable **data loss window**, measured backward from the incident to the last good backup

**The 4 Primary Cloud DR Strategies:**
| Strategy | Cost | RTO / RPO | Description |
|---|---|---|---|
| **Backup & Restore** | Lowest | Longest | Periodic snapshots restored after an outage |
| **Pilot Light** | Medium | Medium | Core DB replicates live; compute spins up via IaC during failover |
| **Warm Standby** | High | Low | Scaled-down but fully functional replica running 24/7 in a secondary region |
| **Multi-Site Active/Active** | Highest | Near Zero | Traffic routed across multiple live regions simultaneously |

> 💡 **DR Drills:** simulate a region failure intentionally, spin up infra via Terraform, and restore data to validate the whole automation pipeline.

### Kubernetes Node Maintenance Workflow (Zero-Downtime Patching)
```bash
# 1. Cordon — mark node unschedulable (no NEW pods land here)
kubectl cordon <node-name>

# 2. Drain — safely evict existing pods to healthy nodes
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# 3. Perform maintenance (kernel updates, runtime patches, reboot)

# 4. Uncordon — return node to the schedulable pool
kubectl uncordon <node-name>
```

### Security & Pipeline Optimization
- **AWS NACL Limitations:** Network ACLs are **stateless** and operate at the subnet boundary — harder to manage individual allow/deny rules at scale compared to **stateful Security Groups**
- **Optimizing Jenkinsfiles for Multiple Environments:** parameterize builds (dev/uat/prod) + use `when` conditionals instead of maintaining separate scripts per branch
- **Hardening Kubernetes:** enforce strict **RBAC**, network segmentation via **Namespaces + NetworkPolicies**, enable **API server audit logging**

---

## 18. "What Are Your Daily Roles & Responsibilities?" — The Big Interview Question

**Simple takeaway:** This is usually the FIRST question in an interview and it sets the tone for everything after — interviewers use it to sniff out real hands-on experience vs memorized theory. Never say "I never faced any issues" — that's a red flag.

### Why It Matters
Your answer determines what the interviewer drills into next — so structure it well and stay ready to go deep on anything you mention.

### A Structured Daily Routine Framework (use this as a template)
1. **Monitoring & Health Checks** — start the day checking dashboards (CloudWatch, Prometheus, Grafana) to confirm all environments are healthy & catch prod alerts
2. **Incident Triage & RCA** — when issues arise, collaborate with developers to find root cause & remediate fast
3. **CI/CD Pipeline Maintenance & Creation** — manage/troubleshoot/optimize existing pipelines (Jenkins / AWS CodePipeline) and build new ones for new features
4. **Deployments & Infra Coordination** — handle/coordinate deployments to ECS/EKS/EC2

### Evolving Pipeline Architecture (good storytelling for interviews)
- **Before:** separate, redundant Jenkinsfiles per branch
- **Optimized:** one **parameterized pipeline** using declarative syntax + `when` directive (e.g., conditionally trigger staging vs prod deployment/notification stages based on branch/environment)
- **Modernized further:** moved to **AWS-native CI/CD** — CodePipeline triggered by GitHub webhooks, artifacts/images built via CodeBuild using `buildspec.yml`

### Real Troubleshooting Scenarios to Have Ready

**A. Jenkins / Webhook Failures**
| Problem | Cause |
|---|---|
| GitHub webhook not triggering builds | Missing trailing `/` in webhook URL (`.../github-webhook/`) |
| GitHub webhook not triggering builds | Missing/outdated plugins (GitHub Integration, GitHub Branch Source) |
| GitHub webhook not triggering builds | Misconfigured Personal Access Token (PAT) or invalid repo URL |
| Workspace fills up disk | Leftover build artifacts — fix with `cleanWs()` in post-build actions |
| Build fails only on certain agents | Mismatched agent labels or missing tool installs on worker nodes |

**B. AWS CodePipeline & Deployment Failures**
- **#1 root cause:** insufficient **IAM roles** on CodePipeline/CodeBuild (can't fetch source, missing `s3:PutObject` on artifact bucket, missing ECR push permission)
- **Triage:** inspect logs in **CloudWatch Logs**; trace latency/intermittent failures with **AWS X-Ray**

### Demonstrating Deep Hands-On Expertise
- Deploying monitoring stacks (**Prometheus & Grafana**) into K8s clusters via **Helm charts**
- Managing **EKS clusters**, node groups, and using **IAM Roles for Service Accounts (IRSA)** / EC2 instance profiles instead of embedding static credentials

---

## 19. Shell Scripting — Text Processing Commands

**Simple takeaway:** Raw command output (like `top`) is messy — these tools let you *parse and extract exactly what you need* for automation scripts (CPU/memory/disk monitors, log processors, etc.)

### Why This Matters
Automation scripts (e.g., a disk-usage alert script) need **clean, structured data**, not raw dumps — that's what `grep`, `sed`, `awk`, `cut`, `sort` are for.

### `grep` — Pattern Search & Matching
Searches for lines matching a string/regex.

| Flag | Meaning |
|---|---|
| `-c` | Count of matching lines |
| `-i` | Case-insensitive search |
| `-n` | Show line numbers with matches |
| `-v` | Invert match (show lines that DON'T match) |

### `sed` — Stream Editor
Non-interactive text transformation / batch find-replace.

| Operation | Meaning |
|---|---|
| `s/old/new/g` | Substitute/replace pattern (globally) |
| `d` | Delete matching lines |
| `i` | Insert text before matching lines |

### `awk` — Field Scanning & Formatting
Treats each line as a **record**, splits it into **fields** by a delimiter (whitespace by default).
- `$1`, `$2`, `$3` → individual columns
- `$0` → the whole line

### `cut` & `sort`
- `cut -d <delimiter> -f <field_number>` → extract specific columns
- `sort` → alphabetical sort; `-n` → numeric sort; `-r` → reverse order

### `xargs` vs `exec`
| Command | Behavior |
|---|---|
| `exec` | **Replaces** the current shell process with the executed command — no new subshell/PID spawned |
| `xargs` | Takes stdin from a pipe and converts it into **arguments** for another command, e.g. `find . -name "*.log" | xargs rm` |

### `wc` (Word Count)
| Flag | Counts |
|---|---|
| `-l` | Lines |
| `-w` | Words |
| `-c` | Bytes/characters |

### `head` & `tail`
```bash
head -n <lines> file.txt     # view top N lines
tail -f file.log              # follow a growing log file in real-time
```

### `chmod` & Octal Permission Calculation
- Categories: **User (u)**, **Group (g)**, **Others (o)**
- Values: **Read (r) = 4**, **Write (w) = 2**, **Execute (x) = 1**
- Calculate: `4 + 2 + 1 = 7` → full read/write/execute
```bash
chmod 777 <file>   # full rwx permissions for user, group, and others
```

### Bonus: Writing to a File with `cat` (Heredoc)
```bash
cat > filename.txt << 'EOF'
your content here
EOF
```

> 💡 **Practice tip:** Try shell scripting & DevOps labs on platforms like **KodeKloud** without needing local VM setups.

---

## 20. Networking & Security Fundamentals

**Simple takeaway:** Interviewers test whether you understand *how traffic actually flows* — SSL termination, TCP vs UDP, subnetting, NAT, and hybrid connectivity are the most common questions.

### Core Protocols
- **SSL/TLS Termination:** decrypting HTTPS traffic at an intermediary (Ingress Controller, Load Balancer, reverse proxy) → then forwards plain HTTP to backend servers to offload CPU work
- **HTTP vs HTTPS:**
  | | Port | Encryption |
  |---|---|---|
  | HTTP | 80 | None (plain text) |
  | HTTPS | 443 | SSL/TLS encrypted |
- **TCP vs UDP:**
  | | TCP | UDP |
  |---|---|---|
  | Connection | Connection-oriented (3-way handshake) | Connectionless |
  | Guarantees | Delivery, ordering, error-checking | None — optimized for speed |
  | Use case | Web, email, file transfer | Streaming, gaming, low-latency apps |
- **OSI 7-Layer Model:** Application → Presentation → Session → Transport → Network → Data Link → Physical (innermost)
- **IPv4 vs IPv6:** IPv4 = 32-bit (`192.168.0.1`); IPv6 = 128-bit hex notation (solves address exhaustion)

### IP Addressing, Subnetting & NAT
- **Subnetting:** breaking large IP blocks into smaller public/private subnets for routing & security isolation
- **CIDR:** prefix-based IP allocation notation (e.g., `/16`, `/24`)
- **NAT (Network Address Translation):** lets private-subnet instances (no public IP) make **outbound** internet calls (e.g., OS patching) while blocking unsolicited **inbound** traffic

### IP Address Types in Cloud
| Type | Behavior |
|---|---|
| **Private IP** | Non-routable on public internet — internal VPC communication only |
| **Public IP** | Dynamic — released when instance stops |
| **Elastic IP (EIP)** | Static, reserved — persists across reboots (costs money if unattached) |

### AWS Cloud Networking
- **Default VPC:** auto-provisioned per region with default subnets, internet gateway, route tables — for quick launches
- **Default Regional Quotas:** e.g., 5 VPCs/region, 200 subnets/VPC, 50 site-to-site VPN connections/region
- **Hybrid Connectivity:**
  | Option | Description |
  |---|---|
  | **AWS Direct Connect** | Dedicated private physical link, on-prem ↔ AWS, bypasses public internet |
  | **VPN Tunneling** | Encrypted connection over public internet (Client VPN or Site-to-Site IPSec) |
  | **VPC Peering** | 1:1 private connection between two VPCs (non-overlapping CIDRs) |
- **Amazon Route 53:** scalable DNS + traffic routing service

### Encryption & Security Practices
- **Encryption at Rest:** securing stored data (symmetric keys — AWS KMS/AES-256)
- **Encryption in Transit:** securing data on the wire (SSL/TLS handshakes)
- **Governance:** regular vulnerability scans, firewall configs, security audits, automated OS patching
- **Network observability:** tools like **Nagios**

---

## 21. Monitoring & Observability Tools

**Simple takeaway:** Know the tool landscape by category, and be crystal clear on the difference between **monitoring** (the "what") and **observability** (the "why").

### Tool Landscape by Category
| Category | Tools |
|---|---|
| Metrics & Dashboards | **Prometheus** (collection/scraping) + **Grafana** (visualization) |
| Infra & Network Monitoring | Nagios, Zabbix, Datadog |
| Cloud-Native | AWS CloudWatch, Azure Monitor |
| Log Management | **ELK Stack** (Elasticsearch, Logstash, Kibana) |

- **K8s Deployment:** standard method = **Helm charts** (e.g., `kube-prometheus-stack`)

### Monitoring vs Observability
| | Monitoring | Observability |
|---|---|---|
| What it does | Tracks predefined metrics/thresholds (CPU, memory, disk, uptime) | Understands internal system state via telemetry |
| Tells you | **When** something is failing | **Why** a complex distributed system is degrading |

### The Three Pillars of Observability
1. **Metrics** — aggregated numeric measurements over time
2. **Logs** — timestamped event records
3. **Traces** — end-to-end request journeys across microservices

### Multi-Layer Monitoring Strategy (Containerized Workloads)
- **Infrastructure Level:** node health, host memory, disk IOPS, network throughput
- **Application Level:** container CPU/memory limits, error rates, throughput, latency, DB query performance
- **Network/Ingress Level:** ingress traffic, TLS handshake metrics, HTTP response codes

### Alert Routing & Automated Remediation
- **Alert routing:** Amazon SNS (email/SMS/PagerDuty/Slack) or Azure Action Groups
- **Automated self-healing:** trigger AWS Lambda or PowerShell scripts on alert → clear temp storage, restart unhealthy services, auto-remediate common faults
- **Synthetic Monitoring:** scripts that periodically hit HTTP/HTTPS endpoints and alert on non-200 responses or high latency

---

## 22. Jenkins CI/CD Pipeline — Declarative Deep-Dive

**Simple takeaway:** Declarative syntax is the industry standard (not Scripted). Know the top-level directives and be ready to walk through a full pipeline stage-by-stage.

### Scripted vs Declarative
Jenkins supports both, but **Declarative** (structured, opinionated blocks) is the widely adopted industry standard for readability & maintainability.

### Top-Level Declarative Directives
| Directive | Purpose |
|---|---|
| `agent` | Where the build runs — `agent any` or `agent { label 'linux-builder' }` |
| `parameters` | Dynamic build inputs (e.g., dropdown for dev/uat/prod) |
| `environment` | Global/stage-level env vars (AWS account IDs, registry URLs, credentials) |
| `tools` | Auto-installs tools (Maven, JDK, NodeJS) and adds them to `PATH` |
| `options` | Pipeline-level settings, e.g. `timeout(time: 1, unit: 'HOURS')`, discard old builds |

### End-to-End Pipeline Stages
1. **Checkout SCM** — pulls repo code from GitHub/GitLab using credentials & branch
2. **Build / Package** — compiles code, generates artifacts (`.jar`, `.war`, `.ear`) via `mvn clean package` or `npm`
3. **Code Quality / Static Analysis** — integrates **SonarQube** (`mvn sonar:sonar`) using stored Jenkins credentials
4. **Container Image Build & Push:**
   ```bash
   docker build -t <ecr-repo-url>:tag .
   docker push <ecr-repo-url>:tag
   ```
5. **Deployment:**
   - Traditional: deploy `.war` to Tomcat via SSH / Publish Over SSH
   - Container/K8s: `kubectl apply -f manifest.yaml` to EKS
   - **Conditional execution:** use `when` blocks (e.g., deploy to Staging on `develop`, Production on `main`/`master`)
6. **Parallel Stages** — run independent tasks concurrently (e.g., backend tests + UI tests) using the `parallel` block

### Post-Execution Actions (`post` block)
- Conditional logic: `always`, `success`, `failure` (e.g., Slack/email notifications)
- **Workspace cleanliness:** run `cleanWs()` inside `post { always { ... } }` to prevent agents from running out of disk space

---

## 23. Ansible — Hands-On Playbook, Roles & Looping

**Simple takeaway:** Interviewers often ask you to **write a playbook live** (e.g., install Nginx across OS types) — practice this exact pattern.

### Classic Interview Exercise: Multi-OS Nginx Install Playbook
```yaml
- name: Install and configure Nginx
  hosts: all
  become: true
  tasks:
    - name: Install Nginx on Debian/Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == 'Debian'

    - name: Install Nginx on RHEL/CentOS
      yum:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == 'RedHat'

    - name: Ensure Nginx is started and enabled at boot
      service:
        name: nginx
        state: started
        enabled: yes
```
**Key directives:**
- `hosts` → target inventory group/host list
- `become: true` → elevate to root via sudo
- `when: ansible_facts['os_family'] == ...` → conditional check using gathered facts to pick the right package manager (`apt` vs `yum`)

**Run it:**
```bash
ansible-playbook -i <inventory_file> playbook.yml
```

### Custom Ansible Modules
- If built-in modules (`yum`, `copy`, `apt`) can't do what you need, write **custom modules** in Python, Bash, or Ruby
- They take standard key-value args (like `name`, `state`) and return **JSON** back to the Ansible engine

### Ansible Roles Directory Structure
| Folder/File | Purpose |
|---|---|
| `tasks/main.yml` | Primary list of steps the role executes |
| `handlers/main.yml` | Event-driven actions triggered by `notify` (e.g., restart a service) |
| `defaults/main.yml` | Default (low-precedence) role variables |
| `vars/main.yml` | High-precedence static variables |
| `files/` | Static scripts/configs copied directly to target |
| `templates/` | Dynamic Jinja2 templates (`.j2`) |
| `meta/main.yml` | Role metadata, author info, dependencies |

> **Benefit:** improves modularity, code reuse, and long-term maintainability across large automation repos.

### Looping in Playbooks
Avoid duplicate tasks — use `loop` + `item`:
```yaml
- name: Install essential tools
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - vim
    - git
    - curl
```
(Can also loop over numerical ranges or dictionaries.)

### 📍 Recommended DevOps Learning Roadmap (per the creator)
1. Linux Foundations & Shell Scripting
2. Git & GitHub (branching, merging, collaboration)
3. CI/CD Pipelines (Jenkins / GitLab CI)
4. Cloud Core Services (AWS EC2, IAM, EKS, VPC)
5. Containerization & Orchestration (Docker → Kubernetes)
6. Infrastructure as Code & Configuration (Terraform → Ansible)
7. Observability & Monitoring (Prometheus, Grafana, CloudWatch)

---

## 24. AWS Site-to-Site VPN

**Simple takeaway:** Think of it as building a secure "tunnel" between two houses (on-prem and AWS) — know the 4 core components and the 5-step setup.

### Common Interview Scenarios
- Secure, encrypted connectivity: **On-Prem ↔ AWS VPC** (hybrid) or **AWS ↔ Azure/GCP** (multi-cloud)
- How remote workers connect into a private cloud VPC

### Core Concepts (The "Two Houses" Analogy)
| Component | Role |
|---|---|
| **Virtual Private Gateway (VGW)** | The entry/exit gate on the **AWS/VPC side** |
| **Customer Gateway (CGW)** | The entry/exit gate on the **remote/customer side** (on-prem router, firewall, or other cloud edge) |
| **IPSec VPN Tunnels** | The secure, encrypted path over the public internet connecting both gateways |
| **Trust Policies & Authentication** | Pre-shared keys (PSK) + handshakes ensuring only authenticated endpoints exchange traffic |

### Step-by-Step Implementation
1. **Create the Virtual Private Gateway (VGW)** — provision & attach it to the target VPC
2. **Create the Customer Gateway (CGW)** — specify the static public IP of the on-prem firewall/router
3. **Establish the Site-to-Site VPN Connection** — link VGW + CGW; AWS auto-provisions **two redundant IPSec tunnels**; choose routing (Static CIDR or Dynamic BGP)
4. **Enable Route Propagation** — in the VPC route table, enable propagation for the VGW so on-prem CIDRs are auto-injected into routing paths
5. **Download Config & Configure Remote Device** — download the vendor-specific config (Cisco, Fortinet, pfSense, StrongSwan, etc.) and apply IPSec/IKE policies on the customer-end device

---

## 25. Cloud Cost Optimization (FinOps)

**Simple takeaway:** Talking about cost optimization shows you think beyond "just make it work" — it signals senior-level cloud economics awareness. Have real examples ready.

### Why It Matters in Interviews
Demonstrates business impact and separates senior candidates (who understand **FinOps** and proactive resource hygiene) from average ones.

### 1. Automated Non-Prod Scheduling (Dev/QA/UAT)
- Non-prod instances rarely need to run 24/7 → **automate off-hours shutdowns**
- **Mechanism:** AWS EventBridge (Cron) triggers a Lambda function (Python/Boto3) or Azure Automation runbook
  - Script checks instance tags (e.g., `Environment: Dev`), filters running instances, stops them outside business hours/weekends
  - Sends a summary report to resource owners

### 2. Storage & Snapshot Lifecycle Policies
- **Problem:** orphaned snapshots accumulate → drives up storage bills
- **Fix:** automated cleanup via **AWS Data Lifecycle Manager (DLM)** — retain snapshots only up to a threshold (e.g., last 30 days), delete the rest
- Also: identify & delete **unattached EBS volumes** after instance termination

### 3. Budgeting, Governance & Idle Resource Discovery
- **Proactive Budget Alerts:** AWS Budgets (account-level) or Azure Cost Management (resource-group level) → alert at 80%/100% of forecast spend
- **Zombie/Idle Instance Audits:** scan weekly/monthly for forgotten unused instances → notify owner, request justification, flag for deletion
- Send weekend-running-VM reports to management to prevent accidental spend

### 4. Database Optimization & Pruning
- Offload old transactional data to cold storage (e.g., **S3 Glacier**) instead of keeping it on expensive primary SSD volumes
- Performance tuning via **indexing** to optimize IOPS and avoid unnecessary compute upsizing

### 5. Secure Programmatic Implementation
- Use **least-privilege IAM roles** (`sts:AssumeRole`) for automation scripts — retrieve temporary session tokens, query the EC2 API, and perform stop/tagging tasks securely

---

## 26. AWS Storage: EFS (Elastic File System)

**Simple takeaway:** EFS = shared network drive for **multiple Linux instances across AZs** — think "shared folder," not "attached disk."

### What Is EFS?
- Fully managed, **serverless**, POSIX-compliant **shared file storage**
- **Regional service** — unlike EBS (tied to 1 AZ), EFS can mount simultaneously across EC2 instances in **different AZs**
- **Use case:** multiple app servers needing concurrent read/write access to the same dataset, config files, media, or shared packages

### Key Capabilities
- **Protocol:** NFS (NFSv4), TCP port **2049** — mount target security group must allow inbound NFS traffic
- **Elastic Scaling:** grows/shrinks automatically — no pre-provisioning needed
- **Lifecycle Management:** auto-transition infrequently accessed files to EFS-IA/Archive tiers after 30/60/90 days
- **Backup:** automated via AWS Backup
- **Performance Modes:** General Purpose vs Max I/O; Bursting vs Provisioned/Elastic throughput
- **Cross-Region Replication:** async replication for DR and read locality

### Critical Gotchas
- **Linux only** (POSIX/NFS) — for Windows SMB shares, use **Amazon FSx for Windows File Server** instead
- Requires the **`amazon-efs-utils`** package on Linux clients to mount securely (TLS + IAM auth)

### Quick Comparison: EBS vs EFS vs S3
| Service | Storage Type | Scope | Access Model | Best For |
|---|---|---|---|---|
| **EBS** | Block | Single AZ | Single instance (Multi-Attach for io1/io2) | OS root drives, databases |
| **EFS** | File (NFS) | Regional (Multi-AZ) | Concurrent multi-instance (ReadWriteMany) | Shared code, CMS (WordPress), home dirs |
| **S3** | Object (REST API) | Regional/Global | HTTP GET/PUT via internet/API | Artifacts, static web, media, backups |

---

## 27. AWS Storage: S3 (Simple Storage Service)

**Simple takeaway:** S3 = flat object storage accessed over HTTP(S), not a filesystem. Know the storage tiers cold — they're a favorite interview topic.

### What Is S3?
- Managed **object storage** for unstructured data (docs, images, videos, logs, backups)
- Accessed via **REST API** (HTTP/HTTPS GET/PUT) — no OS can be installed/booted on S3 (classic gotcha question!)

### Key Limits & Architecture
- **Object size:** 0 bytes up to **5 TB**
- **Default limit:** 100 buckets/account (soft limit, increasable)
- **Flat Namespace:** key-value store (Object Key → Data Value). No nested buckets — "folders" are just prefixes in the key name
- **Global Uniqueness:** bucket names are unique across **all** AWS accounts worldwide

### Core Features & Resiliency
- **Versioning:** keeps multiple variants of an object → protects against accidental deletes/overwrites
  - ⚠️ **Gotcha:** once enabled, versioning **cannot be disabled** — only suspended
  - Deleting an object adds a **Delete Marker** (removable to restore the original)
- **Cross-Region Replication (CRR):** auto-replicates objects across regions (requires versioning enabled)
- **Lifecycle Management:** auto-transition objects to cheaper tiers (e.g., IA after 30 days, Glacier after 90 days, or delete)
- **Static Website Hosting:** serve HTML/CSS/JS directly from S3 — no web server needed
- **Multipart Upload:** recommended for files > 100 MB — splits into parts uploaded in parallel for speed & resilient retries

### S3 Storage Classes (Memorize This Table!)
| Storage Class | Durability | Latency | Best For |
|---|---|---|---|
| **S3 Standard** | ≥3 AZs | Milliseconds | Frequently accessed, latency-sensitive prod workloads |
| **S3 Standard-IA** | ≥3 AZs | Milliseconds | Infrequent access, needs fast retrieval when needed |
| **S3 One Zone-IA** | 1 AZ only | Milliseconds | Non-critical, recreatable data — 20% cheaper, no multi-AZ redundancy |
| **S3 Glacier Flexible/Deep Archive** | ≥3 AZs | Minutes–hours (up to 12–48h for Deep Archive) | Compliance/long-term archives, cheapest cost |
| **S3 Intelligent-Tiering** | ≥3 AZs | Milliseconds | Unpredictable access patterns — auto cost savings |

---

## 28. AWS Storage: EBS (Elastic Block Store)

**Simple takeaway:** EBS = a virtual hard disk tied to ONE Availability Zone and (usually) ONE instance. Know the AZ-move workflow and the mount CLI steps — both are classic hands-on questions.

### What Is EBS?
- **Block storage** — fixed-size, individually indexed blocks; great for low-latency random read/write
- **Volume Types:**
  | Type | Best For |
  |---|---|
  | **gp2 / gp3** (General Purpose SSD) | Balanced price/performance — boot volumes, dev/test, VDI |
  | **io1 / io2** (Provisioned IOPS SSD) | High-throughput, latency-critical DBs & mission-critical apps |
- **Max size:** 16 TB per volume
- **Persistence:** data survives instance reboot/stop-start. Data on non-root volumes survives termination by default (controlled by `DeleteOnTermination`)

### Key Constraints & Gotchas
- **Single-AZ scope:** a standard EBS volume can't attach to an instance in another AZ/region directly
- **Moving volumes across AZ/Region:**
  1. Take a **snapshot** of the volume
  2. Copy the snapshot to the target region (if cross-region)
  3. Create a **new volume** from the snapshot in the target AZ, then attach it
- **Attachment:** standard EBS = single instance only (io1/io2 support **Multi-Attach** within the same AZ for clustered apps)

### Linux Workflow: Provisioning, Formatting & Mounting
```bash
# 1. List attached block devices
lsblk

# 2. Check if a filesystem already exists on the new device
sudo file -s /dev/xvdf

# 3. Create a filesystem
sudo mkfs -t ext4 /dev/xvdf

# 4. Create a mount directory
sudo mkdir -p /data

# 5. Mount the volume
sudo mount /dev/xvdf /data

# 6. Verify mount & disk space
df -h
```
> 💡 For persistence across reboots, add the volume's UUID to `/etc/fstab`.

### EBS Snapshot Mechanics
- **Point-in-time backup:** captures the exact block state at the moment it's taken
- **Decoupled after creation:** once a new volume is spawned from a snapshot, further writes to the *original* volume do **not** propagate to the new one — they're fully independent
- **Storage:** incremental, backed automatically into **Amazon S3** behind the scenes

---

## 29. EBS Quick-Fire Q&A

**Simple takeaway:** Rapid-fire EBS basics that come up as warm-up questions before the deeper scenarios in Section 28.

| # | Question | Answer |
|---|---|---|
| 1 | Can we attach one EBS to multiple EC2? | Generally **no** — standard EBS attaches to a single instance. Exception: **io1/io2** volumes support **Multi-Attach**, but only within the **same AZ**, for clustered applications. |
| 2 | Can we attach EBS with EC2 in another region? | **No, not directly.** EBS is AZ-scoped. You must **snapshot** the volume, copy the snapshot to the target region, then create a new volume from it there. |
| 3 | Types of EBS? | **gp2/gp3** (General Purpose SSD) and **io1/io2** (Provisioned IOPS SSD) — see Section 28 for use cases. |
| 4 | What is the root volume? | The EBS volume that contains the **OS/boot files** for the instance — deleted by default on termination unless `DeleteOnTermination` is set to false. |
| 5 | Attach EBS to EC2 in a different region — how? | **Snapshot → Copy snapshot to target region → Create volume from snapshot → Attach to instance.** |
| 6 | How to mount a volume to an EC2 server? | `lsblk` → `mkfs -t ext4 /dev/xvdf` → `mkdir /data` → `mount /dev/xvdf /data` → add to `/etc/fstab` for persistence (full steps in Section 28). |
| 7 | Is data lost if EC2 is stopped/rebooted? | **No.** EBS data **persists** through both stop/start and reboot — it's only lost if the volume itself is deleted (or the instance is terminated with `DeleteOnTermination=true` on that volume). |

---

## 30. Common Errors & Solutions — Round 1

**Simple takeaway:** A rapid troubleshooting cheat-sheet across Git, Jenkins, Docker, Kubernetes, and Terraform — plus an important real-world safety warning about job-hunt scams.

### Git: Resolving Merge Conflicts
- **Root cause:** multiple devs editing the same lines, or committing without pulling latest changes first
- **Resolution steps:**
  1. `git status` → see conflicting files
  2. Open files, look for conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`
  3. Manually resolve with the other developer
  4. `git add` the resolved files → `git commit`
  5. Maintain good branching hygiene — sync with main/develop regularly

### Jenkins: Real-World Triage
| Problem | Fix |
|---|---|
| **GitHub webhook not triggering builds** | Verify URL ends correctly with trailing slash (`.../github-webhook/`); ensure GitHub Integration plugin is installed/updated; check agent executor capacity |
| **Pipeline failing repeatedly** | Check **Console Output** in UI + `/var/log/jenkins/jenkins.log` |
| **Disk space exhaustion ("No space left on device")** | Add `cleanWs()` in `post { always { ... } }`; configure **Discard Old Builds** policy |

### Docker: Image & Container Issues
| Problem | Cause | Fix |
|---|---|---|
| **Build hangs / infinite loop** | Bloated base images, uncached remote dependencies | Use lightweight **Alpine** images; optimize Dockerfile layer order for build cache |
| **Container exits/crashes suddenly** | High resource usage or OOM kill | `docker logs <container_id>` to check logs; `docker stats` to check CPU/memory usage |

### Kubernetes: Production Pod Failures
| Problem | Cause / Fix |
|---|---|
| **CrashLoopBackOff** | Triage with `kubectl describe pod <pod_name>` (exit codes/events) + `kubectl logs <pod_name>` |
| → DB connectivity failure | Backend DB unreachable → pod fails health check |
| → **OOMKilled (Exit Code 137)** | Exceeded memory limit → adjust `requests`/`limits` |
| → DNS resolution failure | CoreDNS lookup issue blocking internal service calls |
| **ImagePullBackOff / ErrImagePull** | Misspelled image name/tag, or private registry auth failure (missing/expired `imagePullSecrets`) |

### Terraform: Resource Drift
- **Detection:** run `terraform plan` or `terraform refresh` to detect drift when infra was changed manually outside Terraform
- **Remediation:** bring untracked manual resources into state using `terraform import`
- **State file lock issue:** occurs when a previous `apply`/`plan` didn't release the lock (e.g., DynamoDB lock) — resolve by verifying no other process is running, then use `terraform force-unlock <LOCK_ID>` carefully

### ⚠️ Important Safety Note: Job Hunt Fraud Awareness
Real experience shared: a fraudulent caller posed as a recruiter on a job portal.

**Red flags to watch for:**
- **Artificial urgency** — pressuring you to act immediately, keeping you on the phone with no time to think
- **Demanding sensitive IDs upfront** — asking for PAN/Aadhaar or other government ID *before* even scheduling a technical interview

**Advice:**
- Never share confidential identity documents under pressure
- Verify recruiter authenticity before sharing anything; disconnect if pressured
- Block & report suspicious numbers/emails to protect other job seekers

---

## 31. Common Errors & Solutions — Round 2

**Simple takeaway:** These are the "something in prod just broke" questions — Ingress 502s, cloud outages, webhook failures, load balancer issues, and DB recovery.

### Ingress: Resolving 502 Bad Gateway
1. **DNS Resolution:** verify host/path-based domains in Ingress rules resolve correctly
2. **Backend Pod Health:** check if pods behind the Service are healthy & passing **Readiness Probes** — if all are failing, Ingress returns 502
3. **Kubernetes Endpoints:** `kubectl get endpoints <service-name>` → confirm the Service maps to active Pod IPs
4. **Controller Logs:** inspect Ingress Controller logs (e.g., NGINX) for upstream timeout / connection refused errors

### Cloud Instance/Cluster Outages

**AWS Triage Flow:**
1. **Console Status Checks:** compare **System Status Checks** vs **Instance Status Checks**
2. **CloudWatch & CloudTrail:** check CPU/memory/disk metrics; verify if termination was triggered by API call, auto-scaler, or spot interruption
3. **Network Probing:** `ping`/`curl`/`nc` from within VPC/bastion to test ports 22/443, or use **AWS SSM Session Manager**

**Azure Triage Flow:**
1. Check **Azure Activity Logs** filtered by timestamp for shutdown/deallocation events
2. Review **Azure Monitor** + **VM Boot Diagnostics/Serial Console** for kernel panics or disk exhaustion

### Cloud Webhook Trigger Failures (CodePipeline / Azure DevOps)
- **Public Reachability:** verify webhook listener/API Gateway is reachable from GitHub/GitLab IP ranges
- **IAM & Service Roles:** ensure CodePipeline/webhook handler has enough IAM permission to ingest webhooks & read repo metadata
- **Telemetry:** check GitHub → Webhook → **Recent Deliveries**, correlate with CloudWatch Logs / Azure Monitor

### Load Balancing & Traffic Distribution Issues
- **Target Group Health Checks:** confirm targets register healthy — a misconfigured health check path (expects 200, gets 301/404) pulls instances out of service
- **Security Group Rules:** backend Security Groups must allow inbound traffic from the LB's Security Group on the app port
- **Listener/Routing Rules:** check host/path conditions and priority order in ALB listener rules

### Database Outage Recovery & Data Protection
- **Automated Backups & PITR:** daily snapshots + continuous WAL/transaction log archiving
- **Read Replicas & Multi-AZ Failover:** synchronous standby (e.g., RDS Multi-AZ) for automatic failover with minimal RTO
- **Persistent Volumes:** decouple storage from compute (EBS/EFS/SAN) so compute can be recycled without data loss

---

## 32. Real-World Architecture & Best Practices

**Simple takeaway:** These are "design thinking" questions — how you'd architect solutions to common DevOps pain points, not just tool trivia.

### "Works on My Machine" / Environment Drift
- **Problem:** inconsistent packages, OS deps, runtimes, configs across local/dev/QA/prod
- **Fix:** **Docker containerization** — package app + runtime + libraries into an immutable, portable image ("write once, run anywhere")
- **Selling it to stakeholders:** container density, faster startup (seconds vs minutes), lower compute overhead, consistent staging-to-prod parity
- **Pair with Kubernetes:** declarative rollout, automated scaling, self-healing uniformity everywhere

### Security Vulnerabilities in Automation
- **Centralized Secret Management:** never hardcode credentials — use **AWS Secrets Manager**, **Azure Key Vault**, or **HashiCorp Vault** (multi-cloud/hybrid)
- **Automated Secret Rotation:** rotate credentials on a schedule (e.g., every 30/90 days)
- **Principle of Least Privilege (PoLP):** grant only the minimum permission needed — no broad admin access

### Slow CI/CD Build Times
| Strategy | How |
|---|---|
| **Parallel Execution** | Run unit tests, integration tests, linting simultaneously |
| **Caching** | Cache dependency layers (Maven `.m2`, npm cache, Docker layer caching) |
| **Decoupling Monoliths** | Break into modular microservice builds — only modified services rebuild |
| **Agent Scaling** | Scale out build executors, use dynamic containerized build agents |

### Docker Service vs Docker Stack vs Docker Swarm
| Concept | Definition | Example |
|---|---|---|
| **Docker Service** | Deployment/scaling config for a **single** image/container type across a cluster | A backend scaled to 3 replicas |
| **Docker Stack** | A **collection** of interconnected services + networks, declared together in a Compose file | Full app: web-frontend + backend + mongodb |
| **Docker Swarm** | The **clustering/orchestration engine** itself | The Manager + Worker cluster layer coordinating stacks/services |

### Authenticating Automation Scripts Securely to Cloud
**Azure:**
- Authenticate via **Entra ID Service Principal** (Client ID + Client Secret/Certificate)
- Ensure secrets have expiration + auto-rotation

**AWS:**
- **Never** use root keys or long-lived IAM access keys
- Use **IAM Roles with `sts:AssumeRole`**, or federated access via **IAM Identity Center**
- Script assumes the role → gets short-lived temporary tokens (`AccessKeyId`, `SecretAccessKey`, `SessionToken`) → performs task → tokens auto-expire

---

## 33. Azure DevOps Interview Preparation

**Simple takeaway:** If you're targeting Azure roles, know **AZ-400** as your study path and memorize the common pipeline error → fix mapping, plus HTTP status codes cold.

### Recommended Learning Resource
- **Microsoft AZ-400** (Designing and Implementing Microsoft DevOps Solutions) — even without taking the paid exam, **Microsoft Learn** has free docs + hands-on sandbox labs

### Common Azure DevOps Pipeline Errors
| Error | Root Cause | Fix |
|---|---|---|
| **Resource Not Found** | Referenced resource (variable group, service connection, environment, repo) is missing/deleted/misspelled in YAML | Verify resource exists in portal; cross-check YAML spelling |
| **Access Denied to Azure Resources** | Insufficient RBAC on the Service Connection / Service Principal (Entra ID) | Assign Contributor or custom RBAC role on the target Resource Group/Subscription |
| **Task Failure — Missing Variable** | Required input variable not declared in YAML or missing from Variable Group/Environment library | Map all mandatory params + secret variables (e.g., from linked Key Vault) |
| **Pipeline Timeout** | Long-running steps, network bottlenecks, overloaded self-hosted agents | Optimize scripts, cache dependencies, ensure parallel agent capacity; adjust `timeoutInMinutes: 120` if legitimately long |
| **Specified version does not exist in Artifacts feed** | Wrong feed URL, package name, version, or view permissions | Double-check all four |
| **Failed to publish artifact** | Wrong staging directory path or missing permissions | Verify `$(Build.ArtifactStagingDirectory)`; ensure pipeline identity has Contributor/Publisher access |
| **Database connectivity failure during deployment** | Firewall blocking Azure DevOps agent IPs, or missing credentials | Whitelist Azure service IPs, or use self-hosted agents inside the target VNet |
| **DotNetCoreCLI failed (exit code 1)** | Wrong SDK version | Use `UseDotNet@2` task to install the exact required runtime/SDK before build |

### HTTP Status Codes Quick Reference (Memorize This!)
| Range | Category | Examples |
|---|---|---|
| 100–199 | Informational | 100 Continue |
| 200–299 | Success | 200 OK, 201 Created |
| 300–399 | Redirection | 301 Moved Permanently, 302 Found |
| 400–499 | Client Errors | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 500–599 | Server/Upstream Errors | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

---

## 34. Shell Scripting — 6 Scenario Scripts

**Simple takeaway:** These are the exact "write it live on screen-share" tasks interviewers love. Practice writing these from memory.

### Script 1: Monitor Free Memory & Alert on Low Threshold
```bash
#!/bin/bash
total_mem=$(free -m | awk 'NR==2{print $2}')
free_mem=$(free -m | awk 'NR==2{print $7}')
threshold=10

if [ $free_mem -lt $threshold ]; then
    echo "Warning: Available memory is below threshold!"
fi
```

### Script 2: Automated User Creation with Specific Permissions
```bash
#!/bin/bash
if [ "$(id -u)" -ne 0 ]; then
    echo "Must run as root"
    exit 1
fi

read -p "Enter username: " username
useradd -m -g developers "$username"
chmod 750 /home/"$username"
```
- `chmod 750` → Owner: rwx (7), Group: r-x (5), Others: none (0)

### Script 3: Find & Move Large Files (>1 GB)
```bash
find /source/dir -type f -size +1G | while read file; do
    mv "$file" /destination/dir/
done
```

### Script 4: Package Update & Conditional Reboot
```bash
#!/bin/bash
apt-get update && apt-get upgrade -y

if [ -f /var/run/reboot-required ]; then
    reboot
fi
```

### Script 5: Total Line Count of All `.log` Files
```bash
find /var/log -type f -name "*.log" -exec wc -l {} + 
# OR
find /var/log -type f -name "*.log" -exec cat {} + | awk '{total += 1} END {print total}'
```

### Script 6: Check for Software & Install if Missing
```bash
#!/bin/bash
if ! command -v docker &> /dev/null; then
    echo "Docker not found, installing..."
    apt-get install -y docker.io
    systemctl enable --now docker
else
    echo "Docker is already installed."
fi
```

### Core Patterns Every Candidate Must Master
- **Shebang:** always `#!/bin/bash` as the first line
- **Text parsers:** combine `grep` + `awk` + `sed`
- **Conditionals/Loops:** `if [[ ... ]] elif ... fi`, `while read`, `for in`
- **Exit codes:** check `$?` — `0` = success, non-zero = failure

### Interview Strategy: Live Coding
- **Talk while you type** — narrate your logic and edge cases, never code in silence
- **Problem-solving > perfect syntax** — interviewers value structured thinking and debugging approach over memorized flags

---

## 35. Ansible Integration, Ingress Setup, Error Handling, DR & EKS Upgrade (Q&A)

**Simple takeaway:** A grab-bag of "explain the process" questions — walk through each step-by-step in interviews rather than just naming the tool.

### Q1: How do you integrate Ansible into an application?
- **CI/CD Pipeline Integration:** store playbooks in Git/SCM, trigger them in post-deployment steps (Jenkins/GitLab CI) to configure servers after artifact deployment
- **Dedicated Control Node:** a dedicated Linux server running Ansible executes scheduled/ad-hoc playbooks against inventory
- **Ansible Automation Platform / AWX / Tower:** trigger playbooks via REST API + auth tokens for centralized RBAC, credentials, and audit trails
- (Also possible via a web app calling an API, if the app itself exposes automation hooks)

### Q2: How do you configure Ingress for Kubernetes? (Step-by-step)
1. **Install an Ingress Controller** (NGINX, HAProxy, Traefik) via Helm:
   ```bash
   helm repo add ingress-nginx "https://kubernetes.github.io/ingress-nginx"
   helm repo update
   helm install my-ingress ingress-nginx/ingress-nginx
   ```
2. **Create an Ingress Resource** (`ingress.yaml`):
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: my-ingress
     namespace: default
   spec:
     rules:
       - host: myapp.example.com
         http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: my-app-service
                   port:
                     number: 80
   ```
3. **Expose the Controller:** via Cloud Load Balancer (AWS NLB/ALB) or NodePort for on-prem
4. **Configure DNS:** get the LB IP/CNAME via `kubectl get svc -n ingress-nginx`, create a DNS A/CNAME record pointing to it
5. **Test the Ingress**
6. *(Optional)* Enable SSL/TLS termination

### Q3: Error Handling in Ansible
| Directive | Purpose |
|---|---|
| `ignore_errors: yes` | Don't stop the play on other hosts if this task fails |
| `failed_when: "'fatal' in result.stderr"` | Customize what counts as a "failure" based on output |
| `block` / `rescue` / `always` | try–catch–finally style error handling in YAML |
| `when: ansible_facts['os_family'] == "Debian"` | Conditional task execution based on gathered facts |

```yaml
block:
  - name: Risky task
    command: /bin/false
rescue:
  - name: Handle failure
    debug:
      msg: "Task failed"
always:
  - name: Cleanup
    debug:
      msg: "This always runs"
```

### Q4: Disaster Recovery (Recap)
- **RTO** = max tolerable downtime
- **RPO** = max tolerable data loss window
- **Backup & Restore:** cheapest, longest RTO/RPO
- **Multi-Site Active/Active:** most expensive, fastest recovery (near-zero RTO/RPO)
- **DR Drills:** validate Terraform IaC scripts + automated DB restore procedures quarterly

### Q5: EKS Version Upgrade Strategy
1. **Plan & Backup:** review release notes/deprecated APIs, back up stateful data & manifests (e.g., via Velero)
2. **Upgrade local CLI tools:** `kubectl`, `eksctl`, AWS CLI
3. **Upgrade the Control Plane** (via Console/eksctl/Terraform)
4. **Upgrade Managed Node Groups** — rolling replacement of worker nodes to avoid downtime
5. **Update Add-ons:** VPC CNI, CoreDNS, Kube-Proxy, AWS Load Balancer Controller
6. **Monitor for a few days**, then **cordon & drain** old nodes before terminating them

---

## 36. S3 Tiering, Spot Instances, Lost SSH Keys, Git SSH Auth & Jenkins Security (Q&A)

**Simple takeaway:** A practical grab-bag of AWS + Git + Jenkins troubleshooting questions that come up often as "quick scenario" rounds.

### Q1: Cost-effective S3 storage for unpredictable access patterns?
**Answer: S3 Intelligent-Tiering** — automatically moves objects between frequent/infrequent/archive tiers based on real-time usage, with no retrieval fees or manual overhead.

### Q2: What are Spot Instances?
- Surplus AWS compute capacity at **up to 90% discount** vs On-Demand
- AWS can reclaim capacity anytime, giving a **2-minute warning** (via EventBridge/instance metadata) before termination
- **Best for:** fault-tolerant, stateless batch jobs and CI/CD runners

### Q3: Lost SSH key pair — how do you regain access to an EC2 instance?
**Method 1 (preferred): AWS SSM Session Manager**
- If SSM Agent is installed + instance has IAM role `AmazonSSMManagedInstanceCore` → open a root shell from the console, **no SSH key needed**

**Method 2: EBS Root Volume Swap (Rescue Instance)**
1. Stop the inaccessible instance
2. Detach its root EBS volume
3. Launch a temporary "rescue" instance in the same AZ, attach the detached volume as a secondary drive
4. SSH into the rescue instance, mount the volume, add a new public key to `~/.ssh/authorized_keys` on it
5. Unmount, detach, reattach the volume back to the original instance as root, then start it

### Q4: Avoid entering Git username/password every push
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
- Generates `id_rsa` (private) and `id_rsa.pub` (public)
- Add the public key to GitHub/GitLab under SSH Keys
- Switch remote URL from HTTPS to SSH (`git@github.com:...`)

### Q5: Designing a Jenkins pipeline for multiple environments
- Use a **parameterized pipeline**
- Use separate **stage blocks** per environment (dev/stage/prod), typically combined with `when` conditions

### Q6: Securing sensitive credentials in Jenkins
- Store secrets under **Manage Jenkins → Credentials** (Secret text, Username/password, SSH keys)
- Inject dynamically with `withCredentials`:
  ```groovy
  withCredentials([string(credentialsId: 'my-secret-id', variable: 'SECRET_VAR')]) {
      sh 'echo "Using secret variable..."'
  }
  ```
  → values are automatically masked in console output

### Q7: Preventing a Jenkins pipeline from hanging/getting stuck
- Enforce timeouts globally or per-stage in the `options` block:
  ```groovy
  options {
      timeout(time: 30, unit: 'MINUTES')
  }
  ```

### Q8: Integrating Docker, Kubernetes & AWS into a Jenkins pipeline
| Tool | Integration Method |
|---|---|
| **Docker** | Install Docker Pipeline plugin; use `docker.build()` or shell `docker build -t ...` |
| **Kubernetes** | Use Kubernetes CLI/plugin; `kubectl apply -f manifest.yaml` |
| **AWS** | Use AWS Steps plugin or configure AWS CLI with IAM roles; `aws s3 cp ...` |

---

## 37. Jenkins Multi-Environment & Multi-Branch Pipelines (Deep-Dive)

**Simple takeaway:** This is the "draw me a real pipeline" round — know how to structure environments, agents, and branch discovery cleanly.

### Q1: How does a Jenkins pipeline handle multi-environment deployment?

**Basic approach — define environments + conditional stages:**
```groovy
pipeline {
    agent any
    environment {
        Dev_server = "dev-example.com"
        Prod_server = "prod-example.com"
    }
    stages {
        stage('Build') {
            steps { script { /* build steps */ } }
        }
        stage('Deploy to Development') {
            when { branch 'develop' }
            steps { script { deployToEnvironment(Dev_server) } }
        }
        stage('Deploy to Prod') {
            when { branch 'main' }
            steps { script { deployToEnvironment(Prod_server) } }
        }
    }
}
```

**Handling per-environment secrets:**
```groovy
environment {
    URL = credentials('db_url')
    API_Key = credentials('api_key')
}
```

**Best practices layered on top:**
- **Centralize logic with Shared Libraries** — instead of duplicating deployment code, store reusable Groovy functions in a `vars/` folder and call `deployApp(env: 'prod')`
- **Environment-specific secrets:** separate credential IDs per env (`aws-creds-dev`, `aws-creds-prod`)
- **Manual approval gate for Prod:**
  ```groovy
  stage('Approve Prod Deployment') {
      input {
          message "Deploy to Production?"
          ok "Proceed"
          submitter "devops-leads,admin"
      }
  }
  ```
- Use **parallel** blocks to speed up multi-env deployments

### Q2: Dev/Test/Prod run on different servers/agents — how to reference them?

**Why separate agents?** Isolation, compliance, security boundaries, and preventing test builds from touching production tooling.

**Set `agent none` at top level, then define per-stage:**
```groovy
pipeline {
    agent none
    stages {
        stage('Build & Unit Test') {
            agent { label 'build-agent' }
            steps { /* ... */ }
        }
        stage('Deploy to Dev') {
            agent { label 'dev-deployment-node' }
            steps { /* ... */ }
        }
        stage('Deploy to Prod') {
            agent { label 'prod-dmz-node' }
            steps { /* ... */ }
        }
    }
}
```
- Manage agent labels via **Manage Jenkins → Nodes and Clouds**

### Q3: Multi-Branch Pipeline Setup

**Why use it:** automatic branch discovery, independent parallel builds per branch, isolated PR test feedback.

**Setup steps:**
1. Install the **Multibranch Pipeline** plugin
2. **New Item → Multibranch Pipeline**
3. Under **Branch Sources**, connect the Git repository (GitHub/GitLab/Bitbucket) + credentials
4. **Scan Multibranch Pipeline** — Jenkins auto-discovers branches/PRs containing a `Jenkinsfile` and provisions isolated jobs
5. **Filter branch discovery** by name/wildcard/regex (e.g., only `main`, `develop`, `release/*`, `PR-*`) to avoid job clutter from experimental branches
6. Trigger runs on **webhooks** (push events + PR open/synchronize) to gate merges with automated checks

---

## 38. System Design Scenarios: API Security, Data Loss, Leaked Secrets & Multi-Cloud HA

**Simple takeaway:** These are senior-level "design it" questions. Structure your answer in layers (auth → transport → gateway → observability) rather than a flat list — it reads as more mature in an interview.

### Q1: Securing Public-Facing APIs
- **Authentication & MFA:** API keys, OAuth 2.0 (JWT), or mTLS; require MFA for admin actions
- **Authorization & RBAC:** callers only reach endpoints within their scope
- **Transport Encryption:** enforce HTTPS/TLS everywhere; reject plain HTTP
- **CORS:** explicitly whitelist allowed origins/headers
- **API Gateway & Edge Protection:** deploy a Gateway (AWS API Gateway, Kong, Apigee) for rate limiting/throttling/routing + a **WAF** for OWASP Top 10/SQLi/DDoS protection
- **Observability & Audits:** request logging, distributed tracing, periodic security audits

### Q2: Accidental Production Data Deletion — Incident Response
**Immediate actions:**
1. Halt any active scripts/jobs/pipelines that might still be causing deletion
2. Restore from the most recent backup or **Point-in-Time Recovery (PITR)** snapshot

**Preventive safeguards:**
- Automated cross-AZ/cross-region backups + **S3 Object Versioning with MFA Delete**
- **RDS Deletion Protection** flags; restrictive IAM to prevent bulk drops
- Keep stakeholders informed with root-cause updates and recovery timelines per your DR SLA

### Q3: Leaked Secrets Committed to Git — Response & Prevention
1. **Revoke & rotate immediately** — assume the secret is already compromised; generate new credentials **before** cleaning history
2. **Purge from entire Git history:**
   - Use `git-filter-repo` (or BFG Repo-Cleaner / `git filter-branch`) to strip the secret from every commit/tree/ref
   - Force-push the sanitized history:
     ```bash
     git push origin --force --all
     git push origin --force --tags
     ```
3. **Preventive tooling:**
   - Add sensitive files (`.env`, `credentials.json`) to `.gitignore`
   - Use pre-commit hooks with secret scanners (**git-secrets**, **trufflehog**, **gitleaks**)
   - Inject secrets at runtime via a cloud vault instead of files (Secrets Manager/Key Vault/Vault)

### Q4: Designing a Multi-Cloud High-Availability Deployment
- **Compute:** Docker containers + cloud-agnostic Kubernetes (EKS/AKS/GKE) → no provider lock-in
- **IaC:** Terraform manages resources across providers under unified state/modules
- **Global Traffic Routing:** Route 53 latency/geo-routing or Cloudflare, plus multi-cloud reverse proxies (HAProxy/NGINX)
- **Data Replication:** distributed DB engines or multi-region cross-cloud replication
- **Inter-Cloud Connectivity:** Site-to-Site IPSec VPN or direct interconnects (AWS Direct Connect + Azure ExpressRoute)

### Q5: Monitoring & Logging Across Multi-Cloud Platforms
- **Unified Metrics:** Prometheus scrapes metrics across providers → feeds centralized **Grafana** dashboards
- **Centralized Logs:** stream native logs (CloudWatch, Azure Monitor, GCP Cloud Logging) into **ELK Stack** or **Datadog**
- **Distributed Tracing:** **OpenTelemetry** standard, or provider tools (AWS X-Ray, Azure Application Insights) to trace transactions across cloud boundaries

---

## 39. Introduce Yourself & Daily Roles — The Complete Framework

**Simple takeaway:** These are the *always-asked* opening questions. Your answer here sets the direction for the entire interview — structure it deliberately.

### How to Structure "Introduce Yourself" (First 2–3 Minutes)
**Keep personal details minimal:**
- Skip family background / irrelevant history
- State: name, total IT experience, current organization, notice period status
- Briefly mention highest degree/specialization

**Frame Total vs Relevant DevOps Experience:**
- If you transitioned from another domain (sysadmin, infra support, scripting) — clearly separate **total experience** vs **dedicated DevOps experience**
- Why: it anchors interviewer expectations and steers questions toward your real strengths
- Example narrative: *"Started in system administration and infra automation, discovered an affinity for cloud/containers, and pivoted into core DevOps/SRE work."*

### Structured Tool Walkthrough (Follow This Order — Don't Jump Around)
1. **Source Code & Collaboration** — Git, GitHub/GitLab
2. **CI/CD Pipelines** — Jenkins (Declarative, Shared Libraries) or cloud-native CI (CodePipeline, Azure DevOps)
3. **Containerization & Orchestration** — Docker, Kubernetes (EKS/AKS)
4. **IaC & Config Management** — Terraform, Ansible
5. **Scripting & OS** — Bash/Shell, PowerShell, Python
6. **Primary Cloud Platform** — name your primary (e.g., AWS), acknowledge secondary (e.g., Azure)
7. **Observability & Monitoring** — Prometheus, Grafana, CloudWatch, ELK

### Answering "What Are Your Daily Roles & Responsibilities?"
**Daily Operational Rhythm:**
- **Morning:** review dashboards (Prometheus/Grafana/CloudWatch), triage threshold-breach alerts (CPU/memory spikes, 5xx increases)
- **Sprint cadence:** sprint planning, gathering infra requirements from dev teams, writing/enhancing CI/CD pipeline code

**Key deliverables to highlight:**
- **Automation:** reusable Bash/PowerShell scripts or serverless automation (off-hours shutdowns, volume cleanup, resource audits)
- **Container/Cluster deployments:** deploying to K8s (EKS/AKS), updating Helm charts, managing ingress
- **L3 Production Support:** debugging CrashLoopBackOff/OOMKills, fixing pipeline breaks, managing uptime within SLAs

### Tactical Delivery Tips
- **You drive the conversation:** whatever you emphasize is what gets probed deeper — highlight pipelines → expect Jenkins questions; highlight scripting → expect whiteboard Bash
- **Handling weak areas honestly:** *"I have foundational/lab-level experience with X, though my day-to-day production focus has been Y."* — never claim production expertise you don't have
- **Demonstrate senior ownership:** mention cost-optimization efforts, mentoring juniors, zero-downtime migrations

---

## 40. Terraform in CI/CD & Advanced Concepts

**Simple takeaway:** Know the exact pipeline command sequence, the `count` vs `for_each` distinction, and how to keep secrets out of state/output.

### Implementing Terraform in a CI/CD Pipeline
**Command sequence (strict order):**
```bash
terraform init                  # initialize working dir, backend, providers
terraform validate              # validate syntax/semantics
terraform plan -out=tfplan      # generate execution plan (often tied to PR review)
terraform apply tfplan          # apply after manual/automated approval
```

**Example Jenkinsfile skeleton:**
```groovy
pipeline {
    agent any
    environment {
        TF_VAR_aws_access_key_id     = credentials('aws_access_key')
        TF_VAR_aws_secret_access_key = credentials('aws_secret_access_key')
        TF_VAR_region                = 'us-east-1'
    }
    stages {
        stage('Checkout code') { steps { /* checkout */ } }
        stage('Install Terraform') { steps { script { /* install */ } } }
        stage('Terraform init')  { steps { script { sh 'terraform init' } } }
        stage('Terraform Plan')  { steps { script { sh 'terraform plan -out=tfplan' } } }
        stage('Terraform Apply') { steps { script { sh 'terraform apply -auto-approve tfplan' } } }
    }
}
```

**Remote State Management:** store `terraform.tfstate` remotely (S3 + DynamoDB locking, or Terraform Cloud/HCP Terraform) for team concurrency & to avoid corruption.

### `count` vs `for_each`
| Feature | `count` | `for_each` |
|---|---|---|
| Use case | Multiple identical/indexed resources | Distinct resources from a map/set of strings |
| Addressing | By index — `resource.name[0]`, `[1]` | By explicit key — `resource.name["key"]` |
| Deletion Impact | Deleting a middle item **re-indexes everything after it** → unnecessary destroy/recreate | Only the targeted resource is destroyed/updated — others untouched |

### Multi-Cloud Infrastructure with Terraform
- **Provider blocks:** separate blocks per cloud — `provider "aws"`, `provider "azurerm"`, `provider "google"`
- **Input variables:** parameterize credentials/regions/env flags via `variables.tf` + `.tfvars` — avoid hardcoding
- **Reusable modules:** abstract common components (networking, bastions) into modules usable across environments/providers

### Terraform State Drift
- **Definition:** real-world infra differs from what's recorded in `terraform.tfstate` (usually from manual console changes)
- **Detection & Remediation:**
  1. `terraform plan` → compare real infra vs state/config
  2. `terraform refresh` → reconcile state with actual cloud environment
  3. `terraform import` → bring untracked manual resources into state
  4. Update HCL to match legit changes, or re-apply to overwrite unwanted manual edits
- **Prevention:** restrict direct Console write access (enforce IaC-only changes); schedule daily automated `terraform plan` drift checks

### Securing Sensitive Data in Terraform
```hcl
output "db_password" {
  value     = aws_db_instance.default.password
  sensitive = true    # suppresses cleartext in CLI/console logs
}
```
- **Dynamic vault integration:** pull credentials from HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault instead of plain `.tfvars`
- **Environment variables:** inject secrets via `TF_VAR_<name>` or CI-managed runner variables

### Interview Tip: Concepts Over Syntax
Interviewers care more about **dependency chains** than exact parameter names:
> A `aws_vpc` outputs a `vpc_id` → passed into `aws_subnet` → which links to `aws_route_table` via `aws_route_table_association`.

Focus on resource **references** (`aws_vpc.main.id`) rather than memorizing every argument.

---

## 41. Service Mesh

**Simple takeaway:** A service mesh handles *service-to-service* (east-west) traffic so developers don't have to hardcode retries, mTLS, and tracing into every microservice.

### What Is a Service Mesh & Why It's Needed
- A dedicated **Layer 7** infrastructure layer managing **service-to-service (east-west)** communication in microservices
- **Problem it solves:** without it, devs must hardcode retries, timeouts, circuit breaking, mTLS, and tracing into application code across every language — a service mesh abstracts all of this away
- **Popular implementations:** **Istio** (full-featured, most widely used), **Linkerd** (lightweight, Rust-based), **HashiCorp Consul**

### Core Architecture — Two Planes
```
┌─────────────────────────────────────────────┐
│              CONTROL PLANE                    │
│  Istiod: Service Discovery, CA (Certs), Policy│
└─────────────────────────────────────────────┘
              │ pushes config, TLS certs, routes
              ▼
┌─────────────────────────────────────────────┐
│               DATA PLANE                       │
│  ┌───────────┐          ┌───────────┐          │
│  │  Pod A    │          │  Pod B    │          │
│  │ [App]     │          │ [App]     │          │
│  │ [Sidecar] │◄──mTLS──►│ [Sidecar] │          │
│  │  (Envoy)  │          │  (Envoy)  │          │
│  └───────────┘          └───────────┘          │
└─────────────────────────────────────────────┘
```
- **Control Plane (istiod):** the "brain" — translates routing rules, RBAC policies, service discovery into proxy configs and distributes them
- **Data Plane (Envoy sidecars):** high-performance proxies injected into every pod that intercept/inspect all traffic without touching container binaries

### Key Problems Solved
- **Dynamic Service Discovery:** auto-tracks healthy pod IPs as pods scale up/down
- **Resilience & Traffic Control:** built-in retries, timeouts, circuit breakers, rate limiting
- **mTLS & Zero Trust Security:** auto-provisions X.509 certs to every sidecar, encrypting all east-west traffic + service identity/RBAC
- **Deep Observability:** standardized L7 metrics (req/sec, latency, HTTP errors), traces, access logs → exported to Prometheus/Grafana/Jaeger/Kiali with **zero code changes**

### Advanced Deployment Strategies Enabled
- **Canary Releases:** `VirtualService` + `DestinationRule` manifests route a % of traffic (e.g., 90% v1 / 10% v2) by weight or header
- **Blue/Green:** instantly flip 100% of traffic to the new "green" release once validated

### Service Mesh vs Traditional Load Balancing
| Dimension | Traditional LB (ALB/NGINX/HAProxy) | Service Mesh (Istio/Envoy) |
|---|---|---|
| Traffic Scope | North-South (client → cluster boundary) | East-West (service-to-service internal) |
| Layer | L4 (TCP/UDP) or L7 entry routing | L7 application-aware routing at **every hop** |
| Security | TLS termination at the edge; internal traffic often unencrypted | Automatic end-to-end **mTLS** between every pod pair |
| Resilience | Basic round-robin/least-connection | Fine-grained % canary splits, header matching, fault injection, circuit breakers |

### Configuring a Service Mesh in Kubernetes (Istio Example)
1. **Install control plane:** via Helm or `istioctl install --set profile=default`
2. **Enable auto-injection on a namespace:**
   ```bash
   kubectl label namespace default istio-injection=enabled
   ```
3. **Deploy microservices:** a mutating webhook auto-injects the `istio-proxy` (Envoy) sidecar alongside your app container
4. **Apply traffic/security manifests:** `VirtualService`, `DestinationRule`, `PeerAuthentication`, `AuthorizationPolicy`

---

## 42. Real-World Scenarios: API Gateway vs LB, VPC Peering, ASG & Monitoring

**Simple takeaway:** These are "explain the tradeoffs" questions that test practical cloud networking judgment, not memorization.

### API Gateway vs Load Balancer (ALB/NLB)
| | API Gateway | Load Balancer (ALB/NLB) |
|---|---|---|
| **Purpose** | Purpose-built for HTTP/HTTPS APIs — API lifecycle management, rate limiting, request/response transforms, auth (API keys, Cognito, OAuth) | Pure high-throughput traffic distribution |
| **Layer** | Layer 7 | ALB = L7 (HTTP/HTTPS); NLB = L4 (ultra-low-latency TCP/UDP/TLS) |
| **Billing** | True serverless pay-per-request — **$0 when idle** | Constant hourly base charge + Capacity Units (LCUs/NLCUs), regardless of traffic |

### Multi-VPC & Hybrid Connectivity Patterns
| Pattern | Description |
|---|---|
| **VPC Peering** | 1:1 point-to-point connection, non-overlapping CIDRs. Workflow: VPC A sends request → B accepts → update route tables both sides with the peering connection ID (`pcx-xxxx`). **Non-transitive** — if A↔B and B↔C, A **cannot** reach C |
| **AWS Transit Gateway (TGW)** | Hub-and-spoke central router for interconnecting **dozens/hundreds** of VPCs — avoids a messy peering mesh |
| **Site-to-Site IPSec VPN** | Encrypted connectivity over public internet between on-prem gateway and AWS VGW |
| **AWS Direct Connect (DX)** | Dedicated private fiber link — bypasses public internet for consistent latency/throughput |

### Load Balancers & Auto Scaling Groups (ASG) Working Together
- **Target Groups & Health Probing:** LB only routes to **healthy** targets; stops routing immediately on failed health checks
- **ASG Capacity Scaling:** governed by **Min / Desired / Max** thresholds
  - Dynamic scaling policies react to metrics (Target Tracking on CPU, or ALB `RequestCountPerTarget`) → spin up & auto-register new instances during spikes, or drain & terminate when idle
- **Kubernetes/EKS equivalent:** **Cluster Autoscaler** / **Karpenter** provisions EC2 worker nodes as pods scale up

### Application Monitoring vs Log Monitoring
| | Application/Health Monitoring | Log Monitoring & Alerting |
|---|---|---|
| Focus | Operational health & telemetry (uptime pings, response latency, CPU/memory thresholds) | Ingesting/parsing raw logs (app logs, access logs, audit trails) |
| Method | Threshold-based metrics | Pattern-matching filters (`ERROR`, `FATAL`, HTTP 5xx, stack traces) → fires alerts/runbooks **before** full degradation |

---

## 43. Git Advanced: Merge vs Rebase, Submodules, Hooks, Branching & Versioning

**Simple takeaway:** These are the "prove you actually understand Git internals" questions that go beyond basic commands.

### `git merge` vs `git rebase`
| | `git merge` | `git rebase` |
|---|---|---|
| **Commit history** | Non-destructive — preserves exact timeline, creates an explicit merge commit | Rewrites history — moves the whole feature branch to start at the tip of base branch → **linear** history |
| **Traceability** | Individual branch context stays clear | Commits can be reordered/squashed → clean log |
| **Best used for** | Public/shared branches with multiple active contributors | Cleaning up **local** feature branches before opening a PR |

### Git Submodules
- **What:** embeds one Git repo as a subdirectory inside another, while keeping **independent commit histories**
- **Use case:** managing shared libraries/common dependencies across multiple projects without duplicating source code

### Git Hooks (Pre-Commit / Pre-Push) & Security
- Custom scripts in `.git/hooks` that auto-run on events (`pre-commit`, `pre-push`, `post-merge`)
- **Security/quality gates:**
  - `pre-commit` hooks block commits if secret scanners detect exposed tokens/keys
  - Enforce linting, static analysis, unit tests before allowing a push

### Enterprise Branching Strategy (GitFlow)
| Branch | Purpose |
|---|---|
| `main`/`master` | Production-ready, stable code only |
| `develop` | Central integration branch for ongoing dev |
| `feature/*` | Branched off `develop` for a specific feature → merged back via PR |
| `release/*` | Branched off `develop` when prepping a release — final fixes before merging to `main` + `develop` |
| `hotfix/*` | Branched directly from `main` for urgent prod fixes → back-ported to both `main` and `develop` |

### Versioning with Git Tags & GitHub Releases
- **Git Tags:** point to a specific commit marking a milestone (e.g., Semantic Versioning `v1.2.0`) — allows downloading a zipped snapshot at that point
- **GitHub Releases:** built on top of tags — bundle release notes, changelogs, and binary build artifacts for distribution/rollback

---

## 44. Managerial Round — Behavioral Answers

**Simple takeaway:** Managerial rounds test communication, self-awareness, and how you handle people problems — not technical depth. Structure your answers around **proactive communication** and **root-cause-first** thinking.

### Opening & Work Profile (Managerial Lens)
- Frame intro around: total vs relevant experience, primary tools/cloud, project domains
- **"Day in the life" answer:** cross-team collaboration, morning monitoring-alert triage, writing automation (Bash/Python/Terraform), building/enhancing CI/CD pipelines, handling platform migrations
- **Be ready to explain:** are your automation scripts ad-hoc/manual, or fully automated via triggers (cron, EventBridge/Lambda, pipeline webhooks)?

### Scenario: Lack of Recognition Despite Strong Performance
- **Core principle:** proactive communication — staying silent leads to burnout and degraded performance
- **Escalation path:**
  1. **Direct manager/lead 1-on-1** — present objective metrics/deliverables/impact, ask for clear feedback
  2. **Product/project leadership** — if ignored, escalate transparently to understand alignment/roadmap
- **Mutual professionalism:** high-performing teams need psychological safety, empathy, and clear advancement paths

### Scenario: Handling an Underperforming Team Member
1. **First — peer-level root cause discovery (never escalate prematurely):**
   - Is it a technical/onboarding gap?
   - Lack of tooling familiarity?
   - External/personal constraints?
   - Offer pair programming, knowledge-transfer sessions, shared docs
2. **Second — structured escalation:** only if the issue is behavioral/negligent/unprofessional and peer mentoring can't fix it → escalate objectively to management/HR with documented examples

### Answering "Why Switch?" & "Why This Company?"
- **Research first:** check the company's Careers/About Us pages for cultural pillars (customer obsession, ownership, continuous learning, blameless culture)
- **Frame positively:** align your growth goals (scaling architectures, cloud-native adoption, leading automation) with the company's mission — never frame it as frustration with your current employer

---

## 45. Building Your DevOps Resume/CV

**Simple takeaway:** ATS-friendly structure + bold keyword anchoring + absolute honesty (you must be able to defend everything you write).

### Template & Layout
- Clean, minimalist single-page template (2 pages acceptable for 5+ years / multi-client experience)
- **Header:** Full name + role title (e.g., "DevOps Engineer"), location, phone, professional email, clickable LinkedIn URL

### Section-by-Section Structure
**A. Professional Summary**
- 2–3 line elevator pitch: total experience, core focus areas (cloud infra, CI/CD automation, orchestration), business value delivered
- Polish wording with AI tools (ChatGPT) but keep accomplishments **accurate**

**B. Technical Skills (Categorized for ATS visibility)**
| Category | Examples |
|---|---|
| Cloud Platforms | AWS, Microsoft Azure |
| IaC & Config Mgmt | Terraform, Ansible |
| Containerization & Orchestration | Docker, Kubernetes |
| CI/CD & VCS | Jenkins (Declarative), Git, GitHub, GitLab, Jira |
| Scripting & Languages | Bash/Shell, PowerShell, Java, Groovy, Advanced SQL |
| Databases & Tools | PostgreSQL, Maven |

**C. Professional Experience**
- Reverse chronological: Company, Job Title, Location, Duration
- Quantifiable responsibilities + technical ownership under each role

**D. Achievements, Education & Certifications**
- **Achievements:** Technical Excellence Awards, client appreciation letters
- **Certifications:** AWS Certified Solutions Architect, CKA, Azure AZ-104/AZ-400 — dedicate a visible section
- **Education:** degree, specialization, university

### Critical Interview Traps & Formatting Strategy
- **Bold key technical terms & metrics** (e.g., **Cost Optimization**, **DR Drill Activities**, **Terraform**, **Kubernetes EKS Migration**) — recruiters scan in seconds; bold anchors their eyes to your strengths
- **The Rule of Defensibility (cardinal rule):** never copy-paste a bullet point or tool you can't defend under technical grilling. If you list "cost optimization scripts" or "DR drill orchestration," know the **exact commands, failure scenarios, and architecture** behind them

---

## 46. Linux Interview Questions — Complete Answers

**Simple takeaway:** A rapid-fire round of classic Linux fundamentals — practice saying these out loud, not just reading them.

### Q1: Which Linux flavors have you worked on?
- **Debian family:** Ubuntu, Debian (`apt`/`dpkg`)
- **Red Hat family:** RHEL, CentOS, Rocky Linux, Fedora (`dnf`/`yum`)
- **Others:** Alpine (lightweight, common in Docker), Amazon Linux, Kali Linux, Gentoo

### Q2: Hard Link vs Soft (Symbolic) Link
| | Hard Link (`ln`) | Soft Link (`ln -s`) |
|---|---|---|
| Inode number | **Same** as original | **Different** — points to a pathname |
| Data blocks | Points directly to original data — no extra space | Small pointer file storing the target path |
| If original deleted | Data still accessible via the hard link (until link count = 0) | Link **breaks** (dangling link) |
| Filesystem/directory limits | Cannot cross filesystems, cannot link directories | Can cross filesystems, can link directories |

### Q3: Linux Directory Structure (FHS)
| Directory | Purpose |
|---|---|
| `/bin`, `/sbin` | Essential user & admin binaries (`cp`, `ls`, `shutdown`, `fdisk`) |
| `/etc` | Host-specific configuration files (passwords, networking) |
| `/dev` | Device nodes — virtual & physical devices (`/dev/sda`, `/dev/null`) |
| `/usr` | Secondary binaries/libraries — `/usr/bin` (user cmds), `/usr/sbin` (admin cmds), `/usr/share` (docs) |
| `/home` | Personal directories per user (`/home/ram`, `/home/shyam`) |
| `/lib` | Shared libraries and executable binaries |
| `/tmp` | Temporary files — wiped on reboot |
| `/var` | Runtime data — logs, caches, spool (`/var/log`) — **not** auto-deleted (critical history) |
| `/mnt` | Used by admins to mount new filesystems |
| `/proc`, `/sys` | Virtual pseudo-filesystems — process/kernel metadata |
| `/media` | Removable disks (USB, SD card) |
| `/srv` | Services provided by the system (e.g., HTTP server data) |
| `/opt` | Third-party add-on software |

### Q4: Home Directory of Root User
`/root` — kept directly on the root partition (not under `/home`) so it's still accessible during single-user/maintenance mode or if `/home` is a separate unmounted/corrupted partition.

### Q5: Check if a Specific Port Is Enabled/Listening
```bash
ss -tulpn | grep :<port>          # modern standard
netstat -tulpn | grep :<port>     # traditional
lsof -i :<port>                   # process mapping
nmap -p <port> localhost          # port scan
```

### Q6: What Are Daemon Services?
Background, non-interactive processes that run continuously (often ending in `d`) — start at boot or on-demand to handle requests, network listeners, or scheduled jobs. Examples: `sshd`, `systemd`, `crond`, `httpd`. Managed via `systemctl`/systemd unit files.

### Q7: Granting Root Privileges to a User
```bash
# Option A: add to admin group
usermod -aG wheel <username>     # RHEL/CentOS
usermod -aG sudo <username>      # Debian/Ubuntu

# Option B: edit sudoers safely
sudo visudo
# then add:
username ALL=(ALL:ALL) ALL
```
> ⚠️ Always use `visudo` — never edit `/etc/sudoers` directly (syntax errors can lock you out of root).

### Q8: Checking Currently Running Processes
```bash
ps aux              # snapshot, all processes
ps -ef               # snapshot, alt format
top / htop           # real-time interactive view
pgrep -a <name>       # search by process name
pidof <name>          # get PID by name
```

### Q9: File Ownership & Permissions
- Every file has an **Owner (u)**, a **Group (g)**, applied rules for **Others (o)**
- **Permission values:** Read = 4, Write = 2, Execute = 1
- Check with `ls -l` (permissions + ownership) or `ls -la` (includes hidden files)
```bash
sudo chown bob demo.sh      # change owner to 'bob'
chmod u+x filename          # symbolic: add execute for owner
chmod 755 file.sh           # numeric: Owner=rwx(7), Group=r-x(5), Others=r-x(5)
chmod 777 devops.txt        # full rwx for everyone
```

### Q10: Setting Up Passwordless SSH Between Two Linux Systems
```bash
# 1. Generate key pair on source
ssh-keygen -t rsa -b 4096
# or the more modern:
ssh-keygen -t ed25519

# 2. Copy public key to remote host
ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote-host

# 3. (Optional hardening) On the remote host — /etc/ssh/sshd_config
PasswordAuthentication no

# 4. Restart SSH daemon
sudo systemctl restart sshd
```
- Correct permissions matter: `chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/authorized_keys`

### Q11: Managing System Updates & Patches
```bash
# Debian/Ubuntu
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
cat /var/log/apt/history.log     # check update logs
sudo reboot                       # if required

# RHEL/CentOS
sudo yum update -y
# or
sudo dnf upgrade -y
dnf update --security             # security-only updates
```
**Enterprise-scale tools:** AWS Systems Manager (SSM) Patch Manager, Ansible playbooks for fleet-wide patch baselines.

### Q12: Regaining Access to a Server with a Lost SSH Key
1. **Stop** the instance (don't terminate)
2. **Detach** its root EBS volume
3. **Attach** the volume to a healthy "rescue" instance as a secondary/data volume
4. **Mount** it, navigate to `~/.ssh/authorized_keys` under the mounted user directory, and **add a new public key**
5. **Unmount → detach → reattach** the volume back to the original instance as root
6. **Start** the original instance and log in with the new private key

> 💡 **Faster alternative:** if the **AWS SSM Agent** is already installed with the right IAM role (`AmazonSSMManagedInstanceCore`), just use **SSM Session Manager** to open a shell directly — no key swap needed.

---

## 47. Kubernetes Architecture & Request Lifecycle (Deep-Dive)

**Simple takeaway:** This is the #1 most-asked K8s question — walk through exactly what happens when you run `kubectl apply -f manifest.yaml`, component by component.

### Control Plane (Master Node) Components
| Component | Role |
|---|---|
| **kube-apiserver** | Central entry point for the K8s REST API; authenticates/authorizes/validates requests; manages pod/service/RC state |
| **etcd** | Consistent, highly-available key-value store holding the entire cluster state & config |
| **kube-scheduler** | Watches for unscheduled pods, picks the best node based on resource requirements, affinity, taints/tolerations |
| **kube-controller-manager** | Runs reconciliation loops (Node Controller, ReplicaSet Controller, Deployment Controller, Job Controller) to drive actual state → desired state |

### Worker Node Components
| Component | Role |
|---|---|
| **kubelet** | Node agent — ensures containers described in PodSpecs are running & healthy; reports status back to API server |
| **kube-proxy** | Network proxy — maintains network rules (iptables/IPVS) for Service-to-Pod routing & load balancing |
| **Container Runtime** | Pulls images and runs containers (containerd, CRI-O) |

### Other Key Building Blocks
| Concept | Purpose |
|---|---|
| **Pod** | Smallest deployable unit — one or more tightly-coupled containers sharing network namespace |
| **Service** | Stable abstraction/policy for accessing a logical set of Pods, even as Pods come & go |
| **Namespace** | Divides cluster resources between teams/users for isolation |
| **Deployment** | Declarative updates to Pods/ReplicaSets; ensures desired replica count, supports auto-scaling |
| **ReplicaSet** | Ensures a specified number of replica pods are always running |
| **StatefulSet** | Like ReplicaSet but for apps needing stable storage & ordered deployment (e.g., databases) |

### Step-by-Step: What Happens on `kubectl apply -f manifest.yaml`
```
1. kubectl sends an authenticated REST call to kube-apiserver
2. kube-apiserver:
   - Authenticates (token/certificate)
   - Authorizes via RBAC
   - Validates via admission controllers
   - Persists desired state into etcd
3. kube-scheduler:
   - Detects the new unscheduled pod
   - Filters nodes by resources/affinity/taints
   - Binds the pod to a target worker node
4. On the target node, kubelet:
   - Receives the pod spec
   - Commands the Container Runtime to pull the image & start containers
   - Reports status back to kube-apiserver
5. kube-proxy:
   - Updates local routing rules so cluster Services can reach the new Pod IP
```

### Pod Disruption Budget (PDB)
- **Definition:** a policy specifying the **minimum pods that must stay available** during voluntary disruptions (node drains, EKS upgrades, cluster autoscaler downscales, manual maintenance)
- **Why it matters:** prevents accidental outages during rolling infra updates
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2   # or use: maxUnavailable: 1
  selector:
    matchLabels:
      app: web-app
```

### Role of `kube-proxy` (Recap)
- Runs on every node, maintains network rules (iptables/IPVS)
- Translates a **ClusterIP** (virtual Service IP) into actual dynamic backend **Pod IPs** — enabling Layer 4 load balancing inside the cluster

### Role of `kube-controller-manager` (Recap)
- Runs the core reconciliation loops that continuously watch cluster state via the API server and drive it toward desired state
- Manages: **Deployment Controller**, **ReplicaSet Controller**, **Node Controller** (handles unreachable nodes), **EndpointSlice Controller**, **Namespace Controller**

---

## 48. Kubernetes Security Hardening (Deep-Dive)

**Simple takeaway:** Structure your security answer in layers — Identity, Network, Runtime, and Secrets — rather than naming tools randomly.

| Layer | Practices |
|---|---|
| **Identity & Access** | **RBAC** enforcing least privilege for Users & ServiceAccounts |
| **API Server Hardening** | Strong auth (OIDC, MFA, TLS certs); restrict public API server access via CIDR whitelisting or private endpoints |
| **Network** | **NetworkPolicies** to eliminate the default flat network model — restrict pod-to-pod ingress/egress |
| **Runtime & Pod Security** | Pod Security Standards (`restricted` profile), non-root container users, read-only root filesystems, dropping Linux capabilities |
| **Secrets Management** | Avoid plaintext/Base64 K8s Secrets — use **External Secrets Operator** or **Sealed Secrets** integrated with HashiCorp Vault / AWS Secrets Manager / Azure Key Vault |

---

## 49. Python Scripting for DevOps

**Simple takeaway:** Two Python skills interviewers commonly test: parsing Excel data with `pandas`, and hitting REST APIs with `requests`.

### Processing Excel Workbooks (`pandas` / `openpyxl`)
**Libraries:**
- `pandas` → `pd.read_excel()`, dataframe filtering, `.to_excel()`
- `openpyxl` → lower-level cell/sheet styling and modification

**Practical DevOps uses:** parsing server inventory sheets, generating compliance audit reports, bulk-updating infra cost tracking data

```python
import pandas as pd

# Read Excel sheet
df = pd.read_excel("servers_inventory.xlsx", sheet_name="Production")

# Filter records
unhealthy_nodes = df[df["status"] != "Ready"]

# Save processed data back to a new file
unhealthy_nodes.to_excel("unhealthy_nodes_report.xlsx", index=False)
```

### Handling REST API Requests (`requests` library)
```bash
pip install requests
```

```python
import requests

headers = {
    "Authorization": "Bearer YOUR_API_TOKEN",
    "Content-Type": "application/json"
}

# GET request
response = requests.get("https://api.example.com/v1/clusters", headers=headers, timeout=10)
if response.status_code == 200:
    data = response.json()

# POST request
payload = {"instance_type": "t3.medium", "count": 2}
post_resp = requests.post("https://api.example.com/v1/scale", json=payload, headers=headers, timeout=10)
post_resp.raise_for_status()   # raises an exception on 4xx/5xx errors
```

**Key concepts to mention in an interview:**
- Passing **headers** (`Authorization: Bearer <token>`, `Content-Type: application/json`)
- Payload serialization (`json=payload` handles this automatically, or manually via `json.dumps()`)
- Setting **timeouts** (`timeout=10`) to avoid hanging requests
- Validating status codes with `response.raise_for_status()`

---

## 50. Quick-Fire Practice Questions: Git & Jenkins

**Simple takeaway:** A rapid self-test list — try answering these from memory using what you've already learned in Sections 3, 22, 30, and 37 before checking.

1. Command to list all remote repository branches → `git branch -r` (or `git branch -a` for local + remote)
2. Difference between a branch and a tag → *(see Section 3 & 43 — branch is mutable, tag is immutable)*
3. Difference between `git fetch` and `git clone` → *(see Section 3 — clone creates a new local repo, fetch updates refs without touching your working tree)*
4. Command to revert changes from staging area back to working area → `git restore --staged <file>` (older Git: `git reset HEAD <file>`)
5. What does `git revert` do → *(see Section 3 — creates a new commit that undoes a previous commit safely)*
6. How to change Jenkins' default port number → edit `/etc/default/jenkins` (or `JENKINS_PORT` in the service config / `--httpPort=<port>` startup argument) and restart the service
7. What are sticky sessions → a load-balancing technique that routes a client's requests to the **same backend server** for the duration of their session (useful for stateful apps)
8. Difference between Poll SCM and Build Periodically → **Poll SCM** checks the repo periodically and only builds **if there's a new commit**; **Build Periodically** triggers a build on a fixed schedule **regardless** of code changes
9. How to maintain backups in Jenkins → back up the `/var/lib/jenkins` home directory (job configs, plugins, credentials) — manually or via a backup plugin — on a scheduled basis
10. What are Views in Jenkins & how to create them → Views group related jobs together for easier navigation (e.g., by team or pipeline stage); create via **+ New View** on the Jenkins dashboard, name it, and select jobs/filters to include

---

## 51. System Design for DevOps/SRE Interviews

**Simple takeaway:** This is the senior-level round — "design a system that survives a flash sale." Structure your answer top-down: architecture choice → routing → scaling → traffic-spike prep.

### Core Scenario to Prepare For
*"How do you design a scalable, fault-tolerant infrastructure that can absorb massive, sudden traffic spikes (flash sales, festive events, marketing campaigns)?"*

### Monolithic vs Microservices
| | Monolithic | Microservices |
|---|---|---|
| **Advantages** | Simpler to build/deploy initially; resource-efficient at small scale (no inter-service network overhead) | Independent deployment/scaling per component; failure domain isolation; tech-stack flexibility |
| **Disadvantages** | Hard to scale individual parts; tight coupling → one module's failure can crash everything; slow build/deploy | More operational/networking complexity — needs distributed tracing, service discovery, service mesh |

### Edge Routing & Entry Points
| Layer | Role |
|---|---|
| **Amazon Route 53** (DNS) | Maps client domains to endpoints using latency-based, weighted, or failover routing policies |
| **ALB (Layer 7)** | Inspects HTTP/HTTPS headers/hostnames/paths; TLS verification; routes to Auto Scaling Target Groups |
| **NLB (Layer 4)** | Ultra-high throughput TCP/UDP traffic, microsecond latency, static IPs |
| **API Gateway** | Best for serverless workloads, RESTful APIs, payload transforms, per-consumer rate-limiting |

> 💡 **Rule of thumb:** API Gateway → serverless/API-centric; ALB → containerized microservices with long-lived connections & high sustained HTTP traffic.

### Scaling Strategies
| | Vertical (Scale-Up) | Horizontal (Scale-Out) |
|---|---|---|
| **What it means** | Upgrade instance size (t2.micro → t2.large) | Add more instances/pods across multiple AZs |
| **Downsides** | Hard hardware ceiling; needs downtime/reboot; can't dynamically shrink | — |
| **Upsides** | — | Virtually limitless; fault-tolerant (losing one node doesn't kill the app); cost-effective via dynamic scale-down |

### Kubernetes/EKS Dual-Layer Autoscaling
```
[ Inbound Traffic Spike ]
        │
        ▼
[ Horizontal Pod Autoscaler (HPA) ] → scales Pod replicas by CPU/Memory/custom metrics
        │
        ▼ (pods stuck 'Pending' — not enough node capacity)
[ Cluster Autoscaler / Karpenter ] → provisions new EC2 worker nodes
```
- **HPA:** monitors pod resource usage → adjusts replica count
- **Cluster Autoscaler / Karpenter:** watches for `Pending` pods due to insufficient node resources → requests new EC2 nodes

### Architectural Tactics for High-Traffic Spike Days
- **Scheduled Auto Scaling:** pre-warm ASGs/EKS node pools ahead of known event windows — don't wait for reactive metrics
- **Pre-warm Load Balancers:** notify AWS support or pre-configure ALBs to avoid early connection drops
- **In-Memory Caching:** deploy **Amazon ElastiCache (Redis)** in front of the DB for hot reads/session state
- **DB Connection Pooling:** use **Amazon RDS Proxy** to reuse connections and prevent DB connection exhaustion
- **HA Data Layer:** **Amazon Aurora Multi-AZ** with Read Replicas + automated failover → sub-minute RTO

---

## 52. SonarQube — SAST & Quality Gates

**Simple takeaway:** SonarQube = the standard answer for "what SAST tool have you used?" Know the install prerequisites and the Maven pipeline integration cold.

### What SonarQube Does
Analyzes source code across languages to generate audit reports on:
- **Bugs:** logic flaws causing runtime errors
- **Vulnerabilities:** security loopholes (SQL injection, hardcoded secrets, insecure crypto)
- **Code Smells:** maintainability issues, tech debt, duplicate code

- **Quality Gates:** an automated **release gate** in CI/CD — blocks code with unacceptable vulnerability thresholds from building/deploying to production

### Architecture & Prerequisites
- **Runs on Java:** requires OpenJDK/Oracle JRE (v17+ for modern releases)
  ```bash
  # Ubuntu
  sudo apt install openjdk-17-jre -y
  # RHEL/CentOS
  sudo yum install java-17-openjdk -y
  ```
- **Database backend:**
  - Default/evaluation: embedded **H2** in-memory DB
  - **Production requirement:** external relational DB — PostgreSQL, MySQL, or Oracle (H2 can't scale/persist properly)
- **Default port:** `9000` (`http://<server-ip>:9000`)
- **Config file:** `conf/sonar.properties` (DB credentials, server URL, search node settings)
- **Default login:** `admin`/`admin` (forced password reset on first login)

### Integrating into a Maven CI/CD Pipeline
**In `pom.xml`, define connection properties:**
- `sonar.host.url` → SonarQube server endpoint
- `sonar.login`/`sonar.password` (or an auth token from User Settings)

**Trigger analysis in the pipeline:**
```bash
mvn clean verify sonar:sonar -Dsonar.host.url=http://<sonar-ip>:9000 -Dsonar.token=<auth-token>
```

**Enforce the Quality Gate:**
- The scanner sends code metrics + test coverage back to SonarQube
- A pipeline webhook/plugin queries the Quality Gate API — if the code **fails**, the pipeline **auto-aborts** before staging/deployment

---

## 53. OWASP Tools — DAST & SCA in CI/CD

**Simple takeaway:** OWASP tools cover what SonarQube (SAST) doesn't — testing the **running app** (DAST) and scanning **third-party libraries** (SCA).

### The OWASP Tooling Ecosystem
| Tool | Category | What It Does |
|---|---|---|
| **OWASP ZAP (Zed Attack Proxy)** | **DAST** (Dynamic Application Security Testing) | Tests a **running** web app externally for runtime vulnerabilities (SQLi, XSS, broken auth) — no source code access needed |
| **OWASP Dependency-Check** | **SCA** (Software Composition Analysis) | Scans direct & transitive dependencies (Maven `.jar`, npm packages) for known CVEs |
| **OWASP Dependency-Track** | Component Analysis Platform | Monitors SBOM (Software Bill of Materials) to track component risk & supply-chain integrity across releases |

### Installation Options
- **Package managers:** `sudo apt install zaproxy` (Ubuntu), or Windows installer
- **Containerized (recommended for CI/CD):** pull pre-built images — `owasp/zap2docker-stable`, `owasp/dependency-check` — avoids installing heavy deps on build agents

### Jenkins Pipeline Integration

**OWASP ZAP:**
```groovy
stage('OWASP ZAP Security Test') {
    steps {
        zaproxy apiUrl: 'http://localhost:8080', contextFile: 'context_file'
    }
}
```
Install the **OWASP ZAP Jenkins plugin** first; define target URL + a context file (scope, auth rules, exclusions).

**OWASP Dependency-Check:**
```groovy
stage('Dependency-Check') {
    steps {
        dependencyCheck additionalArguments: '', odcInstallation: 'Default'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
```

### GitLab CI / GitHub Actions
Run ZAP/Dependency-Check inside **ephemeral Docker containers** during pipeline jobs, then save the generated report as a build artifact.

```yaml
# GitLab CI example
test:
  image: owasp/zap2docker-stable
  script:
    - docker run -u zap -t -g "https://example.com" -p 8080 owasp/zap2docker-stable
```

```yaml
# GitHub Actions example
- name: Run OWASP Dependency-Check
  uses: owasp/dependency-check-action@v2
  with:
    project: 'my-app'
    path: './'
```

### Enforcement, Reporting & Automation Strategy
- **Fail-build thresholds:** break the pipeline if any vulnerability has **CVSS ≥ 7.0** or severity **HIGH/CRITICAL**
- **Automated alerting:** publish reports as artifacts + notify via Slack/Teams/Email on threshold violations
- **Scheduled scans:** run weekly/nightly — new CVEs can affect deployed dependencies even without code changes

### Positioning DevSecOps Holistically in Interviews
Don't answer with just one tool — walk through the **whole pipeline**:
| Stage | Tool |
|---|---|
| Source Code / Pre-Commit | Git hooks (`gitleaks`, `git-secrets`) |
| Static Code Quality (SAST) | SonarQube |
| Dependency Scanning (SCA) | OWASP Dependency-Check |
| Dynamic Testing (DAST) | OWASP ZAP on staging endpoints |
| Secrets & Cloud Identity | Vault / AWS Secrets Manager / Azure Key Vault + least-privilege IAM |

---

## 54. Terraform: Provisioning EKS Infrastructure (Full Walkthrough)

**Simple takeaway:** This is a complete, defensible Terraform project you can walk through live in an interview — from secrets to VPC to EKS to IAM to outputs.

### A. Provider Configuration & Secure Authentication
**Never hardcode secrets** in the `provider "aws"` block — pull them dynamically from AWS Secrets Manager:
```hcl
provider "aws" {
  region     = "us-west-2"
  access_key = data.aws_secretsmanager_secret_version.aws_secret_version.secret_string["aws_access_key_id"]
  secret_key = data.aws_secretsmanager_secret_version.aws_secret_version.secret_string["aws_secret_access_key"]
}

data "aws_secretsmanager_secret" "aws_secret" {
  name = "my-aws-credentials-secret"
}

data "aws_secretsmanager_secret_version" "aws_secret_version" {
  secret_id = data.aws_secretsmanager_secret.aws_secret.id
}
```
> 💡 In modern setups, prefer **IAM Roles via OIDC** or assumed roles over static keys entirely.

### B. Networking Topology (VPC, Subnets, Security Groups)
```hcl
resource "aws_vpc" "eks_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = { Name = "eks-vpc" }
}

resource "aws_subnet" "eks_subnet" {
  vpc_id                  = aws_vpc.eks_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = { Name = "eks-subnet" }
}

resource "aws_security_group" "eks_sg" {
  vpc_id = aws_vpc.eks_vpc.id

  egress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
  }

  tags = { Name = "eks-security-group" }
}
```

**Best practice for real production setups (mentioned as interview talking points):**
- Split subnets across **multiple AZs** using the `count` meta-argument
- **2 Public Subnets** → internet-facing ALBs & NAT Gateways
- **2 Private Subnets** → EKS worker nodes & backend pods
- Reference `aws_vpc.eks_vpc.id` directly in subnets to maintain explicit dependency ordering

### C. EKS Cluster & IAM Roles
```hcl
resource "aws_eks_cluster" "eks_cluster" {
  name     = "my-eks-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn

  vpc_config {
    subnet_ids         = [aws_subnet.eks_subnet.id]
    security_group_ids = [aws_security_group.eks_sg.id]
  }
}

resource "aws_iam_role" "eks_cluster_role" {
  name = "eks-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Principal = { Service = "eks.amazonaws.com" }
      Effect    = "Allow"
      Sid       = ""
    }]
  })

  managed_policy_arns = [
    "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy",
    "arn:aws:iam::aws:policy/AmazonEKSServicePolicy",
  ]
}
```

### D. Worker Node IAM Role & Node Group
```hcl
resource "aws_iam_role" "eks_node_role" {
  name = "eks-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Principal = { Service = "ec2.amazonaws.com" }
      Effect    = "Allow"
      Sid       = ""
    }]
  })

  managed_policy_arns = [
    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly",
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
    "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess",
  ]
}

resource "aws_eks_node_group" "eks_nodes" {
  cluster_name    = aws_eks_cluster.eks_cluster.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.eks_node_role.arn
  subnet_ids      = [aws_subnet.eks_subnet.id]

  scaling_config {
    desired_size = 2
    max_size     = 3
    min_size     = 1
  }

  instance_types = ["t3.medium"]
}
```

### Essential IAM Policies to Know (Frequently Asked!)
| Attached To | Policy | Purpose |
|---|---|---|
| Cluster Control Plane | `AmazonEKSClusterPolicy` | Core cluster management permissions |
| Cluster Control Plane | `AmazonEKSServicePolicy` | Service-level permissions for the control plane |
| Worker Nodes | `AmazonEKSWorkerNodePolicy` | Grants kubelet permission to interact with the EKS control plane |
| Worker Nodes | `AmazonEC2ContainerRegistryReadOnly` | Lets nodes pull images from private ECR repos |
| Worker Nodes | `AmazonEKS_CNI_Policy` | Enables the AWS VPC CNI plugin to allocate ENIs/IPs to pods |
| Worker Nodes | `AmazonS3ReadOnlyAccess` (or custom least-privilege) | Access to app config/artifacts in S3 |

### Terraform Outputs & Consuming Them
```hcl
output "cluster_endpoint" {
  value = aws_eks_cluster.eks_cluster.endpoint
}

output "cluster_name" {
  value = aws_eks_cluster.eks_cluster.name
}

output "cluster_certificate_authority_data" {
  value = aws_eks_cluster.eks_cluster.certificate_authority[0].data
}

output "node_group_name" {
  value = aws_eks_node_group.eks_nodes.node_group_name
}
```

**Consuming outputs:**
```bash
# From CLI/pipeline — feed into kubectl config steps
terraform output cluster_endpoint
terraform output -raw eks_cluster_endpoint
```
```hcl
# Within the same module
resource "some_resource" "example" {
  endpoint = aws_eks_cluster.eks_cluster.endpoint
}
```
```hcl
# Across modules
module.<module_name>.<output_name>
```

---

## 55. DevOps Project Explanation Framework

**Simple takeaway:** When asked "walk me through a project you built," use this exact 5-phase structure — it shows end-to-end ownership, not just tool trivia.

### The 5-Phase Narrative Structure
1. **Infrastructure as Code (IaC) Provisioning** — VPC, Subnets, Security Groups, EKS Cluster (see Section 54)
2. **Source Code Management (SCM)** — repository architecture, branching strategies (see Section 43)
3. **CI/CD Pipeline Automation** — build, test, containerize, scan (see Sections 22, 52, 53)
4. **Deployment Orchestration to Kubernetes** — manifests, Helm, rollout strategies (see Section 16)
5. **Observability, Monitoring, Alerting & DevSecOps Controls** — Prometheus/Grafana, security gates (see Sections 21, 52, 53)

> 💡 **Interview tip:** Narrate your project in this exact order. It mirrors how interviewers mentally organize a "real project" question, and it naturally leads them to ask deeper follow-ups on the phase you're strongest in.

---

## 56. Project Step 2: Fetch Code from SCM — Branching & Webhooks

**Simple takeaway:** Step 2 of your project narrative (after Terraform provisioning) — explain your repo structure, access control, branching model, and how a Git push triggers Jenkins.

### Repository Identification & Access
- **Remote endpoint structure:** `https://github.com/<org-name>/<repo-name>.git`
  - `github.com` = the SCM platform
  - `<org-name>` = your organization
  - `<repo-name>.git` = the repository
- **Access control:** create **Teams** in the org, assign granular permissions — **Read, Triage, Write, Admin** — based on developer/operator roles

### Model A: Standard GitFlow (large teams, periodic releases)
| Branch | Purpose |
|---|---|
| `master`/`main` | Stable, production-ready release code |
| `develop` | Central integration branch where features aggregate before release |
| `feature/*` | Branched off `develop` for individual features → merged back via PR after review/tests pass |
| `release/*` | Branched off `develop` when prepping a release — minor fixes, metadata, final staging validation → merges into both `main` and `develop` |
| `hotfix/*` | Branched directly off `main` for urgent prod patches → merged into both `main` and `develop` to prevent regression |

### Model B: GitLab Flow / Environment-Based / Issue-Driven Branching
- **Environment branches:** long-lived branches mapped directly to deployment environments — `dev`, `staging`/`qa`, `production`
- **Issue-driven branches:** short-lived task branches (`issue-102-fix`, `feature/cart`) merge into corresponding environments via automated promotion pipelines
- More flexible than GitFlow — blends elements of GitFlow and GitHub Flow depending on release strategy

### Handling Multi-Environment Deployment Logic
**Parameterized pipelines** + **conditional `when` directives:**
```groovy
stage('Deploy to Dev') {
    when { branch 'develop' }
    steps { /* ... */ }
}
stage('Deploy to Production') {
    when { branch 'main' }
    steps { /* ... */ }
}
```

**Continuous Delivery vs Continuous Deployment:**
| | Continuous Deployment | Continuous Delivery (Enterprise Standard) |
|---|---|---|
| Behavior | Every passed build **auto-deploys straight to production** | Auto-deploys up to staging/QA; production needs a **formal Approval Gate** (interactive input, change ticket check, or email notice) |

### Configuring the GitHub Webhook to Trigger Jenkins
1. GitHub repo → **Settings → Webhooks → Add webhook**
2. **Payload URL:** `http://<jenkins-url>/github-webhook/`
   > ⚠️ **Always include the trailing slash** — omitting it causes HTTP 302/404 delivery errors
3. **Content type:** `application/json`
4. **Event trigger:** "Just the push event" or select individual events (Pushes, Pull Requests)

---

## 57. Project Step 3: Setting Up the CI/CD Pipeline (Full Script)

**Simple takeaway:** Step 3 — be ready to write and defend a full parameterized Declarative pipeline live, including per-stage agents, versioning, and scheduled backups.

### Full Parameterized Pipeline Example
```groovy
pipeline {
    agent none

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        booleanParam(name: 'CLEAN', defaultValue: true, description: 'Clean before building')
        choice(name: 'ENVIRONMENT', choices: ['development', 'staging', 'production'], description: 'Choose deployment environment')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            agent { label 'build-node' }
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
                // build commands here
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT} environment..."
                // deployment commands here
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

**Parameter breakdown:**
- `BRANCH` — string param, which branch to build (default `main`)
- `CLEAN` — boolean, whether to wipe the workspace first
- `ENVIRONMENT` — choice param, deployment target

### Standard Declarative Pipeline Stages (Full Production Flow)
1. **Workspace Clean** — `cleanWs()` at start or in post-actions to purge stale artifacts
2. **Checkout SCM** — clone using branch specifiers + credentials
3. **Build & Package** — `mvn clean package` (Java) or `npm` (Node.js)
4. **Security & Quality Gates** — SonarQube (SAST) + OWASP Dependency-Check
5. **Containerization & Registry Push** — build Docker image with immutable tag, push to ECR/Docker Hub
6. **Deployment & Verification** — deploy to Dev/QA/Staging/Prod on EKS/EC2
7. **Post Actions** — `always`/`success`/`failure` blocks send Slack/Teams/Email alerts, archive test reports

### Master-Agent Node Scheduling (Distributed Architecture)
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
- `agent none` at top level prevents reserving a default executor on the controller upfront
- Bind each stage to a specific labeled agent (`build-node`, `security-scanner`, `prod-deployer`)

### Pipeline Resilience & Controls
- **Timeouts:** `options { timeout(time: 30, unit: 'MINUTES') }` — kill stuck/zombie processes
- **Triggers:** `githubPush()` (webhook) or `pollSCM` (scheduled polling)
- **Parallel execution:** run independent stages (unit tests, integration tests, linting) concurrently via the `parallel` directive

### Multi-Branch Pipeline Setup
1. **Manage Plugins → Install Multibranch Pipeline plugin**
2. **Dashboard → New Item → Multibranch Pipeline**
3. Under **Branch Source**, add your Git repository type, URL, and credentials
4. **Scan Multibranch Pipeline** — Jenkins auto-discovers branches and creates jobs for each

### Masked Credential Injection
```groovy
withCredentials([usernamePassword(credentialsId: 'my-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'echo "Using credentials securely"'
}
```

### Versioning Strategies in Jenkins
| Method | How |
|---|---|
| **Git Tags** | Extract release tags (e.g., `v1.4.0`) to tag images/builds |
| **Commit Hash** | Tag images with short SHA — `${GIT_COMMIT[0..7]}` — for exact traceability |
| **Build Number** | Append `${BUILD_NUMBER}` — e.g., `v1.0-${BUILD_NUMBER}` — to distinguish consecutive builds |
| **Version File** | Commit a `version.txt` tracking file, read it during the build step |

### Scheduling Jenkins Backups
1. Install a backup plugin (e.g., **ThinBackup**) or configure cloud backup integration
2. Create a backup directory: `/var/lib/jenkins/jenkinsbackup` (or mounted EFS/S3)
3. Set correct ownership: `chown -R jenkins:jenkins /backup-dir`
4. Schedule recurring cron jobs — **weekly full backups**, **nightly differential/incremental backups** of `config.xml`, job definitions, credentials, and plugin metadata

---

## 58. Project Step 4: Deploying the Application (Manifests & Helm)

**Simple takeaway:** Step 4 — deploy to EKS via raw Kubernetes manifests or (preferably, at enterprise scale) Helm charts. Know why Helm wins for reusability and parameterization.

### Prerequisites
- `kubectl` configured & authenticated to the target EKS cluster:
  ```bash
  aws eks update-kubeconfig --name <cluster_name>
  ```

### Method 1: Raw Kubernetes Manifest (`deployment.yaml`)
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
- Raw pods lack self-healing and zero-downtime lifecycle capabilities
- Deployment controllers manage ReplicaSets → enable **rolling updates, automated rollbacks, zero-downtime deploys, horizontal scaling**

**Applying it in the pipeline:**
```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/my-app
```

### Method 2: Helm Charts (Enterprise Standard)
**Why Helm wins:** reusability, parameterization, and release tracking.

**Setup:**
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm create my-app
```

**Chart directory structure:**
```
my-app/
├── Chart.yaml       # metadata: name, version, appVersion, description
├── values.yaml      # default config values — avoids hardcoding
├── charts/
├── templates/       # parameterized manifests (deployment.yaml, service.yaml, ingress.yaml, hpa.yaml)
└── .helmignore      # files excluded from packaging
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

**`templates/deployment.yaml`** — dynamically injects values:
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
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

**Deploying in the CI/CD pipeline:**
```bash
helm upgrade --install my-app ./helm/my-app \
  --namespace production \
  --set image.tag=${GIT_COMMIT[0..7]} \
  --wait
```
> `helm upgrade --install` creates the release if it doesn't exist, or does an incremental rolling upgrade if it does.

**Post-deployment verification:**
```bash
kubectl get pods -n production
```

### Alternative Deployment Paradigms
| Tool/Approach | Description |
|---|---|
| **GitOps with ArgoCD** | Pull-based model — an in-cluster controller continuously reconciles live cluster state with the Git repo's desired manifests |
| **AWS CodeDeploy** | Requires CodeDeploy agent on nodes; executes `appspec.yml` lifecycle hooks for in-place or blue/green shifts |
| **Azure App Services** | Managed PaaS deployment directly from Azure DevOps pipelines |

### How to Summarize Steps 1–4 in an Interview (Full Narrative)
> *"First, I provisioned the underlying AWS VPC, multi-AZ subnets, security groups, and EKS cluster using modular Terraform configurations stored in remote S3 with DynamoDB state locking. Our development teams follow a GitFlow model — code commits trigger automated CI webhooks to our Jenkins server. The Jenkins Declarative Pipeline checks out the code, compiles it with Maven, enforces SonarQube quality gates and OWASP dependency checks, builds an immutable Docker container tagged with the Git commit hash, and pushes it to private Amazon ECR. Finally, the pipeline triggers an automated Helm upgrade on our EKS cluster, dynamically injecting the new image tag into `values.yaml` and performing a zero-downtime rolling update verified by `kubectl rollout status` checks."*

---

## 59. Project Step 5: Monitoring & Logging Setup

**Simple takeaway:** Step 5 — the final piece: observability via metrics (Prometheus/Grafana) and logs (EFK stack), tied together with alerting.

### Metrics vs Logs — The Core Difference
| | Metrics (Time-Series Data) | Logs (Discrete Event Records) |
|---|---|---|
| What | Periodically scraped numeric aggregations (CPU%, memory, disk I/O, pod restarts, request rates) | Detailed timestamped text records per discrete event (API hit, DB query, stack trace) |
| Best for | Threshold detection, trends, dashboards | Root-cause forensic debugging |

### Metric Collection & Visualization Architecture
```
[ Target Pods / Nodes ] ── exposes /metrics
          │
          ▼
 [ Prometheus Server ] ── scrapes metrics → stores in TSDB
          │
          ├──► [ Grafana ]       → Dashboards (CPU, Pod Status, Latency)
          └──► [ Alertmanager ]  → PagerDuty / Slack / Email
```

**Install Prometheus (Kubernetes via Helm):**
```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```
- Deploy in a dedicated namespace: `kubectl create ns monitoring`
- The `kube-prometheus-stack` chart packages the Prometheus Operator, node-exporter, and kube-state-metrics

**Install Grafana:**
```bash
helm install grafana grafana/grafana
```
- In Grafana: **Data Sources → Prometheus** → enter server URL (`http://prometheus-server.monitoring.svc.cluster.local:9090`) → Save
- Build/import dashboards for CPU, memory, pod status, network throughput

**Prometheus Alert Rule Example:**
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
- **Alertmanager** dedupes, groups, and routes firing alerts to Slack/PagerDuty/Email

### Centralized Log Aggregation (EFK Stack)
```bash
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
```
- **Fluentd/Fluent Bit:** deployed as a **DaemonSet** on every node — tails container stdout/stderr from `/var/log/containers/*.log`, enriches with K8s metadata (pod name, namespace), ships to Elasticsearch

**Sample Fluentd config:**
```
<match>
  @type elasticsearch
  host elasticsearch.default.svc.cluster.local
  port 9200
  logstash_format true
  flush_interval 5s
</match>
```
- **Elasticsearch:** ingests, indexes, stores logs across time-based indices
- **Kibana:** web UI to query logs (Lucene/KQL), search error codes, trace transaction IDs

### Cloud-Native Alternatives
| Cloud | Native Tool |
|---|---|
| AWS | CloudWatch (+ Container Insights for EKS) |
| Azure | Azure Monitor |
| GCP | Cloud Logging/Monitoring (formerly Stackdriver) |
| SaaS | Datadog, New Relic, Dynatrace |

### Log Rotation & Retention
- Rotate logs regularly, archive/delete based on retention policy
- **Loki** (lightweight log aggregation, pairs with Grafana) is a popular alternative to full EFK for K8s clusters

### Interview Summary Answer for Step 5
> *"For observability, we run the kube-prometheus-stack in our EKS cluster deployed via Helm under a dedicated monitoring namespace. Prometheus scrapes pod endpoints, and we visualize CPU, memory, and pod health using pre-built Grafana dashboards. For proactive incident management, Alertmanager routes high-severity alerts — like CrashLoopBackOff or node memory pressure — directly to our team's Slack incident channel. For debugging, application logs are aggregated via Fluent Bit and forwarded to an Elasticsearch/Kibana cluster (or AWS CloudWatch Container Insights), letting us filter by namespace and trace exceptions without SSH access to worker nodes."*

> 🎓 **SRE Insight:** Observability & telemetry form the core of the dedicated **Site Reliability Engineering (SRE)** track — focused on SLOs, proactive alerting, and reducing **MTTR** (Mean Time to Resolution).

---

## 60. Headless Services — Deep Dive

**Simple takeaway:** Regular Services load-balance randomly across pods — which breaks stateful apps (databases) that need traffic to hit a **specific** pod (like the primary). Headless Services solve this by exposing direct per-pod DNS instead of one shared IP.

### The Core Problem: Why Regular Services Fall Short for Databases
- A standard **ClusterIP** Service allocates one virtual IP; `kube-proxy` load-balances traffic across all healthy pods **randomly/round-robin**
- **Why this breaks stateful DBs:** in primary/replica setups (PostgreSQL, MySQL, Redis, Kafka, MongoDB), **writes must go to the primary/leader pod only**, while reads can go to replicas. A standard ClusterIP hides which pod actually receives the request.

### What Is a Headless Service?
```yaml
spec:
  clusterIP: None
```
- **No virtual IP is assigned** — the control plane doesn't configure `kube-proxy` load-balancing for it
- **CoreDNS returns direct A/AAAA records** mapping to the actual IPs of individual backend pods, instead of one shared ClusterIP

### Pairing with StatefulSets
- **StatefulSets** give pods **predictable, sticky, zero-indexed identities**: `db-0`, `db-1`, `db-2` (vs random hashes like `app-7b9d6c-xyz` in stateless Deployments)
- **Deterministic FQDN format:**
  ```
  <pod-name>.<headless-service-name>.<namespace>.svc.cluster.local
  ```
  - Primary: `db-0.my-db-headless.default.svc.cluster.local`
  - Replica 1: `db-1.my-db-headless.default.svc.cluster.local`
  - Replica 2: `db-2.my-db-headless.default.svc.cluster.local`

### Manifests

**Headless Service:**
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

**StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-db
spec:
  serviceName: "my-db-headless"   # links StatefulSet to the Headless Service
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
```

### How the Application Connects
```yaml
env:
  - name: DATABASE_URL
    value: "postgresql://postgres@my-db-headless.default.svc.cluster.local:5432/mydb"
```
- **Write connection:** direct to the primary — `postgres://user:password@postgres-db-0.my-db-headless:5432/proddb`
- **Read connection pool:** pointed at replicas — `postgres-db-1.my-db-headless:5432` or `postgres-db-2.my-db-headless:5432`

### Comparison Table for Interviews
| Feature | Standard Service (ClusterIP) | Headless Service (`clusterIP: None`) |
|---|---|---|
| Cluster IP | Allocated from service CIDR | None |
| Routing/Proxying | Layer 4 via `kube-proxy` (iptables/IPVS) | Bypasses `kube-proxy` — client resolves DNS directly |
| DNS Resolution | Returns a single virtual ClusterIP | Returns multiple A-records or explicit per-pod FQDNs |
| Target Workload | Stateless web apps, REST APIs, microservices | Stateful clusters — DB leaders/followers, Kafka brokers, Elasticsearch nodes, Cassandra rings |

---

## 61. Pod-to-Pod Communication & Traffic Routing

**Simple takeaway:** Kubernetes hides ever-changing Pod IPs behind stable abstractions — Services, CoreDNS, and Ingress — so apps never hardcode backend addresses. Know the 3 communication tiers cold.

### The Core Interview Question
*"How does communication happen between dependent pods (frontend → backend → database), and how is traffic load-balanced across replicas?"*

### The 3 Communication Tiers
```
1. Intra-Pod (Container-to-Container)
   [ App Container ] <──(localhost:port)──> [ Sidecar/Proxy ]

2. Inter-Pod (Same Namespace)
   [ Frontend Pod ] ──(http://backend-service:8080)──► [ Backend Service VIP ]
                                                              │
                                                     (round-robin / iptables)
                                                              ├──► Pod-1
                                                              └──► Pod-2

3. Cross-Namespace
   [ Frontend Pod ] ──(http://backend-service.prod.svc.cluster.local:8080)──► Target Namespace
```

1. **Within the same Pod:** containers share a network namespace/IP → communicate directly via `localhost:<port>`, no DNS needed
2. **Across pods, same namespace:** use the short Service DNS name — `http://<service-name>:<port>` (e.g., `http://backend-service:8080`)
3. **Across namespaces:** requires the **FQDN** — `<service-name>.<namespace>.svc.cluster.local` (e.g., `backend-service.backend.svc.cluster.local`)

### How Traffic Gets Forwarded & Load-Balanced
1. **DNS Resolution:** CoreDNS intercepts the query (`backend-service`) and resolves it to the Service's static ClusterIP
2. **Endpoint Discovery:** the Service controller continuously updates the **Endpoints/EndpointSlice** object, tracking pods matching `spec.selector` labels
3. **Traffic Distribution:** `kube-proxy` rules (iptables random probability, or IPVS weighted round-robin) forward packets from the virtual ClusterIP to one of the healthy backing Pod IPs
   - Example: Request 1 → Pod A, Request 2 → Pod B, Request 3 → Pod C, Request 4 → Pod A (round-robin)

### Service Types & How They Forward Traffic
| Type | Behavior |
|---|---|
| **ClusterIP** (default) | Internal-only; load-balances across matching pods |
| **NodePort** | Exposes a port on every node; forwards to backend pods |
| **LoadBalancer** | Provisions a cloud LB; forwards external traffic to backend pods |
| **Headless** (`clusterIP: None`) | No load-balancing — direct per-pod DNS (see Section 60) |

### Special Routing Modes
- **Headless Services:** for direct pod addressing without `kube-proxy` load-balancing (stateful DB leader/follower topologies) — see Section 60
- **Ingress Controllers:** Layer 7 routing for external HTTP/HTTPS traffic — host-based (`api.example.com`) or path-based (`example.com/api`) rules
- **Egress Policies:** NetworkPolicies restricting traffic **leaving** the cluster to external APIs/databases/third-party gateways

### Summary Answer Structure for Interviews
> *"Communication flows through a decoupled service abstraction. Containers within the same pod communicate directly over localhost. When a frontend pod calls a backend pod, it targets the backend's Service DNS name (`service.namespace.svc.cluster.local`). CoreDNS resolves this to the Service's virtual ClusterIP. kube-proxy on each worker node intercepts the request and routes it to one of the matching pod IPs recorded in the EndpointSlice, typically using round-robin distribution. If specific stateful instances need direct one-on-one targeting — like database replicas — a Headless Service (`clusterIP: None`) is used to bypass the proxy and expose deterministic per-pod DNS records."*

---

## 📌 To Be Continued...
*(More topics will be appended here as they come in.)*
