# 🟢 Day-1 | Fundamentals of Linux | Run Linux using Docker

## 🎯 Session Overview
This session kicks off a **"Linux Zero to Hero"** series designed to teach **Linux fundamentals** without requiring a **paid cloud subscription**.

---

## 🟢 1. What is an Operating System (OS)?

- 🔴 **The Problem:**  
  Hardware (CPU, Memory) is complex. Users and Applications (like YouTube or Python scripts) cannot talk directly to the hardware to request resources.

- 🟢 **The Solution:**  
  An **Operating System** is the intermediate software layer (**bridge**) between the **User/App** and the **Hardware**.

- 🔧 **Responsibilities:**  
  - Process Management  
  - Memory Management  
  - Device Management  
  - Network Management  

---

## 🟢 2. Linux Architecture

The Linux OS is composed of the following layers:

1. 🖥️ **Hardware:** The physical components.  
2. ⚙️ **Kernel:** The **"Engine"**. It performs the heavy lifting (managing memory/CPU).  
3. 🧩 **System Utilities / Libraries:** Basic functions provided out of the box.  
4. 💻 **Shell (CLI):**  
   - The interface that allows users to interact with the kernel.  
   - Without the shell, users cannot interact with the hardware.

---

## 🟢 3. Linux Distributions (Distros)

- 📦 **The Concept:**  
  Because the **Linux Kernel is Open Source**, anyone can download the code, modify it, and wrap it into a product.

- 🏭 **How they differ:**  
  Companies like **Canonical (Ubuntu)** or **Red Hat** take the raw kernel, add their own features/wrappers, and ship it as a **Distribution**.

- 🔁 **Compatibility:**  
  A script written for **Ubuntu** usually works on **Red Hat** because the underlying libraries and kernel are the same.

- 🌍 **Market Share:**  
  Linux powers about **90% of production workloads** because it is **open-source, free, and highly secure**.

---

## 🟢 4. How to Setup Linux (Free Methods)

The instructor strongly advises **against using Cloud EC2 instances for learning**, as they incur costs.  
Instead, use one of the following **local methods**:

---

### 🟢 Method A: WSL (Windows Subsystem for Linux) – *Recommended*

- ❓ **What is it?**  
  It runs a Linux environment directly inside Windows without a heavy Virtual Machine.

- 🛠️ **How to install:**
  1. Open **PowerShell / Command Prompt**  
  2. Run:  
     ```bash
     wsl --install
     ```
  3. Restart your computer

---

### 🟢 Method B: Docker Container

- ❓ **What is it?**  
  If you cannot use WSL (or use a Mac), you can run a **Linux Container** that acts like a full OS.

- 🧾 **The Command:**  
  The session provides a specific command to bind your local storage to the container so you don't lose files.

  > ⚠️ **Note:**  
  You must ensure the local folder (e.g., `/tmp/ubuntu-data`) exists before running this.

---

## 🟢 5. Package Managers

- 🛒 **Definition:**  
  Just as phones have **App Stores**, Linux has **Package Managers** to install, update, and remove software (dependencies like Python or Java).

- 🔧 **The Tool:**  
  On Ubuntu, the package manager is **apt**.

- 📌 **Key Commands:**
  - `apt list` – Lists installed packages  
  - `apt update` – Refreshes the list of available software from the central repository  
  - `apt install python3` – Downloads and installs Python from a trusted source  

---

🟣 **End of Day-1 | Fundamentals of Linux**

====================================================

# 🟢 Day-2 | Linux Folder Structure Explained

## 🎯 Session Overview
This session moves from **setting up Linux (Day 1)** to understanding the **geography of the system: The Linux File System**.  
Think of this as learning **how Linux organizes everything internally**.

---

## 🟢 1. Decoding the Command Prompt

When you log in, you see a prompt like:

root@ubuntu:/#


This is **not random**. It gives you **three important pieces of information**:

### 🔵 User (`root`)
- Tells you **who you are logged in as**
- `root` = **Super Admin** (unrestricted access)
- In AWS/Cloud, you often see:
  - `ubuntu`
  - `ec2-user`
- These are **standard users**, not super admin

---

### 🔵 Host (`@ubuntu`)
- The **name of the machine** you are logged into

---

### 🔵 Location (`/` or `~`)
- `/` (**Slash**):
  - The **Root of the file system**
  - Top-level folder that holds **every other folder**
- `~` (**Tilde**):
  - Represents the **Home Directory** of the current user
  - Example:
    - User: `ubuntu`
    - `~` → `/home/ubuntu`

---

## 🟢 2. The Linux Directory Structure (The Hierarchy)

Unlike Windows (**C:, D:**), Linux uses **one single tree** starting from `/`.

The directories are grouped by **function**:

---

## 🅰️ A. The Binaries (Executable Commands)

❓ How does Linux know what `ls` or `mkdir` means?  
👉 These commands are **programs stored in specific folders**.

### 🔹 `/bin` (User Binaries)
- Common commands available to **all users**
- Examples:
  - `ls`
  - `date`
  - `cat`

---

### 🔹 `/sbin` (System Binaries)
- Administrative commands
- Used by **Root/Admin**
- Example:
  - `useradd` (create users)

---

### 🔹 `/usr` (User Programs)
- In modern Linux:
  - `/bin` and `/sbin` are often **links**
  - Actual binaries live in:
    - `/usr/bin`
    - `/usr/sbin`

---

## 🅱️ B. Configuration & Settings

### 🔹 `/etc` (Et Cetera)
- **Most critical folder** for admins
- Contains **System Configuration Files**

📱 **Analogy:**
- Phone → Settings menu  
- Windows → Control Panel  

📄 **Examples:**
- `/etc/passwd` → User information
- `/etc/hosts` → Network mappings

---

## 🅲 C. User Data & Home

### 🔹 `/home`
- Storage for **normal users**
- Example:
  - User: `abhi`
  - Home directory: `/home/abhi`

---

### 🔹 `/root`
- Home directory of the **Root user**
- Root is the **only user not stored inside `/home`**

---

## 🅳 D. Application & Data Management

### 🔹 `/opt` (Optional)
- Best practice location for **Third-Party Software**
- Examples:
  - Custom Java versions
  - Tomcat
  - Company-specific tools

---

### 🔹 `/var` (Variable)
- Stores files that **change frequently**
- Most important subfolder:
  - `/var/log` → **Logs**
- Example:
  - Web server logs live here

---

### 🔹 `/mnt` (Mount)
- Used to **temporarily attach** new hard disks or file systems

---

## 🅴 E. Temporary & System Files

### 🔹 `/tmp` (Temporary)
- Short-lived files
- System **automatically cleans** this folder
- Similar to a **"Recently Deleted"** area

---

### 🔹 `/lib`
- Shared libraries required by the **Kernel**
- Similar to **DLLs** in Windows
- Users rarely touch this folder

---

### 🔹 `/boot`
- Files required to **start (boot)** the system

---

## 🟢 3. How the Shell Finds Commands (The `$PATH`)

### ❓ The Question
If `ls` exists at:


/usr/bin/ls
```bash
Why can we just type:
```

ls
```bash
instead of the full path?

---

### ✅ The Answer: `$PATH`

- `$PATH` is an **environment variable**
- It contains a **list of directories**, such as:
  - `/bin`
  - `/sbin`
  - `/usr/bin`

### 🔁 How it works:
1. You type a command (e.g., `ls`)
2. Linux searches each directory in `$PATH`
3. If it finds the command → it executes it

---

### 🛑 Debug Tip
If you see:
```
command not found
```bash
It usually means:
- That program is **not present in any directory** listed in `$PATH`

---

🟣 **End of Day-2 | Linux Folder Structure Explained**

```

