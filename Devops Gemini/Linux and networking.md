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

=============================================================================

# 🟢 Day-3 | User Management | File Management | Vi Editor Shortcuts

## 🎯 Session Overview
This session moves from **understanding the folder structure (Day 2)** to **actively managing a Linux system** — creating users, securing files, connecting to servers, and editing configuration files.

---

## 🟢 1. User Management (Accountability & Security)

In a real company, **100 developers cannot share the root password**.  
If someone deletes a critical folder like `/sbin`, you must know **who did it**.  
This is why **individual user accounts and groups** are mandatory.

---

## 🔵 A. Creating & Deleting Users

### ➕ Create Users
- `useradd <name>`
  - **Quick command**
  - Does **not** create a home directory
  - Does **not** ask for user details
  - Best suited for **automation/scripts**

- `adduser <name>`
  - **Interactive command**
  - Asks for:
    - Full name
    - Password
  - Automatically creates the **home directory**
  - Best suited for **human use**

### ➖ Delete Users
- `userdel <name>`
  - Deletes the user account

---

## 🔵 B. Managing Passwords

- `passwd <name>`
  - Sets or changes a user’s password

### 🔐 The Shadow File
- Passwords are stored in:

/etc/shadow

- Stored in **encrypted (hashed) format**

🟡 **Interview Question:**  
> Can you decrypt a password from `/etc/shadow`?

🟢 **Answer:**  
No. It is a **one-way hash**.  
If a user forgets the password, you **cannot recover it** — you must **reset it**.

---

## 🔵 C. Group Management

When managing permissions for **hundreds of users**, you do not update them one by one — you use **groups**.

- `groupadd <name>`
- Creates a new group (e.g., `devops`)

- `usermod -aG <group> <user>`
- Adds a user to a group

- `cat /etc/group`
- Displays all groups created on the system

---

## 🟢 2. Connecting to Servers (SSH)

In real-world environments, you connect to **remote servers** (AWS EC2, on-prem servers) using **SSH (Secure Shell)**.

- **Command:**

ssh <username>@<ip_address>


### 🔧 Troubleshooting
If the server rejects your password:
- Password authentication may be disabled
- Check configuration:

/etc/ssh/sshd_config

- Cloud providers usually set:

PasswordAuthentication no

- They prefer **Key Pairs** instead of passwords

---

## 🟢 3. File Management Commands (CRUD)

Basic **Create, Read, Update, Delete** operations:

| 🔧 Action | 🖥️ Command | 📝 Note |
|--------|---------|------|
| Create Folder | `mkdir <name>` | Make directory |
| Create File | `touch <name>` | Creates empty file |
| Copy | `cp <source> <dest>` | Duplicate file |
| Move / Rename | `mv <source> <dest>` | Rename if same folder |
| Delete File | `rm <name>` | Remove file |
| Delete Folder | `rm -rf <name>` | ⚠️ Dangerous (force delete) |

---

## 🟢 4. The Vi / Vim Editor

Most Linux servers **do not have a GUI**.  
You must edit files using **command-line editors** like **Vim**.

---

## 🔵 Vim Modes

### 1️⃣ Normal Mode
- Default mode
- Used for **navigation**
- Cannot type text

### 2️⃣ Insert Mode
- Press `i`
- Used to **type text**

### 3️⃣ Command Mode
- Press `Esc` then `:`
- Used to **save or quit**

---

## 🔑 Important Vim Shortcuts

- **Save & Quit**

Esc → :wq!

- **Quit Without Saving**
Esc → :q!

- **Go to Top**

:0

- **Go to Bottom**
Shift + G


---

## 🟢 5. Reading Files & Redirection

### 📖 Reading Files Without Editing

- `cat <file>`
- Prints entire file

- `head -n 10 <file>`
- Shows first 10 lines

- `tail -n 10 <file>`
- Shows last 10 lines (useful for logs)

- `less <file>`
- Scrollable interactive view

---

## 🔄 Redirection Operators (`>` vs `>>`)

