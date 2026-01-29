# 🚀 Day-18 | What is CI/CD?

---

Based on the transcript for **"Day-18 | What is CI/CD?"**, here is a summary of the concepts.  
This session takes a step back from specific tools like Kubernetes to explain the **fundamental process of software delivery**.

---

## 1️⃣ The Core Concept: From Laptop to Customer

The session defines **CI/CD** as the **automated bridge** between a developer writing code and a customer actually using it.

### 🔹 The Problem
- You write code on your laptop
- Your customer is across the globe
- Doing **manual checks** (testing, security, deployment) for every single change would take **months**

### 🔹 The Solution (CI/CD)

- **Continuous Integration (CI)**  
  The process of integrating a set of tools to **validate code immediately after a developer commits it**

- **Continuous Delivery (CD)**  
  The process of **deploying the validated application** to a platform (server, cloud, or app store) so customers can use it

---

## 2️⃣ The Standard Pipeline Stages

A CI/CD pipeline automates a **fixed checklist of tasks**.

| Stage | Definition | Example |
|-----|-----------|--------|
| **1. Unit Testing** | Testing a specific function in isolation to ensure logic is correct | If building a calculator, test that `2 + 3 = 5` |
| **2. Static Code Analysis** | Checking syntax errors, formatting, indentation, unused variables | Declared 20 variables but used only 2 |
| **3. Vulnerability Testing** | Scanning for security bugs or outdated dependencies | Detecting a security issue in a new Android update |
| **4. Functional Testing** | End-to-end testing to ensure new changes didn’t break old features | Adding subtraction should not break addition |
| **5. Reporting & Deployment** | Generating reports and delivering the app | Deploying to EC2 or Kubernetes |

---

## 3️⃣ The Architecture: Legacy vs. Modern

This section explains how CI/CD evolved from **always-on servers** to **event-driven automation**.

---

### 🏗️ A. The Legacy Orchestrator: Jenkins

- **Role**  
  Jenkins acts as a **pipe / orchestrator**  
  It watches a Git repository and triggers external tools like Maven or SonarQube

- **Architecture**  
  Master / Node model running on **Virtual Machines**

- **Problems**
  - **Cost Inefficiency**  
    Servers must stay running even when no one is committing code (weekends, nights)
  - **Maintenance Overhead**  
    Difficult to scale down to zero
  - **Idle Resources**  
    Paying for compute even when unused

---

### ⚡ B. The Modern Solution: GitHub Actions

- **Real Example**  
  Kubernetes open-source repository  
  Thousands of contributors with minimal wasted resources

- **How it works**
  - Event-Driven Architecture
  - When a Pull Request (PR) is created:
    - A temporary container (Runner) is created
    - Tests are executed
    - Container is destroyed immediately

- **Key Benefit**
  - No idle servers
  - No cost when no code is being committed

---

## 4️⃣ Deployment Strategy: The 3 Environments

In real organizations, code does **not go directly to production**.

### 🧪 1. Dev Environment
- Simple setup
- Used for initial functional testing
- May be a single server

### 🧰 2. Staging Environment
- Pre-Production environment
- Mimics production architecture
- Uses fewer resources to reduce cost  
  (e.g., 5 nodes instead of 30)

### 🌍 3. Production Environment
- Live environment
- Used by real customers
- Fully scaled and highly available

---

========================================================================

# 🚀 Day-19 | Jenkins ZERO to HERO

---

Based on the transcript for **"Day-19 | Jenkins ZERO to HERO"**, here is a summary of the **practical implementation session**.  
While **Day 18** covered the **theory of CI/CD**, **Day 19** focuses on **installing Jenkins, configuring Docker Agents, and running real pipelines**.

---

## 1️⃣ Installation & Setup (The Basics)

The session begins by setting up a **fresh Jenkins server on AWS**.

- **The Server**  
  An **Ubuntu EC2 instance** is launched.

- **The Prerequisite**  
  Jenkins is a **Java-based application**, so **Java (JDK)** must be installed first.

- **Networking**  
  Jenkins runs on **Port 8080** by default.  
  You must edit the **AWS Security Group (Inbound Rules)** to allow traffic on port **8080**, otherwise the UI will not be accessible.

- **Unlock Key**  
  The initial administrative password is stored in a file on the server.  
  You must `cat` this file to log in for the first time.

---

## 2️⃣ Architecture Evolution: The "Docker Agent" Strategy

The most critical concept in this session is moving from the **Old Way** of Jenkins to the **Modern Way**.

