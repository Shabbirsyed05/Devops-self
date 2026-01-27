# 🚀 Day 1: DevOps Fundamentals

**Course Status:** Day 1 of 40  
**Topic:** Introduction, Culture, and Interview Prep  
**Goal:** Understand the "What" and "Why" of DevOps before learning tools.

---

## 📖 1. What is DevOps?

DevOps is not just a tool. It is a culture or practice adopted by an organization to improve its ability to deliver applications.

### 🎯 The Core Definition
For interviews, the ideal definition is:

> "DevOps is a process of improving application delivery by ensuring proper automation, maintaining quality, and implementing continuous monitoring and testing".

### ⚡ The Objective
The main goal is to reduce the delivery time of a new version or bug fix from weeks/days down to hours/minutes.

---

## 🏛️ 2. The Four Pillars of DevOps

To achieve speed without breaking the application, DevOps relies on these four essential components:

| Pillar       | Icon | Description |
|--------------|------|-------------|
| Automation   | 🤖   | Replacing manual labor (like scripts or machinery) to speed up production. |
| Quality      | ✅   | Ensuring that despite high speed, the end-user experience remains perfect. |
| Monitoring   | 📊   | (Observability) Systems that alert you immediately if automation or quality fails. |
| Testing      | 🧪   | Continuous validation to prove automation is correct and quality is maintained. |

---

## ⏳ 3. Why DevOps? (Evolution)

To understand the value, we compare the modern approach with the "Traditional" workflow from ~10 years ago.

| Feature | 🐢 The Old Way (Pre-DevOps) | 🐇 The DevOps Way |
|--------|-----------------------------|------------------|
| Team Structure | Siloed: Developers, System Admins, and Build/Release Engineers worked separately. | Unified: A single culture where teams collaborate efficiently. |
| Process | Manual Handoffs: Code was physically moved to servers and testers manually. | Automated Pipeline: Manual effort is removed to prevent bottlenecks. |
| Deployment Time | Slow: Could take 10 days or more to deploy a fix. | Rapid: Deployments happen in days, hours, or minutes. |

---

## 💼 4. Interview Preparation

The source emphasizes that the Introduction is where you project your value.  
If you fail here, the interviewer may lose interest in the first 5 minutes.

### ❌ What to AVOID
- Do not claim 10 years of DevOps experience. The field has only been popular for about 7–8 years. Claiming 10+ years is a red flag for lying.
- Do not rush. Take your time. You don't need to finish in 60 seconds.

### ✅ The Ideal Introduction Script

Structure your introduction using this 3-step flow:

1. **Current Role**  
   "I have [X] years of experience as a DevOps Engineer...".

2. **Past Background (The Bridge)**  
   "Before this, I was a [System Admin / Developer / QA]. This gave me strong fundamentals in [Servers / Coding / Testing]...".

   - Why? This helps the interviewer correlate your past skills with your current DevOps capabilities.

3. **Day-to-Day Activities (The 4 Pillars)**  
   "In my current role, I improve delivery by taking care of Automation, ensuring Code Quality, setting up Continuous Monitoring, and managing Continuous Testing".

---

## 🛠️ Tools Mentioned (Optional)

While Day 1 focuses on culture, seasoned candidates may mention these tools during introductions:

- CI/CD: GitHub Actions
- Orchestration: Kubernetes
- Configuration: Ansible
- Infrastructure: Terraform

---

## ⏭️ Next Steps

========================================================================================================================================================================



# 📘 Software Development Life Cycle (SDLC) – Summary

This document provides a concise overview of the **Software Development Life Cycle (SDLC)**, organized for easy revision.

---

## ❓ What is SDLC?

The **Software Development Life Cycle (SDLC)** is a standard process or set of steps followed by the software industry to design, develop, and test high-quality products.  
Whether working at a startup or a large company like Amazon, everyone (developers, testers, and DevOps engineers) follows this standard to ensure the final product meets customer expectations.

---

## 🔄 The Phases of SDLC

To explain the phases, the source uses the example of an e-commerce site, **"example.com,"** deciding to launch a new kids' clothing catalog.

---

### 1️⃣ Planning & Requirements Gathering

