# ☸️ Day-36 | Kubernetes Interview Questions Part-1

---

Based on the transcript for **"Day-36 | Kubernetes Interview Questions Part-1,"** here is a summary of the session.

Unlike previous practical sessions, this module acts as a **Mock Interview**.  
It tests your retention of the concepts covered from **Day 30 to Day 35**.  
The speaker presents **10 scenario-based questions** commonly asked in DevOps interviews.

---

## 1️⃣ Fundamental Comparisons

These questions test if you understand **why we use these tools**.

### 🔹 Docker vs. Kubernetes
- **Docker:**  
  A container platform (single node). If the node crashes, the app dies.
- **Kubernetes:**  
  A container orchestration platform (cluster). It adds **Auto-Healing** (restarting failed pods) and **Auto-Scaling**.

---

### 🔹 Docker Swarm vs. Kubernetes
- **Swarm:**  
  Simple and easy to install, but limited features. Good for small scale.
- **Kubernetes:**  
  The industry standard for Enterprise/Large Scale.  
  It offers advanced networking, third-party integrations (CNCF), and custom resource definitions (CRDs) that Swarm lacks.

---

### 🔹 Container vs. Pod
- A Pod is a wrapper (runtime specification) around a container.
- **Key Difference:**  
  A Pod can hold multiple containers.  
  If it does, those containers share the same **Network (localhost)** and **Storage**.

---

## 2️⃣ Architecture Deep Dive

You must be able to list the components of the **Control Plane (Master)** and **Data Plane (Worker)**.

---

### 🧠 Control Plane Components
- **API Server:**  
  The entry point for all REST commands.
- **Scheduler:**  
  Assigns pods to nodes based on resources.
- **etcd:**  
  Stores the cluster data (object store).
- **Cloud Control Manager:**  
  The logic that talks to AWS/Azure (e.g., to create a Load Balancer).

---

### ⚙️ Data Plane Components
- **Kubelet:**  
  The agent on the worker node.  
  It monitors the Pod. If a Pod dies, Kubelet notifies the API server to trigger auto-healing.
- **Kube-Proxy:**  
  The networker.  
  It maintains network rules (IP Tables) on the node to ensure traffic reaches the correct Pod.
- **Container Runtime:**  
  The software that actually runs the container  
  (e.g., Docker Shim, containerd, CRI-O).

---

## 3️⃣ Logical Isolation & Networking

---

### 🔐 What is a Namespace?
- It is **Logical Isolation** within a single cluster.
- **Scenario:**  
  If you have 20 teams (Project A, Project B), you don't need 20 clusters.  
  You create one cluster with different namespaces.  
  Team A cannot accidentally access or delete Team B’s resources.

---

### 🌐 Service Types (Exposure)
- **ClusterIP:**  
  Internal only.
- **NodePort:**  
  Exposes the app on the Worker Node's IP.  
  Accessible by anyone in the organization/VPN who can ping the node.
- **LoadBalancer:**  
  Exposes the app to the Public Internet.  
  It requests a Public IP from the Cloud Provider (AWS/Azure) so external users can access it.

---

## 4️⃣ The Behavioral Question

The session ends with a critical **"soft skill"** question:

### ❓ *"What are your day-to-day activities as a DevOps Engineer?"*

❌ Do **not** say:  
> "I just run commands."

✅ Instead, explain your responsibilities:

1. **Cluster Management:**  
   Managing upgrades, security patches, and scaling worker nodes.
2. **Subject Matter Expert (SME):**  
   Helping developers troubleshoot why their Pods are crashing or why traffic isn't routing correctly.
3. **Monitoring:**  
   ensuring the cluster is healthy and setting up alerts.

---

==============================================================================================

# ☸️ Day-37 | Kubernetes Services Deep Dive

---

Based on the transcript for **"Day-37 | Kubernetes Services Deep Dive,"** here is a summary of the practical session.

This session moves beyond the theory covered in previous days to demonstrate how Kubernetes Services actually work using a real Python application and network visualization tools.

---

## 1️⃣ The Setup: A Real Application

To demonstrate networking, the session moved away from simple Nginx demos to a **Python Django Application**.

- **The Deployment:**  
  A deployment was created with **2 Replicas**.

- **The Problem Demonstrated:**  
  When a Pod is deleted, the ReplicaSet creates a new one, but the IP address changes (Dynamic Allocation).  
  This proves why users cannot rely on Pod IPs and need a Service to act as a stable middleman.

---

## 2️⃣ Feature 1: Service Discovery (Labels & Selectors)

