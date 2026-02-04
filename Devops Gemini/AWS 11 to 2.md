# 🚀 Day-11 | Infrastructure as Code with AWS CloudFormation (CFT)

This document summarizes the session **“Day-11 | IaC with AWS CFT.”**  
The session introduces **Infrastructure as Code (IaC)** — moving from manual console clicks to writing code that builds complete infrastructure.

---

## 🔷 1. What is CloudFormation (CFT) & IaC?

### 📌 Infrastructure as Code

Instead of manually creating servers and networks:

- You write a **template (code)**
- An IaC tool reads this template  
- It converts the template into **AWS API calls**
- Infrastructure is created automatically

---

### 🧩 What is CFT?

- **CloudFormation Template (CFT)** = AWS native IaC tool  
- Works **only with AWS**  
- Does not support Azure or Google Cloud

---

### 🧠 Two Core Principles

| Principle | Meaning |
|---------|---------|
| Versioned | Templates can be stored in Git/S3 to track history |
| Declarative | “What you see is what you have” |

> By reading the template, you should know exactly what exists in the account.

---

## 🟩 2. CFT vs AWS CLI

| Feature | AWS CLI | AWS CFT |
|------|---------|---------|
| Purpose | Quick actions | Full infrastructure stacks |
| Style | Procedural | Declarative |
| Use Case | List buckets, small tasks | Create VPC + EC2 + Subnets |
| Lifecycle | Script based | Managed as Stack |

**Conclusion:**  
CLI → for small operations  
CFT → for real infrastructure deployment

---

## 🟨 3. YAML vs JSON

CloudFormation supports both formats.

### ✅ YAML – Recommended

- Supports comments  
- Clean indentation  
- Human readable  
- Easy debugging

### ❌ JSON – Not Recommended

- No comments  
- Heavy brackets  
- Hard to maintain

> Instructor strongly recommends **YAML**

---

## 🟪 4. Anatomy of a CloudFormation Template

### 🔴 Mandatory Section

| Section | Purpose |
|-------|---------|
| Resources | Defines what to create (EC2, S3, VPC) |

---

### 🟢 Optional Sections

| Section | Purpose |
|-------|---------|
| Parameters | Values passed at runtime (t2.micro / t2.large) |
| Mappings | Map region → AMI IDs |
| Conditions | Create resources only for Prod/Dev |
| Outputs | Return values like Public IP |

---

### 🧾 Example Concept

```yaml
Resources:
  MyServer:
    Type: AWS::EC2::Instance
```

---

## 🟥 5. Drift Detection – Critical Feature

### ❗ The Problem

1. CFT creates S3 bucket with **Versioning Enabled**
2. Someone manually disables versioning from console
3. Code ≠ Real infrastructure

---

### ✅ The Solution – Drift Detection

- AWS compares:
  - Template (Expected state)
  - Actual infrastructure (Current state)

### 📊 Result

- Stack marked as **“Drifted”**
- Shows exactly what changed  
- Example:
  - Expected: Enabled  
  - Found: Suspended

---

## 🟦 6. How to Write CFTs – Best Practices

### 🧠 Do NOT Memorize Syntax

- Use AWS Documentation  
- “Template Reference” section  
- Copy-paste examples for:
  - EC2  
  - S3  
  - VPC

---

### 🛠 Recommended Tools

| Tool | Purpose |
|----|---------|
| VS Code YAML (Red Hat) | Formatting |
| AWS Toolkit | Autocomplete |
| CFT Designer | Drag & Drop YAML generator |

---

## 🟧 7. Interview Question – CFT vs Terraform

| Criteria | CloudFormation | Terraform |
|-------|----------------|-----------|
| Cloud Support | AWS Only | Multi-Cloud |
| Integration | Native AWS | External Tool |
| Market Demand | Good | Very High |

### 💡 When to Choose What?

- Company 100% AWS → **CloudFormation**
- Multi-Cloud / Hybrid → **Terraform**

> Terraform preferred in job market because it is cloud-agnostic

---

## 📝 Assignment from Instructor

### Task

👉 Create a simple EC2 instance using CloudFormation:

1. Open AWS Documentation  
2. Find EC2 resource syntax  
3. Write YAML template  
4. Deploy stack

---

# ✅ Key Takeaways

- IaC = Infrastructure through Code  
- CFT is AWS native IaC  
- YAML is better than JSON  
- Drift Detection prevents manual mistakes  
- CFT for AWS → Terraform for Multi-Cloud

---

## 🔜 Next Step

👉 Practice writing a basic template  
👉 Move toward Terraform in upcoming sessions


================================================================================

# 🚀 Day-12 | AWS CodeCommit & CI/CD Basics

This document summarizes the session **“Day-12 | AWS CodeCommit & CI/CD Basics.”**  
The session begins a **4-Part CI/CD Module** using **AWS Native DevOps Services** instead of open-source tools like Jenkins.

---

## 🔷 1. AWS CI/CD – Industry Mapping

