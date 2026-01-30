# 🐧 Day 6: Linux Operating System & Shell Scripting Basics

This session explains the **software foundation of servers** created earlier and introduces the **Linux commands** used to control and debug them.

---

## 🖥️ 1. What is an Operating System (OS)?

An **Operating System** acts as a bridge between the **user/software** and the **physical hardware**.

- **The Hardware:**  
  Raw CPU, RAM, and Hard Disk (I/O) you purchase.

- **The Software:**  
  Applications like Jenkins, Java, or Python that you want to run.

- **The Flow:**  
  You cannot communicate directly with the hardware.  
  The user interacts with the application → the application sends a request to the OS → the OS instructs the hardware what to do.

---

## 🐧 2. Why Linux? (The DevOps Standard)

Even if you use Windows or Mac personally, **80–90% of production servers run on Linux**.

- **Free & Open Source:**  
  Linux is free to use, unlike Windows, which reduces licensing costs.

- **Security:**  
  Inherently secure and rarely needs antivirus software.

- **Speed & Stability:**  
  Lightweight and fast.  
  Production systems cannot lag or crash when thousands of users access sites like Netflix or Amazon.

- **Distributions (Flavors):**  
  Linux comes in different distributions:
  - Ubuntu (most popular for learning)
  - CentOS
  - Red Hat
  - Debian

---

## 🧠 3. Linux Architecture: The Kernel

You do not need deep architecture knowledge, but you **must understand the Kernel**.

- **Definition:**  
  The Kernel is the **heart of the Operating System**.

- **4 Key Responsibilities:**  
  In interviews, the Kernel is responsible for:
  1. Device Management
  2. Memory Management
  3. Process Management
  4. System Calls

---

## 💻 4. Shell Scripting & CLI

- **The Problem:**  
  Production servers do not have a **Graphical User Interface (GUI)** because GUIs are heavy and waste resources.

- **The Solution:**  
  Servers are managed using the **Shell (Command Line Interface)**.

- **Universal Skill:**  
  Shell commands are mostly the same across Linux distributions (Ubuntu, CentOS, etc.), so learning once applies everywhere.

---

## 📋 5. Essential Command Cheat Sheet

These are **daily-use commands** for a DevOps engineer.

---

### 🧭 A. Navigation & Listing

| Command | Purpose |
|-------|---------|
| `pwd` | Present Working Directory. Shows where you are |
| `ls` | Lists files and folders in the current directory |
| `ls -ltr` | Shows detailed list with permissions, owner, and timestamps |
| `cd <name>` | Moves into a specific directory |
| `cd ..` | Moves one level back |
| `man ls` | manual (help page) for the ls command |

---

### 📁 B. File Management

| Command | Purpose |
|------|---------|
| `mkdir` | Creates a new directory |
| `touch` | Creates an empty file |
| `rm -r` | Deletes a file or directory (`-r` for directories) |
| `vi <file>` | Opens a text editor to create or edit a file |
| `cat <file>` | Displays file contents |

#### ✏️ How to use `vi`:
1. Type `vi filename` to open
2. Press `i` to enter **Insert Mode**
3. Press `Esc` to exit Insert Mode
4. Type `:wq` to **save and exit**

---

### 📊 C. System Resources (Crucial for Debugging)

Use these when a server is slow or unresponsive.

| Command | Purpose |
|------|---------|
| `top` | Real-time CPU, memory, and process usage |
| `df -h` | Shows disk usage in human-readable format |
| `free` | Displays available RAM |
| `nproc` | Shows number of CPUs |

---

## ✅ Quick Revision Checklist

- **Kernel:** Core component that manages CPU and RAM
- **No GUI:** Servers use CLI to save resources
- **Navigation:** `pwd` to locate yourself, `ls` to view contents
- **Health Check:** Use `top` to identify CPU or memory issues
- **Editor:** `vi` is the standard terminal-based file editor


===============================================================================================


# 🟢 Day 7: Shell Scripting (Zero to Hero)

This session advances your knowledge from simply using Linux commands to **automating tasks** using shell scripts.

---

## 🔵 1. What is Shell Scripting?

Shell scripting is simply the process of **automating manual activities** on a Linux machine.

### 🟡 Why Shell Scripting? (The 1,000 Files Problem)

* If asked to create **one file**, you can do it manually
* If asked to create **1,000 files**, doing it manually is impossible
* <span style="color:green"><b>The Solution:</b></span> You write a script (a set of instructions) to do it for you instantly

---

## 🔵 2. Writing Your First Script

To write a script, you generally follow these steps:

### 🟡 File Extension

* Scripts usually end with <span style="color:blue"><b>.sh</b></span> (e.g., `myscript.sh`)
* Similar to how Python uses `.py`

### 🟡 The Shebang (`#!`)

```bash
#!/bin/bash
```

* The very first line of every script
* <span style="color:green"><b>Purpose:</b></span> Tells the Linux kernel which interpreter (executable) to use

#### 🔸 Bash vs Sh

* Always use <span style="color:blue"><b>/bin/bash</b></span>
* `/bin/sh` may point to `dash` on modern OS (Ubuntu), which can break scripts

