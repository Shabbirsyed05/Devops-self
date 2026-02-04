# ☁️ AWS Zero to Hero | 30-Day DevOps Roadmap

## 🎯 Course Overview
This session (**Day-0**) acts as the **Syllabus & Roadmap** for a **30-day AWS Zero to Hero series**, designed to take you from **absolute beginner** to **job-ready DevOps Engineer**.

⚠️ **Important Note:**  
This course focuses on **real-world AWS services used by DevOps Engineers**, not on rote **certification-only preparation**.

---

## 🟡 1. Resources & Prerequisites

### ✅ Prerequisites
- **None required**
- Starts from the absolute basics (even **AWS account creation**)

### 📦 Resources
- **GitHub Repository:**  
  `AWS devops Zero to Hero`  
  (All notes, commands, and project files are stored here)

### 🧠 Learning Approach
- For complex topics (like **Containers & Kubernetes**), the instructor links **prerequisite videos** from earlier series
- Ensures you are **conceptually ready**, not just following commands blindly

---

## 🟡 2. Week 1: Core Foundation (Compute & Networking)

🔧 This week focuses on the **plumbing of the cloud** — servers, networking, and security.

### 📅 Daily Breakdown
- **Day 1:**  
  🌐 Public Cloud vs Private Cloud  
  🧾 Creating an AWS Account

- **Day 2:**  
  🔐 IAM (Identity & Access Management)  
  - Users  
  - Groups  
  - **Zero Privilege Principle**

- **Day 3:**  
  🖥️ EC2 (Elastic Compute Cloud)  
  - Launching virtual servers  
  - Deploying a simple web application

- **Day 4:**  
  🕸️ Networking (VPC)  
  - VPC  
  - Subnets  
  - Route Tables

- **Day 5:**  
  🛡️ Security  
  - Security Groups  
  - Network ACLs  
  - Restricting inbound & outbound access

- **Day 6:**  
  🌍 Route 53 (DNS)  
  - Domain management  
  - Routing policies

- **Day 7 (Project):**  
  🧩 **Capstone Project**  
  - EC2 + VPC + Security + Route 53  
  - Real-world architecture setup

---

## 🟡 3. Week 2: Storage, IaC & CI/CD

⚙️ This week shifts focus to **automation and DevOps pipelines**.

### 📅 Daily Breakdown
- **Day 8:**  
  🗂️ S3 (Simple Storage Service)  
  - File storage  
  - Static website hosting

- **Day 9:**  
  🧱 CloudFormation  
  - Infrastructure as Code (IaC)  
  - Creating infra using templates

- **Day 10:**  
  💻 AWS CLI  
  - Managing AWS from the terminal

- **Day 11 – Day 14:**  
  🔄 AWS Native CI/CD Tools
  - **CodeCommit** → Source Control  
  - **CodePipeline** → Orchestration  
  - **CodeBuild** → Build  
  - **CodeDeploy** → Deploy to EC2

---

## 🟡 4. Week 3: Monitoring & Serverless Architecture

📊 Focus on **observability** and **event-driven design** using Python.

### 📅 Daily Breakdown
- **Day 15:**  
  📈 CloudWatch  
  - Metrics  
  - Alarms  
  - Triggers

- **Day 16:**  
  ⚡ Lambda  
  - Serverless computing  
  - Python-based functions

- **Day 17 (Project):**  
  🚨 Event-Driven Architecture  
  - CloudWatch Events + Lambda  
  - ⭐ **Highly recommended for interviews**

- **Day 18:**  
  🕵️ CloudTrail  
  - Audit logs  
  - Compliance tracking

- **Day 19:**  
  🗃️ DynamoDB  
  - NoSQL fundamentals  
  - Used for **Terraform State Locking**

---

## 🟡 5. Week 4: Advanced Operations & Containers

🚀 Focus on **scalability, containers, and production readiness**.

### 📅 Daily Breakdown
- **Day 20 – Day 22:**  
  ☸️ EKS (Kubernetes)
  - Chosen over ECS due to **higher interview relevance**
  - Container registries → Kubernetes integration

- **Day 24:**  
  🔑 Systems Manager (SSM)  
  - Secrets  
  - Credentials  
  - API keys (secure management)

- **Day 25:**  
  📈 Auto Scaling  
  - Scale servers up/down automatically  
  - Example: High traffic during **festival or sale events**

- **Day 26 – Day 27:**  
  🗄️ RDS (Relational Databases)  
  ⚖️ Load Balancers  
  - ALB (Application Load Balancer)  
  - NLB (Network Load Balancer)

---

## 🟡 6. Final Phase: Strategy & Interviews

🎯 This phase prepares you for **real interview scenarios**.

- **Day 29:**  
  🚚 Cloud Migration Strategy  
  - How to answer:  
    **“How do you migrate an application from on-premise to AWS?”**

- **Day 30:**  
  🧑‍💼 Career Preparation  
  - Best practices  
  - Resume building  
  - Job search & resources

---

## ✅ Outcome of the Course
By the end of 30 days, you will have:
- Strong **AWS + DevOps fundamentals**
- Hands-on **projects**
- **Interview-ready explanations**
- A solid **GitHub portfolio**

🚀 **From Zero → Job-Ready DevOps Engineer**

====================================================================================

# ☁️ Day-1 | Introduction to AWS  
*AWS Zero to Hero – Foundations*

This session serves as the **kickoff** for the **30-Day AWS Zero to Hero series**, designed to teach AWS **from the perspective of a DevOps Engineer**.

---

## 🔵 1. The Evolution: From Data Centers to Cloud

To understand what the **Cloud** is, we must first understand how companies historically managed infrastructure.

### 🟥 The "Old Way" (Data Centers)
Companies used to buy **physical servers** (from vendors like IBM or HP) and store them in their **own data centers**.

- 🔧 **The Struggle:**  
  Required managing **cabling**, **temperature control**, and **physical security**.
- 💸 **The Waste:**  
  Servers often had huge capacity (e.g., **100GB RAM**) but applications used only a fraction (e.g., **1GB**).  
  The remaining **99% was wasted capital**.

### 🟢 The Solution (Virtualization)
- Virtualization allows **one physical server** to be split into **multiple Virtual Machines**.
- Instead of buying **15 physical servers**, you buy **one** and run **15 virtual servers** on top of it.

### ☁️ Cloud Definition
- Cloud means **accessing these virtual resources remotely**.
- You request a server and get access **without knowing its physical location** (US, India, Europe, etc.).

