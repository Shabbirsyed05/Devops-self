# ☁️ AWS Zero to Hero — Interview Prep Guide (Days 0–10)

> 📺 Based on the **Free 30-Day AWS Zero to Hero Course** by Abhishek
> 🎯 Purpose: Master AWS fundamentals from a DevOps Engineer's perspective

---

## 📋 Table of Contents

| Day | Topic |
|-----|-------|
| [Day 0](#-day-0-course-roadmap) | Course Roadmap (30-Day Syllabus) |
| [Day 1](#-day-1-introduction-to-aws--cloud) | Introduction to AWS & Cloud |
| [Day 2](#-day-2-aws-iam-deep-dive) | AWS IAM Deep Dive |
| [Day 3](#-day-3-ec2-deep-dive--deploy-jenkins) | EC2 Deep Dive & Deploy Jenkins |
| [Day 4](#-day-4-vpc-explained) | VPC Explained |
| [Day 5](#-day-5-security-groups--nacl) | Security Groups & NACL |
| [Day 6](#-day-6-route-53) | Route 53 (DNS as a Service) |
| [Day 7](#-day-7-aws-production-capstone-project) | Production Capstone Project |
| [Day 8](#-day-8-aws-scenario-based-interview-questions) | Scenario-Based Interview Questions |
| [Day 9](#-day-9-aws-s3-deep-dive) | AWS S3 Deep Dive |
| [Day 10](#-day-10-aws-cli-deep-dive) | AWS CLI Deep Dive |
| [Master Cheatsheet](#-master-cheatsheet-days-010) | Master Cheatsheet |

---

---

# 🗺️ Day 0: Course Roadmap

**Goal:** 30-day roadmap to go from zero to job-ready DevOps Engineer on AWS.

> ⚠️ Focused on **real-world DevOps usage**, not certification memorization.

---

## 🗓️ 30-Day Plan

| Week | Focus | Topics |
|---|---|---|
| **Week 1** | Core Foundation | EC2, IAM, VPC, Security Groups, Route 53 |
| **Week 2** | Storage & CI/CD | S3, CloudFormation, CLI, CodePipeline, CodeBuild, CodeDeploy |
| **Week 3** | Monitoring & Serverless | CloudWatch, Lambda, CloudTrail, DynamoDB |
| **Week 4** | Advanced Ops | EKS, SSM, Auto Scaling, RDS, Load Balancers |
| **Final** | Career Prep | Cloud Migration Strategy, Resume, Job Search |

**Key interview topics:** Lambda + CloudWatch event-driven architecture (Day 17), EKS over ECS (higher interview relevance).

---

---

# ☁️ Day 1: Introduction to AWS & Cloud

**Goal:** Understand why cloud replaced data centers and why AWS dominates.

---

## 🔵 The Evolution: Data Centers → Cloud

### ❌ The Old Way (Physical Data Centers)
- Companies bought servers from IBM/HP — stored in their own facilities
- Required managing: cabling, temperature control, physical security
- **The waste:** 100GB RAM server running a 1GB app → 99% idle

### ✅ The Fix: Virtualization
- One physical server split into multiple Virtual Machines
- 1 physical server → 15 VMs → 15 teams served

### ☁️ What "Cloud" Means
> Accessing virtual resources remotely — without knowing or caring where the physical hardware is located.

---

## 🔵 Private vs Public Cloud

| Type | Who Owns It | Who Manages It |
|---|---|---|
| **Private Cloud** | Your organization | You manage hardware + virtualization |
| **Public Cloud** | AWS / Azure / GCP | Provider manages datacenter — you just use resources |

**Why Public Cloud wins:**
- Zero hardware maintenance
- Start small, scale infinitely
- Pay only for what you use

---

## 🔵 Why AWS?

- **First Mover:** AWS pioneered cloud computing — largest market share
- **Job Market:** Most companies require AWS skills
- **Transferable:** Learning AWS makes Azure/GCP easy to adapt to

> 📌 "Cloud Repatriation" (moving back on-prem) affects only ~1–2% of companies.

---

## 🔵 AWS Account Setup

| Setting | Detail |
|---|---|
| Account type | Personal |
| Credit card | Required for identity verification (~$1 charge, refunded) |
| Billing | Monthly bill — account suspended if unpaid |

### 🎙️ Interview Answer: "What is cloud computing?"
> *"Cloud computing is accessing computing resources — servers, storage, networking — on demand over the internet from a provider like AWS. Instead of buying and maintaining physical hardware, you rent virtual resources and pay only for what you use. AWS dominates because it was first to market and has the largest service catalog."*

---

---

# 🔐 Day 2: AWS IAM Deep Dive

**Goal:** Implement least-privilege security — never give everyone root access.

---

## 🔵 The Problem: Root Access is Dangerous

```
New AWS account = Root User (can do EVERYTHING)
├── Delete databases ✅
├── Launch expensive servers ✅
└── Destroy the entire account ✅
```

> Giving 1,000 employees the root password = one person can destroy everything.

---

## 🔵 Authentication vs Authorization

| Concept | What It Checks | Bank Analogy | AWS Context |
|---|---|---|---|
| **Authentication** | Who are you? | Security guard checks your ID | Login with Username + Password |
| **Authorization** | What can you do? | Teller checks your account permissions | Policies attached to your user |

> ⚠️ **A user can be authenticated but have zero authorization** — logged in but can't see or do anything.

---

## 🔵 The 4 IAM Components

### 👤 Users — The Person
- Created for real humans (Devs, QAs, Admins)
- One IAM user per employee — never share
- Force password reset on first login

### 📜 Policies — The Permissions
- JSON documents defining what's allowed/denied
- Without a policy, a user **can do nothing**
- Use AWS Managed Policies for common needs (e.g., `AmazonS3FullAccess`)

### 👥 Groups — The Efficiency Tool
- Create group (e.g., `Dev-Team`) → attach policies → add users
- Policy update applies to **all group members automatically**
- Scales from 5 to 5,000 users easily

### 🏷️ Roles — The Service Identity
- For **machines/services**, not humans — no username/password
- Use when: EC2 needs to talk to S3/RDS
- **Never hardcode credentials in code — use Roles instead**

---

## 🔵 Practical Demo Flow

```
1. Create user (Authentication only)
2. Login → Access S3 → ❌ Access Denied (no authorization)
3. Admin attaches AmazonS3FullAccess policy
4. Login again → ✅ Can list and create S3 buckets
5. Create "Development" Group → attach S3 + EC2 policies → add user → scales!
```

### 🎙️ Interview Answer: "IAM Users vs Roles?"
> *"IAM Users are for humans — each person gets their own user with long-term credentials. IAM Roles are for services — when an EC2 instance needs to access S3, I attach a Role with the appropriate permissions instead of hardcoding credentials in the application. Roles provide temporary credentials automatically via the instance metadata, which is far more secure."*

---

---

# 🖥️ Day 3: EC2 Deep Dive & Deploy Jenkins

**Goal:** Launch virtual servers on AWS and deploy a real application.

---

## 🔵 What is EC2?

**EC2 = Elastic Compute Cloud**

| Word | Meaning |
|---|---|
| **Elastic** | Scale up/down automatically based on demand |
| **Compute** | CPU and RAM processing power |
| **Cloud** | Hosted in AWS data centers — not physical hardware |

**Why use EC2:** Zero maintenance on hardware. Pay-as-you-go — stop instance = no charge.

---

## 🔵 EC2 Instance Types

| Family | Best For |
|---|---|
| **General Purpose** (t2, t3) | Balanced workloads — use this for learning (Free Tier) |
| **Compute Optimized** (c5) | Gaming servers, ML inference |
| **Memory Optimized** (r5) | Big data analytics, large databases |
| **Storage Optimized** (i3) | High read/write IOPS |
| **Accelerated Computing** (p3) | GPU — graphics, deep learning |

---

## 🔵 Regions vs Availability Zones

| Concept | What It Is | Example |
|---|---|---|
| **Region** | Geographic location | Mumbai, N. Virginia, Frankfurt |
| **Availability Zone (AZ)** | Data center inside a region | us-east-1a, us-east-1b |

> 🎯 Deploy across multiple AZs — if one AZ has a power outage, app runs in the other.

**Region selection criteria:** Latency (close to users) + Compliance (data residency laws)

---

## 🔵 Launch EC2 Checklist

| Step | Choice | Note |
|---|---|---|
| OS | Ubuntu | Most common in DevOps |
| Instance type | t2.micro | ⭐ Free Tier — 750 hrs/month |
| Key pair | Create `.pem` file | = Your server password, never lose it |
| Security Group | Configure ports | Default: only SSH (22) open |

---

## 🔵 SSH Access & Permission Fix

```bash
# Connect
ssh -i "key.pem" ubuntu@<Public-IP>

# Common error: "Permissions 0644 are too open"
chmod 600 key.pem   # Fix — restrict to owner only
```

---

## 🔵 Deploy Jenkins on EC2

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y    # Jenkins requires Java
# Install Jenkins via apt (from jenkins.io)
sudo systemctl start jenkins
sudo systemctl status jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Jenkins runs on Port 8080** — must open in Security Group:
```
EC2 → Security Groups → Inbound Rules → Custom TCP → Port 8080 → 0.0.0.0/0
```

### 🎙️ Interview Answer: "What is EC2?"
> *"EC2 is AWS's virtual server service. Elastic means it scales automatically, Compute means CPU/RAM processing power, and Cloud means it's hosted in AWS data centers. I pick instance types based on workload — t2.micro for dev/testing (Free Tier), compute-optimized for ML, memory-optimized for large databases. For high availability, I deploy across multiple Availability Zones within a Region."*

---

---

# 🌐 Day 4: VPC Explained

**Goal:** Understand your private, isolated network inside AWS.

---

## 🔵 The "Gated Community" Analogy

| Real World | AWS Equivalent | Purpose |
|---|---|---|
| Village / City | **AWS Region** | Physical location |
| Secure Gated Property | **VPC** | Your private isolated network |
| House | **EC2 Instance** | Your server |
| Gate | **Internet Gateway (IGW)** | Entry point for internet traffic |
| Internal Roads | **Route Tables** | Traffic direction inside VPC |
| Security Guard | **Security Group** | Controls access to each instance |

> **A VPC is your own private, gated network inside AWS.**

---

## 🔵 Network Sizing: CIDR

```bash
172.16.0.0/16   # Defines how many IPs (houses) you can create inside the VPC
```

---

## 🔵 Subnets: Public vs Private

| | Public Subnet | Private Subnet |
|---|---|---|
| **Internet access** | ✅ Yes | ❌ No |
| **Contains** | Load Balancers, NAT Gateway | Databases, App servers |
| **Exposed** | Yes — internet-facing | No — hidden for security |

---

## 🔵 Traffic Flow: User → EC2

```
Internet → Internet Gateway → Public Subnet (Load Balancer)
         → Route Table → Security Group check → EC2 (Private Subnet)
```

---

## 🔵 NAT Gateway: Private → Internet (One-Way)

**Problem:** App server in private subnet needs to download updates — but can't directly reach internet.

**Solution:**
```
Private EC2 → NAT Gateway (in Public Subnet) → Internet
                   ↑
       NAT hides private IP, uses its own public IP
```

> Private server gets internet access **without exposing its IP** to the world.

### 🎙️ Interview Answer: "Explain VPC"
> *"A VPC is a logically isolated network inside AWS — think of it as a gated community in the cloud. Inside the VPC I create subnets: public subnets for load balancers and NAT gateways that need internet access, and private subnets for databases and app servers that should never be directly reachable from the internet. An Internet Gateway allows inbound traffic, Route Tables direct traffic internally, and Security Groups control access at the instance level."*

---

---

# 🛡️ Day 5: Security Groups & NACL

**Goal:** Control traffic in two layers — instance-level and subnet-level.

---

## 🔵 Shared Responsibility Model

| AWS Responsible For | You Responsible For |
|---|---|
| Physical data centers | Configuring Security Groups correctly |
| Hardware | Opening/closing the right ports |
| Underlying infrastructure | Protecting your data |

> If you open "All Traffic" and get breached — **that is your fault, not AWS's.**

---

## 🔵 Layer 1: Security Groups (Instance Firewall)

| Direction | Default | Rule Type |
|---|---|---|
| **Inbound** | Deny All | Allow rules only (no deny) |
| **Outbound** | Allow All | Port 25 (email) blocked by default |

**Stateful:** If inbound is allowed → return traffic is automatically allowed.

> ⚠️ **You cannot create Deny rules in Security Groups.** Only Allow.

---

## 🔵 Layer 2: Network ACL (Subnet Firewall)

| Feature | Security Group | NACL |
|---|---|---|
| **Level** | EC2 Instance | Entire Subnet |
| **Rules** | Allow only | Allow AND Deny |
| **Evaluated first?** | No — second | Yes — first |
| **Stateful?** | ✅ Yes | ❌ No (stateless) |
| **Rule order** | All evaluated | First match wins (numbered) |

---

## 🔵 NACL Rule Priority (Critical)

```
Rule 100 → Deny Port 8000
Rule 200 → Allow Port 8000

Result: DENIED — AWS stops at Rule 100 and doesn't read Rule 200
```

**Use case for NACL:**
> Block a hacker's IP or entire country — update **one NACL** instead of 50 Security Groups.

---

## 🔵 The Two-Layer Security Model

```
Internet → [NACL — Outer Gate — subnet level] → [Security Group — Inner Door — instance level] → EC2
```

### 🎙️ Interview Answer: "Security Group vs NACL?"
> *"They're two layers of defense. Security Groups work at the instance level — they're stateful and support only Allow rules. If you allow inbound traffic, the return traffic is automatically allowed. NACLs work at the subnet level — they're stateless, support both Allow and Deny rules, and are evaluated before Security Groups. I use NACLs to block malicious IPs at the subnet boundary, and Security Groups to fine-tune port access per instance."*

---

---

# 🌍 Day 6: Route 53

**Goal:** Map human-friendly domain names to dynamic AWS resources.

---

## 🔵 Why Route 53 Exists

**Problem 1 — Memorability:**
```
EC2 Public IP: 3.6.10.171   ← Nobody remembers this
Domain: amazon.com           ← Everyone remembers this
```

**Problem 2 — Dynamic IPs:**
> Restarting a server or Load Balancer assigns a new IP. Users can't update bookmarks constantly.

**Solution:** DNS maps a **static domain name** to a **dynamic IP** automatically.

---

## 🔵 3 Core Functions of Route 53

### 🅰️ Domain Registration
- Buy domain directly from AWS (or from GoDaddy, Namecheap — same result)
- Example: `myapp.com`

### 🅱️ Hosted Zones — The DNS Database
- Stores DNS records: *"When someone types `myapp.com`, route them to Load Balancer at `1.2.3.4`"*
- Created for every domain you manage

### 🅾️ Health Checks — Automatic Failover
```
Route 53 checks Server A health every 1 minute
    ↓
Server A fails → Route 53 stops sending traffic there
    ↓
Traffic redirected to Server B automatically ✅
```

---

## 🔵 Full Traffic Flow with Route 53

```
User types: amazon.com
     ↓
Route 53 looks up Hosted Zone → finds Load Balancer address
     ↓
Load Balancer distributes traffic to EC2 instances
```

### 🎙️ Interview Answer: "What is Route 53?"
> *"Route 53 is AWS's DNS service. It has three functions: domain registration to buy domain names, Hosted Zones which store DNS records mapping domains to IPs or load balancers, and Health Checks which enable automatic failover — if a server fails, Route 53 automatically routes traffic to a healthy one. Think of it as the phonebook of the internet for your AWS resources."*

---

---

# 🏗️ Day 7: AWS Production Capstone Project

**Goal:** Combine VPC + EC2 + Security + Load Balancer into a real enterprise architecture.

---

## 🔵 The Architecture

```
Internet → Route 53 → Application Load Balancer (Public Subnet)
                              ↓
                    EC2 App Servers (Private Subnet)
                         AZ-1a    +    AZ-1b
```

**Key design decisions:**
- **2 Availability Zones** → if one AZ fails, app stays up
- **Public subnet:** Load Balancer + NAT Gateway (need internet)
- **Private subnet:** App servers (hidden from internet)

---

## 🔵 3 New Components Introduced

| Component | Purpose | Analogy |
|---|---|---|
| **Bastion Host (Jump Server)** | SSH access to private instances | Reception desk of a secure building |
| **Auto Scaling Group (ASG)** | Auto-adds/removes EC2 instances | Manager hiring staff during rush hours |
| **NAT Gateway** | Private servers access internet safely | One-way mirror — outbound only |

---

## 🔵 4-Phase Implementation

### Phase 1: VPC Setup (Use "VPC and more" wizard)
```
1 VPC → 2 AZs → 2 Public Subnets → 2 Private Subnets → 1 NAT Gateway
```

### Phase 2: Auto Scaling Group
```
Launch Template (Ubuntu, t2.micro, Port 8000 SG)
       ↓
Auto Scaling Group → Private Subnets → Desired: 2 (1 per AZ)
```

### Phase 3: Bastion Host (Jump Server)
```bash
# 1. Launch Bastion in Public Subnet
# 2. Copy .pem key from laptop to Bastion
scp -i key.pem key.pem ubuntu@<Bastion-IP>:/home/ubuntu

# 3. SSH into Bastion, then jump to private instance
ssh -i key.pem ubuntu@<Private-IP>
```

### Phase 4: Load Balancer
```
Create Target Group → Port 8000 → Private EC2 instances
Create ALB → Internet-facing → Public Subnets → Listener Port 80 → Target Group
```

**Common mistake — 502 Bad Gateway:**
> ALB Security Group missing Port 80 Inbound from `0.0.0.0/0`

---

## 🔵 Verification

```
Browser → http://<ALB-DNS-name>
Result: "My First AWS Project" ✅

Deploy different HTML on Server B → refresh ALB URL
Observe responses switching between Server A and Server B ✅ (Load Balancing confirmed)
```

### 🎙️ Interview Answer: "Design a highly available architecture on AWS"
> *"I'd deploy across two Availability Zones for high availability. The Load Balancer goes in the public subnet — it's internet-facing. Application servers go in private subnets — never directly exposed. A NAT Gateway in the public subnet lets private servers download updates. An Auto Scaling Group ensures capacity scales with traffic. A Bastion Host provides secure SSH access to private instances. Route 53 provides the DNS layer. This pattern is standard in most production AWS environments."*

---

---

# 🎯 Day 8: AWS Scenario-Based Interview Questions

**Goal:** Apply Days 1–7 knowledge to real interview scenarios.

---

## 🔵 Scenario 1: Design a 2-Tier Highly Available App

**Requirements:** Frontend + Backend, survive AZ failure, handle traffic spikes.

**Answer:**
```
Multi-AZ deployment (us-east-1a + us-east-1b)
Load Balancer → Public Subnets
App Servers → Private Subnets
Auto Scaling Group → adds/removes instances based on load
```

---

## 🔵 Scenario 2: SSH into a Private Instance

**Answer:** Use a **Bastion Host** (Jump Server) in the public subnet. SSH into Bastion → SSH into private instance from there.

---

## 🔵 Scenario 3: Private Instance Needs to Download Updates

**Answer:** **NAT Gateway** in public subnet. Private instance routes traffic through NAT → NAT uses its public IP → response returns to private instance. Private IP never exposed.

---

## 🔵 Scenario 4: Isolate One Subnet from Internet

**Answer:** Edit the **Route Table** for that subnet — remove `0.0.0.0/0 → Internet Gateway`. Without that route, the subnet has no internet path.

---

## 🔵 Scenario 5: Securely Access S3 from VPC

**Answer:** Use a **VPC Endpoint** — creates a private connection between VPC and S3. Traffic never leaves the AWS network. More secure and faster than going over public internet.

---

## 🔵 Scenario 6: Block a Hacker's IP

**Answer:** Use **Network ACL** — add a Deny rule for the malicious IP at the subnet level. One update blocks all 50 EC2 instances in the subnet simultaneously.

---

## 🔵 Scenario 7: EC2 Needs to Write to a Database

**Answer:** Attach an **IAM Role** to the EC2 instance. Never hardcode credentials. The role provides temporary credentials via the instance metadata service automatically.

---

## 🔵 IAM Quick Reference

| Component | For | Has Credentials? |
|---|---|---|
| **User** | Human | Yes (username + password) |
| **Group** | Collection of users | N/A |
| **Policy** | Permissions definition | N/A |
| **Role** | Services / machines | No (temporary tokens) |

---

---

# 📦 Day 9: AWS S3 Deep Dive

**Goal:** Understand AWS's most widely used storage service.

---

## 🔵 What is S3?

> **S3 = Simple Storage Service** — Object storage for files, videos, logs, database backups.

**Key characteristics:**
- **Infinite storage** — no limit (each object up to 5TB)
- **Globally unique bucket names** — but data stored in a specific Region
- **11 Nines durability:** 99.999999999% — lose 1 object per billion per 100 years

---

## 🔵 Key Features

### Versioning (Like Git for files)
```
Upload index.html v1
Overwrite with index.html v2
S3 keeps BOTH → restore v1 anytime
```

### Storage Classes (Cost vs Speed)

| Class | Use Case | Cost | Retrieval Time |
|---|---|---|---|
| **S3 Standard** | Frequently accessed | High | Immediate |
| **S3 Glacier** | Old backups | Low | Minutes–Hours |
| **S3 Deep Archive** | Rarely accessed | Very low (~$1–2/TB) | 12–48 hours |

---

## 🔵 Security: Bucket Policy vs IAM Policy

**Scenario:** IAM user has `S3FullAccess` but you want to block access to one sensitive bucket.

**Conflict:**
```
IAM Policy → ALLOW all S3
Bucket Policy → DENY everyone except owner
Result: Bucket Policy WINS — even admin is blocked ✅
```

> Bucket Policies can override IAM permissions — useful for protecting highly sensitive data.

---

## 🔵 Static Website Hosting on S3 (No EC2 Needed)

```
1. Upload index.html to S3 bucket
2. Properties → Static Website Hosting → Enable
3. Permissions → Uncheck "Block All Public Access"
4. Add Bucket Policy: Allow GetObject for Principal *
5. AWS generates a URL → loads your HTML instantly ✅
```

### 🎙️ Interview Answer: "What is S3 and when do you use it?"
> *"S3 is AWS's object storage — for files, images, database backups, and logs. It offers 11 nines durability through automatic replication across multiple Availability Zones. I use versioning to protect against accidental overwrites, and storage classes to optimize cost — active data in Standard, old backups in Glacier, archived data in Deep Archive. Bucket Policies provide granular security control that can override IAM policies."*

---

---

# 💻 Day 10: AWS CLI Deep Dive

**Goal:** Automate AWS tasks from the terminal instead of clicking through the console.

---

## 🔵 Why CLI over Console?

```
Manager: "Create 10 VPCs and 15 EC2 instances"
Console: Click 100+ buttons → slow, error-prone, not repeatable ❌
CLI:     Write one script → runs in seconds → repeatable ✅
```

---

## 🔵 What is AWS CLI?

> Open-source tool (built in Python) that converts terminal commands into AWS API calls.

```
You type: aws s3 ls
CLI converts to: HTTPS API call to AWS
AWS responds: JSON list of buckets
CLI displays: Readable output
```

---

## 🔵 Installation & Setup

```bash
# Verify installation
aws --version

# Configure with access keys (NOT console username/password)
aws configure
# Enter: Access Key ID, Secret Access Key, Region (us-east-1), Output (json)
```

> ⚠️ **Secret Access Key shown only once — save it immediately. If lost, must regenerate.**

---

## 🔵 Common CLI Commands

```bash
# S3
aws s3 ls                                    # List all buckets
aws s3 mb s3://my-new-bucket                 # Make bucket
aws s3 cp file.txt s3://my-bucket/           # Copy file to S3

# EC2
aws ec2 describe-instances                   # List all EC2 instances
aws ec2 run-instances \
  --image-id ami-xxx \
  --instance-type t2.micro \
  --subnet-id subnet-xxx \
  --security-group-ids sg-xxx               # Launch EC2 instance

# IAM
aws iam list-users                           # List all IAM users
```

---

## 🔵 CLI vs Infrastructure as Code

| Feature | AWS CLI | Terraform / CloudFormation |
|---|---|---|
| **Best for** | Quick tasks, one-off actions | Full architecture, production stacks |
| **Reusability** | Low | High |
| **Complexity** | Simple | Complex |
| **Use when** | Listing resources, quick testing | VPC + Subnets + Load Balancers |

### 🎙️ Interview Answer: "CLI vs Terraform?"
> *"AWS CLI is great for quick, one-off tasks like listing resources or simple automation scripts. For production infrastructure — VPCs, subnets, load balancers, multi-service stacks — I use Terraform because it's declarative, reusable, and maintains state. I think of CLI as the screwdriver and Terraform as the blueprint."*

---

---

# 📌 Master Cheatsheet (Days 0–10)

```
╔══════════════════════════════════════════════════════════════════════╗
║          AWS INTERVIEW CHEATSHEET — DAYS 0-10                        ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  CLOUD BASICS:                                                       ║
║  Private Cloud=you manage | Public Cloud=provider manages           ║
║  AWS = largest market share, first mover advantage                  ║
║                                                                      ║
║  IAM:                                                                ║
║  Authentication=who | Authorization=what you can do                 ║
║  User=human | Group=users+policies | Role=services (no password)    ║
║  Without Policy → user is useless (can't do anything)               ║
║  NEVER hardcode credentials → use IAM Roles                         ║
║                                                                      ║
║  EC2:                                                                ║
║  Elastic=scale | Compute=CPU/RAM | Cloud=virtual                    ║
║  chmod 600 key.pem → fix SSH permission error                       ║
║  Jenkins: Port 8080 | Must open in Security Group                   ║
║                                                                      ║
║  VPC:                                                                ║
║  VPC=private network | Subnet=segment | IGW=internet door           ║
║  Public subnet=LB+NAT | Private subnet=apps+DB                      ║
║  NAT Gateway=private→internet (outbound only, IP hidden)            ║
║                                                                      ║
║  SECURITY:                                                           ║
║  NACL=subnet level, Allow+Deny, stateless, evaluated FIRST          ║
║  SG=instance level, Allow only, stateful, evaluated SECOND          ║
║  NACL rule order matters: first match wins                           ║
║                                                                      ║
║  ROUTE 53:                                                           ║
║  Domain Registration + Hosted Zones + Health Checks (failover)      ║
║                                                                      ║
║  PRODUCTION ARCH:                                                    ║
║  Multi-AZ + ALB (public) + ASG + Private EC2 + Bastion Host         ║
║  Bastion = SSH jump server for private instances                     ║
║  502 Bad Gateway = ALB SG missing Port 80 inbound                  ║
║                                                                      ║
║  S3:                                                                 ║
║  Globally unique bucket name | 11 nines durability                  ║
║  Versioning=restore any version | Glacier=cheap long-term           ║
║  Bucket Policy can override IAM policy                               ║
║                                                                      ║
║  CLI:                                                                ║
║  aws configure → Access Key + Secret Key + Region + Format          ║
║  CLI=quick tasks | Terraform=production architecture                 ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Delivery Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Use analogies (gated community, phonebook) | Give only textbook definitions |
| Lead with **Problem → Solution** structure | Jump to the answer without context |
| Mention **Shared Responsibility** for security questions | Forget that misconfig is your fault |
| Say "IAM Role" for service access, not hardcoded credentials | Say "I stored the password in the code" |
| *"Would you like me to go deeper?"* | Stop without engaging |

---

## 🗓️ Course Progress Tracker

| Day | Topic | Status |
|---|---|---|
| Day 0 | Course Roadmap | ✅ Done |
| Day 1 | Cloud Basics & AWS Account | ✅ Done |
| Day 2 | IAM (Users, Groups, Roles, Policies) | ✅ Done |
| Day 3 | EC2 & Jenkins Deployment | ✅ Done |
| Day 4 | VPC & Networking | ✅ Done |
| Day 5 | Security Groups & NACL | ✅ Done |
| Day 6 | Route 53 (DNS) | ✅ Done |
| Day 7 | Production Capstone Project | ✅ Done |
| Day 8 | Scenario-Based Interview Q&A | ✅ Done |
| Day 9 | S3 Deep Dive | ✅ Done |
| Day 10 | AWS CLI Deep Dive | ✅ Done |
| Day 11+ | CloudFormation, CodePipeline, Lambda... | 📅 Upcoming |

---

## 📚 Resources

- 📺 [AWS Zero to Hero Course](https://www.youtube.com/watch?v=Ou9j73aWgyE)
- 🔗 [AWS Free Tier](https://aws.amazon.com/free/)
- 🔗 [AWS CLI Docs](https://docs.aws.amazon.com/cli/)
- 🔗 [AWS IAM Docs](https://docs.aws.amazon.com/iam/)
- 🔗 [AWS VPC Docs](https://docs.aws.amazon.com/vpc/)

---

> ⭐ **Star this repo** if it helped you prepare for your AWS interview!
> 🔔 Paste the next day's notes — I'll overwrite with only those days!