### 🟡 Comments

* Lines starting with `#` are ignored by the computer
* Used to explain your code to humans

---

## 🔵 3. The Editor: Vim vs Touch

You learned two ways to create files, each with a specific purpose:

### 🟡 `touch filename`

* Creates an empty file without opening it
* <span style="color:green"><b>Use Case:</b></span> Automation (e.g., a script creating 100 log files instantly)

### 🟡 `vi` / `vim filename`

* Creates the file and opens it immediately for editing
* <span style="color:green"><b>Use Case:</b></span> Writing code manually

### 🟠 Quick Vim Guide

1. Open file:

   ```bash
   vim script.sh
   ```
2. Insert Mode:

   ```text
   i
   ```
3. Exit Insert Mode:

   ```text
   Esc
   ```
4. Save & Exit:

   ```text
   :wq
   ```

---

## 🔵 4. Permissions (Crucial Security Concept)

In Linux, creating a script isn't enough; you must grant permission to execute it.

### 🔴 The Error

```text
Permission denied
```

Occurs when running:

```bash
./script.sh
```

### 🟡 The Command: `chmod`

* `chmod` = Change Mode

### 🟠 The Magic Numbers (4-2-1 Rule)

* <span style="color:blue"><b>4</b></span> = Read
* <span style="color:blue"><b>2</b></span> = Write
* <span style="color:blue"><b>1</b></span> = Execute
* <span style="color:green"><b>7</b></span> = Full Permissions (4+2+1)

### 🟢 Common Command

```bash
chmod 777 script.sh
```

* Gives read, write, and execute permissions to:

  * User
  * Group
  * Others

---

## 🔵 5. Real-World DevOps Use Case

### 🟣 Scenario

* A DevOps engineer manages **10,000 Virtual Machines** at Amazon

### 🔴 Problem

* He cannot manually check the CPU and RAM health of 10,000 servers every day

### 🟢 Solution

* He writes a shell script that:

  * Logs into every machine automatically
  * Uses system commands:

    * `nproc` → CPU count
    * `free` → Memory usage
    * `top` → Process monitoring
  * Sends an email alert **only if a server is failing**

---

## ✅ Quick Revision Checklist

* <span style="color:blue"><b>Shebang:</b></span> `#!/bin/bash` (Always the first line)
* <span style="color:blue"><b>Execution:</b></span> Run scripts using `./script.sh`
* <span style="color:red"><b>Permission Issue:</b></span> Use `chmod 777 <filename>`
* <span style="color:green"><b>Key Health Commands:</b></span>

  * `top` → Real-time process monitoring
  * `nproc` → Check number of CPUs
  * `free` → Check available RAM



=================================================================================================

# 🧠 Part 2: Advanced Shell Scripting

---

Here is a summary of **Part 2: Advanced Shell Scripting**, based on the new source provided.  
This session takes the basic **"Node Health"** script you started previously and transforms it into a **production-grade tool** using best practices, automation logic, and advanced filtering.

---

## 1️⃣ Scripting Best Practices (The "Set" Commands)

In a professional environment, scripts need to be robust. You shouldn't just list commands; you must handle errors and metadata.

### 📝 Metadata
Always include comments at the top describing the **Author, Date, Version, and Purpose of the script**.  
This helps your team understand what the file does without reading the code,.

### 🐞 Debug Mode (`set -x`)
Instead of writing echo statements everywhere to track progress, use `set -x`.  
This prints every command to the terminal before executing it, making debugging easy,.

### ❌ Error Handling (`set -e`)
By default, a script continues running even if a line fails.

- **The Fix:** `set -e` exits the script immediately if any command returns an error.
- **The Catch:** `set -e` does not catch errors if they happen inside a pipe (e.g., `command1 | command2`).  
  If command1 fails but command2 works, the script keeps going.
- **The Ultimate Fix:** Use `set -o pipefail`.  
  This ensures the script fails if any part of a pipe fails.

### 🔁 Revision Tip
A production script should often start with:


set -x # Debug mode
set -e # Exit on error
set -o pipefail # Exit on pipe failure


---

## 2️⃣ Advanced Text Processing (awk & grep)

The session moved beyond simple commands to extracting specific data (like a Process ID) from a mess of text.

### 📌 The Scenario
Your manager asks you to find the **Process ID (PID)** of all **"Amazon"** applications running on a server.

### 🔍 Step-by-Step
- **Step 1: List Processes**  
  `ps -ef` lists all running processes.

- **Step 2: Filter Rows (grep)**  
  `ps -ef | grep amazon` removes all lines except those containing "amazon".

- **Step 3: Filter Columns (awk)**  
  grep gives you the whole line, but you only want the PID (usually the 2nd column).

  - **The Command:**  
    `awk -F" " '{print $2}'`  
    This tells Linux to treat spaces as separators and print only the second column.

### 🎤 Interview Question
**Why does `date | echo "today"` not print the date?**

**Answer:**  
The pipe (`|`) sends output to stdin (Standard Input).  
The date command sends output, but echo does not read from stdin; it only prints arguments passed to it directly,.