---

## 🔵 2. Private vs. Public Cloud

The session distinguishes between two main ownership models:

| Type | Definition | Management |
|----|----|----|
| 🏢 **Private Cloud** | Resources are used exclusively by one organization within their own boundaries. | You manage the hardware, virtualization, and maintenance. |
| 🌍 **Public Cloud** | A third-party provider (AWS, Azure, Google) owns the hardware and shares it across customers. | The provider manages the data center; **you only use the resources**. |

### ⭐ Why Public Cloud is Popular
- ⚙️ **Zero Maintenance:**  
  No need to manage power, cooling, or hardware patches.
- 📈 **Scalability:**  
  Start with basic storage and scale to advanced services like **Kubernetes** without buying hardware.

---

## 🔵 3. Why AWS? (Amazon Web Services)

With many cloud providers available (Azure, GCP, Oracle), why focus on AWS?

- 🥇 **First Mover Advantage:**  
  AWS pioneered cloud computing and holds the **largest market share**.
- 💼 **Job Market:**  
  Most companies look for **AWS skills**.  
  Cloud concepts are **transferable**—learning AWS makes it easier to adapt to Azure or GCP later.

---

## 🔵 4. Addressing “Cloud Repatriation”

A common rumor is that companies are moving back from Cloud to on-premise data centers.

- 📉 **The Reality:**  
  This happens only in **1–2% of cases**, usually due to extreme security or cost requirements.
- 📊 **The Majority:**  
  Most companies continue adopting **Public Cloud** because building private data centers is extremely expensive.

---

## 🔵 5. Practical Step: Creating an AWS Account

The session concludes with a walkthrough of creating a **free AWS account** for the rest of the course.

### 🧾 Account Details
- **Account Type:** Select **Personal** usage.
- 💳 **Credit/Debit Card Required:**
  - Used for **identity verification**.
  - AWS deducts a small amount (approx. **$1 USD or ₹2 INR**) temporarily.
  - The amount is **refunded**.

### 💰 Billing Logic
- AWS **does not auto-debit** usage charges.
- You receive a **monthly bill**.
- If unpaid, the account will be **suspended**.

---

## ✅ Day-1 Outcome
By the end of this session, you understand:
- Why Cloud replaced traditional data centers  
- The difference between Private and Public Cloud  
- Why AWS dominates the market  
- How to safely create an AWS account  

🚀 **You are now ready to move into IAM, EC2, and core AWS services**

====================================================================


# 🔐 Day-2 | AWS IAM Deep Dive  
*AWS Zero to Hero – Security Foundations*

This session builds on **Day 1 (Cloud Basics)** by addressing the **most critical first step in any AWS environment: Security**.  
The speaker uses the **Bank analogy** to explain why we cannot simply give everyone the **“keys to the castle”**.

---

## 🔴 1. The Core Problem: Root Access vs. IAM

### ⚠️ The Risk
- When you create an AWS account, you start with a **Root User**.
- The Root user can do **anything**:
  - Delete databases  
  - Launch expensive servers  
  - Destroy the entire account

### 🧨 The Scenario
- Imagine hiring **1,000 employees** and giving all of them the **Root password**.
- One person could accidentally (or maliciously) delete **all company data**.

### ✅ The Solution: AWS IAM
- **AWS IAM (Identity and Access Management)** allows you to:
  - Create **specific users**
  - Assign **limited permissions**
  - Protect the environment from human error

---

## 🔵 2. The Two Pillars: Authentication vs. Authorization

IAM works in **two distinct phases**, explained using a **bank analogy**:

| Concept | Definition | 🏦 Bank Analogy | ☁️ AWS Context |
|------|------|------|------|
| 🟢 **Authentication** | Verifying who you are | Security guard checks your ID at the door | Login using Username & Password |
| 🔵 **Authorization** | Verifying what you can do | Teller checks if you can access vault or help desk | Policies attached to your user |

### ⭐ Key Takeaway
- A user can be **Authenticated** but have **Zero Authorization**.
- In this state:
  - You can log in to the AWS Console  
  - ❌ You cannot view, create, or modify any resources

---

## 🧩 3. The 4 Components of IAM

IAM is built using **four main building blocks**:

---

### 👤 A. Users (The Person)
- **Purpose:** Created for **real people** (Developers, QAs, Admins).
- **Best Practice:**  
  - Never share users  
  - Create **one IAM user per employee**
- **Secure Setup:**
  - Enable **Auto-generated password**
  - Force **password reset on first login**

---

### 📜 B. Policies (The Permissions)
- **Purpose:** Define **what actions are allowed or denied**
- **Answers:** The **Authorization** question
- **Format:** JSON (but beginners can use **AWS Managed Policies**)
- **Example:**  
  - `AmazonS3FullAccess`
- **Critical Rule:**  
  - A user **without a policy is useless**

---

### 👥 C. Groups (The Efficiency Tool)
- **The Problem:**  
  - Managing permissions for 50 users individually is painful
- **The Solution:**  
  - Create a **Group** (e.g., `Dev-Team`)
  - Attach policies to the group
  - Add users to the group
- **Benefit:**  
  - Policy updates apply **automatically** to all group members

---

### 🏷️ D. Roles (The Service Access)
- **Purpose:** Temporary or machine-level access (not for humans)
- **Use Case:**  
  - An application needs to talk to a database
- **Best Practice:**  
  - ❌ Do NOT store usernames/passwords in code  
  - ✅ Assign a **Role** to the service
- **Note:**  
  - Roles are an **advanced topic**, covered later in CI/CD sessions

---

## 🛠️ 4. Practical Workflow (Live Demo)

The session demonstrates a complete **Zero to Hero IAM flow**:

1. 👤 **Create User**
   - Created `test-user-501`
   - Only **Authentication** exists at this stage

2. ❌ **Verify Failure**
   - Logged in as `test-user-501`
   - Tried to access **S3**
   - Result: **Access Denied**
   - User entered the “bank” but couldn’t see anything

3. 🔓 **Grant Authorization**
   - Admin attached `AmazonS3FullAccess` policy

4. ✅ **Verify Success**
   - Logged in again
   - User could list and create **S3 buckets**

5. 📦 **Scale with Groups**
   - Created `Development Group`
   - Attached:
     - `S3FullAccess`
     - `EC2FullAccess`
   - Added user to the group
   - Demonstrated **scalable permission management**

---

