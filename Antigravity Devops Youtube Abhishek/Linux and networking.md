# 🐧 Linux Zero to Hero — Interview Prep Guide

> 📺 Based on the **Linux Zero to Hero & Networking Fundamentals** series by Abhishek
> 🎯 Purpose: Master Linux fundamentals, file permissions, process management & networking for interviews

---

## 📋 Table of Contents

| Session | Topic |
|-----|-------|
| [Day 1](#-day-1-fundamentals-of-linux) | Fundamentals of Linux |
| [Day 2](#-day-2-linux-folder-structure) | Linux Folder Structure |
| [Day 3](#-day-3-user-management--file-management--vim) | User Management, File Management & Vim |
| [Day 4](#-day-4-linux-file-permissions) | Linux File Permissions |
| [Day 5](#-day-5-process-management--monitoring--disk) | Process Management, Monitoring & Disk |
| [Networking 1](#-networking-1-ip-addresses-subnetting--ports) | IP Addresses, Subnetting & Ports |
| [Networking 2](#-networking-2-osi-model--journey-of-data) | OSI Model & Journey of Data |
| [Master Cheatsheet](#-master-cheatsheet-linux--networking) | Master Cheatsheet |

---

---

# 🐧 Day 1: Fundamentals of Linux

**Goal:** Understand what an OS is, Linux's role, and how to set up a free Linux environment.

---

## 🔵 What is an Operating System?

```
User / Application
        ↓
   Operating System  ← bridge / intermediate layer
        ↓
   Hardware (CPU, RAM, Disk)
```

**OS Responsibilities:**
- Process Management — which app gets CPU time
- Memory Management — allocating RAM
- Device Management — printers, USB, disks
- Network Management — traffic in/out

---

## 🔵 Linux Architecture

```
Hardware
    ↓
Kernel (The "Engine" — manages CPU, RAM, I/O)
    ↓
System Utilities / Libraries (built-in tools)
    ↓
Shell (CLI interface — your way to talk to the kernel)
```

> Without the Shell, users cannot interact with the hardware.

---

## 🔵 Why Linux? (Market Share)

> Linux powers ~**90% of production workloads**

| Reason | Detail |
|---|---|
| **Free & Open Source** | Anyone can download, modify, redistribute |
| **Secure** | Inherently more secure than Windows for servers |
| **Distributions** | Ubuntu, Red Hat, CentOS — same kernel, different wrappers |

**Distro compatibility:** A script written for Ubuntu usually works on Red Hat — same underlying kernel and libraries.

---

## 🔵 Free Linux Setup (No Cloud Required)

| Method | Best For | Command |
|---|---|---|
| **WSL (Recommended)** | Windows users — runs Linux inside Windows | `wsl --install` in PowerShell |
| **Docker Container** | Mac or when WSL unavailable | `docker run -it ubuntu bash` |

> ⚠️ **Don't use AWS EC2 for Linux learning — costs money. Use WSL or Docker locally.**

---

## 🔵 Package Manager (apt)

> apt = App Store for Ubuntu

```bash
apt list              # List installed packages
apt update            # Refresh list of available software from repos
apt install python3   # Download + install Python from trusted source
apt install vim -y    # -y = auto-confirm without prompt
```

### 🎙️ Interview Answer: "What is the Linux kernel?"
> *"The kernel is the core engine of Linux — it sits between the hardware and the software. It manages CPU scheduling, memory allocation, device I/O, and networking. Users interact with the kernel through a shell (CLI), not directly. The shell translates human-readable commands into kernel system calls."*

---

---

# 📁 Day 2: Linux Folder Structure

**Goal:** Understand how Linux organizes files — the single-tree hierarchy.

---

## 🔵 Decoding the Command Prompt

```
root@ubuntu:/#

root    = who you are (root = super admin)
@ubuntu = hostname (machine name)
/       = current location (root of filesystem)
~       = home directory of current user
```

| Symbol | Meaning |
|---|---|
| `/` | Root of entire filesystem — top-level |
| `~` | Current user's home (`/home/ubuntu` for user ubuntu) |
| `root@` | Root user — unrestricted super admin |
| `ubuntu@` | Standard user — limited permissions |

---

## 🔵 Key Directories and Their Purpose

| Directory | Purpose | Analogy |
|---|---|---|
| `/bin` | Commands for all users (`ls`, `date`, `cat`) | User apps |
| `/sbin` | Admin/system commands (`useradd`, `reboot`) | Admin tools |
| `/usr/bin` | Modern Linux — actual binaries live here | `/bin` is often a symlink here |
| `/etc` | **System configuration files** | Settings / Control Panel |
| `/home` | Home directories for standard users | `/home/abhi` for user abhi |
| `/root` | Home directory for root user | Root user only — NOT inside `/home` |
| `/opt` | Third-party software (Tomcat, custom Java) | Optional installs |
| `/var/log` | Log files — web server logs, app logs | Everything that changes frequently |
| `/tmp` | Temporary files — auto-cleaned | Recently Deleted |
| `/mnt` | Mounting external disks/volumes | Temporarily attach new storage |
| `/lib` | Shared libraries required by kernel | Windows DLLs |
| `/boot` | Files required to start the OS | Bootloader |

> 🎯 **Most important for admins:** `/etc` (configs) and `/var/log` (logs)

---

## 🔵 The $PATH Variable (How Commands Are Found)

```
You type: ls

Linux checks $PATH directories in order:
  /bin → found ls? → execute it ✅

If not found in any $PATH directory → "command not found" ❌
```

```bash
echo $PATH            # See current PATH
which ls              # Where is the ls binary?
# Output: /usr/bin/ls
```

**If you see "command not found":**
- Program is not installed, OR
- Its directory is not in $PATH

### 🎙️ Interview Answer: "What is /etc in Linux?"
> *"/etc contains system configuration files — it's the control panel of Linux. Critical files like /etc/passwd (user info), /etc/hosts (network name mapping), /etc/ssh/sshd_config (SSH settings), and /etc/fstab (disk mounts) all live here. When troubleshooting, /etc is the first place to check for misconfigured services."*

---

---

# 👤 Day 3: User Management, File Management & Vim

**Goal:** Create users, manage files, and edit configuration files on headless servers.

---

## 🔵 User Management

> In a company, 100 devs cannot share the root password — you need individual accounts for **accountability**.

### Creating Users

| Command | Type | Creates Home Dir? | Use For |
|---|---|---|---|
| `useradd <name>` | Non-interactive | ❌ No | Scripts/automation |
| `adduser <name>` | Interactive (asks for name, password) | ✅ Yes | Adding real people |

```bash
useradd devuser          # Quick, non-interactive
adduser john             # Interactive — asks full name, password
passwd john              # Set or change password
userdel john             # Delete user
```

---

### Password Security — /etc/shadow

```bash
cat /etc/shadow          # Passwords stored here (encrypted hashes)
```

> 🎙️ **Interview Question:** *"Can you decrypt a password from /etc/shadow?"*
> **Answer:** No — it's a **one-way hash**. If forgotten, must reset. Cannot be recovered.

---

### Group Management

```bash
groupadd devops                   # Create group
usermod -aG devops john           # Add john to devops group
cat /etc/group                    # View all groups
```

> Manage permissions for 100 users by updating one group instead of 100 accounts.

---

## 🔵 SSH — Connecting to Remote Servers

```bash
ssh ubuntu@192.168.1.5            # Connect to remote server

# If password auth is disabled (cloud default):
cat /etc/ssh/sshd_config          # Check PasswordAuthentication = no
# AWS EC2 uses key pairs instead of passwords
ssh -i key.pem ubuntu@<IP>
```

---

## 🔵 File Management Commands (CRUD)

```bash
mkdir mydir              # Create directory
touch file.txt           # Create empty file
cp source.txt dest.txt   # Copy file
mv old.txt new.txt       # Rename (or move) file
rm file.txt              # Delete file
rm -rf mydir/            # ⚠️ Force delete folder and contents (dangerous!)
```

---

## 🔵 Reading Files

```bash
cat file.txt             # Print entire file
head -n 10 file.txt      # First 10 lines
tail -n 10 file.txt      # Last 10 lines (great for live logs)
less file.txt            # Scrollable interactive view
```

---

## 🔵 Vim Editor — Modes

> Most production servers have no GUI. Vim is essential.

```
3 Modes:
  Normal Mode  → default, navigation only
  Insert Mode  → press 'i' to type text
  Command Mode → press 'Esc' then ':' to save/quit
```

**Essential Vim commands:**
```
i           → Enter Insert mode (start typing)
Esc         → Return to Normal mode
:wq!        → Save and quit
:q!         → Quit WITHOUT saving
:0          → Go to top of file
Shift + G   → Go to bottom of file
```

---

## 🔵 Redirection: > vs >>

```bash
echo "hello" > file.txt    # OVERWRITE — deletes old content, writes new
echo "world" >> file.txt   # APPEND — adds to end, preserves existing data
```

### 🎙️ Interview Answer: "useradd vs adduser?"
> *"useradd is the low-level command — it's non-interactive, doesn't create a home directory by default, and is best for scripts and automation. adduser is a higher-level wrapper that's interactive — it prompts for a full name and password and automatically creates the home directory. For human users, I use adduser. For service accounts in scripts, I use useradd with specific flags."*

---

---

# 🔐 Day 4: Linux File Permissions

**Goal:** Control who can read, write, or execute files — the foundation of Linux security.

---

## 🔵 The 3 Identities

| Identity | Symbol | Who |
|---|---|---|
| **User** | `u` | File owner (usually the creator) |
| **Group** | `g` | Members of the file's assigned group |
| **Others** | `o` | Everyone else on the system |

---

## 🔵 Reading Permissions with ls -ltr

```bash
ls -ltr
# Example output:
-rwxrw-r-- 1 john devops 1024 Jun 24 file.sh
```

**Decoding `-rwxrw-r--`:**
```
Position 1:    "-"  = file type (d=directory, -=file)
Positions 2-4: "rwx" = User permissions   (read+write+execute)
Positions 5-7: "rw-" = Group permissions  (read+write, no execute)
Positions 8-10:"r--" = Others permissions (read only)
```

---

## 🔵 chmod — Changing Permissions

### Method A: Symbolic Mode

```bash
chmod u=rwx,g=rw,o=r file.sh
# User: all | Group: read+write | Others: read only
```

### Method B: Numeric Mode ⭐ (Most Common in Interviews)

| Permission | Value |
|---|---|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |
| None | 0 |

```
7 = 4+2+1 = rwx (full)
6 = 4+2   = rw- (read+write)
5 = 4+1   = r-x (read+execute)
4 = 4     = r-- (read only)
0 = 0     = --- (no access)
```

**Common examples:**
```bash
chmod 777 file   # ⚠️ Everyone full access — INSECURE
chmod 700 file   # Only owner has full access — others: zero
chmod 644 file   # Owner: read+write | Everyone else: read only
chmod 600 key.pem  # Only owner can read — required for SSH keys
```

---

## 🔵 chown — Changing Ownership

```bash
chown john:devops file.sh     # Transfer ownership to user john, group devops
chown -R john /opt/myapp      # Recursively change ownership of entire directory
```

> ⚠️ Only **root** can execute chown — prevents users from dumping files on others.

---

## 🔵 The "Bank vs Locker" Rule — Critical

```
Folder = Bank building
File   = Locker inside the bank

Even if you have the key to the locker,
if you're BANNED from entering the bank → you CANNOT reach the locker.
```

> **Directory permissions take precedence over file permissions.**

If a user has no `execute` permission on `/tmp`, they cannot access `/tmp/file.txt` regardless of the file's own permissions.

### 🎙️ Interview Answer: "What does chmod 644 mean?"
> *"644 means: the owner has read and write (4+2=6), the group has read only (4), and others have read only (4). This is the standard permission for configuration files — the admin who owns it can edit it, but everyone else can only read it. For SSH private keys I use 600 — only the owner can read it, no one else has any access, which is what AWS requires."*

---

---

# ⚙️ Day 5: Process Management, Monitoring & Disk

**Goal:** Manage running processes, identify performance bottlenecks, and expand server storage.

---

## 🔵 Process Management

> A process = a running instance of a program (Python script, web server, shell command).

### Viewing Processes

```bash
ps                # Processes in current shell only
ps aux            # ⭐ Gold standard — ALL processes, ALL users + CPU/RAM usage
ps -ef            # All processes with PID + PPID (no memory column)

# Key columns in ps aux:
# PID = process ID | %CPU = cpu usage | %MEM = memory | COMMAND = what's running
```

### Killing Processes

```bash
kill <PID>          # Standard termination signal (graceful)
kill -9 <PID>       # ⚠️ Force kill — use if standard kill fails
kill -3 <PID>       # Java thread dump — debug without killing the process
```

### Setting Priority (renice)

```bash
renice -n -10 -p <PID>    # Increase priority (more CPU time)
renice -n 15 -p <PID>     # Decrease priority (less CPU time)

# Range: -20 (highest) to 19 (lowest)
# Analogy: CPU = doctor. renice = triage — tells doctor which patient needs urgent care
```

### Services (systemctl)

```bash
systemctl status nginx        # Check service status
systemctl start nginx         # Start service
systemctl stop nginx          # Stop service
systemctl enable nginx        # Auto-start on server reboot
systemctl disable nginx       # Remove from startup
```

---

## 🔵 System Monitoring

| Command | What It Shows |
|---|---|
| `top` | Live dashboard — processes sorted by highest CPU/RAM |
| `htop` | Colorful, user-friendly version of top |
| `free -h` | Total vs used RAM in human-readable format (MB/GB) |
| `nproc` | Number of CPU cores available |
| `df -h` | Disk space per filesystem (e.g., `/` is 90% full) |
| `du -sh /var/log` | Size of a specific directory |

> 💡 For production: integrate these with **Prometheus + Grafana** for alerting.

---

## 🔵 Disk Management — Adding Storage (4-Step Workflow)

**Scenario:** Server runs out of disk space — need to add a new EBS volume (AWS).

```bash
# Step 1: Attach EBS volume in AWS console (same AZ as EC2)

# Step 2: Verify new disk appeared
lsblk
# Shows: xvdf (new unformatted disk)

# Step 3: Format the disk
mkfs -t ext4 /dev/xvdf
# (can also use xfs instead of ext4)

# Step 4: Mount to a directory
mount /dev/xvdf /mnt/demo
# Any files saved to /mnt/demo now stored on the new drive ✅
```

> ⚠️ Mount is temporary — add to `/etc/fstab` for it to persist after reboot.

### 🎙️ Interview Answer: "Server running out of disk space — what do you do?"
> *"First I identify the issue with `df -h` to see which filesystem is full, then `du -sh *` in the problematic directory to find what's consuming space. If it's logs, I clean up or archive old logs. If it's a genuine storage need, on AWS I create a new EBS volume in the same Availability Zone, attach it to the instance, format it with `mkfs -t ext4`, mount it to the appropriate directory, and add it to /etc/fstab so it persists after reboot."*

---

---

# 🌐 Networking 1: IP Addresses, Subnetting & Ports

**Goal:** Understand how devices identify themselves and how networks are segmented.

---

## 🔵 IP Address — The Unique ID

> Every device on a network gets a unique IP address — like a home address for your server.

**IPv4 structure:** `192.168.1.5`
- 4 numbers separated by dots
- Each number: **0 to 255**
- Why 255? IPv4 is 32 bits → 4 bytes × 8 bits → 2⁸ = 256 values (0–255)

**Why IPs matter for DevOps:**
> If a malicious user accesses a system, the network administrator uses the IP to trace exactly which device did it.

---

## 🔵 Subnetting — Security Through Isolation

**Problem:** Everyone on one giant network = hacker compromises one device → accesses everything (including Finance servers).

**Solution:** Split into isolated sub-networks.

```
Company Network:
├── Finance Subnet (Private — no internet)
├── Engineering Subnet (Private — VPN only)
└── Guest WiFi Subnet (Public — internet only)

Hacker breaches Guest WiFi → Finance data is still isolated ✅
```

| Subnet Type | Internet Access | Use For |
|---|---|---|
| **Private** | ❌ No | Databases, app servers |
| **Public** | ✅ Yes (via IGW) | Load balancers, bastion hosts |

**Private IP ranges (reserved — won't conflict with public internet):**
- `10.x.x.x`
- `172.x.x.x`
- `192.x.x.x`

---

## 🔵 CIDR — Calculating Network Size

> CIDR notation (e.g., `/24`) defines how many IP addresses a subnet contains.

**Formula:** `2^(32 - CIDR prefix)` = number of IPs

| CIDR | Calculation | Total IPs | Use Case |
|---|---|---|---|
| `/32` | 2⁰ | **1 IP** | Single specific device |
| `/29` | 2³ | **8 IPs** | Very small cluster |
| `/24` | 2⁸ | **256 IPs** | Standard LAN / home network |
| `/16` | 2¹⁶ | **65,536 IPs** | Large VPC or office network |

---

## 🔵 Ports — Application Identification

> IP address → finds the correct **server**. Port → finds the correct **application** inside that server.

```
IP = Building address
Port = Apartment number

Access format: 192.168.1.5:8080
               ↑           ↑
            Server IP    Application Port
```

**Common ports:**
| Port | Service |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 8080 | Jenkins |
| 3306 | MySQL |
| 6379 | Redis |

### 🎙️ Interview Answer: "What is subnetting and why do we use it?"
> *"Subnetting divides a large network into smaller, isolated logical networks. This improves security — even if a hacker compromises a public subnet, private subnets containing databases or finance systems remain isolated. In AWS, I put load balancers in public subnets with internet access, and application servers and databases in private subnets. CIDR notation controls how many IPs a subnet gets — /24 gives 256 IPs, /16 gives 65,536."*

---

---

# 🧠 Networking 2: OSI Model — Journey of Data

**Goal:** Understand how data travels from your laptop to a server — layer by layer.

---

## 🔵 Prerequisites: Before Data Travels

**Two checks happen before any OSI layer:**

### 1. DNS Resolution
```
You type: google.com
Your laptop checks: Local DNS cache → ISP DNS server
Result: google.com = 8.8.8.8 (IP resolved ✅)
```
> If domain doesn't exist → no point starting data transfer.

### 2. TCP 3-Way Handshake
```
Client: SYN        ("Hi, are you ready?")
Server: SYN-ACK    ("Hi, yes I'm ready!")
Client: ACK        ("Great, let's talk!")
                         ↓
               Data transfer begins ✅
```

---

## 🔵 The 7 OSI Layers

```
┌─────────────────────────────────────────────────────────┐
│          APPLICATION (Browser / App)                     │
│  Layer 7: Application  → HTTP/FTP/SMTP request          │
│  Layer 6: Presentation → Encryption (HTTPS)             │
│  Layer 5: Session      → Session ID (cookies/login)     │
├─────────────────────────────────────────────────────────┤
│          TRANSPORT (OS)                                  │
│  Layer 4: Transport    → Segment data, TCP vs UDP       │
├─────────────────────────────────────────────────────────┤
│          NETWORK (OS + Hardware)                         │
│  Layer 3: Network      → Routing, Source/Dest IP        │
│  Layer 2: Data Link    → Local delivery, MAC addresses  │
│  Layer 1: Physical     → Signals, fiber, ethernet cables│
└─────────────────────────────────────────────────────────┘
```

---

## 🔵 Each Layer Explained

| Layer | Name | Action | Data Unit |
|---|---|---|---|
| **7** | Application | Initiates HTTP/FTP/SMTP request | Data |
| **6** | Presentation | Encrypt/decrypt (HTTPS) | Data |
| **5** | Session | Manages session ID (keeps you logged in) | Data |
| **4** | Transport | Breaks large files into segments; TCP vs UDP | Segments |
| **3** | Network | Routers select path; adds Source + Dest IP | Packets |
| **2** | Data Link | Switches handle local delivery; adds MAC address | Frames |
| **1** | Physical | Actual bits transmitted as electrical/light signals | Bits |

---

## 🔵 TCP vs UDP (Layer 4 Decision)

| | TCP | UDP |
|---|---|---|
| **Reliability** | ✅ Guaranteed delivery (retransmits lost packets) | ❌ Fire and forget |
| **Speed** | Slower (handshake + ACKs) | Faster (no overhead) |
| **Use case** | Web browsing, file transfer, SSH | Video streaming, DNS, gaming |

---

## 🔵 What Happens at the Server (Reverse)

```
Receives Bits (Layer 1)
→ Unpacks Frames (Layer 2)
→ Unpacks Packets (Layer 3)
→ Reassembles Segments (Layer 4)
→ Decrypts (Layer 6)
→ Processes HTTP request (Layer 7)
→ Generates HTML response
→ Sends back down the stack to the client
```

---

## 🔵 OSI vs TCP/IP Model

| | OSI Model | TCP/IP Model |
|---|---|---|
| **Layers** | 7 | 4 |
| **Layers 5–7 (OSI)** | Separate | Combined into "Application" |
| **Use** | Learning & troubleshooting | Real-world implementation |

> Use OSI to troubleshoot: *"Is this a Layer 3 (routing) issue or Layer 4 (port/firewall) issue?"*

### 🎙️ Interview Answer: "Explain the OSI model"
> *"The OSI model has 7 layers. Starting from the top: the Application layer initiates the request (HTTP/HTTPS), the Presentation layer handles encryption for HTTPS, and the Session layer manages session IDs to keep you logged in. At Layer 4, Transport breaks data into segments and decides TCP for reliability or UDP for speed. Layer 3 Network adds routing information — source and destination IPs — so routers can find the path. Layer 2 Data Link handles local delivery using MAC addresses. Layer 1 Physical converts everything to electrical or light signals on cables. At the destination server, this process reverses layer by layer."*

---

---

# 📌 Master Cheatsheet — Linux & Networking

```
╔══════════════════════════════════════════════════════════════════════╗
║          LINUX & NETWORKING INTERVIEW CHEATSHEET                     ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  LINUX BASICS:                                                       ║
║  90% of prod servers = Linux | apt = Ubuntu package manager         ║
║  Kernel = engine | Shell = CLI to talk to kernel                    ║
║                                                                      ║
║  KEY DIRECTORIES:                                                    ║
║  /etc = configs (most important!) | /var/log = logs                 ║
║  /bin = user commands | /sbin = admin commands                      ║
║  /home = user home dirs | /root = root user home                   ║
║  /opt = third-party apps | /tmp = auto-cleaned temp files           ║
║  $PATH = where Linux looks for commands (command not found = not in PATH)
║                                                                      ║
║  USER MANAGEMENT:                                                    ║
║  useradd = scripting (no home dir) | adduser = interactive (home dir)
║  /etc/shadow = hashed passwords (one-way hash, cannot decrypt)      ║
║  usermod -aG groupname username → add user to group                 ║
║  SSH: PasswordAuthentication no → use key pairs                     ║
║                                                                      ║
║  FILE PERMISSIONS (chmod):                                           ║
║  r=4, w=2, x=1 | 7=rwx | 6=rw | 4=r | 0=none                      ║
║  chmod 644 = owner rw, others r | chmod 700 = owner all, others none
║  chmod 600 = SSH key permission | chown user:group file             ║
║  Directory permissions override file permissions (Bank vs Locker)   ║
║                                                                      ║
║  PROCESS MANAGEMENT:                                                 ║
║  ps aux = all processes + CPU/RAM | kill -9 PID = force kill        ║
║  kill -3 PID = Java thread dump | renice range: -20 to 19           ║
║  systemctl start/stop/enable/disable service                        ║
║                                                                      ║
║  MONITORING:                                                         ║
║  top/htop = live dashboard | free -h = RAM | df -h = disk           ║
║  du -sh /dir = directory size | nproc = CPU cores                   ║
║                                                                      ║
║  DISK MANAGEMENT:                                                    ║
║  lsblk → mkfs -t ext4 /dev/xvdf → mount /dev/xvdf /mnt/demo        ║
║  Add to /etc/fstab for persistence after reboot                     ║
║                                                                      ║
║  NETWORKING:                                                         ║
║  IPv4 = 32 bits | each octet 0-255 | CIDR: 2^(32-prefix) = IPs     ║
║  /24 = 256 IPs | /16 = 65,536 IPs | /32 = 1 IP                     ║
║  Private ranges: 10.x, 172.x, 192.168.x                            ║
║  IP = server | Port = app on that server                            ║
║                                                                      ║
║  OSI MODEL (top → bottom):                                          ║
║  7=Application | 6=Presentation | 5=Session                         ║
║  4=Transport (TCP/UDP, segments) | 3=Network (IP, packets)          ║
║  2=Data Link (MAC, frames) | 1=Physical (signals, bits)             ║
║  TCP=reliable+slow | UDP=fast+no guarantee                          ║
║  Before OSI: DNS resolution + TCP 3-way handshake (SYN/SYN-ACK/ACK)║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Delivery Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Use the **Bank vs Locker** analogy for permissions | Just recite permission numbers |
| Explain **why** (accountability) for user management | Just describe useradd commands |
| Draw the OSI stack top-to-bottom | Jump to "Layer 3 is routing" without context |
| Say "one-way hash" for /etc/shadow password question | Say "it's encrypted" (imprecise) |
| Mention `/etc/fstab` for persistent mounts | Forget about reboot persistence |

---

## 📚 Resources

- 📺 [Linux Zero to Hero Series](https://www.youtube.com/watch?v=Ou9j73aWgyE)
- 🔗 [Linux Man Pages](https://linux.die.net/man/)
- 🔗 [WSL Install Guide](https://learn.microsoft.com/en-us/windows/wsl/install)
- 🔗 [OSI Model Reference](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)

---

> ⭐ **Star this repo** if it helped you prepare for your DevOps interview!
> 🔔 Paste the next day's notes — I'll overwrite with only those days!
