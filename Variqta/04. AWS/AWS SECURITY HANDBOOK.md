# 🛡️ AWS Security & Cloud Defense Production Master Handbook
> **From Cloud Security Foundations to Enterprise Multi-Account Defense & Incident Response.**  
> *20-Page Complete Enterprise Curriculum: Zero-Trust IAM, KMS Envelope Encryption, IMDSv2, Multi-Account SCPs, WAF/Shield, GuardDuty, Security Hub CSPM, Inspector, DevSecOps & Cloud Forensics Playbooks.*

---

## 📑 Table of Contents
1. [Executive Mental Model: Defense-in-Depth & Zero-Trust Architecture](#1-executive-mental-model-defense-in-depth--zero-trust-architecture)
2. [Securing the AWS Root User & Account Security Baseline](#2-securing-the-aws-root-user--account-security-baseline)
3. [IAM Fundamentals (Users, Groups, Roles, Policies & Evaluation Logic)](#3-iam-fundamentals)
4. [Advanced IAM, Permission Boundaries & Context-Aware Conditions](#4-advanced-iam-permission-boundaries--context-aware-conditions)
5. [AWS Organizations & Multi-Account Security Architecture (SCPs & Control Tower)](#5-aws-organizations--multi-account-security-architecture)
6. [VPC Network Security Fundamentals (Subnets, Stateful SGs & Stateless NACLs)](#6-vpc-network-security-fundamentals)
7. [Advanced Perimeter Defense (AWS WAF, Shield Advanced, Network Firewall & DNS FW)](#7-advanced-perimeter-defense)
8. [Data Encryption & AWS Key Management Service (KMS Envelope Encryption)](#8-data-encryption--aws-key-management-service-kms)
9. [Enterprise Secrets & Credential Management (Secrets Manager & SSM Parameter Store)](#9-enterprise-secrets--credential-management)
10. [Securing Amazon S3 Storage (Block Public Access, Bucket Policies & Object Lock)](#10-securing-amazon-s3-storage)
11. [Compute & Workload Hardening (IMDSv2, SSM Session Manager & Patch Manager)](#11-compute--workload-hardening)
12. [Enterprise Database Security (RDS Multi-AZ, IAM DB Auth & KMS Encryption)](#12-enterprise-database-security)
13. [AWS CloudTrail & Centralized Security Auditing (Log Integrity & S3 WORM)](#13-aws-cloudtrail--centralized-security-auditing)
14. [Continuous Compliance with AWS Config (Rules, Conformance Packs & Auto-Remediation)](#14-continuous-compliance-with-aws-config)
15. [Threat Detection & AI Intelligence with Amazon GuardDuty](#15-threat-detection--ai-intelligence-with-amazon-guardduty)
16. [Centralized Security Operations with AWS Security Hub (CSPM & ASFF)](#16-centralized-security-operations-with-aws-security-hub)
17. [Vulnerability Management with Amazon Inspector (EC2, ECR & Lambda)](#17-vulnerability-management-with-amazon-inspector)
18. [DevSecOps & Secure CI/CD Pipelines (Shift-Left, OIDC & Cosign Signatures)](#18-devsecops--secure-cicd-pipelines)
19. [AWS Incident Response & Cloud Forensics (10-Step Triage & Containment)](#19-aws-incident-response--cloud-forensics)
20. [Complete Production AWS Multi-Account Security Architecture & 15-Point Checklist](#20-complete-production-aws-multi-account-security-architecture)

---

## 1. Executive Mental Model: Defense-in-Depth & Zero-Trust Architecture

```
                            THE 6-LAYER DEFENSE-IN-DEPTH MODEL
  ┌────────────────────────────────────────────────────────────────────────────────────────┐
  │ Layer 1: Perimeter Security  ──> AWS WAF, AWS Shield Advanced, Route 53 DNS Firewall   │
  │ Layer 2: Network Security    ──> Custom VPC, Private Subnets, Stateful SGs, NACLs      │
  │ Layer 3: Identity & Access   ──> IAM Identity Center (SSO), MFA, Temporary STS Tokens   │
  │ Layer 4: Compute Hardening   ──> IMDSv2 Mandatory, SSM Session Manager (No Port 22)   │
  │ Layer 5: Data Protection     ──> KMS Envelope Encryption (CMKs), S3 Object Lock (WORM) │
  │ Layer 6: Detection & Auditing──> GuardDuty ML Threat Intel, CloudTrail, Security Hub   │
  └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Securing the AWS Root User & Account Security Baseline

```
                        ROOT ACCOUNT ZERO-TRUST HARDENING
  [ Root Account User ] ──> Lock with Hardware FIDO2 / Virtual MFA
                            └── Delete all Access Keys & Secret Keys
                            └── Configure Alternate Security & Billing Contacts
                            └── Restrict daily operations strictly to IAM SSO users
```

---

## 3. IAM Fundamentals

```
                             IAM EVALUATION LOGIC ENGINE
  1. Default Deny ──> 2. Explicit Deny in any Policy? ──Yes──> [ ACCESS DENIED ]
                             │ No
                      3. Explicit Allow in Identity/Resource? ──Yes─> [ ACCESS GRANTED ]
                             │ No
                      4. [ DEFAULT DENIED ]
```

### Identity-Based Policy vs. Resource-Based Policy
* **Identity-Based Policy:** Attached to Users, Groups, or Roles (`"Resource": "arn:aws:s3:::bucket/*"`).
* **Resource-Based Policy:** Attached directly to resources (S3 Bucket, SQS Queue, KMS Key) and includes a `"Principal"` field defining who has access.

---

## 4. Advanced IAM, Permission Boundaries & Context-Aware Conditions

```
                       PERMISSION BOUNDARIES MAXIMUM CEILING
  [ Effective Permissions ] = [ Identity-Based Policy Permissions ] ⋂ [ Permission Boundary Ceiling ]
```

### Context-Aware Condition Policy Example (Enforce MFA & IP Whitelist)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowActionsOnlyWithMFAAndCorpIP",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "Bool": { "aws:MultiFactorAuthPresent": "true" },
        "IpAddress": { "aws:SourceIp": ["203.0.113.0/24"] }
      }
    }
  ]
}
```

---

## 5. AWS Organizations & Multi-Account Security Architecture

```
                                AWS CONTROL TOWER ROOT
                                          │
         ┌────────────────────────────────┼────────────────────────────────┐
         ▼                                ▼                                ▼
  [ Security OU ]                  [ Workloads OU ]                 [ Shared Services OU ]
  ├── Audit Account (CloudTrail)   ├── Production Account           ├── Identity Center (SSO)
  ├── Log Archive Account (S3 WORM)├── Staging Account              ├── Transit Gateway Network
  └── Security Tooling (GuardDuty) └── Development Account          └── Inspection VPC
```

### Critical Service Control Policy (SCP) Guardrails
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyRootUserActions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": { "StringLike": { "aws:PrincipalArn": "arn:aws:iam::*:root" } }
    },
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": { "aws:RequestedRegion": ["us-east-1", "us-west-2"] }
      }
    },
    {
      "Sid": "ProtectSecurityServices",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "guardduty:DeleteDetector",
        "securityhub:DisableSecurityHub"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 6. VPC Network Security Fundamentals

```
                        3-TIER SECURITY GROUP ISOLATION
  [ Internet ] ──(HTTPS: 443)──> [ sg-alb (Public ALB) ]
                                         │
                               (TCP: 8080 - Source: sg-alb)
                                         ▼
                                [ sg-app (Private EC2/EKS) ]
                                         │
                               (TCP: 3306 - Source: sg-app)
                                         ▼
                                [ sg-db (Isolated Database) ]
```

---

## 7. Advanced Perimeter Defense

```
                            PERIMETER DEFENSE IN DEPTH
  [ Internet Ingress ]
          │
          ├──> [ AWS Shield Advanced ] ──> Automatic Layer 3/4 DDoS Mitigation
          ├──> [ AWS WAF ] ──────────────> OWASP Top 10, SQLi, XSS, Rate Limiting
          ├──> [ Route 53 DNS Firewall]──> Blocks C2 / Malicious Domain Exfiltration
          └──> [ AWS Network Firewall ]──> Deep Packet Inspection (Suricata L7 Rules)
```

---

## 8. Data Encryption & AWS Key Management Service (KMS)

```
                            KMS ENVELOPE ENCRYPTION FLOW
  [ Plaintext Sensitive Data ]
               │
               ├──(1. KMS Generates Data Encryption Key - DEK)
               │
               ▼ (2. Encrypted with Plaintext DEK in Memory)
  [ Encrypted Data (Ciphertext) ]
               │
               ▼ (3. KMS Encrypts DEK with Customer Master Key - CMK)
  [ Ciphertext Data Stored with Encrypted DEK ] ──> Plaintext DEK Destroyed from Memory!
```

---

## 9. Enterprise Secrets & Credential Management

```
 ┌──────────────────────┬────────────────────────────────────────┬───────────────────────────────────────┐
 │ Feature              │ AWS Secrets Manager                    │ Systems Manager Parameter Store       │
 ├──────────────────────┼────────────────────────────────────────┼───────────────────────────────────────┤
 │ **Primary Use Case** │ Database passwords, API tokens, OAuth  │ Application config & environment vars │
 │ **Secret Rotation**  │ Built-in automated rotation via Lambda │ Manual or custom scripting            │
 │ **Encryption**       │ Integrated with AWS KMS CMKs           │ Standard (Free) or SecureString (KMS) │
 │ **Cross-Account**    │ Native cross-account sharing           │ Supported with resource policies      │
 └──────────────────────┴────────────────────────────────────────┴───────────────────────────────────────┘
```

---

## 10. Securing Amazon S3 Storage

```
                           S3 ZERO-TRUST SECURITY STACK
  [ S3 Bucket ]
  ├── 1. S3 Block Public Access (Enforced at Account & Bucket level)
  ├── 2. Default Server-Side Encryption with KMS CMKs (`aws:kms`)
  ├── 3. Enforce TLS in Transit via Bucket Policy (`aws:SecureTransport: "false" -> Deny`)
  ├── 4. S3 Object Lock (Compliance Mode WORM - Prevents root/admin deletion)
  └── 5. CloudTrail S3 Data Events + S3 Server Access Logging
```

---

## 11. Compute & Workload Hardening

```bash
# Enforce IMDSv2 on EC2 Instances (Mitigate SSRF Vulnerabilities)
aws ec2 modify-instance-metadata-options \
  --instance-id i-0123456789abcdef0 \
  --http-tokens required \
  --http-put-response-hop-limit 1 \
  --http-endpoint enabled
```

* **SSM Session Manager:** Access EC2 shell instances through encrypted HTTPS tunnels without opening inbound Port 22 or deploying Bastion hosts.

---

## 12. Enterprise Database Security

* **Private Subnet Placement:** Databases never receive public IP addresses.
* **IAM Database Authentication:** Users authenticate to RDS MySQL/Postgres using temporary IAM tokens instead of hardcoded database passwords.
* **Encryption at Rest:** Storage volumes, automated backups, read replicas, and snapshots encrypted via AWS KMS.

---

## 13. AWS CloudTrail & Centralized Security Auditing

```
                          CENTRALIZED AUDITING PIPELINE
  [ Member Account A ] ──┐
  [ Member Account B ] ──┼──> [ Organization CloudTrail ] ──> [ S3 Log Archive (Object Lock) ]
  [ Member Account C ] ──┘                                                 │
                                                                           ▼
                                                             [ Log File Integrity Enabled ]
                                                             (SHA-256 Digest Verification)
```

---

## 14. Continuous Compliance with AWS Config

```
                          AWS CONFIG AUTO-REMEDIATION LOOP
  [ S3 Bucket Created with Public Read ]
                    │
                    ▼
  [ AWS Config Rule Evaluates: `s3-bucket-public-read-prohibited` ] ──> Status: NON_COMPLIANT
                    │
                    ▼
  [ Triggers SSM Automation Document: `AWS-DisableS3BucketPublicReadWrite` ]
                    │
                    ▼
  [ Public Access Block Automatically Re-enabled in < 30 Seconds ]
```

---

## 15. Threat Detection & AI Intelligence with Amazon GuardDuty

```
                               GUARDDUTY DETECTION ENGINE
  Data Ingest Sources:
  ├── VPC Flow Logs ──> Port scanning, unusual outbound SSH, crypto-mining traffic
  ├── DNS Logs      ──> C2 (Command & Control) server communication
  ├── CloudTrail    ──> Anomalous IAM privilege escalation, root login, unusual regions
  └── EKS Audit Logs──> Suspicious container execs and anonymous API requests
```

---

## 16. Centralized Security Operations with AWS Security Hub

```
                         SECURITY HUB CLOUD SECURITY POSTURE (CSPM)
  Ingests & Normalizes (ASFF):
  ├── Amazon GuardDuty Threats
  ├── Amazon Inspector Vulnerabilities
  ├── AWS Config Compliance Violations
  └── IAM Access Analyzer Findings
         │
         ▼
  Evaluates Standards: CIS AWS Foundations Benchmark, NIST SP 800-53, PCI-DSS v4.0
```

---

## 17. Vulnerability Management with Amazon Inspector

* **EC2 Scanning:** Detects unpatched CVEs and software application vulnerabilities.
* **ECR Container Scanning:** Continuously scans container images in Amazon ECR upon push and continuously against new CVE databases.
* **AWS Lambda Code Scanning:** Evaluates function code and third-party dependencies for known vulnerabilities.

---

## 18. DevSecOps & Secure CI/CD Pipelines

```
                             SECURE CI/CD PIPELINE
  [ Dev Commit ] ──> [ IaC Scan (Checkov) ] ──> [ Container Scan (Trivy) ] ──> [ Cosign Signature ]
                                                                                       │
                                                              (OIDC Short-Lived Role)  ▼
                                                                             [ Deploy to EKS ]
```

---

## 19. AWS Incident Response & Cloud Forensics

```
                        10-STEP CLOUD INCIDENT RESPONSE PLAYBOOK
  1. Detect Alert      ──> GuardDuty / Security Hub triggers High/Critical severity finding.
  2. Triage & Scope    ──> Identify affected AWS account, instance ID, IAM role, and timestamp.
  3. Isolate Instance  ──> Attach Quarantine Security Group (Deny all inbound/outbound traffic).
  4. Revoke IAM Tokens ──> Attach inline Deny policy to IAM role and revoke STS sessions.
  5. Preserve Evidence ──> Create EBS Volume Snapshot of compromised instance for forensics.
  6. Memory Capture    ──> Take volatile memory dump via AWS Systems Manager if possible.
  7. CloudTrail Audit  ──> Query CloudTrail for all API calls executed by the compromised identity.
  8. Eradicate Root    ──> Patch underlying vulnerability or update outdated AMI.
  9. Rebuild & Recover ──> Deploy clean workload instance from validated Golden AMI / IaC.
  10. Postmortem       ──> Conduct blameless retrospective and update automated Config/WAF rules.
```

---

## 20. Complete Production AWS Multi-Account Security Architecture

```
                       ENTERPRISE SECURITY BLUEPRINT
  [ Global Users ] ──> [ AWS Shield Advanced + AWS WAF ] ──> [ Route 53 DNS Firewall ]
                                                                   │
                                                                   ▼
                                                     [ Inspection VPC (Network Firewall) ]
                                                                   │
                                                                   ▼ (Transit Gateway)
         ┌─────────────────────────────────────────────────────────┴─────────────────────────────────────────┐
         ▼                                                                                                   ▼
  [ Production Workloads Account ]                                                    [ Security Operations Account ]
  ├── Private Subnet EC2/EKS Workloads (IMDSv2 Enforced)                              ├── AWS Security Hub CSPM Dashboard
  ├── AWS KMS Customer Managed Keys (Automatic Rotation)                             ├── Amazon GuardDuty ML Threat Detector
  └── IAM Roles for Service Accounts (Zero Static Keys)                               └── Central Amazon Inspector Engine
         │                                                                                                   │
         └─────────────────────────────────────────────────────────┬─────────────────────────────────────────┘
                                                                   ▼
                                                    [ Log Archive Account (Isolated) ]
                                                    ├── Centralized CloudTrail Organization Trail
                                                    ├── Immutable S3 Bucket with Object Lock (WORM)
                                                    └── AWS Config Conformance Packs
```

### 15-Point Enterprise Security Readiness Checklist
- [x] **1. Root Account MFA:** Hardware/Virtual MFA enabled on root; zero static access keys exist.
- [x] **2. IAM Identity Center:** Centralized SSO configured with mandatory MFA for all engineers.
- [x] **3. Least Privilege Roles:** EC2, Lambda, and EKS workloads use IAM Roles with least privilege.
- [x] **4. Mandatory IMDSv2:** Instance Metadata Service v2 enforced on all compute instances.
- [x] **5. KMS CMK Encryption:** All EBS volumes, RDS databases, and S3 buckets encrypted with CMKs.
- [x] **6. S3 Block Public Access:** Enabled globally at account level and verified with Access Analyzer.
- [x] **7. S3 Object Lock:** WORM immutability configured on centralized security log archive buckets.
- [x] **8. Multi-Account SCPs:** Guardrails enforced to block root user API calls and unapproved regions.
- [x] **9. Perimeter WAF & Shield:** Layer 7 rate-limiting and Layer 3/4 DDoS protection active.
- [x] **10. Inspection VPC:** AWS Network Firewall or IDS/IPS deployed for centralized traffic inspection.
- [x] **11. Threat Detection:** Amazon GuardDuty enabled across all active regions and member accounts.
- [x] **12. Continuous Compliance:** AWS Config enabled with automatic remediation on critical rules.
- [x] **13. Vulnerability Scanning:** Amazon Inspector continuously scanning EC2, ECR, and Lambda.
- [x] **14. SSM Session Manager:** Remote shell access executed via SSM with zero open Port 22 rules.
- [x] **15. Incident Runbooks:** Quarantine security groups, IAM revocation scripts, and snapshot playbooks tested.