---

## 3️⃣ Networking & Logs (curl vs. wget)

DevOps engineers often need to fetch log files from external storage (like S3 or GitHub) to analyze errors.

| Command | Function | Best Use Case |
|-------|---------|---------------|
| `curl` | Retrieves information and prints it to the screen (Standard Output). | When you want to check an API response or pipe the log data directly into grep without saving the file,. |
| `wget` | Downloads the file and saves it to your hard disk. | When you need to keep a copy of the file locally,. |

---

## 4️⃣ Finding Files

If you know a file exists (e.g., `pam.d`) but don't know where it is, use the `find` command.

- **Syntax:**  
  `find / -name pam.d`
- **Breakdown:**  
  Look inside root (`/`) for a file with the name `pam.d`.
- **Note:**  
  You often need sudo to search the entire system to avoid "Permission Denied" errors,.

---

## 5️⃣ Control Structures (if & for)

You learned how to add logic to your scripts.

### 🔀 If-Else
Used for decision making.

- **Syntax:**  
  `if [ $a -gt $b ]; then ... else ... fi`
- **Note:**  
  In shell scripting, you close an if block by spelling it backwards: `fi`.

### 🔁 For Loops
Used for iteration (e.g., creating 100 files).

- **Syntax:**  
  `for i in {1..100}; do echo $i; done`

---

## 6️⃣ Traps & Signals (Interview Topic)

What happens if a user presses **Ctrl+C** while your script is writing to a critical database?  
It stops halfway, leaving corrupted data.

### 🚨 Signals
- Ctrl+C sends a **SIGINT (Signal Interrupt)** to the OS.

### 🪤 The Trap Command
You can "catch" this signal and run a cleanup function instead of just stopping.

- **Example:**  
  `trap "rm -rf *" SIGINT`  

This tells the script:  
"If someone presses Ctrl+C, delete all the temporary files we created so we don't leave a mess".

---

## ✅ Quick Revision Checklist

- **Production Safety:** Always use `set -e` and `set -o pipefail`.
- **Parsing:** `grep` filters rows; `awk` filters columns.
- **Networking:** `curl` for viewing/APIs; `wget` for downloading files.
- **Logic:** `fi` closes an if statement.
- **Trap:** Used to handle interruptions (like Ctrl+C) gracefully.

============================================================================================


# ☁️ AWS Resource Tracker using Shell Scripting

---

This project creates an AWS Resource Tracker using Shell Scripting to automatically report usage on the AWS cloud. This ensures cost-effectiveness by identifying unused resources.

Here is a summary of the project logic and steps, organized so we can easily revise or expand on specific sections.

---

## 1️⃣ The Business Problem

Organizations move to the cloud to reduce maintenance overhead and manage costs (pay-as-you-go). However, developers often create resources (like EC2 instances or EBS volumes) and leave them running unused, leading to wasted money. A DevOps engineer must track these resources to maintain cost efficiency.

---

## 2️⃣ The Technical Solution

Instead of manually checking the AWS console or writing complex Python scripts, we can use a Shell Script combined with the AWS CLI to generate a resource usage report.

### 🎯 Goals of the Script:
- Track S3 Buckets
- Track EC2 Instances
- Track Lambda Functions
- Track IAM Users

---

## 3️⃣ Prerequisites

Before writing the script, ensure the environment is set up:

- AWS CLI Installed: The command-line interface must be present on the machine.
- AWS Configured: Run `aws configure` to authenticate using your Access Key, Secret Access Key, and default region.

---

## 4️⃣ Drafting the Script

The script (named `aws_resource_tracker.sh`) follows this structure:

- **Shebang:** Start with `#!/bin/bash`.
- **Metadata:** Include comments for the Author, Date, and Version (e.g., v1) to help others maintain the code.
- **Resource Commands:**
  - S3: `aws s3 ls` (Lists all buckets).
  - EC2: `aws ec2 describe-instances` (Lists instance details).
  - Lambda: `aws lambda list-functions`.
  - IAM: `aws iam list-users`.

---

## 5️⃣ Script Refinements (Best Practices)

To make the script production-ready and user-friendly, the following enhancements are added:

- **Echo Statements:** Add print statements (e.g., `echo "Print list of S3 buckets"`) to separate outputs in the report.
- **Debug Mode:** Add `set -x` at the start of the script. This prints the command being executed alongside its output, which helps in debugging.
- **JSON Parsing (JQ):** The default output for EC2 commands is a massive JSON block. Use the `jq` command (a JSON parser) to filter results.
  - Example: Pipe the output to `jq` to extract only the Instance IDs for a cleaner report.
- **Output Redirection:** Instead of just printing to the terminal, redirect the output to a file (e.g., `resource_tracker`) so the report is saved permanently.

---

## 6️⃣ Automation (Cron Jobs)

Manually running the script every day is inefficient and prone to human error (e.g., forgetting to run it at 6 PM).

- **The Solution:** Integrate the script with a Cron Job (a Linux scheduler).
- **How it works:** You schedule the script to run automatically at a specific time (e.g., 7 PM daily), generating the report without manual intervention.

