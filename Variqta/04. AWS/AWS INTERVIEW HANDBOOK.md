# 🎯 AWS Interview & Architecture Master Handbook
> **Your Ultimate Guide to AWS Technical, System Design & Troubleshooting Interview Success.**  
> *20-Page Complete Curriculum: Core Services, Architecture Scenarios, Deep-Dive Q&A, 10 Incident Triage Playbooks, Top 30 Interview Questions & Quick-Recall Cheat Sheet.*

---

## 📑 Table of Contents
1. [The AWS Interview Framework & Hiring Bar](#1-the-aws-interview-framework--hiring-bar)
2. [Amazon EC2 Interview Mastery (Lifecycle, Families, Storage & Pricing)](#2-amazon-ec2-interview-mastery)
3. [Amazon VPC Networking Deep Dive (CIDR Math, Subnets, Routing & NACLs)](#3-amazon-vpc-networking-deep-dive)
4. [Elastic Load Balancing & Auto Scaling (ALB vs. NLB & Target Tracking)](#4-elastic-load-balancing--auto-scaling)
5. [AWS IAM & Zero-Trust Security (Roles, STS, Policies & Evaluation Logic)](#5-aws-iam--zero-trust-security)
6. [Amazon S3 Architecture & Storage Optimization (Classes, OAC, KMS & WORM)](#6-amazon-s3-architecture--storage-optimization)
7. [Amazon RDS vs. Amazon Aurora (Multi-AZ, Read Replicas & Failover)](#7-amazon-rds-vs-amazon-aurora)
8. [Amazon Route 53 & DNS Routing Policies (Alias Records & Health Checks)](#8-amazon-route-53--dns-routing-policies)
9. [Amazon CloudFront & Global Edge Delivery (Caching, TTL, OAC & Invalidation)](#9-amazon-cloudfront--global-edge-delivery)
10. [Amazon API Gateway & Microservice Ingress (REST, HTTP, WebSocket & Auth)](#10-amazon-api-gateway--microservice-ingress)
11. [AWS Lambda & Serverless Compute (Cold Starts, Concurrency & Limits)](#11-aws-lambda--serverless-compute)
12. [Amazon SQS & Decoupled Messaging (Standard vs. FIFO, Visibility & DLQs)](#12-amazon-sqs--decoupled-messaging)
13. [AWS Monitoring & Observability (CloudWatch, CloudTrail, Config & X-Ray)](#13-aws-monitoring--observability)
14. [AWS Security & Key Management (KMS, Secrets Manager, GuardDuty & WAF)](#14-aws-security--key-management)
15. [Hybrid Cloud Networking (Transit Gateway, Direct Connect, VPN & PrivateLink)](#15-hybrid-cloud-networking)
16. [High Availability & Disaster Recovery (RTO vs. RPO & 4 DR Strategies)](#16-high-availability--disaster-recovery)
17. [Full System Design Scenario: Scalable Global E-Commerce Platform](#17-full-system-design-scenario-scalable-global-e-commerce-platform)
18. [Top 30 Most Frequently Asked AWS Interview Questions & Model Answers](#18-top-30-most-frequently-asked-aws-interview-questions)
19. [10 Real-World Production Troubleshooting Playbooks](#19-10-real-world-production-troubleshooting-playbooks)
20. [AWS Quick-Recall Cheat Sheet (Ports, Limits, Protocols & Decision Matrix)](#20-aws-quick-recall-cheat-sheet)

---

## 1. The AWS Interview Framework & Hiring Bar

```
                           THE 4 INTERVIEW STAGES
  [ 1. Screening Round ] ──> [ 2. Technical Deep Dive ] ──> [ 3. System Design ] ──> [ 4. Behavioral ]
  (30-45 mins: Basics)       (60-90 mins: Scenarios)        (45-60 mins: Scale)       (30-45 mins: Principles)
```

### What Interviewers Actually Evaluate
1. **Core Service Integration:** Understanding how AWS services connect cleanly and securely.
2. **Trade-Off Analysis:** Articulating *why* you chose Aurora over DynamoDB, or ALB over NLB.
3. **Failure Isolation:** Designing for failure across Availability Zones and Regions.
4. **Cost vs. Performance:** Eliminating waste while sustaining sub-second latency.

---

## 2. Amazon EC2 Interview Mastery

```
                             EC2 INSTANCE LIFECYCLE
  [ Pending ] ──> [ Running ] ──> [ Stopping ] ──> [ Stopped ] ──> [ Terminated ]
                     │                                 │
                     └──(Reboot)                       └──(Billed for EBS Storage Only)
```

### High-Frequency EC2 Q&A
* **Q: Can you change the instance type of an existing EC2 instance?**  
  *Answer:* Yes, if the root volume is EBS-backed. Stop the instance, change the instance type in the console/CLI, and start it again. (Not possible with Instance Store-backed instances).
* **Q: What is the difference between EBS and Instance Store?**  
  *Answer:* **EBS** is network-attached persistent block storage (retained on stop/terminate, supports snapshots). **Instance Store** is physically attached ephemeral NVMe storage providing ultra-high IOPS, but all data is lost when the instance is stopped or hardware fails.

---

## 3. Amazon VPC Networking Deep Dive

```
                             VPC CIDR /24 SUBNET ANATOMY
  Total IPs = 256. AWS reserves 5 IP addresses in every subnet:
  • 10.0.1.0   : Network Address
  • 10.0.1.1   : VPC Router
  • 10.0.1.2   : AWS DNS (AmazonProvidedDNS)
  • 10.0.1.3   : Reserved by AWS for future use
  • 10.0.1.255 : Network Broadcast Address
  Usable IPs per /24 subnet = 251.
```

### Security Groups vs. Network ACLs
| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Operating Level** | Instance / ENI Level | Subnet Boundary Level |
| **State** | **Stateful** (Inbound return traffic auto-allowed) | **Stateless** (Must allow inbound & outbound) |
| **Rule Types** | Allow rules only | Allow and Deny rules (Numbered order) |
| **Evaluation** | Evaluated after NACL | Evaluated first upon packet arrival |

---

## 4. Elastic Load Balancing & Auto Scaling

```
                         ALB VS. NLB COMPARISON
 ┌──────────────────────┬────────────────────────────────────┬────────────────────────────────────┐
 │ Feature              │ Application Load Balancer (ALB)    │ Network Load Balancer (NLB)        │
 ├──────────────────────┼────────────────────────────────────┼────────────────────────────────────┤
 │ **OSI Layer**        │ Layer 7 (Application)              │ Layer 4 (Transport)                │
 │ **Protocols**        │ HTTP, HTTPS, gRPC, WebSockets      │ TCP, UDP, TLS                      │
 │ **Routing Criteria** │ Host, Path, HTTP Headers, Cookies  │ IP Address and Port only           │
 │ **Static IP / EIP**  │ No (Dynamic DNS name)              │ Yes (Static Elastic IP per AZ)     │
 │ **Performance**      │ High (millions req/sec)            │ Ultra-High (tens of millions/sec)  │
 └──────────────────────┴────────────────────────────────────┴────────────────────────────────────┘
```

---

## 5. AWS IAM & Zero-Trust Security

```
                           IAM EVALUATION LOGIC ENGINE
  1. Default Deny ──> 2. Is there an Explicit DENY? ──Yes──> [ ACCESS DENIED ]
                            │ No
                      3. Is there an Explicit ALLOW? ──Yes─> [ ACCESS GRANTED ]
                            │ No
                      4. [ ACCESS DENIED ]
```

### IAM Trust Policy (Who Can Assume the Role)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 6. Amazon S3 Architecture & Storage Optimization

```
                           S3 STORAGE CLASS RETRIEVAL & COST
  Class                  Durability    Min Duration    Retrieval Fee    Primary Use Case
  ┌────────────────────┬─────────────┬───────────────┬────────────────┬───────────────────────────┐
  │ S3 Standard        │ 11 9s       │ None          │ None           │ Active, frequent datasets │
  │ S3 Intelligent     │ 11 9s       │ None          │ None           │ Unknown access patterns   │
  │ S3 Standard-IA     │ 11 9s       │ 30 Days       │ Per GB         │ Monthly backups / DR      │
  │ Glacier Instant    │ 11 9s       │ 90 Days       │ Per GB (ms)    │ Quarterly archival        │
  │ Glacier Deep Arch  │ 11 9s       │ 180 Days      │ Per GB (12-48h)│ 7-10 Year compliance WORM │
  └────────────────────┴─────────────┴───────────────┴────────────────┴───────────────────────────┘
```

---

## 7. Amazon RDS vs. Amazon Aurora

```
                        AURORA STORAGE ARCHITECTURE (6-WAY REPLICATION)
  [ Aurora DB Cluster ]
         │
         ├──> [ AZ-A: Copy 1 ] [ AZ-A: Copy 2 ]
         ├──> [ AZ-B: Copy 3 ] [ AZ-B: Copy 4 ]
         └──> [ AZ-C: Copy 5 ] [ AZ-C: Copy 6 ]
  (Self-healing 10GB chunks; survives loss of an entire AZ + 1 additional copy without write loss)
```

---

## 8. Amazon Route 53 & DNS Routing Policies

* **Simple Routing:** Single resource lookup (No health checks).
* **Weighted Routing:** Distribute traffic by percentage (e.g., 90% production, 10% canary).
* **Latency-Based Routing:** Routes users to the AWS region providing lowest latency.
* **Failover Routing:** Active-Passive disaster recovery triggered by Route 53 health check status.
* **Geolocation Routing:** Routes users based on geographic origin (e.g., Europe to Frankfurt region).
* **Alias Record Advantage:** Free DNS queries when pointing to AWS resources (ALB, CloudFront, S3).

---

## 9. Amazon CloudFront & Global Edge Delivery

```
                       CLOUDFRONT ORIGIN ACCESS CONTROL (OAC)
  [ User ] ──(HTTPS)──> [ CloudFront Edge Cache ] ──(SigV4 Signed Request)──> [ Private S3 Bucket ]
```

* **Cache Invalidation:** Force instant cache purge using `/*` or specific path `/images/logo.png`.
* **Signed URLs vs. Signed Cookies:** Signed URLs for individual file access (e.g., video download link); Signed Cookies for accessing multiple files (e.g., entire premium course library).

---

## 10. Amazon API Gateway & Microservice Ingress

```
                           API GATEWAY AUTHENTICATION TYPES
  1. IAM Authentication      ──> Internal AWS services and IAM principals.
  2. Cognito User Pools      ──> Mobile and web apps with OAuth2/OIDC user sign-in.
  3. Lambda Authorizer       ──> Custom OAuth bearer tokens, HMAC, or third-party validation.
  4. JWT Authorizer (HTTP)   ──> Fast, low-cost native OpenID/OAuth JWT token decoding.
```

---

## 11. AWS Lambda & Serverless Compute

```
                             LAMBDA EXECUTION MODEL
  [ Event Trigger ] ──> [ Cold Start: Download Code + Init Runtime ] ──> [ Warm Handler Execution ]
                               │
                               └──(Mitigation: Provisioned Concurrency pre-warms environments)
```

### Lambda Technical Limits
* **Maximum Execution Timeout:** 15 minutes (900 seconds).
* **Memory Allocation:** 128 MB to 10,240 MB (CPU scales proportionally with memory).
* **Temporary Storage (`/tmp`):** 512 MB to 10 GB.
* **Unzipped Deployment Package:** 250 MB.

---

## 12. Amazon SQS & Decoupled Messaging

```
                         STANDARD QUEUE VS. FIFO QUEUE
 ┌──────────────────────┬────────────────────────────────────┬────────────────────────────────────┐
 │ Feature              │ Standard Queue                     │ FIFO Queue                         │
 ├──────────────────────┼────────────────────────────────────┼────────────────────────────────────┤
 │ **Ordering**         │ Best-effort ordering               │ Strict First-In-First-Out (FIFO)   │
 │ **Delivery**         │ At-least-once (Duplicates possible)│ Exactly-once processing            │
 │ **Throughput**       │ Nearly unlimited TPS               │ 300 msg/s (3,000 with batching)    │
 │ **Naming**           │ Any valid name                     │ Must end in `.fifo` suffix         │
 └──────────────────────┴────────────────────────────────────┴────────────────────────────────────┘
```

---

## 13. AWS Monitoring & Observability

```
 ┌───────────────────┬──────────────────────────────────────────────────────────────────┐
 │ Service           │ Primary Engineering Purpose                                      │
 ├───────────────────┼──────────────────────────────────────────────────────────────────┤
 │ **CloudWatch**    │ System performance metrics (CPU, RAM via agent), logs, and alarms│
 │ **CloudTrail**    │ Security auditing: records every API call (Who, What, When)      │
 │ **AWS Config**    │ Evaluates resource configuration drift and compliance history    │
 │ **AWS X-Ray**     │ Distributed tracing, latency breakdown, and service call maps    │
 └───────────────────┴──────────────────────────────────────────────────────────────────┘
```

---

## 14. AWS Security & Key Management

* **AWS KMS (Envelope Encryption):** KMS encrypts a Data Encryption Key (DEK) with the Customer Master Key (CMK); the application uses the plaintext DEK to encrypt data locally, then destroys plaintext DEK from memory.
* **AWS Secrets Manager:** Automatically rotates database credentials using Lambda without application downtime.
* **Amazon GuardDuty:** Continuous threat intelligence using ML to detect compromised EC2 instances, crypto-mining, and unauthorized IAM actions.

---

## 15. Hybrid Cloud Networking

```
                         AWS HYBRID NETWORKING SPECTRUM
  [ Corporate Data Center ]
         │
         ├──(Internet / IPsec)──────────> [ AWS Site-to-Site VPN ] (Encrypted, variable latency)
         └──(Dedicated Physical Fiber)──> [ AWS Direct Connect ] (1/10/100 Gbps, ultra-low latency)
                                                         │
                                                         ▼
                                             [ AWS Transit Gateway ] ──> [ 100+ VPCs ]
```

---

## 16. High Availability & Disaster Recovery

```
                           THE 4 DISASTER RECOVERY TIERS
  Strategy             RTO           RPO           Cost Scale
  ┌──────────────────┬─────────────┬─────────────┬────────────┐
  │ Backup & Restore │ Hours       │ Hours       │ $          │
  │ Pilot Light      │ 10-60 Mins  │ 5-15 Mins   │ $$         │
  │ Warm Standby     │ Minutes     │ < 5 Mins    │ $$$        │
  │ Multi-Site Active│ Seconds     │ Near Zero   │ $$$$$      │
  └──────────────────┴─────────────┴─────────────┴────────────┘
```

---

## 17. Full System Design Scenario: Scalable Global E-Commerce Platform

```
                       ENTERPRISE E-COMMERCE ARCHITECTURE
  [ Global Users ] ──> [ Route 53 (Latency Routing) ] ──> [ CloudFront CDN + AWS WAF ]
                                                                   │
                                                                   ▼
                                                   [ Application Load Balancer ]
                                                                   │
         ┌─────────────────────────────────────────────────────────┴─────────────────────────────────────────┐
         ▼                                                                                                   ▼
  [ Private Subnet AZ-A: EC2 ASG ]                                                    [ Private Subnet AZ-B: EC2 ASG ]
  ├── Web Application Instance                                                        ├── Web Application Instance
  └── IAM Role Attached                                                               └── IAM Role Attached
         │                                                                                                   │
         └─────────────────────────────────────────────────────────┬─────────────────────────────────────────┘
                                                                   ▼
                                  ┌────────────────────────────────┴────────────────────────────────┐
                                  ▼                                                                 ▼
                   [ ElastiCache Redis Cluster ]                                     [ Amazon Aurora Multi-AZ (ACID) ]
                   (Product Catalog & Cart Cache)                                    (Orders, Payments, User Accounts)
```

---

## 18. Top 30 Most Frequently Asked AWS Interview Questions

1. **Q: What is the difference between an IAM Role and an IAM User?**  
   *Answer:* An IAM User has permanent long-term credentials (password, access keys). An IAM Role has no permanent credentials and is assumed dynamically by services, users, or applications to receive temporary STS credentials.
2. **Q: How does Aurora achieve sub-30 second failover compared to standard RDS?**  
   *Answer:* Aurora uses a shared distributed storage volume across 3 AZs. When the primary writer fails, an Aurora Read Replica is promoted to writer simply by pointing to the existing shared storage volume, eliminating the need to replay WAL logs.
3. **Q: What is the purpose of a Dead-Letter Queue (DLQ)?**  
   *Answer:* A DLQ isolates problematic or unprocessable messages after a maximum retry count (`maxReceiveCount`), preventing poison-pill messages from blocking the main queue.
4. **Q: How do you allow private EC2 instances to download OS patches without public IPs?**  
   *Answer:* Deploy a **NAT Gateway** in a public subnet, and configure the private subnet's route table with `0.0.0.0/0 -> nat-xxxx`.
5. **Q: How can you protect against accidental deletion of critical S3 objects?**  
   *Answer:* Enable **S3 Versioning**, **MFA Delete**, and **S3 Object Lock (Compliance Mode)** with strict IAM bucket policies.

---

## 19. 10 Real-World Production Troubleshooting Playbooks

| # | Outage / Incident Scenario | Primary Root Cause | Step-by-Step Diagnostic & Fix Runbook |
|:---:|:---|:---|:---|
| **1** | **EC2 SSH Connection Timeout** | Security Group / Route Table block. | Verify Inbound Port 22 allows client IP; ensure subnet route table has `0.0.0.0/0 -> IGW`. |
| **2** | **RDS DB Connection Failed** | SG isolation or DB subnet misconfig. | Verify DB Security Group allows inbound 3306/5432 from App SG; check DB instance status. |
| **3** | **S3 403 Access Denied** | Missing IAM/KMS or Bucket Policy Deny.| Check IAM permissions (`s3:GetObject`), bucket policy explicit denies, and KMS key decrypt rights. |
| **4** | **ALB Unhealthy Targets (503)**| Application crashed or bad health path.| Check Target Group health check path (`/healthz`); verify web app is listening on target port. |
| **5** | **CloudWatch Alarm Not Firing**| Insufficient data or wrong metric. | Verify metric namespace (`AWS/EC2`), statistics (`Average`), and evaluation period thresholds. |
| **6** | **Lambda Function Timing Out** | Slow downstream DB or VPC ENI lock. | Increase Lambda timeout limit; increase RAM; verify NAT Gateway routing if inside private VPC. |
| **7** | **Route 53 DNS Not Resolving** | TTL caching or wrong NS delegation. | Check A/Alias record values; test via `dig @8.8.8.8 domain.com`; verify domain registrar NS records. |
| **8** | **CloudWatch Logs Missing** | Missing IAM permissions on EC2/Lambda. | Attach `AWSLambdaBasicExecutionRole` or `CloudWatchAgentServerPolicy` to the execution role. |
| **9** | **VPC Private Subnet No Internet**| NAT Gateway missing or misrouted. | Ensure NAT Gateway resides in **Public Subnet** with attached Elastic IP, and route table points to it. |
| **10**| **KMS Access Denied** | KMS Key Policy excludes caller role. | Update KMS Key Policy statement to explicitly allow caller's IAM Role ARN for `kms:Decrypt`. |

---

## 20. AWS Quick-Recall Cheat Sheet

```
                               DEFAULT SERVICE LIMITS & PORTS
  • Default VPC Limit   : 5 per Region             • Port 22   : SSH (Linux)
  • Default S3 Limit    : 100 Buckets per Account   • Port 80   : HTTP
  • SQS Visibility Limit: 12 Hours Max (Default 30s)• Port 443  : HTTPS
  • Lambda Timeout Limit: 15 Minutes (900 Seconds)  • Port 3306 : MySQL / Aurora
  • Default Subnet IPs  : 5 Reserved by AWS         • Port 5432 : PostgreSQL
  • Route 53 TTL Default: 300 Seconds (5 Minutes)   • Port 1433 : Microsoft SQL Server
```