## 📝 5. Assignment for Revision

To solidify your understanding, complete this task:

1. 🔐 Log in using **Root account** (only for setup)
2. 👤 Create a new **IAM User**
3. 🚫 Log in as that user and confirm you **cannot see anything**
4. 🔓 Attach **AmazonS3FullAccess**
5. ✅ Verify you can now **create an S3 bucket**

---

## 🎯 Day-2 Outcome

By the end of this session, you understand:
- Why Root access is dangerous  
- The difference between Authentication & Authorization  
- How Users, Policies, Groups, and Roles work together  
- How to implement **least privilege security** in AWS  

🚀 **You are now ready to move into EC2 and real infrastructure**

============================================================================

# 🚀 Day-3 | EC2 Deep Dive | Deploy Jenkins on AWS

This document summarizes the **core concepts** and **hands-on deployment steps** covered in **Day-3 of the EC2 Deep Dive session**, focused on launching an EC2 instance and deploying **Jenkins on AWS**.

---

## 📌 What is EC2? (Acronym Breakdown)

**EC2 (Elastic Compute Cloud)** allows you to rent virtual servers from AWS.

### 🔍 Meaning
- **Elastic** → Scale up or scale down automatically based on demand  
- **Compute** → Provides CPU and RAM (processing power)  
- **Cloud** → Virtual server hosted in AWS data centers (not physical hardware)

### ✅ Why Use EC2?
1. **Zero Maintenance**  
   AWS manages physical hardware, upgrades, and data-center security.
2. **Cost Effective (Pay-as-you-go)**  
   You only pay for what you use.  
   💡 Servers can be stopped during nights or holidays to avoid billing.

---

## 🧾 EC2 Instance Types (The “Menu”)

AWS offers different instance families based on workload requirements:

| Instance Type | Best Used For |
|---------------|---------------|
| **General Purpose** | Balanced workloads (Used in this course – Free Tier) |
| **Compute Optimized** | High performance tasks (Gaming servers, ML workloads) |
| **Memory Optimized** | Fast processing of large datasets (Big Data Analytics) |
| **Storage Optimized** | High read/write storage operations |
| **Accelerated Computing** | GPU-based workloads (Graphics processing) |

---

## 🌍 Global Infrastructure: Regions vs Availability Zones

### 🗺️ Region
A **Region** is a geographical location:
- Examples: **North Virginia**, **Mumbai**, **Frankfurt**
- **Selection Criteria**:
  - 📉 Latency (closer to users)
  - 📜 Compliance (data residency laws)

### 🏢 Availability Zone (AZ)
- A **data center inside a Region**
- Examples: `us-east-1a`, `us-east-1b`

🔒 **Purpose: High Availability**  
If one AZ fails (e.g., power outage or fire), your application continues running in another AZ.

---

## 🧪 Practical Lab: Launching an EC2 Instance

### Step 1: OS Selection
- Choose **Ubuntu** (Free Tier eligible)

### Step 2: Instance Size
- Select **t2.micro**
  - 🧠 1 vCPU
  - 💾 1 GB RAM

⚠️ **Free Tier Warning**  
AWS provides **750 hours/month**.  
Running **two instances continuously** will exceed this limit and generate charges.

### Step 3: Key Pair (Login)
- Create a `.pem` file (example: `aws-login.pem`)
- This file acts as your **password**
- ❗ Do **not lose** this key

---

## 🔐 Connecting via SSH (Common Permission Error)

### SSH Command
```bash
ssh -i "key.pem" ubuntu@<Public-IP>
```
❌ Common Error
```bash
Permissions 0644 for 'key.pem' are too open
```
✅ Fix

Restrict permissions so only the owner can read the key:
```bash
chmod 600 key.pem
```
🛠️ Project: Deploying Jenkins on EC2

Once logged into the server:

1️⃣ Update System
```bash
sudo apt update
```
2️⃣ Install Java (Required for Jenkins)
```bash
sudo apt install openjdk-11-jdk
```
3️⃣ Install Jenkins

Download and install the Jenkins package

4️⃣ Verify Jenkins Service
```bash
sudo systemctl status jenkins
```
🚨 The Gotcha: Security Groups (Firewall)

After installation, accessing Jenkins via browser:
```bash
http://<EC2-Public-IP>:8080
```
❌ It will not work initially

❓ Why?

AWS uses Security Groups (virtual firewalls).
By default, only SSH (Port 22) is allowed.

✅ Solution: Allow Port 8080

Go to EC2 Console

Open Security Groups

Edit Inbound Rules

Add:

Type: Custom TCP

Port: 8080

Source: Anywhere (0.0.0.0/0)

🎉 Result:
The Jenkins login page will now load successfully in your browser.

✅ Outcome

By the end of this session, you will:

Understand EC2 fundamentals

Launch an EC2 instance

Connect securely using SSH

Deploy Jenkins on AWS

Configure Security Groups correctly

================================================================================

# 🌐 Day-4 | Best VPC Explanation | VPC Explained in 30 Minutes

This document summarizes **Day-4 of the AWS training** based on the session  
**“Best VPC explanation | VPC explained in 30 mins”** using a **Real World vs AWS analogy**.

VPC (Virtual Private Cloud) is often considered the **most challenging AWS networking topic**, and this session simplifies it using real-life comparisons.

---

## 🏘️ 1. Core Concept: The “Gated Community” Analogy

To explain VPC clearly, the instructor compares **AWS** to a **real estate developer**.

### 🌍 Real World vs AWS Mapping

| Real World Analogy | AWS Technical Concept | Purpose |
|-------------------|----------------------|---------|
| Village / City | **AWS Region** (Mumbai, Ohio, etc.) | Physical location where AWS infrastructure exists |
| Secure Property | **VPC (Virtual Private Cloud)** | Isolated and secure network for your project |
| House | **EC2 Instance** | The actual server running your application |
| Gate | **Internet Gateway (IGW)** | Entry point for internet traffic |
| Internal Roads | **Route Tables** | Direct traffic inside the VPC |
| Security Guard | **Security Group** | Allows or blocks traffic at instance level |

📌 **Key Idea:**  
A **VPC is your own private, gated network inside AWS**.

---

## 📐 2. Sizing the Network: CIDR & Subnets

### 🧮 VPC Size (CIDR Block)

When creating a VPC, you must define an **IP address range** using CIDR notation.

**Example:**
```bash
172.16.0.0/16
```