---

===========================================================================

# 🧑‍💻 Day-9 | Git and GitHub

---

Based on the provided transcript for **"Day-9 | Git and GitHub,"** here is a summary of the concepts and workflow.  
This covers the transition from basic Version Control Systems (VCS) to using Git and GitHub effectively.

---

## 1️⃣ The Business Problem: Why Version Control?

Before Git, developers faced two major challenges when building applications (like a calculator app) in a team:

- **Sharing Code:** If "Dev 1" works on addition and "Dev 2" works on subtraction, they need a way to merge their files. Sharing code via email is impossible when managing thousands of files.
- **Versioning:** Requirements change constantly (e.g., changing a feature from "adding 2 numbers" to "adding 4 numbers," then back to 2). Developers need a history of these versions to revert code instantly without rewriting it.

---

## 2️⃣ The Solution: Distributed vs. Centralized

- **Centralized (Old Way - SVN/CVS):** Developers relied on a single server. If that central server went down, no one could communicate or save work. It had a single point of failure.
- **Distributed (New Way - Git):** Git creates a distributed system. Every developer has a full copy of the code (a "Fork") on their local machine. If the main server goes down, work continues because everyone has their own backup.

---

## 3️⃣ Git vs. GitHub

It is crucial to understand the difference between the tool and the platform:

- **Git:** An open-source, command-line tool. You can install it on your local laptop or an EC2 instance to track files.
- **GitHub (or GitLab/Bitbucket):** A "wrapper" built on top of Git. It provides a user interface, project management tools, and a central place to host the code so teams can collaborate easily.

---

## 4️⃣ The Git Lifecycle (Workflow)

The script outlines a specific lifecycle for tracking a file (e.g., `calculator.sh`):

- **Initialize:** Run `git init` to create a hidden `.git` folder that tracks the project.
- **Check Status:** Run `git status` to see if files are "untracked" or "modified".
- **Stage (Add):** Run `git add [filename]` to tell Git, "I want you to track this specific file".
- **Commit:** Run `git commit -m "message"` to save a snapshot of the file as a new version (e.g., "Version 1").
- **Verify:**
  - Use `git diff` to see exactly what lines changed in the code.
  - Use `git log` to see a list of all commit IDs and authors.
- **Revert:** If a mistake is made, use `git reset` with a specific commit ID from the log to go back to a previous version.

---

## 5️⃣ Sharing the Code (GitHub)

To move code from a local laptop to a shared team environment:

1. Create an account on GitHub.com.
2. Select "New Repository" and choose visibility (Public/Private).
3. Push the local Git repository to this remote GitHub repository so others can access or fork the code.

---

--------------------------------------------------------------------------------

=========================================================================================

# 🌳 Day-10 | Git Branching Strategy

---

Based on the transcript for **"Day-10 | Git Branching Strategy,"** here is a summary of the standard workflow used by organizations to manage code releases.  
This builds upon the Git commands we covered in Day-9.

---

## 1️⃣ The Business Problem

The primary goal of any IT organization (like Amazon or Flipkart) is to deliver new features to customers frequently (e.g., every 15 days or 2 months),. However, developers cannot push experimental code directly to the main product because it might break existing functionality. They need a strategy to separate **"stable code"** from **"work in progress"**.

---

## 2️⃣ The Solution: Git Branching Strategy

A Branching Strategy is a set of rules that dictates how a team uses Git to manage development, testing, and releases. It ensures that new changes (like adding "Bikes" to an "Uber" app) do not disrupt the current users (who are using "Uber Cabs").

---

## 3️⃣ The Four Key Branches

The source outlines a standard strategy involving four specific types of branches:

### 🔹 Master (or Main) Branch
- **Purpose:** This is the source of truth. It must always be up-to-date and hold the latest merged code,.
- **Rule:** Active development happens elsewhere, but everything eventually merges back here.

### 🔹 Feature Branch
- **Purpose:** Used for developing new functionality (e.g., feature-bikes, feature-percentage).
- **Workflow:** Created from Master. Developers work here until they are confident. Once tested, it is merged back into Master and the branch is deleted,,.

### 🔹 Release Branch
- **Purpose:** Used to prepare a specific version for the customer (e.g., release-v1.27).
- **Workflow:** Created from Master when features are "frozen." No new features are added here; only final testing and stability checks occur before shipping to the customer,.

### 🔹 Hotfix (Bugfix) Branch
- **Purpose:** Used for emergency fixes when a critical bug is found in production (live environment).
- **The "Tricky" Rule:** Unlike other branches, a Hotfix must be merged into both the Release branch (to fix the live app) AND the Master branch (to ensure future versions have the fix),.

---

## 4️⃣ Real-World Example: Kubernetes

The source highlights Kubernetes as a prime example of this strategy.

- It is an open-source project with over 3,300 contributors.
- They use a Master branch for active development.
- When a release window approaches (e.g., January), they cut a release-1.27 branch to lock in the code for testing.

---

## 5️⃣ Summary of the Workflow