The most critical concept in this session was how a Service **"finds"** the Pods it is supposed to manage.

- **The Mechanism:**  
  Services do not track IP addresses; they track **Labels**.

- **The Rule:**  
  The selector in the Service YAML must match the label in the Pod YAML **exactly**.

- **The "Break" Demo:**  
  To prove this, the speaker intentionally modified the Service selector to a wrong value.  
  Even though the Pods were running, the Service could not connect (Service Discovery failed), proving that accurate labeling is the only link between the two.

---

## 3️⃣ Feature 2: Exposing Applications

The session demonstrated two ways to let users access the application:

---

### 🔹 NodePort (Internal/VPN Access)

- **How it works:**  
  It opens a specific port (e.g., **30007**) on the Worker Node (or Minikube IP).

- **Use Case:**  
  Ideal for internal users or people connected to the company VPN who can ping the Node directly.

---

### 🔹 LoadBalancer (External/Public Access)

- **How it works:**  
  It asks the Cloud Provider (AWS/Azure) to provision a Public IP.

- **The Minikube Limitation:**  
  On a laptop (Minikube), the "External IP" remains **Pending** because there is no Cloud Controller to give you a public IP.  
  In a real cloud environment, this would provide a reachable internet address.

---

## 4️⃣ Feature 3: Load Balancing (Visualized with Kubeshark)

The session introduced a tool called **Kubeshark** to visualize network traffic, which is often invisible in the CLI.

- **The Test:**  
  The speaker sent **6 curl requests** to the Service.

- **The Result:**  
  Kubeshark showed the traffic flowing in a **Round Robin** fashion.

  - Request 1 -> Pod A (IP ...0.5)  
  - Request 2 -> Pod B (IP ...0.7)  
  - Request 3 -> Pod A.

- **Conclusion:**  
  This visually proved that the Service automatically distributes load across all available healthy Pods without manual configuration.

---

## 5️⃣ Summary of Tools Used

- **Minikube:**  
  The local cluster environment.

- **Kubectl:**  
  Used to create/edit deployments and view "Pending" status on LoadBalancers.

- **Kubeshark:**  
  A traffic viewer used to debug and visualize packet flow between Services and Pods.

---

================================================================================================


This installs the **Nginx Ingress Controller** automatically.

---

### Step 2: Create the Ingress Resource
Create an `ingress.yaml` file defining a **Host-Based Rule**.

**Example Rule:**
- If traffic comes from `foo.bar.com`
- Route it to the backend Service

---

### Step 3: The Local DNS Trick (`/etc/hosts`)
Since you do not actually own the domain `foo.bar.com`, the internet does not know where to send traffic.

**The Hack:**
- Edit your local `/etc/hosts` file
- Map `foo.bar.com` → Minikube Ingress IP

This simulates real DNS behavior and allows testing Ingress locally.

---

## 5️⃣ Types of Ingress Routing

Ingress supports multiple routing strategies:

- **Host-Based Routing**  
  - `amazon.com` → App A  
  - `amazon.in` → App B  

- **Path-Based Routing**  
  - `example.com/payment` → Payment Service  
  - `example.com/login` → Login Service  

- **TLS / Secure Routing**  
  - Ingress manages HTTPS certificates  
  - Internal Pods do not need to handle SSL themselves  

---

=================================================================================

# 🔐 Day-39 | Introduction to Kubernetes RBAC

---

Based on the transcript for **"Day-39 | INTRODUCTION TO K8s RBAC,"** here is a summary of the concepts.

This session shifts focus from **Networking (Services / Ingress)** to **Security**.

---

## 1️⃣ What is RBAC?

**RBAC (Role-Based Access Control)** is the method Kubernetes uses to decide **who can access the cluster** and **what they are allowed to do**.

### ❗ Why is RBAC Critical?
- Without RBAC, **anyone accessing the cluster may have full admin access**
- A developer could accidentally delete a database
- A compromised Pod could destroy the API Server

### 🎯 Scope of RBAC
RBAC manages access for **two types of entities**:
1. **Users** – Humans (Developers, DevOps Engineers)
2. **Service Accounts** – Applications / Pods (e.g., Jenkins bot, Monitoring tools)

---

## 2️⃣ The "User" Mystery (Interview Question)

**Interview Question:**  
👉 *How do you run the user add command in Kubernetes?*

### ✅ The Answer
You **cannot**.  
Kubernetes **does not manage users natively**.

### 🔑 The Real Mechanism
Kubernetes **offloads user management** to **External Identity Providers (IdP)**.