To understand AWS DevOps tools easily, they are mapped to familiar industry tools:

| Standard Tool | AWS Equivalent | Purpose |
|---------------|----------------|---------|
| GitHub / GitLab | AWS CodeCommit | Source Code Hosting (Version Control) |
| Jenkins | AWS CodePipeline | CI/CD Workflow Orchestration |
| Maven / Docker | AWS CodeBuild | Build & Test Application |
| ArgoCD / Ansible | AWS CodeDeploy | Deploy App to EC2/Kubernetes |

---

## 🟩 2. What is AWS CodeCommit?

### 📌 Definition
AWS CodeCommit is a **fully managed Git-based source control service** used to host private repositories.

---

### ❗ Problem in Traditional Setup

- Companies install GitLab on their own EC2  
- Need to manage:
  - Patching  
  - Security  
  - Scaling  
- As teams grow → servers need upgrades

---

### ✅ AWS Solution

- CodeCommit scales automatically  
- AWS manages:
  - Hardware  
  - Security  
  - Availability  
- Works the same for:
  - 1 repository  
  - 10,000 repositories

---

### 🔒 Privacy Focus

- Designed mainly for **Private Repositories**
- Unlike GitHub:
  - No public open community model  
  - Enterprise-oriented

---

## 🟨 3. Practical Demo – Setting Up CodeCommit

### 🅐 Step 1 – IAM Setup (Very Important)

> ❌ Do NOT use Root Account

#### Required Actions

1. Create IAM User  
2. Attach policy:

```
AWSCodeCommitPowerUser
```

- Gives proper access  
- Avoids full Admin risk

---

### 🅑 Step 2 – Create Repository

- Open CodeCommit Console  
- Click → **Create Repository**

> Note: Browser upload supports only one file at a time – not practical.

---

### 🅒 Step 3 – Clone & Push Code

#### Requirements

- Install Git on laptop  
- Use HTTPS Clone URL

#### Authentication

- Use IAM Git credentials  
- NOT AWS console password

---

### 📘 Assignment from Session

1. Clone repository  
2. Create a file locally  
3. Commit changes  
4. Push back to AWS

---

## 🟥 4. Reality Check – Disadvantages

The instructor shared real industry perspective:

### ❌ Limitations

- Not widely used compared to GitHub  
- Missing modern features like:
  - GitHub Copilot  
  - Rich online editor  
  - Marketplace integrations  
- Less developer-friendly UI  
- Strong vendor lock-in to AWS

---

## 🟦 5. Course Strategy Change

### 🎯 Practical Decision

- Upcoming modules will use:

> ✅ GitHub + AWS CodeBuild + CodeDeploy + CodePipeline

Instead of:

> ❌ CodeCommit based pipeline

---

### Reason

- Most companies use:
  - AWS → Infrastructure  
  - GitHub → Source Code

> This approach is more interview-relevant and real-world friendly.

---

# ✅ Key Takeaways

- CodeCommit = AWS managed Git service  
- Secure & private by design  
- Requires IAM based access  
- Good for AWS-only ecosystems  
- Industry prefers GitHub in reality

---

## 🔜 Next Steps

👉 Day-13 → AWS CodeBuild  
👉 Day-14 → AWS CodeDeploy  
👉 Day-15 → AWS CodePipeline

=========================================================

# 🚀 Day-13 | AWS CodePipeline – Orchestration Deep Dive

This document summarizes the session **“Day-13 | AWS Code Pipeline.”**  
The focus of this session is **Orchestration in CI/CD.**

> If CodeCommit is the **Warehouse**, then CodePipeline is the **Factory Manager** that moves code from development → testing → production automatically.

The instructor explains everything using a comparison with **Jenkins**, the industry standard.

---

## 🔷 1. What is AWS CodePipeline?

### 📌 Definition
AWS CodePipeline is a **fully managed Continuous Delivery service**.

### 🎯 Role

- It is an **Orchestrator**
- It does **NOT**:
  - Build code  
  - Deploy code  
- It only **coordinates** between tools

---

### 🎼 Analogy

> CodePipeline = Orchestra Conductor  
> CodeBuild / CodeDeploy = Musicians

The conductor doesn’t play instruments —  
but controls the entire performance.

---

## 🟩 2. Jenkins vs AWS CodePipeline

### 📊 Component Mapping

| Feature | Jenkins Workflow | AWS Native Workflow |
|-------|------------------|---------------------|
| Source Code | GitHub / GitLab | CodeCommit (or GitHub) |
| Orchestrator | Jenkins | AWS CodePipeline |
| Build (CI) | Jenkins Stages / Maven | AWS CodeBuild |
| Deploy (CD) | Ansible / ArgoCD | AWS CodeDeploy |

---

### 🔁 Standard Workflow

1. Developer commits code  
2. CodePipeline detects change (Webhook)  
3. Triggers **CodeBuild** → compile & test  
4. If success → triggers **CodeDeploy**  
5. App deployed to EC2 / Kubernetes

---