1. **Start:** Everyone bases their work off Master.
2. **Develop:** Create a Feature branch to write code. Merge to Master when done.
3. **Ship:** Create a Release branch to test and deploy to customers.
4. **Repair:** Create a Hotfix branch if the customer reports a critical error.

---

================================================================================

# 🧠 Day-11 | Git Interview Q&A and Commands

---

Based on the transcript for **"Day-11 | Git Interview Q&A and Commands,"** here is a summary of the advanced workflows and interview scenarios.  
This session moves beyond the basics to cover how DevOps engineers handle remote repositories, authentication, and complex merge strategies.

---

## 1️⃣ Repository Initialization & Tracking

There are two ways to start a project, leading to a common interview question about the `.git` folder.

- **The Command:** `git init` initializes a repository locally.
- **What happens internally?** It creates a hidden folder called `.git`.
- **Why it matters:** This folder is responsible for tracking changes, logging history, and managing configuration. If deleted, you lose the version history.

### The Workflow:
1. **Modify:** Create or edit a file (e.g., `calculator.sh`).
2. **Stage:** Run `git add [filename]` to tell Git to track the file.
3. **Commit:** Run `git commit -m "message"` to save the version.

---

## 2️⃣ Connecting to Remote (GitHub)

Code sitting on a local laptop is risky and isolated. It must be pushed to a remote repository (like GitHub).

- **The Workflow:** The standard cycle is `git add -> git commit -> git push`.
- **The "Remote" Error:** If you run `git push` on a locally created repo, it will fail because Git doesn't know where to send the code.

### The Fix:
- **Option A (Existing Repo):** Use `git clone [url]` to download a repository. This automatically configures the remote link.
- **Option B (New Local Repo):** Use `git remote add [url]` to manually link your local folder to a GitHub repository.

---

## 3️⃣ Authentication: HTTPS vs. SSH

To push code securely, you must authenticate.

- **HTTPS:** Uses a username and password (or token).
- **SSH (Secure Shell):** Uses a public/private key pair.
  - **Setup:** You generate keys locally (using `ssh-keygen`), keep the private key on your machine, and paste the public key into your GitHub settings. This allows you to push code without entering a password every time.

---

## 4️⃣ Critical Interview Question: Clone vs. Fork

This is a frequent interview topic regarding how to get code from others.

- **Git Clone:** Downloads a specific repository to your local machine so you can work on it.
- **Git Fork:** Creates a server-side copy of a repository under your own GitHub account.
  - **Use Case:** If you want to contribute to a huge open-source project (like ArgoCD), you "Fork" it to your account first, then "Clone" your version. This allows independent collaboration without altering the original source directly.

---

## 5️⃣ Merging Strategies: Merge vs. Rebase

When combining code from a feature branch (e.g., `feature-division`) into the main branch, there are different methods depending on the desired history.

### 🔹 Cherry Pick:
- Used when you only want one specific commit from another branch, not the whole thing.
- **Command:** `git cherry-pick [commit-id]`.

### 🔹 Git Merge:
- Combines branches but adds the new changes to the top of the history.
- **Result:** The history is non-linear. It can be harder to track the order of changes in large projects.

### 🔹 Git Rebase:
- Moves your feature branch changes to apply after the latest main branch commits.
- **Result:** Creates a linear history. This is preferred in large teams (like those creating a new Amazon feature) because it makes the log easier to read and track.

---

==========================================================================

# ☁️ Day-12 | Deploy and expose your First App to AWS

---

Based on the transcript for **"Day 12 | Deploy and expose your First App to AWS,"** here is a summary of the workflow.  
This session, featuring guest Kunal Verma, moves from local development to a live production deployment on the cloud.

---

## 1️⃣ The Goal: From Localhost to the World

Developing an application on a personal laptop (Localhost) is only the first step. To let users access it from anywhere in the world, the application must be moved to a remote server. In this project, the goal is to deploy a Node.js application (a Stripe payment demo) onto an AWS EC2 instance.

---

## 2️⃣ Phase 1: Local Setup (Sanity Check)

Before moving to the cloud, verify the code works locally.

- **Clone Repository:** Download the project code from GitHub.
- **Environment Variables (.env):**
  - **The Problem:** The app uses secret keys (Stripe API keys) that cannot be uploaded to public GitHub for security reasons.
  - **The Fix:** Create a hidden file named `.env` locally and paste the secrets there. The app reads from this file.
- **Run:** Execute `npm install` (to get dependencies) and `npm run start`. Verify it runs on `localhost:3000`.

---

## 3️⃣ Phase 2: AWS Account Setup (IAM)

It is a security risk to use the AWS "Root" (owner) account for daily tasks.

- **Create User:** Go to IAM (Identity and Access Management) and create a new user (e.g., "Kunal").
- **Permissions:** Attach policies to this user (e.g., AdministratorAccess or EC2FullAccess) so they have permission to create servers.
- **Login:** Sign out of Root and sign in as the new IAM user.

---

## 4️⃣ Phase 3: Provisioning the Server (EC2)