### ❌ Overwrite

echo "hello" > file.txt
- Deletes old content
- Writes new content

### ➕ Append

echo "hello" >> file.txt

- Adds content to the end
- Preserves existing data

---

🟣 **End of Day-3 | User Management | File Management | Vi Editor Shortcuts**

===============================================================================

# 🔐 Day-4 | Linux File Permissions Management

## 🎯 Session Overview
This session builds directly on **Day 3 (User Management)**.  
The **core problem** addressed here is **security**: even if you have multiple users, if everyone can delete everyone else's files, the system is useless.  
👉 You need **File Permissions**.

---

## 🟢 1. The Three Identities (Who?)

Linux divides authorization into **three specific categories** for every file and folder:

1. **User (u)**  
   👤 The owner. Usually the person who created the file.

2. **Group (g)**  
   👥 A collection of users (e.g., `"devs"`).  
   If a file belongs to a group, everyone in that group shares the same access level.

3. **Others (o)**  
   🌍 Everyone else on the system who is **not** the user and **not** in the group.

---

## 🟢 2. Deciphering Permissions (`ls -ltr`)

When you run `ls -ltr`, you see a string like:

-rwxrw-r--


This is **not random**. It is a code composed of **10 characters**.

### 🔎 Breakdown

- **1st Character** → File Type  
  - `d` = Directory  
  - `-` = File

- **Next 9 Characters** → Permissions (split into 3 sets)

| Set | Applies To | Meaning |
|----|-----------|--------|
| Set 1 | User | Permissions for the Owner |
| Set 2 | Group | Permissions for the Group |
| Set 3 | Others | Permissions for Everyone Else |

---

## 🟢 3. The Permissions (What?)

Each set contains a combination of **three letters**:

- **r (Read)** 📖  
  Capability to view the file contents.

- **w (Write)** ✏️  
  Capability to modify or delete the file.

- **x (Execute)** ▶️  
  Capability to run the file (e.g., executing a script).

---

## 🟢 4. Changing Permissions: The `chmod` Command

To change access levels, we use the **`chmod`** command.  
There are **two ways** to use it:

---

### 🔵 Method A: Symbolic Mode (The Alphabet Way)

You explicitly state **which letter goes to which identity**.

- **Command**

chmod u=rwx,g=rw,o=r filename


- **Meaning**
- User → all access  
- Group → read/write  
- Others → read only

---

### 🔵 Method B: Numeric Mode (The Math Way) ⭐ Most Common

Linux assigns a **number** to each permission.

| Permission | Value |
|-----------|-------|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |
| No Permission | 0 |

#### ➕ How to Calculate
You **sum up** the numbers for the permissions you want.

- **7 (All)** → Read (4) + Write (2) + Execute (1)
- **6 (Read/Write)** → Read (4) + Write (2)
- **4 (Read Only)** → Read (4)

#### 📌 Examples

- `chmod 777 file`  
🚨 Everyone can do everything (**Insecure**)

- `chmod 700 file`  
🔒 Only the User has full access; Group and Others have zero access

- `chmod 644 file`  
👤 User can read/write; everyone else can only read

---

## 🟢 5. Changing Ownership: The `chown` Command

Sometimes you don’t want to change permissions—you want to **transfer ownership**.

- **Command**

chown user:group filename

- **Example**

chown qe:qe test.sh

Transfers the file from `"developer"` to `"qe"`.

- **Note** ⚠️  
Only the **Root user** can typically execute this command to prevent users from dumping files on others.

---

## 🟢 6. The "Bank vs. Locker" Rule (Crucial Concept)

What if you have permission to read a file, but **not** the folder it sits inside?

### 🏦 The Analogy
- **Folder** → Bank  
- **File** → Locker  

Even if you have the **key to the locker**, if you are **banned from entering the bank**, you **cannot reach the locker**.

### 📌 The Rule
👉 **Directory permissions take precedence**.

