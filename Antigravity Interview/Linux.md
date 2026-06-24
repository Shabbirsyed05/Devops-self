# 🐧 Linux Interview Guide — Complete Q&A + Shell Scripts

> 📚 Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `03-linux/`
> 🎯 Purpose: Master Linux troubleshooting, commands & shell scripting for DevOps interviews

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| [1](#-1-10-most-used-linux-commands) | 10 Most Used Linux Commands |
| [2](#-2-restore-a-lost-pem-file) | Restore a Lost PEM File |
| [3](#-3-var-folder-is-full) | /var Folder is Full |
| [4](#-4-server-slow--high-cpu-usage) | Server Slow — High CPU Usage |
| [5](#-5-app-returns-connection-refused) | App Returns Connection Refused (NGINX) |
| [6](#-6-ssh-stopped-working) | SSH Stopped Working |
| [7](#-7-list-log-files-older-than-7-days) | List Log Files Older Than 7 Days |
| [8](#-8-remove-log-files-older-than-30-days) | Remove Log Files Older Than 30 Days |
| [9](#-9-cronjob-log-rotation-automation) | Cronjob Log Rotation Script |
| [10](#-10-create-users-from-csv) | Create Users from CSV |
| [11](#-11-service-health-monitor) | Service Health Monitor Script |
| [12](#-12-delete-files-larger-than-100mb) | Delete Files Larger Than 100MB |
| [13](#-13-list-users-logged-in-today) | Users Logged In Today |
| [14](#-14-website-does-not-load) | Website Does Not Load |
| [15](#-15-remove-first-and-last-lines-sed) | Remove First & Last Lines with sed |
| [Scripts](#-shell-scripts-reference) | Shell Scripts Reference |
| [Cheatsheet](#-master-cheatsheet) | Master Cheatsheet |

---

---

# 🛠️ 1. 10 Most Used Linux Commands

**Q: What are 10 Linux commands you use daily? (Excluding ls, cd, pwd)**

---

### 1. `tail -f` — Real-time log monitoring
```bash
tail -f /var/log/nginx/error.log
# Use -f to follow live — essential for debugging running apps
```

### 2. `grep` — Search through logs/output
```bash
grep -i "timeout" /var/log/app.log
grep -rn "ERROR" /var/log/         # recursive search across all log files
```

### 3. `systemctl` — Service control
```bash
systemctl status nginx
systemctl restart docker
systemctl enable nginx    # auto-start on boot
```

### 4. `journalctl` — Systemd service logs
```bash
journalctl -u docker.service -f       # follow docker logs
journalctl --since "1 hour ago"       # last hour's logs
```

### 5. `ps aux | grep` — Find processes
```bash
ps aux | grep nginx
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head   # sorted by CPU
```

### 6. `df -h` / `du -sh` — Disk usage
```bash
df -h             # disk space per filesystem
du -sh *          # folder sizes in current directory
du -sh /var/*     # find what's filling /var
```

### 7. `chmod` / `chown` — Permissions
```bash
chmod +x deploy.sh
chmod 600 key.pem       # required for SSH keys
chown ubuntu:ubuntu script.sh
```

### 8. `find` — Locate files
```bash
find /var/log -name "*.log" -mtime +7     # logs older than 7 days
find / -type f -size +100M               # files larger than 100MB
```

### 9. `curl` — Test endpoints
```bash
curl -I http://localhost:8080             # check HTTP headers
curl -v https://api.example.com           # verbose — shows SSL, headers
```

### 10. `rsync` — File sync/backup
```bash
rsync -avz /app/ user@server:/backup/    # sync with progress, verbose
# Faster and more reliable than scp for large directories
```

---

---

# 🔑 2. Restore a Lost PEM File

**Q: Can you restore a lost PEM file? If not, how do you regain EC2 access?**

> **Short answer:** PEM files are **never stored on AWS** — cannot be restored. But you can regain access via an EBS rescue process.

### Recovery Workflow (7 Steps)

```bash
# Step 1: Create a new key pair
aws ec2 create-key-pair --key-name new-key \
  --query 'KeyMaterial' --output text > new-key.pem
chmod 400 new-key.pem

# Step 2: Stop the affected instance
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxx
```

```
# Steps 3-4 (AWS Console):
3. EC2 → Volumes → find root volume (/dev/xvda) → Detach
4. Attach that volume to a working (rescue) instance as /dev/sdf
```

```bash
# Step 5: SSH into rescue instance, mount the broken volume
sudo mkdir /mnt/recovery
sudo mount /dev/xvdf1 /mnt/recovery

# Add new public key to authorized_keys on the broken volume
sudo nano /mnt/recovery/home/ec2-user/.ssh/authorized_keys
# Paste the contents of new-key.pub
```

```
# Steps 6-7 (AWS Console):
6. Unmount, detach volume from rescue instance
7. Re-attach as /dev/xvda to original instance → Start instance
```

```bash
# Connect with new key
ssh -i new-key.pem ec2-user@<public-ip>  ✅
```

**Prevention:**
- Backup PEM in a password manager (1Password, Bitwarden)
- Create a secondary user with a different key
- Use EC2 Instance Connect for emergency browser-based access

### 🎙️ Answer
> *"PEM files cannot be recovered — AWS doesn't store the private key after generation. To regain access, I stop the instance, detach the root EBS volume, attach it to a rescue EC2 as a secondary volume, mount it, and edit the `authorized_keys` file to add a new public key. Then I re-attach the volume to the original instance and SSH in with the new key. This is why I always back up PEM files and add a secondary user for redundant access."*

---

---

# 💾 3. /var Folder is Full

**Q: /var is 90% full. What are your next steps?**

### Step-by-Step Investigation

```bash
# Step 1: Find what's consuming space inside /var
sudo du -sh /var/* | sort -hr | head -10
# Usually culprits: /var/log, /var/lib/docker, /var/cache

# Step 2: Clean journal logs (most common cause)
sudo journalctl --vacuum-size=200M     # keep only 200MB of journal
sudo journalctl --vacuum-time=7d       # keep only last 7 days

# Step 3: Remove rotated/compressed old logs
sudo rm -rf /var/log/*.gz
sudo rm -rf /var/log/*.[0-9]
# OR truncate a specific large log without deleting it:
sudo truncate -s 0 /var/log/syslog

# Step 4: Clear package manager cache
sudo apt clean          # Debian/Ubuntu
sudo yum clean all      # RHEL/CentOS

# Step 5: Docker artifacts (if server runs containers)
docker system df            # see what's taking space
docker system prune -a      # ⚠️ removes unused images/volumes/containers
```

**Common causes of /var filling up:**
- Verbose debug logging (app crash loops, failed cron jobs)
- Docker image layers accumulating
- Orphaned package cache
- Email spools or core dumps

**Prevention:** Set up `logrotate`, Prometheus disk alerts, or cron cleanup jobs.

### 🎙️ Answer
> *"I start with `du -sh /var/* | sort -hr` to find the biggest consumer. Usually it's /var/log or /var/lib/docker. For logs, I vacuum journald, remove rotated .gz files, and set up logrotate. For Docker, `docker system prune` removes unused layers. Then I add monitoring so we get alerted before it hits 90% again."*

---

---

# ⚡ 4. Server Slow — High CPU Usage

**Q: Linux server is slow due to high CPU. How do you fix it?**

### Systematic Diagnosis

```bash
# Step 1: Check load average (compare to CPU count)
uptime
# load average: 6.02, 4.33, 2.89
# If load > number of CPU cores → overloaded

nproc    # check how many cores you have

# Step 2: Identify CPU-heavy processes
top -o %CPU      # interactive, sorted by CPU
htop             # colorful, user-friendly alternative

# Step 3: Drill down with ps
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head -20

# Step 4: Check per-process CPU over time
pidstat -u 1 5   # sample every 1 second, 5 times

# Step 5: Check logs for the cause
journalctl -xe
tail -f /var/log/syslog

# Step 6: Take action
kill -9 <pid>                  # force kill runaway process
systemctl restart <service>    # restart misbehaving service

# Throttle CPU without killing (temporary relief)
renice -n 15 -p <pid>         # lower priority
cpulimit -p <pid> -l 30       # limit to 30% CPU
```

**Common root causes:**
- Cron script looping due to bad condition
- Java app in infinite recursion / GC thrash
- Docker container with no CPU limits scraping
- Misconfigured antivirus or audit daemon

**Prevention:** CPU limits in containers (`--cpus`), Prometheus alerts at 80% for 5 min.

### 🎙️ Answer
> *"First `uptime` to check load average vs CPU count — if consistently higher, the server is overloaded. Then `top` or `htop` to identify the offending process by PID. I drill down with `ps` and `pidstat`, check app logs for the cause (crash loops, bad queries), then either kill/restart the process or tune it with `renice`. Long-term, I add CPU limits to containers and set up alerting."*

---

---

# 🔌 5. App Returns Connection Refused (NGINX)

**Q: Application on NGINX returns "Connection Refused". How do you fix it?**

> "Connection Refused" = server rejected the TCP handshake. Not a 4xx/5xx — the app isn't even answering.

### Systematic Troubleshooting

```bash
# Step 1: Reproduce
curl -I http://localhost
# curl: (7) Failed to connect to localhost port 80: Connection refused

# Step 2: Is NGINX actually running?
sudo systemctl status nginx
sudo journalctl -u nginx -xe    # if it failed to start, see why

# Step 3: Is NGINX listening on the expected port?
sudo netstat -tulnp | grep nginx
ss -tuln | grep :80
# No output = NGINX not listening on port 80

# Step 4: Test NGINX config for syntax errors
sudo nginx -t

# Step 5: Is the app backend running?
sudo netstat -tulnp | grep 5000    # check if app is on proxy_pass port
curl http://localhost:5000          # test backend directly

# Step 6: Check firewall / security groups
sudo ufw status
sudo iptables -L
# Cloud: check AWS Security Group inbound rules for port 80/443

# Step 7: SELinux (if applicable)
sudo getenforce    # if Enforcing → may block NGINX from binding ports
```

**Check NGINX proxy config:**
```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;   # app must be running on 5000
    }
}
```

**Common causes:**
- App crashed and isn't listening on its port
- Wrong port in `proxy_pass`
- Port 80/443 blocked by firewall or security group
- NGINX not restarted after config change

### 🎙️ Answer
> *"Connection Refused means the server rejected the TCP handshake entirely — not an app error. I check: is NGINX running (`systemctl status nginx`)? Is it listening (`ss -tuln | grep :80`)? Is the backend app actually running on the proxy_pass port (`netstat -tulnp | grep 5000`)? Is the port open in the firewall/security group? Usually it's one of: NGINX not restarted after config change, the app backend crashed, or a wrong port in proxy_pass."*

---

---

# 🔒 6. SSH Stopped Working

**Q: SSH to an EC2 instance stopped working. How do you troubleshoot?**

### Diagnosis by Error Message

| Error | Likely Cause |
|---|---|
| `Permission denied (publickey)` | Wrong key, wrong user, or authorized_keys corrupted |
| `Connection refused` | SSH service down or port 22 blocked |
| `Operation timed out` | Network issue — security group, NACL, routing |

### Systematic Steps

```bash
# Step 1: Check instance status
aws ec2 describe-instance-status --instance-ids <id>
ping <public-ip>

# Step 2: Verify Security Group — port 22 open for your IP
# AWS Console: EC2 → Security Groups → Inbound → SSH port 22

# Step 3: Confirm correct username
ssh -i key.pem ec2-user@<ip>      # Amazon Linux
ssh -i key.pem ubuntu@<ip>        # Ubuntu
ssh -i key.pem centos@<ip>        # CentOS
# ❌ Never use 'root' for cloud AMIs by default

# Step 4: Check PEM file permissions
chmod 400 key.pem    # must be 400, not 644 or 600

# Step 5: Verify public IP (elastic vs dynamic)
# Dynamic public IP changes on stop/start — check current IP in console

# Step 6: Try EC2 Instance Connect (Amazon Linux — browser-based)
# AWS Console → EC2 → Connect → EC2 Instance Connect

# Step 7: From inside (via EC2 Connect), check SSH service
sudo systemctl status sshd
tail -n 50 /var/log/auth.log    # failed login attempts + errors
sudo nano /etc/ssh/sshd_config  # check PasswordAuthentication, Port
```

**Rescue mode (last resort):**
```
Stop instance → Detach root volume → Attach to rescue EC2
→ Mount → Fix authorized_keys or sshd_config → Reattach → Start
```

### 🎙️ Answer
> *"The error message tells you where to look first. 'Timed out' = network (security group port 22 blocked). 'Connection refused' = SSH service down. 'Permission denied' = key or username mismatch. I check instance health, security group rules, correct username for the AMI, and PEM file permissions (must be 400). If everything looks fine but SSH is broken, EC2 Instance Connect lets me get in via browser to check sshd status and logs."*

---

---

# 📋 7. List Log Files Older Than 7 Days

**Q: How do you find log files older than 7 days in /var/log?**

```bash
# List files modified more than 7 days ago
find /var/log -type f -mtime +7

# With size and timestamp
find /var/log -type f -mtime +7 -exec ls -lh {} \;

# Understand the flags:
# -type f     = files only (not directories)
# -mtime +7   = modified MORE than 7 days ago (+7 = strictly older)
# -mtime -7   = modified LESS than 7 days ago (newer)
# -mtime  7   = modified exactly 7 days ago
```

> ⚠️ Always list before deleting — review what will be affected.

---

---

# 🗑️ 8. Remove Log Files Older Than 30 Days

**Q: Remove log files older than 30 days using `-exec`.**

```bash
# Dry run first (list files that would be deleted)
find /var/log -type f -name "*.log" -mtime +30 -exec ls -lh {} \;

# Actual deletion
sudo find /var/log -type f -name "*.log" -mtime +30 -exec rm -f {} \;

# Breakdown:
# -name "*.log"    = only .log files
# -mtime +30       = older than 30 days
# -exec rm -f {}   = force delete each match
# \;               = end of -exec command
```

**Faster alternative using `-delete`:**
```bash
sudo find /var/log -type f -name "*.log" -mtime +30 -delete
```

> `-delete` is faster than `-exec rm` but cannot be combined with other exec actions.

---

---

# 🔄 9. Cronjob Log Rotation Automation

**Q: Write a script that compresses logs >7 days and deletes logs >30 days. Run daily via cron.**

### `log_rotation.sh`

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

# Validate directory exists
if [ ! -d "$LOG_DIR" ]; then
    echo "[$(date)] ERROR: Log directory $LOG_DIR does not exist!" >> "$LOG_FILE"
    exit 1
fi

# Compress .log files older than 7 days but newer than 30 days
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" \
    -exec gzip {} \; \
    -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 \
    -exec rm -f {} \; \
    -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

# Delete uncompressed logs older than 30 days (safety net)
find "$LOG_DIR" -type f -name "*.log" -mtime +30 \
    -exec rm -f {} \; \
    -exec echo "[$(date)] Deleted (uncompressed): {}" >> "$LOG_FILE" \;

echo "[$(date)] Log rotation completed successfully." >> "$LOG_FILE"
```

**Set up daily cron job:**
```bash
crontab -e
# Add this line (runs daily at 2 AM):
0 2 * * * /opt/scripts/log_rotation.sh >> /var/log/cron_cleanup.log 2>&1
```

**Key logic:**
- `! -name "*.gz"` — don't re-compress already compressed files
- `-mtime +7 -mtime -30` — between 7 and 30 days old → compress only
- `-mtime +30` — older than 30 days → delete

---

---

# 👥 10. Create Users from CSV

**Q: Read a CSV of usernames/passwords, create each user, force password change on first login.**

### Input CSV (`users.csv`)
```csv
username,password
alice,Password@123
bob,Secure@456
carol,DevOps@789
```

### `create_users.sh`

```bash
#!/bin/bash

INPUT="users.csv"

if [[ ! -f "$INPUT" ]]; then
    echo "CSV file not found!"
    exit 1
fi

# Skip header row, read each line
tail -n +2 "$INPUT" | while IFS=',' read -r username password; do

    # Skip if user already exists
    if id "$username" &>/dev/null; then
        echo "User '$username' already exists. Skipping..."
        continue
    fi

    # Create user
    useradd "$username"

    # Set password (chpasswd reads username:password from stdin)
    echo "${username}:${password}" | chpasswd

    # Force password change on first login
    chage -d 0 "$username"

    echo "✅ Created user: $username"
done
```

```bash
chmod +x create_users.sh
sudo ./create_users.sh
```

**Key commands:**
- `IFS=',' read -r` — splits each CSV line on comma
- `chpasswd` — sets password from `user:password` stdin format
- `chage -d 0` — sets last password change to epoch 0, forcing immediate change on first login

### 🎙️ Answer
> *"I use `tail -n +2` to skip the CSV header, then `IFS=',' read -r` to split each line into username and password variables. `useradd` creates the account, `chpasswd` sets the password securely via stdin, and `chage -d 0` forces a password reset on first login. I also check if the user already exists with `id` to make the script idempotent."*

---

---

# 🏥 11. Service Health Monitor

**Q: Write a script that checks nginx, sshd, docker — restarts if stopped, prints a report.**

### `monitor_services.sh`

```bash
#!/bin/bash

services=("nginx" "sshd" "docker")

echo "-----------------------------------"
echo "   Service Health Check Report"
echo "-----------------------------------"

for service in "${services[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "$service is ✅ RUNNING"
    else
        echo "$service is ❌ STOPPED"
        echo "Attempting to restart $service..."

        sudo systemctl restart "$service" &>/dev/null

        if systemctl is-active --quiet "$service"; then
            echo "$service has been ✅ restarted successfully."
        else
            echo "⚠️  Failed to restart $service. Manual intervention needed."
        fi
    fi
    echo "-----------------------------------"
done
```

**Example output:**
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

**Schedule via cron (every 10 minutes):**
```bash
*/10 * * * * /opt/scripts/monitor_services.sh >> /var/log/service_check.log 2>&1
```

---

---

# 📦 12. Delete Files Larger Than 100MB

**Q: How do you find and delete files larger than 100MB from a directory?**

```bash
# Dry run first — list files over 100MB
find /path/to/directory -type f -size +100M -exec ls -lh {} \;

# Delete them
find /path/to/directory -type f -size +100M -exec rm -f {} \;

# Faster alternative
find /path/to/directory -type f -size +100M -delete

# Size units: k=kilobytes, M=megabytes, G=gigabytes
# +100M = strictly greater than 100MB
# Real example — clean up large files in /tmp:
find /tmp -type f -size +100M -exec rm -f {} \;
```

> Always run dry-run with `ls -lh` before deleting on production systems.

---

---

# 👤 13. List Users Logged In Today

**Q: How do you get the list of users who logged into the system today?**

```bash
# List all users who logged in today
last | grep "$(date '+%a %b %e')" | awk '{print $1}' | sort | uniq

# Breakdown:
# last                     → reads /var/log/wtmp login history
# date '+%a %b %e'         → today's date in format: "Thu Jun 13"
# grep "..."               → filter only today's entries
# awk '{print $1}'         → extract usernames (first column)
# sort | uniq              → deduplicate

# Example output:
# ubuntu
# admin
# deploy
```

**Alternative using journalctl:**
```bash
journalctl --since today | grep 'session opened' | awk '{print $NF}'
```

**Check if wtmp log exists:**
```bash
ls -lh /var/log/wtmp    # if missing or rotated, last may show no output
```

---

---

# 🌐 14. Website Does Not Load

**Q: Your website is not loading. Describe the step-by-step investigation.**

### Troubleshooting Flow (Layer by Layer)

```bash
# Step 1: Is it down for everyone or just you?
curl -I https://yourdomain.com
# Check: https://downforeveryoneorjustme.com

# Step 2: DNS resolution
dig yourdomain.com
nslookup yourdomain.com
# Should return the correct server IP

# Step 3: Can you reach the server at all?
ping <server-ip>
telnet yourdomain.com 443
nc -zv yourdomain.com 80

# Step 4: Check web server status (on the server)
sudo systemctl status nginx
sudo systemctl status apache2
sudo journalctl -u nginx -xe

# Step 5: Check application logs
tail -f /var/log/nginx/error.log
tail -f /var/log/app/app.log

# Step 6: Resources (is the server dead?)
df -h          # disk full?
free -m        # OOM (out of memory)?
top            # CPU 100%?

# Step 7: Backend services up?
systemctl status mysql
systemctl status redis
curl http://localhost:5000   # test app backend directly

# Step 8: SSL certificate valid?
curl -Iv https://yourdomain.com 2>&1 | grep -i "SSL\|expire\|certificate"

# Step 9: Firewall / Security Group
sudo ufw status
# Cloud: check AWS Security Group — ports 80 and 443 open?

# Step 10: Recent deployment broke it?
kubectl rollout undo deployment your-deployment    # Kubernetes rollback
# Or redeploy previous version via CI/CD pipeline
```

**Investigation order:**
```
DNS → Network/Firewall → Web Server → Application → Resources → Backend → SSL → Recent change
```

### 🎙️ Answer
> *"I work from the outside in. First check if DNS resolves correctly. Then test network reachability — can I ping and telnet to port 80/443? Then SSH in and check if the web server is running. Check app logs for errors. Verify resources (disk, memory, CPU). Test backend services. Check SSL expiry. If it started after a deploy, roll back. The goal is to rule out each layer systematically."*

---

---

# ✂️ 15. Remove First and Last Lines (sed)

**Q: How do you remove the first and last line of a file using sed?**

```bash
# Remove first and last line
sed '1d; $d' filename.txt

# Breakdown:
# 1d   = delete line number 1 (first line)
# $d   = delete last line ($ = end of file address)
# ;    = separator between two sed commands

# Example:
cat file.txt
# Line 1
# Line 2
# Line 3
# Line 4
# Line 5

sed '1d; $d' file.txt
# Line 2
# Line 3
# Line 4

# Save to a new file
sed '1d; $d' file.txt > trimmed.txt

# Edit in-place (modify the original)
sed -i '1d; $d' file.txt
```

**Other useful sed patterns:**
```bash
sed -n '2,4p' file.txt        # print only lines 2–4
sed 's/foo/bar/g' file.txt    # replace all occurrences of foo with bar
sed '/^$/d' file.txt          # remove blank lines
```

---

---

# 📜 Shell Scripts Reference

### `create_users.sh` (simplified version)

```bash
#!/bin/bash

FILE="users.csv"

tail -n +2 "$FILE" | while read line; do
    username=$(echo $line | cut -d',' -f1)
    password=$(echo $line | cut -d',' -f2)
    useradd "$username"
    echo "$username:$password" | chpasswd
    echo "Created the user: $username"
done
```

---

### `log_rotation.sh`

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

if [ ! -d "$LOG_DIR" ]; then
    echo "[$(date)] ERROR: $LOG_DIR does not exist!" >> "$LOG_FILE"
    exit 1
fi

find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" \
    -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

find "$LOG_DIR" -type f -name "*.gz" -mtime +30 \
    -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

find "$LOG_DIR" -type f -name "*.log" -mtime +30 \
    -exec rm -f {} \; -exec echo "[$(date)] Deleted (uncompressed): {}" >> "$LOG_FILE" \;

echo "[$(date)] Log rotation completed." >> "$LOG_FILE"
```

---

### `monitor_services.sh`

```bash
#!/bin/bash

services=("nginx" "sshd" "docker")

echo "-----------------------------------"
echo " Service Health Check Report"
echo "-----------------------------------"

for service in "${services[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "$service is ✅ RUNNING"
    else
        echo "$service is ❌ STOPPED"
        echo "Attempting to restart $service..."
        sudo systemctl restart "$service"
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

---

# 📌 Master Cheatsheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           LINUX INTERVIEW MASTER CHEATSHEET                          ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  MONITORING & DIAGNOSIS:                                             ║
║  tail -f /var/log/X.log  → real-time log                            ║
║  top / htop              → live process CPU/RAM                      ║
║  ps aux --sort=-%cpu     → top CPU processes                         ║
║  uptime                  → load average vs nproc (CPU count)        ║
║  df -h / du -sh *        → disk usage                               ║
║  free -h                 → RAM usage                                 ║
║  journalctl -u svc -f    → systemd service logs live                ║
║                                                                      ║
║  FIND COMMAND:                                                       ║
║  -type f         = files only                                        ║
║  -mtime +7       = older than 7 days | -mtime -7 = newer            ║
║  -size +100M     = larger than 100MB                                 ║
║  -exec rm {} \;  = delete each match                                 ║
║  -delete         = faster deletion (no -exec needed)                 ║
║                                                                      ║
║  PROCESS MANAGEMENT:                                                 ║
║  kill -9 PID     = force kill | kill -3 PID = Java thread dump     ║
║  renice -n 15 -p PID = lower CPU priority                           ║
║  systemctl is-active --quiet svc = check status in scripts          ║
║                                                                      ║
║  LOST PEM FILE:                                                      ║
║  Cannot restore → Stop EC2 → Detach root EBS → Mount on rescue     ║
║  → Edit authorized_keys → Reattach → Start → SSH with new key      ║
║                                                                      ║
║  /var FULL:                                                          ║
║  du -sh /var/* | sort -hr | head → find biggest dirs                ║
║  journalctl --vacuum-size=200M → clean journal logs                  ║
║  docker system prune -a → clean unused Docker artifacts              ║
║                                                                      ║
║  CONNECTION REFUSED:                                                 ║
║  systemctl status nginx → netstat -tulnp → nginx -t → SG ports     ║
║                                                                      ║
║  SSH TROUBLESHOOT:                                                   ║
║  Timeout=SG/network | Refused=SSH down | Denied=key/user wrong      ║
║  chmod 400 key.pem | correct username per AMI                       ║
║                                                                      ║
║  SHELL SCRIPT PATTERNS:                                              ║
║  tail -n +2 file.csv       → skip CSV header row                    ║
║  IFS=',' read -r a b       → parse comma-separated values           ║
║  chpasswd                  → set password from username:pass stdin   ║
║  chage -d 0 user           → force password change on first login   ║
║  systemctl is-active --quiet svc → returns 0 if running             ║
║                                                                      ║
║  SED TRICKS:                                                         ║
║  sed '1d; $d' file         → remove first and last lines            ║
║  sed 's/foo/bar/g' file    → replace all occurrences                ║
║  sed '/^$/d' file          → remove blank lines                      ║
║  sed -n '2,4p' file        → print only lines 2–4                   ║
║                                                                      ║
║  USERS LOGGED IN TODAY:                                              ║
║  last | grep "$(date '+%a %b %e')" | awk '{print $1}' | sort|uniq  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Always say "dry-run first" before deletion | Say "I'd just delete the files" |
| Mention `journalctl -xe` for service failures | Just say "check the logs" |
| Use `systemctl is-active --quiet` in scripts | Use `ps aux | grep` for service checks |
| Say "rotate the secret first" for SSH PEM question | Jump to technical steps without mentioning prevention |
| Mention monitoring + alerting as the long-term fix | Stop at the immediate fix |

---

## 📚 Resources

- 🔗 [devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)
- 🔗 [Linux `find` command manual](https://linux.die.net/man/1/find)
- 🔗 [sed manual](https://linux.die.net/man/1/sed)
- 🔗 [Cron expression reference](https://crontab.guru/)

---

> ⭐ **Star this repo** if it helped you prepare for your DevOps interview!
> 🔔 Paste the next topic's notes — I'll overwrite with only those!