- Determines how many **IP addresses (houses/servers)** you can create inside the VPC.

---

### 🧩 Subnets (Sub-Projects)

Instead of placing everything in one network, the VPC is divided into **Subnets**.

#### 🔓 Public Subnet
- Accessible from the internet
- Used for:
  - Load Balancers
  - Frontend services

#### 🔒 Private Subnet
- No direct internet access
- Used for:
  - Databases
  - Backend applications

✅ **Best Practice:**  
Expose only what is required. Keep critical resources private.

---

## 🔄 3. Traffic Flow: How a User Reaches Your Application

This explains how a request travels from a **user’s laptop** to your **EC2 instance**.

### 🚶 Request Journey

1. 🌐 **Internet**  
   User sends a request (e.g., website access)

2. 🚪 **Internet Gateway (IGW)**  
   Request enters the VPC

3. 🏗️ **Public Subnet & Load Balancer**  
   Load Balancer receives the request

4. 🗺️ **Route Table**  
   Determines where traffic should go

5. 🛂 **Security Group**  
   Checks whether traffic is allowed (e.g., Port 80)

6. 🖥️ **EC2 Instance**  
   Request reaches the application

---

## 🔁 4. Outbound Internet Access: NAT Gateway

### ❌ The Problem

Your application is in a **Private Subnet** for security, but it needs to:
- Download updates
- Access Google or external APIs

Private instances **cannot access the internet directly**.

---

### ✅ The Solution: NAT Gateway (Network Address Translation)

#### 📍 Placement
- NAT Gateway is placed in a **Public Subnet**

#### ⚙️ How It Works
1. Private EC2 sends request to NAT Gateway  
2. NAT Gateway hides the private IP  
3. NAT Gateway uses its **public IP** to access the internet  
4. Response is forwarded back to the private EC2  

🔐 **Result:**  
The private server gets internet access **without exposing its IP**.

---

## 📋 5. Summary Checklist (Quick Revision)

- **VPC** → Isolated private cloud network  
- **Subnet** → Segment of a VPC (Public or Private)  
- **Internet Gateway** → Allows internet access  
- **Route Table** → Controls traffic routing  
- **NACL** → Subnet-level security (stateless)  
- **Security Group** → Instance-level security (stateful)  
- **VPC Flow Logs** → Capture and analyze network traffic  

---

## ✅ Final Outcome

After this session, you should be able to:
- Explain VPC using real-world examples
- Design Public and Private Subnets
- Understand inbound and outbound traffic flow
- Confidently explain NAT Gateway in interviews

---

⭐ **VPC becomes simple when you see it as a gated community** 🏘️☁️

==============================================================================

# 🔐 Day-5 | AWS Security Group and NACL

This document summarizes **Day-5 of the AWS VPC module**, based on the session  
**“AWS Security Group and NACL”**.

📌 **Context**  
- **Day 4** → Focused on *Connectivity* (how traffic flows)  
- **Day 5** → Focused on *Security* (how traffic is controlled and protected)

This session explains **how AWS networking security works in layers**.

---

## 🤝 1. Core Principle: Shared Responsibility Model

Before using any security tool, AWS establishes one golden rule:

### 🔐 Security is a *Shared Responsibility*

### 🧱 AWS Responsibility
- Physical data centers
- Hardware
- Underlying infrastructure

### 👨‍💻 Your Responsibility (DevOps / Cloud Engineer)
- Securing applications
- Opening/closing ports
- Protecting data
- Configuring Security Groups and NACLs correctly

⚠️ **Important Implication**  
AWS provides the tools, but **misconfiguration is your responsibility**.  
If you allow **All Traffic**, any breach is **not AWS’s fault**.

---

## 🛡️ 2. Layer 1: Security Groups (Instance Firewall)

A **Security Group (SG)** is a virtual firewall that works at the **EC2 instance level**.

### 🔍 Default Behavior

#### 📥 Inbound Traffic
- **Deny All** by default  
- No one can access the instance until you explicitly open a port

#### 📤 Outbound Traffic
- **Allow All** by default  
- Instance can access the internet freely  

🚫 **Exception:**  
- **Port 25** is blocked by default to prevent spam email abuse

---

### ⚙️ Rule Logic (Very Important)

- Security Groups support **ONLY Allow rules**
- You **cannot** create a Deny rule
- You **cannot** say:
  > “Allow everyone except IP X”

📌 Security Groups are **stateful**:
- If inbound traffic is allowed, return traffic is automatically allowed

---

## 🚧 3. Layer 2: Network ACL (Subnet Firewall)

A **Network ACL (NACL)** works at the **Subnet level** and acts as the **outer security layer**.

### 🔥 Power of NACLs

✅ Supports **Allow AND Deny rules**  
✅ Protects **all instances inside a subnet** at once

### 🎯 Common Use Case
- Blocking:
  - A hacker’s IP
  - A malicious IP range
  - A specific country’s traffic

Instead of updating **50 Security Groups**, you block once at the **NACL level**.

---

### 🔢 Rule Priority (Critical Concept)

- NACL rules are **numbered** (e.g., 100, 200, `*`)
- AWS processes rules in **ascending order**

#### 🧪 Example
- Rule 100 → Deny Port 8000  
- Rule 200 → Allow Port 8000  

📛 **Result:**  
Traffic is **DENIED**, because AWS stops at Rule 100.

---

## ⚖️ 4. Comparison: Security Group vs NACL

| Feature | Security Group (SG) | Network ACL (NACL) |
|------|------------------|-------------------|
| Level | Instance (EC2) | Subnet |
| Rules | Allow only | Allow and Deny |
| Scope | Specific instances | All resources in subnet |
| Defense Layer | Second / Inner layer | First / Outer layer |
| Rule Evaluation | All rules evaluated | First match wins |
| Nature | Stateful | Stateless |

📌 **Interview Tip:**  
NACL is always evaluated **before** the Security Group.

---

## 🧪 5. Practical Lab: “Allow vs Deny” Conflict

The instructor demonstrated how **Security Groups and NACLs interact**.

---

### 🧩 Setup
- Python HTTP server running on **Port 8000**
- Hosted inside a **Custom VPC**

---

### ✅ Scenario A: Traffic Allowed
- Security Group → Allow Port 8000
- NACL → Default (Allow All)

🎉 **Result:**  
Traffic reached the application successfully

---

### ❌ Scenario B: NACL Blocks Traffic
- Security Group → Allow Port 8000
- NACL → Rule #100 → Deny Port 8000

