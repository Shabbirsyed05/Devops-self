# ☁️ AWS Zero to Hero — Interview Prep Guide (Days 11–20)

> 📺 Based on the **Free 30-Day AWS Zero to Hero Course** by Abhishek
> 🎯 Purpose: Master CloudFormation, CI/CD, Lambda, CloudFront & ECR for interviews

---

## 📋 Table of Contents

| Day | Topic |
|-----|-------|
| [Day 11](#-day-11-infrastructure-as-code--cloudformation) | IaC with CloudFormation |
| [Day 12](#-day-12-aws-codecommit) | AWS CodeCommit |
| [Day 13](#-day-13-aws-codepipeline) | AWS CodePipeline (Orchestration) |
| [Day 14](#-day-14-aws-end-to-end-ci-project) | End-to-End CI Project |
| [Day 15](#-day-15-aws-ultimate-cicd-pipeline) | Ultimate CI/CD Pipeline (CD) |
| [Day 16](#-day-16-aws-cloudwatch-deep-dive) | AWS CloudWatch |
| [Day 17](#-day-17-aws-lambda-introduction) | AWS Lambda (Serverless) |
| [Day 18](#-day-18-aws-cost-optimization-project) | Cost Optimization Project |
| [Day 19](#-day-19-aws-cloudfront--cdn) | AWS CloudFront & CDN |
| [Day 20](#-day-20-aws-ecr--elastic-container-registry) | AWS ECR |
| [Master Cheatsheet](#-master-cheatsheet-days-1120) | Master Cheatsheet |

---

---

# 🧱 Day 11: Infrastructure as Code — CloudFormation

**Goal:** Replace console clicks with reusable, version-controlled code.

---

## 🔵 What is IaC & CloudFormation?

> **IaC:** Write a template → tool reads it → AWS API calls → infrastructure created automatically.

**CloudFormation (CFT)** = AWS's native IaC tool. Works **only with AWS** (not Azure/GCP).

| Principle | Meaning |
|---|---|
| **Versioned** | Templates stored in Git/S3 — full history tracked |
| **Declarative** | Read the template → know exactly what exists in the account |

---

## 🔵 CLI vs CloudFormation

| Feature | AWS CLI | AWS CloudFormation |
|---|---|---|
| **Style** | Procedural (how to do it) | Declarative (what I want) |
| **Best for** | Quick one-off tasks | Full infrastructure stacks |
| **Lifecycle** | Script-based | Managed as a "Stack" |

---

## 🔵 YAML vs JSON for Templates

| Format | Recommendation | Why |
|---|---|---|
| **YAML** | ✅ Recommended | Supports comments, clean indentation, human-readable |
| **JSON** | ❌ Avoid | No comments, heavy brackets, hard to maintain |

---

## 🔵 CloudFormation Template Anatomy

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Parameters:               # Optional — runtime values (t2.micro / t2.large)
  InstanceType:
    Type: String

Conditions:               # Optional — create resources only for Prod/Dev

Mappings:                 # Optional — map region → AMI ID

Resources:                # MANDATORY — what to create
  MyServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-xxxxxxx

Outputs:                  # Optional — return Public IP, ARN, etc.
  PublicIP:
    Value: !GetAtt MyServer.PublicIp
```

---

## 🔵 Drift Detection — Critical Feature

**Problem:** CFT creates S3 bucket with Versioning Enabled. Someone manually disables it via console.
```
Template: Versioning = Enabled
Reality:  Versioning = Suspended
Code ≠ Real infrastructure ❌
```

**Solution: Drift Detection**
- AWS compares template (expected) vs actual infrastructure (current)
- Stack marked **"Drifted"** with exact diff shown

---

## 🔵 CFT vs Terraform

| Criteria | CloudFormation | Terraform |
|---|---|---|
| **Cloud support** | AWS only | Multi-cloud (AWS + Azure + GCP) |
| **Integration** | Native AWS | External tool |
| **Job market** | Good | **Very high** |

> **Rule:** Company 100% AWS → CloudFormation. Multi-cloud/Hybrid → Terraform.

### 🎙️ Interview Answer: "CFT vs Terraform?"
> *"CloudFormation is AWS's native IaC tool — it's deeply integrated and requires no extra setup, but works only with AWS. Terraform is cloud-agnostic — the same tool manages AWS, Azure, and GCP, which makes it more versatile. In the job market, Terraform is more in demand because most companies use hybrid or multi-cloud strategies. I choose CFT for pure AWS environments and Terraform when the infrastructure spans multiple clouds."*

---

---

# 📁 Day 12: AWS CodeCommit

**Goal:** Understand AWS's managed Git service and why industry uses GitHub instead.

---

## 🔵 AWS CI/CD Tool Mapping

| Standard Tool | AWS Equivalent | Purpose |
|---|---|---|
| GitHub / GitLab | **CodeCommit** | Source Control |
| Jenkins | **CodePipeline** | CI/CD Orchestration |
| Maven / Docker | **CodeBuild** | Build & Test |
| ArgoCD / Ansible | **CodeDeploy** | Deploy to EC2/K8s |

---

## 🔵 What is CodeCommit?

> Fully managed, private Git repository service — AWS handles patching, scaling, and availability.

**Why it exists:** Companies used to self-host GitLab on EC2 — requiring manual maintenance. CodeCommit eliminates that.

---

## 🔵 Practical Setup

```bash
# 1. Create IAM User with: AWSCodeCommitPowerUser policy
# 2. Create repository in CodeCommit console
# 3. Clone using HTTPS URL (use IAM Git credentials — NOT console password)
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-repo

# 4. Normal git workflow
git add . && git commit -m "msg" && git push
```

---

## 🔵 Reality Check: Industry Uses GitHub

| Feature | CodeCommit | GitHub |
|---|---|---|
| Community | ❌ None | ✅ Massive open source |
| Features | Basic | Copilot, Actions, Marketplace |
| UI | Basic | Rich, developer-friendly |
| Lock-in | AWS only | Portable |

> **Course strategy:** Use **GitHub + AWS CodeBuild + CodeDeploy + CodePipeline** — more interview-relevant.

---

---

# 🎼 Day 13: AWS CodePipeline

**Goal:** Understand orchestration — the conductor that coordinates CI/CD tools.

---

## 🔵 What is CodePipeline?

> A **fully managed Continuous Delivery orchestrator**.

```
CodePipeline = Orchestra Conductor
CodeBuild / CodeDeploy = Musicians
```

**Key point:** CodePipeline does NOT build or deploy — it only coordinates between tools.

---

## 🔵 Standard CI/CD Flow

```
Developer commits → CodePipeline detects (Webhook)
                         ↓
               CodeBuild: compile + test
                         ↓
               CodeDeploy: deploy to EC2/K8s
```

---

## 🔵 CodePipeline vs Jenkins — Interview Question

| | Jenkins | CodePipeline |
|---|---|---|
| **Hosting** | Self-hosted | Fully managed (AWS) |
| **Cloud support** | Any cloud (agnostic) | AWS only (vendor lock-in) |
| **Plugin ecosystem** | Massive | Limited |
| **Maintenance** | You manage master + agents | Zero — AWS handles everything |
| **Cost** | Pay for EC2 always | Serverless — pay per pipeline execution |
| **Best for** | Multi-cloud, complex pipelines | AWS-only startups without DevOps team |

### 🎙️ Interview Answer: "Jenkins vs CodePipeline?"
> *"Jenkins is self-hosted and cloud-agnostic — ideal for multi-cloud or on-prem scenarios with complex pipeline requirements. CodePipeline is fully managed — zero maintenance, serverless pricing, and deep AWS integration. The downside of CodePipeline is vendor lock-in: if the company moves to Azure, the pipeline must be rewritten. I'd choose Jenkins for flexibility and CodePipeline for speed and simplicity in pure AWS environments."*

---

---

# 🐳 Day 14: AWS End-to-End CI Project

**Goal:** Automate: `GitHub Commit → Build Docker Image → Push to Docker Hub`

---

## 🔵 Architecture

```
Developer commits to GitHub
        ↓
CodePipeline detects change
        ↓
CodeBuild builds Docker image
        ↓
Image pushed to Docker Hub ✅
```

---

## 🔵 Step 1: Store Secrets in SSM Parameter Store

> Never hardcode Docker credentials in code or scripts.

```
SSM → Parameter Store → SecureString:
  DOCKER_REGISTRY = docker.io
  DOCKER_USERNAME = myuser
  DOCKER_PASSWORD = mypassword
```

---

## 🔵 Step 2: Configure CodeBuild

| Setting | Value | Why |
|---|---|---|
| Source | GitHub via OAuth | Connect to repo |
| Environment | Managed Ubuntu | Standard build env |
| IAM Role | Must allow SSM read | To fetch credentials |
| **Privileged Mode** | ✅ **MUST ENABLE** | Required for docker build/push |

> ⚠️ **Without Privileged Mode enabled, Docker commands will fail.**

---

## 🔵 Step 3: buildspec.yml — Heart of CI

```yaml
version: 0.2

env:
  parameter-store:
    DOCKER_REGISTRY: /myapp/docker_registry
    DOCKER_USERNAME: /myapp/docker_username
    DOCKER_PASSWORD: /myapp/docker_password

phases:
  install:
    runtime-versions:
      python: 3.11

  pre_build:
    commands:
      - pip install -r requirements.txt
      - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin $DOCKER_REGISTRY

  build:
    commands:
      - docker build -t $DOCKER_REGISTRY/$DOCKER_USERNAME/myapp:latest .

  post_build:
    commands:
      - docker push $DOCKER_REGISTRY/$DOCKER_USERNAME/myapp:latest
```

---

## 🔵 Common Errors & Fixes

| Error | Fix |
|---|---|
| `Cannot connect to Docker daemon` | Enable **Privileged Mode** in CodeBuild |
| `docker build requires 1 argument` | Add context path: `docker build -t image .` |
| `Authentication failed` | Add `docker login` using SSM credentials |

### 🎙️ Interview Answer: "What is buildspec.yml?"
> *"buildspec.yml is the configuration file CodeBuild reads to know what to do during a build. It's organized into phases: install sets the runtime, pre_build handles dependencies and authentication, build compiles or packages the application, and post_build handles publishing — like pushing a Docker image to a registry. Secrets are pulled from SSM Parameter Store, never hardcoded."*

---

---

# 🚀 Day 15: AWS Ultimate CI/CD Pipeline (CD)

**Goal:** Complete the pipeline — deploy the Docker image to EC2 using CodeDeploy.

---

## 🔵 Full Architecture

```
GitHub commit → CodePipeline → CodeBuild (CI) → CodeDeploy (CD) → EC2
```

---

## 🔵 CodeDeploy Agent (NOT Serverless)

> Unlike CodeBuild, CodeDeploy requires a **software agent installed on the EC2 instance**.

```bash
# Install prerequisites
sudo apt install ruby wget -y

# Download agent (from AWS S3)
wget https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install
chmod +x install
sudo ./install auto

# Verify
sudo systemctl status codedeploy-agent
```

The agent **polls CodeDeploy continuously**: *"Do you have a new deployment for me?"*

---

## 🔵 IAM Roles Required

| Role | Purpose |
|---|---|
| **Service Role** | Allows CodeDeploy → EC2 communication |
| **Instance Role** | Allows EC2 → CodeDeploy + S3 access |

---

## 🔵 Finding Instances: Tags NOT IPs

> ⚠️ CodeDeploy does NOT use IP addresses. It finds servers using **Tags**.

```
EC2 Tag: Name = sample-python-app
Deployment Group: targets instances with tag Name = sample-python-app
```

---

## 🔵 appspec.yml — Heart of CD

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/app

hooks:
  ApplicationStop:
    - location: scripts/stop_container.sh
      timeout: 300
      runas: root

  AfterInstall:
    - location: scripts/start_container.sh
      timeout: 300
      runas: root
```

**Lifecycle Hooks:**
- `ApplicationStop` → remove old container (prevents port conflict)
- `AfterInstall` → pull new image + run container

---

## 🔵 Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| Deployment fails instantly | Missing `appspec.yml` | Add to root of repo |
| Script error: docker not found | Docker not installed on EC2 | `sudo apt install docker.io -y` |
| Port already in use (5000) | Old container still running | Cleanup in `ApplicationStop` hook |

---

## 🔵 buildspec.yml vs appspec.yml

| | buildspec.yml | appspec.yml |
|---|---|---|
| **Used by** | CodeBuild | CodeDeploy |
| **Purpose** | Build & test instructions | Deployment instructions |
| **Phases** | install, pre_build, build, post_build | lifecycle hooks (Stop, Install, Start) |

### 🎙️ Interview Answer: "How does CodeDeploy work?"
> *"CodeDeploy requires an agent installed on the EC2 instance — it polls the CodeDeploy service for new deployments. It finds target instances using Tags, not IP addresses, which makes it dynamic. The appspec.yml file controls the deployment — lifecycle hooks like ApplicationStop remove the old container, and AfterInstall pulls and runs the new one. Without the ApplicationStop hook, you get port conflicts on redeployment."*

---

---

# 📊 Day 16: AWS CloudWatch Deep Dive

**Goal:** Monitor, alert, and debug AWS resources automatically.

---

## 🔵 What is CloudWatch?

> Monitoring and observability service — the "Watchman" of your AWS account.

**4 Core Features:**

| Feature | What It Does |
|---|---|
| **Monitoring** | Tracks health of resources |
| **Metrics** | Numeric data (CPU%, requests, bytes) |
| **Alarms** | Triggers action when threshold crossed |
| **Logs** | Detailed event history |

---

## 🔵 Metrics — The Data

```
AWS provides 1,036+ default metrics out of the box

EC2 reports CPU every:
  5 minutes (default/basic monitoring)
  1 minute  (detailed monitoring — extra cost)
```

> ⚠️ **CloudWatch tracks CPU by default — NOT Memory (RAM). RAM requires Custom Metrics.**

---

## 🔵 Alarms — The Action

```
Threshold: CPU >= 50%
      ↓
Alarm fires
      ↓
SNS Topic
      ↓
Email / Slack / SMS notification
```

> ⚠️ **Must click "Confirm Subscription" in email inbox — otherwise alarm will NOT trigger notifications.**

---

## 🔵 Log Groups

- CloudWatch auto-creates log groups for AWS services (CodeBuild, Lambda, etc.)
- Logs persist even after the service/project is deleted
- Use for debugging failures from weeks ago

---

## 🔵 Metrics vs Logs — Interview Distinction

| | Metrics | Logs |
|---|---|---|
| **Type** | Numeric time-series data | Text event records |
| **Example** | CPU = 72% at 10:05 AM | "ERROR: DB connection refused at 10:05 AM" |
| **Use for** | Alarms, dashboards | Debugging, audit trails |

### 🎙️ Interview Answer: "CloudWatch Metrics vs Logs?"
> *"Metrics are numeric time-series data — like CPU utilization or request count — used to set thresholds and trigger alarms. Logs are text records of what happened — like application error messages or API calls — used for debugging. For RAM monitoring specifically, CloudWatch doesn't track it by default because it requires the CloudWatch Agent installed on the instance to push custom metrics."*

---

---

# ⚡ Day 17: AWS Lambda — Introduction

**Goal:** Understand serverless compute and its DevOps automation use cases.

---

## 🔵 Lambda vs EC2

| Feature | EC2 (Server) | Lambda (Serverless) |
|---|---|---|
| **Management** | Manual — patch, update, maintain | Fully managed by AWS |
| **Cost** | Pay per hour even when idle | Pay per execution (milliseconds) |
| **Best for** | 24/7 long-running apps | Short, event-driven tasks |
| **Visibility** | Full OS/IP control | No server visibility |

**Analogy:**
- EC2 = Owning a restaurant (fixed cost, always open)
- Lambda = Ordering food only when hungry (pay per use)

---

## 🔵 Lambda Workflow

```
Event Trigger → AWS spins up tiny runtime → Code runs → Runtime destroyed
  ↓
Pay only for the milliseconds of execution
```

**Common triggers:**
- S3 file upload
- CloudWatch schedule (cron)
- API Gateway HTTP request
- SNS/SQS message

---

## 🔵 Lambda Technical Components

| Component | Detail |
|---|---|
| **Runtimes** | Python, Java, Go, Node.js (Shell scripting NOT supported) |
| **Handler** | Entry point method — first thing AWS calls |
| **Triggers** | Events that "kick" the function |
| **IAM Role** | Lambda has ZERO permissions by default — must attach Role |
| **Timeout** | Default 3 seconds — increase for longer tasks |

---

## 🔵 DevOps Use Cases for Lambda

### Cost Optimization
```
CloudWatch triggers Lambda daily
Lambda scans for unused EBS volumes
Sends notification or auto-deletes → saves money
```

### Security Compliance
```
Lambda detects public S3 bucket → SNS alert
Lambda detects gp2 EBS (banned type) → auto-remediate
```

> ✅ Cheaper than running EC2 just to execute a cleanup script.

### 🎙️ Interview Answer: "Lambda vs EC2?"
> *"Lambda is serverless — you provide the code and AWS handles everything else. It's event-driven: a CloudWatch schedule, S3 upload, or API call triggers it. You pay only for the milliseconds it runs. EC2 is a persistent VM — ideal for 24/7 workloads but costly when idle. For DevOps, I use Lambda for automation tasks like cleaning up stale resources, security compliance checks, and cost optimization — tasks that run occasionally and don't justify a full-time server."*

---

---

# 💰 Day 18: AWS Cost Optimization — Lambda Project

**Goal:** Automate deletion of stale EBS snapshots using Lambda + boto3.

---

## 🔵 The Problem: Stale Resources

```
Developer:
1. Creates EC2 instance
2. Takes EBS snapshots (backups)
3. Deletes EC2 + EBS volume
4. FORGETS to delete snapshots ← billing continues forever 💸
```

> These forgotten resources are called **"Stale Resources"** — a major hidden AWS cost.

---

## 🔵 The Solution Architecture

```
EventBridge (daily trigger) → Lambda (Python/boto3) → Scan snapshots → Delete stale ones
```

---

## 🔵 The Algorithm

```python
for each EBS Snapshot:
    if parent volume does NOT exist:
        DELETE snapshot  ← volume was deleted, snapshot is orphaned

    elif parent volume EXISTS but NOT attached to running EC2:
        DELETE snapshot  ← volume is idle, snapshot not needed
```

---

## 🔵 Implementation Steps

```
1. Create EC2 → take snapshot → terminate EC2
   (simulates orphaned snapshot)

2. Create Lambda (Python runtime)
   - Change timeout: 3s → 10s (default 3s too short)
   - Paste boto3 cleanup script

3. Attach IAM Role with permissions:
   - ec2:DescribeSnapshots
   - ec2:DeleteSnapshot
   - ec2:DescribeVolumes
   - ec2:DescribeInstances

4. Test → snapshot deleted ✅
5. Add EventBridge rule for daily/weekly automation
```

---

## 🔵 Resume Point (Write This!)

> *"Implemented an event-driven AWS Lambda solution using boto3 to automatically detect and delete stale EBS snapshots not attached to active volumes, reducing cloud storage costs."*

### 🎙️ Interview Answer: "Cost optimization in AWS?"
> *"I built a Lambda function triggered daily by EventBridge that scans all EBS snapshots and deletes any whose parent volume no longer exists or isn't attached to a running instance. This eliminates orphaned snapshot billing automatically. The function uses boto3 with a custom IAM role restricted to only the necessary EC2 describe and delete permissions. This is more efficient than running an EC2 just for a periodic script."*

---

---

# 🌍 Day 19: AWS CloudFront — CDN

**Goal:** Deliver content fast globally and secure your S3 origin.

---

## 🔵 The Problem: Latency

```
Server: North Virginia (USA)
User: India

Without CDN → request travels through many global hops → slow ❌
```

---

## 🔵 The Solution: Edge Locations

```
CloudFront caches content in Mumbai Edge Location
User in India → downloads from Mumbai → instant ✅
```

**Edge Locations** = AWS data centers near users worldwide, used only for caching.

---

## 🔵 Why Not Direct S3 Hosting in Production?

| Problem | Direct S3 | With CloudFront |
|---|---|---|
| **Latency** | One region only — far users are slow | Cached at global edge locations |
| **Security** | Bucket must be public | Bucket stays private (OAI) |
| **Cost** | Every download charges S3 rates | Cache hits = cheaper than S3 direct |

---

## 🔵 OAI — Origin Access Identity (Key Security Concept)

> Forces users to access content **only through CloudFront**, never directly via S3 URL.

```
S3 Bucket Policy: "Allow access ONLY to OAI. Deny everyone else."

Direct S3 URL → ❌ Access Denied (403)
CloudFront URL → ✅ Content loads
```

---

## 🔵 Setup Steps

```
1. Create S3 bucket → upload index.html
2. Create CloudFront Distribution → select S3 as origin
3. Enable OAI (Origin Access Identity) → click "Update Bucket Policy" (auto-configured)
4. Set Default Root Object: index.html
5. Choose edge locations (All = global, Specific = cheaper)
6. Deploy → test both URLs to verify OAI works
```

**Cleanup order (prevent billing):**
```
1. Disable distribution (wait a few minutes)
2. Delete distribution
3. Delete S3 bucket
```

### 🎙️ Interview Answer: "What is CloudFront and OAI?"
> *"CloudFront is AWS's CDN — it caches content at Edge Locations near users globally, reducing latency. For S3 static websites, I always use CloudFront instead of direct S3 access for three reasons: performance (edge caching), security (OAI keeps the bucket private), and cost (cache hits are cheaper than S3 requests). OAI is a virtual identity I attach to the CloudFront distribution — the S3 bucket policy allows only that identity, so direct S3 links return 403 errors."*

---

---

# 📦 Day 20: AWS ECR — Elastic Container Registry

**Goal:** Understand AWS's private container image registry and when to use it over Docker Hub.

---

## 🔵 What is ECR?

| Word | Meaning |
|---|---|
| **Elastic** | Highly scalable, managed by AWS |
| **Container** | Stores container images |
| **Registry** | Central store to share/pull images |

> ECR supports **OCI standard** — works with Docker, Podman, Buildah (not just Docker).

---

## 🔵 ECR vs Docker Hub

| Feature | Docker Hub | AWS ECR |
|---|---|---|
| **Default visibility** | Public (great for open source) | Private (enterprise-ready) |
| **Access control** | Separate login | Integrated with **AWS IAM** |
| **AWS integration** | External | Native with ECS, EKS, Fargate |
| **Vulnerability scanning** | Manual | Auto-scans on push ✅ |

> Companies prefer ECR because security is controlled through existing IAM roles — no separate account management.

---

## 🔵 Authentication: No Username/Password

```bash
# Get temporary token from AWS and pipe into docker login
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

**How it works:**
1. AWS CLI generates a temporary auth token
2. Token is piped into `docker login`
3. Local Docker gets authenticated with ECR — valid for 12 hours

---

## 🔵 Tag & Push Workflow

```bash
# 1. Build image locally
docker build -t my-app .

# 2. Tag with ECR format (REQUIRED before push)
docker tag my-app:latest \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest

# 3. Push to ECR
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
```

> Tags must match ECR format exactly: `account-id.dkr.ecr.region.amazonaws.com/repo:tag`

---

## 🔵 IAM Permissions Required

```
Root account:   Works by default
IAM user:       Must attach ECR Power User policy
                OR specific permissions:
                - ecr:PutImage
                - ecr:GetDownloadUrlForLayer
                - ecr:BatchCheckLayerAvailability
```

### 🎙️ Interview Answer: "ECR vs Docker Hub?"
> *"Docker Hub is public by default — ideal for open-source projects. ECR is private by default and integrates natively with AWS IAM, so I control access using the same roles and policies already managing my AWS environment. ECR also auto-scans images for vulnerabilities on push. Authentication doesn't use a username/password — the AWS CLI generates a temporary 12-hour token. I use ECR for production workloads with EKS or ECS, and Docker Hub for public open-source images."*

---

---

# 📌 Master Cheatsheet (Days 11–20)

```
╔══════════════════════════════════════════════════════════════════════╗
║          AWS INTERVIEW CHEATSHEET — DAYS 11-20                       ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  CLOUDFORMATION:                                                     ║
║  IaC = templates → API calls → infrastructure auto-created          ║
║  Resources section = MANDATORY | Others = optional                  ║
║  Drift Detection = template vs actual state comparison               ║
║  CFT=AWS only | Terraform=multi-cloud (higher job demand)           ║
║                                                                      ║
║  AWS CI/CD TOOLS:                                                    ║
║  CodeCommit=Git | CodePipeline=Orchestrator | CodeBuild=Build/Test  ║
║  CodeDeploy=Deploy to EC2/K8s | Industry prefers GitHub+AWS          ║
║                                                                      ║
║  CODEPIPELINE vs JENKINS:                                            ║
║  Jenkins=cloud-agnostic, plugin-rich, self-hosted (you maintain)    ║
║  CodePipeline=AWS-only, serverless, zero maintenance                 ║
║                                                                      ║
║  BUILDSPEC vs APPSPEC:                                               ║
║  buildspec.yml → CodeBuild (CI: build, test, push image)            ║
║  appspec.yml → CodeDeploy (CD: stop old, install, start new)        ║
║  SSM Parameter Store → store Docker credentials (never hardcode)    ║
║  Privileged Mode → MUST enable for docker build/push                ║
║  CodeDeploy uses TAGS not IP addresses to find EC2                  ║
║                                                                      ║
║  CLOUDWATCH:                                                         ║
║  Metrics=numbers | Logs=text | Alarms=action on threshold           ║
║  CPU tracked by default | RAM = Custom Metric required               ║
║  Confirm SNS subscription email → else alarm notifications fail     ║
║                                                                      ║
║  LAMBDA:                                                             ║
║  Serverless | Pay per ms | Event-driven | No shell scripting        ║
║  Default timeout=3s | IAM Role required (zero permissions default)  ║
║  boto3 = Python SDK for AWS APIs                                     ║
║  Triggers: CloudWatch schedule, S3 upload, API Gateway              ║
║                                                                      ║
║  COST OPTIMIZATION:                                                  ║
║  Stale snapshots = orphaned EBS backups → ongoing hidden cost       ║
║  Lambda + boto3 + EventBridge = automated cleanup                   ║
║                                                                      ║
║  CLOUDFRONT:                                                         ║
║  CDN = cache at Edge Locations near users → low latency             ║
║  OAI = forces S3 access via CloudFront only (bucket stays private)  ║
║  Direct S3 URL with OAI → 403 | CloudFront URL → 200               ║
║                                                                      ║
║  ECR:                                                                ║
║  Private Docker Hub for AWS | OCI standard (not just Docker)        ║
║  Auth = temp token via `aws ecr get-login-password` (12 hrs)        ║
║  Must tag image with ECR format before push                          ║
║  Auto-scans for vulnerabilities on push                              ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Delivery Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Use **Problem → Solution** structure | Jump straight to commands |
| Say "Privileged Mode" for Docker in CodeBuild | Forget this critical step |
| Explain **Tags** for CodeDeploy (not IPs) | Say "CodeDeploy uses the server IP" |
| Mention **OAI** for any CloudFront + S3 question | Stop at "I used CloudFront for caching" |
| *"Would you like me to go deeper?"* | Stop without engaging |

---

## 🗓️ Course Progress Tracker

| Day | Topic | Status |
|---|---|---|
| Day 11 | CloudFormation & IaC | ✅ Done |
| Day 12 | CodeCommit | ✅ Done |
| Day 13 | CodePipeline (Orchestration) | ✅ Done |
| Day 14 | End-to-End CI Project | ✅ Done |
| Day 15 | Ultimate CI/CD Pipeline (CD) | ✅ Done |
| Day 16 | CloudWatch (Metrics, Alarms, Logs) | ✅ Done |
| Day 17 | Lambda Introduction | ✅ Done |
| Day 18 | Cost Optimization Project | ✅ Done |
| Day 19 | CloudFront & CDN | ✅ Done |
| Day 20 | ECR (Container Registry) | ✅ Done |
| Day 21+ | EKS, SSM, Auto Scaling, RDS... | 📅 Upcoming |

---

## 📚 Resources

- 📺 [AWS Zero to Hero Course](https://www.youtube.com/watch?v=Ou9j73aWgyE)
- 🔗 [CloudFormation Docs](https://docs.aws.amazon.com/cloudformation/)
- 🔗 [AWS CodePipeline Docs](https://docs.aws.amazon.com/codepipeline/)
- 🔗 [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/)
- 🔗 [CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/)
- 🔗 [ECR Docs](https://docs.aws.amazon.com/ecr/)

---

> ⭐ **Star this repo** if it helped you prepare for your AWS interview!
> 🔔 Paste the next day's notes — I'll overwrite with only those days!