- **Launch Instance:** Select "Ubuntu" as the Operating System and "t2.micro" (Free Tier) as the instance type.
- **Key Pair:** Create and download a `.pem` file (e.g., `demo.pem`). Crucial: This file is the "key" to the server; if lost, you cannot log in.
- **Permissions:** On your local machine, run `chmod 400 demo.pem` to secure the key file, otherwise AWS will reject the connection.
- **SSH Connect:** Log in to the remote server using the terminal:  
  `ssh -i demo.pem ubuntu@[public-ip-address]`.

---

## 5️⃣ Phase 4: Server Configuration

The new EC2 instance is like a brand-new, empty laptop. It needs tools to run the app.

- **Update:** Run `sudo apt update` to refresh the package manager.
- **Install Tools:** Install Node.js and NPM (Node Package Manager) using standard Linux commands (referenced from DigitalOcean docs in the video).
- **Clone & Config:**
  1. Clone the GitHub repo into the EC2 instance.
  2. Repeat the Secret Step: Since the `.env` file wasn't on GitHub, you must create it again inside the server using a text editor like `vim` or `nano`.
- **Start App:** Run `npm run start`. The app is now running inside the cloud server.

---

## 6️⃣ Phase 5: The "Magic" Step (Security Groups)

Even though the app is running, if you type the EC2 Public IP into a browser, it will fail.

- **The Reason:** AWS Security Groups (Firewalls) block all traffic by default, except for SSH (port 22).
- **The Fix:**
  1. Go to the EC2 Security Groups tab.
  2. Edit Inbound Rules.
  3. Add a Custom TCP Rule: Port 3000, Source Anywhere (0.0.0.0/0).
- **Result:** The application is now accessible to anyone on the internet via the Public IP.

---

=================================================================================

# ☁️ Day-13 | Top 15 AWS Services

---

Based on the transcript for **"Day 13 | Top 15 AWS Services,"** here is a summary of the essential services a DevOps engineer must master.  
This session transitions from learning individual tools (like EC2) to understanding the broader AWS ecosystem needed to build secure, automated pipelines.

---

## 1️⃣ Core Infrastructure (Compute & Storage)

These are the fundamental building blocks for hosting applications and data.

- **EC2 (Elastic Compute Cloud):** Virtual servers used to deploy applications (as we did in Day 12).
- **VPC (Virtual Private Cloud):** The network layer. You must understand subnets and inbound/outbound rules to secure your EC2 instances so they aren't exposed dangerously to the public,.
- **EBS (Elastic Block Store):** The "hard drive" for your EC2. In production, you often need to attach persistent storage volumes to instances to save database files or logs,.
- **S3 (Simple Storage Service):** Object storage for files like images, backups, or large datasets (e.g., Excel sheets). AWS now encrypts S3 data by default,.

---

## 2️⃣ Security & Governance ( The Guardrails)

DevOps engineers act as "security guards" to ensure the team doesn't accidentally expose data or break compliance rules.

- **IAM (Identity and Access Management):** The system for creating users and assigning permissions. You use this to ensure a developer has "Read Only" access while an admin has "Admin" access,.
- **KMS (Key Management Service):** Manages security keys and secrets. It is used to encrypt sensitive data stored in S3 buckets or EBS volumes.
- **AWS Config:** A compliance tool that monitors your setup. It checks if resources follow rules (e.g., "All S3 buckets must be encrypted").
- **CloudTrail:** An auditing service that records API calls. It answers "Who did what?" (e.g., "Who deleted this database?") by logging activity for up to 6 months,.

---

## 3️⃣ Monitoring & Automation

Automation is the heart of DevOps. These tools help detect issues and fix them without human intervention.

- **CloudWatch:** The monitoring service. It tracks metrics (CPU usage) and logs. It alerts you if a server crashes or usage spikes,.
- **Lambda:** Serverless compute. It runs code in response to events without you managing a server.
  - **Real-World Example:** You can set up a system where CloudWatch detects an unencrypted EBS volume and triggers a Lambda function to automatically encrypt it or send a warning email,.

---

## 4️⃣ CI/CD (Build & Deploy)

AWS provides its own set of tools to replace external tools like Jenkins, though they are best suited for teams strictly using AWS (not multi-cloud),.

- **CodeBuild:** Compiles source code and runs tests.
- **CodeDeploy:** Automates the deployment of code to EC2 instances.
- **CodePipeline:** Orchestrates the workflow, similar to a Jenkins Pipeline.

---

## 5️⃣ Containers & Scaling

As teams move away from simple virtual machines to microservices, these tools manage containers.

- **EKS (Elastic Kubernetes Service):** Managed Kubernetes. If you already know Kubernetes, EKS is simply AWS managing the control plane for you.
- **ECS (Elastic Container Service):** AWS's proprietary alternative to Kubernetes for running Docker containers.
- **ElasticSearch (ELK Stack):** Used for centralized logging. When running 1,000 microservices, you use this to search through logs to find which specific service is throwing errors.

---

## 6️⃣ Cost Management

- **Billing & Costing:** A DevOps engineer must monitor spending to see if the organization is overspending on resources like EC2 or S3.