🚫 **Result:**  
Traffic blocked  
Even though the **inner door (SG)** was open, the **outer gate (NACL)** blocked it

---

### 🔢 Scenario C: Rule Priority Check
- NACL Rule 100 → Deny Port 8000
- NACL Rule 200 → Allow All Traffic

🛑 **Result:**  
Traffic still blocked  
AWS reads Rule 100 first and **stops processing**

---

## ✅ Final Takeaways

- **Security Groups** protect individual instances
- **NACLs** protect entire subnets
- NACLs are evaluated **first**
- A single **Deny in NACL** overrides all Security Group rules
- Security in AWS is **your responsibility**

---

⭐ **Think of it like security gates:**  
🏰 *NACL = Outer Gate* → 🚪 *Security Group = Inner Door*

==========================================================================

# 🌍 Day-6 | Route 53 Explained in 15 Minutes

This document summarizes **Day-6 of the AWS networking series**, based on the session  
**“Route 53 explained in 15 mins”**.

📌 This session focuses on **DNS (Domain Name System)**, which is the **final missing piece** before building a **production-grade AWS network**.

---

## 🧠 1. What is Route 53?

### 📘 Definition
**Route 53** is **DNS as a Service** provided by AWS.

### 🔍 The Analogy
- **EC2** → Compute as a Service  
- **EKS** → Kubernetes as a Service  
- **Route 53** → DNS as a Service  

📌 **Role of Route 53:**  
It manages the mapping between **human-friendly domain names** and **AWS resources (IP addresses / Load Balancers)**.

---

## ❓ 2. Why Do We Need Route 53? (The Problem with IPs)

When you deploy an application, AWS assigns it a **public IP address**  
Example:
```bash
3.6.10.171
```
### ⚠️ Problems with Using IP Addresses

#### 🧠 Issue 1: Memorability
- Humans cannot remember numbers like `3.6.10.171`
- We prefer names like:
  - amazon.com
  - flipkart.com

#### 🔄 Issue 2: Dynamic Nature
- Public IPs can **change**
- Restarting a server or Load Balancer may assign a **new IP**
- You cannot ask users to update bookmarks every time

---

### ✅ The Solution: DNS
DNS automatically maps a **static domain name** to a **dynamic IP address**.

📌 Route 53 handles this mapping for you.

---

## 🧩 3. Core Components of Route 53

Route 53 provides **three main functionalities**.

---

### 🅰️ A. Domain Registration

#### 🛒 Buying a Domain Name
To use a domain like:
```bash
abhishek.com
```
You must first **buy the name**.

#### 🌐 Options
- **AWS Route 53** → Buy directly inside AWS
- **Third-Party Providers** → GoDaddy, Namecheap, etc.

📌 You can buy the domain anywhere and still use **Route 53** for DNS management.

---

### 🅱️ B. Hosted Zones (The DNS Database)

#### 📘 Definition
A **Hosted Zone** is where your **DNS records** are stored.

It acts like a **database of rules**.

#### ⚙️ How It Works
- You create a Hosted Zone for your domain
- Inside it, you add records like:
  > “When someone types `myapp.com`, route them to Load Balancer IP `1.2.3.4`”

📌 Whether the domain is purchased on AWS or GoDaddy, **Hosted Zones live in Route 53**.

---

### 🅾️ C. Health Checks (Reliability & High Availability)

#### ❤️ What Health Checks Do
Route 53 doesn’t blindly send traffic.

It **monitors server health** before routing users.

#### 🔄 Workflow
1. Route 53 sends periodic health checks (e.g., every 1 minute)
2. If **Server A** is healthy → traffic is sent
3. If **Server A** fails → traffic is stopped
4. Users are redirected to **Server B**

✅ This enables **automatic failover**.

---

## 🔄 4. How Traffic Flows with Route 53

With Route 53 added, the **user journey** looks like this:

1. 👤 **User**  
   Types `amazon.com` in the browser

2. 🌍 **Route 53**  
   - Looks into the Hosted Zone  
   - Finds the IP / Load Balancer address

3. ⚖️ **Load Balancer**  
   - Receives traffic  
   - Distributes it to EC2 instances in public/private subnets

---

## 🚀 5. Next Steps (Day-7 Preview)

At this point, we have learned **all the building blocks**:

- **VPC** → The Network  
- **EC2** → The Server  
- **Security Groups** → The Firewall  
- **Route 53** → The Address  

### 🔜 Next Session (Day-7)
All components will be **combined** to build a:

✅ **Full-Fledged Production Architecture**  
- Public & Private Subnets  
- Load Balancer  
- DNS  
- Security  

📌 This setup is **very common in AWS interviews**.

---

⭐ **Think of Route 53 as the phonebook of the internet** 📖🌐  

============================================================

# 🏗️ Day-7 | AWS Project Used In Production (Capstone Project)

This document summarizes **Day-7 of the AWS training series**, based on the session  
**“AWS Project Used In Production”**.

📌 This is a **capstone project** that combines **all previously learned concepts**:
- VPC
- EC2
- Security Groups
- Subnets
- NAT Gateway
- Load Balancer
- Auto Scaling

🎯 **Goal:**  
Build a **secure, highly available, and scalable production-grade architecture** used in real-world enterprises.

---

## 🧱 1. The Architecture – “Secure Design”

The instructor transitions from basic demos to a **real-world enterprise design**.

### ✅ High Availability
- Architecture spans **2 Availability Zones (AZs)**
  - Example: `us-east-1a` and `us-east-1b`
- If one AZ goes down, the application continues running in the other

---

### 🗂️ Subnet Division

#### 🔓 Public Subnet
Contains:
- Load Balancer
- NAT Gateway  

📌 These components **must access the internet**.

#### 🔒 Private Subnet
Contains:
- Application Servers (EC2)

📌 These servers are **hidden from the internet** for security.

---

## 🧩 2. New Concepts Introduced

To support this architecture, **three new components** are introduced:

| Component | Purpose | Real-World Analogy |
|--------|--------|------------------|
| **Bastion Host (Jump Server)** | Secure SSH access to private instances | Reception desk of a secure building |
| **Auto Scaling Group (ASG)** | Automatically adds/removes servers | Shop manager hiring during rush hours |
| **NAT Gateway** | Allows private servers to access the internet | One-way mirror (outbound only) |

---

## 🛠️ 3. Step-by-Step Implementation

The deployment is completed in **four phases**.