## 🟥 3. Interview Question  
### 👉 Why Choose Jenkins over CodePipeline?

### ❌ Limitations of CodePipeline

#### 🔐 Vendor Lock-in – Biggest Issue

- Works only with AWS  
- If company moves to:
  - Azure  
  - GCP  
  - On-Prem  
→ Pipeline must be rewritten

---

### ✅ Advantages of Jenkins

- Cloud Agnostic  
- Works everywhere  
- Massive plugin ecosystem  
- Integrates with almost any tool

---

## 🟦 4. Interview Question  
### 👉 Why Choose CodePipeline over Jenkins?

### ✅ Benefits of CodePipeline

#### 🛠 Zero Maintenance

- No Master Node  
- No Worker Nodes  
- AWS handles:
  - Patching  
  - Security  
  - Availability

---

#### 💰 Cost & Scalability

- Serverless model  
- Pay-as-you-go  
- Auto scales automatically  
- Ideal for startups without DevOps team

---

## 🟨 5. Next Steps – Day 14 Preview

### 🧪 Upcoming Practical Stack

The instructor will implement:

| Layer | Tool |
|-----|------|
| Source | GitHub |
| Orchestrator | AWS CodePipeline |
| CI | AWS CodeBuild |

---

### 📌 Important Note

> GitHub will be used instead of CodeCommit  
> Because CodeCommit is rarely used in real industry

---

# ✅ Key Takeaways

- CodePipeline = Orchestrator, not builder  
- Works with CodeBuild & CodeDeploy  
- Jenkins → flexible & cloud-agnostic  
- CodePipeline → managed & serverless  
- Choice depends on company strategy

---

## 🔜 Next Session

👉 Day-14 → Practical CI with GitHub + CodeBuild + CodePipeline

===============================================================================

# 🚀 Day-14 | AWS End-to-End CI Project

This document summarizes the session **“Day-14 | AWS End to End CI Project.”**  
The focus of this session is the **Continuous Integration (CI)** part of CI/CD.

### 🎯 Goal

👉 Automate the flow:

> **GitHub Commit → Build Docker Image → Push to Docker Hub**

---

## 🔷 1. Architecture Overview

The instructor uses **real-world tools** instead of AWS CodeCommit.

| Layer | Tool |
|-----|------|
| Source | GitHub |
| Orchestrator | AWS CodePipeline |
| Builder | AWS CodeBuild |
| Destination | Docker Hub |

### 🧩 Flow

1. Developer commits code to GitHub  
2. CodePipeline detects change  
3. CodeBuild builds Docker image  
4. Image pushed to Docker Hub

---

## 🟩 2. Prerequisites – Application Code

The GitHub repository must contain:

- `app.py` → Python Flask application  
- `requirements.txt` → dependencies  
- `Dockerfile` → packaging instructions

### 🐳 Dockerfile Responsibilities

- Install Python  
- Copy project files  
- Expose required ports  
- Define startup command

---

## 🟨 3. Step-1 – Security with SSM Parameter Store

### ❗ Problem

Docker Hub requires:

- Username  
- Password

> Never hardcode credentials in scripts

---

### ✅ Solution – AWS Systems Manager (SSM)

Store as **Secure Strings**:

- Docker Registry URL → `docker.io`  
- Docker Username  
- Docker Password

#### Location

```
SSM → Parameter Store → SecureString
```

---

## 🟪 4. Step-2 – Configuring AWS CodeBuild

### 🅐 Source

- Connect to GitHub via OAuth

### 🅑 Environment

- Managed Ubuntu Image

### 🅒 IAM Role – Critical Step

> You must grant CodeBuild role permission to read SSM

Without this → build fails to fetch credentials

---

### 🅓 Privileged Mode – Very Important

- Must enable **Privileged Flag**
- Required for:
  - docker build  
  - docker login  
  - docker push

---

## 🟦 5. Step-3 – Buildspec.yml (Heart of CI)

The `buildspec.yml` defines the pipeline actions.

### 🔁 Four Phases

#### 1. Env
- Import variables from SSM

#### 2. Install
- Set runtime → Python 3.11

#### 3. Pre_Build
```bash
pip install -r requirements.txt
docker login
```

#### 4. Build
```bash
docker build -t <tag> .
```

#### 5. Post_Build
```bash
docker push <tag>
```

---

## 🟥 6. Troubleshooting – Real Errors

### ❌ Error 1  
**Cannot connect to Docker Daemon**

✅ Fix → Enable Privileged Mode

---

### ❌ Error 2  
**Docker build requires 1 argument**

✅ Fix → Add context path:

```
docker build -t image .
```

---

### ❌ Error 3  
**Authentication Failed**

✅ Fix → Add:

```
docker login
```

using SSM credentials

---

## 🟧 7. Step-4 – Automating with CodePipeline

### Pipeline Stages

| Stage | Configuration |
|----|---------------|
| Source | GitHub – main branch |
| Build | CodeBuild Project |
| Deploy | Skipped (Day-15) |

---