---

==========================================================================================================

# ⚙️ Day-14 | Configuration Management with Ansible

---

Based on the transcript for **"Day-14 | Configuration Management with Ansible,"** here is a summary of the theoretical concepts required before starting the practical projects.  
This session focuses on why we need automation tools and why Ansible became the industry standard.

---

## 1️⃣ The Business Problem: Scaling Infrastructure

In the past, System Administrators managed physical data centers with a manageable number of servers (e.g., 50 Linux, 50 Windows). They manually logged into each server to run updates, install patches, or set up databases.

- **The Scripting Trap:** Admins tried to automate this using Shell Scripts (Linux) or PowerShell (Windows). However, as companies moved to the Cloud and Microservices, the number of servers exploded (10x increase).
- **The Complexity:** Writing scripts that loop through thousands of servers while handling different OS versions (Ubuntu vs. CentOS) became impossible to maintain.

---

## 2️⃣ The Solution: Configuration Management

Configuration Management tools were invented to automate the setup and maintenance of servers. While tools like Puppet, Chef, and Salt exist, Ansible has emerged as the winner in the market.

**What does it do?**  
Instead of manually logging into 100 servers, you write the configuration once (e.g., "Install Git on all servers") and the tool ensures it happens everywhere.

---

## 3️⃣ Ansible vs. Puppet (The Core Architecture)

Understanding the architecture differences is the most common interview topic for this section.

| Feature | Puppet (Old Way) | Ansible (Modern Way) |
|------|------------------|----------------------|
| Mechanism | Pull Model: Agents on the servers "ask" the master for updates. | Push Model: You write code on your laptop and "push" it to the servers. |
| Architecture | Master-Slave: You must install "agent" software on every server and configure certificates. | Agentless: No installation required on the target servers. It uses existing connections. |
| Language | Puppet Language: Requires learning a specific, complex syntax. | YAML: Uses simple, human-readable YAML manifests. |
| Ease of Use | High learning curve. | Very easy to start (Python-based). |

---

## 4️⃣ How Ansible Connects (Protocols)

Since Ansible is agentless, it relies on standard remote connection protocols to talk to your servers:

- **Linux Servers:** Uses SSH (Secure Shell).
- **Windows Servers:** Uses WinRM (Windows Remote Management).
- **Cloud Agnostic:** Ansible does not care if the server is on AWS, Azure, or GCP. It only needs a public IP address (or DNS) and a valid connection (SSH/WinRM).

---

## 5️⃣ Ansible Galaxy

Ansible allows you to write custom modules using Python.

- **Concept:** If you write code to configure a specific Load Balancer, you can upload it to Ansible Galaxy.
- **Benefit:** This is a community hub (similar to Docker Hub) where DevOps engineers share modules. You can download pre-written code from others to save time.

---

## 6️⃣ Limitations of Ansible

While powerful, Ansible is not perfect. The source highlights three main disadvantages:

1. **Windows Support:** While improved by Red Hat, configuring Windows is still trickier than Linux.
2. **Debugging:** Error logs can be difficult to read and understand compared to other tools.
3. **Performance:** Because it uses a push mechanism over SSH, it can face speed issues when handling massive parallel executions (e.g., 10,000 servers at once).

---

## 7️⃣ Summary of Interview Questions

The session concluded with six common interview questions:

1. What language is Ansible built on? Python.
2. Does it support Windows? Yes, via WinRM.
3. Puppet vs. Ansible? (Focus on Push vs. Pull and Agentless architecture).
4. Push or Pull? Ansible is Push.
5. File Format? YAML.
6. Does it support all clouds? Yes, it connects via IP/SSH regardless of the provider

---

============================================================================


# ⚙️ Day-15 | Ansible Zero to Hero

---

Based on the transcript for **"Day-15 | Ansible Zero to Hero,"** here is a summary of the practical session.  
This moves from the theory covered in Day 14 to actually installing and running Ansible commands.

---

## 1️⃣ The Setup: Architecture & Installation

To practice, you need two servers: one **Ansible Master** (where you write code) and one **Target Node** (where code is executed).

- **Installation:** On the Master (Linux/Ubuntu), run `sudo apt update` followed by `sudo apt install ansible`.
- **Verification:** Run `ansible --version` to confirm it is installed correctly.

---

## 2️⃣ The Bridge: Passwordless Authentication

Ansible needs to communicate with the Target Node without stopping to ask for a password every time.

- **Generate Key:** On the Master, run `ssh-keygen` to create a Public and Private key pair.
- **The Handshake:** Copy the content of the Public Key (`id_rsa.pub`) from the Master and paste it into the `~/.ssh/authorized_keys` file on the Target Node.
- **Test:** Run `ssh [Target-IP]` from the Master. If it logs in without a password, Ansible is ready to work.

---

## 3️⃣ Quick Automation: Ad-Hoc Commands

You don't always need to write a script. For simple, one-time tasks (like checking disk space or creating a file), use **Ad-Hoc commands**.

- **The Command:**  
  `ansible -i inventory -m [module] -a [arguments]`