---

### 🔹 Phase 1: Network Setup (VPC)

#### ⚡ Shortcut Used
- AWS **“VPC and more” wizard**
- Faster and less error-prone than manual creation

#### ⚙️ Configuration
- 1 VPC  
- 2 Availability Zones  
- 2 Public Subnets  
- 2 Private Subnets  
- 1 NAT Gateway  

📌 **Critical:**  
NAT Gateway is required for private instances to download packages.

---

### 🔹 Phase 2: Compute Setup (Auto Scaling Group)

#### 🚀 Launch Template
Defines **what to launch**:
- Ubuntu OS
- `t2.micro`
- Security Group:
  - Allow SSH
  - Allow Port 8000

#### 📈 Auto Scaling Group (ASG)
Defines **where to launch**:
- Explicitly selects **Private Subnets**
- Ensures application servers are **not internet-facing**

#### 🔢 Capacity
- Desired instances: **2**
- One instance per Availability Zone

---

### 🔹 Phase 3: Access Problem (Bastion Host)

Since application servers are **private**, direct SSH is not possible.

#### 🧑‍✈️ Step 1: Launch Bastion Host
- Ubuntu EC2
- Launched in **Public Subnet**

---

#### 🔑 Step 2: Transfer Key to Bastion
Copy your `.pem` file from laptop to Bastion Host:

```bash
scp -i key.pem key.pem ubuntu@<Bastion-IP>:/home/ubuntu
```
# 🏗️ Day-7 | AWS Project Used In Production (Capstone Project)

This document summarizes **Day-7 of the AWS training series**, based on the session  
**“AWS Project Used In Production”**.

📌 This is the **capstone project** where all previously learned AWS concepts are combined into a **real-world, production-grade architecture**.

🎯 **Goal:**  
Deploy an application that is **secure**, **highly available**, and **scalable**, following industry best practices.

---

## 🧱 1. Architecture Overview – Secure Production Design

The instructor moves away from basic demos and introduces an **enterprise-level architecture**.

### ✅ High Availability
- Architecture spans **two Availability Zones (AZs)**  
  - Example: `us-east-1a` and `us-east-1b`
- If one AZ fails, the application continues running in the other

---

### 🗂️ Subnet Design

#### 🔓 Public Subnets
Contain:
- Application Load Balancer (ALB)
- NAT Gateway

📌 These components must communicate with the internet.

#### 🔒 Private Subnets
Contain:
- Application Servers (EC2 instances)

📌 These servers are **not exposed to the internet**, improving security.

---

## 🧩 2. New Concepts Introduced

To support this production setup, three new components are introduced:

| Component | Purpose | Real-World Analogy |
|--------|--------|------------------|
| **Bastion Host (Jump Server)** | Secure SSH access to private instances | Reception desk of a secure building |
| **Auto Scaling Group (ASG)** | Automatically scales EC2 instances | Manager adding staff during peak hours |
| **NAT Gateway** | Allows private servers to access the internet securely | One-way mirror (outbound only) |

---

## 🛠️ 3. Step-by-Step Implementation

The deployment is completed in **four structured phases**.

---

### 🔹 Phase 1: Network Setup (VPC)

#### ⚡ Shortcut Used
- AWS **“VPC and more” wizard**
- Faster and safer than creating everything manually

#### ⚙️ Configuration
- 1 VPC  
- 2 Availability Zones  
- 2 Public Subnets  
- 2 Private Subnets  
- 1 NAT Gateway  

📌 **Important:**  
NAT Gateway is required for private instances to download updates and packages.

---

### 🔹 Phase 2: Compute Setup (Auto Scaling)

#### 🚀 Launch Template
Defines **what to launch**:
- Ubuntu OS
- `t2.micro` instance
- Security Group allowing:
  - SSH
  - Port 8000

#### 📈 Auto Scaling Group (ASG)
Defines **where to launch**:
- Private Subnets are explicitly selected
- Ensures application servers are not internet-facing

#### 🔢 Capacity Settings
- Desired capacity: **2 EC2 instances**
- One instance in each Availability Zone

---

### 🔹 Phase 3: Access Challenge (Bastion Host)

Since application servers are in **Private Subnets**, direct SSH access is not possible.

#### 🧑‍✈️ Step 1: Launch Bastion Host
- Ubuntu EC2 instance
- Placed in a **Public Subnet**

---

#### 🔑 Step 2: Copy Key to Bastion Host
Use `scp` to copy the `.pem` key from your laptop to the Bastion Host:

```bash
scp -i key.pem key.pem ubuntu@<Bastion-IP>:/home/ubuntu
```
🔁 Step 3: Jump to Private Instance

SSH into the Bastion Host

From the Bastion Host, SSH into the private IP of the application server

📌 This is why it is called a Jump Server.

🔹 Phase 4: Application Deployment & Load Balancing

🧩 Application Setup

Simple Python HTTP server

Runs on Port 8000

Deployed on private EC2 instances

🎯 Target Group

Created for Port 8000

Targets private EC2 instances

⚖️ Application Load Balancer (ALB)

Internet-facing ALB

🧪 4. Troubleshooting & Verification

❌ Initial Issue

Load Balancer returned:

502 Bad Gateway

or request timeout

🔧 Root Cause & Fix

ALB Security Group was missing:

Inbound HTTP (Port 80)

Source: 0.0.0.0/0

✅ Final Result

Accessing the ALB DNS name successfully displays:
Deployed in Public Subnets

Listener:

Port 80

Forwards traffic to:

Target Group → EC2 instances on Port 8000
```bash
My First AWS Project
```
📝 5. Assignment (Hands-On Practice)
📌 Current State

Python application is running on only one private instance

🎯 Task

Log in to the second private instance

Deploy a different HTML page
Example:
```bash
This is Server B
```
🔄 Validation

Refresh the Load Balancer DNS URL

Observe responses switching between:

Server A

Server B

✅ This confirms Load Balancing and High Availability.

🏁 Final Outcome

By completing this project, you have:

Built a production-grade AWS architecture

Implemented high availability using multiple AZs

Secured applications using private subnets

Used Auto Scaling and Load Balancers

Applied real-world enterprise best practices

⭐ This architecture is extremely important for AWS interviews and real production environments ☁️🚀

=====================================================================

# 🎯 Day-8 | AWS Scenario Based Interview Questions

This document summarizes **Day-8 of the AWS training**, based on the session  
**“AWS Scenario Based Interview Questions”**.

