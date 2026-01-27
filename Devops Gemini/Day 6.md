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