### 🎉 Final Result

- Commit made on GitHub  
- CodePipeline triggered automatically  
- CodeBuild executed build  
- Docker image updated on Docker Hub with **“Just Now”** timestamp

---

# ✅ Key Takeaways

- CI automated using AWS native tools  
- Credentials stored securely in SSM  
- Privileged mode required for Docker  
- buildspec.yml controls entire process  
- GitHub + AWS = real industry setup

---

## 🔜 Next Session

👉 Day-15 → Continuous Deployment (CD)  
Deploy Docker image to EC2 / Kubernetes


====================================================================

# 🚀 Day-15 | AWS ULTIMATE CI/CD PIPELINE

This document summarizes **Day-15 | AWS Ultimate CI/CD Pipeline** session.  
The focus is on **Continuous Delivery (CD)** — deploying the Docker image built in Day-14 to an EC2 instance using **AWS CodeDeploy**.

---

## 🔷 1. Architecture – Connecting the Dots

### 🎯 Goal  
Automate the final step:

> **Docker Hub Image → EC2 Server Deployment**

### Components Used

| Purpose | Service |
|-------|---------|
| Orchestration | AWS CodePipeline |
| Build Output | Docker Hub |
| Deployment Service | AWS CodeDeploy |
| Target Server | EC2 Instance |

CodePipeline connects:
👉 **CodeBuild (CI) → CodeDeploy (CD)**

---

## 🟩 2. Prerequisite – CodeDeploy Agent

Unlike CodeBuild, **CodeDeploy is NOT serverless**.

### 🔧 Requirement  
A software bridge must run inside EC2:

- ✅ CodeDeploy Agent (Ruby based)
- Polls CodeDeploy service continuously  
- Asks → *“Do you have new deployment for me?”*

### Installation Steps

- Install:
  - ruby  
  - wget  
- Download agent from:
  ```
  aws-codedeploy-<region> S3 bucket
  ```

---

## 🟨 3. Step-1 – EC2 & IAM Setup

### 🅐 IAM Roles Needed

#### 1. Service Role  
Allows **CodeDeploy → EC2 communication**

#### 2. Instance Role  
Allows **EC2 → CodeDeploy & S3 access**

---

### 🅑 Tagging – MOST IMPORTANT 🔥

CodeDeploy does NOT use IP addresses.

> It finds servers using TAGS

Example used:

```
Name : sample-python-app
```

Deployment group targets instances using this tag.

---

## 🟪 4. Step-2 – Configure CodeDeploy

### 🅐 Application  
Logical container  
Example: `PaymentService`

### 🅑 Deployment Group – Defines “WHERE”

- Select EC2 using Tags

### 🅒 Deployment Strategy

Used in session:

- ✅ In-Place Deployment  
- ❌ Blue/Green (kept for advanced)

---

## 🟦 5. Step-3 – appspec.yml (Heart of CD)

Like CI uses `buildspec.yml`,  
CD uses **appspec.yml**

### 📁 Location  
> Must be in ROOT of GitHub repo

---

### 🔁 Lifecycle Hooks Used

| Hook | Purpose |
|----|---------|
| ApplicationStop | Remove old container |
| AfterInstall | Pull & run new image |

### Scripts Used

- `stop_container.sh`  
- `start_container.sh`

---

## 🟥 6. Troubleshooting – Real World Issues

### ❌ Issue 1 – Missing appspec.yml  
Deployment failed instantly

✅ Fix → Add file to repo root

---

### ❌ Issue 2 – Docker Not Installed

Error from script

✅ Fix:
```bash
sudo apt install docker.io
```

---

### ❌ Issue 3 – Port Already In Use (5000)

#### Problem  
Old container still running  
New container cannot bind same port

#### Solution  
Cleanup in:

> ApplicationStop Hook

---

## 🟧 7. Assignment – Cleanup Script

### 🎯 Task  
Create robust:

```
stop_container.sh
```

### Logic

1. Check running container → `docker ps`  
2. Capture container ID  
3. Force remove  
   ```
   docker rm -f <ID>
   ```

### Purpose

> Every deployment starts with a clean slate

---

## 🟩 8. Final Result – FULL AUTOMATION 🎉

### End-to-End Flow

1. Developer commits to GitHub  
2. CodePipeline triggers  
3. CodeBuild builds Docker Image  
4. CodeDeploy deploys to EC2  
5. Website updates automatically

---

# ✅ Key Takeaways

- CodeDeploy requires agent on EC2  
- Tags used instead of IP  
- appspec.yml controls deployment  
- Lifecycle hooks manage containers  
- Cleanup script avoids port conflict  
- Fully automated CI → CD pipeline

---

## 🔜 Next Steps

- Blue/Green Deployment  
- Load Balancer Integration  
- Zero Downtime Releases

---

### 💡 Interview Pointers

- Difference: buildspec.yml vs appspec.yml  
- Why tags over IP?  
- Role of CodeDeploy agent  
- In-Place vs Blue/Green  
- Port conflict handling