- **Inventory File:** A simple text file containing the IP addresses of your target servers. You can group them (e.g., `[webservers]` vs `[dbservers]`) to run commands on specific groups.
- **Example:**  
  To create a file named `"test"` on all servers:  
  `ansible -i inventory all -m shell -a "touch test"`

---

## 4️⃣ Robust Automation: Playbooks

For complex, multi-step tasks (like installing and starting a server), you write a **Playbook** using YAML.

### Structure of the Example Playbook:

- **Header:** Starts with `---` (YAML indicator) and defines the hosts (targets).
- **Permissions:** Uses `become: yes` to execute commands as the "Root" user (sudo).
- **Tasks:** The actual steps to perform.
  1. **Install:** Uses the `apt` module to install Nginx (`state: present`).
  2. **Start:** Uses the `service` module to ensure Nginx is running (`state: started`).
- **Execution:**  
  Run the playbook using:  
  `ansible-playbook -i inventory first-playbook.yaml`
- **Debugging:**  
  If something fails, add `-v` (verbose) to the command to see exactly what Ansible is doing in the background.

---

## 5️⃣ Scaling Up: Ansible Roles

Writing 50 tasks in one file becomes messy. **Ansible Roles** allow you to organize complex projects (like setting up a Kubernetes cluster) into a structured folder system.

- **Creation:**  
  Run `ansible-galaxy role init [role_name]` to automatically generate the folder structure.
- **The Structure:**
  - **Tasks:** Contains the main steps (`main.yaml`).
  - **Vars/Defaults:** Stores variables (like port numbers or package names).
  - **Files/Templates:** Stores static files (like SSL certificates) or dynamic templates (HTML files).
  - **Handlers:** triggers actions like "Restart Service" only if a change occurs.

---
# ⚙️ Day-15 | Ansible Zero to Hero

---

Based on the transcript for **"Day-15 | Ansible Zero to Hero,"** here is a summary of the practical session.  
This moves from the theory covered in Day 14 to actually installing and running Ansible commands.

---

## 1️⃣ The Setup: Architecture & Installation

To practice, you need two servers: one **Ansible Master** (where you write code) and one **Target Node** (where code is executed).

- **Installation:** On the Master (Linux/Ubuntu), run `sudo apt update` followed by `sudo apt install ansible`.
- **Verification:** Run `ansible --version` to confirm it is installed correctly.

---

## 2️⃣ The Bridge: Passwordless Authentication

Ansible needs to communicate with the Target Node without stopping to ask for a password every time.

- **Generate Key:** On the Master, run `ssh-keygen` to create a Public and Private key pair.
- **The Handshake:** Copy the content of the Public Key (`id_rsa.pub`) from the Master and paste it into the `~/.ssh/authorized_keys` file on the Target Node.
- **Test:** Run `ssh [Target-IP]` from the Master. If it logs in without a password, Ansible is ready to work.

---

## 3️⃣ Quick Automation: Ad-Hoc Commands

You don't always need to write a script. For simple, one-time tasks (like checking disk space or creating a file), use **Ad-Hoc commands**.

- **The Command:**  
  `ansible -i inventory -m [module] -a [arguments]`
- **Inventory File:** A simple text file containing the IP addresses of your target servers. You can group them (e.g., `[webservers]` vs `[dbservers]`) to run commands on specific groups.
- **Example:**  
  To create a file named `"test"` on all servers:  
  `ansible -i inventory all -m shell -a "touch test"`

---

## 4️⃣ Robust Automation: Playbooks

For complex, multi-step tasks (like installing and starting a server), you write a **Playbook** using YAML.

### Structure of the Example Playbook:

- **Header:** Starts with `---` (YAML indicator) and defines the hosts (targets).
- **Permissions:** Uses `become: yes` to execute commands as the "Root" user (sudo).
- **Tasks:** The actual steps to perform.
  1. **Install:** Uses the `apt` module to install Nginx (`state: present`).
  2. **Start:** Uses the `service` module to ensure Nginx is running (`state: started`).
- **Execution:**  
  Run the playbook using:  
  `ansible-playbook -i inventory first-playbook.yaml`
- **Debugging:**  
  If something fails, add `-v` (verbose) to the command to see exactly what Ansible is doing in the background.

```bash
---
- name: Install and start Nginx on Ubuntu
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Ensure Nginx is started and enabled
      service:
        name: nginx
        state: started
        enabled: yes
```
---

## 5️⃣ Scaling Up: Ansible Roles

Writing 50 tasks in one file becomes messy. **Ansible Roles** allow you to organize complex projects (like setting up a Kubernetes cluster) into a structured folder system.

- **Creation:**  
  Run `ansible-galaxy role init [role_name]` to automatically generate the folder structure.
- **The Structure:**
  - **Tasks:** Contains the main steps (`main.yaml`).
  - **Vars/Defaults:** Stores variables (like port numbers or package names).
  - **Files/Templates:** Stores static files (like SSL certificates) or dynamic templates (HTML files).
  - **Handlers:** triggers actions like "Restart Service" only if a change occurs.

---




