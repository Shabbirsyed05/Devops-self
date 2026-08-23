# 🌐 AWS Networking & VPC Architecture Production Master Handbook
> **From VPC Fundamentals to Enterprise Hybrid & Multi-Account Network Architecture.**  
> *20-Page Complete Curriculum: CIDR Math, 3-Tier Multi-AZ VPCs, Route Tables, NAT Gateways, SG Chaining, NACLs, Transit Gateway, PrivateLink, Direct Connect, Inspection VPCs & Network Triage Playbooks.*

---

## 📑 Table of Contents
1. [The AWS Global Network Model & Core Networking Foundations](#1-the-aws-global-network-model--core-networking-foundations)
2. [IPv4, IPv6 & Enterprise CIDR Planning (RFC 1918 & Subnet Math)](#2-ipv4-ipv6--enterprise-cidr-planning)
3. [Amazon VPC Architecture (Default vs. Custom & DNS Resolutions)](#3-amazon-vpc-architecture)
4. [3-Tier Multi-AZ Subnet Architecture (Public, Private App & Isolated DB)](#4-3-tier-multi-az-subnet-architecture)
5. [Route Tables, Longest Prefix Matching & Route Priority Engine](#5-route-tables-longest-prefix-matching--route-priority-engine)
6. [Internet Gateways (IGW), Public IP Allocation & EIPs](#6-internet-gateways-igw-public-ip-allocation--eips)
7. [NAT Gateways & Private Subnet Internet Egress (Multi-AZ & Cost Levers)](#7-nat-gateways--private-subnet-internet-egress)
8. [Security Groups (Stateful Filtering & Multi-Tier SG Chaining)](#8-security-groups)
9. [Network ACLs (NACLs - Stateless Subnet Filtering & Ephemeral Ports)](#9-network-acls-nacls)
10. [Amazon Route 53 & Hybrid DNS Architecture (Inbound/Outbound Endpoints)](#10-amazon-route-53--hybrid-dns-architecture)
11. [Elastic Load Balancing: ALB, NLB & Gateway Load Balancer (GWLB)](#11-elastic-load-balancing-alb-nlb--gateway-load-balancer-gwlb)
12. [VPC Peering Architecture (Non-Transitive Routing & Peering Limits)](#12-vpc-peering-architecture)
13. [AWS Transit Gateway (TGW - Hub-and-Spoke & Multi-Account Segmentation)](#13-aws-transit-gateway-tgw)
14. [VPC Endpoints & AWS PrivateLink (Gateway vs. Interface Endpoints)](#14-vpc-endpoints--aws-privatelink)
15. [Hybrid Cloud Networking (Direct Connect, Site-to-Site VPN & Redundancy)](#15-hybrid-cloud-networking)
16. [Enterprise Network Security Architecture (Inspection VPC & Network Firewall)](#16-enterprise-network-security-architecture)
17. [Network Observability (VPC Flow Logs, Traffic Mirroring & Reachability Analyzer)](#17-network-observability)
18. [AWS Network Troubleshooting Matrix (8 Production Incident Playbooks)](#18-aws-network-troubleshooting-matrix)
19. [Complete Production Multi-AZ Enterprise Network Architecture Blueprint](#19-complete-production-multi-az-enterprise-network-architecture-blueprint)
20. [10-Point Enterprise Production Network Checklist & Decision Matrix](#20-10-point-enterprise-production-network-checklist--decision-matrix)

---

## 1. The AWS Global Network Model & Core Networking Foundations

```
                           AWS GLOBAL NETWORK TOPOLOGY
  [ Global Backbone (100 GbE Private Fiber) ]
         │
         ├──> [ AWS Region (e.g. us-east-1) ]
         │    ├── [ AZ 1 (us-east-1a) ] ── (Low-latency < 1ms synchronous link)
         │    ├── [ AZ 2 (us-east-1b) ]
         │    └── [ AZ 3 (us-east-1c) ]
         │
         └──> [ 600+ Edge Locations / PoPs ] ──> CloudFront CDN & Global Accelerator
```

### End-to-End Packet Ingress Flow
```
  User Request ──> Route 53 DNS ──> CloudFront Edge ──> Internet Gateway ──> ALB ──> EC2 Private Subnet
```

---

## 2. IPv4, IPv6 & Enterprise CIDR Planning

```
                           RFC 1918 PRIVATE IP RANGES
 ┌──────────────────────┬─────────────────────────┬──────────────────────────────┬───────────────────────┐
 │ Class / Range        │ Subnet Mask             │ Total IP Addresses           │ Best Use Case         │
 ├──────────────────────┼─────────────────────────┼──────────────────────────────┼───────────────────────┤
 │ `10.0.0.0/8`         │ `255.0.0.0`             │ 16,777,216 IPs               │ Large Enterprise Orgs │
 │ `172.16.0.0/12`      │ `255.240.0.0`           │ 1,048,576 IPs                │ Medium Environments   │
 │ `192.168.0.0/16`     │ `255.255.0.0`           │ 65,536 IPs                   │ Small Labs / Branch   │
 └──────────────────────┴─────────────────────────┴──────────────────────────────┴───────────────────────┘
```

### AWS 5 Reserved IP Addresses per Subnet
In any AWS subnet (e.g. `10.0.1.0/24` with 256 total IPs), exactly **5 IPs are reserved**:
* `10.0.1.0` : Network Address.
* `10.0.1.1` : VPC Router.
* `10.0.1.2` : AWS DNS (AmazonProvidedDNS / Route 53 Resolver).
* `10.0.1.3` : Reserved by AWS for future use.
* `10.0.1.255`: Network Broadcast Address.
* **Usable IPs per `/24` Subnet = 251**.

---

## 3. Amazon VPC Architecture

```
                        DEFAULT VPC VS. PRODUCTION CUSTOM VPC
  Default VPC (172.31.0.0/16)  ──> All subnets are Public; IGW attached; NOT isolated.
  Custom VPC (10.0.0.0/16)     ──> Dedicated Public, Private App & Isolated DB subnets.
```

### VPC DNS Resolution Engine
* `enableDnsSupport = true`: Enables the internal Amazon DNS server (`169.254.169.253` or base VPC `.2`).
* `enableDnsHostnames = true`: Assigns public DNS hostnames to EC2 instances with public IPs.

---

## 4. 3-Tier Multi-AZ Subnet Architecture

```
                             ENTERPRISE 3-TIER VPC TOPOLOGY
                                  VPC: 10.0.0.0/16
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │ TIER 1: PUBLIC INGRESS SUBNETS (ALB, Bastion, NAT Gateways)                            │
 │ ├── AZ-A: 10.0.1.0/24        ├── AZ-B: 10.0.2.0/24        ├── AZ-C: 10.0.3.0/24        │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │ TIER 2: PRIVATE APPLICATION SUBNETS (EC2 ASG, ECS Tasks, EKS Pods)                     │
 │ ├── AZ-A: 10.0.10.0/24       ├── AZ-B: 10.0.11.0/24       ├── AZ-C: 10.0.12.0/24       │
 ├────────────────────────────────────────────────────────────────────────────────────────┤
 │ TIER 3: ISOLATED DATABASE SUBNETS (Amazon Aurora Multi-AZ, ElastiCache Redis)          │
 │ ├── AZ-A: 10.0.20.0/24       ├── AZ-B: 10.0.21.0/24       ├── AZ-C: 10.0.22.0/24       │
 └────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Route Tables, Longest Prefix Matching & Route Priority

```
                          LONGEST PREFIX MATCHING ENGINE
  Route Table Destination      Target
  ┌──────────────────────────┬───────────────────────────┐
  │ 10.0.1.0/24 (Most Specific) eni-0abc123 (Custom ENI) │
  │ 10.0.0.0/16 (VPC Local)    local                     │
  │ 0.0.0.0/0   (Default)      igw-0xyz987 (Internet)    │
  └──────────────────────────┴───────────────────────────┘
  * Packet sent to 10.0.1.55 matches 10.0.1.0/24 and routes to eni-0abc123!
```

---

## 6. Internet Gateways (IGW), Public IP Allocation & EIPs

* **Internet Gateway (IGW):** Fully managed, horizontally scalable VPC component that performs 1-to-1 NAT translation between private IP addresses and public IPv4/Elastic IPs.
* **Elastic IP (EIP):** Static public IPv4 address that remains allocated to your AWS account until explicitly released; can be re-associated instantly during instance failover.

---

## 7. NAT Gateways & Private Subnet Internet Egress

```
                           MULTI-AZ NAT GATEWAY ARCHITECTURE
  [ Public Subnet AZ-A: NAT GW 1 (EIP) ]     [ Public Subnet AZ-B: NAT GW 2 (EIP) ]
                 ▲                                          ▲
                 │ (Outbound Only)                          │ (Outbound Only)
  [ Private Subnet AZ-A: EC2 App ]           [ Private Subnet AZ-B: EC2 App ]
```

### Cost Optimization Rule of Thumb
* **Multi-AZ NAT:** Deploy 1 NAT Gateway per AZ in production to prevent single-AZ outage failures and eliminate cross-AZ data processing fees ($0.01/GB).
* **VPC Endpoints:** Route Amazon S3 and DynamoDB traffic through **Gateway Endpoints** (100% free) to avoid heavy NAT processing costs ($0.045/GB).

---

## 8. Security Groups

```
                        3-TIER SECURITY GROUP CHAINING
  [ Internet ] ──(Port 80/443)──> [ sg-alb (Public ALB) ]
                                          │
                               (Port 8080 - Source: sg-alb)
                                          ▼
                                [ sg-app (Private EC2) ]
                                          │
                               (Port 3306 - Source: sg-app)
                                          ▼
                                [ sg-db (Isolated RDS) ]
```

---

## 9. Network ACLs (NACLs)

```
 ┌──────────────────────┬────────────────────────────────────┬────────────────────────────────────┐
 │ Feature              │ Security Group                     │ Network ACL (NACL)                 │
 ├──────────────────────┼────────────────────────────────────┼────────────────────────────────────┤
 │ **Statefulness**     │ **Stateful** (Return allowed)      │ **Stateless** (Must allow both ways)│
 │ **Operating Boundary**│ Instance / ENI Level              │ Subnet Boundary Level              │
 │ **Rule Processing**  │ Evaluates all rules                │ Evaluates in numerical order (1-100│
 │ **Default Action**   │ Default Deny Inbound / Allow Out   │ Default Allow All (Custom: Deny All│
 │ **Ephemeral Ports**  │ Not required (Handled by state)    │ **Mandatory** (Allow 1024-65535)   │
 └──────────────────────┴────────────────────────────────────┴────────────────────────────────────┘
```

---

## 10. Amazon Route 53 & Hybrid DNS Architecture

```
                          HYBRID ROUTE 53 RESOLVER FLOW
  [ On-Premises DNS Server ] ──(Inbound Endpoint: 10.0.1.10)──> [ AWS Private Hosted Zone ]
  [ AWS EC2 Instance ] ───────(Outbound Endpoint / Rules)────> [ On-Premises Active Directory ]
```

---

## 11. Elastic Load Balancing: ALB, NLB & GWLB

```
 ┌──────────────────────┬─────────────────────────┬─────────────────────────┬────────────────────────────┐
 │ Feature              │ Application (ALB)       │ Network (NLB)           │ Gateway (GWLB)             │
 ├──────────────────────┼─────────────────────────┼─────────────────────────┼────────────────────────────┤
 │ **OSI Layer**        │ Layer 7 (HTTP/HTTPS/gRPC│ Layer 4 (TCP/UDP/TLS)   │ Layer 3 (IP Packets)       │
 │ **Target Types**     │ Instance, IP, Lambda    │ Instance, IP, ALB       │ 3rd-Party Virtual Firewall │
 │ **Static EIP**       │ No                      │ Yes (Static EIP per AZ) │ No                         │
 │ **Client IP Preserv**│ In `X-Forwarded-For`    │ Native IP preserved     │ GENEVE Protocol (Port 6081)│
 └──────────────────────┴─────────────────────────┴─────────────────────────┴────────────────────────────┘
```

---

## 12. VPC Peering Architecture

```
                       VPC PEERING NON-TRANSITIVE ROUTING
  [ VPC A (10.0.0.0/16) ] <──(Peering Active)──> [ VPC B (10.1.0.0/16) ] <──(Peering Active)──> [ VPC C (10.2.0.0/16) ]
  
  ❌ VPC A CANNOT communicate with VPC C through VPC B! (Must create direct Peering A <-> C or use TGW)
```

---

## 13. AWS Transit Gateway (TGW)

```
                            TRANSIT GATEWAY HUB-AND-SPOKE
                               [ AWS Transit Gateway ]
                                          │
         ┌──────────────────┬─────────────┴────────────┬──────────────────┐
         ▼                  ▼                          ▼                  ▼
  [ Production VPC ]  [ Staging VPC ]           [ Shared Services ]  [ Direct Connect DX ]
  (Route Table 1)     (Route Table 2)           (Route Table 3)      (On-Premises Hub)
```

---

## 14. VPC Endpoints & AWS PrivateLink

```
                       GATEWAY VS. INTERFACE ENDPOINTS
  Gateway Endpoints (Free)       ──> Amazon S3 & Amazon DynamoDB (Configured in Route Table).
  Interface Endpoints (PrivateLink) ──> ENI in Subnet; private IP; AWS KMS, SSM, ECR, CloudWatch.
```

---

## 15. Hybrid Cloud Networking

```
 ┌──────────────────────┬─────────────────────────┬──────────────────────┬───────────────────────────────┐
 │ Feature              │ AWS Site-to-Site VPN    │ AWS Direct Connect   │ Direct Connect + VPN Backup   │
 ├──────────────────────┼─────────────────────────┼──────────────────────┼───────────────────────────────┤
 │ **Medium**           │ Public Internet (IPsec) │ Dedicated Fiber      │ Dedicated Fiber + IPsec Fail  │
 │ **Bandwidth**        │ Up to 1.25 Gbps/tunnel  │ 1 Gbps to 100 Gbps   │ Enterprise High-Availability  │
 │ **Latency**          │ Variable                │ Ultra-Low & Predict. │ Automated BGP Failover        │
 │ **Provisioning**     │ Minutes                 │ Weeks / Months       │ Production Enterprise Std     │
 └──────────────────────┴─────────────────────────┴──────────────────────┴───────────────────────────────┘
```

---

## 16. Enterprise Network Security Architecture

```
                            INSPECTION VPC ARCHITECTURE
  Internet Ingress ──> [ AWS Network Firewall / Suricata IDS/IPS ] ──> [ Transit Gateway ] ──> [ App VPCs ]
```

---

## 17. Network Observability

* **VPC Flow Logs:** Captures IP traffic flowing through ENIs (`srcaddr`, `dstaddr`, `srcport`, `dstport`, `protocol`, `action: ACCEPT/REJECT`).
* **Reachability Analyzer:** Mathematically validates path configuration between source and destination ENIs/Gateways without sending live packets.
* **Traffic Mirroring:** Copies raw L3–L7 packet payloads from ENIs to an analyzer instance (Wireshark/Zeek) for deep security forensics.

---

## 18. AWS Network Troubleshooting Matrix

| Issue Scenario | What to Check First | Key CLI Command | Fix Action Item |
|:---:|:---|:---|:---|
| **1. EC2 Cannot Reach Internet** | Subnet Route Table & NAT/IGW | `aws ec2 describe-route-tables` | Add `0.0.0.0/0 -> IGW` (Public) or `-> NAT GW` (Private). |
| **2. Private EC2 Cannot Reach S3**| Gateway Endpoint Route Table | `aws ec2 describe-vpc-endpoints` | Attach S3 Gateway Endpoint to private subnet route table. |
| **3. ALB Returns 502/503** | Target Group Health & App SG | `aws elbv2 describe-target-health`| Verify EC2 SG allows traffic from ALB SG; fix `/healthz`. |
| **4. VPCs Cannot Ping** | Peering Route Table & CIDR overlap| `aws ec2 describe-vpc-peering-connections`| Add remote CIDR to route tables; check NACLs. |
| **5. DNS Query Fails** | Route 53 Resolver & VPC attributes| `aws ec2 describe-vpc-attribute` | Enable `enableDnsSupport` and `enableDnsHostnames`. |
| **6. Asymmetric NACL Drop** | Inbound/Outbound Ephemeral ports | `aws ec2 describe-network-acls` | Add outbound allow for TCP `1024-65535` to NACL. |
| **7. Blackhole Route Error** | Deleted ENI/NAT still in Route Table| `aws ec2 describe-route-tables` | Remove inactive target route or update to valid Gateway. |
| **8. NAT Gateway Down** | Elastic IP & Public Subnet location | `aws ec2 describe-nat-gateways` | Ensure NAT GW is in Public Subnet with active IGW route. |

---

## 19. Complete Production Multi-AZ Enterprise Network Architecture Blueprint

```
                      ENTERPRISE MULTI-AZ NETWORK BLUEPRINT
  [ Global Users ] ──> [ Route 53 DNS ] ──> [ CloudFront + WAF ] ──> [ Internet Gateway ]
                                                                             │
         ┌───────────────────────────────────────────────────────────────────┴───────────────────────────────────┐
         ▼                                                                                                       ▼
  [ AZ-A Public Subnet (10.0.1.0/24) ]                                                    [ AZ-B Public Subnet (10.0.2.0/24) ]
  ├── ALB Endpoint                                                                        ├── ALB Endpoint
  └── NAT Gateway AZ-A (EIP)                                                              └── NAT Gateway AZ-B (EIP)
         │                                                                                                       │
         ▼                                                                                                       ▼
  [ AZ-A Private App Subnet (10.0.10.0/24) ]                                              [ AZ-B Private App Subnet (10.0.11.0/24) ]
  ├── EC2 / EKS Application Cluster                                                       ├── EC2 / EKS Application Cluster
  └── Attached Interface Endpoints (KMS, ECR, SSM)                                        └── Attached Interface Endpoints
         │                                                                                                       │
         └─────────────────────────────────────────────────┬─────────────────────────────────────────────────────┘
                                                           ▼
                                         [ AZ-A & AZ-B Isolated Database Subnets ]
                                         ├── Amazon Aurora PostgreSQL Multi-AZ (10.0.20.0/24)
                                         └── Amazon ElastiCache Redis Cluster (10.0.21.0/24)
```

---

## 20. 10-Point Enterprise Production Network Checklist

- [x] **1. CIDR Sizing:** `/16` VPC CIDR allocated with zero overlap across corporate on-prem or peer VPCs.
- [x] **2. Multi-AZ Subnetting:** Public, Private App, and Isolated DB subnets spread across $\ge 3$ AZs.
- [x] **3. Redundant NAT Gateways:** 1 NAT Gateway deployed per AZ to eliminate single-AZ blast radius.
- [x] **4. Isolated Database Tier:** RDS/Aurora placed in private subnets with no IGW route or public IPs.
- [x] **5. Security Group Chaining:** Explicit SG-to-SG references enforced across ALB $\rightarrow$ App $\rightarrow$ DB tiers.
- [x] **6. Free Gateway Endpoints:** S3 and DynamoDB Gateway Endpoints active to eliminate NAT data fees.
- [x] **7. Hybrid Transit Gateway:** Centralized TGW deployed with route table segmentation for Prod/Dev.
- [x] **8. Hybrid DNS Resolution:** Route 53 Resolver Inbound and Outbound endpoints active.
- [x] **9. Full Observability:** VPC Flow Logs enabled on all VPCs and streamed to CloudWatch/S3.
- [x] **10. Tested Disaster Recovery:** Direct Connect with automated Site-to-Site VPN BGP failover verified.
