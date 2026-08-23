# 🚨 AWS Troubleshooting & Production Incident Triage Master Handbook
> **The SRE, Cloud Engineer & Solutions Architect Guide to Diagnosing, Mitigating, and Surviving Production Outages on AWS.**  
> *Complete Production Triage Curriculum: Incident Severity (SEV 1–4), Observability Triangle, IAM Access Denied Trees, EC2/EBS/EFS Failures, VPC/NACL Network Triage, ALB 502/504 Errors, RDS/DynamoDB Starvation, Lambda OOM (137), EKS Debugging, IaC Lock Recovery, Security Forensics & Blameless 5-Whys Postmortems.*

---

## 📑 Table of Contents
1. [Executive Incident Severity Matrix & Production Triage Lifecycle](#1-executive-incident-severity-matrix--production-triage-lifecycle)
2. [The Observability Golden Triangle (CloudWatch, CloudTrail & AWS Config)](#2-the-observability-golden-triangle)
3. [IAM & Security "Access Denied" Root Causes & Decision Tree](#3-iam--security-access-denied-root-causes--decision-tree)
4. [Compute & Storage Troubleshooting (EC2 Status Checks, EBS & EFS)](#4-compute--storage-troubleshooting)
5. [VPC, Routing & Hybrid Network Debugging (SGs, NACLs & Reachability)](#5-vpc-routing--hybrid-network-debugging)
6. [DNS, Edge & Load Balancer Triage (Route 53, ALB 502 vs. 504)](#6-dns-edge--load-balancer-triage)
7. [Database Failure Modes (RDS/Aurora Starvation & DynamoDB Hot Partitions)](#7-database-failure-modes)
8. [Serverless & Asynchronous Integration Debugging (Lambda & SQS DLQs)](#8-serverless--asynchronous-integration-debugging)
9. [Containers & Kubernetes Diagnostics (ECS & Amazon EKS Outages)](#9-containers--kubernetes-diagnostics)
10. [Infrastructure as Code Debugging (CloudFormation & Terraform State Locks)](#10-infrastructure-as-code-debugging)
11. [Edge & Security Perimeter Triage (CloudFront 502 & WAF False Positives)](#11-edge--security-perimeter-triage)
12. [Performance Optimization & Latency Bottleneck Isolation Matrix](#12-performance-optimization--latency-bottleneck-isolation-matrix)
13. [AWS Cloud Security Incident Response & Forensics Playbook](#13-aws-cloud-security-incident-response--forensics-playbook)
14. [Disaster Recovery, Blameless RCA & 5 Whys Postmortem Templates](#14-disaster-recovery-blameless-rca--5-whys-postmortem-templates)
15. [Senior SRE & Cloud Architect Interview Playbooks (Top Scenarios)](#15-senior-sre--cloud-architect-interview-playbooks)
16. [Production Readiness & Outage Prevention Checklist](#16-production-readiness--outage-prevention-checklist)

---

## 1. Executive Incident Severity Matrix & Production Triage Lifecycle

```
+-------------------------------------------------------------------------------+
|                            INCIDENT SEVERITY MATRIX                           |
+----------+----------------------+--------------------------+------------------+
| Severity | Impact Level         | Criteria / Example       | Response Target  |
+----------+----------------------+--------------------------+------------------+
| SEV 1    | Critical Outage      | Core business down, 100% | < 15 minutes     |
|          |                      | users affected, no bypass|                  |
| SEV 2    | Major Degradation    | Partial outage, degraded | < 30 minutes     |
|          |                      | performance, key feature |                  |
| SEV 3    | Minor Impact         | Non-critical feature bug,| < 2 hours        |
|          |                      | workaround available     |                  |
| SEV 4    | Minimal / Cosmetic   | Minor UI glitch, typo,   | < 1 business day |
|          |                      | low priority backlog     |                  |
+----------+----------------------+--------------------------+------------------+
```

### The SRE Incident Response Lifecycle
$$\text{Detect} \longrightarrow \text{Triage} \longrightarrow \text{Isolate} \longrightarrow \text{Mitigate} \longrightarrow \text{Resolve} \longrightarrow \text{Validate} \longrightarrow \text{Postmortem (5 Whys)}$$

---

## 2. The Observability Golden Triangle

```
          +----------------------------------------------------+
          |                    USER ACTION                     |
          +-------------------------+--------------------------+
                                    |
                                    v
+------------------------------------+-----------------------------------+
|                                    |                                   |
v                                    v                                   v
[ Amazon CloudTrail ]          [ Amazon CloudWatch ]                 [ AWS Config ]
"WHO did WHAT and WHEN"       "WHAT is happening NOW"            "WHAT CHANGED in state"

API calls & user identity    - Numerical metrics (CPU, 5xx)     - Resource configuration history
Source IP & UserAgent        - Structured logs (Logs Insights)  - Compliance rule evaluation
Management & Data events     - Real-time alarms & EventBridge   - Historical config diffs
```

### CloudWatch Logs Insights Query Recipes
```sql
-- Recipe 1: Finding Top 5xx Errors Across API Gateway / ALB
fields @timestamp, @message, status, path
| filter status >= 500
| stats count() by path, status
| sort count() desc
| limit 20

-- Recipe 2: Finding Slowest Lambda Executions & Out of Memory Errors
fields @timestamp, @message, @duration, @maxMemoryUsed
| filter @message like /REPORT/ or @message like /Task timed out/
| sort @duration desc
| limit 25
```

---

## 3. IAM & Security "Access Denied" Root Causes & Decision Tree

```
                                  [ Incoming Request ]
                                           |
                                           v
                             +---------------------------+
                             | Explicit Deny in Policy?  | --(Yes)--> [ ACCESS DENIED ]
                             +-------------+-------------+
                                           | (No)
                                           v
                             +---------------------------+
                             |  SCP Evaluates to Allow?  | --(No)---> [ ACCESS DENIED ]
                             +-------------+-------------+
                                           | (Yes)
                                           v
                             +---------------------------+
                             | Identity Policy Allows?   | --(No)---> [ ACCESS DENIED ]
                             +-------------+-------------+
                                           | (Yes)
                                           v
                             +---------------------------+
                             | Within Perms Boundary?    | --(No)---> [ ACCESS DENIED ]
                             +-------------+-------------+
                                           | (Yes)
                                           v
                             +---------------------------+
                             | KMS Key Policy Allows?    | --(No)---> [ ACCESS DENIED ]
                             +-------------+-------------+
                                           | (Yes)
                                           v
                                   [ ACCESS GRANTED ]
```

### Primary Causes of IAM Permission Failures
1. **Explicit Deny Statement:** Overrides all Allow permissions across all identity/resource policies.
2. **Missing KMS Key Delegation:** Caller has `s3:GetObject` or `secretsmanager:GetSecretValue` but lacks `kms:Decrypt` on the CMK.
3. **Organizations SCP Boundary:** Parent OU blocks the AWS service or region.
4. **Permissions Boundary Ceiling:** The requested API action exceeds the boundary attached to the identity.
5. **Cross-Account Trust Missing:** Calling entity is not declared in the target role's `sts:AssumeRole` trust policy.

---

## 4. Compute & Storage Troubleshooting

### 1. EC2 Status Check Failures & SSH Timeouts
* **System Status Check Failed (0/2 Passed):** Underlying AWS hypervisor or physical server issue.
  * *Immediate Fix:* Stop and Start the EC2 instance (forces migration to a healthy physical host).
* **Instance Status Check Failed (1/2 Passed):** OS kernel crash, memory exhaustion, corrupted `/etc/fstab`, or misconfigured firewall.
  * *Immediate Fix:* Inspect console logs (`aws ec2 get-console-output`), detach root EBS volume to a rescue instance, or connect via SSM Session Manager.
* **SSH Connection Timed Out (Port 22):**
  * Verify Security Group allows inbound TCP 22 from caller IP.
  * Verify Route Table has `0.0.0.0/0 -> IGW` (Public Subnet).
  * Ensure private key has correct permissions: `chmod 400 key.pem`.

### 2. EBS & EFS Storage Bottlenecks
* **EBS Volume IOPS Saturation:** High `VolumeQueueLength` and read/write latency spikes.
  * *Fix:* Upgrade from `gp2` to `gp3` and provision independent IOPS and throughput, or migrate to `io2 Block Express`.
* **EFS Burst Credit Exhaustion:** `BurstCreditBalance` drops to zero $\rightarrow$ throughput throttles down to baseline (50 KiB/s per GiB).
  * *Fix:* Switch from Bursting Throughput mode to **Elastic Throughput** mode.

---

## 5. VPC, Routing & Hybrid Network Debugging

```
  [ Client / EC2 ]
         |
         v
  [ Security Group ] ---------> Stateful firewall (Return traffic allowed automatically).
         | (Allowed)
         v
  [ Network ACL (NACL) ] -----> Stateless subnet filter (Must allow Inbound AND Outbound Ephemeral: 1024-65535).
         | (Allowed)
         v
  [ Route Table ] ------------> Target destination must exist (0.0.0.0/0 -> IGW, NAT GW, or TGW).
         |
         +------------------------------------+------------------------------------+
         |                                    |                                    |
         v                                    v                                    v
  [ Internet Gateway ]                 [ NAT Gateway ]                  [ Transit Gateway ]
  Public Subnet Ingress/Egress         Private Subnet Internet Egress   Multi-VPC / On-Prem Hub
```

### Network Debugging Swiss Army Knife
* **VPC Reachability Analyzer:** Simulates packet paths between two endpoints without sending real packets, identifying blocking Security Groups, NACLs, or Route Tables in seconds.
* **VPC Flow Logs:** Filter for `REJECT` actions to pinpoint whether NACLs or Security Groups dropped the connection.

---

## 6. DNS, Edge & Load Balancer Triage

```
 ┌──────────────┬───────────────────────────────┬─────────────────────────────────────────────────┐
 │ Error Code   │ Diagnostic Meaning            │ Primary Root Cause & Fix                        │
 ├──────────────┼───────────────────────────────┼─────────────────────────────────────────────────┤
 │ **ALB 502**  │ Bad Gateway                   │ Target app crashed, closed TCP socket, or sent  │
 │              │                               │ malformed HTTP headers; check app server logs.  │
 ├──────────────┼───────────────────────────────┼─────────────────────────────────────────────────┤
 │ **ALB 504**  │ Gateway Timeout               │ Target took longer than ALB idle timeout        │
 │              │                               │ (database lock, thread lock, downstream delay). │
 ├──────────────┼───────────────────────────────┼─────────────────────────────────────────────────┤
 │ **Unhealthy**│ Target Group Health Failing   │ `/healthz` returning 404/500; app bound to      │
 │              │                               │ `127.0.0.1` instead of `0.0.0.0`; SG blocking.  │
 ├──────────────┼───────────────────────────────┼─────────────────────────────────────────────────┤
 │ **NXDOMAIN** │ Route 53 Resolution Failure   │ Registrar NS records do not match the assigned  │
 │              │                               │ Route 53 Hosted Zone name servers.              │
 └──────────────┴───────────────────────────────┴─────────────────────────────────────────────────┘
```

---

## 7. Database Failure Modes

### 1. RDS & Aurora High CPU & Connection Starvation
* **Root Causes:** Unindexed slow queries, missing connection poolers, connection storms from Lambda scaling.
* **Remediation:**
  * Enable **Amazon RDS Performance Insights** to identify SQL queries with the highest Average Active Sessions (AAS).
  * Deploy **Amazon RDS Proxy** to pool database connections and absorb serverless concurrency spikes.
  * Provision Read Replicas to offload read-heavy `SELECT` traffic.

### 2. DynamoDB Hot Partitions & Throttling
* **Symptom:** `ProvisionedThroughputExceededException` (HTTP 400).
* **Hot Partition Trap:** Uneven partition key distribution (e.g. partition key based on date/status) directs all reads/writes to a single physical partition.
* **Remediation:**
  * Switch to **DynamoDB On-Demand Capacity** mode for spiky workloads.
  * Implement write sharding by appending a random suffix (e.g. `order_id_1`, `order_id_2`).
  * Deploy **DynamoDB Accelerator (DAX)** for read-intensive microsecond caching.

---

## 8. Serverless & Asynchronous Integration Debugging

```
+------------------------------------------------------------------------------------+
|                         AWS LAMBDA FAILURE INVESTIGATION                           |
+---------------------+-------------------------------+------------------------------+
| Issue               | Root Cause                    | Fix / Optimization           |
+---------------------+-------------------------------+------------------------------+
| Cold Starts         | Large package, VPC ENI init,  | Provisioned Concurrency,     |
|                     | runtime boot delay            | reduce bundle size, ARM64    |
| Timeouts            | Downstream DB/API slow,       | Increase timeout, exponential|
|                     | infinite loop                 | backoff, tune queries        |
| OOM (Exit Code 137) | Processing large payload in   | Increase memory (scales CPU),|
|                     | memory, memory leak           | stream via /tmp or S3        |
| Throttling (429)    | Account concurrency limit hit | Reserved Concurrency, SQS/   |
|                     | (Default 1,000)               | EventBridge buffer           |
+---------------------+-------------------------------+------------------------------+
```

### SQS Message Duplication & Poison Pills
* **Message Duplication:** Processing time exceeds `VisibilityTimeout` $\rightarrow$ SQS redelivers the message to another worker.
  * *Fix:* Increase `VisibilityTimeout` or invoke `ChangeMessageVisibility` dynamically during long jobs.
* **Poison Pill Messages:** Malformed message crashes consumer continuously in an infinite loop.
  * *Fix:* Configure a **Dead-Letter Queue (DLQ)** with `maxReceiveCount` (e.g. 3 retries before dead-letter routing).

---

## 9. Containers & Kubernetes Diagnostics

### ECS Task Failure Diagnostics
* **`Essential container in task exited`:** Application crashed or failed entrypoint command.
* **`CannotPullContainerError`:** ECR execution role missing (`AmazonECSTaskExecutionRolePolicy`) or private subnet lacks NAT Gateway / VPC Endpoint to ECR.
* **Container `OOMKilled`:** Container exceeded memory hard limit specified in task definition.

### Amazon EKS Debugging Hierarchy
```bash
# 1. Node NotReady: Inspect kubelet service & node pressure
systemctl status kubelet
journalctl -u kubelet -n 100 --no-pager

# 2. Pod Pending: Check resource capacity, unschedulable taints, missing PVCs
kubectl describe pod <pod-name> -n <namespace>

# 3. CrashLoopBackOff: Inspect prior container exit logs
kubectl logs <pod-name> --previous -n <namespace>
```

---

## 10. Infrastructure as Code Debugging

### CloudFormation & Terraform State Recovery
* **CloudFormation `UPDATE_ROLLBACK_FAILED`:** A resource being deleted/updated encountered a dependency lock or permission issue.
  * *Fix:* Use `ContinueUpdateRollback` and specify `ResourcesToSkip` to bypass unrecoverable blockers.
* **Terraform State Lock Stuck (`Error acquiring the state lock`):** Previous CI/CD pipeline crashed without releasing DynamoDB lock table.
  * *Fix:* Verify no active pipeline is executing, then force-unlock: `terraform force-unlock <LOCK-ID>`.
* **State Drift:** Manual AWS console changes diverge from `.tf` declarations.
  * *Fix:* Run `terraform plan` to identify drift; update code or import missing resources.

---

## 11. Edge & Security Perimeter Troubleshooting

* **CloudFront 502 Bad Gateway:** Origin SSL certificate expired, domain mismatch on origin certificate, or origin server took longer than `OriginResponseTimeout`.
* **AWS WAF False Positives:** Managed rule blocks legitimate user requests (e.g. SQLi rule blocking valid markdown text).
  * *Fix:* Inspect WAF sampled requests in CloudWatch Insights, identify matching `RuleId`, and set rule action to `Count` or add a scope-down exclusion.
* **Secrets Manager `AccessDeniedException` on Decrypt:** Caller has `secretsmanager:GetSecretValue` permission but lacks `kms:Decrypt` permission on the customer-managed KMS key.

---

## 12. Performance Optimization & Latency Bottlenecks

```
                                [ REQUEST LATENCY SPIKE ]
                                            |
                         +------------------+------------------+
                         |                                     |
                         v                                     v
            [ Infrastructure Level ]                 [ Application / DB Level ]
            - CPU/Memory Throttling                  - DB Slow Queries / Locks
            - EBS Burst Exhaustion                   - Missing Indexing
            - Network Bandwidth Saturation           - Connection Starvation
            - Cross-AZ Data Transfer                 - Synchronous Cascading Calls
```

---

## 13. AWS Cloud Security Incident Response & Forensics Playbook

```
                         STEP-BY-STEP SECURITY CONTAINMENT
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Compromised IAM Keys : Deactivate access key immediately via CLI; revoke all active │
 │                           IAM role sessions via AWSRevokeExtendedSessions policy.      │
 │ 2. Infected EC2 Instance: Attach Quarantine Security Group (Inbound/Outbound Deny All);│
 │                           take EBS Volume Snapshots for forensics before terminating.  │
 │ 3. Public S3 Leak       : Enable S3 Block Public Access at account and bucket level.   │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 14. Disaster Recovery, Blameless RCA & 5 Whys Postmortem Templates

### Blameless RCA Example (5 Whys Framework)
* **Incident:** The production checkout API experienced a 45-minute outage during a flash sale.
  * **Why 1:** Application pods crashed with OOM errors.
  * **Why 2:** Traffic surged by 400% during marketing flash sale.
  * **Why 3:** Database reached maximum connection limits, causing API requests to queue in memory.
  * **Why 4:** Connection pooling was not configured between application servers and Aurora.
  * **Why 5:** The staging environment did not have automated load testing integrated into the CI/CD pipeline.
* **Corrective Actions:** Deploy Amazon RDS Proxy, enforce container memory limits, and mandate automated load testing for flash sale events.

---

## 15. Senior SRE & Cloud Architect Interview Playbooks

### High-Frequency Architectural & Troubleshooting Scenarios
1. **Q: A private EC2 instance cannot download security patches from the internet. How do you troubleshoot this step-by-step?**  
   *Answer:*
   * Check the subnet route table: verify a default route (`0.0.0.0/0`) points to a NAT Gateway.
   * Verify the NAT Gateway is deployed in a Public Subnet with an Elastic IP (EIP) attached.
   * Verify the public subnet's route table has `0.0.0.0/0` pointing to an Internet Gateway (IGW).
   * Inspect the private subnet's Network ACL (NACL) to ensure outbound HTTP/HTTPS and inbound ephemeral return ports (`1024–65535`) are allowed.
   * Check the EC2 instance's Security Group egress rules to confirm outbound traffic on ports 80/443 is permitted.
2. **Q: What causes an ALB to return HTTP 504 Gateway Timeout versus HTTP 502 Bad Gateway?**  
   *Answer:*
   * **HTTP 502 (Bad Gateway):** ALB successfully established a TCP connection with the target instance, but the target closed the connection unexpectedly, sent a malformed HTTP header, or experienced an immediate application crash.
   * **HTTP 504 (Gateway Timeout):** ALB established a connection with the target, but the target application failed to return a response before the ALB idle timeout expired (typically caused by slow database queries, deadlocks, or downstream API timeouts).
3. **Q: How do you resolve a "Hot Partition" issue in Amazon DynamoDB?**  
   *Answer:*
   * Identify hot keys using CloudWatch Contributor Insights for DynamoDB.
   * Implement write sharding by appending a random suffix (e.g., `user_id_1`, `user_id_2`) to distribute high-volume writes across partitions.
   * Deploy DynamoDB Accelerator (DAX) or ElastiCache to absorb read-heavy spikes in-memory.
   * Redesign composite primary keys (Partition Key + Sort Key) to ensure uniform data access.

---

## 16. Production Readiness & Outage Prevention Checklist

- [x] **1. Multi-AZ Everything:** Compute, databases, and NAT Gateways distributed across $\ge 3$ AZs.
- [x] **2. RDS Proxy Active:** Connection pooling configured to prevent serverless database exhaustion.
- [x] **3. Ephemeral Ports Open:** NACLs explicitly permit inbound/outbound ephemeral ports `1024-65535`.
- [x] **4. Dead-Letter Queues:** All SQS queues and Lambda asynchronous triggers backed by DLQs.
- [x] **5. IMDSv2 Enforced:** Token-based instance metadata enabled across all EC2 instances.
- [x] **6. KMS Key Policies:** Calling IAM roles explicitly delegated in Customer Managed Key policies.
- [x] **7. CloudWatch Alarms:** Alarms configured on ALB 5xx rates, Lambda throttles, and RDS CPU.
- [x] **8. Tested Runbooks:** Disaster recovery, quarantine security groups, and rollback runbooks verified.