- The **API Server acts like an OAuth server**
- It trusts external systems such as:
  - AWS IAM
  - Google
  - LDAP
  - Okta

### 📌 Example
If you use **AWS EKS**:
- You authenticate using **AWS IAM credentials**
- Kubernetes simply verifies that AWS confirms your identity

---

## 3️⃣ The 3 Pillars of RBAC Architecture

To implement access control, **three components must work together**:

| Component | Definition | Analogy |
|---------|-----------|--------|
| **User / Service Account** | The entity requesting access (Pods use ServiceAccount YAML). | 👤 Person (e.g., Abhishek) |
| **Role** | YAML file defining permissions (e.g., read Pods, delete Secrets). | 🪪 ID Badge |
| **RoleBinding** | Connects a User to a Role. Without this, permissions do not apply. | 🎟️ Giving the Badge |

### ⚠️ Important Note on Scope
- **Role** → Permissions within a **single Namespace**
- **ClusterRole** → Permissions across the **entire Cluster**  
  *(ClusterRole will be detailed in the next session)*

---

## 4️⃣ Service Accounts for Pods

### 🔄 Default Behavior
- Every Pod automatically gets a **default Service Account**

### 🚨 Security Risk
- If a hacker compromises a Pod:
  - They inherit **all permissions** of that Service Account

### 🔐 Best Practice
- Always define **custom Service Accounts**
- Restrict permissions:
  - Example: Prevent Pods from reading Secrets

---

## 5️⃣ Practical Tool: OpenShift Sandbox (Free Cluster)

To practice RBAC in a **real-world environment**, the speaker recommends **Red Hat OpenShift Sandbox**.

### ⭐ Why OpenShift Sandbox?
- Free shared Kubernetes cluster (30 days)
- Mimics production-like restrictions
- Forces understanding of **RBAC permissions**
- Better than Minikube for security practice

### 🛠️ How to Set Up
1. Search for **"OpenShift Sandbox"**
2. Create a free **Red Hat account**
3. Login automatically creates a **dedicated Namespace**
4. Access options:
   - Web Console (UI)
   - Copy login token for local CLI usage

---

===========================================================================


# 🚀 Day-40 | Kubernetes Custom Resources & Custom Controller

---

Based on the transcript for **"Day-40 | KUBERNETES CUSTOM RESOURCES | CUSTOM CONTROLLER,"** here is a summary of the concepts.

This session marks an **advanced transition**.  
In **Days 30–39**, we used **Native Kubernetes resources** (Pod, Service, Deployment) that come **out of the box**.  
Today, we learn how to **extend Kubernetes** to do things it was **not originally built to do**.

---

## 1️⃣ The Problem: Native Resources Aren’t Enough

• **Context:**  
Standard Kubernetes knows how to manage **Pods** and **Load Balancers**.  
However, at scale, organizations need advanced capabilities like:
- Service Mesh (Istio)
- GitOps (ArgoCD)
- Advanced Security (Kyverno)

• **The Limitation:**  
Kubernetes core developers **cannot add logic for every tool** into the main Kubernetes codebase.  
Doing so would make Kubernetes impossible to maintain.

• **The Solution:**  
Kubernetes allows you to **extend its API**.  
You can teach Kubernetes **new words and new capabilities** using **Custom Resources**.

---

## 2️⃣ The 3 Core Components (The “Trinity”)

To add a new feature (like Istio) to Kubernetes, **three components must exist**.

---

### 🧾 A. CRD (Custom Resource Definition) — *“The Rulebook”*

• **Definition:**  
A YAML file that tells Kubernetes:  
> “I am introducing a new type of object (e.g., VirtualService).”

• **Function:**  
- Defines the **schema and validation rules**
- Just like Kubernetes knows a Deployment must have `spec` and `template`,
  a CRD defines what fields are allowed in a custom object

• **Analogy:**  
Adding a **new word** to the Kubernetes dictionary

---

### 📄 B. CR (Custom Resource) — *“The Request”*

• **Definition:**  
The actual YAML file created by the **User / Developer** to use the new feature

• **Function:**  
- Looks similar to a Deployment YAML
- Uses a custom `kind`, such as:
  - `kind: VirtualService`
  - `kind: Application`

• **Validation:**  
When a CR is created, Kubernetes validates it **against the CRD**

---

### 🧠 C. Custom Controller — *“The Brain”*

• **Definition:**  
A program (usually written in **Golang**) running as a **Pod inside the cluster**

