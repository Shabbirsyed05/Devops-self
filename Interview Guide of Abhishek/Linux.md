# Linux Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `03-linux/` folder

---

# Table of Contents

1. [Most Used Linux Commands](#1-most-used-linux-commands)
2. [Restore a Lost PEM File](#2-restore-a-lost-pem-file)
3. [/var Folder is Full](#3-var-folder-is-full)
4. [Server Slow — High CPU Usage](#4-server-slow--high-cpu-usage)
5. [App Returns Connection Refused](#5-app-returns-connection-refused)
6. [SSH Stopped Working](#6-ssh-stopped-working)
7. [List Log Files Older Than 7 Days](#7-list-log-files-older-than-7-days)
8. [Remove Log Files Older Than 30 Days](#8-remove-log-files-older-than-30-days)
9. [Cronjob Log Rotation Automation](#9-cronjob-log-rotation-automation)
10. [Create Users from CSV](#10-create-users-from-csv)
11. [Service Health Monitor](#11-service-health-monitor)
12. [Delete Files Larger Than 100MB](#12-delete-files-larger-than-100mb)
13. [Users Logged In Today](#13-users-logged-in-today)
14. [Website Does Not Load](#14-website-does-not-load)
15. [Remove First and Last Lines from a File](#15-remove-first-and-last-lines-from-a-file)
16. [Shell Scripts](#16-shell-scripts)
    - [create_users.sh](#create_userssh)
    - [log_rotation.sh](#log_rotationsh)
    - [monitor_services.sh](#monitor_servicessh)

---

# 1. Most Used Linux Commands

## Question
What are 10 Linux commands you use daily? (Excluding basic ones like `ls` and `cd`)

### 📝 Short Explanation
This question aims to assess your hands-on experience with Linux, focusing on diagnostic, file manipulation, service management, and automation-related commands that go beyond basic navigation.

## ✅ Answer
Here are 10 Linux commands I use regularly, excluding the basics like `cd`, `ls`, and `pwd`:

---

### 1. `tail -f`
```bash
tail -f /var/log/nginx/error.log
```
🔍 Monitor log files in real time — very useful for debugging issues as they happen.

---

### 2. `grep`
```bash
grep -i "timeout" /var/log/app.log
```
🔎 Search through files, logs, or command outputs for specific patterns. I use this to quickly isolate errors.

---

### 3. `systemctl`
```bash
systemctl restart nginx
```
🛠️ Control system services — starting, stopping, checking status of systemd services.

---

### 4. `journalctl`
```bash
journalctl -u docker.service -f
```
🧾 View logs for systemd-managed services. Especially handy for debugging issues with services like Docker or Kubelet.

---

### 5. `ps aux | grep`
```bash
ps aux | grep nginx
```
📋 List running processes. I use this to find rogue or resource-intensive processes.

---

### 6. `df -h` / `du -sh`
```bash
df -h       # Check available disk space
du -sh *    # See folder sizes in current directory
```
💾 Essential for disk space monitoring and cleaning up large files or folders.

---

### 7. `chmod` / `chown`
```bash
chmod +x deploy.sh
chown ubuntu:ubuntu script.sh
```
🔐 Manage file permissions and ownership — very common in CI/CD and provisioning tasks.

---

### 8. `find`
```bash
find /var/log -name "*.log" -mtime +7
```
🔍 Locate files based on name, date, type, etc. Great for automating cleanup or audits.

---

### 9. `curl`
```bash
curl -I http://localhost:8080
```
🌐 Test web endpoints, APIs, or service availability from the command line.

---

### 10. `rsync`
```bash
rsync -avz /app/ user@server:/backup/
```
📁 Efficient file syncing and backup — much faster and more reliable than `scp` for large directories.

---

---

# 2. Restore a Lost PEM File

## Question
Can you restore a lost PEM file? If not, how can you still access the EC2 instance?

### 📝 Short Explanation
This question evaluates your knowledge of secure SSH access, key-based authentication, and how to regain access to EC2 instances when your only login method (PEM file) is lost.

## ✅ Answer
No, you **cannot restore** a lost PEM file — it's not stored on AWS or recoverable.
However, you can **regain access** by using a workaround: create a new key pair, attach it to the instance via a temporary EC2 rescue process, and restore SSH access.

### 📘 Detailed Explanation

PEM files (private keys) are **never retrievable from AWS** after initial creation. Losing the PEM file means you cannot SSH into the EC2 instance using the existing key pair.

But here's how you can still regain access:

---

### ✅ Recovery Steps:

#### Step 1: Create a new key pair
```bash
aws ec2 create-key-pair --key-name new-key --query 'KeyMaterial' --output text > new-key.pem
chmod 400 new-key.pem
```

---

#### Step 2: Stop the affected instance
In the AWS Console or CLI:
```bash
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxx
```

---

#### Step 3: Detach the root EBS volume from the stopped instance

- Go to **EC2 > Volumes**
- Find the volume attached to your instance (usually `/dev/xvda`)
- **Detach** it

---

#### Step 4: Attach the volume to a temporary (working) instance
- Attach it as a secondary volume (e.g., `/dev/sdf`)

---

#### Step 5: SSH into the temporary instance
Mount the volume:
```bash
sudo mkdir /mnt/recovery
sudo mount /dev/xvdf1 /mnt/recovery
```

Edit the `authorized_keys` file on the broken instance's volume:
```bash
sudo nano /mnt/recovery/home/ec2-user/.ssh/authorized_keys
```

Add the **public key** from your new key pair (`new-key.pub`)

---

#### Step 6: Detach the volume from the rescue instance
- Unmount the volume
- Detach it and re-attach to the original instance as `/dev/xvda`

---

#### Step 7: Start the original instance and SSH with new key
```bash
ssh -i new-key.pem ec2-user@<public-ip>
```

---

### 🧠 Prevention Tips:

- Always back up PEM files in a secure location (like a password manager).
- Create a **secondary user** with a different key for backup access.
- Use EC2 Instance Connect for temporary browser-based access (only works for Amazon Linux 2+ and enabled roles).

> Summary:
> You cannot restore a PEM file, but you can regain access by editing the `authorized_keys` on the root volume through a temporary rescue EC2 instance.

---

---

# 3. /var Folder is Full

## Question
`/var` is almost 90% full. What will be your next steps?

### 📝 Short Explanation
This question checks your troubleshooting and disk management skills. The `/var` directory is commonly used for logs, spools, caches, and runtime data — so issues here can break system processes or fill up disks silently.

## ✅ Answer
My first step is to identify what's consuming the space inside `/var`. Then I would clean up unnecessary files like rotated logs, caches, or orphaned packages — and put alerts or log rotation in place to avoid recurrence.

### 📘 Detailed Explanation

---

### ✅ Step 1: Inspect Disk Usage Under `/var`

```bash
sudo du -sh /var/* | sort -hr | head -10
```
This will show which directories inside `/var` are consuming the most space — usually it's `/var/log`, `/var/cache`, or `/var/lib/docker`.

---

### ✅ Step 2: Clean Log Files
If `/var/log` is the culprit:

```bash
sudo journalctl --vacuum-size=200M
sudo rm -rf /var/log/*.gz /var/log/*.[0-9]
```

Or truncate large log files:
```bash
sudo truncate -s 0 /var/log/syslog
```

---

### ✅ Step 3: Clear Package Cache
If using `apt` or `yum`, clear the package manager cache:

```bash
sudo apt clean         # Debian/Ubuntu
sudo yum clean all     # RHEL/CentOS
```

---

### ✅ Step 4: Check Docker Artifacts
If the server runs containers:

```bash
docker system df        # See what's taking space
docker system prune -a  # Remove unused containers/images
```

**⚠️ Warning:** Prune removes *unused* images and volumes — be cautious on production systems.

---

### ✅ Step 5: Consider Moving or Archiving Data
If data in `/var` is needed but rarely accessed:
- Archive old logs to `/home` or S3
- Use `logrotate` to compress and limit logs:
  ```bash
  sudo nano /etc/logrotate.conf
  ```

---

### ✅ Step 6: Set Up Alerts and Monitoring
- Install `ncdu`, `duf`, or setup Prometheus/Grafana alerts for disk usage thresholds.
- Automate cleanup with cron or systemd timers if appropriate.

---

### 🧠 Why `/var` Fills Up:
- Verbose logging (e.g., failed cron jobs, app debug logs)
- Docker images/layers
- Orphaned cache files
- Email spools or crash dumps

> Summary:
> Quickly inspect, clean, and automate monitoring. Ensure critical services like journald, docker, and package managers are not starved of space.

---

---

# 4. Server Slow — High CPU Usage

## Question
Linux Server is slow due to high CPU utilization. How will you fix it?

### 📝 Short Explanation
This question assesses your ability to diagnose performance issues, identify root causes, and take targeted actions to reduce CPU load on a production server.

## ✅ Answer
I would begin by identifying which processes are consuming the most CPU using tools like `top`, `htop`, or `pidstat`, then analyze whether it's due to a misbehaving application, runaway process, or scheduled job. Based on the findings, I'd take corrective action — either by killing the process, adjusting resource limits, or scaling the workload.

### 📘 Detailed Explanation

---

### ✅ Step 1: Check Load Average
```bash
uptime
```
Example output:
```
14:02:03 up  3 days,  4:55,  2 users,  load average: 6.02, 4.33, 2.89
```
A load average consistently higher than the number of CPU cores indicates overutilization.

---

### ✅ Step 2: Identify CPU-Heavy Processes
```bash
top -o %CPU
```
or more interactively:
```bash
htop
```

This shows which processes are consuming the most CPU.

---

### ✅ Step 3: Drill Down with `ps` or `pidstat`
```bash
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head
```

or:
```bash
pidstat -u 1 5
```

These give detailed insight into CPU consumption over time.

---

### ✅ Step 4: Investigate the Cause
Based on what you see, ask:
- Is it a specific app (e.g., Java, Python, Node.js)?
- Is there a cron job or batch script running?
- Is a service misconfigured and looping?
- Is it caused by a known bug (e.g., zombie processes)?

---

### ✅ Step 5: Take Corrective Action
- Kill or restart runaway process:
  ```bash
  kill -9 <pid>
  systemctl restart <service>
  ```
- Scale the application or move workloads
- Limit resource usage using `nice`, `cpulimit`, or cgroups
- Tune app performance (e.g., DB queries, memory leaks)

---

### ✅ Step 6: Check Logs
```bash
journalctl -xe
tail -f /var/log/syslog
```
Logs may reveal:
- App crashes
- High retry loops
- Configuration issues

---

### ✅ Step 7: Implement Preventive Measures
- Set CPU/memory limits in containerized apps
- Use monitoring tools like `Prometheus + Grafana`
- Configure alerts for high CPU (e.g., above 80% for 5 mins)
- Refactor long-running or expensive tasks

---

### 🧠 Real-Life Examples:
- A cron script looping due to a bad condition
- A Java app stuck in infinite recursion
- Docker containers running unbounded scraping jobs
- Antivirus or audit daemon consuming CPU after log floods

> Summary:
> Use `top`, `htop`, `ps`, and `pidstat` to identify heavy processes. Fix the root cause and add monitoring to avoid similar issues in the future.

---

---

# 5. App Returns Connection Refused

## Question
Application deployed on NGINX returns "Connection Refused". How will you fix it?

### 📝 Short Explanation
This question evaluates your ability to troubleshoot reverse proxy setups and network-level issues. "Connection Refused" often means the NGINX server or upstream service isn't reachable at all.

## ✅ Answer
I would first check whether NGINX itself is running and listening on the correct port, then verify that the application backend is also up and accessible. It could be a misconfiguration in the NGINX config or the application not listening on the expected socket or port.

### 📘 Detailed Explanation

---

### ✅ Step 1: Reproduce the Error
Try to access the app from the browser or use:
```bash
curl -I http://localhost
```
If you get:
```
curl: (7) Failed to connect to localhost port 80: Connection refused
```
It confirms the server refused the TCP handshake — not a 4xx/5xx error.

---

### ✅ Step 2: Check if NGINX is Running
```bash
sudo systemctl status nginx
```
If it's inactive or failed:
```bash
sudo systemctl restart nginx
sudo journalctl -u nginx -xe
```

---

### ✅ Step 3: Is NGINX Listening on the Expected Port?
```bash
sudo netstat -tulnp | grep nginx
```
or:
```bash
ss -tuln | grep :80
```
No output? Then NGINX is not listening on the port you're accessing.

---

### ✅ Step 4: Check NGINX Configuration
```bash
sudo nginx -t
```
This tests the NGINX config for syntax errors.

Also verify your `/etc/nginx/sites-enabled/default` or your custom config:

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;  # Is your app running here?
    }
}
```

---

### ✅ Step 5: Verify the Application Backend
If NGINX is trying to proxy to `http://localhost:5000`, is your app actually running on that port?

```bash
sudo netstat -tulnp | grep 5000
curl http://localhost:5000
```

If this fails, restart or debug your app.

---

### ✅ Step 6: Check Firewall/Security Groups (Cloud Hosts)
On cloud VMs, make sure the port is open in:
- AWS Security Group
- GCP firewall rules
- `ufw` or `iptables` on the VM

```bash
sudo ufw status
sudo iptables -L
```

---

### ✅ Step 7: Look for SELinux or AppArmor Restrictions
If using SELinux:
```bash
sudo getenforce
```
If it's `Enforcing`, and ports/services are restricted, update policies or temporarily disable for testing.

---

### 🧠 Common Real-Life Causes:
- App crashed or not listening on correct port
- Wrong proxy_pass value in NGINX
- Port blocked by firewall
- NGINX service not restarted after config change
- App takes too long to start — NGINX proxies fail

> Summary:
> Check if NGINX is running, verify app backend availability, inspect NGINX configs, and ensure network rules allow traffic. Fix any misalignment between proxy settings and actual service ports.

---

---

# 6. SSH Stopped Working

## SSH to an instance stopped working. How will you troubleshoot the issue?

### 📝 Short Explanation
This question checks how well you can debug access issues to a Linux server, particularly EC2 instances or cloud VMs. SSH failures could be due to network, OS-level, key-based, or IAM misconfigurations.

## ✅ Answer
I would follow a step-by-step process to identify whether the issue is with networking (e.g., security group), the instance itself (e.g., crashed SSH service), or the credentials (e.g., PEM file or key mismatch). Based on the findings, I would take corrective actions accordingly.

---

### 📘 Detailed Explanation

---

### ✅ Step 1: Confirm the Error Message
From your local machine:
```bash
ssh -i my-key.pem ec2-user@<instance-public-ip>
```

Typical errors:
- `Permission denied (publickey)`
- `Connection refused`
- `Operation timed out`

The error gives the first clue.

---

### ✅ Step 2: Check Instance Health & Reachability
- Is the instance **running**?
- Is it **reachable**?

Use AWS Console or CLI:
```bash
aws ec2 describe-instance-status --instance-id <id>
ping <public-ip>
```

If instance is unreachable → investigate VPC/subnet/NACL routing issues.

---

### ✅ Step 3: Verify Security Group Rules
Make sure port 22 is open **from your IP**:
```text
Inbound rule:
Type: SSH
Port: 22
Source: your IP (e.g., 203.0.113.0/32)
```

If using a **bastion host**, check its connectivity as well.

---

### ✅ Step 4: Check Network ACLs & Route Tables
Ensure NACLs are not blocking traffic and public subnet has a route to internet gateway.

---

### ✅ Step 5: Confirm Public IP or Elastic IP
Check if the instance has a **public IP** or **Elastic IP** attached.
Elastic IPs don't change, but public IPs do if the instance is stopped and started.

---

### ✅ Step 6: Validate PEM File & User
Make sure:
- The PEM file is correct (`chmod 400`)
- You're using the right username:
  - `ec2-user` for Amazon Linux
  - `ubuntu` for Ubuntu
  - `centos` for CentOS

---

### ✅ Step 7: Try EC2 Instance Connect (Amazon Linux only)
If the PEM is lost or SSH doesn't work:
- Use **EC2 Instance Connect** via AWS Console
- Once inside, you can check:
  ```bash
  sudo systemctl status sshd
  tail -n 50 /var/log/auth.log  # or secure/log
  ```

---

### ✅ Step 8: Rescue Mode (Advanced)
If all else fails:
- Stop the instance
- Detach the root volume
- Attach it to another instance
- Mount it and edit `~/.ssh/authorized_keys` or repair config
- Reattach and start the original instance

---

### 🧠 Real-Life Causes I've Faced:
- Team used a wrong key pair name
- Security group updated accidentally
- User tried SSH with wrong username (e.g., root)
- Instance rebooted with new IP and old DNS cached
- `/etc/ssh/sshd_config` edited incorrectly

> Summary:
> Identify error → check network reachability → inspect key/user mismatch → use EC2 Connect if possible → rescue via EBS if needed.

---

---

# 7. List Log Files Older Than 7 Days

## Question
How do you find and list the log files older than 7 days in the `/var/log` folder?

### 📝 Short Explanation
This question tests your comfort with Linux file management and log housekeeping — a common task for DevOps and sysadmins.

## ✅ Answer
You can use the `find` command with the `-mtime` option to locate files older than 7 days:

```bash
find /var/log -type f -mtime +7
```

### 📘 Detailed Explanation

#### 🔍 Breakdown of the command:

- `find`: The Linux command to search for files in a directory hierarchy.
- `/var/log`: The target directory that contains log files.
- `-type f`: Limits the search to files (not directories).
- `-mtime +7`: Filters files **modified more than 7 days ago**.
  - `+7` means strictly older than 7 days.
  - `-7` would mean newer than 7 days.

---

#### 🛠️ Practical Usage:
If you want to **view the size and timestamp** of those files:
```bash
find /var/log -type f -mtime +7 -exec ls -lh {} \;
```

If you want to **delete** those files:
```bash
sudo find /var/log -type f -mtime +7 -delete
```
⚠️ Be careful with deletion — make sure you've reviewed the list first.

---

> Summary:
> Use `find /var/log -type f -mtime +7` to list log files older than 7 days — a must-know for log maintenance in production servers.

---

---

# 8. Remove Log Files Older Than 30 Days

## Question
How do you find and remove log files older than 30 days using `-exec` in a folder?

### 📝 Short Explanation
This version of the task evaluates your comfort with `find -exec`, which is helpful when you want to take specific actions on matched files (like logging or conditional deletion).

## ✅ Answer
You can use `-exec rm` with `find` to remove log files older than 30 days:

```bash
sudo find /path/to/folder -type f -name "*.log" -mtime +30 -exec rm -f {} \;
```

### 📘 Detailed Explanation

#### 🔍 Breakdown of the command:
- `sudo`: Used if the directory (like `/var/log`) requires root access.
- `find`: The base command to search files.
- `/path/to/folder`: Replace with your target directory (e.g., `/var/log`).
- `-type f`: Ensures only files are matched.
- `-name "*.log"`: Filters files ending with `.log`.
- `-mtime +30`: Filters files modified **over 30 days ago**.
- `-exec rm -f {} \;`: Executes the `rm -f` command on each matched file:
  - `{}` gets replaced by the current file path.
  - `\;` indicates the end of the command.

---

#### ✅ Example:
```bash
sudo find /var/log -type f -name "*.log" -mtime +30 -exec rm -f {} \;
```

This command will delete all `.log` files in `/var/log` older than 30 days.

---

#### 🧠 Bonus Tip – Dry run before delete:
You can review the files that would be deleted:
```bash
find /var/log -type f -name "*.log" -mtime +30 -exec ls -lh {} \;
```

---

> Summary:
> Using `-exec` gives you fine-grained control over file operations. It's especially useful when you want to extend the logic beyond simple deletion, such as archiving or compressing matched files.

---

---

# 9. Cronjob Log Rotation Automation

## Question
Your application generates large logs in `/var/log/myapp/` and there's no log rotation setup.

**Task:**
Write a shell script that compresses logs older than 7 days and deletes logs older than 30 days. Also, run it daily via cron.

### 📝 Short Explanation
This question tests your ability to manage disk space with log compression and retention — a common task in DevOps. You're expected to automate it safely and consistently using a cron job.

## ✅ Answer

### 🖥️ Shell Script: `log_cleanup.sh`

```bash
#!/bin/bash

# Directory where logs are stored
LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

# Ensure the log directory exists
if [ ! -d "$LOG_DIR" ]; then
    echo "[$(date)] ERROR: Log directory $LOG_DIR does not exist!" >> "$LOG_FILE"
    exit 1
fi

# Compress logs older than 7 days (but newer than 30)
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

# Optional: Delete uncompressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.log" -mtime +30 -exec rm -f {} \; -exec echo "[$(date)] Deleted (uncompressed): {}" >> "$LOG_FILE" \;

# Done
echo "[$(date)] Log rotation completed successfully." >> "$LOG_FILE"
```

---

---

# 10. Create Users from CSV

## Question
You've received a CSV file with a list of usernames and passwords to create users on a Linux system.

**Task:**
Write a shell script to read the CSV and:
- Create each user with the specified password.
- Force password change on first login.

```csv
username,password
alice,Password@123
bob,Secure@456
carol,DevOps@789
```

### 📝 Short Explanation
This task tests your ability to read structured data (CSV) and automate user management using shell scripting — commonly required for onboarding automation in DevOps roles.

## ✅ Answer


```bash
#!/bin/bash

INPUT="users.csv"

# Check if the file exists
if [[ ! -f "$INPUT" ]]; then
  echo "CSV file not found!"
  exit 1
fi

# Skip header and read each line
tail -n +2 "$INPUT" | while IFS=',' read -r username password; do
  # Check if user already exists
  if id "$username" &>/dev/null; then
    echo "User '$username' already exists. Skipping..."
    continue
  fi

  # Create the user
  useradd "$username"

  # Set the password
  echo "${username}:${password}" | chpasswd

  # Force password change on first login
  chage -d 0 "$username"

  echo "User '$username' created successfully."
done
```

---

### ✅ Script Usage:
Make it executable and run with root privileges:
```bash
chmod +x create_users.sh
sudo ./create_users.sh
```

---

### 🧠 How It Works:

- `IFS=',' read -r ...` parses each line into `username` and `password`.
- `useradd` creates the user.
- `chpasswd` sets the password using `echo "$username:$password"`.
- `chage -d 0` forces the user to reset their password on the first login.

> Summary:
> This script reads a CSV file, creates users with the specified passwords, and ensures they are prompted to change their password at first login — a great example of secure onboarding automation.

---

---

# 11. Service Health Monitor

## Question
You are asked to monitor multiple services like `nginx`, `sshd`, and `docker`.

**Task:**
- Write a shell script that checks the status of each service.
- If a service is stopped, attempt to restart it.
- Print a clearly formatted report.

### 📝 Short Explanation
This tests your ability to write robust service monitoring automation for multiple services, which is a common expectation in DevOps and SRE roles.

## ✅ Answer

### 🖥️ Shell Script: `multi_service_monitor.sh`

```bash
#!/bin/bash

# List of services to monitor
services=("nginx" "sshd" "docker")

# Report Header
echo "-----------------------------------"
echo "  Service Health Check Report"
echo "-----------------------------------"

# Loop through services
for service in "${services[@]}"; do
  if systemctl is-active --quiet "$service"; then
    echo "$service is ✅ RUNNING"
  else
    echo "$service is ❌ STOPPED"
    echo ""
    echo "Attempting to restart $service..."

    systemctl restart "$service" &> /dev/null

    # Check if restart was successful
    if systemctl is-active --quiet "$service"; then
      echo "$service has been ✅ restarted successfully."
    else
      echo "❌ Failed to restart $service. Manual intervention needed."
    fi
  fi
  echo "-----------------------------------"
done
```

---

### ✅ Example Output (if `docker` is down):

```
-----------------------------------
  Service Health Check Report
-----------------------------------
nginx is ✅ RUNNING
-----------------------------------
sshd is ✅ RUNNING
-----------------------------------
docker is ❌ STOPPED

Attempting to restart docker...
docker has been ✅ restarted successfully.
-----------------------------------
```

---

### 📘 Detailed Explanation

- **`services=(...)`**: An array of services to monitor.
- **`systemctl is-active`**: Checks if a service is running.
- **`systemctl restart`**: Tries to restart the service if it's not active.
- **Conditional Restart Check**: After restarting, the script confirms whether the service started successfully.
- **Output Formatting**: Clean section dividers and emojis provide clarity in console or logs.

---

### ⏰ Optional Cron Usage
To run every 10 minutes:

```bash
*/10 * * * * /path/to/multi_service_monitor.sh >> /var/log/service_check.log 2>&1
```

> Summary:
> This script checks and restarts critical services like `nginx`, `sshd`, and `docker`, and reports the status clearly. It's a lightweight way to keep essential services alive without external monitoring tools.

---

---

# 12. Delete Files Larger Than 100MB

## Question
How do you find and delete files larger than 100MB from a given directory?

### 📝 Short Explanation
This question tests your ability to manage disk space and clean up large files using command-line tools — a frequent task for DevOps and Linux admins.

## ✅ Answer

### 🖥️ Command (Using `find` and `-exec`)

```bash
find /path/to/directory -type f -size +100M -exec rm -f {} \;
```

---

### 📘 Detailed Explanation

#### 🔍 Breakdown:
- `find`: Linux command to search for files and directories.
- `/path/to/directory`: Replace with your target path (e.g., `/var/log`, `/tmp`).
- `-type f`: Limits results to files only.
- `-size +100M`: Matches files larger than 100 megabytes.
- `-exec rm -f {} \;`: Deletes each matched file:
  - `{}` is replaced by the filename.
  - `\;` ends the `-exec` command.

---

### ✅ Preview Without Deleting (Dry Run)

If you just want to see the files that would be deleted:

```bash
find /path/to/directory -type f -size +100M -exec ls -lh {} \;
```

This prints the size and path of each file over 100MB.

---

### ⚠️ Best Practices
- Always dry-run before deleting anything in production.
- Consider logging deletions or compressing instead of deleting if space permits.
- Automate safely with cron jobs for specific paths, e.g., `/tmp` or `/var/cache`.

> Summary:
> Use `find` with `-size +100M` and `-exec rm` to clean up oversized files and free up space. Always preview first to avoid accidental deletion of important files.

---

---

# 13. Users Logged In Today

## Question
How do you get the list of users who logged into the system today?

### 📝 Short Explanation
This tests your understanding of Linux log inspection and user activity tracking — helpful in auditing, troubleshooting, and system monitoring.

## ✅ Answer

### 🖥️ Command

```bash
last | grep "$(date '+%a %b %e')" | awk '{print $1}' | sort | uniq
```

---

### 📘 Detailed Explanation

#### 🔍 Breakdown:
- `last`: Displays recent login history from `/var/log/wtmp`.
- `date '+%a %b %e'`:
  - Outputs today's date in the format used by `last`, e.g., `Thu Jun 13`.
  - `%a` = abbreviated weekday, `%b` = abbreviated month, `%e` = day of month (with space-padding).
- `grep "$(date ...)"`: Filters only login entries for today.
- `awk '{print $1}'`: Extracts the usernames from the matched lines.
- `sort | uniq`: Removes duplicates to show unique users who logged in today.

---

### ✅ Example Output
```
ubuntu
admin
deploy
```

These are users who successfully logged in on the current date.

---

### 🧠 Bonus Tip
If your system has rotated or missing wtmp logs, this command might show no results. You can verify with:
```bash
ls -lh /var/log/wtmp
```

Or check journal logs:
```bash
journalctl --since today | grep 'session opened'
```

> Summary:
> This command helps you identify which users logged in today — useful for basic auditing, usage tracking, or verifying automated logins.

---

---

# 14. Website Does Not Load

## Question
Your website is not loading.

**Task:**
Describe the step-by-step investigation process to identify and fix the issue.

### 📝 Short Explanation
This is a high-pressure but common scenario that tests your ability to troubleshoot full-stack issues — from DNS and networking to web server and app code.

## ✅ Answer

Start from **external checks** and move inward, layer by layer:

---

### 🧭 1. **Is the site down for everyone or just me?**
Use:
```bash
curl -I https://yourdomain.com
ping yourdomain.com
```
Or check with [https://downforeveryoneorjustme.com](https://downforeveryoneorjustme.com)

---

### 🌐 2. **DNS Resolution**
```bash
dig yourdomain.com
nslookup yourdomain.com
```
✅ Expect to get the correct IP.
❌ No IP? Check DNS settings in Route53 (AWS) or other DNS provider.

---

### 🔁 3. **Is the domain routing to the correct server?**
Compare:
```bash
curl -v https://yourdomain.com
```
with server IP. If misrouted, verify **DNS records**, **load balancer config**, or **CDN rules**.

---

### 📡 4. **Network/Firewall Check**
- From your system:
```bash
telnet yourdomain.com 443
nc -zv yourdomain.com 80
```
- Check **security groups**, **firewalls**, or **NACLs** in cloud if ports are blocked.

---

### 🖥️ 5. **Is the Web Server running?**
SSH into your instance and check:

```bash
sudo systemctl status nginx
sudo systemctl status apache2
```

❌ If it's down, restart:
```bash
sudo systemctl restart nginx
```

---

### 🧱 6. **Check Application Logs**
Look at logs for crash reports or errors:
- `/var/log/nginx/error.log`
- `/var/log/httpd/error_log`
- App logs: `app.log`, `stderr`, etc.

---

### 💾 7. **Check Disk/Memory/CPU**
```bash
df -h
top or htop
free -m
```
✅ Ensure the server isn't unresponsive due to resource exhaustion.

---

### 🔄 8. **Check Backend Services (DB, Cache, etc.)**
Your web app may be up, but failing due to:
- MySQL/Postgres down
- Redis/Memcached connection error
- App server crashes

---

### 🔐 9. **SSL Certificate Issues**
```bash
curl -Iv https://yourdomain.com
```
Look for:
```
SSL certificate problem
```

If expired, renew via Let's Encrypt or your CA.

---

### 🧪 10. **Rollback or Revert**
If the issue started after a deploy:
- Rollback to the previous working build.
- Use:
```bash
kubectl rollout undo deployment your-deployment
```
or redeploy old version via your CI/CD.

---

### 🧠 Bonus Tip:
- Always check **uptime monitoring**, **alerting tools**, or **dashboards**.
- Build a **runbook** for your team for repeated scenarios.

> Summary:
> Troubleshooting a website that won't load requires a methodical approach — from DNS to server and app. Think layers: DNS → Network → Web Server → Application → Infrastructure → Dependencies.

---

---

# 15. Remove First and Last Lines from a File

## Question
How do you remove the first and last line of a file using `sed`?

### 📝 Short Explanation
This task checks your understanding of line-addressing in `sed`, which is a powerful stream editor used in shell scripting for text manipulation.

## ✅ Answer

### 🖥️ Command:

```bash
sed '1d; $d' filename.txt
```

---

### 📘 Detailed Explanation

#### 🔍 Breakdown:
- `sed`: Stream editor used to process text line-by-line.
- `'1d'`: Deletes the **first line** (`1` = line number).
- `'$d'`: Deletes the **last line** (`$` = end of file).
- `filename.txt`: The file to be processed.

Together, the command says:
> "Delete the first line **and** the last line from `filename.txt`."

---

### ✅ Example:

If `file.txt` contains:
```
Line 1
Line 2
Line 3
Line 4
Line 5
```

Command:
```bash
sed '1d; $d' file.txt
```

Output:
```
Line 2
Line 3
Line 4
```

---

### 🧠 Bonus Tip:
If you want to **save the result to a new file**:
```bash
sed '1d; $d' file.txt > trimmed.txt
```

> Summary:
> The `sed '1d; $d'` command is an efficient way to remove both the first and last lines of a text file using line number and end-of-file markers.

---

---

# 16. Shell Scripts

The following are standalone shell scripts included in the `03-linux/` folder of the repository.

---

## create_users.sh

```bash
#!/bin/bash

FILE="users.csv"

tail -n +2 "$FILE" | while read line
do
        username=$(echo $line | cut -d',' -f1)
        password=$(echo $line | cut -d',' -f2)


        useradd "$username"

        echo "$username:$password" | chpasswd

        echo "Created the user: $username"

done
```

---

## log_rotation.sh

```bash
#!/bin/bash

########################
# Your application writes logs to /var/log/myapp/. You want to:
#    Compress logs older than 7 days
#    Delete logs older than 30 days
#    Automate it via a daily cron job
#########################

# Directory where logs are stored
LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

# Ensure the log directory exists
if [ ! -d "$LOG_DIR" ]; then
    echo "[$(date)] ERROR: Log directory $LOG_DIR does not exist!" >> "$LOG_FILE"
    exit 1
fi

# Compress logs older than 7 days (but newer than 30)
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

# Optional: Delete uncompressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.log" -mtime +30 -exec rm -f {} \; -exec echo "[$(date)] Deleted (uncompressed): {}" >> "$LOG_FILE" \;

# Done
echo "[$(date)] Log rotation completed successfully." >> "$LOG_FILE"
```

---

## monitor_services.sh

```bash
#!/bin/bash

# List of services to monitor
services=("nginx" "sshd" "docker")

echo "-----------------------------------"
echo " Service Health Check Report"
echo "-----------------------------------"

# Loop through each service
for service in "${services[@]}"; do
    # Check service status
    if systemctl is-active --quiet "$service"; then
        echo "$service is ✅ RUNNING"
    else
        echo "$service is ❌ STOPPED"

        # Optional: Try to restart the service
        echo "Attempting to restart $service..."
        sudo systemctl restart "$service"

        # Re-check status
        if systemctl is-active --quiet "$service"; then
            echo "$service has been ✅ restarted successfully."
        else
            echo "⚠️  Failed to restart $service. Check logs."
        fi
    fi
    echo "-----------------------------------"
done
```

---

