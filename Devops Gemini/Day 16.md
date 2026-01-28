# 🧱 Day-16 | Infrastructure as Code (IaC) & Terraform

---

Based on the transcript for **"Day-16 | Infrastructure as Code (IaC) & Terraform,"** here is a summary of the theoretical concepts required before starting the practical implementation.  
This session explains why the industry moved from provider-specific tools to Terraform.

---

## 1️⃣ The Business Problem: The "Language" Barrier

Imagine you are a DevOps engineer at a company like Flipkart. You spend months automating infrastructure on AWS using their native tool, CloudFormation (CFT),.

### Scenario A (Migration)
Management decides to switch to Microsoft Azure or On-Premise (OpenStack) to save money.

- **The Issue:** Your CloudFormation scripts are useless on Azure. You must re-write everything from scratch using Azure Resource Manager (ARM) or Heat Templates,.

### Scenario B (Hybrid Cloud)
Your company decides to use AWS for storage (S3) but Azure for DevOps pipelines.

- **The Issue:** You are forced to learn and master two completely different tools simultaneously,.

---

## 2️⃣ The Solution: Terraform

To solve the problem of learning dozens of different tools for different clouds, HashiCorp created Terraform.

- **One Tool, Any Cloud:** Instead of learning CFT, ARM, and Heat Templates, you only learn Terraform.
- **The Workflow:** You write a script in Terraform language. Terraform then translates your script into the specific commands required by the cloud provider (AWS, Azure, GCP, etc.),.
- **Easier Migration:** If you move from AWS to Azure, you don't have to learn a new language. You simply update the "Provider" details in your existing Terraform scripts to target the new cloud.

---

## 3️⃣ Key Concept: API as Code

The source describes Terraform's underlying mechanism as **"API as Code"**.

### What is an API?

- **Manual:** When you visit google.com in a browser, you are using a User Interface (UI).
- **Programmatic:** When a script talks to Google without a browser to get data, it uses an Application Programming Interface (API).

### How Terraform uses this:

- Every Cloud Provider (AWS, Azure) exposes APIs to create resources.
- Instead of you writing complex Python or Shell scripts to hit these APIs, you write simple Terraform configuration files.
- **The Translation:** You write "Create EC2" in Terraform. Terraform converts that request into the specific API call that AWS understands and executes it.

---

## 4️⃣ Summary of Tools mentioned

| Tool | Cloud Provider | Disadvantage |
|-----|---------------|--------------|
| CloudFormation (CFT) | AWS | Only works on AWS. |
| Azure Resource Manager | Azure | Only works on Azure. |
| Heat Templates | OpenStack (On-Prem) | Only works on OpenStack. |
| Terraform | Agnostic (All) | Works across all providers using a single syntax. |

---


==========================================================================


# 🧱 Day-17 | Everything about Terraform

---

Based on the transcript for **"Day-17 | Everything about Terraform,"** here is a summary of the workflow, best practices, and interview scenarios.  
This session moves from the theoretical **"API as Code"** concept (Day 16) to building a production-ready Terraform setup.

---

## 1️⃣ The Core Concept: Terraform as a Translator

Terraform automates infrastructure by acting as a **"translator"** between your code and the Cloud Provider.

- **How it works:** You write configuration files (e.g., "Create EC2"). Terraform converts these files into specific API calls that AWS (or Azure/GCP) understands,.
- **The Advantage:** You learn one tool to manage any cloud. If you switch from AWS to Azure, the logic remains similar, even if the specific resource names change.

---

## 2️⃣ The Terraform Lifecycle (The 4 Commands)

Every Terraform project follows a specific order of execution:

1. **terraform init:** Initializes the project. It downloads the necessary plugins (providers) required to talk to AWS,.
2. **terraform plan (Dry Run):** A safety check. It shows you exactly what resources will be created or deleted before actually doing it. This prevents accidental deletions.
3. **terraform apply:** Executes the API calls to actually create the infrastructure in the cloud.
4. **terraform destroy:** Deletes all resources tracked by the project.

---

## 3️⃣ The "Heart" of Terraform: The State File

The most critical concept in this session is the **State File (`terraform.tfstate`)**.