============================================================================================

# 📘 Day-16 | AWS CloudWatch Deep Dive

This document summarizes **Day-16 | AWS CloudWatch Deep Dive** session.  
The focus is on **Monitoring & Observability** in AWS.

> 🧠 If IAM is the *Security Guard*, CloudWatch is the **Watchman / Gatekeeper** that records everything happening inside your AWS account.

---

## 🔷 1. What is CloudWatch? – The Gatekeeper

### 🟢 Definition
CloudWatch is a **monitoring and observability service** that watches AWS resources such as:

- EC2  
- S3  
- CodeBuild  
- Lambda  
- and many more

### 🧩 Gatekeeper Analogy

Just like a security guard notes:

> “Who entered? What did they do? At what time?”

CloudWatch answers:

> ❓ *What happened to my server while I was asleep?*

---

### 🧱 4 Core Features of CloudWatch

1. **Monitoring** – Track health of resources  
2. **Metrics** – Numeric data about usage  
3. **Alarms** – Take action when limits cross  
4. **Logs** – Detailed event history  

---

## 🟩 2. Deep Dive – Metrics & Alarms

These two work together like:

> 🧪 Thermometer (Metrics) + Fire Alarm (Alarms)

---

### 🅐 Metrics – The Data

A **Metric** is a measurable value representing behavior.

#### Examples

- CPU Utilization  
- Disk Read / Write  
- Network Input / Output  
- API Request Counts  

#### Important Points

- AWS provides **1,036 default metrics** out of the box  
- EC2 sends CPU data every:
  - 5 minutes (default)  
  - 1 minute (with detailed monitoring)

> ⚠️ By default CloudWatch tracks CPU, but **NOT Memory (RAM)**  
To monitor RAM → you need **Custom Metrics**

---

### 🅑 Alarms – The Action

Collecting metrics is useless without action.

#### How Alarms Work

1. You set a **Threshold**
   - Example: `CPU > 50%`
2. When crossed → Alarm triggers
3. Action happens via **SNS**

#### Common Actions

- Email notification  
- Slack message  
- SMS alert  

---

## 🟨 3. Deep Dive – Log Groups

### 📝 Automated Logging

CloudWatch automatically creates **Log Groups** for AWS services.

#### Example

- AWS CodeBuild build logs  
- Even if project is deleted  
- Logs remain in CloudWatch

> 🕵️ Useful to debug failures from weeks ago

---

## 🟪 4. Practical Demo – CPU Spike Alert

### 🎯 Objective  
Detect when EC2 CPU crosses 50%

---

### Step-1: Setup

- Launch Ubuntu EC2 (T2.micro)

---

### Step-2: Simulation

- Used Python script:

```
CPU_spike.py
```

- Artificially pushed CPU to **50–100%**

---

### Step-3: Alarm Configuration

Path:

```
CloudWatch → Alarms → Create Alarm
```

- Metric: EC2 → Per-Instance → CPU Utilization  
- Condition: **>= 50%**

---

### Step-4: SNS Notification

- Create SNS Topic  
- Add Email  

> 🚨 CRUCIAL STEP  
You must click **Confirm Subscription** in inbox  
Otherwise alarm will NOT work

---

### ✅ Result

- Python script increased CPU  
- CloudWatch detected spike  
- Email received:

> “Danger EC2 instance CPU reached 50%”

---

## 🟥 5. Topics Postponed

Due to time constraints:

### Coming Soon

1. **Custom Metrics**
   - Tracking Memory / RAM

2. **Cost Optimization**
   - Using CloudWatch to reduce AWS bills  
   - Moved to **Day-18 after Lambda**

---

# 🎯 Key Takeaways

- CloudWatch = Monitoring + Logs + Alarms  
- Default tracks CPU, not RAM  
- Metrics are numbers  
- Alarms take actions  
- SNS connects alarms to humans  
- Logs remain even after service deletion  

---

## 💡 Interview Pointers

- Difference between Metrics vs Logs  
- Why RAM not default metric?  
- Role of SNS in CloudWatch  
- Detailed vs Basic monitoring  
- Use of Log Groups for debugging  

---

### 🚀 Next

- Custom Metrics for Memory  
- Lambda + CloudWatch automation  
- Cost alerts & optimization

=================================================================

# 🚀 Day-17 | AWS LAMBDA INTRODUCTION

This document summarizes **Day-17 – AWS Lambda Introduction** session.  
The focus shifts from **Server-based architecture (EC2)** to **Serverless Architecture (Lambda)**, mainly from a **DevOps Automation** perspective rather than pure development.

---

## 🔷 1. What is AWS Lambda? – Serverless Compute

### 🧩 The Problem it Solves

Like EC2, Lambda provides **Compute Power**, but:

> ❌ No server management  
> ❌ No OS patching  
> ❌ No scaling configuration

AWS handles everything — you only provide the **code**.

---

### ⚙️ Lambda Workflow

1. **Event Trigger**  
   - File uploaded to S3  
   - CloudWatch schedule  
   - API request  

