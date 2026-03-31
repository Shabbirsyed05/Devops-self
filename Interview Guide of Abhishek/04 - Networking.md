# Networking Interview Guide

> Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide) — `04-networking/` folder

---

# Table of Contents

1. [What is DNS?](#1-what-is-dns)
2. [Explain OSI Model with Example](#2-explain-osi-model-with-example)
3. [Forward Proxy vs Reverse Proxy](#3-forward-proxy-vs-reverse-proxy)
4. [Slowness in Application](#4-slowness-in-application)
5. [Curl Works with IP but Not Domain](#5-curl-works-with-ip-but-not-domain)
6. [502 Bad Gateway](#6-502-bad-gateway)
7. [0.0.0.0 vs 127.0.0.1](#7-0000-vs-127001)
8. [Public vs Private Subnet](#8-public-vs-private-subnet)
9. [Change Private Subnet to Public](#9-change-private-subnet-to-public)

---

# 1. What is DNS?

## Question
What is DNS? Explain in simple words.

### 📝 Short Explanation
DNS is like the internet's **phonebook**. It helps translate website names into IP addresses that computers use to talk to each other.

## ✅ Answer

DNS stands for **Domain Name System**.

When you type a website like `www.google.com` into your browser, your computer doesn't understand that name directly. Instead, it asks the DNS system to find the IP address (like `142.250.64.100`) that matches `www.google.com`.

Once it gets the IP address, your computer can connect to the right server and load the website.

---

### 📘 Detailed Explanation

- **Domain name**: A human-friendly name like `amazon.com`.
- **IP address**: A computer-friendly address like `192.0.2.1`.
- **DNS Resolver**: The system that looks up domain names and returns IP addresses.

#### 🔄 Step-by-Step Process:
1. You enter `www.example.com` in your browser.
2. Your computer asks the **DNS resolver** for the IP address.
3. The resolver checks its cache or queries other DNS servers.
4. Once it finds the matching IP, it returns it to your browser.
5. Your browser connects to the IP and loads the website.

---

### 🧠 Example:
Think of **DNS** like asking someone for a restaurant's phone number:
- You say: "I want to call Domino's Pizza."
- DNS replies: "Here's the number: 123-456-7890."
- Now you can call and place your order.

> Summary:
> DNS is a behind-the-scenes system that lets us use easy names like `google.com` instead of hard-to-remember IP addresses.

---

# 2. Explain OSI Model with Example

## Question
Explain the complete flow of a request from client to server using the OSI Model.

### 📝 Short Explanation
This question checks if you understand how a network request travels across layers — from user input in a browser to reaching a remote server — using the 7 layers of the OSI Model.

## ✅ Answer

The **OSI Model** has **7 layers**, and each layer plays a specific role in transferring data from a client to a server.

### 📘 Detailed Explanation

#### 1. **Application Layer (Layer 7)**
- You type `https://example.com` in a browser.
- The browser creates an HTTP request.
- 🧠 **Key Protocols**: HTTP, HTTPS, DNS, FTP, SMTP

#### 2. **Presentation Layer (Layer 6)**
- Ensures the data is in the right format.
- Handles encryption (SSL/TLS) and compression.

#### 3. **Session Layer (Layer 5)**
- Manages session establishment and teardown between client and server.

#### 4. **Transport Layer (Layer 4)**
- Breaks data into **segments** and ensures **reliable delivery**.
- Adds source and destination **port numbers**.
- 🧠 **Key Protocols**: TCP (reliable), UDP (faster but no guarantee)

#### 5. **Network Layer (Layer 3)**
- Adds source and destination **IP addresses**.
- Chooses the **best route** for the packet across the internet.
- 🧠 **Key Protocol**: IP (Internet Protocol)

#### 6. **Data Link Layer (Layer 2)**
- Converts data into **frames**.
- Adds MAC addresses of the devices on a local network.

#### 7. **Physical Layer (Layer 1)**
- Transfers raw **bits** (0s and 1s) over physical hardware like cables, Wi-Fi signals, or fiber optics.

---

### 🔄 Then What Happens?

- At the server side, the data flows **upward from Layer 1 to Layer 7**.
- Each layer **removes the headers** added by the client side.
- The server **responds**, and the response travels **back through the same layers** in reverse.

### 🧠 Analogy

> Sending a letter:
> - You write a message (App layer).
> - Put it in an envelope (Presentation).
> - Mark sender/recipient (Session).
> - Choose a postal service (Transport).
> - Address it (Network).
> - The postman routes it (Data Link).
> - Finally, it goes through trucks/planes (Physical).

> Summary:
> The OSI model breaks network communication into layers so we can troubleshoot and build systems more effectively.

---

# 3. Forward Proxy vs Reverse Proxy

## Question
Explain the difference between a Forward Proxy and a Reverse Proxy.

### 📝 Short Explanation
Both proxies act as intermediaries but sit at **different ends of the request flow**.

## ✅ Answer

| Aspect | Forward Proxy | Reverse Proxy |
|---|---|---|
| **Position** | Between **client** and the internet | Between **internet** and the **server** |
| **Who it represents** | Acts **on behalf of the client** | Acts **on behalf of the server** |
| **Use Cases** | Anonymize clients, Bypass firewalls, Control access | Load balancing, SSL termination, Caching, Protect internal servers |

### 📘 Detailed Explanation

#### 🔁 Forward Proxy
- Used by clients to access external servers.
- The target server only sees the proxy, **not the real client**.
- 🧠 **Analogy**: Like a travel agent booking your ticket on your behalf.

#### 🔃 Reverse Proxy
- Sits in front of a group of servers and routes client requests to them.
- The client thinks it's talking directly to the server.
- 🧠 **Analogy**: Like a receptionist at an office connecting you to the right person.

> Summary:
> A **Forward Proxy** serves the **client** and hides them from the server. A **Reverse Proxy** serves the **server** and hides it from the client.

---

# 4. Slowness in Application

## Question
A user reports that the application is slow. Explain how you would troubleshoot.

## ✅ Answer

### 🔍 Step-by-Step Investigation:

1. **Clarify the Scope** — One user or many? Specific pages? Which environment?
2. **Check Frontend** — Use browser dev tools (Network, Performance tabs). Check TTFB.
3. **Backend API Performance** — Check server response times via APM tools.
4. **Database Slowness** — Slow queries? Missing indexes? Use `EXPLAIN`.
5. **Infrastructure** — Check CPU, memory, disk I/O using `top`, `htop`, `vmstat`, `iostat`.
6. **Monitor Logs & Alerts** — Check for errors or recent deployments.
7. **Caching & CDN** — Cache misses? CDN serving properly?
8. **Network or DNS Latency** — Run `ping`, `traceroute`, `mtr`.
9. **Rollbacks or Restarts** — If slowness started after a new release.
10. **After Fix: Monitor & Prevent** — Add performance alerts and SLOs.

> Summary:
> App slowness can come from **frontend, backend, DB, infrastructure, or network**. Use a systematic layer-by-layer approach.

---

# 5. Curl Works with IP but Not Domain

## Question
`curl` works with an IP address but fails with the domain name. Why?

## ✅ Answer

The issue is most likely **DNS-related**.

### 🔍 Common Causes:
1. **DNS Not Resolving** — Run `nslookup example.com` and `dig example.com`.
2. **Wrong DNS Configuration** — Check `/etc/resolv.conf` for valid nameservers.
3. **Firewall Blocking DNS** — Port 53 might be blocked. Test: `dig example.com @8.8.8.8`
4. **Domain Doesn't Exist or Typo** — Verify with `whois example.com`.
5. **Host File Override** — Check `/etc/hosts` for incorrect entries.
6. **Internal DNS Only** — Ensure you're on the right VPN/network.

### ✅ Fix:
```bash
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null
```

> Summary:
> When `curl` works with IP but fails with domain, it's almost always a **DNS resolution problem**.

---

# 6. 502 Bad Gateway

## Question
Your website is returning a **502 Bad Gateway**. Explain and troubleshoot.

## ✅ Answer

A **502** error is returned when the **reverse proxy or load balancer** cannot reach the backend service.

### 📘 Common Root Causes:
1. **Backend Service is Down** — Check: `systemctl status your-app`
2. **Wrong Upstream Config in NGINX** — Verify `proxy_pass` port/host.
3. **Backend Too Slow / Times Out** — Adjust: `proxy_read_timeout 60s;`
4. **Firewall Blocking** — Test: `nc -zv localhost 5000`
5. **Incorrect SSL Termination** — Fix `proxy_pass` protocol.
6. **App Crashed or OOM** — Check: `journalctl -u your-app` or `docker logs`

### 🛠️ Troubleshoot:
```bash
tail -f /var/log/nginx/error.log
systemctl restart your-app
curl http://localhost:5000/health
```

> Summary:
> A **502 Bad Gateway** means your proxy could not communicate with your backend service.

---

# 7. 0.0.0.0 vs 127.0.0.1

## Question
What is the difference between `0.0.0.0` and `127.0.0.1`?

## ✅ Answer

| Address | Meaning | Use Case |
|---|---|---|
| `127.0.0.1` | Loopback address (localhost) | Computer talks to itself |
| `0.0.0.0` | All IPv4 addresses on local machine | Servers listen on **all interfaces** |

- `127.0.0.1`: Traffic **never leaves your machine**. Example: `curl http://127.0.0.1:8080`
- `0.0.0.0`: Makes server available to **other devices** on the network. Example: `python3 -m http.server --bind 0.0.0.0`

### 🧠 Analogy
- `127.0.0.1`: "I'm talking to myself only."
- `0.0.0.0`: "I'm open to talk to anyone who connects to me."

---

# 8. Public vs Private Subnet

## Question
What is the difference between Public and Private Subnets?

## ✅ Answer

| Type | Internet Access | Use Case | Route Table |
|---|---|---|---|
| **Public Subnet** | Yes (via IGW) | Load balancers, Bastion hosts | Route to Internet Gateway |
| **Private Subnet** | No direct access | Databases, app servers | NAT Gateway optional |

- **Public Subnet**: Has a route to an **Internet Gateway (IGW)**.
  ```text
  0.0.0.0/0 → igw-xxxxxxxx
  ```
- **Private Subnet**: Has **no direct route** to the internet. Uses **NAT Gateway** for outbound.
  ```text
  0.0.0.0/0 → nat-xxxxxxxx
  ```

### 🧠 Analogy
- **Public Subnet**: House with a door that opens directly to the street.
- **Private Subnet**: Room in a gated community — exit only through controlled paths.

---

# 9. Change Private Subnet to Public

## Question
You accidentally created a private subnet. How do you fix it?

## ✅ Answer

### 🛠️ Steps:

1. **Update Route Table** — Add route: `0.0.0.0/0 → igw-xxxxxxxx`
2. **Associate Correct Route Table** — Link updated route table to the subnet.
3. **Enable Auto-Assign Public IPs** — VPC → Subnets → Modify auto-assign IP settings.
4. **Assign Public IP to Existing EC2** — Allocate and attach an Elastic IP.
5. **Ensure IGW is Attached to VPC** — Verify in VPC → Internet Gateways.

> Summary:
> A public subnet needs a **route to the internet via an IGW**, and EC2 instances need **public IPs**. Adjusting these settings converts a private subnet into a public one.

---