| Architecture | Description | Problems / Benefits |
|-------------|------------|---------------------|
| **Old Way (Static Agents)** | Permanent EC2 instances (Worker Nodes) attached to Jenkins Master | ❌ Expensive & Messy. Dependency conflicts (Java 8 vs Java 11). Idle servers waste money |
| **New Way (Docker Agents)** | Jenkins spins up a container only when a job starts | ✅ Efficient & Clean. Containers are destroyed after execution. No idle resources |

---

## 3️⃣ Configuring Jenkins for Docker

To enable the **Modern Docker Agent approach**, the following steps were demonstrated:

1. **Install Docker**  
   Docker must be installed on the Jenkins server (or remote node).

2. **Permissions (Critical Step)**  
   By default, the `jenkins` user cannot access Docker.  
   You must add it to the Docker group:

=========================================================================

# 🚀 Day-20 | GitHub Actions

Based on the transcript for **"Day-20 | GitHub Actions,"** here is a summary of the concepts. This session builds directly on Day 19, offering a modern, serverless alternative to the Jenkins setups we previously discussed.

---

## 1️⃣ What is GitHub Actions?

**GitHub Actions** is a CI/CD platform built directly into GitHub.

* **The Difference:** Unlike Jenkins, which requires you to provision, install, and maintain a separate server, GitHub Actions is a SaaS (Software as a Service) solution.
* **The Scope:** It is platform-centric. It is designed specifically for code hosted on GitHub. If your company uses GitLab or Azure DevOps, this tool is not suitable.

---

## 2️⃣ How to Setup a Pipeline (The Workflow)

To create a pipeline in Jenkins, we wrote a Jenkinsfile. In GitHub Actions, the process is slightly different but follows a similar logic.

* **File Location:** You must create a specific directory structure in your repository:

  ```
  .github/workflows
  ```
* **File Format:** The pipeline is defined in a YAML file.
* **Triggers:** You define events that start the pipeline using the `on:` keyword.

  * Examples: `on: push`, `on: pull_request`, or `on: issues`.
  * **Flexibility:** You can have multiple workflow files (e.g., one for CI, one for checking PR titles, one for security scanning) running simultaneously.

---

## 3️⃣ Key Components of the Syntax

The session breaks down the anatomy of a GitHub Actions YAML file:

| Component           | Definition                                             | GitHub Context                                                                                                                                                        |
| ------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runner**          | The environment where the code runs.                   | You usually specify `runs-on: ubuntu-latest`. This creates a temporary container provided by GitHub.                                                                  |
| **Job**             | A collection of steps (similar to a Jenkins Pipeline). | You can define multiple jobs (e.g., one for Python 3.8, one for Python 3.9).                                                                                          |
| **Step**            | A specific task within a job.                          | Examples: "Checkout Code", "Install Python", "Run Test".                                                                                                              |
| **Action (Plugin)** | Reusable code to perform complex tasks.                | Instead of installing plugins on a server (like Jenkins), you simply "call" them in the YAML. <br> Example: `uses: actions/checkout@v3` automatically pulls the code. |

---

## 4️⃣ Comparison: GitHub Actions vs. Jenkins

This is a critical interview topic covered in the session.

| Feature         | Jenkins (Day 19)                                      | GitHub Actions (Day 20)                                                                   |
| --------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Hosting**     | Self-hosted (requires maintenance of Master/Nodes).   | Managed by GitHub (No maintenance required).                                              |
| **Cost**        | You pay for the EC2 instances/servers even when idle. | Free for public/open-source repos. Private repos have a minute limit (approx. 2000 mins). |
| **Plugins**     | Must be manually installed/updated by an admin.       | "Plugins" (Actions) are used directly in code from the Marketplace.                       |
| **Portability** | High. Can move Jenkins to any cloud/platform.         | Low. Locked into the GitHub ecosystem.                                                    |

---

## 5️⃣ Advanced Features

* **Matrix Testing:** The session demonstrates running the same test across multiple environments simultaneously (e.g., Python 3.8 and 3.9) simply by listing them in the YAML.
* **Secrets Management:** Just like Jenkins Credentials, GitHub has a native **Secrets** tab in repository settings. This is used to store sensitive data like kubeconfig files or SonarQube tokens.
* **Self-Hosted Runners:** If you need specific hardware or security, you can link your own private server to GitHub Actions instead of using the default `ubuntu-latest` runners.

---

## 6️⃣ Practical Demo Summary