2. **Spin Up** – AWS creates a tiny runtime environment  
3. **Execute** – Your code runs  
4. **Tear Down** – Environment is destroyed instantly  

> 💡 You pay only for the few milliseconds your code runs.

---

## 🟩 2. EC2 vs Lambda – The Real Comparison

| Feature | EC2 (Server) | Lambda (Serverless) |
|-------|---------------|----------------------|
| Management | Manual – patch, update, maintain | Fully automated by AWS |
| Cost Model | Pay per hour (even if idle) | Pay per execution time |
| Visibility | Full control of OS & IP | No server visibility |
| Best For | 24/7 applications | Short event-driven tasks |

### 🍔 Food Delivery Analogy

- **EC2 = Owning a Restaurant** → fixed cost  
- **Lambda = Ordering Food Only When Hungry** → pay per use

---

## 🟦 3. DevOps Use Cases – Why Lambda Matters

Developers use Lambda for apps,  
DevOps Engineers use Lambda for:

- Automation  
- Cost Optimization  
- Security Compliance  

---

### 🅐 Cost Optimization (Day-18 Project Preview)

#### Scenario

A developer creates an **EBS volume** and forgets it →  
It stays unused for months → 💸 money wasted.

#### Lambda Solution

- CloudWatch triggers Lambda daily  
- Lambda scans for unused volumes  
- Deletes or sends notification

> ✅ Cheaper than running EC2 just to execute a script

---

### 🅑 Security Compliance

#### Scenario

Company bans:

- Public S3 buckets  
- gp2 EBS volumes  

#### Lambda Solution

- Detect violation instantly  
- Send SNS alert  
- Auto-remediate

---

## 🟪 4. Technical Components of Lambda

### 🧠 Runtimes

Supported:

- Python  
- Java  
- Go  
- Node.js  

> ❗ Limitation: Shell scripting NOT supported

---

### 🧩 Lambda Handler

- Entry point of the function  
- First method AWS executes  

---

### 🔔 Triggers (Event Driven)

Lambda needs a “kick”:

- CloudWatch Events → run at 10 AM daily  
- S3 Events → on file upload  
- API Gateway → HTTP request

---

### 🔐 IAM Roles

By default Lambda has **ZERO permissions**.

To delete EBS → attach IAM role with:

- EC2 permissions  
- S3 permissions  
- SNS permissions (as required)

---

## 🟨 5. Practical Demo

### Step-1: Create Function

- Runtime: Python  
- Default template modified

---

### Step-2: Code Output

```
Hello from AWS Zero to Hero
```

---

### Step-3: Public Access

- Enabled **Function URL**  
- Browser executed Lambda directly

---

### Step-4: Environment Variables

- Store configs outside code  
- Change without redeploy

---

## 🟥 Next Steps – Day 18 Preview

### Real-World Project

- Lambda + CloudWatch  
- Detect stale resources  
- Cost Optimization automation

---

# 🎯 Key Takeaways

- Lambda = Serverless compute  
- Pay only when code runs  
- Event-driven architecture  
- No OS / patch management  
- Perfect for DevOps automation

---

## 💡 Interview Points

- EC2 vs Lambda differences  
- Why Lambda cheaper for cron jobs  
- Role of IAM in Lambda  
- What is Handler  
- Common triggers

---

### 🚀 What’s Next?

- Lambda + CloudWatch automation  
- Delete unused EBS  
- Real cost saving project

===========================================================

# 💰 Day-18 | AWS Cost Optimization – Real World Project

This document summarizes **Day-18 – AWS Cost Optimization** session.  
It is the practical implementation of **AWS Lambda (Day-17)** focused on a high-value DevOps use case:

> 🎯 **Automated Cleanup of Stale Cloud Resources**

---

## 🔴 1. The Problem – “Stale Resources”

### ❗ The Cloud Myth
Moving to cloud ≠ automatic cost savings.  
If resources are not managed properly → **costs can increase.**

---

### 🧪 Real Scenario

1. Developer launches an **EC2 instance**  
2. Takes **EBS Snapshots** (backups)  
3. Later deletes:
   - EC2 instance  
   - EBS volume  

❌ But forgets to delete snapshots

---

### 💸 The Impact

- AWS continues billing for snapshots  
- Storage remains forever  
- These are called:

> 🗑 **STALE RESOURCES**

---

### 👨‍💻 DevOps Responsibility

A DevOps Engineer must:

- Detect unused resources  
- Automate cleanup  
- Reduce cloud bill

---

## 🟢 2. The Solution – Serverless Automation

Instead of manual checking → build automation with:

| Component | Purpose |
|---------|---------|
| AWS Lambda | Run cleanup code |
| Python (boto3) | Talk to AWS APIs |
| CloudWatch Event | Trigger daily/weekly |

---

### 🧱 Architecture

- EventBridge → triggers Lambda  
- Lambda → scans snapshots  
- boto3 → calls EC2 APIs  
- Deletes unused snapshots

