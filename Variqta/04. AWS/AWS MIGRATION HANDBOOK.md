# 🛠️ AWS Junior Cloud Engineer Master Handbook
> **Real-World Technical Questions, Scenarios, Production Fundamentals & Troubleshooting Playbooks.**  
> *20-Page Complete Practical Curriculum: Cloud Mindset, VPC, EC2, IAM, S3, RDS/Aurora, ALB/ASG, Route 53, CloudFront, CloudWatch, CLI Operations, Security Principles & Full Mock Interview.*

---

## 📑 Table of Contents
1. [The Junior AWS Engineer's Mindset & Interview Foundations](#1-the-junior-aws-engineers-mindset--interview-foundations)
2. [AWS Global Infrastructure & High-Availability Basics](#2-aws-global-infrastructure--high-availability-basics)
3. [AWS IAM & Access Management (Roles, Policies & Permission Denials)](#3-aws-iam--access-management)
4. [Amazon EC2 Fundamentals & "Why Can't I Connect?" Triage](#4-amazon-ec2-fundamentals--why-cant-i-connect-triage)
5. [Amazon VPC Fundamentals (CIDR Math, Subnets, IGW & NAT Gateways)](#5-amazon-vpc-fundamentals)
6. [Security Groups vs. Network ACLs (Stateful vs. Stateless Triage)](#6-security-groups-vs-network-acls)
7. [Amazon S3 Storage & Public-Access Troubleshooting Matrix](#7-amazon-s3-storage--public-access-troubleshooting-matrix)
8. [AWS Load Balancing (ALB vs. NLB & 502/503/504 Error Triage)](#8-aws-load-balancing)
9. [EC2 Auto Scaling Groups (Scaling Policies & Instance Churn Triage)](#9-ec2-auto-scaling-groups)
10. [AWS Database Ecosystem (RDS, Aurora Multi-AZ & DynamoDB Triage)](#10-aws-database-ecosystem)
11. [DNS & Amazon Route 53 (Alias Records & "Works by IP Not Domain" Triage)](#11-dns--amazon-route-53)
12. [Amazon CloudFront & Edge Delivery (TTL, Invalidation & Stale Content)](#12-amazon-cloudfront--edge-delivery)
13. [Monitoring with Amazon CloudWatch (7-Step Incident Investigation Workflow)](#13-monitoring-with-amazon-cloudwatch)
14. [AWS Logging & Auditing (CloudTrail, CloudWatch Logs & VPC Flow Logs)](#14-aws-logging--auditing)
15. [AWS CLI & Operational Mastery (JMESPath Queries & Safe Debugging)](#15-aws-cli--operational-mastery)
16. [Serverless AWS Fundamentals (Lambda, API Gateway & Timeout Triage)](#16-serverless-aws-fundamentals)
17. [AWS Security Fundamentals (6 Golden Principles & Junior Traps)](#17-aws-security-fundamentals)
18. [Web Application Architecture Scenario (Full Production Design)](#18-web-application-architecture-scenario)
19. [Real-World AWS Troubleshooting Matrix (Top 7 Incident Playbooks)](#19-real-world-aws-troubleshooting-matrix)
20. [Junior AWS Mock Interview (Technical, Scenario, Architecture & Triage)](#20-junior-aws-mock-interview)

---

## 1. The Junior AWS Engineer's Mindset & Interview Foundations

```
                          HOW TO SOLVE ANY AWS SCENARIO QUESTION
  [ 1. Understand Problem ] ──> [ 2. Clarify Constraints ] ──> [ 3. Break Down Layers ]
                                                                        │
  [ 6. Review & Optimize ] <── [ 5. Propose Architecture ] <── [ 4. Think Out Loud ]
```

### Cloud Fundamentals vs. Memorization
* **Focus on Concepts:** Understand *why* a service is used, *how* data flows, and *what trade-offs* exist.
* **Avoid Raw Memorization:** Don't just recite feature lists; explain real-world failure handling and cost implications.

---

## 2. AWS Global Infrastructure & High-Availability Basics

```
                               GLOBAL RESILIENCE HIERARCHY
  [ AWS Region (e.g. us-east-1) ]
  ├── [ Availability Zone A ] ──> [ Data Center 1 ] [ Data Center 2 ] (Dedicated Power/Network)
  ├── [ Availability Zone B ]
  └── [ Availability Zone C ]
  
  [ 600+ Global Edge Locations / PoPs ] ──> CloudFront CDN Low-Latency Caching
```

---

## 3. AWS IAM & Access Management

```
                                  IAM ACCESS EVALUATION
  Principal Request ──> [ Explicit Deny? ] ──Yes──> [ Access Denied ]
                              │ No
                        [ Explicit Allow? ] ──Yes─> [ Access Granted ]
                              │ No
                        [ Default Deny ]
```

### Common Permission-Denied Scenarios & Fixes
1. **AccessDenied when using AWS CLI:** IAM User/Role lacks permission $\rightarrow$ Attach required IAM policy.
2. **AccessDenied on S3 Bucket:** S3 Bucket Policy explicit deny or missing `s3:GetObject` $\rightarrow$ Review bucket policy.
3. **Cannot assume IAM Role:** Trust relationship policy does not list caller entity $\rightarrow$ Update trust policy (`sts:AssumeRole`).
4. **Access Key not working:** Key is inactive, deleted, or permissions missing $\rightarrow$ Rotate access key.

---

## 4. Amazon EC2 Fundamentals & "Why Can't I Connect?" Triage

```
                            EC2 SSH/RDP CONNECTION TRIAGE
  Cannot Connect to EC2?
  │
  ├── 1. Security Group ────> Verify Inbound Port 22 (SSH) allows your current IP
  ├── 2. Route Table ───────> Verify Subnet has default route `0.0.0.0/0 -> igw-xxxx`
  ├── 3. Public IP ─────────> Ensure instance has a public IPv4 or Elastic IP
  ├── 4. Key Pair (.pem) ───> Verify correct private key with `chmod 400 key.pem`
  ├── 5. OS User Name ──────> Amazon Linux: `ec2-user`, Ubuntu: `ubuntu`, RHEL: `ec2-user`
  └── 6. Instance State ────> Ensure EC2 status is `Running` (2/2 checks passed)
```

---

## 5. Amazon VPC Fundamentals

```
                             VPC CIDR /24 SUBNET RESERVATIONS
  Subnet: 10.0.1.0/24 (Total 256 IPs - 5 AWS Reserved = 251 Usable IPs):
  • 10.0.1.0   : Network Address
  • 10.0.1.1   : VPC Router
  • 10.0.1.2   : AWS DNS Server (AmazonProvidedDNS)
  • 10.0.1.3   : Reserved by AWS for future use
  • 10.0.1.255 : Network Broadcast Address
```

---

## 6. Security Groups vs. Network ACLs

```
 ┌──────────────────────┬────────────────────────────────────┬────────────────────────────────────┐
 │ Feature              │ Security Group (SG)                │ Network ACL (NACL)                 │
 ├──────────────────────┼────────────────────────────────────┼────────────────────────────────────┤
 │ **Scope**            │ Attached to ENI (Instance level)   │ Attached to Subnet Boundary        │
 │ **Statefulness**     │ **Stateful** (Return traffic allowed)│ **Stateless** (Must allow both ways)│
 │ **Rule Types**       │ Allow rules only                   │ Allow and Deny rules (Numbered)    │
 │ **Evaluation Order** │ Evaluated after NACL               │ Evaluated first on packet arrival  │
 └──────────────────────┴────────────────────────────────────┴────────────────────────────────────┘
```

---

## 7. Amazon S3 Storage & Public-Access Troubleshooting Matrix

```
                       S3 PUBLIC ACCESS 6-STEP VERIFICATION
  1. Check Bucket Policy         ──> Ensure no unintended "Principal": "*" allows.
  2. Check Block Public Access   ──> Verify all 4 Block Public Access settings are enabled.
  3. Check IAM User Policies     ──> Verify calling user has explicit `s3:*` actions.
  4. Check Object ACLs           ──> Verify Object ACL does not grant public read.
  5. Check Bucket ACLs           ──> Verify Bucket ACL does not grant public read.
  6. Test Access                 ──> Test via incognito browser and AWS CLI.
```

---

## 8. AWS Load Balancing & 502/503/504 Error Triage

```
 ┌──────────────┬───────────────────────────────┬─────────────────────────────────────────────────┐
 │ Error Code   │ What It Means                 │ Primary Root Cause & Fix                        │
 ├──────────────┼───────────────────────────────┼─────────────────────────────────────────────────┤
 │ **502 Bad GW**│ Invalid response from target  │ App crashed or closed connection; check app logs│
 │ **503 Unavail│ No healthy targets in group   │ Targets failing health check; fix health path   │
 │ **504 Timeout│ Target failed to respond in time│ App processing slow; increase idle timeout / DB │
 └──────────────┴───────────────────────────────┴─────────────────────────────────────────────────┘
```

---

## 9. EC2 Auto Scaling Groups

```
                          ASG CAPACITY & LIFECYCLE
  [ Min Capacity: 2 ] ──> [ Desired Capacity: 3 ] ──> [ Max Capacity: 6 ]
                                 │
  ├── CPU > 60% ──> ASG Launches New Instances (Scale Out)
  └── CPU < 20% ──> ASG Terminates Excess Instances (Scale In)
```

---

## 10. AWS Database Ecosystem

* **Amazon RDS:** Managed relational DB (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server).
* **Amazon Aurora:** High-performance cloud-native relational DB (6 copies of data across 3 AZs, storage auto-scales to 128 TiB).
* **Amazon DynamoDB:** Fully managed serverless NoSQL key-value/document store with single-digit ms latency.

---

## 11. DNS & Amazon Route 53

```
                TRIAGE: "APP WORKS BY IP BUT NOT BY DOMAIN NAME"
  1. Test DNS Resolution      ──> Run `nslookup your-domain.com` or `dig your-domain.com`.
  2. Check Nameservers (NS)   ──> Ensure domain registrar NS matches Route 53 Hosted Zone NS.
  3. Check A / Alias Records  ──> Verify A/Alias record points to correct ALB DNS or IP.
  4. Check Route 53 Health    ──> If using failover routing, verify primary health check status.
  5. Clear Local DNS Cache    ──> Run `ipconfig /flushdns` (Windows) or restart browser.
```

---

## 12. Amazon CloudFront & Edge Delivery

```
                       CLOUDFRONT EDGE CACHING ARCHITECTURE
  [ Users Worldwide ] ──> [ CloudFront Edge Locations (600+ PoPs) ] ──> [ Origin: S3 / ALB ]
                                         │
                               (Cache Hit: Sub-10ms)
```

* **Stale Content Fix:** Invalidate cache via `aws cloudfront create-invalidation --distribution-id XXX --paths "/*"`.

---

## 13. Monitoring with Amazon CloudWatch

```
                       7-STEP INCIDENT INVESTIGATION WORKFLOW
  1. Identify Alert ──> 2. Check Metrics ──> 3. Review CloudWatch Logs Insights
                                                       │
  7. Document Postmortem <── 6. Take Action & Verify <── 5. Confirm Root Cause <── 4. Correlate Events
```

---

## 14. AWS Logging & Auditing

```
 ┌──────────────────────┬───────────────────────────────┬────────────────────────────────────────┐
 │ Service              │ What It Records               │ Primary Use Case                       │
 ├──────────────────────┼───────────────────────────────┼────────────────────────────────────────┤
 │ **CloudTrail**       │ API calls & account events    │ Security auditing (Who, What, When)    │
 │ **CloudWatch Logs**  │ Application & OS system logs  │ Real-time debugging & log aggregation  │
 │ **VPC Flow Logs**    │ Network IP/Port traffic       │ Network security & blocked packet audit│
 └──────────────────────┴───────────────────────────────┴────────────────────────────────────────┘
```

---

## 15. AWS CLI & Operational Mastery

```bash
# Filter running EC2 instances using JMESPath queries
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,PrivateIpAddress]" \
  --output table

# Dry-run test to check IAM permissions without making changes
aws s3 sync ./my-folder s3://my-bucket/ --dryrun
```

---

## 16. Serverless AWS Fundamentals

```
                               SERVERLESS API WORKFLOW
  [ Client ] ──> [ API Gateway ] ──> [ AWS Lambda ] ──> [ Amazon DynamoDB / S3 ]
                                            │
                                            ▼
                               [ CloudWatch Logs (/aws/lambda/...) ]
```

---

## 17. AWS Security Fundamentals

```
                            6 GOLDEN SECURITY PRINCIPLES
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ 1. Least Privilege  : Grant only the minimum permissions required for a task.         │
 │ 2. Encrypt All Data : Enable KMS encryption at rest and enforce TLS 1.2+ in transit.   │
 │ 3. Monitor Always   : Enable CloudWatch alarms, GuardDuty, and CloudTrail everywhere.  │
 │ 4. Audit Actions    : Centralize logs into an immutable S3 bucket with Object Lock.   │
 │ 5. Automate Security: Use AWS Config rules to auto-remediate non-compliant resources. │
 │ 6. Defense in Depth : Layer firewalls (AWS WAF -> Security Groups -> NACLs).          │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 18. Web Application Architecture Scenario

```
                       HIGHLY AVAILABLE WEB APPLICATION
  [ Users ] ──> [ Route 53 (DNS) ] ──> [ Application Load Balancer (Public Subnet) ]
                                                        │
         ┌──────────────────────────────────────────────┴──────────────────────────────────────────────┐
         ▼                                                                                             ▼
  [ AZ-A Private Subnet ]                                                       [ AZ-B Private Subnet ]
  ├── EC2 Web Instance (Managed by ASG)                                         ├── EC2 Web Instance (Managed by ASG)
  └── IAM Role Attached                                                         └── IAM Role Attached
         │                                                                                             │
         └──────────────────────────────────────────────┬──────────────────────────────────────────────┘
                                                        ▼
                                       [ Amazon RDS Multi-AZ Database ]
                                       [ Amazon S3 Static Asset Storage]
```

---

## 19. Real-World AWS Troubleshooting Matrix

| Issue Scenario | What to Check First | Key CLI Command / Log | Likely Cause & Fix |
| :--- | :--- | :--- | :--- |
| **EC2 Unreachable** | Inbound Security Group & Route Table | `aws ec2 describe-instance-status` | Add Inbound Port 22 rule; add `0.0.0.0/0 -> IGW`. |
| **ALB Unhealthy Target**| Target Group health check path & app logs | `aws elbv2 describe-target-health` | App is down or wrong health path `/healthz`. |
| **App Cannot Reach RDS**| RDS Security Group inbound source | `nc -zv <db-endpoint> 3306` | Allow Inbound 3306 from App SG only. |
| **S3 AccessDenied** | Bucket Policy & IAM permissions | `aws s3 ls s3://bucket-name/` | Grant `s3:GetObject` or remove explicit Deny. |
| **DNS Resolves Wrong IP**| Route 53 A/Alias record & TTL cache | `dig your-domain.com +trace` | Update Route 53 record; wait for TTL expiry. |
| **CPU Spikes to 100%** | Linux process list & CloudWatch | `top` / `htop` on instance | Kill rogue process; configure ASG to scale out. |
| **Lambda Timeout** | Execution duration metric & logs | `aws logs tail /aws/lambda/fn` | Increase Lambda timeout limit; increase RAM. |

---

## 20. Junior AWS Mock Interview

### 5 Rapid-Fire Architectural Q&A
1. **Q: What is the difference between an IAM User and an IAM Role?**  
   *Answer:* An IAM User has long-term credentials (password, access keys) for human operators. An IAM Role has no permanent credentials and is assumed dynamically by AWS services (EC2, Lambda) to obtain temporary, auto-expiring STS tokens.
2. **Q: Why should an RDS database be placed in a Private Subnet?**  
   *Answer:* Databases store sensitive persistent data and must never be directly exposed to the public internet. Placing RDS in a private subnet ensures it is only reachable by application servers inside the VPC.
3. **Q: What happens when an EC2 instance fails an ALB health check?**  
   *Answer:* The ALB marks the instance as `Unhealthy`, immediately stops routing new traffic to it, and routes all requests to remaining healthy instances. If attached to an ASG, the ASG will terminate and replace the unhealthy instance.
4. **Q: What is the difference between S3 Standard and S3 Glacier?**  
   *Answer:* **S3 Standard** provides millisecond retrieval for frequently accessed data. **S3 Glacier** is low-cost archival storage for rarely accessed data with retrieval times ranging from minutes to hours.
5. **Q: How does a NAT Gateway differ from an Internet Gateway?**  
   *Answer:* An **Internet Gateway (IGW)** allows two-way communication between public subnets and the internet. A **NAT Gateway** allows instances in private subnets to initiate outbound connections to the internet (for updates/patches) while blocking all inbound connections initiated from the outside.
