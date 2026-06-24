# 🌐 Networking Interview Guide — Complete Q&A

> 📚 Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `04-networking/`
> 🎯 Purpose: Master networking concepts and troubleshooting for DevOps interviews

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| [1](#-1-what-is-dns) | What is DNS? |
| [2](#-2-osi-model-with-example) | OSI Model — Full Request Flow |
| [3](#-3-forward-proxy-vs-reverse-proxy) | Forward Proxy vs Reverse Proxy |
| [4](#-4-application-slowness-troubleshooting) | App Slowness Troubleshooting |
| [5](#-5-curl-works-with-ip-but-not-domain) | curl Works with IP but Not Domain |
| [6](#-6-502-bad-gateway) | 502 Bad Gateway |
| [7](#-7-00000-vs-127001) | 0.0.0.0 vs 127.0.0.1 |
| [8](#-8-public-vs-private-subnet) | Public vs Private Subnet |
| [9](#-9-convert-private-subnet-to-public) | Convert Private Subnet to Public |
| [Master Cheatsheet](#-master-cheatsheet) | Master Cheatsheet |

---

---

# 🌍 1. What is DNS?

**Q: What is DNS? Explain in simple terms.**

> **DNS = Domain Name System** — the internet's phonebook.

```
You type:     www.google.com
DNS converts: www.google.com → 142.250.64.100 (IP)
Browser uses: 142.250.64.100 to connect to Google's server
```

**Why it's needed:**
- Humans remember names: `amazon.com`, `github.com`
- Computers need IPs: `192.0.2.1`, `140.82.112.4`
- DNS is the translator between the two

### How DNS Resolution Works (Step by Step)

```
1. You type: www.example.com
2. Browser checks → local DNS cache
3. Not cached → asks OS resolver
4. OS asks → ISP's DNS server (Recursive Resolver)
5. ISP asks → Root DNS server → TLD server (.com)
6. TLD points to → Authoritative DNS for example.com
7. Authoritative returns the IP: 93.184.216.34
8. IP cached locally → Browser connects → Page loads
```

**DNS Record Types (frequently asked):**

| Record | Purpose | Example |
|---|---|---|
| **A** | Domain → IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Domain → IPv6 address | `example.com → 2606:2800::1` |
| **CNAME** | Alias → another domain | `www.example.com → example.com` |
| **MX** | Mail server routing | `example.com → mail.example.com` |
| **TXT** | Verification / SPF | Google site verify |
| **NS** | Nameserver for domain | Who handles DNS for this domain |

### 🎙️ Interview Answer
> *"DNS is the internet's phonebook — it translates human-readable domain names like google.com into IP addresses like 142.250.64.100 that computers use to actually connect. When you type a URL, your browser checks its local cache first, then the OS, then the ISP's recursive resolver, which queries root → TLD → authoritative DNS servers until it gets the IP. In AWS, Route 53 is the managed DNS service."*

---

---

# 🔵 2. OSI Model with Example

**Q: Explain the complete flow of a browser request using the OSI Model.**

### The 7 Layers (Top → Bottom: Sending, Bottom → Top: Receiving)

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 7: APPLICATION    → HTTP/HTTPS request initiated         │
│  Layer 6: PRESENTATION   → SSL/TLS encryption, data format     │
│  Layer 5: SESSION        → Session ID created (keeps you logged in)│
│  Layer 4: TRANSPORT      → Data broken into segments, TCP/UDP   │
│  Layer 3: NETWORK        → IP addresses added, routing          │
│  Layer 2: DATA LINK      → MAC addresses, framing               │
│  Layer 1: PHYSICAL       → Bits transmitted via cable/Wi-Fi     │
└─────────────────────────────────────────────────────────────────┘
```

### Full Request Example: `https://example.com`

| Layer | What Happens |
|---|---|
| **L7 Application** | Browser creates HTTP GET request for `https://example.com` |
| **L6 Presentation** | TLS encryption applied — data scrambled for security |
| **L5 Session** | Session created — keeps you logged in across requests |
| **L4 Transport** | Data split into TCP segments. Ports added: Src=random, Dst=443 |
| **L3 Network** | IP header added: Src=192.168.1.5, Dst=93.184.216.34 |
| **L2 Data Link** | MAC addresses of router/switch added as Ethernet frame |
| **L1 Physical** | Electrical signals / light pulses sent over cable or Wi-Fi |

### At the Server (Reverse — Bottom to Top)

```
Physical signal arrives → Data Link unpacks frame → Network reads IP
→ Transport reassembles segments → Session managed
→ Presentation decrypts → Application processes HTTP request → Sends response
```

### Key Protocol Mapping

| Layer | Protocols |
|---|---|
| L7 Application | HTTP, HTTPS, FTP, SMTP, DNS, SSH |
| L4 Transport | TCP (reliable), UDP (fast, no guarantee) |
| L3 Network | IP (routing and addressing) |
| L2 Data Link | Ethernet, Wi-Fi (MAC addressing) |

**Before OSI even starts:**
```
1. DNS Resolution → google.com → 8.8.8.8 (must resolve first)
2. TCP 3-Way Handshake: SYN → SYN-ACK → ACK (connection established)
   THEN data transfer begins
```

### 🎙️ Interview Answer
> *"The OSI model breaks network communication into 7 layers. When you type https://example.com, Layer 7 creates the HTTP request, Layer 6 encrypts it with TLS, Layer 5 manages the session, Layer 4 breaks data into TCP segments and adds port numbers, Layer 3 adds source and destination IP addresses for routing, Layer 2 adds MAC addresses for local delivery, and Layer 1 transmits actual bits. Before this, DNS resolves the domain name and a TCP 3-way handshake establishes the connection."*

---

---

# 🔄 3. Forward Proxy vs Reverse Proxy

**Q: What is the difference between a Forward Proxy and a Reverse Proxy?**

```
Forward Proxy:
  Client → [Forward Proxy] → Internet
  (proxy acts for the client — server sees proxy, not real client)

Reverse Proxy:
  Internet → [Reverse Proxy] → Servers
  (proxy acts for the server — client sees proxy, not real server)
```

### Side-by-Side Comparison

| Aspect | Forward Proxy | Reverse Proxy |
|---|---|---|
| **Position** | Client side | Server side |
| **Represents** | The client | The server |
| **Hides** | Client IP from server | Server IPs from client |
| **Use cases** | Bypass geo-blocks, corporate filtering, anonymity | Load balancing, SSL termination, caching, DDoS protection |
| **Examples** | Squid Proxy, corporate web filter | NGINX, HAProxy, AWS ALB, Cloudflare |

### Forward Proxy — Detailed

```
Company Network:
  Employee → [Squid Proxy] → Google.com
  ↑
  Blocks social media, logs traffic, hides internal IPs from external
```

**Use cases:**
- Corporate internet filtering (block social media)
- Bypass geo-restrictions
- Anonymize client identity
- Cache frequently accessed content

### Reverse Proxy — Detailed

```
Internet → [NGINX Reverse Proxy] → App Server 1
                                 → App Server 2
                                 → App Server 3
```

**Use cases:**
- **Load balancing** — distribute traffic across multiple servers
- **SSL termination** — NGINX handles HTTPS, backend stays HTTP
- **Caching** — serve static files without hitting the app
- **Security** — hide and protect backend servers from direct exposure

### 🎙️ Interview Answer
> *"A forward proxy sits between the client and the internet — it acts on behalf of the client, hiding the client's identity from external servers. Corporate web filters use forward proxies. A reverse proxy sits in front of servers — it acts on behalf of the server, hiding the backend from clients. NGINX as a reverse proxy does load balancing, SSL termination, and caching. The key distinction: forward proxy = client-side, reverse proxy = server-side."*

---

---

# 🐢 4. Application Slowness Troubleshooting

**Q: A user reports the application is slow. How do you troubleshoot?**

### Layer-by-Layer Investigation

```bash
# Step 1: Scope — Is it one user or all? One page or everything?
# Narrows down: client-side, region, specific feature

# Step 2: Frontend check (browser dev tools)
# F12 → Network tab → sort by Time → find slow requests
# Check TTFB (Time to First Byte) — high TTFB = backend slow

# Step 3: Backend API response times
curl -o /dev/null -s -w "Total: %{time_total}s\n" https://api.example.com/endpoint
# >2s = backend bottleneck

# Step 4: Server resources
top                    # CPU usage
free -h                # memory (look for swap usage)
iostat -x 1 5          # disk I/O
vmstat 1 5             # virtual memory stats

# Step 5: Database queries
# Check slow query log (MySQL):
tail -f /var/log/mysql/slow.log
# Run EXPLAIN on slow queries

# Step 6: Application logs
tail -f /var/log/app/application.log | grep -i "slow\|timeout\|error"
journalctl -u myapp --since "1 hour ago"

# Step 7: Network latency
ping api.example.com        # packet loss?
traceroute api.example.com  # where is the slowdown?
mtr api.example.com         # continuous traceroute with stats

# Step 8: Cache hit rate
# Check Redis/Memcached stats
# CDN — is CloudFront/Nginx serving cached responses?
```

**Common root causes by layer:**

| Layer | Symptom | Fix |
|---|---|---|
| Frontend | Large JS/CSS, no CDN | Minify, use CDN, lazy load |
| Backend API | High response times | Profile code, fix N+1 queries |
| Database | Slow queries | Add indexes, use EXPLAIN, cache |
| Infrastructure | CPU/RAM maxed | Scale up/out, add limits |
| Network | High latency, packet loss | CDN, check routing, ISP |

### 🎙️ Interview Answer
> *"I start by scoping — is it one user or everyone? One page or all pages? Then I check from the outside in: browser dev tools for TTFB, backend API response times with curl, server resources with top/vmstat/iostat, database slow query logs, and network latency with ping/mtr. App slowness usually comes from slow DB queries, resource exhaustion, missing cache, or a recent bad deployment. After fixing, I add SLO alerts so we catch it faster next time."*

---

---

# 🔍 5. curl Works with IP but Not Domain

**Q: `curl http://IP` works but `curl http://domain.com` fails. Why?**

> **Root cause:** Almost always a **DNS resolution problem**.

```bash
# Reproduce
curl http://93.184.216.34       # ✅ Works
curl http://example.com         # ❌ Fails (could not resolve host)
```

### Diagnostic Steps

```bash
# Step 1: Test DNS resolution
nslookup example.com
dig example.com
# No answer? DNS is broken

# Step 2: Test with a public DNS server (bypass local DNS)
dig example.com @8.8.8.8       # Use Google's DNS
dig example.com @1.1.1.1       # Use Cloudflare's DNS
# Works? → Local DNS config is broken

# Step 3: Check DNS nameserver config
cat /etc/resolv.conf
# Should contain:
# nameserver 8.8.8.8
# nameserver 1.1.1.1

# Step 4: Check /etc/hosts for overrides
cat /etc/hosts
# If example.com is mapped to wrong IP → override fix

# Step 5: Check if port 53 (DNS) is blocked
nc -zv 8.8.8.8 53             # test UDP port 53
# Blocked? Firewall or security group blocking DNS

# Step 6: Check domain exists
whois example.com             # is the domain registered?
```

### Fix

```bash
# Quick fix — set public DNS
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Permanent fix — edit netplan (Ubuntu) or /etc/systemd/resolved.conf
sudo nano /etc/systemd/resolved.conf
# [Resolve]
# DNS=8.8.8.8 1.1.1.1
sudo systemctl restart systemd-resolved
```

**Other less common causes:**
- Domain has expired or doesn't exist
- You're on a VPN that blocks external DNS
- Application using a hardcoded wrong DNS server

### 🎙️ Interview Answer
> *"If curl works with an IP but not a domain name, DNS resolution is failing. I diagnose with `dig example.com` — if that fails, I test with a public server: `dig example.com @8.8.8.8`. If that works, the local nameserver in `/etc/resolv.conf` is broken and I fix it to point to 8.8.8.8. I also check `/etc/hosts` for conflicting overrides and verify port 53 isn't blocked by the firewall."*

---

---

# 🔴 6. 502 Bad Gateway

**Q: Your website returns a 502 Bad Gateway. Explain and troubleshoot.**

> **502 = Proxy received an invalid response from the upstream (backend) server.**
> The reverse proxy (NGINX/ALB) is running — but it can't reach the backend app.

```
Browser → NGINX → ???  ← backend app is unreachable
                ↑
         NGINX returns 502
```

### Common Root Causes & Fixes

| Cause | Diagnosis | Fix |
|---|---|---|
| Backend service down | `systemctl status myapp` | `systemctl restart myapp` |
| Wrong proxy_pass port | Check nginx.conf `proxy_pass` | Fix port to match app |
| Backend too slow / timeout | `proxy_read_timeout` exceeded | Increase timeout or fix slow query |
| App crashed / OOM | `journalctl -u myapp` or `docker logs` | Restart, check memory limits |
| Firewall blocking backend | `nc -zv localhost 5000` | Open port in iptables/SG |
| Bad SSL config | proxy_pass uses wrong protocol | Fix http vs https in proxy_pass |

### Diagnostic Commands

```bash
# Check NGINX error logs (most informative)
tail -f /var/log/nginx/error.log
# Look for: "connect() failed", "upstream timed out"

# Is the backend app running?
systemctl status myapp
docker ps | grep myapp

# Can NGINX reach the backend port?
nc -zv localhost 5000        # port test
curl http://localhost:5000/health   # direct backend test

# Check app logs for crash/error
journalctl -u myapp --since "10 minutes ago"
docker logs myapp --tail 100

# NGINX config check
sudo nginx -t                # syntax error?
sudo nginx -s reload         # reload after fix
```

**Common NGINX fix — proxy timeout:**
```nginx
server {
    location / {
        proxy_pass http://localhost:5000;
        proxy_read_timeout 120s;       # increase if backend is slow
        proxy_connect_timeout 10s;
        proxy_send_timeout 120s;
    }
}
```

### 🎙️ Interview Answer
> *"502 Bad Gateway means NGINX (the proxy) is running but can't reach the backend application. The first check is `systemctl status` and `journalctl` for the backend — did it crash? Then I test the connection directly with `curl http://localhost:5000` and `nc -zv localhost 5000`. NGINX error logs usually say 'connect() failed' with the specific reason. Common fixes: restart the crashed backend, fix the proxy_pass port in nginx.conf, increase proxy_read_timeout for slow backends, or open the firewall port between NGINX and the app."*

---

---

# 🖥️ 7. 0.0.0.0 vs 127.0.0.1

**Q: What is the difference between `0.0.0.0` and `127.0.0.1`?**

| | `127.0.0.1` | `0.0.0.0` |
|---|---|---|
| **Name** | Loopback / localhost | Wildcard address |
| **Traffic stays on** | Your machine only | All network interfaces |
| **Accessible from** | Only the same machine | Any device on the network |
| **Use case** | Local dev, self-testing | Production servers, accepting external traffic |

```bash
# 127.0.0.1 — server only accessible from the same machine
python3 -m http.server --bind 127.0.0.1 8080
curl http://127.0.0.1:8080       # ✅ Works from same machine
curl http://192.168.1.5:8080     # ❌ Cannot reach from another machine

# 0.0.0.0 — server accessible from any machine
python3 -m http.server --bind 0.0.0.0 8080
curl http://192.168.1.5:8080     # ✅ Works from any machine on network
```

**In Kubernetes / Docker context:**
```yaml
# Pod binding on 0.0.0.0 — accessible within cluster networking
containerPort: 8080
# hostIP: 0.0.0.0  (default)
```

**Common interview trap:** *"Why can't I access my app from another machine?"*
> Because it's bound to `127.0.0.1` — change to `0.0.0.0` and open the firewall port.

### 🎙️ Interview Answer
> *"127.0.0.1 is the loopback address — traffic never leaves the machine, so only local processes can reach it. 0.0.0.0 is the wildcard — when a server binds to it, it listens on all network interfaces, making it accessible from any device that can route to the machine. In development, I bind to 127.0.0.1 for security. In production, services bind to 0.0.0.0 and I control access through firewalls and security groups."*

---

---

# 🏘️ 8. Public vs Private Subnet

**Q: What is the difference between Public and Private Subnets in AWS?**

**The single deciding factor:**
> A subnet is **public** if its route table has a route: `0.0.0.0/0 → Internet Gateway`
> A subnet is **private** if it has **no such route** (or routes via NAT Gateway for outbound-only)

### Side-by-Side Comparison

| | Public Subnet | Private Subnet |
|---|---|---|
| **Internet access** | ✅ Inbound + Outbound | ❌ No inbound; NAT for outbound |
| **Route table** | `0.0.0.0/0 → igw-xxx` | `0.0.0.0/0 → nat-xxx` or nothing |
| **EC2 public IP** | Needed (or Elastic IP) | Not needed (not reachable anyway) |
| **Put here** | Load balancers, NAT Gateway, Bastion hosts | App servers, databases, Lambda |
| **Security** | Exposed to internet | Shielded from direct access |

### Architecture Pattern

```
Internet
    ↓
Internet Gateway (VPC boundary)
    ↓
Public Subnet:
  - Application Load Balancer (receives user traffic)
  - NAT Gateway (gives private subnet outbound internet)
  - Bastion Host (SSH jump server)
    ↓
Private Subnet:
  - EC2 App Servers (unreachable from internet directly)
  - RDS Database (extra secure — not even outbound by default)
```

### Route Table Examples

```
# Public Subnet Route Table
Destination     Target
10.0.0.0/16     local           ← VPC internal traffic
0.0.0.0/0       igw-0abc1234    ← ALL other traffic → Internet Gateway

# Private Subnet Route Table
Destination     Target
10.0.0.0/16     local           ← VPC internal traffic
0.0.0.0/0       nat-0def5678    ← outbound only → NAT Gateway
                                ← no inbound from internet possible
```

### 🎙️ Interview Answer
> *"The difference is the route table. A public subnet has a route sending 0.0.0.0/0 traffic to an Internet Gateway — so it's reachable from and can reach the internet. A private subnet has no such route — instances can't be directly accessed from the internet. For outbound internet access (like downloading packages), a private subnet uses a NAT Gateway in the public subnet. Best practice: load balancers and bastion hosts in public subnets, databases and app servers in private subnets."*

---

---

# 🔧 9. Convert Private Subnet to Public

**Q: You accidentally created a private subnet. How do you fix it?**

### 5-Step Fix

```
Step 1: Ensure an Internet Gateway is attached to the VPC
  VPC Console → Internet Gateways → Verify "igw-xxx" is attached to your VPC
  If not → Create → Attach to VPC

Step 2: Update (or create) the Route Table
  VPC → Route Tables → Select the subnet's route table → Routes → Edit
  Add route:
    Destination: 0.0.0.0/0
    Target: igw-xxxxxxxx  (your Internet Gateway)
  Save

Step 3: Associate the updated route table with the subnet
  VPC → Subnets → Select subnet → Route table tab → Edit association
  → Select the route table that has the IGW route

Step 4: Enable auto-assign public IPs for the subnet
  VPC → Subnets → Select subnet
  → Actions → Edit subnet settings
  → Enable "Auto-assign public IPv4 address" ✅

Step 5: Assign public IP to existing EC2 (if already running)
  EC2 → Elastic IPs → Allocate → Associate to the instance
  (Existing instances don't automatically get public IPs even after subnet change)
```

**Verification:**
```bash
# After making changes, test connectivity
ssh -i key.pem ubuntu@<new-public-ip>
curl http://<new-public-ip>
```

> **Root cause of "private subnet" mistake:** Route table was missing `0.0.0.0/0 → igw` route, OR auto-assign public IP was disabled.

### 🎙️ Interview Answer
> *"The fix has three parts: route table, subnet settings, and existing instances. Add `0.0.0.0/0 → IGW` to the route table and associate it with the subnet. Enable 'Auto-assign public IPv4' in subnet settings so new instances get public IPs. For existing instances that are already running, allocate an Elastic IP and associate it — they won't automatically get a public IP just because the subnet changed."*

---

---

# 📌 Master Cheatsheet

```
╔══════════════════════════════════════════════════════════════════════╗
║          NETWORKING INTERVIEW MASTER CHEATSHEET                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  DNS:                                                                ║
║  Phonebook of internet: domain → IP                                 ║
║  A=IPv4 | AAAA=IPv6 | CNAME=alias | MX=mail | NS=nameserver        ║
║  Resolution: browser cache → OS → ISP resolver → root → TLD → auth ║
║                                                                      ║
║  OSI MODEL (top → bottom):                                          ║
║  7=Application (HTTP) | 6=Presentation (TLS) | 5=Session (ID)      ║
║  4=Transport (TCP/UDP, ports, segments)                              ║
║  3=Network (IP, routing, packets)                                   ║
║  2=Data Link (MAC, Ethernet, frames)                                ║
║  1=Physical (cables, Wi-Fi, bits)                                   ║
║  Before OSI: DNS resolve → TCP 3-way handshake (SYN/SYN-ACK/ACK)  ║
║                                                                      ║
║  PROXIES:                                                            ║
║  Forward = client-side, hides client, corporate filters             ║
║  Reverse = server-side, hides server, LB + SSL + caching           ║
║  NGINX = reverse proxy | Squid = forward proxy                      ║
║                                                                      ║
║  curl WORKS WITH IP, NOT DOMAIN:                                     ║
║  dig example.com → test DNS | dig @8.8.8.8 → bypass local DNS      ║
║  /etc/resolv.conf → nameserver config | /etc/hosts → overrides     ║
║  Fix: echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf        ║
║                                                                      ║
║  502 BAD GATEWAY:                                                    ║
║  Proxy running but can't reach backend                               ║
║  Check: systemctl status app → curl localhost:PORT → nginx logs     ║
║  Causes: app crashed | wrong proxy_pass port | timeout | OOM        ║
║                                                                      ║
║  0.0.0.0 vs 127.0.0.1:                                              ║
║  127.0.0.1 = loopback (local only, never leaves machine)            ║
║  0.0.0.0 = all interfaces (accessible from any device on network)   ║
║                                                                      ║
║  SUBNETS:                                                            ║
║  Public = route table has 0.0.0.0/0 → IGW                          ║
║  Private = no direct route to internet (NAT for outbound only)     ║
║  Public: LB + Bastion + NAT GW | Private: App servers + DB         ║
║                                                                      ║
║  PRIVATE → PUBLIC SUBNET FIX:                                       ║
║  1. Attach IGW to VPC                                               ║
║  2. Route table: add 0.0.0.0/0 → igw-xxx                          ║
║  3. Associate route table with subnet                               ║
║  4. Enable auto-assign public IP in subnet settings                 ║
║  5. Allocate Elastic IP for existing EC2 instances                  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Delivery Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Use **phonebook** analogy for DNS | Just say "it translates names to IPs" |
| Say "Before OSI: DNS + 3-way handshake" | Start OSI at Layer 7 without prerequisites |
| Distinguish **Forward = client side** clearly | Mix up which proxy hides what |
| For 502: say "proxy is up, backend is down" | Say "502 means the server is down" |
| For curl+IP works: say "dig @8.8.8.8 to test" | Just say "it's a DNS issue" without diagnostics |
| For subnet fix: mention **Elastic IP for existing EC2** | Forget that existing instances need manual IP assignment |

---

## 📚 Resources

- 🔗 [devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)
- 🔗 [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- 🔗 [Cloudflare Learning — DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)
- 🔗 [NGINX Reverse Proxy Guide](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

---

> ⭐ **Star this repo** if it helped you prepare for your DevOps interview!
> 🔔 Paste the next topic's notes — I'll overwrite with only those!