📌 Unlike previous days, this session is **purely interview-focused**.  
It consolidates knowledge from **Day 1–Day 7**:
- EC2
- VPC
- IAM
- Security Groups
- NACL
- Route 53
- Load Balancing
- Auto Scaling

🎯 **Goal:**  
Test whether you understand **how to use AWS services in real-world scenarios**, not just their definitions.

---

## 🏗️ 1. Architecture Scenarios (High Availability & Scaling)

### ❓ Scenario 1: Design a 2-Tier Highly Available & Scalable Application

#### 🧩 Problem
- Application has:
  - Frontend (Tier 1)
  - Backend (Tier 2)
- Must:
  - Survive data center failure
  - Handle sudden traffic spikes

#### ✅ Solution
1. **High Availability**
   - Deploy across **multiple Availability Zones**
     - Example: `us-east-1a`, `us-east-1b`

2. **Scalability**
   - Use **Auto Scaling Groups (ASG)**
   - Automatically adds/removes instances based on load

3. **Secure Placement**
   - Load Balancer → **Public Subnets**
   - Application Servers → **Private Subnets**

---

### ❓ Scenario 2: How Do You Access a Private Instance?

#### 🧩 Problem
- Application server is in a **Private Subnet**
- Direct SSH from internet is not allowed

#### ✅ Solution
- Use a **Bastion Host (Jump Server)**
  - Bastion Host → Public Subnet
  - Admin SSHs into Bastion
  - From Bastion → SSH into Private Instance

📌 This maintains **security while allowing admin access**.

---

## 🌐 2. Networking & Connectivity Scenarios

### ❓ Scenario 3: How Do Private Instances Download Updates Securely?

#### 🧩 Problem
- Private instance needs internet access
- Must not be exposed to the internet

#### ✅ Solution: **NAT Gateway**
- Place NAT Gateway in **Public Subnet**
- Performs **Network Address Translation**
- Masks private IP using its public IP

🔐 Private server stays hidden while accessing the internet.

---

### ❓ Scenario 4: Restrict Internet Access for Only One Subnet

#### 🧩 Problem
- Subnet A → Needs internet
- Subnet B → Must be completely isolated

#### ✅ Solution: **Route Tables**
- Modify Route Table of Subnet B
- Remove:
```bash
0.0.0.0/0 → Internet Gateway
```

🚫 Without this route, Subnet B cannot access the internet.

---

### ❓ Scenario 5: Securely Access S3 from Within a VPC

#### 🧩 Problem
- EC2 instance must access S3
- Traffic should not go over public internet

#### ✅ Solution: **VPC Endpoint**
- Creates private connection between:
  - VPC ↔ S3 / DynamoDB
- Traffic stays inside AWS network

📌 Improves **security and performance**.

---

## 🔐 3. Security Scenarios (Firewalls & Access Control)

### ❓ Scenario 6: Difference Between Security Group and NACL

| Feature | Security Group (SG) | Network ACL (NACL) |
|------|------------------|------------------|
| Scope | Instance Level | Subnet Level |
| State | Stateful | Stateless |
| Traffic Control | Allow only | Allow and Deny |
| Use Case | Protect individual servers | Protect entire subnet |

📌 **Interview Tip:**  
NACL is evaluated **before** Security Group.

---

### ❓ Scenario 7: How Do You Implement Strict Network Defense?

#### ✅ Solution
- Use **Network ACLs**
- Block:
  - Malicious IPs
  - Country-level IP ranges
- Stops traffic at **subnet boundary** before it reaches EC2

🛡️ Stronger, centralized security control.

---

## 👤 4. IAM Scenarios (Users vs Roles)

### ❓ Scenario 8: Explain IAM Users, Groups, Roles, and Policies

#### 👨‍💻 IAM Users
- Created for **people**
- Used for **authentication**

---

#### 📜 IAM Policies
- Define **permissions**
- Without a policy, a user **can do nothing**

---

#### 👥 IAM Groups
- Collection of users
- Policies attached **once**
- Easier management for large teams

Example:
- Developers Group
- Admin Group

---

#### 🤖 IAM Roles
- Used by **AWS services**, not people
- No username or password
- Temporary credentials

📌 **Real Example**
- EC2 instance needs to write to a Database
- Do **NOT** store credentials in code
- Attach an **IAM Role** to EC2 with DB access permissions

---

## 🏁 Final Takeaways

- Interviews test **architecture thinking**, not memorization
- Always think in terms of:
  - High Availability
  - Security
  - Scalability
  - Least Privilege
- Understand **when and why** to use each AWS service

⭐ **If you can explain these scenarios clearly, you are interview-ready** 🚀☁️

===============================================================================
# 📦 Day-9 | AWS S3 Buckets Deep Dive

This document summarizes the session **“Day-9 | AWS S3 Buckets Deep Dive.”**  
The session introduces the **Storage module** of AWS with a deep focus on **Amazon S3 – Simple Storage Service**, one of the most widely used AWS services.

---

## 🟦 1. What is S3? (Simple Storage Service)

### 🔸 The Problem
Organizations generate huge amounts of data such as:

- Database backups (sometimes in TBs)  
- Application logs  
- Media files (images & videos)

Storing all this on physical hard drives becomes **expensive, unscalable, and hard to manage.**

---

### ✅ The Solution – Amazon S3

S3 provides:

- “Infinite” cloud storage  
- Storage for **Objects** like:
  - Files  
  - Videos  
  - Logs  
  - Database dumps

---

### 🌐 Global vs Regional Concept

#### 🌍 Global Namespace
- Bucket names must be **unique across the entire world**  
- Example: You cannot create a bucket named `test` if someone already owns it.

#### 📍 Regional Storage
- The bucket name is global  
- But the actual data lives in a **specific AWS Region** (e.g., us-east-1)  
- Ensures:
  - Low latency  
  - Compliance & data residency

---

## 🟪 2. The “11 Nines” Durability

### ⭐ AWS Guarantee
> **99.999999999% durability**

### 🧠 Meaning
- If you store **1 Billion objects**  
- You may lose **only 1 object in 100 years**

### 🛡 How AWS Achieves This
- Automatic replication  
- Across multiple Availability Zones  
- Across multiple data centers  

Even if one data center is destroyed, your data remains safe.

---

## 🟩 3. Key Features of S3

### 🚀 Scalability
- No limit on total storage  
- Each object can be up to **5TB**

---