• **Function:**  
- Watches for creation of Custom Resources
- When a CR is detected, it performs the actual work  
  (e.g., configuring network rules, syncing Git repositories)

• **Key Insight:**  
If you create a **CRD and CR** but **do not install the Controller**,  
👉 **Nothing will happen**  
The resource will just sit there with no brain to process it.

---

## 3️⃣ Comparison: Native vs Custom Kubernetes

| Feature | Native Kubernetes | Custom Kubernetes (e.g., Istio) |
|------|------------------|--------------------------------|
| Definition | Deployment Resource (Built-in) | CRD (Custom Resource Definition) |
| Instance | deployment.yaml | virtual-service.yaml (CR) |
| Brain | Deployment Controller | Custom Controller (e.g., Istiod) |
| Action | Creates ReplicaSet / Pod | Configures Traffic Rules / Mesh |

---

## 4️⃣ The DevOps Workflow (Real Company View)

### 👨‍💻 DevOps Engineer

• **Responsibility:**  
Usually does **not** write Golang controllers  
(unless you are a specialized Kubernetes developer)

• **Task:**  
- Install **CRDs and Controllers**
- Typically done using **Helm Charts**
- Ensure controllers are **running and healthy**

• **Debugging:**  
If a developer says:  
> “My Istio setup isn’t working”

You must:
- Check **Custom Controller logs**
- Verify whether the CR is being processed

---

### 🧑‍💻 Developer / User

• **Task:**  
- Simply writes the **Custom Resource YAML**
- Uses the new Kubernetes capability without worrying about internals

---

## 5️⃣ Practical Example: Istio

The session demonstrates the concept using **Istio (Service Mesh)**.

### 🔧 Workflow

1️⃣ **Install Istio**
- Use **Helm**

2️⃣ **Result**
- CRDs are installed (API is extended)
- Istio Controller (brain) is deployed

3️⃣ **Usage**
- Run:
```bash
kubectl get crd
```

========================================================

# 🚀 Day-41 | Kubernetes Live Project | ConfigMaps & Secrets

---

Based on the transcript for **"Day-41 | KUBERNETES LIVE PROJECT | CONFIGMAPS & SECRETS,"** here is a summary of the concepts and the practical demo.

This session addresses a **fundamental development problem**:  
👉 **How do we separate configuration (data) from application code (logic)?**

---

## 1️⃣ The Core Problem: Hardcoding

Backend applications usually depend on external configuration such as:

- Database Port (e.g., 3306)
- Database Username / Password
- Connection Type

### ❌ Why not hardcode them?

If these values are hardcoded inside application code or Docker images:

1. **Flexibility Issue**  
   - Any change (like port update) requires rebuilding the entire Docker image.

2. **Security Risk**  
   - Hardcoded passwords are exposed to anyone with access to the code or image.

---

## 2️⃣ The Solution: Two Kubernetes Object Types

Kubernetes provides **two dedicated resources** to manage configuration safely.

---

### 🧾 A. ConfigMap (Non-Sensitive Data)

• **Purpose:**  
Stores standard configuration data such as:
- Database Host
- Port
- URLs
- Themes

• **Visibility:**  
Stored in **plain text**

• **Analogy:**  
`.env` file or `settings.json`

---

### 🔐 B. Secrets (Sensitive Data)

• **Purpose:**  
Stores confidential information such as:
- Passwords
- API Keys
- Tokens

#### 🔒 Security Feature 1: Encryption at Rest
- Secrets are **encrypted in etcd**
- Even if etcd is compromised, data is unreadable without the key

#### 🔒 Security Feature 2: Base64 Encoding
- When viewed via CLI, Secrets appear Base64-encoded  
- Base64 is **not encryption**, only encoding

#### ✅ Recommendation
- Follow **Least Privilege RBAC**
- Developers:
  - ✔ Access Pods & ConfigMaps
  - ❌ Restricted access to Secrets

---

## 3️⃣ The Classic Interview Question

### ❓ What is the difference between a ConfigMap and a Secret?

| Feature | ConfigMap | Secret |
|------|----------|--------|
| Data Type | Non-Sensitive (Ports, URLs) | Sensitive (Passwords, Keys) |
| Storage | Plain text | Encrypted at rest (etcd) |
| User Access | Generally open to developers | Restricted via RBAC |

---

## 4️⃣ How to Inject Data into Pods (Practical Demo)

The session demonstrated **two methods** to inject configuration into Pods.

---

### 🔹 Method 1: Environment Variables (`env`)

