# 🎯 DevOps Interview Q&A — Tool-Wise (Active Recall Edition)

> Same knowledge base as the Master Guide, restructured as **Question → Answer** pairs, grouped by tool. Answering these out loud (or writing them down) repeatedly is how this moves from "read once" to "remembered forever." Nothing from the source material is removed — only reframed as Q&A for spaced-repetition style practice.

## Table of Contents

1. [Kubernetes](#1-kubernetes)
2. [Docker](#2-docker)
3. [Terraform / IaC](#3-terraform--iac)
4. [Ansible](#4-ansible)
5. [AWS](#5-aws)
6. [Azure](#6-azure)
7. [Linux, Shell & SQL](#7-linux-shell--sql)
8. [CI/CD — Jenkins & GitHub Actions](#8-cicd--jenkins--github-actions)
9. [GitOps — Helm, ArgoCD & Flux](#9-gitops--helm-argocd--flux)
10. [Monitoring, Logging & SRE](#10-monitoring-logging--sre)
11. [Code Quality & DevSecOps](#11-code-quality--devsecops)
12. [RBAC & Security](#12-rbac--security)
13. [OpenShift](#13-openshift)
14. [Migration & DR Scenarios](#14-migration--dr-scenarios)
15. [DevOps Culture & Career](#15-devops-culture--career)

---

## 1. Kubernetes

**Q: What's the difference between a Pod and a Deployment, and why not just run raw Pods?**
> A: A **Pod** is the smallest deployable unit — one or more containers sharing a network namespace, talking via `localhost`. A raw Pod has **no self-healing and no rolling update capability**. A **Deployment** manages ReplicaSets on top of Pods, giving you rolling updates, automated rollbacks, zero-downtime deploys, and scalability. That's why Deployments — not bare Pods — are the standard for running applications.

**Q: Write a Deployment manifest with liveness/readiness probes, resource limits, and a PVC mount.**
> A:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: my-app-pvc
```

**Q: What are the Kubernetes Service types and when do you use each?**
> A:
> - `ClusterIP` (default) — internal-only cluster access
> - `NodePort` — fixed port exposed on every node
> - `LoadBalancer` — cloud-managed external LB
> - `Headless` (`clusterIP: None`) — no virtual IP; DNS resolves directly to each pod

**Q: What problem does a Headless Service solve, and how does it work?**
> A: A standard `ClusterIP` Service load-balances traffic randomly/round-robin via `kube-proxy`. This **breaks stateful systems** (databases, Kafka) where writes must go to a specific primary/leader pod while reads can go to replicas — a normal Service hides *which* pod you're hitting.
> Fix: set `clusterIP: None`. No virtual IP is assigned; CoreDNS returns **direct A-records per pod** instead of one shared IP. Best paired with a **StatefulSet**, which gives stable pod names (`db-0`, `db-1`, `db-2`).
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```
> Resulting DNS pattern: `<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local` — e.g., `db-0.my-db-headless.default.svc.cluster.local`.

**Q: Compare Standard Service vs. Headless Service.**
> A:

| Feature | Standard Service (ClusterIP) | Headless Service |
|---------|-------------------------------|----------------------|
| Cluster IP | Allocated from service CIDR | None |
| Routing | `kube-proxy` (iptables/IPVS) | Bypasses proxy — client resolves DNS directly |
| DNS | Single virtual ClusterIP | Multiple per-pod A-records/FQDNs |
| Best for | Stateless apps, REST APIs | Stateful workloads: DB leaders/replicas, Kafka, Elasticsearch, Cassandra |

**Q: How does traffic actually get forwarded between pods?**
> A: Pod sends a request to a Service DNS name → **CoreDNS** resolves it to the Service's **ClusterIP** → the Service tracks matching pods via **Endpoints/EndpointSlice** (label selectors) → **kube-proxy** (per node) intercepts and forwards traffic, typically **round-robin** via iptables/IPVS.
```
Request 1 → Pod A
Request 2 → Pod B
Request 3 → Pod C
Request 4 → Pod A  (cycle repeats)
```

**Q: Ingress vs. Ingress Controller — what's the difference, and can a Load Balancer act as an Ingress?**
> A: **Ingress** = the API object defining routing rules. **Ingress Controller** = the component (NGINX, AGIC, ALB Controller) that implements those rules. A plain L4 Load Balancer **cannot** do host/path-based HTTP routing like an Ingress Controller — it operates at a different (transport) layer.

**Q: Ingress vs. `Service: LoadBalancer` — which do you pick and why?**
> A: `LoadBalancer` allocates a **dedicated** cloud LB per exposed service → cost scales linearly; best for raw TCP/UDP, non-HTTP, low-latency websockets. **Ingress** gives you **one** entry point (single ALB/VIP) multiplexing traffic across many services via path (`/api`) or host (`app.domain.com`) rules — far more cost-effective for standard HTTP/HTTPS.

**Q: Your Pod is stuck in `CrashLoopBackOff`. What do you check?**
> A: `kubectl logs podname` (app errors) and `kubectl describe pod podname` (events — OOMKilled, probe failures). Common causes: wrong command/entrypoint, missing env vars/config, failing liveness probe, resource limits too low.

**Q: Your Service isn't reachable. What's your triage?**
> A: `kubectl get svc` (correct type?), `kubectl describe svc` (endpoints populated?). Check label mismatch between Service selector and Pod labels, pod not Ready (readiness probe failing), or NetworkPolicies blocking traffic.

**Q: A Pod is stuck in `Pending`. Why?**
> A: `kubectl describe pod` — insufficient cluster resources (CPU/memory), node selector/affinity mismatch, taints without tolerations, or PVC not bound.

**Q: Cluster nodes are under heavy CPU load — what do you do?**
> A: `kubectl top pods` to find heavy consumers, check requests/limits, use HPA, investigate inefficient code/memory leaks, consider cluster autoscaling.

**Q: A rolling update just took the app down — how do you respond?**
> A: `kubectl rollout status deployment <name>` to check state, then `kubectl rollout undo deployment <name>` to roll back. Investigate probes, image issues, config errors afterward.

**Q: You updated a ConfigMap but pods didn't pick up the change. Why, and what's the fix?**
> A: ConfigMaps **don't auto-restart Pods**. Fix: `kubectl rollout restart deployment <name>`.

**Q: An app can't read its Secrets. What's your checklist?**
> A: `kubectl get secrets` to confirm it exists → check the mount method (env var vs. volume) → confirm correct key names → confirm RBAC permissions.

**Q: Pods intermittently can't talk to each other. What are the possible causes and how do you debug?**
> A: Causes: NetworkPolicies blocking traffic, CoreDNS issues, service misconfiguration, pod restarts/scaling. Debug: `kubectl exec` into a pod and test `curl`/`nslookup`.

**Q: Data disappears after a pod restart. Why, and how do you fix it?**
> A: The pod was using **ephemeral storage** instead of persistent storage. Fix: use a **PersistentVolume (PV) + PersistentVolumeClaim (PVC)** with the correct StorageClass, and mount it properly.

**Q: A node shows `NotReady`. What's your investigation path?**
> A: `kubectl describe node <name>` — check kubelet status, disk/memory pressure, network issues. SSH into the node, restart kubelet, check `journalctl -u kubelet`.

**Q: A Deployment was deleted but its Pods are still running. Why?**
> A: Orphaned ReplicaSets, pods spawned by another controller (StatefulSet/DaemonSet/standalone), finalizer hooks blocking cleanup, or `kube-controller-manager` API communication lag.

**Q: You updated the image tag in the YAML, but the Pod is still running the old image. What's going on?**
> A: Check `imagePullPolicy`. Using the mutable `:latest` tag **without** `imagePullPolicy: Always` prevents kubelet from pulling the new image — it assumes the tag it already has locally is fine.

**Q: HPA is configured, the CPU threshold is exceeded, but pod count stays at 1. Why?**
> A: Missing Metrics Server, missing `resources.requests` on the container (HPA can't calculate % utilization without a baseline request), or min/max replica constraints capping it at 1.

**Q: A container gets `OOMKilled` even though no limits were set. Why?**
> A: Without limits, a container can consume as much host node memory as is available. When memory spikes, the **Linux kernel OOM killer** terminates processes on the node — or, if a namespace `LimitRange` exists, it enforces a default or blocks pod creation entirely.

**Q: How would you design a zero-downtime deployment on Kubernetes?**
> A: Combine Rolling Updates, Blue-Green, or Canary (via Argo Rollouts, Istio traffic routing, feature flags). Set accurate `readinessProbe`/`livenessProbe`/`startupProbe`. Configure `maxUnavailable: 0` and `maxSurge: 1` (or `25%`) so old pods only terminate after new pods pass readiness checks. For canary: shift traffic 5% → 20% → 50% → 100% while watching error rates/latency.

**Q: A worker node enters `NotReady`. Walk through what happens internally.**
> A: The `node-lifecycle-controller` detects missed heartbeats and marks the node `NotReady`. The scheduler stops assigning new pods to it; existing pods remain until the **eviction timeout** (default 5 minutes), then are rescheduled elsewhere. **PodDisruptionBudgets (PDB)** guarantee minimum available instances during this shuffle. Stateless pods recreate easily; **StatefulSets** need the CSI driver to unmount/remount PVs onto the new node.

**Q: What happens if etcd crashes?**
> A: etcd is the consistent key-value store holding the entire cluster state (nodes, pods, secrets, configs). If it crashes, `kube-apiserver` loses read/write capability and becomes unresponsive. **Running workloads keep running** — kubelet and the container runtime continue executing already-running containers. But the control plane **freezes**: no new scheduling, scaling, secret retrieval, or deployment updates. Mitigation: run multi-node HA clusters (odd quorum: 3 or 5 nodes) across failure zones, and restore via `etcdctl snapshot restore` from automated snapshots.

**Q: You're paged for a production outage. What's your triage order?**
> A: Structured triage, never panic-reboot: 1) **Telemetry & Logs** — CPU, memory, IOPS, connection pools, error rate surges. 2) **Layer Isolation** — is the fault Network/DNS, Ingress/LB, Application logic, or downstream Databases? 3) **Fast Mitigation** — if unresolved in 5–10 minutes, roll back to the last known-good deployment, saving crash logs for post-mortem.

**Q: Trace a packet's path from a Pod to the internet.**
> A: Pod Network Namespace → veth pair → Linux Bridge/CNI plugin (Calico, Cilium, Flannel) → node routing tables → iptables/eBPF (SNAT) → node's physical NIC (`eth0`) → VPC Router/NAT Gateway → Internet Gateway → external web.

**Q: How do you harden a Kubernetes cluster (defense-in-depth)?**
> A: **Admission control**: OPA Gatekeeper or Kyverno webhooks. **Identity & isolation**: strict least-privilege RBAC, namespace boundaries, NetworkPolicies blocking flat pod-to-pod access. **Runtime & supply chain**: sign images (Sigstore/Cosign), scan base images (Trivy), runtime threat detection (Falco), KMS encryption for etcd secrets.

**Q: Walk through the full lifecycle of `kubectl apply -f deployment.yaml`.**
> A: 1) `kubectl` sends an HTTP request to `kube-apiserver`. 2) API Server runs Authentication → Authorization (RBAC) → Mutating/Validating Admission Controllers. 3) Desired state is persisted into **etcd**. 4) The Deployment/ReplicaSet Controller detects the diff and creates Pod objects. 5) `kube-scheduler` assigns pods to nodes based on resource filters/affinities. 6) kubelet on the target node instructs the container runtime (containerd/CRI-O) to pull images and start containers, then reports status back to the API Server.

**Q: What is the Sidecar pattern, and why use it?**
> A: An auxiliary container deployed alongside the main app container in the same pod — used for logging, proxying, or secret synchronization, sharing the pod's network/storage namespace.

**Q: What does a Service Mesh give you that plain Kubernetes networking doesn't?**
> A: Managed service-to-service communication, mTLS encryption, fine-grained traffic shifting, and observability — via tools like Istio or Linkerd.

**Q: What is GitOps in the Kubernetes context?**
> A: Reconciling cluster state automatically against a declarative Git repository, using tools like Flux CD or Argo CD — see [Section 9](#9-gitops--helm-argocd--flux) for depth.

**Q: What is AKS, and how does it split responsibility between Azure and you?**
> A: Azure's managed Kubernetes service. Azure manages significant portions of the control-plane infrastructure (API Server, Scheduler, Controllers, etcd). You still manage applications, node pools, K8s configuration, networking, RBAC, security, scaling, and observability.

**Q: What's an AKS Node Pool?**
> A: A group of nodes with a particular VM configuration. **System Node Pool** runs core K8s workloads; **User Node Pool** runs application workloads.

**Q: HPA vs. Cluster Autoscaler vs. KEDA — what does each scale?**
> A: **HPA** changes the number of **Pods** (e.g., 3→10) based on CPU/memory/custom metrics. **Cluster Autoscaler** changes the number of **Nodes** (e.g., 3→6). **KEDA** scales workloads based on **event-driven metrics** (queue depth, etc.).

**Q: How does AKS authenticate to pull images from ACR, and what permission does it need?**
> A: `AKS → Identity → Azure RBAC → ACR → Pull Image`. The common permission is `AcrPull` — never give AKS full `Contributor` access to ACR when it only needs to pull images.

---

## 2. Docker

**Q: How do you reduce a Docker image's size?**
> A: Use minimal base images (Alpine, Distroless), multi-stage builds, chain `RUN` commands with `&&` to reduce layer count, and avoid unnecessary dependencies.

**Q: How do you clean up unused Docker resources?**
> A: `docker system prune -a` — prunes dangling/unused images, containers, and volumes.

**Q: What is a Dockerfile?**
> A: A text manifest of instructions packaging an app + dependencies into an immutable, platform-independent image.

**Q: `ADD` vs. `COPY`?**
> A: `COPY` copies local files/dirs plainly. `ADD` also auto-extracts local tar archives and can fetch remote URLs.

**Q: `ENTRYPOINT` vs. `CMD`?**
> A: `ENTRYPOINT` is the immutable base executable. `CMD` provides default arguments, overridable at `docker run`.

**Q: What does `WORKDIR` do?**
> A: Sets the working directory for subsequent `RUN`/`CMD`/`ENTRYPOINT`/`COPY` instructions.

**Q: What does `EXPOSE` actually do?**
> A: It's purely documentation of which ports the container listens on — `-p` is still required at runtime to actually map ports.

**Q: Why would a Dockerfile have multiple `FROM` instructions?**
> A: This is standard for **multi-stage builds** — used to discard build-time dependencies from the final image.

**Q: `RUN` vs. `CMD` vs. `ENTRYPOINT` — when does each execute?**
> A: `RUN` executes at **build time** and commits a new layer. `CMD`/`ENTRYPOINT` execute at **container startup**.

**Q: `ARG` vs. `ENV`?**
> A: `ARG` only exists during the build (`docker build --build-arg`). `ENV` persists into the running container.

**Q: Explain multi-stage builds and why they matter.**
> A: You compile in a "builder" image (Maven/Go/Node), then copy only the compiled output into a lightweight runtime image (Alpine/Distroless) — dramatically smaller final image, with build tools never shipped to production.

**Q: What's risky about `COPY . .` in a CI/CD pipeline?**
> A: It blindly copies test artifacts, `.git` history, cache folders, and potential secrets. Always pair it with a strict `.dockerignore`.

**Q: How do you trace image provenance?**
> A: Use `LABEL` (maintainer, Git commit SHA, build version) for traceability.

**Q: How should secrets be handled during a Docker build?**
> A: Never bake them into `ENV`/`ARG`. Inject at runtime via env vars/secret stores, or use BuildKit secret mounts (`--mount=type=secret`).

**Q: A container starts and immediately exits. How do you debug it?**
> A: `docker logs <container_id>`, then inspect the `CMD`/`ENTRYPOINT` foreground process — the container exits when its PID 1 process finishes or crashes.

**Q: Why is the `:latest` tag risky?**
> A: It breaks reproducibility, silently pulls unexpected base image changes, and hurts caching. **Always pin specific version tags or SHA digests.**

**Q: How does Docker's build cache invalidation work?**
> A: Cache invalidates from the **first modified layer downward**. Put infrequently-changing instructions (e.g., `package.json`/`pom.xml`) **before** copying volatile source code to maximize cache hits.

**Q: How do you achieve deterministic Docker builds?**
> A: Pin explicit base image tags/digests, lock dependencies (`package-lock.json`), avoid mutable downloads.

**Q: How do you harden a Docker image for security?**
> A: Run as non-root (`USER <non-root-uid>`), use minimal base images (Alpine/scratch/Distroless), scan images (Trivy/Grype), keep `.dockerignore` current.

**Q: Containers vs. VMs — what's the fundamental difference?**
> A: Containers virtualize at the **OS level**, sharing the host kernel — lightweight. VMs virtualize at the **hardware level** via a hypervisor, requiring a full dedicated guest OS — heavier footprint.

---

## 3. Terraform / IaC

**Q: Multiple developers are applying Terraform simultaneously and the state gets corrupted. How do you prevent this?**
> A: Use a remote backend with locking — e.g., S3 (stores state) + DynamoDB (prevents concurrent writes via a lock table):
```hcl
backend "s3" {
  bucket         = "my-tf-state"
  key            = "prod/terraform.tfstate"
  region         = "ap-south-1"
  dynamodb_table = "tf-lock"
}
```
On Azure: Azure Blob Storage with native lease-based locking.

**Q: How do you recover if the Terraform state file is deleted?**
> A: Restore via remote backend versioning (S3/Azure Blob object versioning), or rebuild the mapping using `terraform import`.

**Q: Your `.tfstate` has secrets stored in plaintext. What do you do?**
> A: Encrypt the remote backend, use ephemeral outputs, and delegate secret retrieval to KMS/Vault at runtime instead of storing raw secrets in state. Marking a variable `sensitive = true` only redacts console/log output — it does **not** remove the value from the raw state file.

**Q: Someone manually changed a resource in the cloud console. How do you handle the drift?**
> A: `terraform plan` detects the drift. Use `terraform apply` to revert the change back to code-defined state, OR `terraform import` if the resource was never tracked at all. Enforce IaC discipline going forward — avoid manual console changes.

**Q: How do you structure Terraform for multiple environments (Dev/QA/Prod)?**
> A: Two common approaches:
> - **Approach A (preferred in enterprise) — Module Directory Pattern**: reusable versioned modules under `/modules`, with separate `/environments/dev`, `/qa`, `/prod` directories, each with its own `backend.tf`, `main.tf` (calling shared modules), and `terraform.tfvars`.
> - **Approach B — Workspace-Based**: single config, `terraform workspace select <env>`. *Interview note: workspaces share the same backend storage, risking cross-environment blast radius — module-per-directory with separate state is generally preferred.*
> Also common: `terraform apply -var-file=dev.tfvars` / `-var-file=prod.tfvars` with separate backend configs per environment.

**Q: Why use Terraform modules, and how do you pass data between them?**
> A: Modules give you **Reusability (DRY)** — define infra once, reuse across environments — and **Separation of Concerns** (networking, control-plane, compute in separate directories). A child module's internal resources are private by default; it must expose a value via its own `outputs.tf`. The root module then references it as `module.<module_name>.<output_name>`.
```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
  env    = var.environment
}
module "eks" {
  source     = "./modules/eks"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
```

**Q: Why avoid hardcoding values inside child modules?**
> A: Hardcoding breaks reusability across environments. All environment-specific values (instance types, CIDRs, replica counts) must flow through `variables.tf`.

**Q: `terraform plan` shows a resource will be destroyed unexpectedly. How do you handle it?**
> A: **Never run `apply` blindly** — inspect the plan to see what triggered the replacement (immutable argument change, or an identifier rename). If a resource block was renamed (e.g., `aws_db_instance.db1` → `db2`), Terraform sees `db1` as deleted and `db2` as new. Fix with a `moved` block (1.1+):
```hcl
moved {
  from = aws_db_instance.db1
  to   = aws_db_instance.db2
}
```
As a safeguard, add `lifecycle { prevent_destroy = true }` on critical stateful resources so `apply` fails rather than destroying data.

**Q: How do you handle sensitive data (like DB passwords) in Terraform code?**
> A:
```hcl
variable "db_password" {
  sensitive = true
}
```
Combined with environment variables and a secret manager (AWS Secrets Manager, Azure Key Vault, Vault).

**Q: How do you update an EC2 instance with zero downtime?**
> A:
```hcl
lifecycle {
  create_before_destroy = true
}
```
Combined with a load balancer for a smooth switchover.

**Q: A resource fails because its dependency isn't ready yet. What's the fix?**
> A: Terraform auto-handles most dependencies via implicit references, but you can enforce ordering explicitly:
```hcl
depends_on = [aws_instance.app]
```
Use this only when no direct attribute reference exists but ordering still matters (e.g., IAM role attachment before EKS cluster creation).

**Q: You need to manage hundreds of similar resources. `count` or `for_each`?**
> A: Depends on homogeneity:

| Scenario | Construct | Why |
|---|---|---|
| Identical config, distinct names | `count = 20` | Index addressing — simple for homogeneous arrays |
| Heterogeneous configs (different sizes/subnets/OS) | `for_each = var.instances_map` | Map keys — removing one item doesn't re-index the rest |

> **Pitfall**: deleting a specific `count` index can cause Terraform to destroy/recreate all subsequent indexed items. Best practice: use `for_each` with unique map keys.

**Q: How do you conditionally create a resource only in a specific environment?**
> A:
```hcl
count = var.environment == "dev" ? 1 : 0
```

**Q: What do `lifecycle` and `dynamic` blocks do?**
> A: `lifecycle` controls resource behavior: `create_before_destroy`, `prevent_destroy`, `ignore_changes`. `dynamic` blocks repeat nested arguments (e.g., generating multiple security group ingress rules from a list).

**Q: A `terraform apply` fails mid-way. How do you recover?**
> A: Terraform is **not fully transactional**. Fix the underlying issue and re-run `terraform apply`. Use version control for code-level rollback; consider a blue/green strategy for infra-level rollback.

**Q: How do you bring existing (unmanaged) infrastructure under Terraform control?**
> A:
```bash
terraform import aws_instance.example i-123456
```
Then write a matching `.tf` configuration, and run `terraform plan` to verify it syncs cleanly. (Terraform 1.5+ offers declarative `import {}` blocks to auto-generate config.)

**Q: `resource` vs. `data` blocks — what's the difference?**
> A: `resource` declares infrastructure Terraform **creates and manages**. `data` is a **read-only query** for existing infrastructure created outside the current root module (e.g., fetching a pre-existing subnet ID or the latest AMI).

**Q: What are Terraform meta-arguments?**
> A: Special top-level arguments usable in any resource block: `depends_on`, `count`, `for_each`, `provider`, `lifecycle`.

**Q: What is Terraform Sentinel?**
> A: HashiCorp's Policy-as-Code framework (Cloud/Enterprise) that evaluates guardrails before `apply`. **Soft Mandatory** policies warn or allow admin override (e.g., missing cost-center tag). **Hard Mandatory** policies strictly block `apply` on failure (e.g., exposing port 22 to `0.0.0.0/0`, non-whitelisted VM sizes).

**Q: How does Terraform CLI typically run inside a CI/CD pipeline, securely?**
> A: Runs on pipeline runners (self-hosted agents/GitHub Actions runners/Jenkins nodes) with the Terraform CLI installed. State and locking via S3+DynamoDB or Azure Blob lease locking. Credentials injected dynamically via **OIDC/federated workload identities** or IAM instance profiles — avoiding long-lived API keys.

---

## 4. Ansible

**Q: Terraform vs. Ansible — what's the division of labor?**
> A: *"Terraform builds the house. Ansible furnishes and maintains it."* **Terraform** = infrastructure provisioning (VMs, VPCs, subnets, storage). **Ansible** = configuration management — installs packages, manages config files, starts services *after* provisioning.

**Q: Push vs. pull configuration management — where does Ansible fit?**
> A: **Ansible** is **push-based and agentless** — the control node pushes changes over SSH (or WinRM for Windows). **Puppet/Chef** are **pull-based** — local agents periodically pull config from a master server.

**Q: Can the Ansible control node run on Windows?**
> A: **No** — it requires a POSIX-compliant environment (Linux/Unix). On Windows, it can only run inside **WSL**. However, Ansible **can manage** Windows target nodes, via the **WinRM** connection plugin.

**Q: Static vs. Dynamic inventory — what's the difference?**
> A: **Static**: manually defined hosts/groups/IPs (default: `/etc/ansible/hosts`). **Dynamic**: scripts (Python/cloud plugins) that query cloud providers or Terraform outputs for live host IPs — no manual copy-pasting.

**Q: What's the difference between core and custom Ansible modules?**
> A: **Core modules** are built-in (`yum`, `apt`, `copy`, `service`, `win_copy`). **Custom modules** are user-written (commonly Python) for bespoke automation.

**Q: How do you structure a reusable Ansible playbook using roles?**
> A: Break it into: `tasks/` (main execution steps), `handlers/` (conditional tasks, run only when notified), `vars/`/`defaults/` (variables), `templates/`/`files/` (config files + Jinja2 templates), `meta/` (role metadata/dependencies).

**Q: How do you encrypt secrets in an Ansible playbook?**
> A: `ansible-vault encrypt <secrets_file.yml>` — encrypts passwords, tokens, and keys inside YAML files.

**Q: Write a basic Ansible playbook to install a package.**
> A:
```yaml
- hosts: all
  become: true
  tasks:
    - name: Install git
      yum:
        name: git
        state: present
```

**Q: How do handlers work, and what's the common mistake?**
> A: Handlers run tasks only when notified by a change:
```yaml
tasks:
  - name: Configure Apache
    template:
      src: httpd.conf.j2
      dest: /etc/httpd/conf/httpd.conf
    notify: restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```
> ⚠️ The `notify:` string must **exactly match** the handler's `name:`. Handlers run once at the end of a play, only if triggered.

**Q: How do you avoid repetitive tasks in a playbook?**
> A: Use loops:
```yaml
- name: Install list of packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - wget
    - vim
    - zip
```

**Q: How do you run only part of a playbook?**
> A: Use tags:
```yaml
tasks:
  - name: Install packages
    yum:
      name: git
      state: present
    tags:
      - install
```
```bash
ansible-playbook playbook.yml --tags "install"
```

**Q: How does Ansible get server IPs from Terraform after provisioning?**
> A: Two options:
> - **Manual output passing**: `export instance_ip=$(terraform output -raw instance_ip)` then `ansible-playbook -i ${instance_ip}, playbooks/setup.yaml`
> - **Dynamic inventory script (preferred at scale)**: a Python script calls `terraform output -json`, parses instance IPs, and generates a live Ansible inventory automatically — no manual copy-pasting.
```python
#!/usr/bin/env python
import json, subprocess

def get_terraform_output():
    output = subprocess.check_output(['terraform', 'output', '-json'], universal_newlines=True)
    return json.loads(output)

def main():
    inventory = {'all': {'hosts': []}}
    tf_output = get_terraform_output()
    for resource in tf_output['instances']['value']:
        inventory['all']['hosts'].append(resource['public_ip'])
    print(json.dumps(inventory, indent=2))

if __name__ == "__main__":
    main()
```

**Q: How do you manage Terraform + Ansible across multiple environments?**
> A: Separate Terraform **workspaces** (`prod`, `staging`, `dev`) paired with separate Ansible **inventory files** (`prod.yaml`, `dev.yaml`, `staging.yaml`), or dynamic scripts pulling from each workspace's state.

**Q: How often should Terraform and Ansible run in a pipeline?**
> A: **Terraform** — weekly, or only when infrastructure changes. **Ansible** — on every commit/PR, to deploy and configure services.

---

## 5. AWS

**Q: How do you increase disk space on a Linux server running in AWS?**
> A: Two-step process: **expand the underlying storage** (EBS volume resize) and then **resize the filesystem** to utilize the new space (`growpart` + `resize2fs`/`xfs_growfs`).

**Q: How do you restrict access to a specific S3 object?**
> A: Use S3 Bucket Policies or IAM Policies with object-level permissions.

**Q: How do you install software on an EC2 instance automatically at launch?**
> A: Use **User Data** — a bootstrap script executed on first boot.

**Q: Where should a NAT Gateway be placed, and why?**
> A: In a **Public Subnet**. It lets instances in a private subnet reach the internet (for updates, downloads) without exposing them to inbound internet traffic. Requirements: must be in a public subnet, must have an Elastic IP (EIP), requires a route to an Internet Gateway (IGW).

**Q: How do you connect a resource in AWS Account A to a resource in AWS Account B?**
> A: Establish a **VPC Peering** connection: create the peering request from one account (Requester), accept it in the other (Accepter), update route tables in both VPCs, and ensure security groups/NACLs allow the traffic. Alternatives depending on scale/architecture: **Transit Gateway** (multi-VPC scale), **PrivateLink** (expose specific services securely), or **VPN/Direct Connect** (hybrid/external connectivity).

**Q: How do you stop communication between pods in two different Kubernetes namespaces?**
> A: Define a `NetworkPolicy` restricting ingress/egress. By default pods are non-isolated, so you must explicitly create policies denying cross-namespace traffic unless allowed — using `namespaceSelector` or `podSelector`. Unmatched traffic is denied by default once a policy exists (CNI-dependent).

**Q: Traffic on your single-EC2 app spikes 10x. What's your scaling plan?**
> A: Move behind an **Elastic Load Balancer**, use **Auto Scaling Groups**, store static content in **S3 + CloudFront**, use **RDS Multi-AZ** for reliability, and optionally add **ElastiCache**.

**Q: Your AWS bill doubled. How do you investigate and reduce cost?**
> A: Analyze usage via **Cost Explorer**, identify and resize/stop underutilized EC2 instances, use **Reserved Instances/Savings Plans** for predictable workloads, enable **Auto Scaling** to avoid overprovisioning, move infrequently accessed S3 data to **Glacier**, and right-size RDS.

**Q: What's your DR strategy if an entire AWS region fails?**
> A: Multi-region architecture with cross-region replication (S3/RDS read replicas), Route 53 failover routing, and a strategy chosen by RTO/RPO: **Backup & Restore** (cheap, slow) → **Pilot Light** → **Warm Standby** → **Active-Active** (fastest, costliest).

**Q: How do you securely store sensitive data in S3?**
> A: Encryption at rest (SSE-S3/SSE-KMS) and in transit (HTTPS), strict least-privilege IAM policies, bucket policies + block public access, versioning + MFA delete, and CloudTrail monitoring.

**Q: On ECS, how do your microservices communicate?**
> A: Service discovery (AWS Cloud Map), an internal load balancer for synchronous calls, and asynchronous communication via SQS (decoupling) or SNS (fan-out) — secured with IAM roles + VPC networking.

**Q: Your logs are scattered and the app fails randomly. What's your approach?**
> A: Centralize with **CloudWatch Logs + Log Insights**, set alarms for CPU/memory/error rates, enable tracing with **AWS X-Ray**, and build dashboards.

**Q: RDS is slow due to heavy reads. What are your options?**
> A: Add **Read Replicas**, use **ElastiCache**, optimize queries/indexing, consider migrating to **Aurora**, and use connection pooling.

**Q: Design a CI/CD pipeline entirely on native AWS services.**
> A: **CodeCommit** (repo) → **CodeBuild** (build/test) → **CodeDeploy** (deploy) → **CodePipeline** (orchestration) — add approval steps and rollback strategies.

**Q: How would you migrate an EC2 monolith to serverless?**
> A: Break it into microservices, use **Lambda** for compute, **API Gateway** for APIs, **DynamoDB/S3** for data, and event-driven architecture (SQS/SNS).

**Q: Different teams need different AWS access levels. How do you manage this?**
> A: IAM roles/policies with least privilege, IAM groups for team-based access, MFA enforcement, and AWS Organizations for multi-account governance.

---

## 6. Azure

**Q: What is a Resource Group, and is it a network boundary?**
> A: A logical container for Azure resources sharing the same lifecycle/administration boundary. **It's a management boundary, not necessarily a network boundary.**

**Q: Describe Azure's subscription hierarchy.**
> A: `Azure Tenant → Management Groups → Subscriptions → Resource Groups → Resources`. A subscription provides billing boundary, resource boundary, access-control boundary, and quota boundary.

**Q: Microsoft Entra ID vs. Azure RBAC — what's the difference?**
> A: **Entra ID** (formerly Azure AD) handles **Authentication (AuthN)** — identities of users, groups, service principals, external federations. **Azure RBAC** handles **Authorization (AuthZ)** — what an authenticated identity is permitted to do on Azure resources (`Reader`, `Contributor`, `Owner`, `AcrPull`, etc.).
```
GitHub Actions → OIDC authentication → Entra ID → Azure RBAC → ACR / AKS / Storage
```

**Q: What is Managed Identity, and why is it better than storing credentials?**
> A: Managed Identity lets Azure resources authenticate to other Azure services **without storing passwords or client secrets**.
```
AKS → Managed Identity → Azure Key Vault
```
> Avoids manually storing/rotating `CLIENT_ID`, `CLIENT_SECRET`, `PASSWORD`. Improves security, credential rotation, auditing, and least privilege.

**Q: System-assigned vs. User-assigned Managed Identity?**
> A: **System-assigned**: created with the resource, deleted when the resource is deleted (1:1 lifecycle). **User-assigned**: created as a standalone resource, can be assigned to multiple resources independently of their lifecycles.

**Q: Managed Identity vs. Service Principal?**
> A: **Service Principal**: an app registration requiring **manual** credential/secret rotation. **Managed Identity**: an Azure-managed credential, **automatically rotated** by Entra ID.

**Q: How would an app running on AKS securely access Key Vault?**
> A: Use **Workload Identity** or a managed identity-based approach rather than putting Azure credentials inside the Pod:
```
Application Pod → AKS Workload Identity → Microsoft Entra ID → Azure RBAC → Key Vault
```

**Q: Can an existing Azure subnet be extended from /24 to /23?**
> A: **No.** You cannot expand/modify the IP address range of an existing subnet while resources or active NICs are attached to it.

**Q: NSG vs. Azure Firewall?**
> A: **NSG**: basic network traffic filtering at the subnet/NIC level (allow/deny rules by source, destination, port, protocol, direction). **Azure Firewall**: a managed, centralized network security service with more advanced capabilities.

**Q: Azure Load Balancer vs. Application Gateway vs. Traffic Manager vs. Front Door?**
> A:

| Service | Layer | Scope | Notes |
|---------|-------|-------|-------|
| Load Balancer | L4 (TCP/UDP) | Regional | Basic traffic distribution |
| Application Gateway | L7 (HTTP/HTTPS) | Regional | URL/host routing, TLS termination, WAF |
| Traffic Manager | DNS-level | Global | Routes at DNS layer, no data path involvement |
| Front Door | L7 | Global | Edge routing + CDN + WAF, actively proxies traffic |

**Q: Can a Load Balancer function as an Ingress?**
> A: No — a standard L4 Load Balancer can't do HTTP host/path-based routing; that's the job of an Ingress Controller operating at L7.

**Q: Private Endpoint vs. Service Endpoint?**
> A: **Private Endpoint**: gives the PaaS resource a private IP inside your VNet — not exposed publicly for that access path. **Service Endpoint**: extends VNet identity to the Azure service over the Azure backbone, but the service still has a public IP internally.

**Q: VNet Peering is set up but VNet B can't reach VNet A. What's your troubleshooting?**
> A: Check non-transitive peering limitations, asymmetric/missing peering connections (must be initiated on **both** sides), overlapping CIDR blocks, and restrictive NSG inbound/outbound rules.

**Q: A Private Endpoint is created but the on-prem server can't connect. Why?**
> A: DNS resolution failure (private DNS zone not forwarding/resolvable via Azure Private DNS Resolver from on-prem), missing route tables, or VPN/ExpressRoute gateway routing table omissions.

**Q: What are the ways to connect on-premises infrastructure to Azure?**
> A: Site-to-Site VPN, Point-to-Site VPN, and dedicated private lines via Azure ExpressRoute.

**Q: What are the Azure storage types, and when do you use each?**
> A: **Blob Storage** (object storage — files, backups, logs, images, artifacts), **Azure Files** (managed file shares), **Queue Storage** (message queueing), **Table Storage** (NoSQL key-value).

**Q: How do you access an Azure Storage Account from AKS when it's in a different region?**
> A: It works but adds latency. Use the Blob CSI driver/SDK-based access with Managed/Workload Identity; ensure the network path (private endpoint/VNet peering if private) allows cross-region connectivity.

**Q: Azure Functions vs. Web Apps vs. Logic Apps?**
> A: **Functions**: serverless, event-driven compute for short-lived code. **Web Apps (App Service)**: managed hosting for web applications, supports deployment slots. **Logic Apps**: low-code workflow automation/orchestration connecting services and APIs.

**Q: Your Application Gateway backend health shows "down." What are the likely culprits?**
> A: Mismatched custom health probe path/port, backend NSG blocking traffic from the App Gateway subnet, SSL certificate mismatch, or the backend service listening on `localhost` instead of `0.0.0.0`.

**Q: A developer gets "denied by policy" trying to create a VM. What do you check?**
> A: Investigate the Azure Policy definitions and their evaluation **effects**: `Deny`, `Audit`, `Modify`, `DeployIfNotExists`.

**Q: Azure Policy vs. RBAC — what's the conceptual difference?**
> A: **RBAC** controls **who** can do **what** (e.g., "John cannot delete AKS"). **Policy** controls **what configurations/resources are allowed or required** (e.g., "Resources must have required tags").

**Q: What is an Azure Landing Zone?**
> A: A structured foundation for enterprise Azure environments addressing Identity, Networking, Governance, Security, Subscriptions, Policies, Logging, and Management — especially relevant for regulated enterprises like healthcare or banking.

**Q: What do HTTP status codes 500/502/503/504 each mean?**
> A: **500** — unhandled error in the app itself. **502** — upstream sent an invalid response to the gateway/proxy. **503** — service temporarily unavailable (overloaded/maintenance). **504** — upstream server didn't respond in time.

**Q: Why prefer OIDC federation over storing a long-lived Azure client secret in GitHub Actions?**
> A: A client secret is long-lived — if leaked, an attacker has standing access. OIDC provides short-lived federated authentication:
```
GitHub Actions → OIDC Token → Microsoft Entra ID → Federated Credential → Azure RBAC → Azure Resources
```

**Q: Azure Monitor vs. Log Analytics?**
> A: **Azure Monitor** is the broad platform (Metrics, Logs, Alerts, Application telemetry). **Log Analytics** is the workspace for querying logs using **KQL (Kusto Query Language)**:
```kql
AzureActivity
| where TimeGenerated > ago(1h)
| summarize count() by ResourceGroup
```

**Q: How do you monitor an AKS cluster end-to-end?**
> A: K8s-level: `kubectl get pods`, `kubectl top pods/nodes`. Azure-level: Azure Monitor, Container Insights, Log Analytics, Application Insights. Watch: CPU, memory, pod restarts, node health, API latency, error rates, request volume, disk, network, exceptions.

**Q: An AKS application suddenly becomes unavailable. What's your systematic troubleshooting flow?**
> A: Establish scope/impact → check recent changes, health, Resource Health → for AKS: check ingress/App Gateway, services, endpoints, pods, events, logs → investigate dependencies (Key Vault, DB, DNS, networking) → rollback if deployment-related → RCA afterward.

**Q: An AKS app can't access Key Vault. Walk through your checklist.**
> A: `Pod → Workload/Managed Identity → Entra ID → Azure RBAC → Key Vault`. Checklist: correct identity used? federated correctly? has Key Vault permissions? network path accessible? DNS resolving? secret name correct? audit logs? recent changes?

**Q: AKS can't pull an image from ACR. What's your process?**
> A: `AKS → Identity → RBAC → ACR → Image`. `kubectl describe pod` → look for `ImagePullBackOff`/`ErrImagePull` → check image name/tag, ACR availability, AKS identity, `AcrPull` permission, network connectivity, private endpoint/DNS.

**Q: Your app is slow but the infrastructure looks healthy. How do you investigate before scaling?**
> A: Don't just scale. Check: `App metrics → Latency → Error rate → CPU/Memory → Database → External APIs → Network → Recent deployments`. Likely causes: DB query performance, connection pool exhaustion, external API latency, CPU throttling, memory pressure, GC pauses, network latency, or code changes.

**Q: RPO vs. RTO — define both with an example.**
> A: **RPO (Recovery Point Objective)**: how much data loss is acceptable. **RTO (Recovery Time Objective)**: how quickly the system must be restored. Example: RPO = 15 minutes, RTO = 1 hour.

**Q: Region vs. Availability Zone?**
> A: **Region**: a geographic Azure location (e.g., East US, West Europe). **Availability Zone**: a physically separate datacenter within a region.

---

## 7. Linux, Shell & SQL

**Q: What is "load average" and how do you interpret it?**
> A: The 1-, 5-, and 15-minute load metrics shown by `uptime`/`top`, interpreted relative to available CPU cores.

**Q: How do you prevent disk saturation from growing log files?**
> A: `logrotate` — automates rotation to prevent disk saturation and node pressure.

**Q: How do you check long-running processes?**
> A:
```bash
ps -eo pid,etime,cmd
top
htop
```

**Q: Write a script that checks if a path is a directory, and if so, counts and prints the number of files in it — otherwise exits non-zero.**
> A:
```bash
#!/bin/bash
path="$1"

if [ -d "$path" ]; then
  count=$(find "$path" -maxdepth 1 -type f | wc -l)
  echo "Directory exists at: $path — File count: $count"
else
  echo "Not a valid directory: $path"
  exit 1
fi
```

**Q: Find all log files older than 7 days.**
> A:
```bash
find /var/log -type f -name "*.log" -mtime +7
```

**Q: Write a script that counts how many files (recursively) contain a specific word.**
> A:
```bash
#!/bin/bash
word="username"
dir="/path/to/search"
count=$(grep -rl "$word" "$dir" | wc -l)
echo "Number of files containing the word: $count"
```
> `-r` = recursive, `-l` = list matching filenames only, `wc -l` = count matched files.

**Q: SQL: how do you find the 7th highest value in a column?**
> A: Order descending, limit to 7, then flip the subquery to ascending and take the first row:
```sql
SELECT marks FROM (
  SELECT marks FROM students
  ORDER BY marks DESC
  LIMIT 7
) AS top7
ORDER BY marks ASC
LIMIT 1;
```
> Logic: grab the top 7 → flip their order → the first of the flipped list is the 7th highest.

**Q: How do you run a command in the background?**
> A: `nohup command &`

**Q: What are your go-to Linux commands for file inspection, text processing, users/networking, service management, and archiving?**
> A:

| Category | Commands |
|---|---|
| File inspection | `head -n <N>`, `tail -n <N>`, `tail -f` (live-stream) |
| Text processing | `sed` (inline edits), `awk` (field-based), `grep -i/-r/-c` |
| Users/networking | `whoami`, `w`/`users`, `uptime`, `last`, `ifconfig`/`hostname -I` |
| Service management | `systemctl start/restart/status`, `ps -ef | grep`, `top`, `free -m/-g` |
| Archiving | `zip -r`, `unzip`, `tar -cvf`/`tar -xvf`, `scp` |

**Q: What real-world Linux automation should you be ready to discuss in interviews?**
> A: Cost optimization scripts (auto-shutdown non-prod servers outside business hours), mass maintenance (rolling restarts with health checks between nodes), log rotation/disk space remediation (cron detects `/var/log` > 85%, archives to S3), and CrashLoopBackOff/OOM remediation scripts (triage stuck pods, extract exit codes like OOMKilled/137, dump logs, notify on-call).

---

## 8. CI/CD — Jenkins & GitHub Actions

**Q: Describe your end-to-end CI/CD pipeline stages.**
> A: Code checkout → build/packaging (Maven for Java, pip/npm for Python/Node) → security scans → container image generation → deployment. Full production-grade version: Workspace Clean (`cleanWs()`) → Checkout SCM → Build & Package → Security & Quality Gates (SonarQube + OWASP Dependency-Check) → Containerization & Registry Push (ECR/Docker Hub) → Deployment & Verification (Dev/QA/Staging/Prod) → Post Actions (Slack/Teams alerts, archive reports).

**Q: How do you handle rollback in a deployment pipeline?**
> A: Automated pipeline rollbacks, GitOps reconciliations, image tag updates, or dedicated rollback scripts.

**Q: What are common pipeline implementation challenges?**
> A: Inter-tool authentication/credential integration, managing runner capacity/timeouts, scan-stage bottlenecks, and pipeline security hardening.

**Q: How do you design pipelines for a microservices architecture?**
> A: Independent, decoupled pipelines per service; immutable container artifacts tagged with Git commit SHAs; progressive rollouts (Canary/Blue-Green).

**Q: How do you harden a CI/CD pipeline (DevSecOps)?**
> A: Avoid embedded credentials via dynamic secret injection (Vault/Secrets Manager); artifact signing; vulnerability scanning (SAST/DAST/Trivy); audit logging.

**Q: Write a Jenkins pipeline with parameters and conditional stages.**
> A:
```groovy
pipeline {
    agent none

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
        booleanParam(name: 'CLEAN', defaultValue: true, description: 'Clean before building')
        choice(name: 'ENVIRONMENT', choices: ['development', 'staging', 'production'], description: 'Choose deployment environment')
    }

    stages {
        stage('Checkout') {
            agent { label 'maven-builder' }
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/example/repo.git'
            }
        }
        stage('Clean') {
            when { expression { return params.CLEAN } }
            steps { deleteDir() }
        }
        stage('Build') {
            steps { echo "Building for: ${params.ENVIRONMENT}" }
        }
        stage('Deploy') {
            steps { echo "Deploying to ${params.ENVIRONMENT}" }
        }
    }

    post {
        always { echo 'Cleaning up after build...' }
    }
}
```

**Q: How do you ensure specific tasks run on specific Jenkins agents?**
> A: Set `agent none` at the pipeline top level — this avoids reserving an executor on the master/controller. Each stage then binds to a specific **Agent Label**:
```groovy
pipeline {
    agent none
    stages {
        stage('Build & Test') {
            agent { label 'maven-builder' }
            steps { sh 'mvn clean package' }
        }
        stage('Security Scan') {
            agent { label 'security-agent' }
            steps { sh 'sonar-scanner' }
        }
    }
}
```

**Q: What pipeline resilience features should every production pipeline have?**
> A: **Timeout** (`options { timeout(time: 30, unit: 'MINUTES') }` to kill stuck jobs), **Triggers** (`githubPush()` webhook or `pollSCM`), **Parallel Execution** (run unit/integration/lint tests simultaneously), **Masked Credentials** (`withCredentials([usernamePassword(...)])`).

**Q: What are common Docker image versioning strategies?**
> A: Git Tags (`v1.4.0`), Commit Hash (`${GIT_COMMIT[0..7]}`), Build Number (`v1.0-${BUILD_NUMBER}`), or a Version File (`version.txt`).

**Q: What's the difference between Continuous Delivery and Continuous Deployment?**
> A: **Continuous Deployment**: every passing build auto-deploys to production, no manual approval. **Continuous Delivery** (enterprise standard): auto-deploys to staging/QA, but production requires a manual **Approval Gate**.

**Q: How do you set up a GitHub → Jenkins webhook, and what's the #1 pitfall?**
> A: Payload URL `http://<jenkins-url>/github-webhook/`, Content-Type `application/json`, Trigger on push event. **Pitfall**: forgetting the trailing slash `/` — it commonly causes HTTP 302/404 errors.

**Q: A production deployment just failed. Walk through your response.**
> A: Stop further deployments → analyze logs (release pipeline, app monitoring, K8s/VM) → rollback via previous stable artifact or slot swap → RCA → add preventive checks: pre-deployment validation, smoke tests, health probes, automated rollback conditions.

**Q: Your build pipeline suddenly becomes very slow. What do you check?**
> A: Identify the slow stage via pipeline analytics → dependency caching, parallel jobs, incremental builds, reduce unnecessary tests, optimize Docker layers, use self-hosted agents for heavy workloads.

**Q: Developers are pushing directly to `main`, causing unstable releases. How do you fix this process-wise?**
> A: Branch policies: mandatory PRs, minimum reviewers, successful build validation, work item linking, restrict direct pushes. Enforce CI validation before merge; consider GitFlow or trunk-based branching depending on team size.

**Q: A secret was accidentally exposed in a pipeline. What are your immediate and follow-up actions?**
> A: Immediate: revoke/rotate the secret, identify exposure scope, audit logs/access. Follow-up: move secrets to Key Vault/Vault, use secret variables (never hardcode), restrict pipeline permissions, enable log masking, review RBAC.

**Q: What are the most common real-world Jenkins failures, and their fixes?**
> A:

| Failure | Root Cause | Fix |
|---|---|---|
| Agent Offline | SSH credential expiry, network drop, Java version mismatch | Renew credentials, check connectivity, align Java versions |
| Hanging builds | Thread deadlocks, exhausted executors, missing `-auto-approve`, no timeout | Add `timeout(...)`, use `cleanWs()` |
| Disk full | Stale Maven deps, old logs, untagged Docker images | `docker system prune -af` cron, workspace discard policy |
| Broken plugins | Jenkins core upgraded without checking compatibility, Docker Hub rate limits | Check compatibility matrix first; use authenticated pulls/mirror |

**Q: How do you build a rollback strategy in a GitOps + Jenkins pipeline (not just "redeploy the old image")?**
> A: Two robust strategies:
> 1. **Parameterized Rollback Pipeline via Registry API**: a separate dedicated pipeline accepts `service_name`, queries the previous stable tag via the registry API (e.g., Docker Hub API), parses it with `jq`, commits the older tag back into the K8s manifest repo → Flux/ArgoCD reconciles and rolls back.
> 2. **Jenkins Archive Artifacts Snapshot**: before patching the manifest with a new image tag, back up the existing manifest via `archiveArtifacts`. On rollback, pull the archived manifest from the last successful build and commit it back to Git — GitOps restores the previous state with no manual edits.

**Q: What's the "Build Once, Promote Everywhere" pattern, and why is it preferred?**
> A: An immutable artifact (Docker image/`.jar`) is built **once** from the commit SHA. The **same image digest** flows through Dev → QA/Staging → Production, never rebuilt — environment differences handled purely via externalized config (ConfigMaps, Secrets, env vars). This is the 12-Factor standard, preferred over rebuilding per environment because it eliminates drift risk.

**Q: Who builds the QA CI/CD pipeline — QA testers or DevOps engineers?**
> A: The **DevOps engineer** designs, provisions, and maintains the pipeline infrastructure/code. The **QA team** contributes automated test scripts (Selenium/Cypress/Playwright) that run *inside* a pipeline stage.

**Q: How do you conditionally run stages for different environments in one parameterized pipeline?**
> A: Use the `when` directive:
```groovy
stage('Deploy to Production') {
    when {
        expression { params.ENVIRONMENT == 'Production' }
    }
    steps {
        input message: 'Approve production deployment?'
        sh 'helm upgrade --install prod-release ./charts/app'
    }
}
```

**Q: What is event-driven architecture in DevOps, and how do you defend it in an interview?**
> A: Workflows triggered automatically by *events* rather than manual steps or fixed schedules. Name the actual mechanism: **CI/CD Triggers** (Git webhook → Jenkins/GitHub Actions run instantly), **Dynamic Autoscaling** (CloudWatch/Prometheus threshold breach → ASG or K8s HPA/Karpenter), **Self-Healing & Rollback** (health-check failure → automated rollback/restart/traffic shift at the LB).

**Q: What are GitHub Actions workflows/jobs/steps/actions?**
> A: **Workflows** live in `.github/workflows/*.yml`, triggered by GitHub events. **Jobs** — one or more per workflow, each on a fresh VM. **Steps** — ordered actions within a job. **Actions** — reusable plugins/scripts.

**Q: GitHub-hosted vs. self-hosted runners — when do you pick each?**
> A: **GitHub-hosted**: standard, ephemeral, no special requirements. **Self-hosted**: private VPC access needed, proprietary build tools, specialized hardware/caching.

**Q: What are GitHub Actions security best practices?**
> A: Store tokens only in GitHub Actions Secrets — never in code. Limit `GITHUB_TOKEN` permissions via explicit `permissions:` blocks (least privilege). Pin action versions to **specific commit SHAs**, not mutable tags.

**Q: What are GitHub Actions "contexts," and why is direct interpolation into bash risky?**
> A: Contexts are built-in metadata objects (`github`, `secrets`, `env`, `runner`, `job`/`steps`), accessed via `${{ context.property }}`. Interpolating untrusted context values (like an issue title) directly into `run:` is a script-injection risk:
```yaml
# ❌ Risky
run: echo "${{ github.event.issue.title }}"
# ✅ Safe
env:
  TITLE: ${{ github.event.issue.title }}
run: echo "$TITLE"
```

---

## 9. GitOps — Helm, ArgoCD & Flux

**Q: What is Helm, and how did Helm 3 change the security model?**
> A: The package manager for Kubernetes (like `apt` for Ubuntu). Helm 2 required an in-cluster **Tiller** component (a security risk); **Helm 3 removed Tiller entirely**, using the user's kubeconfig and RBAC directly.

**Q: `helm install` vs. `helm upgrade` — what's the safe pattern?**
> A: `helm install` **fails** if the release already exists. `helm upgrade` updates an existing release. Best practice: `helm upgrade --install <release> <chart>` — installs if new, upgrades if it exists.
```bash
helm upgrade --install my-app ./my-app \
  --namespace production \
  --set image.tag=${GIT_COMMIT[0..7]} \
  --wait
```

**Q: What's the purpose of `values.yaml` in a Helm chart, and how do you override it?**
> A: Defines default config variables (image tags, replica counts, resource limits, ports) parameterizing the templates. Override via a custom YAML file (`-f values-prod.yaml`, recommended for GitOps/multi-env) or CLI flags (`--set replicaCount=3`).

**Q: What is a Helm Release?**
> A: A specific running instance of a chart deployed to a cluster — the same chart can be deployed multiple times under different release names (`app-dev`, `app-staging`).

**Q: What is GitOps, in one sentence?**
> A: Git is the **single source of truth** for both application code and Kubernetes infrastructure state; an operator continuously reconciles the live cluster to match it.

**Q: What's the ArgoCD reconciliation loop, and what does "OutOfSync" mean?**
> A: ArgoCD continuously compares the **Desired State** (Git manifests/Helm/Kustomize) with the **Live State** (running cluster). A mismatch = `OutOfSync`. **Automated Sync** auto-applies changes to restore parity; **Manual Sync** flags drift and waits for operator approval.

**Q: How is CI/CD labor split between traditional CI tools and ArgoCD?**
> A: CI tools (Jenkins/GitHub Actions) build/test/scan and update the image tag in Git. **ArgoCD handles CD** — pulling changes into the cluster, without ever exposing cluster credentials to the CI server. This is the core benefit of pull-based deployment.

**Q: Describe ArgoCD's architecture components.**
> A: **API Server** (handles UI/CLI/CI requests, auth, RBAC), **Repository Server** (clones Git, renders manifests), **Application Controller** (core reconciliation loop — compares & syncs live vs. desired), **Redis Cache** (speeds up comparisons/sessions), **Web UI/CLI** (visualize topologies, sync status, approvals).

**Q: How do you roll back an ArgoCD-managed deployment?**
> A: **Git-native (preferred)**: `git revert <commit-hash>` — ArgoCD auto-detects and rolls back. **UI/CLI**: `argocd app rollback <app-name>`. **Self-healing/auto-pruning** also reverts manual `kubectl edit` drift automatically.

**Q: What are ArgoCD ApplicationSets used for?**
> A: Dynamically generating many `Application` resources from one template — e.g., deploying the same app across 50 regional clusters without repetitive YAML.

**Q: ArgoCD vs. Flux CD — how do you choose?**
> A: **ArgoCD**: centralized server, rich Web UI, SSO, granular RBAC — best for complex enterprise/multi-tenant setups. **Flux CD**: decentralized, lightweight Kubernetes controllers, CLI-centric — best for minimalist, headless environments.

**Q: DevOps vs. GitOps — what's the actual difference?**
> A:

| Aspect | DevOps | GitOps |
|--------|--------|--------|
| Scope | Broad org culture, full lifecycle | Deployment automation & CD specifically |
| Center of Gravity | Tool-agnostic | Git-centric, single source of truth |
| Configuration | Declarative OR imperative | Strictly declarative |
| Reconciliation | Often push-based | Pull-based (in-cluster operator) |

> Simple way to remember it: DevOps = the whole philosophy/culture. GitOps = a specific *implementation* of the "CD" part of DevOps, using Git as the control mechanism.

---

## 10. Monitoring, Logging & SRE

**Q: Metrics vs. Logs — how do you explain the difference?**
> A: **Metrics** = time-series numeric data (CPU %, memory, pod restarts) — best for trends/thresholds/dashboards. **Logs** = discrete, timestamped text events (API request, stack trace) — best for root-cause/forensic debugging.

**Q: How do you set up Prometheus + Grafana on Kubernetes?**
> A:
```bash
kubectl create ns monitoring
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```
`kube-prometheus-stack` bundles the Prometheus Operator, node-exporter, and kube-state-metrics. Connect Grafana → Prometheus via Data Sources → `http://prometheus-server.monitoring.svc.cluster.local:9090`.

**Q: How does the EFK logging stack work?**
> A: **Fluentd/Fluent Bit** runs as a **DaemonSet** on every node — tails `stdout`/`stderr` from `/var/log/containers/*.log`, enriches with pod/namespace metadata, ships to **Elasticsearch** (indexes/stores logs). **Kibana** is the UI for search/filter/visualize (Lucene/KQL, trace errors, inspect stack traces).

**Q: How does Prometheus alerting flow to Slack?**
> A:
```
[ Target Pods/Nodes ] → (exposes /metrics) → [ Prometheus Server ] (scrapes, stores in TSDB)
   → [ Grafana ] → Dashboards
   → [ Alertmanager ] → Slack / PagerDuty / Email
```
Example rule:
```yaml
groups:
  - name: example-alerts
    rules:
      - alert: HighCPUUsage
        expr: sum(rate(container_cpu_usage_seconds_total{image!="", container!="POD"}[1m])) by (container) > 0.8
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container CPU usage is too high"
```

**Q: SRE vs. DevOps — how does SRE make DevOps concrete?**
> A: SRE implements DevOps through engineering metrics: **SLIs** (what you measure), **SLOs** (the target for that measurement), and **Error Budgets** (how much unreliability is acceptable before you must slow feature work and focus on reliability).

**Q: It's midnight and production is down. What's your response order?**
> A: **1) Fast mitigation over deep analysis**: capture diagnostic snapshots, then mitigate fast (rollback latest release, failover to backup region/cluster, restart degraded pods). **2) Incident communication**: war-room/channel, notify on-call/stakeholders, regular status updates. **3) Post-recovery RCA**: document exact timeline, correlate telemetry spikes, preserve logs.

**Q: How do you reduce MTTR (Mean Time to Recovery)?**
> A: Granular observability (high-resolution metrics at short scrape intervals), automated runbooks/self-healing scripts, and smaller/incremental deployments (small batch releases + canary rollouts → smaller rollback blast radius).

**Q: How do you troubleshoot high latency in a distributed system?**
> A: Layer-by-layer diagnostics: Network/DNS → Ingress/LB → Application logic → Downstream DBs/caches → External APIs. Use distributed tracing (Jaeger, Zipkin, OpenTelemetry, AWS X-Ray) to pinpoint the exact slow span.

**Q: How do you debug intermittent microservice failures?**
> A: Inspect APM/logs for error spikes, retry storms, socket exhaustion, timeouts. Apply resilience patterns: Circuit Breakers (Resilience4j, Envoy/Istio), Exponential Backoff with Jitter. Reproduce in staging by elevating debug logs and simulating peak traffic.

**Q: How do you design a system for High Availability?**
> A: Multi-AZ/Multi-Region with active-active or active-passive failover; stateless service design (decouple compute from storage); strict timeouts/deadlines to prevent thread pool exhaustion; data layer protection (async read replicas, multi-region replication, automated snapshots).

**Q: What are the "Three Pillars of Observability" and how do you avoid alert fatigue?**
> A: **Metrics** (trends), **Logs** (discrete events), **Traces** (request flow across boundaries). Avoid alert fatigue by deprecating noisy non-actionable alerts and alerting on **user-facing SLO symptoms** (elevated error rate, latency breach) rather than raw CPU spikes that self-resolve.

**Q: What makes a good blameless post-mortem/RCA?**
> A: Link metric timestamps to trigger events (config changes, DB connection pool saturation), identify systemic contributing factors, and track preventative action items — without assigning individual blame.

---

## 11. Code Quality & DevSecOps

**Q: Code Smell vs. Bug vs. Vulnerability — how do you distinguish them?**
> A:

| Category | Impact | Definition | Effect |
|---|---|---|---|
| Code Smell | Maintainability | Sub-optimal design/practices | Works correctly but is messy/fragile/hard to refactor |
| Bug | Reliability | Flaws, runtime exceptions, logic errors | Crashes, exceptions, wrong output |
| Vulnerability | Security | Exploitable security flaws | Runs fine, but data/access exposed |

> Memory hook: 🧹 Code Smell = "it works, but it's ugly and risky." 🐞 Bug = "it's broken." 🔓 Vulnerability = "it's a door left open for attackers."

**Q: Give real-world examples of code smells and their fixes.**
> A: **Hardcoded values/endpoints** → externalize into env vars/config files. **Deeply nested if/else (arrow anti-pattern)** → guard clauses/early returns/polymorphism. **Bloated functions** (violates SRP) → break into small helpers. **Duplicated code** → extract into shared methods/libraries.

**Q: Give real-world examples of bugs.**
> A: **Off-by-one/index out of bounds** (array size 4 accessed at index 4 → `ArrayIndexOutOfBoundsException`), **divide-by-zero** without checking the denominator, **uninitialized/scope issues** (variable declared in inner block, referenced outside).

**Q: Give real-world examples of vulnerabilities and remediations.**
> A: **Hardcoded credentials** → inject at runtime via Vault/Secrets Manager/Key Vault. **SQL Injection** (string-concatenated queries) → parameterized queries/PreparedStatements + ORM sanitization. **Broken authentication** → enforce OAuth2/JWT verification, strict RBAC, origin whitelisting.

**Q: How do you enforce code quality across teams using SonarQube?**
> A: *"Every Pull Request triggers a Maven build coupled with a SonarQube scan. We configure strict Quality Gates: any PR introducing Blocker/Critical Bugs, Security Vulnerabilities, or dropping Code Coverage below 80% automatically fails the Quality Gate — blocking artifact generation or promotion to staging until resolved."*

**Q: What are the four main scan types in a CI/CD pipeline, and what does each cover?**
> A: **SAST** (static — scans source code before build), **SCA** (scans dependencies/third-party libraries — e.g., Snyk), **Container Image Scanning** (scans layers/OS packages for CVEs before push — Trivy, Aqua), **DAST** (dynamic — tests the running application/endpoints).

---

## 12. RBAC & Security

**Q: Azure RBAC vs. AWS IAM vs. Kubernetes RBAC — what layer does each secure?**
> A: **Azure RBAC**: Azure cloud resource access (subscriptions, RGs, VMs, AKS **infrastructure**). **AWS IAM**: identities/permissions for AWS resources (EC2, S3, EKS). **Kubernetes RBAC**: permissions **inside** the cluster (namespaces, pods, deployments, secrets). Memory hook: Cloud RBAC secures the *infrastructure* layer; K8s RBAC secures the *workload* layer — you need **both**.

**Q: Why do you need both Cloud-level RBAC and Kubernetes RBAC?**
> A: They secure different layers — Cloud RBAC controls access to AKS/EKS infrastructure, VMs, networking, storage; K8s RBAC controls what users can do *inside* the cluster. Together they provide layered security, least privilege, namespace isolation, and better governance.

**Q: How do you ensure least-privilege access across a cloud + K8s environment?**
> A: RBAC across Azure/AWS/K8s with minimum permissions per role/namespace; avoid cluster-admin; use Just-In-Time (JIT) elevated access; groups instead of direct grants; regular access reviews; immediate revocation on offboarding/team changes.

**Q: How do you do user audit and logging across multi-cloud + Kubernetes?**
> A: Centralized logging — Azure Activity Logs/Monitor/Sentinel, AWS CloudTrail/CloudWatch/GuardDuty, Kubernetes API server audit logs — all feeding a central SIEM (Sentinel, Splunk, ELK) for monitoring, alerting, compliance, and forensic analysis.

**Q: A developer moves from one team to another. How do you ensure access is correctly transitioned?**
> A: Centralized identity/access management (Entra ID / AWS IAM Identity Center) with **group-based** RBAC — access is never assigned directly to users. Remove from the old group, add to the new team group → automatically updates Azure RBAC, AWS IAM roles, and Kubernetes RoleBindings. Validate via audit logging and periodic access reviews.

**Q: How do you ensure cluster/control-plane isolation between environments?**
> A: Separate Dev/QA/Production into different Kubernetes clusters; use managed services (AKS/EKS) to secure the control plane; implement namespace isolation, RBAC, NetworkPolicies, node isolation, and Pod Security Standards; restrict API server access via private networking/VPN; enable audit logging.

**Q: Write a Kubernetes Role limiting a developer to read-only access on pods/services in one namespace.**
> A:
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: payments-dev
  name: developer-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
```

---

## 13. OpenShift

**Q: What is OpenShift, and how does it relate to Kubernetes?**
> A: Red Hat OpenShift is an enterprise container platform built **on top of** Kubernetes. Kubernetes is the core orchestration engine; OpenShift adds developer tools, built-in CI/CD, enhanced security, monitoring, an image registry, a web console, and an Operator framework. Memory hook: OpenShift = Kubernetes + enterprise features + automation.

**Q: What's the OpenShift equivalent of a namespace, and how do you create one?**
> A: A **Project**. `oc new-project demo-project`.

**Q: What is CRC (OpenShift Local)?**
> A: Lets developers run OpenShift locally on a laptop.

**Q: What does OpenShift's architecture look like?**
> A: Control plane nodes (API server, scheduler, etcd), worker nodes (run workloads), and an Ingress router (handles external traffic).

**Q: What's the OpenShift CLI, and give a few essential commands.**
> A: `oc` — the OpenShift equivalent of `kubectl`, with extra OpenShift-specific commands.
```bash
oc login
oc get pods
oc get svc
oc describe pod
oc logs
oc exec
```

---

## 14. Migration & DR Scenarios

**Q: You need to migrate a critical PostgreSQL StatefulSet from EBS gp2 to gp3 with a max 2-minute downtime window. Walk through your plan.**
> A: 1) **Pre-migration safety**: snapshot the volume via AWS Backup/EBS Snapshots; pre-flight checks (row counts, schema checksums, disk metrics). 2) **Data copy & delta sync**: provision a new gp3 volume via a new StorageClass/PVC; run asynchronous replication while the DB stays online, minimizing the final delta. 3) **2-minute cutover**: stop write traffic (or read-only mode) → shut down the Postgres pod → run the final delta sync → update the PVC/StatefulSet volume binding to gp3 → restart the pod. 4) **Validation**: automated health queries (row counts, read/write tests), monitor CloudWatch EBS IOPS/queue metrics. 5) **Rollback**: don't delete the old gp2 volume immediately — if checks fail, re-point back to gp2 and restart.

**Q: You need to upgrade a 200+ microservice AKS cluster from an end-of-life version with zero customer downtime. What's your approach?**
> A: Assess API deprecations (review release notes, audit Helm charts with `pluto`/`kubent`, validate CSI drivers/CNI/AGIC compatibility) → validate in dev/staging first → **Blue-Green cluster rollout**: spin up a new "green" cluster on the target version via Terraform/Bicep, deploy identical manifests, shift traffic progressively at App Gateway/Front Door (5%→25%→100%) → synchronize StatefulSet disk snapshots/DB replicas before cutover → if 5xx errors occur, switch routing back to blue immediately.

**Q: You're migrating 400 on-prem VMs to AWS within 6 months with mixed criticality workloads. What's your migration strategy?**
> A: Prepare the landing zone first (multi-account AWS Organizations, VPCs, Direct Connect, extended Active Directory) → discovery & dependency mapping (app-to-DB connections) → phased migration waves (Dev → UAT → Production) using **AWS MGN** for compute, **AWS DMS + SCT** for databases, **AWS DataSync** for file storage → validate in an isolated subnet, then cutover via lowered DNS TTL and Route 53 updates → keep on-prem warm/standby for a soak period before decommissioning.

**Q: How would you migrate a 5-year-old AWS production platform to Azure over 12 months with zero downtime?**
> A: Map every AWS service to its Azure equivalent (EKS→AKS, S3→Blob Storage, RDS→Azure DB, Route53→Traffic Manager, CloudWatch→Monitor, Secrets Manager→Key Vault, AWS Terraform→AzureRM/Bicep) → provision the Azure landing zone → deploy workloads to AKS with continuous DB/blob replication → phased traffic shift (5%→25%→100% via weighted DNS) → keep AWS passive until Azure stability is confirmed.

**Q: A 25TB MySQL database doing 8,000 writes/sec must migrate to Aurora with a 5-minute window and zero data loss. What's your plan?**
> A: Use **AWS DMS** for schema conversion + continuous CDC replication while the source stays live. During the 5-minute window: pause/queue writes → apply final CDC delta → switch connection strings to Aurora → resume traffic. Validate row counts/checksums and zero replication lag before cutover. If Aurora underperforms post-cutover, revert connection strings to the still-warm MySQL-on-EC2 endpoint.

**Q: Your EKS control plane was accidentally deleted, affecting 150+ apps. RTO < 30 min, RPO < 5 min. How do you recover?**
> A: 1) **Rebuild fast** via `terraform apply` against version-controlled IaC (if IaC is missing, 30-min RTO is practically impossible). 2) **Bootstrap addons**: VPC CNI, CoreDNS, kube-proxy, EBS CSI driver, then install Velero pointed at the backup bucket. 3) **Velero restore**: `velero restore create --from-backup prod-cluster-backup-latest --wait` — reconstitutes manifests, reattaches EBS volumes from snapshots to PVCs. 4) **DB & DNS cutover**: point to the warm cross-region DB replica, restore the Load Balancer Controller, update Route 53. 5) **Validate**: zero CrashLoopBackOff, synthetic transactions pass, DB write integrity confirmed, monitoring stack emitting metrics again.

**Q: A financial app has 30 minutes of downtime per release. Redesign for zero downtime with instant rollback.**
> A: Implement Blue-Green: Blue (active) serves 100% traffic; Green (idle) gets the new version and is smoke-tested without affecting users; cutover switches ALB target group weights or a Route 53 DNS update; Blue stays on standby for instant rollback. Next release cycle, roles alternate — Green becomes active, Blue becomes the new staging target. Implementation: either dual in-cluster Deployments switching Service selectors (cost-effective) or dual identical clusters switching at Route 53 (high isolation, critical banking).

**Q: Design a multi-region DR architecture for an app running only out of AWS Mumbai.**
> A: Replicate infra to a secondary region via parameterized Terraform/CloudFormation. Use Route 53 Failover Routing + Health Checks for DNS failover; CloudFront origin failover groups for edge caching. RDS/Aurora async Cross-Region Read Replica (promote on failover); S3 Cross-Region Replication with versioning; Redis runs **independent** caches per region (avoid WAN sync latency). Use stateless JWTs or distributed sessions to avoid forcing re-login on failover. CI/CD: Active-Active (deploy to both simultaneously) or Active-Passive (secondary pre-provisioned, spun up on incident).

**Q: You need to modernize 300+ legacy .NET Framework apps on Windows VMs into AKS. What's your approach?**
> A: Assess and separate stateless APIs from SMB/SQL-bound services; migrate low-risk services first. Containerize using a Windows kernel base image (`FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2022`), push to ACR. Use hybrid AKS node pools (Linux for system services, Windows Server for the .NET workloads), Azure Files CSI driver for SMB compatibility. Externalize secrets to Key Vault via Secrets Store CSI Driver; secure DB connectivity via Private Endpoints. Keep legacy VMs running in parallel; decommission only after full production burn-in.

**Q: What are the "6 Rs" of cloud migration strategy?**
> A: **Rehost** (lift & shift, no redesign), **Replatform** (lift, tinker & shift — minor optimizations using managed services), **Refactor/Re-architect** (re-engineer into cloud-native), **Repurchase** (replace with SaaS), **Retain** (keep on-prem for compliance), **Retire** (decommission obsolete systems).

**Q: You need to migrate ~700 Jenkins pipelines to Azure DevOps without interrupting releases. How do you approach it?**
> A: Migrate pipelines, credentials, agents, artifacts, deployment strategies, approvals, secrets, and rollback mechanisms incrementally. Run **both CI/CD systems in parallel** before decommissioning Jenkins, validating each migrated pipeline against its Jenkins counterpart before cutting teams over.

---

## 15. DevOps Culture & Career

**Q: What is the DevOps Maturity Model measuring?**
> A: Organizational transformation across automation, cross-team collaboration, continuous delivery, and observability. Higher maturity = faster release cadence with minimal manual intervention.

**Q: What are the 12-Factor App principles most relevant to a DevOps engineer?**
> A: Strict config/code separation, stateless processes, backing service abstraction, disposability (fast startup/graceful shutdown), and dev/prod parity.

**Q: SRE vs. DevOps — what's the relationship?**
> A: SRE **implements** DevOps through concrete engineering metrics — SLIs (what you measure), SLOs (the target), Error Budgets (allowed unreliability before reliability work takes priority over features).

**Q: What's the biggest interview mistake candidates make when reciting technical concepts?**
> A: Reading a cheat sheet's one-liners without elaborating on real-world mechanics and architecture patterns. Interviewers want you to narrate *why* something fails, *how* you'd verify it, and *how* you'd fix/prevent it — like a real incident, not a flashcard.

**Q: Why do enterprises run domain-specific interview rounds (Banking, Healthcare, Insurance)?**
> A: They test regulatory compliance, strict SLAs, and data handling constraints beyond general tool proficiency — e.g., PCI-DSS/SOX for banking, HIPAA/GDPR for healthcare, tiered data retention, zero-data-loss DR drills.

**Q: You're switching into DevOps from another role. What are your two practical routes?**
> A: **Internal transition**: ask for partial allocation to an internal DevOps/Cloud project — even 2–4 months of cross-skilling gives defensible production context. **Self-learning/external switch**: hands-on labs (KodeKloud), AWS/Azure free tier, self-hosted tools (EC2, Jenkins, Docker, Minikube/kind/EKS) to build end-to-end projects.

**Q: What are the 3 core questions career-switchers must nail?**
> A: 1) *"What are your exact roles and responsibilities?"* — anchor on tools you're genuinely solid in. 2) *"What does a typical day look like?"* — structure it: standup → alerts → root-cause → automation. 3) *"What tools/cloud tech do you own?"* — be specific, don't generalize.

**Q: Describe a realistic "day in the life" of a DevOps engineer.**
> A: `09:00–09:30` system checks/alerts/Jira → `09:30–10:00` standup → `10:00–13:00` P0/P1 priority work (pipeline debugging, hotfixes) → `14:00–17:00` core project execution (Terraform modules, CI/CD refactoring) → `17:00–18:00` cross-team syncs, documentation, runbook updates.

**Q: What's the "Automate Recurring Issues" principle?**
> A: If an issue happens once, document the fix. If it happens twice, **automate the remediation** (automated pod restarts, disk cleanup jobs, log archiving).

**Q: How should you handle new recruiter outreach while already holding an offer?**
> A: Be upfront about the existing offer and your compensation/project expectations immediately — this respects everyone's time and avoids wasted interview cycles if they can't match your benchmark.

---

## 📌 Final Rapid-Fire Recall Round

**Q: `git fetch` vs. `git pull`?**
> A: `git fetch` downloads new commits/branches without touching your working directory. `git pull` = `git fetch` + `git merge`.

**Q: Can the Ansible control node run on Windows?**
> A: No — needs WSL. But it *can* manage Windows targets via WinRM.

**Q: What's the CI/CD scan order?**
> A: SAST (source code) → SCA (dependencies) → Container scan (image CVEs) → DAST (running app).

**Q: What happens to running pods if etcd crashes?**
> A: They keep running (kubelet/runtime continue) — but no new scheduling/scaling/secrets; the control plane freezes.

**Q: What's the zero-downtime K8s deploy config?**
> A: `maxUnavailable: 0`, `maxSurge: 1` + accurate readiness probes — old pods only die after new ones are ready.

**Q: 502 vs. 503 vs. 504?**
> A: 502 = bad response from upstream. 503 = service overloaded/down. 504 = upstream timed out.

**Q: How do you recover a lost Terraform state file?**
> A: Remote backend versioning (S3/Blob) OR `terraform import` to rebuild.

**Q: What does `helm upgrade --install` do that plain `helm install` doesn't?**
> A: It creates the release if it doesn't exist, or upgrades it if it does — avoiding the "already exists" failure of plain `install`.