### 🕒 Versioning

Just like Git:

- If you overwrite  
  ```
  index.html
  ```
- S3 keeps the old version

You can:

- Restore previous versions  
- Recover from accidental deletes  
- Rollback buggy uploads

---

### 💰 Storage Classes

Different tiers for different needs:

| Storage Class | Purpose | Cost | Retrieval |
|----------------|---------|------|-----------|
| S3 Standard | Frequently accessed | Expensive | Immediate |
| S3 Glacier | Old backups | Cheap | Minutes–Hours |
| S3 Deep Archive | Rarely used | Very Cheap ($1–2) | 12–48 Hours |

---

## 🟥 4. Demo 1 – Bucket Policies (Security Override)

### 🧩 Scenario

- IAM user has → **S3FullAccess (Admin)**
- But one bucket has **highly sensitive data**
- Owner wants to block even Admin users

### ⚔ Conflict

- IAM Policy → **ALLOW**  
- Bucket Policy → wants **DENY**

### ✅ Solution

Apply a Bucket Policy (JSON):

> “Deny Everything to Everyone unless the user is Me”

### 🎯 Result

- Even Full Admin users were **blocked**
- Bucket Policy overrode IAM permission

---

## 🟧 5. Demo 2 – Static Website Hosting

S3 can host websites **without EC2 / Apache / Nginx**

### 🛠 Steps

#### Step 1 – Upload
```
index.html
```

#### Step 2 – Enable Hosting
- Properties → Static Website Hosting → Enable

#### Step 3 – Permissions (CRITICAL)

- Uncheck  
  ```
  Block All Public Access
  ```

- Add Bucket Policy:
  ```
  Allow → GetObject → Principal *
  ```

### 🌐 Result

- AWS generates a URL  
- Opening it loads your HTML page instantly  
- No server required

---

# 🟦 Key Takeaways

- ✅ S3 provides infinite object storage  
- 🌍 Bucket names are globally unique  
- 🛡 11 nines durability  
- 🔁 Supports versioning  
- 💸 Multiple storage classes  
- 🌐 Can host static websites  
- 🔐 Bucket Policies can override IAM

---

### 🚀 S3 = Backbone of AWS Storage

Used for:

- Backups  
- Logs  
- Media hosting  
- Data lakes  
- Static websites  


=======================================================

# 🖥️ Day-10 | AWS CLI Deep Dive

This document summarizes the session **“Day-10 | AWS CLI Deep Dive.”**  
The session marks a major shift from **Manual Console Operations → Automation using AWS CLI.**

---

## 🔴 1. The Problem with the AWS Console (UI)

### ❌ Limitations of UI
- Good for learning  
- Not suitable for automation  
- Slow for bulk operations

### 🧪 Scenario
> Manager asks you to create **10 VPCs** or **15 EC2 instances**

Doing this manually in the console is:

- Time consuming  
- Error prone  
- Not repeatable

### ✅ Solution
Use **APIs** to automate AWS tasks → Enter **AWS CLI**

---

## 🟦 2. What is AWS CLI?

### 📘 Definition
AWS CLI is:

- An **open-source tool**  
- Built using **Python**  
- Acts as a bridge between:
  - Your terminal  
  - AWS APIs

---

### ⚙️ How it Works

1. You type:
   ```
   aws s3 ls
   ```
2. CLI converts it into an **API Call**
3. Sends request to AWS
4. Receives response (mostly JSON)
5. Displays result in terminal

---

### 🌟 Benefit

You DON’T need to:

- Write Python scripts  
- Handle HTTP requests  
- Manage authentication manually  

CLI handles everything!

---

## 🟩 3. Installation & Setup

### 📌 Prerequisite
- Python must be installed

---

### 💻 Operating System Tips

| OS | Recommendation |
|----|----------------|
| Mac/Linux | Use native terminal |
| Windows | Use Git Bash or Oracle VirtualBox (Linux) |
| CMD | Not DevOps friendly |

---

### ✔ Verify Installation

Run:

```
aws --version
```

If version appears → Installation successful ✅

---

## 🟪 4. Authentication – `aws configure`

### 🔐 Important Concept
CLI does **NOT** use:

- AWS console username  
- Password login

It uses:

- Access Key  
- Secret Key

---

### 🚨 Security Warning

> ❗ Never share your Secret Access Key  
> ❗ If lost → it cannot be recovered, only replaced

---

### 🛠 Configuration Command

Run:

```
aws configure
```

Enter:

- Access Key ID  
- Secret Access Key  
- Default Region → `us-east-1`  
- Output Format → `json`

---

## 🟧 5. Practical Demonstrations

### 🅐 Simple Task – List S3 Buckets

#### Command

```
aws s3 ls
```

#### Result
- Instantly lists all buckets  
- Much faster than UI navigation

---

### 🅑 Complex Task – Create EC2 Instance

#### Command

```
aws ec2 run-instances
```

#### Required Parameters

- Image ID  
- Instance Type  
- Subnet ID  
- Security Group

---

### 🐞 Troubleshooting

If any parameter missing:

- CLI throws clear error  
- Tells exactly what is required

---

## 🟨 6. How to Learn CLI Commands

### 🧠 You Don’t Need to Memorize!

#### Strategy

Search:

> “AWS CLI Reference [Service Name]”

Examples:

- AWS CLI Reference S3  
- AWS CLI Reference EC2

---

### 📘 Documentation Pattern

Look for:

- Synopsis  
- Available commands like:
  - `ls`  
  - `mb` (make bucket)  
  - `cp` (copy)

---

## 🟥 7. CLI vs Infrastructure as Code

| Feature | AWS CLI | Terraform / CloudFormation |
|-------|---------|-----------------------------|
| Purpose | Quick actions | Full architecture |
| Best For | One-off tasks | Production stacks |
| Complexity | Simple | Complex |
| Reusability | Low | High |

---

### 🧩 When to Use What?

#### Use CLI for:
- Listing resources  
- Quick testing  
- Simple automation

#### Use IaC for:
- VPC + Subnets  
- Load Balancers  
- Multi-service architecture

> IaC will be covered in **Day-11**

---

# ✅ Key Takeaways

- AWS CLI = Automation Tool  
- Converts commands → API calls  
- Requires Access Keys  
- Faster than console  
- Best for quick operations  
- Not ideal for large infra

---

## 🚀 Next Step

👉 Day-11 → Infrastructure as Code (Terraform / CloudFormation)

================================================================================