• **How it works:**  
- Use `valueFrom` and `configMapKeyRef` in Deployment YAML
- Pulls individual keys (e.g., `DB_PORT`)

• **Critical Limitation:**  
- If ConfigMap is updated, **Pods do NOT see changes**
- Environment variables are read only at **process startup**
- Pod restart is required

---

### 🔹 Method 2: Volume Mounts (Recommended)

• **How it works:**  
- Mount ConfigMap as a file inside the Pod  
- Example path: `/opt/DB_PORT`

• **Key Benefit: Auto-Update**  
- When ConfigMap changes:
  - Kubernetes updates the file automatically
  - No Pod restart required

• **Important Note:**  
- Application must be capable of **re-reading the file dynamically**

---

## 5️⃣ Practical Commands for Revision

### 📌 Create a ConfigMap
```bash
kubectl create configmap <name> --from-literal=KEY=VALUE
```

📌 Create a Secret
```bash
kubectl create secret generic <name> --from-literal=KEY=VALUE
```

📌 Decode a Secret (Debugging)
```bash
echo <base64-value> | base64 --decode
```
⚠ Important:
This proves why RBAC is mandatory — Base64 is not encryption, only encoding.

===============================================================================
# 📊 Day-42 | Kubernetes Monitoring using Prometheus & Grafana

---

Based on the transcript for **"Day-42 | Kubernetes Monitoring using Prometheus & Grafana,"** here is a summary of the concepts and practical workflow.

This session focuses on **Observability**.  
Now that we know how to deploy applications (previous sessions), we need to know **if they are actually healthy, running, and performing well**.

---

## 1️⃣ The Power Duo: Prometheus & Grafana

The industry standard for Kubernetes monitoring involves **two tools working together**.

---

### 🧠 Prometheus (The Brain & Storage)

◦ Open-source monitoring system and **Time Series Database (TSDB)**  
◦ **Job:**  
- Collects (scrapes) metrics from the cluster  
- Stores metrics on disk  
- Executes alert logic  

◦ **Example Alerts:**  
- Send Slack message if API server is down  
- Trigger alert if CPU usage exceeds threshold

---

### 🎨 Grafana (The Face)

◦ Visualization tool  
◦ **Does NOT store data**  

◦ **Job:**  
- Connects to Prometheus as a data source  
- Queries metrics  
- Displays charts and dashboards so humans can easily understand cluster health

---

## 2️⃣ Prometheus Architecture (How it Works)

Prometheus uses a **Pull Mechanism (Scraping)**.

1. **The Target**  
   - Kubernetes API Server exposes a `/metrics` endpoint containing raw cluster data

2. **The Scraper**  
   - Prometheus periodically visits the endpoint  
   - Collects metrics  
   - Stores them in its internal database

3. **The Alert Manager**  
   - When a metric breaches a threshold (e.g., CPU > 90%)  
   - Prometheus sends alerts to Alert Manager  
   - Alert Manager forwards notifications to:
     - Slack
     - Email
     - PagerDuty

---

## 3️⃣ Key Component: Kube-State-Metrics

A **very common interview topic** covered in this session.

---

### ❌ Limitation of Default API Server
- Provides only **basic health metrics**
- Not enough for deep observability

---

### ✅ The Solution: Kube-State-Metrics

• Installed as an additional component  
• Communicates with the Kubernetes API  

• **What it Provides:**  
- Desired vs running replicas  
- Pod restart counts  
- Deployment and StatefulSet states  

⚠ Without Kube-State-Metrics, dashboards will look **almost empty**

---

## 4️⃣ Installation: The Helm Way

Monitoring stacks are **complex**, so the session recommends **Helm Charts** instead of raw YAML.

---

### 🧰 Steps

1. Add Prometheus and Grafana Helm repositories  
2. Install using:
   - `helm install prometheus ...`
   - `helm install grafana ...`

---

### ✅ Benefit of Helm

Helm automatically installs:
- Service Accounts  
- ConfigMaps  
- Services  
- Dependencies  

This significantly simplifies setup and maintenance.

---

## 5️⃣ Practical Workflow: From Install to Dashboard

The session demonstrates how to go from **installation to visualization in minutes**.

---

### Step 1: Expose Grafana

- Change Grafana Service from `ClusterIP` to `NodePort`
- Allows browser access to Grafana UI

---

### Step 2: Add Prometheus as Data Source

- Login to Grafana UI  
- Add Prometheus URL as a data source

---

### Step 3: Import Dashboard (The Cheat Code)

- Import dashboard using **ID: 3662**