If you do not have permission to access a folder (`/tmp`), you **cannot access the file inside** (`/tmp/file`), **regardless of the file’s permissions**.

---

🟣 **End of Day-4 | Linux File Permissions Management**

=================================================================

# ⚙️ Day-5 | Process Management | Monitoring | Networking | Disk Management

## 🎯 Session Overview
This session acts as the **final chapter of the "Linux Zero to Hero" series**, covering how to manage running applications, monitor system health, and expand storage capabilities.

---

## 🟢 1. Process Management

A **Process** is simply a running instance of a program (e.g., a Python script, a web server, or a shell command).  
The **Operating System** manages how these processes share the **CPU and Memory**.

---

### 🔵 A. Viewing Processes

- **`ps`**  
  Lists processes running in the **current shell session**.

- **`ps aux`** ⭐  
  The **gold standard command**.  
  Lists all processes from all users and includes **CPU and Memory utilization** columns.

- **`ps -ef`**  
  Similar to `ps aux`, but it does **not** show memory utilization.  
  Useful for viewing the **Process ID (PID)** and **Parent Process ID (PPID)**.

---

### 🔵 B. Managing Processes (Kill & Prioritize)

Sometimes a process hangs or consumes **100% CPU**. You need to intervene.

- **`kill <PID>`**  
  Sends a standard termination signal to stop the process.

- **`kill -9 <PID>`** 🚨  
  **Force Kill**. Use this if the standard `kill` command fails.

- **`kill -3 <PID>`**  
  Used primarily for **Java applications** to generate **thread dumps** for debugging without killing the app.

- **`renice -n <value> -p <PID>`**  
  Changes the **priority** of a process.  
  - Range: **-20 (Highest Priority)** → **19 (Lowest Priority)**  
  - 🩺 **Analogy**: The CPU is like a doctor; `renice` tells the doctor which patient (process) is in the ICU and needs immediate attention.

---

### 🔵 C. Services (`systemctl`)

Services are special processes that run in the **background** and start automatically when the server boots (e.g., Apache or Nginx).

- **Command**
# ⚙️ Day-5 | Process Management | Monitoring | Networking | Disk Management

## 🎯 Session Overview
This session acts as the **final chapter of the "Linux Zero to Hero" series**, covering how to manage running applications, monitor system health, and expand storage capabilities.

---

## 🟢 1. Process Management

A **Process** is simply a running instance of a program (e.g., a Python script, a web server, or a shell command).  
The **Operating System** manages how these processes share the **CPU and Memory**.

---

### 🔵 A. Viewing Processes

- **`ps`**  
  Lists processes running in the **current shell session**.

- **`ps aux`** ⭐  
  The **gold standard command**.  
  Lists all processes from all users and includes **CPU and Memory utilization** columns.

- **`ps -ef`**  
  Similar to `ps aux`, but it does **not** show memory utilization.  
  Useful for viewing the **Process ID (PID)** and **Parent Process ID (PPID)**.

---

### 🔵 B. Managing Processes (Kill & Prioritize)

Sometimes a process hangs or consumes **100% CPU**. You need to intervene.

- **`kill <PID>`**  
  Sends a standard termination signal to stop the process.

- **`kill -9 <PID>`** 🚨  
  **Force Kill**. Use this if the standard `kill` command fails.

- **`kill -3 <PID>`**  
  Used primarily for **Java applications** to generate **thread dumps** for debugging without killing the app.

- **`renice -n <value> -p <PID>`**  
  Changes the **priority** of a process.  
  - Range: **-20 (Highest Priority)** → **19 (Lowest Priority)**  
  - 🩺 **Analogy**: The CPU is like a doctor; `renice` tells the doctor which patient (process) is in the ICU and needs immediate attention.

---

### 🔵 C. Services (`systemctl`)

Services are special processes that run in the **background** and start automatically when the server boots (e.g., Apache or Nginx).

- **Command**
systemctl status|start|stop <service_name>
```bash
systemctl status|start|stop <service_name>
```