The speaker provided examples in the repository to illustrate these concepts:

1. **Python App:** A simple "Addition" function tested with pytest. It triggers on every commit.
2. **Java App:** A full flow that checks out code, sets up Java, runs Maven build, checks code quality with SonarQube, and deploys to Kubernetes.

========================================================================

# 🔵 Day-21 | CI/CD Interview Questions

## 🎯 Purpose of This Session
This session serves as a **checkpoint**. After learning the **theory (Day 18)** and **practical implementation (Days 19–20)** of Jenkins and GitHub Actions, this session focuses on **how to answer scenario-based interview questions** to prove your experience.

---

## 🟢 1. The "Golden" Question: *"Explain your CI/CD Process"*

This is the **most common interview question**. You **cannot** simply say *"I use Jenkins."* You must describe the **flow of data through tools**.

### 📌 Scenario
Assume you are working with a **Java application**.

### 🔁 Flow Answer
1. **Source Code:** A developer commits code to **GitHub**.
2. **Trigger:** This event triggers a **Jenkins pipeline via a Webhook**.
3. **Build:** Jenkins checks out the code and uses **Maven** to build the artifact.
4. **Quality Check:**  
   - **SonarQube** for static analysis  
   - **AppScan** for security
5. **Delivery:** Once built, **ArgoCD (GitOps)** deploys the new image to a **Kubernetes cluster**.

> 🟡 **Tip:**  
> If you don't know Kubernetes, you can say you deploy to **EC2 instances**, but you must be **specific about the tools used**.

---

## 🟢 2. Triggers: Polling vs Webhooks

**Question:** How does Jenkins know when to start working?

| 🔧 Method | ⚙️ Mechanism | 📊 Verdict |
|---------|-------------|-----------|
| 🔴 **Poll SCM** | Jenkins checks GitHub every few minutes | Inefficient – wastes CPU & Network |
| 🟠 **Build Triggers** | Runs on a schedule (Cron Job) | Laggy – delayed execution |
| 🟢 **Webhooks** | GitHub sends a JSON payload only on events | **Best Practice** – event-driven & immediate |

---

## 🟢 3. Handling Secrets (Security)

**Interview Question:** *"How do you manage passwords and API keys?"*

- 🔹 **Basic Answer:**  
  Use the native **Jenkins Credentials Plugin**.
- 🔹 **Pro Answer (Recommended):**  
  Integrate **HashiCorp Vault**, an industry-standard external tool for secure secrets management.  
  Using Vault shows **enterprise-grade security knowledge**.

---

## 🟢 4. Advanced Pipeline Concepts

These questions test experience with **complex, real-world environments**.

### 🧩 A. Shared Libraries (Shared Modules)

**Scenario:**  
Team A, Team B, and Team C all need similar pipelines (Checkout → Build → Deploy).

**Solution:**  
Create a **Shared Module**:
- Write the pipeline logic **once**
- Import it into multiple pipelines
- Reduces **duplication** and **maintenance effort**

---

### 🧩 B. Multi-Language Pipelines

**Scenario:**  
A 3-tier application:
- **Frontend:** React  
- **Backend:** Java  
- **Data Layer:** Python  

**Solution:**  
Use **Docker Agents**:

- Stage 1 → `node` container  
- Stage 2 → `maven` container  
- Stage 3 → `python` container  

✅ **Benefit:**  
No need to install all languages on the Jenkins server. Clean, isolated, and scalable.

---

## 🟢 5. Administration & Maintenance

Important for **DevOps / Platform Engineer roles**.

- 🗄️ **Backups:**  
  Backup the hidden **.jenkins** home directory using `rsync`.

- 📈 **Auto-Scaling:**  
  Configure Jenkins with **AWS Auto Scaling Groups** to:
  - Add worker nodes during high load
  - Remove them when idle

- 🔗 **JNLP (Java Network Launch Protocol):**  
  Enables Jenkins **Worker Nodes (Agents)** to communicate with the **Master**.  
  Agents download `agent.jar` and connect back to the master.

---

## 🟢 6. Quick Tips for Success

- ✅ **Know your Jenkins Version:**  
  Always check the latest version before interviews.
- ✅ **Memorize Plugins:**  
  Keep 5–10 plugins ready in your mind:
  - Git  
  - Maven  
  - Docker Pipeline  
  - SonarQube  
  - Blue Ocean  

---

🟣 **End of Day-21 | CI/CD Interview Questions**

=================================================================