- **Action:** Business analysts and product owners research if customers actually want the new feature (e.g., determining which age ranges of kids' clothes are in demand).
- **Output:** If the idea is good, they create a **Software Requirement Specification (SRS)** document detailing exactly what is needed.

---

### 2️⃣ Defining & Designing

- **Action:** Architects and senior members plan the technical structure.

- **High-Level Design (HLD):**  
  Focuses on the "big picture," such as system scalability, availability, and database selection.

- **Low-Level Design (LLD):**  
  Focuses on specific logic, such as function calls and SQL queries.

---

### 3️⃣ Building (Development)

- **Action:** Developers write the code based on the design documents.
- **Tool:** The code is stored and shared in a source code repository, typically **Git**.

---

### 4️⃣ Testing

- **Action:** Quality Assurance (QA) or QE teams deploy the code to a testing server.  
  They check for bugs to ensure the product is high quality before it reaches the customer.

---

### 5️⃣ Deployment

- **Action:** The tested application is pushed to the production server, making it live for the actual customers.

> **Note:** In modern organizations, this is often done using **Agile methodology**, which repeats this cycle in short **"sprints"** rather than doing it all at once.

---

## ⚙️ Where DevOps Fits In

While a DevOps engineer should understand the whole cycle, their primary goal is to improve efficiency through automation.

### 🔑 The "Big Three" Focus Areas

DevOps engineers are most concerned with automating these three specific phases:

1. Building  
2. Testing  
3. Deployment  

### 🎯 The Goal

To remove manual intervention.  
A DevOps engineer does not write the application code or manually test it; instead, they write scripts to ensure the code moves from the developer's computer to production automatically and quickly.

---

## ✅ Quick Revision Checklist

- **Definition:** SDLC is a standard to design, develop, and test software.
- **Goal:** Deliver a high-quality product to the customer.
- **Key Documents:** Software Requirement Specification (SRS), High-Level Design (HLD), Low-Level Design (LLD).
- **DevOps Role:** Automate the Build, Test, and Deploy phases to increase speed and efficiency.

=====================================================================================================================================================================


# 🏢 DevOps Engineer: Organizational Context & Roles

This document summarizes the **practical organizational context and roles for a DevOps engineer**, complementing SDLC knowledge by focusing on **roles, workflow, and project management tools**.

---

## 🔗 The "Chain of Command": How Work Reaches You

In a real organization (like the example given of **Amazon Fresh**), a DevOps engineer rarely gets requirements directly from a customer.  
Instead, the requirement flows through a specific chain of roles before becoming a technical task.

### 1️⃣ The Customer
- Provides feedback or requests  
  *(e.g., "We want 15-minute grocery delivery")*

### 2️⃣ Business Analyst (BA)
- Interacts with the customer
- Gathers requirements
- Creates a **Business Requirement Document (BRD)**

### 3️⃣ Product Manager (PM)
- Sets the vision and prioritizes the requirements
- Decides **when** a feature should be built  
  *(e.g., "We will do this in Q1")*
- Makes decisions based on market competitors

### 4️⃣ Product Owner (PO)
- Converts the PM's vision into actionable items
- Creates **Epics** or **Features**
- Defines exactly what **"done"** looks like

### 5️⃣ Solutions Architect (SA)
- Technical expert
- Creates **High-Level Design (HLD)** and **Low-Level Design (LLD)**
- Determines technical feasibility  
  *(e.g., "Do we have the skills/tools for this?")*

---

## 👥 The Scrum Team: Where DevOps Fits In

Once the design is ready, the **Scrum Team** takes over to build the feature.

### 👤 Team Members
- Developers  
- QA Engineers  
- DevOps Engineers  
- Database Administrators  
- Sometimes Technical Writers  

### 🛠️ How DevOps Gets Tasks
- Requirements usually come from **Developers** or **Architecture designs**

**Example:**  
A developer needs to build the *"15-minute delivery"* feature.  
To do this, they may need:
- A Kubernetes cluster  
- A Docker file  
- A Git repository  

Creating these resources becomes the task assigned to the **DevOps engineer**.

### 🚀 Beyond Requests
- DevOps engineers also **proactively identify gaps** in the SDLC
- Example: Converting manual testing into an automated **CI/CD pipeline**

### 📡 Post-Launch Role
- Once the product is live, **Site Reliability Engineers (SRE)** often take over
- They monitor availability, reliability, and handle alerts
- DevOps and SRE roles can overlap

---

## 📋 Project Management: Using Jira

Organizations use project management tools like **Jira** to track work and give management visibility.

### 🧩 Key Jira Concepts

- **Epics:**  
  Large bodies of work created by the Product Owner  
  *(e.g., "Enable 15-minute delivery")*

- **Stories:**  
  Smaller tasks broken down from Epics  
  *(e.g., "Create AWS RDS Database", "Configure Kubernetes Cluster")*

- **The Sprint:**  
  A **2–3 week cycle** where the team commits to completing selected stories

  - **Backlog Refinement:**  
    Continuous process where new stories are added  
    If a DevOps engineer is busy, tasks remain in the backlog until the next sprint

---

## ✅ Quick Revision Checklist

- **No Direct Client Contact:** DevOps engineers work with internal teams, not external customers
- **Role Distinction:**
  - **PM:** Prioritizes (Strategy)
  - **PO:** Breaks down into tasks (Execution)
  - **Architect:** Designs the tech (Blueprint)
- **DevOps Trigger:** Work often starts when a developer needs infrastructure
- **The Tool:** Jira is used to track **Epics** (big goals) and **Stories** (daily tasks)


========================================================================================================================================


# 🖥️ Day 3: Virtual Machines (VMs) – DevOps Infrastructure Basics

This session shifts focus from the SDLC process to the **fundamental infrastructure** that makes efficient software development possible.

---

## ⚙️ The Core Philosophy: Efficiency

The instructor emphasizes that **DevOps is all about efficiency**.  
To explain this, a **Landlord Analogy** is used:

- **Inefficient:**  
  You own a 1-acre plot of land but build a house that only uses half an acre.  
  The rest is wasted.

- **Efficient:**  
  You build a second property on the unused half and rent it out.  
  Now, two families live on the same physical land without interfering with each other.

- **The Lesson:**  
  Virtualization does the exact same thing for computer servers, ensuring resources aren't wasted.

---

## 🚫 The Problem: Underutilization of Physical Servers

Before virtualization, companies faced a major waste problem:

- **The Scenario:**  
  A company (e.g., *example.com*) buys a powerful physical server from a vendor like **HP** or **IBM** with **100GB of RAM**.

- **The Waste:**  
  They deploy an application that only requires **4GB of RAM**.  
  The remaining **96GB** sits idle and wasted.

- **The Limit:**  
  Previously, one physical server was often dedicated to just one team or task.  
  This meant a data center with **100 servers** could only support **100 teams**.

---

## 🧠 The Solution: Virtualization & Hypervisors

To solve this, we use **Virtualization** to create **logical machines** instead of relying only on physical ones.

- **The Hypervisor:**  
  Specific software (like **VMware** or **Xen**) installed on a physical server.  
  It is the key to the whole process.

- **Logical Isolation:**  
  The hypervisor logically partitions the physical server.  
  It tricks the hardware into thinking it is multiple separate computers.

- **The Result:**  
  One physical server can host **VM 1**, **VM 2**, and **VM 3** simultaneously.

  - Each VM has its own virtual CPU, memory, and hardware.
  - They function independently; if **VM 1** crashes, it does not affect **VM 2**.

---

## ☁️ How the Cloud (AWS) Uses This

The instructor connects virtualization directly to **Cloud Computing** (like AWS or Azure).

1. **Physical Infrastructure:**  
   AWS builds massive data centers (in regions like **Mumbai** or **Singapore**) filled with racks of physical servers.

2. **Your Request:**  
   When you request a virtual machine (called an **EC2 instance** in AWS) via their portal, you are sending a request to one of those data centers.

3. **The Hypervisor's Job:**  
   AWS finds a physical server with available space.  
   The hypervisor on that server creates a logical slice (the VM) for you.

4. **Access:**  
   You never touch the physical server.  
   AWS sends you an **IP address** and a **key**, giving you remote access to your logical machine.

---

## ✅ Quick Revision Checklist

- **DevOps Goal:** Improve efficiency (stop wasting resources).
- **Physical Server:** The actual *bare metal* hardware bought from companies like HP.
- **Hypervisor:** The software that slices a physical server into multiple Virtual Machines  
  *(e.g., VMware, Xen)*.
- **Virtual Machine (VM):** A logical computer with its own OS and resources that lives on a physical server.
- **Latency:** The delay in data transfer.  
  You choose a data center region (e.g., Mumbai vs. Ohio) based on where you are to reduce this delay.


====================================================================================================================

# 🖥️ Day 4: Creating Virtual Machines (AWS & Azure)

This session moves from the **theory of virtualization** to the **practical creation of Virtual Machines (VMs)** and highlights the importance of **automation** in DevOps.

---

## 🔁 1. The Workflow: Manual vs. Automated

To understand how to create a Virtual Machine (VM), two approaches are explained: the **manual way** (for learning) and the **automated way** (for DevOps efficiency).

### 🖱️ The Manual Process (UI)

- **Action:** A user (Mr. X) logs into the Cloud Console (AWS or Azure) via a web browser.
- **Request:** Clicks buttons to request a VM (called an **EC2 Instance** in AWS).
- **Response:** The cloud provider validates the request and returns details such as the **IP address** needed to access the machine.
- **Limitation:** This is inefficient. If 100 VM requests are received, repeating this manually 100 times wastes time and introduces errors.

---

### 🤖 The Automated Process (DevOps Way)

- **Action:** Instead of using a browser, a DevOps engineer writes a **script**.
- **Mechanism:** Cloud providers expose an **API** (e.g., AWS EC2 API).  
  The script sends requests to this API.
- **Security:** The API request must be:
  - **Valid** – correct format  
  - **Authenticated** – you are who you say you are  
  - **Authorized** – you have permission to create VMs  

---

## 🛠️ 2. Tools for Automation

The following tools allow interaction with cloud APIs instead of using the manual console.

| Tool | Description | Best Use Case |
|-----|-------------|---------------|
| AWS CLI | Command Line Interface | Simple scripting directly from the terminal |
| SDKs (e.g., Boto3) | Software Development Kits (Python) | If you know programming and want to code the infrastructure |
| AWS CFT | CloudFormation Templates | Native AWS templating language to define infrastructure |
| AWS CDK | Cloud Development Kit | Newer AWS tool; gets support for new AWS features faster |
| Terraform | Open Source / Multi-Cloud | Industry leader; works across AWS, Azure, and Google Cloud |

**Tip:**  
If your company uses a **Hybrid Cloud** (e.g., AI on Google Cloud + Storage on AWS), **Terraform** is the best choice because it manages multiple platforms.

---

## 🧪 3. Practical Lab: Creating a VM on AWS

A walkthrough of creating your first VM **manually** on AWS:

1. **Account Creation:**  
   A credit/debit card is required for identity verification  
   (a small temporary charge is made).

2. **Service Selection:**  
   Go to **EC2 (Elastic Compute Cloud)** and click **"Launch Instance"**.

3. **OS Selection:**  
   **Ubuntu** is recommended for beginners as it is widely used in the DevOps community.

4. **The Golden Rule:**  
   Always select **"Free Tier Eligible"** (usually `t2.micro`).  
   This provides **750 hours free per month for one year**.

5. **Key Value Pair (Crucial):**
   - Create and download a **key pair** (usually a `.pem` file).
   - ⚠️ **Warning:** This file is your *password*.  
     If you lose it, you **cannot log in** to your server later.

---

## ☁️ 4. AWS vs. Azure (Quick Comparison)

While the technical process is similar, some administrative differences exist:

- **Login:**  
  Azure allows easy sign-up/login using a **GitHub account**.

- **Free Tier Duration:**
  - **AWS:** ~1 year free tier
  - **Azure:** Shorter free period (~30–45 days), making long-term free practice harder

---

## ✅ Quick Revision Checklist

- **EC2:** The AWS name for a Virtual Machine
- **Efficiency:** Core DevOps goal achieved by replacing manual console clicks with scripts
- **API:** Backend interface that scripts communicate with (e.g., AWS EC2 API)
- **Terraform:** Preferred tool for **Hybrid Cloud** automation
- **Key Pair:** Security file required to log in to your instance — **never lose it**


========================================================================================================


# 🔐 Connecting to an EC2 Instance from Windows (Practical Guide)

This session follows the creation of a Virtual Machine and focuses on **how to access and control an AWS EC2 instance from a Windows laptop**.

---

## ❗ The Problem: Windows vs. Linux Connectivity

Many DevOps students using **Windows laptops** face issues when connecting to **Linux-based EC2 instances**.

- **The Issue:**  
  Tools like **PuTTY** can be confusing because they require converting key files into a `.ppk` format.

- **The Solution:**  
  The source recommends using **MobaXterm** instead of PuTTY.  
  It is easier to use and supports standard key files (`.pem`) directly.

---

## ☁️ Step 1: Setting up the EC2 Instance (AWS Side)

Before connecting, ensure the instance is launched correctly.

1. **Launch Instance:**  
   Select **Ubuntu** as the operating system and **t2.micro (Free Tier)**.

2. **Key Pair (Crucial Step):**  
   When creating the key pair, select the **`.pem` format**.
   - **Note:** PuTTY requires `.ppk`, but **MobaXterm works best with `.pem`**.

3. **Network Check:**  
   Ensure the **Public IP** option is enabled so the instance is reachable from the internet.

---

## 💻 Step 2: Installing the Tool (Windows Side)

1. **Download:**  
   Search for **MobaXterm** and download the **Home Edition** or **Community Edition** (Installer version).

2. **Install:**  
   The download comes as a ZIP file.  
   Extract it and run the installer.

---

## 🔗 Step 3: Establishing the Connection

After opening MobaXterm, follow these steps to connect:

1. **Open Session:**  
   Click **Session** and select **SSH (Secure Shell)**.

2. **Remote Host:**  
   Paste the **Public IP address** of your EC2 instance (copied from the AWS console).

3. **Username:**  
   Enter `ubuntu`  
   *(default username for Ubuntu EC2 instances)*

4. **Authentication (The Key):**
   - Go to **Advanced SSH settings**
   - Check **Use private key**
   - Browse and select the **`.pem` file** downloaded from AWS

5. **Connect:**  
   Click **OK** and accept the prompt.  
   You will be logged directly into the **Linux terminal**.

---

## ✅ Quick Revision Checklist

- **The Tool:** MobaXterm (recommended over PuTTY for Windows users)
- **The Protocol:** SSH
- **The Key Format:** `.pem` (not `.ppk`)
- **The Username:** `ubuntu` for Ubuntu EC2 instances
- **Verification:**  
  Run commands like `sudo apt update` to confirm access and control over the remote machine


==============================================================================================================

# 🚀 Day 5: Connecting to VMs & AWS Automation

This session bridges the gap between **creating a server** and **efficiently controlling it** using terminals and automation.

---

## 🔌 1. Connecting to Your EC2 Instance

There are two main ways to log in to the EC2 server you created.

---

### 🅰️ Method A: Browser-based (AWS Instance Connect)

- **How:**  
  AWS Console → Select your instance → Click **Connect**
- **Pros:**  
  Very easy, no setup required
- **Cons:**  
  Not efficient for daily work  
  Sessions time out quickly when idle  
  Managing multiple servers this way is slow

---

### 🅱️ Method B: The Terminal (The DevOps Way)

- **Tools:**
  - **Mac:** iTerm
  - **Windows:** MobaXterm or Git Bash  
    *(avoid the default Command Prompt)*

- **The Command:**


ssh -i <your-key.pem> ubuntu@<public-ip>


---

### ⚠️ Critical Troubleshooting: Permission Errors

If you see errors like:
- `Permission denied`
- `Unprotected Private Key File`
- `Permissions are too open`

**Reason:**  
Your `.pem` key file is readable by other users on your system.

**Fix:**  
Restrict permissions so only you can read the file.

- **Command:**

chmod 600 <your-key-name.pem>


- **Note:**  
This is a **mandatory security step**. AWS will not allow SSH access otherwise.

---

## 🤖 2. Automation: The AWS CLI

Clicking buttons in the browser is **not scalable**.  
DevOps engineers automate tasks using the **AWS Command Line Interface (CLI)**.

---

### 🛠️ Step 1: Installation

- Download the AWS CLI installer for **Windows / Mac / Linux** from AWS documentation
- Verify installation:

aws --version


---

### 🔐 Step 2: Authentication (Access Keys)

Since terminals have no login screen, AWS uses **Access Keys**.

1. Go to **AWS Console → Security Credentials**
2. Click **Create Access Keys**
3. ⚠️ **Warning:**  
 You will see:
 - Access Key ID
 - Secret Access Key  

 Copy and store them securely.  
 **The Secret Access Key is shown only once.**

---

### ⚙️ Step 3: Configuration

- Run:

aws configure

- Enter:
- Access Key ID
- Secret Access Key
- Default region (e.g., `us-east-1`)
- Output format (`json`)

---

### ✅ Step 4: Testing

Once configured, you can manage AWS directly from your terminal.

- **Example:**

aws s3 ls


This lists all your S3 buckets **without opening a browser**.

---

## 🧩 3. Other Automation Options (Brief Overview)

These advanced tools fall under **Infrastructure as Code (IaC)** and will be covered later.

- **CloudFormation Templates (CFT):**  
AWS-native way to define infrastructure using **JSON or YAML** files

- **Boto3 (Python):**  
Python library to manage AWS resources  
*(e.g., script to list all running EC2 instances)*

---

## ✅ Quick Revision Checklist

- **Best Terminal:** iTerm (Mac) / MobaXterm (Windows)
- **SSH Fix:** Use `chmod 600` on your `.pem` key
- **AWS CLI:** Tool to manage AWS from the command line
- **aws configure:** Links your terminal to AWS using Access Keys
- **Access Keys:** Username/password for scripts and automation — **keep them secret**


=========================================================================================