◦ Community-standard dashboard  
◦ Automatically displays:
- CPU usage
- Memory usage
- Network I/O
- Pod uptime

◦ Uses metrics collected via:
- Prometheus
- Kube-State-Metrics

---

## 6️⃣ Custom Application Monitoring

What if you want to monitor **your own application**, not just Kubernetes infrastructure?

---

### 🔧 Application Requirement

• Developers must use **Prometheus Client Libraries**  
• Application must expose a `/metrics` endpoint

---

### ⚙ Prometheus Configuration

• Update Prometheus ConfigMap  
• Add new scrape target:
  - Application IP or Service
  - Scrape interval (e.g., every 10 seconds)

This allows Prometheus to monitor **custom Python/Java applications** alongside cluster metrics.

---

======================================================================

# 🌐 Day-43 | AWS LIVE PROJECT | Deploy App Using HTTPD

---

Based on the transcript for **"Day-43 | AWS LIVE PROJECT | DEPLOY APP USING HTTPD,"** this is a summary of the practical session featuring guest speaker **Varun Bansal**.

This session acts as a **capstone project**, combining **Git (local source control)** with **AWS (cloud infrastructure)** to host a **live website accessible to the public**.

---

## 1️⃣ The Goal

The objective is to take a **static website (HTML/CSS)** from a local computer and deploy it onto an **AWS EC2 instance**, making it accessible to anyone on the internet using a **Public IP address**.

---

## 2️⃣ Phase 1: Source Code Management (Git & GitHub)

Before touching AWS, the code must be moved to a **central repository**.

---

### 🔹 The Setup
- Local folder contains:
  - `index.html`
  - Asset folders (CSS, images, etc.)

---

### 🔹 The Process
- Initialize a Git repository
- Commit the website files
- Push the code to a new GitHub repository named **"AWS demo"**

---

### 🔹 Why GitHub?
In a cloud environment:
- You **cannot drag and drop files** from your laptop
- Servers **pull code from repositories**
- GitHub acts as the source of truth

---

## 3️⃣ Phase 2: Launching the Server (AWS EC2)

The session walks through provisioning a **virtual server on AWS**.

---

### 🖥️ Instance Configuration

- **OS Selection:** Red Hat Linux (Free Tier eligible)
- **Instance Type:** `t2.micro`
  - 1 vCPU
  - 1 GB RAM
  - Free Tier eligible

---

### 🔐 Key Pair (Security)

- AWS does not use passwords
- Uses a **Key Pair (.pem file)** for authentication

Steps:
- Create a key named `demo-website`
- Download the `.pem` file

📌 This key acts as the **door key** to your EC2 server

---

### 🌐 Network Settings (Critical)

#### Auto-assign Public IP
- **Must be enabled**
- Without it, the server is not reachable from the internet

#### Security Group (Firewall Rules)
Allow:
- **SSH (Port 22):** For admin login
- **HTTP (Port 80):** For public website access

⚠ If Port 80 is not allowed, the website will not load

---

## 4️⃣ Phase 3: Connecting to the Server

Once the instance is running, connection is done via terminal.

---

### 🔒 Fix Key Permissions

```bash
chmod 400 key.pem
```

Makes the key private

AWS rejects keys with open permissions

🔑 SSH Command
```bash
ssh -i key.pem ec2-user@<Public_IP>
```

Logs the user into the remote Linux server

5️⃣ Phase 4: Configuration & Deployment

Now inside the EC2 instance, the environment is prepared.

Step 1: Switch to Root User
```bash
sudo su -
```
---
Step 2: Update System & Install Git
```bash
yum update -y
yum install git -y
```
---

Step 3: Clone Website Code
```bash
git clone <repo_url>

```
Pulls the website files from GitHub to EC2

---
Step 4: Install Apache Web Server (HTTPD)
```bash
yum install httpd -y
```
---
Step 5: Move Website Files

Apache serves content from:
```bash
/var/www/html
```

Copy files:
```bash
cp -r <cloned_folder>/* /var/www/html/
```
Recursively copies website files to Apache root
---
Step 6: Start Web Server
```bash
systemctl start httpd
```
Starts the Apache service

6️⃣ Phase 5: Verification & Troubleshooting
✅ Verification

Copy the EC2 Public IP

Paste it into a browser

❌ Common Issue: HTTPS Error

Browsers default to https://

SSL is not configured

✔ Fix:
```bash
http://<Public_IP>
```

Result

Website loads successfully

Public users can access the live site using the EC2 Public IP