---

## 🧠 3. The Logic – Algorithm

### Step-by-step Decision Flow

1. **Fetch** all EBS Snapshots in account  
2. For each snapshot:
   - Does parent volume exist?

### If NO  
> ➜ Volume already deleted → **Delete Snapshot**

### If YES  
- Is volume attached to running EC2?

### If NO  
> ➜ Volume idle → **Delete Snapshot**

---

## 🟣 4. Implementation – Hands On

### 🅐 Phase A – Simulation

1. Launch EC2 instance  
2. Create snapshot  
3. Terminate instance  

> Snapshot remains → simulates waste scenario

---

### 🅑 Phase B – Lambda Setup

1. Create Lambda Function  
   - Runtime: Python  

2. Timeout Setting  
   - Default: 3 sec ❌  
   - Change to: 10 sec ✅  

3. Paste boto3 script from course GitHub

---

### 🅒 Phase C – IAM Permissions (CRITICAL)

Without permissions Lambda will fail ❌

#### Required Actions

- `ec2:DescribeSnapshots` → list snapshots  
- `ec2:DeleteSnapshot` → delete them  
- `ec2:DescribeVolumes` → check parent volume  
- `ec2:DescribeInstances` → check running state

> 🔐 IAM Role must be attached to Lambda

---

## 🟡 5. Resume & Interview Value

### 🔥 Resume Point

You can mention:

> “Implemented an event-driven AWS Lambda solution using Boto3 to automatically detect and delete stale EBS snapshots not attached to active volumes, reducing cloud storage costs.”

---

## 🎯 Key Takeaways

- Cloud ≠ cost saving by default  
- Automation is mandatory  
- Lambda + boto3 = powerful combo  
- Event-driven cost governance  
- Real DevOps responsibility

---

## ❓ Interview Questions You Can Answer

- What are stale snapshots?  
- How Lambda helps in cost optimization?  
- Why IAM role is required?  
- Difference between manual vs automated cleanup?  
- How EventBridge triggers Lambda?

---

## 🚀 Next Steps

- Integrate SNS alerts before delete  
- Add tagging validation  
- Weekly cost reports  
- Multi-region cleanup

================================================================

# 🌍 Day-19 | AWS CloudFront – CDN Deep Dive

This document summarizes **Day-19 – AWS CloudFront** session featuring guest speaker **Piyush**.  
The focus is on **Content Delivery Networks (CDN)** and how CloudFront improves performance, security, and cost efficiency.

---

## 🚀 1. What is CloudFront?

### 🧩 Definition
**AWS CloudFront** is a fully managed **Content Delivery Network (CDN)** service.

---

### ❗ The Problem – Latency

#### Instagram Analogy
- Assume Instagram server is in **North Virginia (USA)**
- A user in **India** requests a photo  
- Without CDN → request travels through many routers (“hops”) across the globe  
- Result:

> 🐌 High Latency → Slowness → Buffering → Bad User Experience

---

### ✅ The Solution – Edge Locations

With CloudFront:

- AWS stores **cached copies** of content in data centers near users  
- These are called **Edge Locations** (e.g., Mumbai)

👉 User in India downloads from **Mumbai Edge**, not USA

> ⚡ Result: Instant loading & better performance

---

## 🔐 2. Why Use CloudFront with S3?

Hosting website directly on S3 (as done in Day-8) is **not production best practice**.

### ❌ Problems with Direct S3 Hosting

1. **Latency**  
   - S3 exists in one region only  
   - Far users experience slowness

2. **Security**  
   - S3 must be Public  
   - Anyone can access files directly

3. **Cost**  
   - Direct S3 download = expensive  
   - CloudFront caching = cheaper

---

## 🛡 3. Key Concept – OAI (Origin Access Identity)

### 🎯 Goal
Users must access website **ONLY via CloudFront**, never directly via S3.

---

### 🧠 How OAI Works

- Create a virtual identity → **Origin Access Identity**
- Update S3 Bucket Policy:

> “Allow access ONLY to this OAI. Deny everyone else.”

---

### 🟢 Result

| Access Method | Outcome |
|---------------|---------|
| Direct S3 Link | ❌ Access Denied |
| CloudFront Link | ✅ Allowed |

---

## 🛠 4. Practical Setup Steps

### Step 1 – Create Origin
- Create S3 bucket  
- Upload `index.html`

---

### Step 2 – Create Distribution
- Select S3 as **Origin Domain**

---

### Step 3 – Security Config

- Enable **Origin Access Identity (OAI / OAC)**
- Click:

> ✅ “Update Bucket Policy”

AWS automatically secures the bucket.

---

### Step 4 – Region Settings

You can choose:

- 🌐 All Edge Locations (Global)  
- 🌎 Specific Regions (e.g., US & Europe only)

> Limiting regions → faster deploy + lower cost

---

### Step 5 – Default Root Object

Set:

```
index.html
```

So `domain.com` opens homepage automatically.

---

### Step 6 – WAF (Firewall)

