# 🚀 DevOps Interview Master Guide — Organized Tool-Wise

> **Purpose:** A long-term reference for DevOps interview prep. Organized **by tool/technology** (not by topic order) so related concepts sit together and are easy to recall under interview pressure.
> Every section starts with a **Simple Summary** (plain-language, easy to remember) followed by the **Full Details** (commands, YAML, real scenarios) — nothing from the original notes has been removed, only re-grouped and reformatted.

---

## 🧭 How to Use This Guide

1. Read the **Simple Summary** boxes first — they compress each tool down to the one or two ideas an interviewer actually wants to hear.
2. Use the **Full Details** underneath when you need exact commands, YAML syntax, or a real scenario to describe.
3. Use the **Table of Contents** to jump straight to the tool being asked about.

---

## 📚 Table of Contents

1. [Big-Picture Project Narrative](#1-big-picture-project-narrative)
2. [Git & GitHub](#2-git--github)
3. [Jenkins](#3-jenkins)
4. [GitHub Actions](#4-github-actions)
5. [Docker](#5-docker)
6. [Kubernetes](#6-kubernetes)
7. [Helm](#7-helm)
8. [ArgoCD / GitOps](#8-argocd--gitops)
9. [Cert-Manager](#9-cert-manager)
10. [Terraform](#10-terraform)
11. [Ansible](#11-ansible)
12. [Terraform + Ansible Together](#12-terraform--ansible-together)
13. [AWS](#13-aws)
14. [Azure](#14-azure)
15. [Linux & Shell Scripting](#15-linux--shell-scripting)
16. [SQL](#16-sql)
17. [SonarQube & DevSecOps Scanning](#17-sonarqube--devsecops-scanning)
18. [Monitoring & Logging (Prometheus/Grafana/EFK)](#18-monitoring--logging-prometheusgrafanaefk)
19. [SRE & Observability](#19-sre--observability)
20. [RBAC & Security (Cloud + Kubernetes)](#20-rbac--security-cloud--kubernetes)
21. [OpenShift](#21-openshift)
22. [Cloud Migration & Disaster Recovery](#22-cloud-migration--disaster-recovery)
23. [DevOps Culture, Maturity & SDLC](#23-devops-culture-maturity--sdlc)
24. [Career, Behavioral & Company-Specific Prep](#24-career-behavioral--company-specific-prep)
25. [Quick Recall Cheat Sheet](#25-quick-recall-cheat-sheet)

---

## 1. Big-Picture Project Narrative

> 🧠 **Simple Summary:** Every DevOps project follows the same 5 steps. Memorize this one sentence:
> *"Build the land (Terraform) → Store the code (Git) → Automate the pipeline (Jenkins) → Ship the app (K8s/Helm) → Watch it (Prometheus/Grafana/EFK)."*

### Full Details

| Step | Topic | What It Covers |
|------|-------|-----------------|
| 1 | Infrastructure Provisioning | Terraform (VPC, Subnets, EKS Cluster) |
| 2 | Source Code Management (SCM) | Branching strategy, team access, webhooks |
| 3 | CI/CD Pipeline | Jenkins Declarative Pipeline (build, test, scan, push) |
| 4 | Application Deployment | Kubernetes Manifests / Helm Charts on EKS |
| 5 | Monitoring & Logging | Prometheus, Grafana, EFK stack, alerting |

### 🎤 Full Interview Summary Script (walk-through answer)

1. **Infrastructure:** *"First, I provisioned the underlying AWS VPC, multi-AZ subnets, security groups, and EKS cluster using modular Terraform configurations stored in remote S3 with DynamoDB state locking."*
2. **SCM & Branching:** *"Our development teams follow a GitFlow model. Code commits trigger automated CI webhooks to our Jenkins server."*
3. **CI/CD Pipeline:** *"The Jenkins Declarative Pipeline checks out the code, compiles it with Maven, enforces SonarQube quality gates and OWASP dependency checks, builds an immutable Docker container tagged with the Git commit hash, and pushes it to private Amazon ECR."*
4. **Deployment:** *"Finally, the pipeline triggers an automated Helm upgrade (`helm upgrade --install`) on our EKS cluster, dynamically injecting the new image tag into `values.yaml` and performing a zero-downtime rolling update verified by `kubectl rollout status` checks."*
5. **Monitoring:** *"For observability, we run `kube-prometheus-stack` for metrics/dashboards, Alertmanager for proactive Slack alerts, and EFK/CloudWatch for centralized log aggregation and debugging."*

### DevOps Learning Roadmap (Step-by-Step Sequence)

1. **Linux Fundamentals** — filesystem hierarchy, disk mount points, permissions; troubleshooting CPU/memory/uptime/process management; basic networking (`ping`, checking open ports)
2. **Shell Scripting** — bash scripts with conditionals/loops/arrays; parsing CLI output with `grep`, `awk`, `sed`, `cut`
3. **Git & GitHub** — working dir/staging/local repo/remote branches; `add`/`commit`/`push`/`pull`/merge, `.gitignore`, Personal Access Tokens (PAT)
4. **CI/CD Pipelines** (Jenkins/GitHub Actions/GitLab CI) — checkout, build (Maven/npm), push artifacts/images, secrets management, parameterized builds, webhook triggers
5. **Cloud Infrastructure & Networking** — VPCs/VNets, public/private subnets, CIDR blocks, Internet/NAT Gateways, 3-tier design, DNS (Route 53), ALBs
6. **Containerization & Orchestration** — Dockerfile directives, image building, registries; Kubernetes Pods/Deployments/Services/Ingress
7. **IaC & Config Management** — Terraform for cloud provisioning; Ansible as a secondary tool for config management
8. **GitOps & Advanced Automation** — ArgoCD/Flux CD; Python for cloud automation scripts; DevSecOps scanning (SonarQube, Trivy)

---

## 2. Git & GitHub

> 🧠 **Simple Summary:** Git has 3 local areas (working → staging → repo) plus a remote. Know **one** branching model cold (GitFlow is the safe pick), know `fetch` vs `pull`, and be ready to explain webhooks.

### Repository Basics

```
https://github.com/mycompany-test/project.git
```
| Part | Meaning |
|------|---------|
| `github.com` | SCM (Source Code Management) platform |
| `mycompany-test` | Organization |
| `project.git` | Repository |

Teams are created in GitHub/GitLab and given specific access levels (Read, Triage, Write, Admin) based on role.

### 🌳 Branching Strategies

#### Model A: Git Flow (best for large teams / scheduled releases)

| Branch | Purpose |
|--------|---------|
| `main` / `master` | Always production-ready code |
| `develop` | Integration branch — new features merge here first |
| `feature/*` | Built from `develop`, merged back via Pull Request after review |
| `release/*` | Cut from `develop` when preparing a release — final testing & bug fixes |
| `hotfix/*` | Branched from `main` to urgently fix production bugs, then merged into both `main` and `develop` |

**Analogy:** `develop` = kitchen prepping dishes, `release` = final plating/QA, `main` = served to customers, `hotfix` = emergency fix on a served dish.

#### Model B: GitLab Flow / Environment-Based Branching

- **Long-lived branches** mapped to environments: `dev`, `staging/qa`, `production`
- **Short-lived branches** tied to issues/tasks: `issue-102-fix`, `feature/cart`
- These merge into environment branches through **automated promotion pipelines**

> 💬 **Interview tip:** Pick ONE model and describe it confidently. Git Flow = structured/enterprise. GitLab Flow = flexible/issue-driven.

### 🔗 Webhook Integration (GitHub → Jenkins)

| Setting | Value |
|---------|-------|
| Payload URL | `http://<jenkins-url>/github-webhook/` |
| Content-Type | `application/json` |
| Trigger | "Just the push event" or select individual events (push, PR) |

> ⚠️ **Critical Tip:** Always include the trailing slash `/` — missing it often causes HTTP 302/404 errors from Jenkins.

### 📜 Essential Git Commands Cheat Sheet

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

### `git fetch` vs `git pull`

| Command | What It Does |
|---------|----------------|
| `git fetch` | Downloads new commits/branches/tags to local Git metadata — **does not touch your working directory** |
| `git pull` | `git fetch` + `git merge` (or rebase) — actively integrates remote changes into your current branch |

> **Formula:** `git pull` = `git fetch` + `git merge`

### Resolving Git Merge Conflicts (Experienced-level Q)

Explain how conflicts arise between feature and base branches → identify conflict markers → manually reconcile in editor → run tests to validate → complete the merge commit.

---

## 3. Jenkins

> 🧠 **Simple Summary:** Jenkins = the CI/CD engine. Know: declarative vs scripted pipelines, `agent none` + labeled agents, parameterized pipelines, the `when` directive for conditional stages, and how to secure secrets with `withCredentials`.

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

### 🏗️ Full Production-Grade Pipeline Stages

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

### Declarative vs Scripted Pipelines

| Type | Description |
|------|--------------|
| **Scripted** | Groovy-based, imperative, more flexible but more complex |
| **Declarative** | Modern, structured syntax: `pipeline { agent any stages { ... } }` |

### Real-World Jenkins Scenario Q&A

**1. Multi-Environment Deployments (Dev → Staging → Prod)**
- **Parameterized pipelines:** build parameters (`choice`, `string`, active choices) or config files to select the target environment
- **Conditional execution:** `when { expression { ... } }` blocks for environment-specific logic
- **Folder/scope-level configs:** store env credentials/variables at the Jenkins folder level to avoid cross-contamination

**2. Handling Pipeline Timeout & Partial Failure**
- **State tracking/idempotency:** treat configs like IaC state — verify applied state rather than assume it
- **Retry mechanisms:** wrap transient ops in `retry(count) { ... }` with `timeout(time: 10, unit: 'MINUTES')`
- **Restart from failed stage:** re-trigger from the specific failed stage instead of rebuilding from scratch

**3. Production Deployment Gates & Approvals**
```groovy
input message: 'Approve Prod Deployment?', submitter: 'qa-leads'
```
- **Enterprise auditing:** integrate with ServiceNow/Jira — pipeline queries the change request state and only proceeds once **Approved**

**4. Secret & Credential Handling**
- Use **Jenkins Credentials Provider** + `withCredentials` binding (`usernamePassword`, `amazonWebServicesCredentials`, `string`) — ensures secrets are masked in console logs

**5. Automated Rollbacks on Failure**
```groovy
post {
    failure {
        // fetch previously archived known-good artifact
        // or trigger a rollback workflow to redeploy the prior stable tag
    }
}
```

**6. Unique Docker Image Tagging**
- Tag with Git commit SHA (`GIT_COMMIT`) or semver + build number (`v1.2.0-${BUILD_NUMBER}`)
- **Avoid mutable tags like `latest`** — enables exact traceability back to source

**7. Optimizing Long-Running Test Suites**
- Use the `parallel` block to distribute independent test suites (unit, functional, linting, security scans) across multiple agents/executors simultaneously

### Common Real-World Jenkins Failures

| Failure | Root Cause | Fix |
|---------|------------|-----|
| **Agent Offline** | SSH credential expiry, network/firewall drop, JVM/Java version mismatch | Renew credentials, check connectivity, align Java versions |
| **Hanging/Stuck Builds** | Thread deadlocks, exhausted executors, unhandled interactive prompts (e.g., missing `-auto-approve`), no timeout | Add `options { timeout(...) }`, use `cleanWs()` |
| **Disk Full** | Stale Maven `~/.m2` deps, old logs, untagged Docker images | Schedule `docker system prune -af`, enable workspace discards |
| **Broken Plugins/Dependencies** | Jenkins core upgraded without checking plugin compatibility, Docker Hub rate limits | Check compatibility matrix before upgrading; use authenticated pulls |

### Rollback: rollback to a previous build

Go to job → Build History → select previous build → click **Rebuild** (or download/redeploy that build's artifact).

### Hands-On IaC Project: Terraform + Jenkins → EC2

**Jenkinsfile:**
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

**Setting up the job:** Jenkins → New Item → Pipeline → "Pipeline script from SCM" → connect GitHub repo → Save and Build.

### CI/CD Tool Migration — Jenkins to Azure DevOps

- Scenario: ~700 Jenkins pipelines, hundreds of shared libraries, self-hosted Linux/Windows agents, SonarQube, Artifactory, Kubernetes deployments — migrate to Azure DevOps Pipelines with zero release interruption.
- Approach: migrate pipelines, credentials, agents, artifacts, deployment strategies, approvals, secrets, and rollback mechanisms incrementally; run **both CI/CD systems in parallel** before decommissioning Jenkins, validating each migrated pipeline against its Jenkins counterpart before cutting teams over.

---

## 4. GitHub Actions

> 🧠 **Simple Summary:** Hierarchy is **Workflows → Jobs → Steps/Actions**. Secrets go in GitHub Secrets, never in code. Pin actions to a commit SHA, not a mutable tag.

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

### Interview Q&A

1. **How do you implement CI/CD with GitHub Actions?** — Workflows → Jobs → Steps/Actions; triggers via `push`, `pull_request`, `schedule` (cron), or `workflow_dispatch` (manual)
2. **Rollback strategies:** revert the commit/tag, or re-deploy the prior immutable image/release artifact
3. **External integrations:** Slack notifications, Jira ticket updates, AWS IAM OIDC auth for secretless cloud access
4. **GitHub-Hosted vs. Self-Hosted Runners:**

| Type | Use When |
|------|-----------|
| **GitHub-Hosted** | Standard, ephemeral, no special requirements |
| **Self-Hosted** | Private VPC access needed, proprietary build tools, specialized hardware/caching |

5. **Multi-environment deployments:** configure **GitHub Environments** (Dev, Staging, Prod) with required protection rules, environment-specific secrets, manual reviewer approvals
6. **Custom GitHub Actions:** build reusable **Composite Actions**, **Docker container actions**, or **JavaScript actions**
7. **Debugging a failing workflow:** review live step logs; enable `ACTIONS_STEP_DEBUG` for verbose runner logs; re-run failed jobs
8. **Security best practices:**
   - Store tokens strictly in **GitHub Actions Secrets** — never in code
   - Limit `GITHUB_TOKEN` permissions via explicit `permissions:` blocks (Principle of Least Privilege)
   - Pin action versions to **specific commit SHAs**, not mutable tags (e.g., `@v3` can change; a SHA can't)

### 🧩 GitHub Actions Contexts — Deep Dive

Built-in objects containing metadata about a workflow run, the runner environment, secrets, and the triggering event:
```yaml
${{ <context>.<property> }}
# e.g. ${{ github.ref }}, ${{ secrets.DOCKER_TOKEN }}
```

| Context | Purpose | Example Use |
|---------|---------|--------------|
| `github` | Run & git metadata (`github.actor`, `github.event`, `github.repository`, `github.ref`, `github.run_number`) | Branch checks, event inspection, audit trail |
| `secrets` | Secure vault values (`secrets.MY_TOKEN`) | Masked API keys, registry tokens, SSH keys |
| `env` | Environment variables at workflow/job/step level | Passing custom config to steps |
| `runner` | Runner metadata (`runner.os`, `runner.arch`, `runner.temp`) | OS-conditional logic (Linux vs. Windows) |
| `job` / `steps` | Status & outputs of current jobs/steps | Inter-job dependencies, execution tracking |

**Conditional Execution:**
```yaml
if: github.ref == 'refs/heads/main'
```

**Accessing Commit Messages:**
```yaml
${{ github.event.head_commit.message }}
```

**⚠️ Security: Script Injection Precaution**
```yaml
# ❌ Risky — direct interpolation
run: echo "${{ github.event.issue.title }}"

# ✅ Safe — assign to an env var first
env:
  TITLE: ${{ github.event.issue.title }}
run: echo "$TITLE"
```

**Debugging Contexts at Runtime:**
```yaml
- name: Dump GitHub Context
  run: echo '${{ toJSON(github) }}'
```

**Dynamic Artifact Naming:** append `${{ github.run_number }}` or `${{ github.sha }}` for unique, deterministic artifact/release names.

**Real-World Scenario Answers**

| Scenario | Solution |
|----------|----------|
| Deploy only when PR has a specific label | `if: github.event.pull_request && contains(github.event.pull_request.labels.*.name, 'deploy')` |
| Set target URL dynamically by branch | Evaluate `${{ github.ref }}` — route `refs/heads/main` to prod, feature branches to preview/lower envs |
| Restrict execution to specific users | `if: contains(fromJSON('["lead-admin", "authorized-dev"]'), github.actor)` |

---

## 5. Docker

> 🧠 **Simple Summary:** Know `ENTRYPOINT` vs `CMD`, `COPY` vs `ADD`, multi-stage builds, and why `:latest` is dangerous. Always pin versions, always use `.dockerignore`, always run as non-root.

### Beginner Level

| Question | Simple Answer |
|----------|-----------------|
| What is a Dockerfile? | A text manifest of instructions packaging an app + dependencies into an immutable, platform-independent image |
| `ADD` vs. `COPY` | `COPY` copies local files/dirs plainly. `ADD` also auto-extracts local tar archives and can fetch remote URLs |
| `ENTRYPOINT` vs. `CMD` | `ENTRYPOINT` = the immutable base executable. `CMD` = default arguments, overridable at `docker run` |
| `WORKDIR` | Sets the working directory for subsequent `RUN`/`CMD`/`ENTRYPOINT`/`COPY` |
| `EXPOSE` | Documents which ports the container listens on (informational — `-p` is still needed at runtime) |
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

### Practical Docker Tasks

- **Reduce image size:** minimal base images (Alpine, Distroless), multi-stage builds, chain `RUN` commands, avoid unnecessary dependencies
- **Clean unused containers/images:**
  ```bash
  docker system prune -a
  ```
- **Docker Networking:** network drivers (`bridge`, `host`, `overlay`, `none`), container-to-container communication, port mapping (`-p`), port exposure (`EXPOSE`)
- **Containers vs. VMs:**

| | Containers | Virtual Machines |
|---|---|---|
| Virtualization level | OS-level, share host kernel | Hardware-level, hypervisor-based |
| Footprint | Lightweight | Full dedicated guest OS |

---

## 6. Kubernetes

> 🧠 **Simple Summary:** Everything routes through a **Service** (stable DNS name + virtual IP). `kube-proxy` load-balances round-robin — except for **Headless Services** (`clusterIP: None`), which give direct per-pod DNS for stateful workloads like databases. Deployments give self-healing + rolling updates that a raw Pod doesn't.

### 6.1 Core Concepts

| Concept | Description |
|---------|--------------|
| **Deployment** | Manages the lifecycle of identical pods — replica count, update strategy, rollbacks |
| **Pod** | Smallest deployable unit; containers *inside the same pod* share a network namespace and can talk via `localhost` |

**Why Deployment instead of a raw Pod?**
> A raw Pod has no self-healing and no rolling update capability. A `Deployment` manages ReplicaSets, giving you rolling updates, automated rollbacks, zero-downtime deploys, and scalability.

### 6.2 Plain Manifest Example

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

Apply and verify:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/my-app
```

### 6.3 Service Types

| Type | Description |
|------|-------------|
| `ClusterIP` (default) | Exposes service **inside the cluster only** |
| `NodePort` | Exposes service on a fixed port on every node |
| `LoadBalancer` | Exposes service externally via cloud load balancer |
| `Headless` (`clusterIP: None`) | No virtual IP — DNS resolves directly to each individual pod |

### 6.4 🎯 Headless Service — The Big Interview Topic

**The problem it solves:**
> A standard `ClusterIP` service load-balances traffic randomly/round-robin across pods via `kube-proxy`. This breaks stateful systems (databases, Kafka, etc.) where writes must go to a specific primary/leader pod, while reads can go to replicas.

**The fix:**
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
- Best paired with a **StatefulSet**, giving pods stable, predictable names (`db-0`, `db-1`, `db-2`).

**Resulting DNS pattern:**
```
<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local
```

**Full Example: StatefulSet DB + App Deployment**
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

**Standard vs. Headless Comparison**

| Feature | Standard Service (ClusterIP) | Headless Service (`clusterIP: None`) |
|---------|-------------------------------|----------------------------------------|
| Cluster IP | Allocated from service CIDR | None |
| Routing/Proxying | Handled by `kube-proxy` (iptables/IPVS) | Bypasses `kube-proxy` — client resolves DNS directly |
| DNS Resolution | Returns single virtual ClusterIP | Returns multiple A-records / per-pod FQDNs |
| Best for | Stateless web apps, REST APIs, microservices | Stateful workloads: DB leaders/replicas, Kafka brokers, Elasticsearch, Cassandra |

### 6.5 Pod-to-Pod Communication & Traffic Routing

| Scope | How They Talk |
|-------|----------------|
| **Same Pod** (container-to-container) | Directly via `localhost` (shared network namespace) |
| **Same Namespace** (pod-to-pod) | Via short service DNS: `http://backend-service:8080` |
| **Cross-Namespace** | Via full FQDN: `backend-service.backend.svc.cluster.local` |

**How Traffic Actually Gets Forwarded**
1. A pod sends a request to a Service's DNS name
2. **CoreDNS** resolves this to the Service's **ClusterIP**
3. The Service tracks matching pods via **Endpoints / EndpointSlice** (label selectors)
4. **kube-proxy** intercepts traffic and forwards it round-robin via iptables/IPVS

**Special Routing Modes**

| Mode | Use Case |
|------|----------|
| **Headless Service** | Direct, deterministic pod addressing — bypasses `kube-proxy` (stateful DBs) |
| **Ingress Controller** | Layer-7 routing for external HTTP/HTTPS traffic — host-based or path-based rules |
| **Egress Policies** | Network policies restricting traffic *leaving* the cluster |

### 6.6 Ingress vs. Load Balancer

| Option | Cost/Behavior | Best For |
|--------|-----------------|----------|
| `Service type: LoadBalancer` | Allocates a **dedicated** cloud LB per exposed service → cost scales linearly | Raw TCP/UDP, non-HTTP protocols, low-latency websockets |
| **Ingress Controller** | **One** entry point multiplexes traffic across many services via path-based/host-based rules | Standard HTTP/HTTPS routing — much more cost-effective |

**Ingress vs Ingress Controller:** Ingress = the API object defining routing rules. Ingress Controller = the component (NGINX, AGIC, ALB Controller) that implements those rules. A plain L4 Load Balancer **cannot** do host/path-based HTTP routing like an Ingress controller.

### 6.7 CronJobs

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
Schedule format: `Minute | Hour | Day-of-Month | Month | Day-of-Week`.
**Use cases:** DB snapshot exports, cert-manager validation runs, log archiving, cache warming.

### 6.8 `kubectl` Cheat Sheet

**Declarative vs. Imperative**
| Approach | Description |
|----------|--------------|
| **Imperative** | Direct CLI commands: `kubectl create ns test`, `kubectl run nginx --image=nginx` |
| **Declarative** | Define desired state in YAML/JSON: `kubectl apply -f manifest.yaml` |

```bash
kubectl get ns
kubectl get pods -n <namespace_name>
kubectl describe pod <pod_name>
kubectl get pods -o wide

kubectl create ns my-app
kubectl run my-pod --image=nginx -n my-app
kubectl apply -f ingress-gateway.yaml -n <namespace>

kubectl logs <pod_name>
kubectl exec -it <pod_name> -- /bin/bash
kubectl get pods --watch

kubectl scale deployment <deployment_name> --replicas=2
kubectl delete pod <pod_name>
kubectl delete ns <namespace_name>
```

### 6.9 Troubleshooting Scenarios (Top 10 + More)

| # | Scenario | Root Cause / Fix |
|---|----------|---------------------|
| 1 | Pod stuck in `CrashLoopBackOff` | `kubectl logs`/`kubectl describe pod` — wrong entrypoint, missing env/config, failing liveness probe, resource limits too low |
| 2 | Service not reachable | `kubectl get svc` (correct type?), `kubectl describe svc` (endpoints populated?), label selector mismatch, pod readiness, network policies |
| 3 | Pod stuck `Pending` | Insufficient cluster resources, node selector/affinity mismatch, taints without tolerations, PVC not bound |
| 4 | High CPU usage cluster-wide | `kubectl top pods`, check requests/limits, use HPA, investigate inefficient code/memory leaks, cluster autoscaling |
| 5 | Rolling update failure | `kubectl rollout status` → `kubectl rollout undo deployment <name>`; investigate probes, image issues, config errors |
| 6 | ConfigMap updated but pods didn't pick it up | ConfigMaps don't auto-restart pods — run `kubectl rollout restart deployment <name>` |
| 7 | App can't read Secrets | Verify with `kubectl get secrets`, check mount method (env var vs. volume), confirm key names and RBAC permissions |
| 8 | Intermittent pod-to-pod communication failure | Check network policies, DNS (CoreDNS) issues, service misconfig — debug with `kubectl exec` + `curl`/`nslookup` |
| 9 | Data disappears after pod restart | Pod was using **ephemeral storage** — use a **PV + PVC** with the correct StorageClass |
| 10 | Node shows `NotReady` | `kubectl describe node` — kubelet status, disk/memory pressure, network issues; restart kubelet, check `journalctl -u kubelet` |
| 11 | Deployment deleted but pods still running | Orphaned ReplicaSets, pods spawned by another controller (StatefulSet/DaemonSet/standalone), finalizer hooks, controller-manager API lag |
| 12 | Image updated in YAML but pod still uses old image | Mutable `:latest` tag **without** `imagePullPolicy: Always` prevents kubelet from pulling the new image |
| 13 | HPA configured, threshold exceeded, pod count stays at 1 | Missing Metrics Server, missing `resources.requests` (HPA needs a baseline), or min/max replica constraints |
| 14 | OOMKilled with no resource limits set | Without limits, a container can consume as much node memory as available; memory spikes → Linux OOM killer terminates the process |

### 6.10 Full Deployment YAML (probes + PVC + resources)

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

### 6.11 Advanced Kubernetes Internals (Architect-Level)

**Zero-Downtime Microservices Deployment**
- Combine Rolling Updates, Blue-Green, or Canary deployments (Argo Rollouts, Istio traffic routing, feature flags)
- Accurate `readinessProbe`, `livenessProbe`, `startupProbe`
- `maxUnavailable: 0`, `maxSurge: 1` (or `25%`) — old pods only terminate after new pods pass readiness checks
- Canary analogy: shift traffic 5% → 20% → 50% → 100% while watching error rates and latency

**Worker Node Enters `NotReady`**
- `node-lifecycle-controller` detects missed heartbeats, marks node `NotReady`
- Scheduler stops assigning new pods to it; existing pods remain until the **eviction timeout** (default 5 min), then get rescheduled
- **PodDisruptionBudgets (PDB):** guarantee minimum available instances during voluntary disruptions
- **Stateless vs. StatefulSets:** stateless pods recreate easily; StatefulSets need CSI driver unmount/remount of PVs to the new node

**Disaster Recovery: What Happens if etcd Crashes?**
> etcd is the consistent key-value store holding the entire cluster state (nodes, pods, secrets, configs).
- **Impact:** `kube-apiserver` loses read/write capability. **Running workloads keep running** (kubelet + container runtime continue) — but control plane operations **freeze**: no new scheduling, scaling, secret retrieval, or deployment updates.
- **Mitigation:** multi-node HA clusters (odd quorum: 3 or 5 nodes); restore from automated snapshots via `etcdctl snapshot restore`

**Kubernetes Networking: Pod-to-Internet Path**
```
Pod Network Namespace → veth pair → Linux Bridge/CNI Plugin (Calico, Cilium, Flannel)
   → Node routing tables → iptables/eBPF (SNAT)
   → Node's physical NIC (eth0) → VPC Router/NAT Gateway → Internet Gateway → External Web
```

**Kubernetes Cluster Hardening (Defense-in-Depth)**

| Layer | Practices |
|-------|-----------|
| **Admission Control** | Admission webhooks — OPA Gatekeeper, Kyverno |
| **Identity & Isolation** | Strict least-privilege RBAC, namespace boundaries, NetworkPolicies |
| **Runtime & Supply Chain** | Sign images (Sigstore/Cosign), scan base images (Trivy), runtime threat detection (Falco), KMS encryption for etcd secrets |

**Control Plane Lifecycle: `kubectl apply -f deployment.yaml` End-to-End**
1. `kubectl` sends an HTTP request to `kube-apiserver`
2. API Server runs Authentication → Authorization (RBAC) → Mutating/Validating Admission Controllers
3. Desired state is persisted into **etcd**
4. **Deployment/ReplicaSet Controller** detects the state diff and creates Pod objects
5. **kube-scheduler** assigns pods to worker nodes based on resource filters and affinities
6. **kubelet** on the target node instructs the container runtime (containerd/CRI-O) to pull images and start containers, and reports pod status back

**Sidecar Pattern:** auxiliary container deployed alongside the main app container — for logging, proxying, or secret sync.

**Service Mesh:** manages service-to-service communication, mTLS encryption, traffic shifting, observability (Istio, Linkerd).

### 6.12 Blue-Green Deployment

| Environment | Role |
|-------------|------|
| **Blue** | Active — currently serving all live production traffic (older version) |
| **Green** | Idle — new version deployed and thoroughly tested under production-like conditions |

**Zero-downtime cutover:** Once Green passes validation, the router/load balancer switches traffic from Blue to Green instantly. Blue stays on standby for immediate rollback.

**How Traffic Routing Actually Happens**
- **Ingress backend switch:** `backend.service.name` from `blue-service` → `green-service`
- **Service selector switch:** change label selector (e.g., `version: v1.0` → `version: v2.0`)
- DNS/URL stays identical for end users throughout.

| Context | Approach |
|---------|----------|
| **Local Lab (Minikube)** | Simulate with multiple profiles: `minikube start -p green --driver=docker` |
| **Enterprise Myth** | "Two entire duplicate Kubernetes clusters" — too expensive in practice |
| **Enterprise Reality** | Two **namespaces** in the same cluster shifting Ingress/mesh traffic; OR side-by-side `app-blue`/`app-green` Deployments switching Service selector or Service Mesh (Istio/Linkerd) traffic weight; OR dedicated node pools via node selectors/taints |

### 6.13 AKS (Azure Kubernetes Service) Specific

**What is AKS?** Microsoft's managed Kubernetes service — Azure manages the control-plane infrastructure. You manage applications, node pools, config, networking, RBAC, security, scaling, observability.

**AKS Node Pool:** group of nodes with a particular VM configuration.
- **System Node Pool** → core Kubernetes workloads
- **User Node Pool** → application workloads

**AKS Scaling**
- **HPA:** changes number of Pods based on CPU/memory/custom metrics
- **Cluster Autoscaler:** changes number of Nodes
- **KEDA:** scales workloads based on event-driven metrics (queues)

**How AKS pulls images from ACR:**
```
AKS → Identity → Azure RBAC → ACR → Pull Image
```
Common permission: `AcrPull` (don't give full `Contributor`).

**AKS Cluster Upgrade Scenario (v1.27 → v1.30)**
- 200+ microservices, stateless Deployments + StatefulSets on Azure Managed Disks; App Gateway, Key Vault, Azure Monitor, ACR integrated. Zero downtime required.
- **API & Deprecation Assessment:** review release notes for deprecated/removed APIs; audit Helm charts/manifests with static checkers (`pluto`, `kubent`); validate CSI drivers, Azure CNI, AGIC, Key Vault CSI driver compatibility.
- **Validation in Dev/Staging:** replicate the full workload in a lower environment.
- **Blue-Green Cluster Rollout:** spin up a new "green" AKS cluster on v1.30+ via Terraform/Bicep, deploy identical manifests, shift traffic progressively at App Gateway/Front Door layer (5% → 25% → 100%).
- **StatefulSets & Rollback:** synchronize persistent disk snapshots/DB replicas from blue to green before cutover; switch routing back to blue immediately on degradation.

**AKS Disaster Recovery — Cluster Deleted (Velero-based)**
- RTO < 30 min, RPO < 5 min. Backups via Velero; DBs replicated cross-region.
- **Velero:** runs in-cluster, periodically captures manifests (Deployments, ConfigMaps, Secrets, Services, Ingress, RBAC) as YAML in S3, plus native EBS VolumeSnapshots via CSI driver.
- **Recovery Flow:**
  1. **Cluster Rebuilding:** fast `terraform apply` against version-controlled IaC
  2. **Addons & CSI Setup:** reinstall VPC CNI, CoreDNS, kube-proxy, EBS CSI driver + StorageClass; install Velero pointing to the backup bucket
  3. **Velero Restore:**
     ```bash
     velero restore create --from-backup prod-cluster-backup-latest --wait
     ```
  4. **DB & DNS Cutover:** point to warm cross-region DB replica; restore Load Balancer Controller/Ingress; update Route 53 records
  5. **Smoke Validation:** `kubectl get pods -A` for zero CrashLoopBackOff/ImagePullBackOff; synthetic checkout/login transactions; confirm Prometheus/Grafana/Fluent Bit emitting metrics

**Blue-Green Deployment Architecture (EKS behind ALB)**
- Financial app, ~1M transactions/day, 30-min downtime per release. Goal: zero downtime + instant rollback.
- Blue/Green are **not fixed labels** — they alternate each release cycle.
- *Dual In-Cluster Deployments* (cost-effective): `app-blue`/`app-green` in same cluster; switch ALB listener rule target-group weights or Service selector.
- *Dual-Cluster Blue/Green* (high isolation, critical banking): two identical EKS clusters, traffic switching via Route 53 (weighted/failover routing).

### 6.14 Windows Containers / .NET on AKS (Modernization)

- 300+ legacy .NET Framework apps on Windows VMs, coupled to SMB shares and SQL Server, migrating to AKS with zero disruption.
- **Assessment:** separate stateless web APIs from services bound to SMB shares/SQL; migrate low-risk internal services first.
- **Containerization:** .NET Framework needs a Windows kernel base image:
  ```dockerfile
  FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2022
  ```
- **AKS Architecture:** hybrid node pools — Linux node pool for system services + Windows Server node pool for .NET apps. Azure Files CSI driver (SMB) for persistent volumes.
- **Secrets & DB Connectivity:** externalize connection strings to Key Vault via Secrets Store CSI Driver; secure AKS-to-SQL via Private Endpoints/VNet integration.
- **Safe Cutover:** automated build/test/push/deploy via Helm; keep legacy VMs running in parallel, decommission only after full production burn-in.

---

## 7. Helm

> 🧠 **Simple Summary:** Helm = `apt`/`yum` for Kubernetes. Helm 3 removed the insecure Tiller component. Always use `helm upgrade --install` (creates if new, upgrades if exists).

### What is Helm?
> The package manager for Kubernetes. **Helm 2** required an in-cluster component called **Tiller** (security risk). **Helm 3 removed Tiller entirely**, using the user's kubeconfig and RBAC directly.

### Chart Structure
```
mychart/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml          # Default config values injected into templates
├── templates/            # Deployments, Services, Ingress (Go template engine)
│   └── _helpers.tpl      # Reusable template snippets/naming conventions
└── charts/               # Sub-chart dependencies
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

**`templates/deployment.yaml`:**
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

**Deploy in CI/CD:**
```bash
helm upgrade --install my-app ./my-app \
  --namespace production \
  --set image.tag=${GIT_COMMIT[0..7]} \
  --wait
```

### Viewing Releases
```bash
helm list          # current namespace
helm list -A        # all namespaces
helm history <release-name>
```

### Overriding Default Values
```bash
# Method 1: custom YAML file (recommended for GitOps/multi-env)
helm upgrade --install my-app ./my-chart -f values-prod.yaml

# Method 2: CLI flags
helm upgrade --install my-app ./my-chart --set replicaCount=3 --set image.tag="v2.1.0"
```

### Uninstalling a Release
```bash
helm uninstall <release-name> -n <namespace>
```

**What is a Helm Release?** A specific **running instance** of a chart deployed to a cluster. The same chart can be deployed multiple times under different release names (`app-dev`, `app-staging`).

---

## 8. ArgoCD / GitOps

> 🧠 **Simple Summary:** GitOps = Git is the single source of truth. ArgoCD is **pull-based** (cluster pulls from Git) — the opposite of Jenkins/CI which is **push-based**. Rollback = `git revert` (preferred).

### What is ArgoCD & GitOps?

**GitOps core principle:** Git is the single source of truth for both app code and Kubernetes infra state.

**Reconciliation Loop:** ArgoCD continuously compares the **Desired State** (Git manifests/Helm/Kustomize) with the **Live State** (what's running in the cluster). A mismatch = **OutOfSync**.

| Sync Mode | Behavior |
|-----------|----------|
| **Automated Sync** | Auto-applies changes to restore parity |
| **Manual Sync** | Flags drift in UI/CLI, waits for operator approval |

**CI/CD split of labor:** CI tools (Jenkins/GitHub Actions) build/test/scan + update the image tag in Git. **ArgoCD handles CD** — pulling changes into the cluster without exposing cluster credentials to the CI server.

### ArgoCD Architecture

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

### Rollbacks & Versioning

| Method | How It Works |
|--------|----------------|
| **Git-native rollback** (preferred) | `git revert <commit-hash>` → ArgoCD auto-detects and rolls the cluster back |
| **UI/CLI rollback** | `argocd app rollback <app-name>` |
| **Self-healing & auto-pruning** | Reverts manual `kubectl edit` drift; prunes resources removed from Git |

### Helm & Kustomize Support
- **Helm:** ArgoCD tracks a Git folder with `Chart.yaml`/`values.yaml` or a Helm repo, renders templates with value overrides
- **Kustomize:** native support for environment overlays (`overlays/dev`, `overlays/prod`)

### ApplicationSets & Multi-Cluster
- **ApplicationSets:** dynamically generate many ArgoCD `Application` resources from one template — e.g., deploy the same app across 50 regional clusters without repetitive YAML
- **Multi-cluster setup:**
  1. `argocd cluster add <kubecontext>` to register a remote cluster
  2. Specify target cluster in the Application manifest's `spec.destination`

### Security Best Practices
RBAC, SAML/OIDC Authentication, secure Git access, application secrets management, audit logs.

### ArgoCD vs. Flux — Quick Comparison

| Dimension | ArgoCD | Flux CD |
|-----------|--------|---------|
| Architecture | Centralized server, rich Web UI, SSO, granular RBAC | Decentralized, lightweight Kubernetes controllers |
| User Interface | Rich interactive Web UI with live resource trees | CLI-centric; relies on third-party UIs (e.g., Weave GitOps) |
| Best Fit | Complex enterprise setups, multi-tenant teams | Minimalist, headless environments |
| Helm Support | Advanced, built-in | Requires more setup (Helm Controller) |

### DevOps vs. GitOps

| Aspect | DevOps | GitOps |
|--------|--------|--------|
| **Scope** | Broad org culture & practices — full lifecycle | Focused specifically on deployment automation & continuous delivery |
| **Center of Gravity** | Tool-agnostic — mixes CI/CD tools, scripts, cloud platforms | Git-centric — Git is the single source of truth |
| **Configuration Model** | Declarative OR imperative | Strictly declarative (K8s YAML, Helm, Kustomize) |
| **Reconciliation** | Often push-based | Pull-based — in-cluster operator continuously reconciles drift |

> **Simple way to remember it:** DevOps = the whole philosophy/culture. GitOps = a specific *implementation* of the "CD" part of DevOps, using Git as the control mechanism.

### Rollback Strategies in DevOps (GitOps + Jenkins) — Deep Dive

**The Interview Problem:** Vague answer *"we just deploy the previous image"* isn't enough — interviewers want the **systematic automation** behind the rollback.

**Base GitOps Deployment Workflow**
```
Docker Image → Docker Hub → Archive current manifest as artifact →
Update K8s Manifest (GitHub Repo) → Flux CD → Deployment
```

**Strategy 1: Parameterized Rollback Pipeline via Registry API**
- Keep rollback as a separate, dedicated pipeline
- Accept `service_name` as a parameterized input
- Query the previous stable tag via the container registry API (e.g., Docker Hub API)
- Use `jq` on the Jenkins agent to parse the JSON response and extract the tag just before the latest build
- Commit the older image tag back into the K8s manifest repo → Flux CD reconciles and rolls back

**Strategy 2: Jenkins Archive Artifacts Snapshot**
1. In the main pipeline: before patching the K8s manifest, back up the existing manifest file via Jenkins' `archiveArtifacts`
2. On rollback trigger: pull the archived manifest from the last successful build and commit it back to Git

---

## 9. Cert-Manager

> 🧠 **Simple Summary:** Cert-Manager auto-issues and auto-renews TLS certs in Kubernetes. Flow: **Issuer** (how to get certs) → **Certificate** (what domains) → stored in a **Secret** → auto-renewed → attached to an **Ingress**.

### How It Works
1. **Install Cert-Manager** into the cluster (deploys controllers + CRDs)
2. **Issuer / ClusterIssuer:** defines *how* certificates are obtained (Let's Encrypt, Vault, internal CA)
3. **Certificate resource:** defines *what* certificate you need (domain names, which Secret to store it in)
4. **Cert-Manager requests the cert**, completes the CA's challenge (e.g., HTTP-01/DNS-01), stores the result in a Kubernetes **Secret**
5. **Automatic Renewal:** Cert-Manager watches expiry and renews certs automatically

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

## 10. Terraform

> 🧠 **Simple Summary:** Terraform provisions infra. Always use a **remote backend with locking** (S3 + DynamoDB, or Azure Blob with lease locking). Modules must expose values via `outputs.tf` to be usable elsewhere. Use `for_each` over `count` when deleting individual resources matters.

### 10.1 Why Use Terraform Modules?

| Reason | Simple Explanation |
|--------|----------------------|
| **Reusability & DRY** | Define infrastructure patterns once, reuse across Dev/QA/Staging/Prod |
| **Separation of Concerns** | Keep networking (VPC), cluster control-plane (EKS), and compute (worker nodes) in separate, decoupled directories |

### 10.2 Standard Enterprise Directory Layout

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

### 10.3 Child Module Construction

**VPC Module** — creates `aws_vpc` (DNS enabled), public/private subnets across AZs, `aws_internet_gateway`, `aws_nat_gateway` with Elastic IPs:
```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

**EKS Module** — IAM Cluster Role (`AmazonEKSClusterPolicy`) + `aws_eks_cluster`. Accepts `vpc_id`/`subnet_ids` as inputs (never hardcoded). Outputs: `cluster_name`, `cluster_endpoint`, `cluster_certificate_authority_data`.

**Worker Node Module (`ec2_node`)** — node IAM roles (`AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`) + `aws_eks_node_group`. Scaling via `min_size`/`max_size`/`desired_size`.

### 10.4 Root Module: Sourcing & Output Chaining

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  env    = var.environment
}

module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}

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

**Q: "How do you pass data between modules?"**
> A child module's internal resources are private by default. It must explicitly expose an attribute via `outputs.tf`. The root module then references it as `module.<module_name>.<output_name>`.

**Q: "Why avoid hardcoding values inside child modules?"**
> Hardcoding breaks modularity and reusability across environments. All environment-specific values must be surfaced through `variables.tf`.

### 10.5 State Management

**State File Conflict in Team** — multiple developers apply simultaneously and state gets corrupted:
```hcl
backend "s3" {
  bucket         = "my-tf-state"
  key            = "prod/terraform.tfstate"
  region         = "ap-south-1"
  dynamodb_table = "tf-lock"
}
```
On Azure: Azure Blob Storage with native lease-based locking.

**Recovering a Deleted State File:** remote backend versioning (S3/Azure Blob object versioning) or rebuild mappings using `terraform import`.

**Secrets Visible in Terraform State:** encrypt remote backends, use ephemeral outputs, delegate secret retrieval to KMS/Vault at runtime. Mark `sensitive = true` — note this does **not** remove values from the raw state file, so backend encryption still matters.

**Drift Between Infra and Code:** `terraform plan` detects drift → `terraform apply` reverts, OR `terraform import` if untracked. Enforce IaC discipline to avoid manual changes.

### 10.6 Structuring for Multiple Environments

**Approach A — Module Directory Pattern (most popular in enterprise):** shared `/modules` path + separate `/environments/dev`, `/environments/qa`, `/environments/prod` directories, each with its own `backend.tf`, `main.tf`, `terraform.tfvars`.

**Approach B — Workspace-Based Pattern:** `terraform workspace select <env>`.
> **Interview note:** Module-per-directory with separate state backends is typically preferred in production because workspaces share the same backend storage and risk cross-environment blast radius.

```bash
terraform apply -var-file=dev.tfvars
terraform apply -var-file=prod.tfvars
```

### 10.7 Unexpected Resource Destruction in `plan`

- Never run `apply` blindly — investigate what triggered replacement.
- If a resource block is renamed (`aws_db_instance.db1` → `db2`), Terraform views `db1` as deleted and `db2` as new.
  ```hcl
  moved {
    from = aws_db_instance.db1
    to   = aws_db_instance.db2
  }
  ```
- **Destruction safeguard:**
  ```hcl
  lifecycle {
    prevent_destroy = true
  }
  ```

### 10.8 Zero Downtime Resource Replacement

```hcl
lifecycle {
  create_before_destroy = true
}
```
Combine with a load balancer for smooth switchover.

### 10.9 Dependencies

```hcl
depends_on = [aws_instance.app]
```
- **Implicit dependency:** inferred from attribute references (e.g., passing `aws_vpc.main.id` into `aws_subnet.main`).
- **Explicit dependency:** manually specified when no direct reference exists but ordering is still required.

### 10.10 `count` vs `for_each`

| Scenario | Preferred Construct | Why |
|---|---|---|
| Identical config, distinct names | `count = 20` | Index addressing — simple for homogeneous arrays |
| Heterogeneous configs | `for_each = var.instances_map` | Map keys — removing an item by key doesn't re-index the rest |

**Targeted deletion pitfall with `count`:** deleting a specific indexed element can cause Terraform to destroy/recreate all subsequent indexed items.
```bash
terraform destroy -target="azurerm_linux_virtual_machine.vm[1]"
```
> **Best practice:** use `for_each` with unique map keys.

### 10.11 Conditional Resource + Dynamic Block

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

### 10.12 Rollback of a Failed Apply

Terraform is **not fully transactional**. Fix the issue, re-run `terraform apply`. Use version control for code rollback; consider blue/green for infra-level rollback.

### 10.13 Importing Existing Infrastructure

```bash
terraform import aws_instance.example i-0abcd1234efgh5678
```
Then define matching `.tf` config, run `terraform plan` to sync. In Terraform 1.5+, use declarative `import {}` blocks.

### 10.14 Data Sources vs Resources

- `resource`: declares infrastructure Terraform creates, updates, and manages.
- `data`: read-only queries to fetch info about existing infrastructure created outside the current root module.

### 10.15 Meta-Arguments & Policy-as-Code

- **Meta-arguments:** `depends_on`, `count`, `for_each`, `provider`, `lifecycle`.
- **Terraform Sentinel** (Policy-as-Code): evaluates guardrails before applying.
  - *Soft Mandatory:* warns or allows admin override
  - *Hard Mandatory:* strictly blocks `terraform apply` on failure (e.g., open port 22 to `0.0.0.0/0`)

### 10.16 Delivery Models

- **Terraform CLI in CI/CD:** runs on pipeline runners with the CLI installed; state via S3+DynamoDB or Azure Blob; credentials via OIDC/federated workload identities
- **Terraform Cloud/Enterprise:** managed remote state, VCS-driven workspaces, audit logs, private module registries

### 10.17 Terraform + Azure Integration

```
Terraform → Azure Provider → Microsoft Entra Authentication → Azure Subscription → VNet / AKS / ACR
```
```bash
terraform init
terraform validate
terraform plan
terraform apply
```
State commonly stored: `Terraform → Azure Storage Account → Blob Container → terraform.tfstate`.

### 10.18 Top 10 Terraform Scenario-Based Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | State file corrupted (multiple devs applying at once) | Remote backend with locking: S3 (state) + DynamoDB (lock table) |
| 2 | Drift — someone manually changed a resource | `terraform plan` detects it → `terraform apply` reverts, or `terraform import` if untracked |
| 3 | Managing Dev/Prod with same infra, different configs | Workspaces or separate state files + `-var-file=dev.tfvars`/`prod.tfvars` |
| 4 | Reusing VPC setup across projects | Terraform Module (`module "vpc" { source = "./modules/vpc" ... }`) |
| 5 | DB passwords exposed in code | Mark `sensitive = true`, use env vars or a secret manager |
| 6 | Zero-downtime EC2 update | `lifecycle { create_before_destroy = true }` + load balancer |
| 7 | Resource fails — dependency not ready | Terraform auto-handles most dependencies; enforce with `depends_on` |
| 8 | Managing hundreds of similar resources | `count` or `for_each` |
| 9 | Apply failed mid-way | Fix and re-run `terraform apply`; use version control; consider blue/green |
| 10 | Bring existing infra under Terraform control | `terraform import aws_instance.example i-123456`, write matching config, `terraform plan` |

### 10.19 Basic Hands-On: Terraform EC2 Example

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

---

## 11. Ansible

> 🧠 **Simple Summary:** Ansible is **push-based and agentless** (SSH/WinRM) — unlike Puppet/Chef which are pull-based and need agents. The control node **cannot run natively on Windows** (needs WSL) but **can manage** Windows targets via WinRM.

### 11.1 Configuration Management vs. Provisioning

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure provisioning (VMs, VPCs, subnets, storage) |
| **Ansible** | Configuration management — installs packages, manages config files, starts services post-provisioning |

### 11.2 Push vs. Pull Architecture

| Tool | Model | Agent Required? |
|------|-------|-------------------|
| **Ansible** | Push-based | ❌ Agentless — control node pushes changes over SSH |
| **Puppet / Chef** | Pull-based | ✅ Local agents periodically pull config from a master server |

### 11.3 OS Support
- **Linux nodes:** connected via **SSH**
- **Windows nodes:** connected via **WinRM**
- ⚠️ The Ansible **control node itself cannot run natively on Windows** (only inside **WSL**), but it *can* manage/configure Windows target nodes

### 11.4 Inventories

| Type | Description |
|------|--------------|
| **Static Inventory** | Manually defined hosts/groups/IPs (default: `/etc/ansible/hosts`) |
| **Dynamic Inventory** | Scripts (Python/cloud plugins) that query cloud providers or Terraform outputs for live host IPs |

### 11.5 Modules

| Type | Description |
|------|--------------|
| **Core Modules** | Built-in (e.g., `yum`, `apt`, `copy`, `service`, `win_copy`) |
| **Custom Modules** | User-written (commonly Python) for bespoke automation |

### 11.6 Execution Commands

```bash
# Run a playbook (verbose)
ansible-playbook -i <inventory_file> <playbook.yml> -v

# Syntax check only
ansible-playbook <playbook.yml> --syntax-check

# Ad-hoc one-liner command (no playbook needed)
ansible all -i <inventory> -m shell -a "date"
```

### 11.7 Roles — Modularizing Playbooks

| Directory | Purpose |
|-----------|---------|
| `tasks/` | Main execution steps |
| `handlers/` | Conditional tasks — run only when notified by a change |
| `vars/` & `defaults/` | Variable definitions |
| `templates/` & `files/` | Config files & Jinja2 templates |
| `meta/` | Role metadata & dependencies |

### 11.8 Ansible Vault (Secrets Encryption)

```bash
ansible-vault encrypt <secrets_file.yml>
```
Used to securely encrypt passwords, tokens, and keys inside YAML files.

### 11.9 Playbook Syntax & Structure

**Core Play Directives**
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
> Each task starts with `- name:` — indentation defines the hierarchy, so it must be exact.

**Handlers — Run Only When Notified**
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

**Loops**
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

**Variables**
```yaml
vars:
  package_name: httpd

tasks:
  - name: Install web server
    yum:
      name: "{{ package_name }}"
      state: present
```

**Tags**
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

> 🎯 **Interview Advice:** Focus on understanding the **overall architecture and syntax flow** (`hosts`, `become`, `tasks`, module structure, YAML indentation). A minor forgotten parameter is fine if the logical execution flow is correct.

---

## 12. Terraform + Ansible Together

> 🧠 **Simple Summary:** *"Terraform builds the house. Ansible furnishes and maintains it."* Terraform's outputs feed Ansible's inventory.

### 1. Division of Labor

| Tool | Role |
|------|------|
| **Terraform** | Infrastructure-as-Code — provisions VMs, VPCs, databases declaratively (HCL) |
| **Ansible** | Configuration management — connects via SSH/WinRM to install packages, configure services, deploy apps |

**Typical Workflow**
1. Terraform provisions infrastructure (EC2s, VPCs, DBs)
2. Ansible configures the software on that infrastructure
3. Terraform `outputs` (IPs, instance IDs) feed into Ansible's inventory

### 2. Example Repository Structure

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

### 3. How Ansible Gets Data From Terraform

**Option A — Manual output passing:**
```bash
terraform init
terraform apply

export instance_ip=$(terraform output -raw instance_ip)
ansible-playbook -i ${instance_ip}, playbooks/setup.yaml
```

**Option B — Dynamic Inventory (preferred at scale):**
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
> **Simple explanation:** This script asks Terraform "what did you just build?", turns the answer into a list of IPs, and Ansible uses that list as its target hosts.

### 4. Handling Multiple Environments

```bash
terraform workspace select prod
terraform apply

ansible-playbook -i inventory/prod.yaml playbooks/setup.yaml
```

### 5. Running It All in a CI/CD Pipeline

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

## 13. AWS

> 🧠 **Simple Summary:** Know NAT Gateway placement (public subnet), VPC Peering for cross-account access, and the classic scaling/DR/cost-optimization patterns (ELB+ASG, Multi-AZ RDS, Route 53 failover).

### 13.1 Core Networking & Resource Q&A

| Question | Answer |
|----------|--------|
| **Increase disk space on a Linux server?** | Two steps: expand the underlying storage (EBS volume resize) then resize the filesystem (`growpart` + `resize2fs`/`xfs_growfs`) |
| **Restrict access to specific S3 objects?** | S3 Bucket Policies or IAM policies with object-level permissions |
| **Install software on EC2 automatically at launch?** | Use **User Data** (bootstrap script executed on first boot) |
| **Where should a NAT Gateway live?** | In a **public subnet**, associated with an Elastic IP, with a route to an Internet Gateway |
| **Connect a resource in Account A to Account B?** | **VPC Peering** — create request from Requester, accept in Accepter, update route tables, adjust security groups/NACLs. Alternatives: Transit Gateway (multi-VPC scale), PrivateLink (expose specific services), VPN/Direct Connect (hybrid) |
| **Stop communication between pods in different namespaces?** | Define a `NetworkPolicy` restricting ingress/egress with `namespaceSelector`/`podSelector` |

### 13.2 Top 10 AWS Cloud Scenario Q&A

| # | Scenario | Answer |
|---|----------|--------|
| 1 | Traffic spikes 10x on a single EC2 app | ELB + Auto Scaling Groups, static content on S3 + CloudFront, RDS Multi-AZ, optional ElastiCache |
| 2 | AWS bill doubled | Cost Explorer, resize/stop underutilized EC2, Reserved Instances/Savings Plans, Auto Scaling, move cold S3 data to Glacier, right-size RDS |
| 3 | Region failure — DR strategy? | Multi-region architecture, cross-region replication (S3/RDS), Route 53 failover routing; choose by RTO/RPO: Backup & Restore, Pilot Light, Warm Standby, Active-Active |
| 4 | Store sensitive data in S3 securely | Encryption at rest (SSE-S3/SSE-KMS) + in transit (HTTPS), least-privilege IAM, block public access, versioning + MFA delete, CloudTrail monitoring |
| 5 | Microservices on ECS — how do they communicate? | Service discovery (AWS Cloud Map), internal load balancer, SQS (decoupling)/SNS (fan-out), IAM roles + VPC networking |
| 6 | Logs scattered, app fails randomly | Centralize with CloudWatch Logs + Log Insights, alarms (CPU/memory/error rate), X-Ray tracing, dashboards |
| 7 | RDS slow due to heavy reads | Read Replicas, ElastiCache, optimize queries/indexing, consider Aurora, connection pooling |
| 8 | Automate deployment (CI/CD design) | CodeCommit → CodeBuild → CodeDeploy → CodePipeline; add approval steps + rollback strategy |
| 9 | Migrate EC2 monolith to serverless | Break into microservices, Lambda for compute, API Gateway for APIs, DynamoDB/S3 for data, event-driven (SQS/SNS) |
| 10 | Different teams need different access levels | IAM roles/policies with least privilege, IAM groups, MFA, AWS Organizations for multi-account governance |

---

## 14. Azure

> 🧠 **Simple Summary:** Hierarchy: **Tenant → Management Groups → Subscriptions → Resource Groups → Resources**. Entra ID = *who you are* (AuthN). Azure RBAC = *what you can do* (AuthZ). Prefer Managed Identity over Service Principal secrets.

### 14.1 Foundations

Key services for a DevOps engineer: **AKS, ACR, Virtual Network, Key Vault, Managed Identity, Azure Monitor, Log Analytics, Storage, Azure RBAC**.

**Resource Group:** a logical container for Azure resources sharing the same lifecycle/administration boundary — a **management boundary**, not necessarily a network boundary.

**Subscription hierarchy:**
```
Azure Tenant → Management Groups → Subscriptions → Resource Groups → Resources
```
Provides: billing boundary, resource boundary, access-control boundary, quota boundary.

### 14.2 Identity, RBAC & Managed Identity

**Microsoft Entra ID** (formerly Azure AD): identity/authentication (**AuthN**) — Users, Groups, Applications, Service Principals, Managed Identities.

**Azure RBAC:** what an identity is authorized to do (**AuthZ**) — e.g., `Reader`, `Contributor`, `Owner`, `AcrPush`, `AcrPull`.
```
GitHub Actions → OIDC authentication → Entra ID → Azure RBAC → ACR / AKS / Storage
```

**Managed Identity — very important**
Allows Azure resources to authenticate to other Azure services without storing credentials.
- **System-assigned**: created with the resource; deleted when the resource is deleted.
- **User-assigned**: created separately, can be assigned to multiple resources.

**Managed Identity vs Service Principal**
- **Service Principal:** app registration needing manual credential/secret rotation.
- **Managed Identity:** Azure-managed credential, auto-rotated by Entra ID.

**How would an app on AKS access Key Vault?**
```
Application Pod → AKS Workload Identity → Microsoft Entra ID → Azure RBAC → Key Vault
```

### 14.3 Key Vault & Secrets

Securely stores Secrets, Encryption keys, Certificates. Avoid putting these into Git, Docker images, Terraform code, or K8s YAML — use Key Vault + an identity/access mechanism instead.

### 14.4 Networking

**Virtual Network (VNet):**
```
VNet
 +---- Subnet A
 +---- Subnet B
 +---- Subnet C
```

**Can an existing subnet be extended from `/24` to `/23`?** **No** — you cannot change the IP range of an existing Azure subnet while resources/NICs are attached to it.

**NSG (Network Security Group):** controls inbound/outbound traffic via rules.
**NSG vs Azure Firewall:** NSG = basic filtering at subnet/NIC level; Azure Firewall = managed, centralized, advanced.

**Load Balancer vs. Front Door vs. Traffic Manager vs. App Gateway**

| Type | Layer | Scope |
|------|-------|-------|
| **Load Balancer** | L4 (TCP/UDP) | Regional |
| **Application Gateway** | L7 (HTTP/HTTPS) | Regional — URL/host routing, TLS termination, WAF |
| **Traffic Manager** | DNS-level | Global (no data path involvement) |
| **Front Door** | L7 edge | Global — CDN, TLS termination, health-based routing, WAF, actively proxies traffic |

**Private Endpoint vs Service Endpoint**
- **Service Endpoint:** extends VNet identity to Azure services over Azure's backbone.
- **Private Endpoint:** provides a private IP in your VNet — preferred for enterprise private connectivity.

**VNet Peering — VNet B can't access VNet A:** non-transitive peering limitations, asymmetric/missing peering (must be set up on **both** sides), overlapping CIDRs, restrictive NSG rules.

**On-Premises to Azure connectivity options:** Site-to-Site VPN, Point-to-Site VPN, dedicated private line via **Azure ExpressRoute**.

### 14.5 Compute & Storage

**Azure Managed Disk:** tiers — Standard HDD, Standard SSD, Premium SSD, Ultra Disk.

**VM Scale Sets (VMSS):** deploy/manage a group of load-balanced VMs — scaling, HA, automated updates.

**Azure Storage Types:** Blob (object storage), Files (managed file shares), Queue (messaging), Table (NoSQL key-value).

### 14.6 AKS-Specific Azure Q&A — see [Section 6.13](#613-aks-azure-kubernetes-service-specific)

### 14.7 Application Gateway Troubleshooting

Backend health down — common culprits: mismatched health probe path/port, backend NSG blocking App Gateway subnet traffic, SSL cert mismatch, backend listening on `localhost` instead of `0.0.0.0`.

### 14.8 Azure Policy & Governance

**Types of Effects:** `Deny`, `Audit`, `Modify`, `DeployIfNotExists`.

**Azure Policy vs RBAC**
- **RBAC** controls **who** can do **what**.
- **Policy** controls **what configurations/resources are allowed or required**.

**Azure Landing Zone:** a structured foundation addressing Identity, Networking, Governance, Security, Subscriptions, Policies, Logging, Management — especially relevant for regulated enterprises.

### 14.9 HTTP Status Codes

| Code | Meaning |
|------|---------|
| **500** | Internal Server Error — unhandled error in the application |
| **502** | Bad Gateway — upstream returned an invalid response |
| **503** | Service Unavailable — overloaded / down for maintenance |
| **504** | Gateway Timeout — upstream didn't respond in time |

### 14.10 CI/CD in Azure

```
GitHub → GitHub Actions → Tests + Security → Docker Build → ACR → GitOps (Flux) → AKS → Pods
```

**GitHub Actions → Azure authentication (OIDC):**
```yaml
- name: Azure Login
  uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```
> Why OIDC instead of client secrets? A client secret is long-lived — if leaked, an attacker has standing access. OIDC provides short-lived federated authentication.

### 14.11 Monitoring & Observability in Azure

**Azure Monitor vs Log Analytics**
- **Azure Monitor:** broad platform — Metrics, Logs, Alerts, Application telemetry.
- **Log Analytics:** workspace for querying logs using **KQL**:
  ```kql
  AzureActivity
  | where TimeGenerated > ago(1h)
  | summarize count() by ResourceGroup
  ```

**Azure Application Insights:** APM capability — monitors Requests, Response times, Exceptions, Dependencies, Availability.

**Monitoring AKS:** Kubernetes-level (`kubectl top pods`, `kubectl top nodes`) + Azure-level (Azure Monitor, Container Insights, Log Analytics, Application Insights).

### 14.12 Common Azure Troubleshooting Scenarios

**AKS app cannot access Key Vault**
```
Pod → Workload Identity / Managed Identity → Entra ID → Azure RBAC → Key Vault
```
Checklist: correct identity → federated correctly → has Key Vault permissions → network accessible → DNS resolving → secret name correct → audit logs → recent changes.

**AKS can't pull image from ACR**
```
AKS → Identity → RBAC → ACR → Image
```
```bash
kubectl describe pod <pod>
```
Look for `ImagePullBackOff`/`ErrImagePull` → check image name/tag, ACR availability, AKS identity, `AcrPull` permission, network/DNS.

**Terraform pipeline cannot authenticate to Azure**
```
Pipeline → Authentication method → Entra ID → Service principal / Workload Identity → Azure RBAC → Subscription
```
Verify: OIDC permission, federated credential, Client ID, Tenant ID, Subscription ID, repo/branch/environment conditions, RBAC permissions. `az account show` is a good first check.

### 14.13 Disaster Recovery Concepts (Azure)

- **RPO (Recovery Point Objective):** how much data loss is acceptable.
- **RTO (Recovery Time Objective):** how quickly the system must be restored.
- **Region vs Availability Zone:** Region = geographic location; Availability Zone = physically separate datacenter within a region. Deploying across zones improves availability.

### 14.14 Azure Security Architecture (End-to-End)

```
Identity → Entra ID → RBAC → Network → VNet / NSG / Firewall → Application → AKS security → Secrets → Key Vault → Monitoring → Azure Monitor / Sentinel
```

**How would you secure an AKS cluster?** Identity-based access through Entra ID, Kubernetes RBAC + Azure RBAC, private networking, managed/workload identities instead of static credentials, Key Vault for secrets, network policies, image scanning, least-privilege, node/cluster patching, monitoring/audit logging.

---

## 15. Linux & Shell Scripting

> 🧠 **Simple Summary:** Know `grep`/`sed`/`awk`, process/service management, and be ready to **write** a script live (check file exists, count files, background a process) — not just explain it.

### File Inspection & Logs

| Command | Purpose |
|---------|---------|
| `head -n <N> <file>` / `tail -n <N> <file>` | Read first/last N lines |
| `tail -f <file>` | Live-stream newly appended lines |

### Text Processing

| Tool | Purpose |
|------|---------|
| `sed` | Inline substitutions/deletions without opening an editor |
| `awk` | Record/field-based text manipulation |
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
| `top` / `htop` | Interactive real-time CPU/process monitoring |
| `free -m` / `free -g` | RAM & swap usage (MB/GB) |
| `ps -eo pid,etime,cmd` | Check long-running processes |

### Archiving, Compression & Transfer

| Command | Purpose |
|---------|---------|
| `zip -r <archive.zip> <folder>` | Recursively compress a folder |
| `unzip <archive.zip>` | Extract a zip archive |
| `tar -cvf <name.tar> <folder>` | Create a tar archive |
| `tar -xvf <name.tar>` | Extract a tar archive |
| `scp /local/path user@remote:/dest/path` | Securely copy files between servers over SSH |

### Practical Automation Use Cases

| Use Case | What It Does |
|----------|----------------|
| **Cost Optimization Scripts** | Auto-shutdown non-prod instances outside business hours; restart before workday |
| **Mass Maintenance / Coordinated Restarts** | Rolling restarts across servers, verifying health checks before moving to the next node |
| **Log Rotation & Disk Space Remediation** | Cron scripts detect `/var/log > 85%`, archive to S3, purge stale cache; use `logrotate` |
| **CrashLoopBackOff / OOM Remediation** | Scripts triage stuck pods, extract exit codes (`OOMKilled`/Exit 137), dump logs, notify on-call |

### Common Live-Coding Scripts

**Count files containing a specific word:**
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

**Directory check + file count:**
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

**Find log files older than 7 days:**
```bash
find /var/log -type f -name "*.log" -mtime +7
```

**Run a command in the background:**
```bash
nohup command &
```

### 🎯 Interview Strategy Tips
- **Think out loud** — never code in silence; narrate your logic as you go
- Even imperfect syntax under pressure is fine if your reasoning is sound — it shows problem-solving skill

---

## 16. SQL

> 🧠 **Simple Summary:** For "Nth highest" problems: sort descending, `LIMIT N`, then flip to ascending and take the first row.

### Find the 7th Highest Marks

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

---

## 17. SonarQube & DevSecOps Scanning

> 🧠 **Simple Summary:** 🧹 Code Smell = "works but ugly/risky." 🐞 Bug = "broken." 🔓 Vulnerability = "a door left open for attackers." Quality Gates block merges automatically.

### The Core Comparison

| Category | Primary Impact | Definition | What Happens to the System |
|----------|------------------|------------|------------------------------|
| **Code Smell** | Maintainability & Readability | Sub-optimal design or poor practices | Program still works but is messy, fragile, hard to refactor |
| **Bug** | Reliability & Correctness | Flaws, runtime exceptions, logic errors | Program crashes, throws exceptions, or produces wrong calculations |
| **Vulnerability** | Security & Integrity | Security flaws / exposed entry points | System may run fine, but data/access is exposed to exploitation |

### 🧹 Code Smells — Examples & Fixes

| Smell | Problem | Fix |
|-------|---------|-----|
| **Hardcoded Values & Endpoints** | Any change needs a recompile/rebuild/redeploy | Externalize into env vars, config files, or parameter stores |
| **Deeply Nested Control Logic** (Arrow Anti-Pattern) | Multiple nested `if/else` hurt readability & testing | Guard clauses, early returns, polymorphism |
| **Bloated Functions** | One function doing too much | Break into small, modular, reusable helper functions |
| **Duplicated Code** | Bug fixes need to happen in every copy | Extract shared logic into reusable methods/libraries |

### 🐞 Bugs — Examples

| Bug Type | Example |
|----------|---------|
| **Off-by-One / Index Out of Bounds** | Array of size 4 accessed at index `4` → `ArrayIndexOutOfBoundsException` |
| **Divide-by-Zero** | Division without checking if the denominator is `0` |
| **Scope & Uninitialized Variables** | Variable declared in an inner block, referenced outside → `NullPointerException` |

### 🔓 Vulnerabilities — Examples & Remediations

| Vulnerability | Example | Remediation |
|----------------|---------|--------------|
| **Hardcoded Credentials** | Credentials in cleartext (`-u username -p password`) in scripts/config | Inject secrets at runtime via Vault, AWS Secrets Manager, Azure Key Vault |
| **SQL Injection (SQLi)** | `SELECT * FROM users WHERE id = ' + input + '` | Parameterized queries / `PreparedStatement`s, ORM with input sanitization |
| **Broken Authentication & Input Validation** | Processing requests without verifying identity | Enforce OAuth2/JWT verification, strict RBAC, origin whitelisting |

### 🎯 Interview One-Liner: Enforcing Quality Across Teams

> "In our CI/CD pipeline, every Pull Request triggers a Maven build coupled with a SonarQube scan. We configure strict **Quality Gates**: any PR that introduces Blocker/Critical Bugs, Security Vulnerabilities, or drops Code Coverage below 80% automatically **fails the Quality Gate** — blocking container artifact generation until issues are resolved."

### Scan Types in a CI/CD Pipeline

| Scan Type | Purpose |
|-----------|---------|
| Unit Testing / Code Quality | SonarQube — static analysis, code smells, coverage thresholds |
| **SAST** (Static App Security Testing) | Scans source code for security flaws before build |
| **DAST** (Dynamic App Security Testing) | Tests running applications/endpoints for runtime/API vulnerabilities |
| **SCA** (Software Composition Analysis) | Scans dependencies/third-party libraries (e.g., Snyk) |
| Container Image Scanning | Scans layers/OS packages for CVEs before registry push (Trivy, Aqua) |

**Full CI/CD scan order:** SAST (source code) → SCA (dependencies) → Container scan (image CVEs) → DAST (running app)

---

## 18. Monitoring & Logging (Prometheus/Grafana/EFK)

> 🧠 **Simple Summary:** Metrics = numeric trends (Prometheus). Logs = discrete text events (EFK). Prometheus scrapes → Grafana visualizes → Alertmanager routes alerts to Slack/PagerDuty.

### Common Tools

| Category | Tools |
|----------|-------|
| Metrics + Dashboards | Prometheus + Grafana |
| Cloud-native | AWS CloudWatch, Azure Monitor, Google Stackdriver |
| Logging | ELK / EFK (Elasticsearch, Fluentd/Fluent Bit, Kibana) |
| SaaS platforms | Datadog, New Relic, Dynatrace, Nagios |

### 🆚 Metrics vs. Logs

| | **Metrics** | **Logs** |
|---|---|---|
| Type | Time-series numeric data | Discrete, timestamped text events |
| Examples | CPU %, memory, pod restarts, HTTP request rate | API request hit, DB query, stack trace error |
| Best for | Trends, thresholds, dashboards | Root-cause / forensic debugging |

### a) Metrics: Prometheus + Grafana

```bash
kubectl create ns monitoring
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```
> `kube-prometheus-stack` bundles the Prometheus Operator, node-exporter, and kube-state-metrics.

**Connect Grafana → Prometheus:** Data Sources → Prometheus → enter server URL (`http://prometheus-server.monitoring.svc.cluster.local:9090`) → build dashboards.

### b) Logging: EFK Stack

```bash
helm install elasticsearch elastic/elasticsearch
helm install kibana elastic/kibana
```
- **Fluentd/Fluent Bit** runs as a **DaemonSet** on every node — tails `stdout`/`stderr` logs from `/var/log/containers/*.log`, enriches with pod/namespace metadata, ships to Elasticsearch.

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
- **Kibana** is the UI to search/filter/visualize logs (Lucene/KQL, trace errors, inspect stack traces).

### c) Alerts & Notifications

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

- **Alertmanager:** receives firing alerts from Prometheus, deduplicates/groups them, routes to Slack, PagerDuty, or Email.
- **Grafana Alerts:** can also be configured directly on dashboard panels.

### d) Architecture Diagram

```
[ Target Pods / Nodes ]
        │ (exposes /metrics)
        ▼
[ Prometheus Server ] ──(scrapes)──► Stores in TSDB
        │
        ├──► [ Grafana ]        ──► Dashboards
        └──► [ Alertmanager ]   ──► Slack / PagerDuty / Email
```

### 🎯 Interview One-Liner

> "We run `kube-prometheus-stack` in a dedicated `monitoring` namespace via Helm. Prometheus scrapes pod metrics, Grafana visualizes them, and Alertmanager routes high-severity alerts (e.g., CrashLoopBackOff) to Slack. For logs, Fluent Bit ships container logs to Elasticsearch/Kibana (or CloudWatch Container Insights), letting us debug without SSH access to nodes."

---

## 19. SRE & Observability

> 🧠 **Simple Summary:** SRE = DevOps with numbers. **SLI** = what you measure. **SLO** = the target. **Error Budget** = how much unreliability is allowed before you stop shipping features and fix reliability instead. In an outage: mitigate first, communicate second, RCA third.

### Handling a Midnight Production Outage

1. **Priority 1 — Fast mitigation over deep analysis:** capture diagnostic snapshots, then mitigate fast (rollback latest release, failover to backup region/cluster, restart degraded pods)
2. **Priority 2 — Incident communication:** open an incident war-room/channel, notify on-call leads/stakeholders, provide regular status updates
3. **Priority 3 — Post-recovery RCA:** document the exact timeline, correlate telemetry spikes, preserve logs for RCA

### Reducing MTTR

- **Granular observability:** high-resolution metrics (Prometheus/Grafana) at short scrape intervals
- **Automated runbooks/playbooks:** standardized responses to frequent failures, self-healing scripts
- **Smaller, incremental deployments:** small batch releases + canary rollouts → smaller, faster rollback blast radius

### Troubleshooting High Latency

- **Layer-by-layer diagnostics:** Network/DNS → Ingress/LB → Application logic → Downstream DBs/caches → External 3rd-party APIs
- **Distributed tracing:** Jaeger, Zipkin, OpenTelemetry, AWS X-Ray to pinpoint the slow span

### Debugging Intermittent Microservice Failures

- Inspect APM/logs for error spikes, retry storms, socket exhaustion, timeouts
- **Resilience patterns:** Circuit Breakers (Resilience4j, Envoy/Istio); Exponential Backoff with Jitter on retries
- **Reproduction:** elevate debug logs, simulate peak traffic in staging

### Designing for High Availability

- Multi-AZ / Multi-Region, active-active or active-passive failover
- **Stateless service design** — decouple compute from persistent storage
- Enforce strict timeouts/deadlines to prevent thread pool exhaustion
- Data layer protection: async read replicas, multi-region replication, automated snapshots

### Observability, Alert Fatigue & RCA

- **Three Pillars of Observability:** Metrics (health/trends), Logs (discrete events), Traces (request flow)
- **Reduce alert fatigue:** deprecate noisy non-actionable alerts; alert on **user-facing SLO symptoms**, not raw CPU spikes that self-resolve
- **Blameless post-mortem:** link metric timestamps to trigger events, identify systemic contributing factors, track preventative action items

### SRE vs. DevOps

> **SRE implements DevOps** through concrete engineering metrics:
- **SLIs** (Service Level Indicators) — what you measure
- **SLOs** (Service Level Objectives) — the target for that measurement
- **Error Budgets** — how much unreliability is acceptable before you must slow down feature work and focus on reliability

---

## 20. RBAC & Security (Cloud + Kubernetes)

> 🧠 **Simple Summary:** Cloud RBAC (Azure RBAC / AWS IAM) secures **infrastructure**. Kubernetes RBAC secures **workloads inside the cluster**. You need both. Never assign access directly to users — always use groups.

### Azure RBAC vs. AWS IAM vs. Kubernetes RBAC

| Layer | Controls |
|-------|-----------|
| **Azure RBAC** | Access to Azure cloud resources (subscriptions, resource groups, VMs, AKS **infrastructure**) |
| **AWS IAM** | Identities & permissions for AWS resources (EC2, S3, EKS) |
| **Kubernetes RBAC** | Permissions **inside** the cluster (namespaces, pods, deployments, secrets) |

> **Simple way to remember it:** Cloud RBAC secures the *infrastructure* layer. Kubernetes RBAC secures the *workload* layer inside the cluster.

### Common Governance Interview Q&A

| Question | Answer |
|----------|--------|
| **How do you ensure least-privilege access?** | RBAC across Azure/AWS/K8s; minimum permissions per role/namespace; avoid cluster-admin; Just-In-Time (JIT) elevated access; use groups not direct grants; review access regularly; revoke immediately on offboarding |
| **How do you do user audit & logging?** | Centralized logging: Azure Activity Logs/Monitor/Sentinel, AWS CloudTrail/CloudWatch/GuardDuty, Kubernetes API server audit logs — all feeding into a central SIEM (Sentinel, Splunk, ELK) |
| **How do you handle access removal when a dev switches teams?** | Centralized IAM (Entra ID / AWS IAM Identity Center) with **group-based** RBAC — never assign directly to users. Move the user between groups → auto-updates Azure RBAC, AWS IAM roles, and K8s RoleBindings. Validate via audit logs and periodic reviews |
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

## 21. OpenShift

> 🧠 **Simple Summary:** OpenShift = Kubernetes + enterprise features (built-in CI/CD, enhanced security, monitoring, registry, web console, Operators). `oc` is its CLI (like `kubectl`, with extras).

### What is OpenShift?
> Red Hat OpenShift is an enterprise container platform built **on top of Kubernetes**. Adds developer tools, built-in CI/CD, enhanced security, monitoring, an image registry, web console, and an Operator framework.

### Core Concepts

| Concept | Description |
|---------|--------------|
| **Container** | Packages app code, runtime, libraries, dependencies (Docker is the common engine) |
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

---

## 22. Cloud Migration & Disaster Recovery

> 🧠 **Simple Summary:** The **6 Rs** framework: Rehost, Replatform, Refactor, Repurchase, Retain, Retire. For near-zero-downtime DB migrations, use continuous replication (AWS DMS / CDC) and keep the old system as a warm rollback target.

### The 6 Rs of Migration

| R | Meaning | Example |
|---|---------|---------|
| **Rehost** (Lift & Shift) | Move workloads as-is, no redesign | Azure VM → AWS EC2 with matching vCPU/memory/OS |
| **Replatform** (Lift, Tinker & Shift) | Minor optimizations using cloud-managed services | Self-hosted DB → Amazon Aurora/RDS |
| **Refactor / Re-architect** | Re-engineer into cloud-native architecture | Monolith → microservices on containers/serverless |
| **Repurchase** | Replace custom software with off-the-shelf SaaS | — |
| **Retain** | Keep non-migratable/compliance-bound components on-prem | — |
| **Retire** | Decommission obsolete servers/services | — |

### Cross-Cloud Service Mapping: Azure → AWS

| Azure | AWS Equivalent | Notes |
|-------|------------------|-------|
| Azure Kubernetes Service (AKS) | Amazon EKS | Node groups, CNI networking, pod manifests |
| Azure PostgreSQL Flexible Server | Amazon Aurora / RDS | Aurora for high performance + managed scaling |
| Management Groups & Resource Groups | AWS Organizations & Member Accounts | Resource Groups vs. separate Accounts + IAM boundaries |
| Network Security Groups (NSGs) | Security Groups / NACLs | Subnet routing, VPC peering, endpoint access |

### AWS → Azure Service Mapping

| Domain | AWS Service | Azure Equivalent | Migration/Cutover Consideration |
|---|---|---|---|
| Compute/Containers | Amazon EKS | Azure Kubernetes Service (AKS) | Standardize manifests via Helm |
| Object Storage | Amazon S3 | Azure Blob Storage | Replicate via AzCopy or Azure Data Factory |
| Databases | Amazon RDS/Aurora | Azure Database for PostgreSQL/MySQL/SQL | Dual-write or continuous replication via Azure DMS |
| DNS & Routing | Amazon Route 53 | Azure DNS / Traffic Manager | Weighted DNS routing for progressive traffic split |
| Observability | Amazon CloudWatch | Azure Monitor & Log Analytics | Migrate log shippers (Fluent Bit) to Log Analytics |
| Secrets & Identity | AWS Secrets Manager / IAM | Azure Key Vault & Microsoft Entra ID | Transition IAM roles to Managed Identities |
| IaC | AWS-provider Terraform | AzureRM-provider Terraform / Bicep | Re-architect Terraform modules |

### Practical Migration Workflow (e.g., Java Microservices: Azure → AWS)

1. **Pre-Migration Backups** — full snapshots of source databases and storage volumes
2. **Database Migration via AWS DMS:** set up replication instance, ensure network reachability, define table mapping, run full-load + CDC tasks
3. **Application & Cluster Provisioning:** provision target EKS clusters, VPCs, subnets, node groups via IaC; deploy Helm charts, validate pod health
4. **Staging & Production Cutover:** run the migration runbook in a sandbox first → schedule maintenance window → apply final delta syncs → re-point DNS/LB → monitor live telemetry

### Real-World Migration & DR Scenarios

**Kubernetes Storage Migration (AWS EBS gp2 → gp3), max 2 min downtime**
1. **Pre-Migration Safety & Baseline:** snapshot via AWS Backup/EBS Snapshots; log row counts, schema checksums, disk metrics
2. **Data Copy & Delta Sync:** provision new gp3 volume; attach both volumes to a migration pod or run async replication while DB stays online
3. **The 2-Minute Cutover Window:** stop write traffic → shut down PostgreSQL pod → final delta sync → update PVC/StatefulSet binding → restart pod on gp3
4. **Validation:** automated health queries, monitor CloudWatch EBS IOPS/disk queue metrics
5. **Rollback:** don't delete the old gp2 volume immediately — re-point PVC back if health checks fail

> **Tip:** AWS EBS Elastic Volumes can modify gp2→gp3 on-the-fly without detaching, potentially avoiding a manual cutover window.

**Entire Data Center Migration to AWS (400 VMs, 6-month deadline)**
```
Step 1: Landing Zone Setup → Step 2: Discovery & Dependency Mapping → Step 3: Phased Wave Execution → Step 4: DNS Cutover & Fallback
```
- **Landing Zone:** multi-account setup (AWS Organizations), VPCs, subnets, Direct Connect, domain controllers to extend Active Directory
- **Discovery:** map app-to-database connections (IIS→SQL Server, Java→Oracle)
- **Waves:** Dev/Test → Staging/UAT → Mission-critical Production
  - **Compute:** AWS MGN (block-level replication to EC2)
  - **Databases:** AWS DMS + SCT (CDC replication to RDS/Aurora)
  - **File Storage:** AWS DataSync over Direct Connect to FSx/EFS
- **Cutover & Rollback:** test EC2 instances from replicated volumes; lower DNS TTL in advance; keep on-prem infra warm/read-only-standby for a soak period

**Cross-Cloud Migration (AWS → Azure, zero downtime, 12 months)**
1. **Discovery & Landing Zone:** provision Azure Management Groups, Subscriptions, VNets, ExpressRoute/VPN
2. **Workload Deployment:** deploy to AKS; continuous replication of databases and blob storage
3. **Phased Traffic Shift:** weighted DNS routing 5% → 25% → 100%
4. **Fallback Safety Net:** keep AWS running passively until Azure stability confirmed

**Database Migration with Near-Zero Downtime (MySQL → Aurora, 25TB, 5-min window)**
- AWS DMS for schema conversion + continuous CDC replication while source stays live
- During the 5-minute window: pause/queue writes → final CDC delta → switch connection strings → resume traffic
- **Validation:** compare row counts/checksums, verify zero replication lag, smoke tests
- **Rollback:** revert connection strings to original MySQL-on-EC2 (kept warm, not decommissioned immediately)

**Multi-Region Disaster Recovery (AWS, complete regional outage survival)**
```
                          Route 53 DNS (Failover / Health Checks)
                                      │
            ┌─────────────────────────┴─────────────────────────┐
            ▼                                                   ▼
 Primary Region (Mumbai - ap-south-1)          Secondary DR Region (e.g., ap-southeast-1 / us-east-1)
 ├── CloudFront Edge Caching                   ├── Standby / Active CloudFront
 ├── EKS Cluster (Active Workloads)            ├── Replicated EKS Cluster (via Terraform/IaC)
 ├── Amazon RDS (Primary Read/Write) ─────────►├── Amazon RDS (Cross-Region Read Replica)
 ├── Amazon S3 (Primary Bucket) ──────────────►├── Amazon S3 (Cross-Region Replication - CRR)
 └── Regional Redis Cache (Independent)        └── Regional Redis Cache (Independent Warm Cache)
```
- **Infrastructure Replicability (IaC):** parameterized Terraform/CloudFormation modules deploy identical topology in the secondary region
- **Global Ingress & DNS Failover:** Route 53 Failover Routing + Health Checks; CloudFront origin failover groups
- **Database & Storage Sync:** RDS/Aurora async Cross-Region Read Replica (promote to master on failover); S3 CRR with versioning; Redis runs as **independent** per-region caches
- **User Session Continuity:** stateless JWTs or distributed session state, not localized sticky sessions
- **CI/CD Deployment Strategy:** Active-Active (deploy to both clusters simultaneously) or Active-Passive (secondary pre-provisioned, ready to scale on incident)

### Cloud Migration Strategy Steps (General)

1. **Assessment:** inventory stateful data, DBs, compute dependencies
2. **Database migration:** managed replication (e.g., AWS DMS) for continuous schema/data sync with minimal downtime
3. **Compute/storage replication:** AWS MGN — replication agents on source VMs for continuous block-level sync
4. **Validation & cutover:** validate data parity, smoke test in staging, shift traffic via Route 53 weighted records, decommission legacy instances

---

## 23. DevOps Culture, Maturity & SDLC

> 🧠 **Simple Summary:** DevOps Maturity = automation + collaboration + delivery speed + observability. 12-Factor App = rules for cloud-native apps (config/code separation, stateless, disposable). SRE = DevOps measured with SLIs/SLOs/Error Budgets.

### DevOps Maturity Model
Evaluates organizational transformation across **automation, cross-team collaboration, continuous delivery, and observability**. Higher maturity = faster release cadence with minimal manual intervention.

### 12-Factor App Methodology
Principles for building cloud-native SaaS apps:
- Strict config/code separation
- Stateless processes
- Backing service abstraction
- Disposability (fast startup/graceful shutdown)
- Dev/prod parity

### SRE vs. DevOps
SRE implements DevOps through concrete engineering metrics: **SLIs**, **SLOs**, and **Error Budgets** (see [Section 19](#19-sre--observability)).

### Artifact Promotion: "Build Once, Promote Everywhere"

**The interview dilemma:** *Is the exact same tested artifact promoted to production, or does a new pipeline rebuild it from `main`?*

**Approach A: Rebuild Per Target Pipeline**
- Artifact built & tested during Dev/QA runs; on merge to `main`, a separate production pipeline rebuilds from that branch; smoke/regression tested in staging; manual approval gate before going live.

**Approach B: "Build Once, Promote Everywhere" (12-Factor Standard) ✅ Industry preferred**
- An **immutable artifact** is built **only once**, from the commit SHA. The **same image digest** flows through Dev → QA/Staging → Production — never rebuilt. Environment differences handled purely through **externalized config** (ConfigMaps, Secrets, env vars).

| | Rebuild Per Pipeline | Build Once, Promote Everywhere |
|---|---|---|
| Risk | Possible drift between builds | No drift — same binary everywhere |
| Speed | Slower (redundant builds) | Faster |
| Industry standard? | Sometimes used | ✅ Preferred / 12-Factor standard |

> **QA/Pre-Prod parity matters:** Staging must mirror production (networking, DB engines, ingress rules) — otherwise config drift can hide environment-specific failures.

### 🎯 Interview One-Liner
> "We build the Docker image once in CI, tag it with the immutable Git commit SHA, and push it to ECR. That exact image digest is deployed to QA, passes integration tests, and is promoted to Production behind a manual approval gate — only ConfigMaps and Secrets change between environments."

### Event-Driven Architecture in DevOps

**Definition:** Actions/workflows triggered automatically by *events* rather than manual steps or fixed schedules.

| Trigger Event | Automated Response |
|----------------|----------------------|
| Code pushed to Git | CI/CD pipeline starts automatically |
| High traffic load | Auto-Scaling Group / Kubernetes HPA launches new instances/pods |
| Failed deployment | Automatic rollback to the previous stable version |
| System outage / high error rate | Alerts fire + automated remediation |

**How to defend this on your resume:**

| Mechanism | Example |
|-----------|---------|
| CI/CD Triggers | Git webhook → Jenkins/GitHub Actions run instantly on push/PR |
| Dynamic Autoscaling | CloudWatch/Prometheus threshold → triggers ASG or K8s HPA/Karpenter |
| Self-Healing & Rollback | Health-check failure → automated rollback, container restart, or traffic shift at LB level |

### QA Pipeline Ownership

> The **DevOps engineer** designs, provisions, and maintains the pipeline infrastructure/code. The **QA team** contributes automated test scripts (Selenium/Cypress/Playwright) that run *inside* a pipeline stage.

**Standard QA Pipeline Flow:** Git checkout → build/compile → static analysis & security scan (SonarQube) → automated integration/e2e tests → publish test reports & notify build status.

**Do you rebuild the image for each environment?** **No.** Build once, tag with commit SHA, push to registry, deploy that same digest everywhere.

---

## 24. Career, Behavioral & Company-Specific Prep

> 🧠 **Simple Summary:** For career-switchers: get 2-4 months of internal cross-skilling if possible, or build hands-on labs otherwise. Use the STAR method for behavioral answers. Always negotiate transparently if you're holding multiple offers.

### Career Transition Guide: Switching Into DevOps

| Route | Approach |
|-------|----------|
| **A. Internal Transition** (fastest legitimacy) | Ask your manager for release/partial allocation to an internal DevOps/Cloud project. Even 2–4 months of cross-skilling gives real, defensible production context |
| **B. Self-Learning / External Switch** | Use hands-on labs (KodeKloud), AWS/Azure free tier, self-host tools (EC2, Jenkins, Docker, Minikube/kind/EKS) to build end-to-end projects |

**Three Core Interview Questions Career-Switchers Must Nail**

1. **"What are your exact roles and responsibilities?"** → Anchor around tools you're actually solid in
2. **"What does a typical day look like for you?"** → standup → monitoring alerts → root-cause investigation → building automation
3. **"What tools and cloud technologies do you own?"** → Be specific, don't generalize

**High-Priority Tools to Focus On**

| Area | Tools |
|------|-------|
| Kubernetes Orchestration | EKS/AKS, Pods, Deployments, Services, Helm |
| CI/CD & Containerization | Jenkins / GitHub Actions / GitLab CI, multi-stage Dockerfiles, ECR |
| DevSecOps | SonarQube (SAST), OWASP Dependency-Check (SCA), HashiCorp Vault / AWS Secrets Manager |
| IaC & Scripting | Modular Terraform, Shell/Python automation |

> **Overcoming Imposter Syndrome:** Every engineer faces a ramp-up curve on a new codebase/infra/team. After **30–60 days**, the rhythm becomes natural.

### A Day in the Life of a DevOps Engineer

**Teams a DevOps Engineer interfaces with:** Core Product Development, Internal Platform/Automation, Dev & QA Infrastructure, Customer & Production Operations, SRE/Support.

**Sample Daily Schedule**
```
[ 09:00 - 09:30 ]  ──► System checks, monitoring review, email alerts, Jira backlog
[ 09:30 - 10:00 ]  ──► Daily Standup (yesterday / today / blockers)
[ 10:00 - 13:00 ]  ──► P0/P1 priority work (pipeline debugging, hotfixes, unblocking devs)
[ 14:00 - 17:00 ]  ──► Core project execution (Terraform modules, CI/CD refactoring, scripts)
[ 17:00 - 18:00 ]  ──► Cross-team syncs, documentation, runbook updates
```

**The "Automate Recurring Issues" Principle**
> If an issue happens **once**, document it. If it happens **twice**, **automate the fix**.

### 🎯 Interview One-Liner
> "My day starts with reviewing monitoring alerts and our Jira board for high-priority blockers. During standup, I align with developers on sprint deliverables. The core of my day splits between project automation — writing Terraform modules, optimizing CI/CD stages, refining Helm templates — and platform maintenance, like investigating failed pipelines and rotating cluster secrets. For recurring incidents, I turn manual fixes into automated scripts and update our runbooks."

### Behavioral & Scenario-Based Questions (By Category)

**CI/CD Tools:** setting up a pipeline from scratch, unexpected pipeline failures, productivity improvements via pipeline optimization.
**Cloud Platforms:** cloud migration experience, troubleshooting production issues, IaC tool experience.
**Containers & Orchestration:** containerizing an app, debugging a K8s cluster issue, configuring scaling/load balancing.
**Monitoring & Security:** catching a performance issue proactively, responding to a security vulnerability, implementing logging/alerting from scratch.
**Collaboration & Culture:** bridging Dev and Ops, handling a high-pressure incident, introducing a process improvement.

### Sample Behavioral Answers (STAR-style, reusable)

- **Setting up CI/CD from scratch:** Migrating Jenkins → AWS-native tools (CodePipeline/CodeBuild). Challenges: granular IAM roles, cross-service permissions, secure webhook triggering.
- **Unexpected pipeline failure:** Root causes — missing build dependencies, agent/runner outages, broken webhook secrets. Resolution — check logs, inspect executor availability, add automated retry logic.
- **Productivity improvement:** Added Git pre-commit hooks for coding/security standards; shifted feedback left by auto-running tests on push.
- **Cloud migration:** Multi-cloud migration (Azure → AWS) using AWS DMS for large-scale data transfer.
- **Production outage (502 Bad Gateway):** Isolated backend pods crashing/OOMKilled → checked container runtime logs, inspected resource limits, collaborated with dev team on root cause.
- **IaC change management:** Managed Terraform state/drift; enforced environment promotion gates (dev/staging first, peer-reviewed plans before prod).
- **Containerization benefits:** Consistent runtime environments, simpler dependency management, predictable scaling. Challenges: deconstructing monolithic dependencies, secure config/secret injection.
- **Cluster outage escalation:** Assess multi-namespace impact; escalate to managed cloud support for control-plane issues; follow with RCA.
- **Proactive detection:** Caught CPU climbing above 80–90% or disk saturation from unrotated logs, resolved before SLA breach.
- **Security remediation:** Fixed overly permissive IAM roles and unauthenticated endpoints; enforced centralized identity (AD/Azure AD/Entra ID SSO) and DR/snapshot strategies.
- **Bridging Dev & Ops:** Wrote self-service deployment scripts and standardized pipeline templates; built shared observability dashboards to cut MTTR.

### Detailed Real-World Scenario Answers

**1. Database Saturation & CrashLoopBackOff**
- Pods hit OOM errors and CrashLoopBackOff due to connection exhaustion from unoptimized, massive DB tables.
- **Short-term fix:** purged historical records older than 3 months, added DB indexes to cut query latency.
- **Long-term fix:** scheduled automation jobs to archive/clean historical data periodically.

**2. Post-Deployment Data Inconsistencies**
- Runtime errors after a release due to missing DB column values / config drift.
- **Process:** isolated pod logs → replicated & verified fixes in lower environments first → applied hotfix to prod.

**3. Terraform: Ad-hoc Scripts → Modular Code**
- Migrated to reusable Terraform modules to eliminate boilerplate duplication.
- **State management:** fixed concurrency/drift issues by moving state to a remote backend with locking (S3 + DynamoDB), storing code in Git, running plan/apply via CI/CD with mandatory PR review.
- **Why Terraform over CloudFormation:** unified syntax across multi-cloud (Azure + AWS).

**4. VM → Kubernetes Containerization**
- Traditional VM setups had silent weekend outages only caught Monday morning.
- **Benefit:** Kubernetes gave automated self-healing, auto-restarts, predictable resource isolation.
- **Challenge:** steep learning curve on K8s architecture and container networking debugging.

**5. DR Drill Failure & Recovery**
- After a DR drill restore, workloads across multiple namespaces failed.
- **Debug steps:** `kubectl get pods -A` → `kubectl describe pod` to check events/timestamps → found mismatched env vars & broken ConfigMaps from the restore.
- **Fix:** rolled back to the previous stable snapshot/release, then did post-incident RCA to fix restoration automation scripts.

**6. Auto-Scaling & Load Balancing**
- **Auto-scaling:** HPA scales pod replicas on CPU/memory thresholds; Cluster Autoscaler adds/removes worker nodes when pending pods exceed capacity.
- **Traffic routing:** AWS ALB (via AWS Load Balancer Controller/Ingress) for L7 path-based routing into K8s target groups/microservices.

### Company-Specific Prep

**Globant — DevOps Engineer (1–1.5 hr technical round)**

| Domain | Topics Asked |
|--------|----------------|
| **Linux** | File hierarchy, `ip addr`/`ifconfig`, `who`/`w`, `kill`/`kill -9`/`pkill`, `chmod`/`chown` |
| **Git** | `git fetch` vs `git pull`, purpose & mechanics of `git cherry-pick` |
| **Jenkins** | Pipeline lifecycle/stages, common plugins, trigger mechanisms (poll SCM, webhooks, cron) |
| **Docker** | `FROM`, `CMD` vs `ENTRYPOINT`, `COPY` vs `ADD`, Docker Swarm vs services/nodes, Compose, networking, volumes |
| **Kubernetes** | Control plane vs worker nodes, master unreachable behavior, headless services, ReplicaSet vs ReplicationController, Taints & Tolerations, Ingress, Helm, `kubectl --dry-run=client -f <file.yaml>` |
| **Terraform/Ansible** | `terraform apply -auto-approve`, Ansible syntax + `--syntax-check`, custom modules, roles, Tower/AWX, `ansible-vault` |

> **Key takeaway:** Even if your background is AWS-heavy, be ready for Azure DevOps-flavored questions if the role leans that way.

**BMC Software — DevOps / Automation Focus**

Heavy emphasis on **live Bash coding**, not just theory: Shebang purpose, dynamic user input (`read`), writing test conditions (e.g., check if a file exists and is writable: `-w`), file automation (moving/renaming files with a dynamic timestamp).

> **Key takeaway:** Prepare to write real Bash scripts live, not just explain concepts.

### Career Advice: Why Keep Interviewing Even With an Offer in Hand

- **Compensation gap**, **lack of project clarity**, **location & work-life fit** are all valid reasons to keep looking.
- **Be upfront** with new recruiters: disclose an existing offer, state compensation/project expectations clearly.
- **Run parallel interview pipelines** — most companies need 2–3 rounds, which takes weeks; offers can fall through.
- **Know your value**, **demand project & team clarity**, and **do your due diligence** (talk to current/former employees on LinkedIn) before committing.

---

## 25. Quick Recall Cheat Sheet

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

---

*End of guide. Organized tool-wise from the original DevOps End-to-End Project Interview Notes and the DevOps Interview Master Guide (Sakshi Gautam's series) for long-term recall.*