- **Purpose:** Terraform uses this file to "track" the infrastructure. It maps your code to the real-world resources (e.g., mapping a config block to an actual EC2 Instance ID).
- **The Problem:**
  - It contains sensitive data (secrets/keys).
  - It cannot be merged like code (Git merge doesn't work on state files).
  - **Rule #1:** Never store the State File on your local laptop or in a Git repository,.

---

## 4️⃣ The Ideal Architecture: Remote Backend & Locking

To fix the State File issues, organizations use a **Remote Backend**. This is a standard interview answer for *"How do you set up Terraform?"*

- **Storage (S3):** Store the State File centrally in an AWS S3 Bucket. This ensures the team shares a single source of truth,.
- **Locking (DynamoDB):** Integrate with a DynamoDB table.
  - **Why?** If two engineers run `terraform apply` at the same time, they might corrupt the infrastructure. DynamoDB "locks" the state file so only one person can modify it at a time,.

---

## 5️⃣ Code Structure & Best Practices

Do not put everything in one file. A professional setup splits the code:

- **main.tf:** Contains the resource logic (EC2, VPC).
- **variables.tf (Inputs):** defines variables (e.g., `instance_type`) so you don't hardcode values. This allows you to easily change settings without touching the main code.
- **outputs.tf:** Defines what data to show after deployment (e.g., printing the Public IP of the new server so the user can log in),.
- **Modules:** Reusable blocks of code. Instead of writing the code for a "Web Server" 10 times, write it once as a Module and reuse it across Dev, Stage, and Prod environments,.

---

## 6️⃣ Limitations (Interview Prep)

Be prepared to discuss the disadvantages of Terraform:

- **No Bi-Directional Sync:** If someone manually logs into the AWS Console and changes a server (e.g., changes RAM), Terraform won't automatically know. It relies strictly on the State File, leading to **"State Drift"**.
- **Not GitOps Friendly:** Because the State File is so sensitive and complex, it doesn't play perfectly with standard GitOps tools like ArgoCD compared to Kubernetes manifests,.

---

===================================================================================

# 🗂️ Day-22 | Project Management tools for DevOps

---

Based on the transcript for **"Day-22 | Project Management tools for DevOps,"** here is a summary of the non-technical but critical workflows a DevOps engineer encounters in their first week.  
This session shifts focus from coding to Project Management and Process.

---

## 1️⃣ The Context: The "First Week"

While technical skills (Linux, Docker) get you the job, understanding how the team organizes work is what you do once you join. You need to know how tasks are assigned, tracked, and documented.

---

## 2️⃣ The Methodology: Agile vs. Waterfall

Most organizations today follow Agile, and you are expected to understand it immediately.

- **Waterfall (The Old Way):** The team designs, codes, and tests the entire application in one massive cycle (e.g., 4–5 months). If a mistake is found at the end, it is too late to fix easily.
- **Agile (The New Way):** Work is broken into small chunks called Sprints (usually 2–3 weeks).
  - **Benefit:** Developers deliver small parts to the QA team frequently. If there is a bug, feedback is immediate.

---

## 3️⃣ The Tracker: Jira

Jira is the industry-standard tool used to implement Agile. It acts as the central dashboard for the company.

- **High Level:** Executives use it to track high-priority issues (P1 bugs) across the entire organization.
- **Team Level:** Developers use it to see their "Stories" (tasks) for the current Sprint and track their backlog.
- **Action Item:** The source recommends creating a Free Trial account with Atlassian to get familiar with the interface so you aren't lost on your first day.

---

## 4️⃣ The Library: Confluence & SharePoint

Code lives in Git, but knowledge lives in Confluence (Atlassian) or SharePoint (Microsoft).

- **Knowledge Sharing:** If you build an internal microservice, you cannot Google how it works. You must read the internal documentation hosted here.
- **Continuity:** These platforms ensure that if a senior engineer leaves the company, their knowledge doesn't leave with them—it remains documented for the new hires.
- **Open Source Alternative:** For public projects, teams often use Read the Docs, which hosts static documentation websites linked directly to GitHub.

---

## 5️⃣ The Firefighter: ServiceNow

In large organizations, DevOps engineers interact with ServiceNow for two specific processes:

### 🔥 Incident Management (Reacting)
- **Scenario:** A monitoring tool (like CloudWatch) detects a crash.
- **Workflow:** It triggers an alert -> ServiceNow automatically creates an "Incident Ticket" -> It assigns it to the relevant engineer to fix immediately.

### 🛠️ Change Management (Proacting)
- **Scenario:** You have fixed the bug and need to deploy code to Production.
- **Workflow:** You cannot just "push" code to a live bank server. You must open a "Change Request." This tracks the downtime, the approval from vendors/managers, and the rollout plan.

---

## 6️⃣ Project Management for Open Source

If you are working on open-source projects (where you don't pay for Jira), teams use GitHub Projects or Azure Boards. These allow you to track "To Do," "In Progress," and "Done" tickets directly within the repository.

---

================================================================================================


# 📦 Day-23 | Introduction to Containers

---

Based on the transcript for **"Day-23 | Introduction to Containers,"** here is a summary of the concepts.  
This session marks the beginning of the containerization module, explaining the theory before moving to practical tools like Docker and Kubernetes.

---

## 1️⃣ The Business Problem: Resource Wastage

To understand containers, we must look at the history of hosting applications.

- **Physical Servers:** In the beginning, companies bought massive physical servers (e.g., 100GB RAM). If an app only used 10GB, the remaining 90GB was wasted.
- **Virtual Machines (VMs):** To fix this, Hypervisors were invented. This allowed running multiple "Virtual Machines" on one physical server. Each VM had its own Operating System (OS).
- **The Remaining Problem:** While better, VMs are still heavy. Each VM requires a full OS, which consumes significant resources even when idle. If a company runs 1 million VMs, the wasted resources add up to a massive financial loss.

---

## 2️⃣ The Solution: Containers

Containers are the next step in this evolution. They provide a way to package an application so it runs efficiently without the overhead of a full OS.

- **Lightweight:** Unlike a VM, a container does not have a full operating system. It shares the resources (kernel) of the host machine.
- **Size Difference:**
  - **VM Snapshot:** typically 1GB to 3GB (because it includes a full OS).
  - **Container Image:** typically 100MB to 500MB (because it only contains the app and minimal dependencies).
- **Portability:** Because they are so small, containers are very easy to "ship" (move) from a developer's laptop to a test server or the cloud.

---

## 3️⃣ Architecture: VM vs. Container

This is a fundamental concept for interviews.

| Feature | Virtual Machine (VM) | Container |
|------|----------------------|-----------|
| Isolation | High. Has a completely separate OS. Highly secure. | Logical. Shares the host OS. Slightly less secure than VMs but efficient. |
| Operating System | Runs a Full OS (e.g., a complete Windows/Linux installation). | Runs a Minimal OS. It packages only the application libraries and system dependencies. |
| Deployment Model | Runs on a Hypervisor. | Runs on a Container Engine (like Docker). |

**Note:** In modern cloud environments (Model 2), containers are often run  VMs (e.g., EC2 instances) to get the best of both worlds: efficiency and management.

---

## 4️⃣ Docker & The Lifecycle

Docker is the most popular tool to create and manage containers. The source outlines the standard lifecycle:

1. **Docker File:** A text file where you write instructions (like a script).
2. **Docker Image:** You run `docker build` to convert the file into an Image. This is a read-only template.
3. **Docker Container:** You run `docker run` to turn the Image into a live, running Container.

---

## 5️⃣ Advanced Tool: Buildah

The session introduces a newer tool called **Buildah** to address a specific flaw in Docker.

- **The Flaw (Single Point of Failure):** Docker relies on a "Docker Engine." If this engine crashes, all containers on that machine stop working.
- **The Buildah Solution:** Buildah helps create images without depending on a central engine (daemonless), making it more robust for certain automated workflows. It is often used with tools like Podman and Skopeo.

---

=========================================================================================

# 🐳 Day-24 | Docker Zero to Hero Part-1

---

Based on the transcript for **"Day-24 | Docker Zero to Hero Part-1,"** here is a summary of the core concepts, architecture, and practical workflow.  
This session focuses on the **"How" and "Why" of Docker** to prepare you for building your first container.

---

## 1️⃣ Why Containers are Lightweight (Deep Dive)

The most common interview discussion revolves around why a container is smaller than a Virtual Machine (VM).

- **Shared Kernel:** Unlike a VM, which has a full "Guest Operating System" (kernel + libs + apps), a container shares the Kernel of the host machine (e.g., the EC2 instance).
- **What is Inside:** A container image only holds the application code, binaries, and system libraries required to create "logical isolation".
- **Real-World Stats:**
  - **Ubuntu VM Image:** Typically ~2.3 GB because it has a full OS.
  - **Ubuntu Docker Image:** Only ~28 MB because it strips out the kernel and networking stack, relying on the host for those.
- **Resource Efficiency:** Because they are so small, you can run 30–40 containers on a machine that might only support one or two VMs.

---

## 2️⃣ Docker Architecture

Docker is the platform that implements the concept of containerization. It uses a **Client-Server architecture**.

- **Docker Client (CLI):** This is where you type commands (e.g., `docker build`). It sends requests to the Daemon.
- **Docker Daemon:** The "Heart" of Docker. It is a background process that listens to API requests and manages the containers. If the Daemon crashes, all running containers stop.
- **Docker Registry:** A storage location for Docker images (like Docker Hub). It allows you to share images with your team or the world.
  - **Interview Note:** GitHub is for storing source code; Docker Hub is for storing container images.

---

## 3️⃣ The Docker Lifecycle (Workflow)

To create a running application, you follow three distinct steps:

1. **Dockerfile:** A text file containing a set of instructions (e.g., "Install Python," "Copy my code").
2. **Docker Image:** You run `docker build` to convert the file into an Image. This is a read-only template or snapshot.
3. **Docker Container:** You run `docker run` to turn the Image into a live, executing Container. This is the final output that users interact with.

---

## 4️⃣ Practical: Installation & Common Errors

The session demonstrated installing Docker on an AWS EC2 Ubuntu instance.

- **Installation:** Use `sudo apt install docker.io`.
- **The "Permission Denied" Error:**
  - **Problem:** By default, the Docker Daemon runs as the Root user. If you run `docker run` as a normal user (e.g., ubuntu), it will fail.
  - **Solution:** Add your user to the Docker group using `sudo usermod -aG docker $USER`. You must then log out and log back in for changes to take effect.

---

## 5️⃣ Writing the First Dockerfile

The project involved containerizing a simple Python **"Hello World"** app.

The Dockerfile structure used:

- **FROM ubuntu:latest:** Defines the base OS image.
- **WORKDIR /app:** Sets the working directory inside the container.
- **COPY . .:** Copies the code (app.py) from your laptop into the container.
- **RUN apt-get install python3:** Installs dependencies inside the image.
- **CMD ["python3", "app.py"]:** The command that runs automatically when the container starts.

---

## 6️⃣ Sharing the Image (Push/Pull)

Once the image is built locally, it needs to be shared.

1. **Login:** Run `docker login` and enter your Docker Hub credentials.
2. **Push:** Run `docker push [username]/[repo_name]:[tag]`. This uploads the image to the remote registry.
3. **Pull:** Any other developer can now run `docker pull` to download and run your exact application without installing Python manually on their machine.

---

=======================================================================

# 🐳 Day-25 | Docker Containerization for Django

---

Based on the transcript for **"Day-25 | Docker Containerization for Django,"** here is a summary of the workflow.  
This session advances from basic commands to containerizing a real-world Python web application.

---

## 1️⃣ The Goal: Containerizing a Real App

In the previous session, we ran a simple CLI script. Today, the goal is to deploy a **Django (Python) Web Application**.

- **The Workflow:** The process is standard regardless of the app's size. Whether it is a single-page app or a complex microservice, the DevOps steps (**Build -> Ship -> Run**) remain the same.
- **The Benefit:** By containerizing the app, you solve the **"It works on my machine"** problem. The container bundles all dependencies (like Python and Django versions), so it runs identically on a developer's Mac, a QA's Windows laptop, or a production Linux server.

---

## 2️⃣ Pre-requisite: Do DevOps Engineers Need to Code?

The speaker addresses a common question: **"Do I need programming experience?"**

- **The Answer:** You do not need to be an expert developer, but you must understand the application flow.
- **Why?** To write a Dockerfile, you need to know:
  1. Which language the app uses (e.g., Python).
  2. Where the dependencies are listed (e.g., `requirements.txt`).
  3. How to start the application (e.g., `manage.py runserver`).

---

## 3️⃣ The Dockerfile Steps

To containerize the Django app, the Dockerfile was built with these specific instructions:

1. **FROM ubuntu:** Selects the base OS (the speaker chose Ubuntu to demonstrate installing Python manually, though python images exist).
2. **WORKDIR /app:** Creates a standard directory inside the container to store code.
3. **COPY requirements.txt .:** Moves the dependency list from your laptop to the container.
4. **RUN pip install ...:** Installs the required libraries (Django) inside the image.
5. **COPY . .:** Copies the actual source code into the container.

---

## 4️⃣ Critical Concept: Entrypoint vs. CMD

This is a **major interview topic** covered in this session. Both commands tell the container how to start, but they function differently.

### 🔹 ENTRYPOINT (The Executable)

- These instructions **cannot be overridden easily** by the user.
- **Use Case:** You define the main executable (e.g., `python3`) here because you don't want a user to accidentally try running the container as a Node.js app.

### 🔹 CMD (The Arguments)

- These instructions **can be overridden** by the user at runtime.
- **Use Case:** You place arguments here (e.g., `manage.py runserver`). If a user wants to run the server on a different IP or pass different flags, they can overwrite this part without rebuilding the image.

---

## 5️⃣ Deployment & Networking (Port Mapping)

Once the image is built using `docker build`, you must run it.

- **The Common Error:** If you run `docker run -it [image]`, the app will start inside the container, but you cannot access it from your browser.
- **The Fix (Port Mapping):** You must map the container's port to the host's port.
  - **Command:** `docker run -p 8000:8000 [image_id]`
  - **Explanation:** This connects Port **8000** on the container (where Django runs) to Port **8000** on your EC2 instance.
- **The Firewall (AWS Security Groups):** Even with port mapping, AWS blocks traffic by default. You must edit the EC2 Security Group Inbound Rules to allow **"Custom TCP"** traffic on **Port 8000** from **"Anywhere"**.

---

======================================================================================================================================