- Disabled for demo  
- Reason:

> 💵 WAF base cost ≈ $14/month

---

## 🧪 5. Verification

| Link Type | Result |
|-----------|--------|
| S3 URL | ❌ Forbidden |
| CloudFront URL | ✅ Website Loads Fast |

---

### ⚠ Pricing Warning

- CloudFront has free tier  
- Wrong config (like WAF) → charges  
- **Always delete after practice**

---

## 🧹 6. Cleanup (Very Important)

### Follow This Order

1. **Disable** CloudFront Distribution  
2. Wait for disable (few minutes)  
3. **Delete** Distribution

> This prevents unwanted billing.

---

## 🎯 Key Takeaways

- CDN reduces latency  
- CloudFront = global caching  
- OAI secures S3 origin  
- Cheaper than direct S3  
- Essential for production websites

---

## 💬 Interview Questions You Can Answer

- What is CloudFront and why use it?  
- Difference between S3 hosting vs CloudFront?  
- What is OAI/OAC?  
- How CloudFront improves security?  
- How caching reduces cost?

---

## ➡ Next Steps

- Add Custom Domain + ACM SSL  
- Enable Cache Policies  
- Integrate WAF rules  
- Logging & Monitoring

==========================================================

# 📦 Day-20 | AWS ECR – Elastic Container Registry

This document summarizes **Day-20 – AWS ECR** session.  
The topic covers **Elastic Container Registry (ECR)** — the AWS equivalent of **Docker Hub** used to store and manage container images.

---

## 🔍 1. What is ECR?

### 🧩 Acronym Breakdown

| Term | Meaning |
|-----|---------|
| **Elastic** | Highly scalable and highly available service managed by AWS |
| **Container** | Stores container images (code + dependencies) |
| **Registry** | Central storage to share/download images |

### 🧠 Simple Analogy

> Just like you upload photos to Instagram to share them,  
> you upload container images to a **Registry** so servers or teams can download them.

---

### 📌 Important Note

AWS documentation sometimes calls ECR a **“Docker Registry.”**

👉 In reality, ECR supports **OCI (Open Container Initiative)** standards  
→ Works with **Docker, Podman, Buildah**, not just Docker.

---

## 🎯 2. ECR vs Docker Hub (Interview Question)

| Feature | Docker Hub | AWS ECR |
|-------|------------|---------|
| Default Visibility | 🌍 Public – ideal for open source | 🔒 Private – ideal for enterprises |
| Access Control | Separate accounts & teams | Integrated with **AWS IAM** |
| Integration | External tool | Native with **ECS, EKS, Fargate** |
| Support | Third-party | Direct AWS support |

### 💡 Conclusion

> Companies prefer **ECR** because security and access are controlled through existing AWS IAM users and roles.

---

## 🛠 3. Practical Demo – Push Image to ECR

### Step 1 – Create Repository

- Open **ECR Console**
- Click **Get Started**
- Default visibility → **Private**

#### Optional Feature
✅ **Scan on Push**

> AWS automatically scans images for security vulnerabilities when uploaded.

---

### Step 2 – View Push Commands

- After creating repo, AWS provides:

> 🧾 “View Push Commands”

- Gives exact copy-paste commands for:
  - Linux / Mac  
  - Windows

#### Prerequisite
✔ AWS CLI must be installed & configured

---

### Step 3 – Authentication (Login)

You cannot login with normal username/password.

#### Command Concept

```bash
aws ecr get-login-password | docker login
```

### How It Works

1. AWS CLI generates **temporary auth token**  
2. Token is piped (`|`) into docker login  
3. Local Docker client gets authenticated with ECR

---

### Step 4 – Tag & Push Image

#### 1. Build Locally

```bash
docker build -t test-image .
```

#### 2. Tag Image

You must rename image to match ECR format:

```
aws_account_id.dkr.ecr.region.amazonaws.com/repo-name:latest
```

#### 3. Push

```bash
docker push <ecr-image-url>
```

---

## 🔐 4. IAM Permissions

### If Using Root Account
- Access works by default

### If Using IAM User
You must attach:

- **ECR Power User** policy  
  OR  
- Specific permissions:
  - PutImage  
  - GetDownloadUrlForLayer  
  - BatchCheckLayerAvailability

> ❗ Without permissions → push will FAIL

---

## 🎓 Key Takeaways

- ECR = Private Docker Hub of AWS  
- Supports OCI standard tools  
- Deep integration with IAM  
- Used with ECS / EKS / Fargate  
- Secure & enterprise-ready

---

## 💬 Interview Questions You Can Answer

- What is AWS ECR and why use it?  
- Difference between ECR and Docker Hub?  
- How authentication works in ECR?  
- Why tagging is required before push?  
- How IAM controls ECR access?

---

## ➡ Next Steps

- Use ECR with **ECS / EKS deployment**  
- Enable **image scanning**  
- Implement **lifecycle policies** to delete old images  
- Integrate with **CI/CD pipeline**

---

=========================================================

