# 🚀 AWS Hands-On Projects & Production Engineering Master Handbook
> **Build Real-World Cloud Projects. Learn by Doing. Master 20 Practical AWS Labs.**  
> *20 Complete End-to-End Projects: EC2, S3, IAM, VPC, RDS, ALB, ASG, CloudWatch, SNS, Lambda, API Gateway, DynamoDB, S3 Event Workflows, CloudFront OAC, CloudFormation, EBS Backups, Outage Triage & Capstone Production Architecture.*

---

## 📑 Table of Contents
1. [Master Learning Roadmap & Project Architecture Matrix](#1-master-learning-roadmap--project-architecture-matrix)
2. [Project 1: Launch Your First EC2 Web Server (Apache on AL2023)](#project-1-launch-your-first-ec2-web-server)
3. [Project 2: Host a Static Website on Amazon S3](#project-2-host-a-static-website-on-amazon-s3)
4. [Project 3: Create a Secure IAM User (Least Privilege & MFA)](#project-3-create-a-secure-iam-user)
5. [Project 4: Build Your First Custom VPC from Scratch](#project-4-build-your-first-custom-vpc-from-scratch)
6. [Project 5: Deploy EC2 Inside Your Custom VPC](#project-5-deploy-ec2-inside-your-custom-vpc)
7. [Project 6: Create a Managed Relational Database with Amazon RDS](#project-6-create-a-managed-relational-database-with-amazon-rds)
8. [Project 7: Build a Secure Two-Tier Web Application (EC2 + RDS)](#project-7-build-a-secure-two-tier-web-application)
9. [Project 8: Create Your First Application Load Balancer (ALB)](#project-8-create-your-first-application-load-balancer)
10. [Project 9: Build an Auto Scaling Web Application (ASG + ALB)](#project-9-build-an-auto-scaling-web-application)
11. [Project 10: Monitor EC2 with CloudWatch Alarms & Metrics](#project-10-monitor-ec2-with-cloudwatch-alarms--metrics)
12. [Project 11: Create an Enterprise SNS Real-Time Notification System](#project-11-create-an-enterprise-sns-real-time-notification-system)
13. [Project 12: Build Your First Serverless Function with AWS Lambda](#project-12-build-your-first-serverless-function-with-aws-lambda)
14. [Project 13: Build a Serverless REST API (API Gateway + Lambda)](#project-13-build-a-serverless-rest-api)
15. [Project 14: Build a Serverless Database App (API GW + Lambda + DynamoDB)](#project-14-build-a-serverless-database-app)
16. [Project 15: Create an Event-Driven S3 File Processing Workflow](#project-15-create-an-event-driven-s3-file-processing-workflow)
17. [Project 16: Host a Private S3 Website Secured with CloudFront OAC](#project-16-host-a-private-s3-website-secured-with-cloudfront-oac)
18. [Project 17: Deploy Infrastructure as Code with AWS CloudFormation](#project-17-deploy-infrastructure-as-code-with-aws-cloudformation)
19. [Project 18: Build an Automated EBS Snapshot & Disaster Recovery Project](#project-18-build-an-automated-ebs-snapshot--disaster-recovery-project)
20. [Project 19: Troubleshoot a Broken AWS Application (6-Layer Diagnostic Matrix)](#project-19-troubleshoot-a-broken-aws-application)
21. [Project 20: Capstone Production Architecture (Multi-Tier, Multi-AZ, WAF & ASG)](#project-20-capstone-production-architecture)

---

## 1. Master Learning Roadmap & Project Architecture Matrix

```
                                 THE 4-STAGE AWS HANDS-ON ROADMAP
 ┌───────────────────────┬───────────────────────┬───────────────────────┬───────────────────────┐
 │ Stage 1: Core Infra   │ Stage 2: Resilient App│ Stage 3: Serverless   │ Stage 4: Enterprise   │
 │ (Projects 1 to 6)     │ (Projects 7 to 11)    │ (Projects 12 to 16)   │ (Projects 17 to 20)   │
 ├───────────────────────┼───────────────────────┼───────────────────────┼───────────────────────┤
 │ • EC2 Web Servers     │ • Two-Tier Web App    │ • Lambda Functions    │ • CloudFormation IaC  │
 │ • S3 Static Hosting   │ • ALB Load Balancers  │ • API Gateway REST    │ • EBS Disaster Recov. │
 │ • IAM Security & MFA  │ • Auto Scaling (ASG)  │ • DynamoDB CRUD       │ • 6-Layer Outage Fix  │
 │ • Custom VPCs & CIDR  │ • CloudWatch Alarms   │ • S3 Event Pipelines  │ • Production Capstone │
 │ • Amazon RDS Databases│ • SNS Alert Systems   │ • CloudFront OAC      │   Multi-Tier Blueprint│
 └───────────────────────┴───────────────────────┴───────────────────────┴───────────────────────┘
```

---

## Project 1: Launch Your First EC2 Web Server

```
                               PROJECT 1 ARCHITECTURE
  [ User Browser ] ──(HTTP Port 80)──> [ Security Group ] ──> [ EC2 Instance (Amazon Linux 2023) ]
                                                               └── Apache HTTPD Web Server
```

### Step-by-Step CLI Execution
```bash
# 1. SSH into the instance with restricted key permissions
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<EC2-PUBLIC-IP>

# 2. Update packages and install Apache HTTPD
sudo yum update -y
sudo yum install -y httpd

# 3. Create a custom index page and start service
echo "<h1>Welcome to My First AWS Web Server!</h1>" | sudo tee /var/www/html/index.html
sudo systemctl start httpd
sudo systemctl enable httpd

# 4. Verify web server status
sudo systemctl status httpd
```

---

## Project 2: Host a Static Website on Amazon S3

```
                               PROJECT 2 ARCHITECTURE
  [ Global Users ] ──(HTTP)──> [ S3 Bucket Website Endpoint ] ──> [ index.html / style.css ]
```

### S3 Public Read Bucket Policy (`bucket-policy.json`)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-unique-bucket-name/*"
    }
  ]
}
```

---

## Project 3: Create a Secure IAM User

```
                               PROJECT 3 IAM ARCHITECTURE
  [ Human User / Admin ] ──(IAM Auth + MFA Authenticator)──> [ IAM Group: Developers ]
                                                                     │
                                                                     ▼ (Least Privilege Policy)
                                                         [ AmazonS3ReadOnlyAccess ]
```

### Key Production Security Rules
1. **Never use the Root Account** for daily engineering tasks.
2. **Always enforce Virtual MFA** for all console users.
3. Assign permissions to **IAM Groups**, never directly to individual users.

---

## Project 4: Build Your First Custom VPC from Scratch

```
                               PROJECT 4 VPC TOPOLOGY
                             VPC CIDR: 10.0.0.0/16 (65,536 IPs)
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │  Internet Gateway (IGW) ──> Attached to VPC                                            │
 ├───────────────────────────────────────────┬────────────────────────────────────────────┤
 │  PUBLIC SUBNET (10.0.1.0/24 - AZ-A)       │  PRIVATE SUBNET (10.0.2.0/24 - AZ-B)       │
 │  ├── Route Table: 0.0.0.0/0 ──> IGW       │  ├── Route Table: Local VPC routing only   │
 │  └── Auto-assign Public IPv4 enabled      │  └── No direct Internet ingress            │
 └───────────────────────────────────────────┴────────────────────────────────────────────┘
```

---

## Project 5: Deploy EC2 Inside Your Custom VPC

```bash
# Verify route tables and connectivity from inside Public EC2
ping -c 4 8.8.8.8
curl -I https://aws.amazon.com

# If SSH connection fails, verify the 4-layer checklist:
# 1. Is EC2 in a Public Subnet?
# 2. Does the Subnet Route Table have 0.0.0.0/0 -> IGW?
# 3. Does the Security Group allow Port 22 from your current IP?
# 4. Does the Network ACL allow inbound/outbound ephemeral traffic?
```

---

## Project 6: Create a Managed Relational Database with Amazon RDS

```
                               PROJECT 6 DATABASE ISOLATION
  [ Public Subnet: EC2 Web Server ] 
                 │
                 ▼ (Private VPC Network - Port 3306)
  [ Private Subnet Group: Amazon RDS MySQL ] (No Public IP!)
```

---

## Project 7: Build a Secure Two-Tier Web Application

```
                           TWO-TIER APPLICATION SECURITY GROUPS
  [ Web Security Group ] ──────(Allow MySQL Port 3306)──────> [ DB Security Group ]
  • Inbound: Port 80 (0.0.0.0/0)                               • Inbound: Port 3306 (Source: sg-web)
  • Outbound: All Traffic                                      • Outbound: None (Isolated)
```

---

## Project 8: Create Your First Application Load Balancer (ALB)

```
                               PROJECT 8 ALB FLOW
                               [ Internet Users ]
                                       │
                                       ▼
                       [ Application Load Balancer ]
                                       │
                 ┌─────────────────────┴─────────────────────┐
                 ▼ (Round Robin)                             ▼
  [ EC2 Instance 1 (AZ-A) ]                   [ EC2 Instance 2 (AZ-B) ]
  Health Check: HTTP:80/                      Health Check: HTTP:80/
```

---

## Project 9: Build an Auto Scaling Web Application

```
                       PROJECT 9 AUTO SCALING LIFECYCLE
  [ ALB ] ──> [ Auto Scaling Group: Min=2, Desired=3, Max=6 ]
                    │
                    ├──> CPU > 60% ──> ASG Launches New Instances (Scale Out)
                    └──> CPU < 20% ──> ASG Terminates Excess Instances (Scale In)
```

---

## Project 10: Monitor EC2 with CloudWatch Alarms & Metrics

```bash
# Generate high CPU load on Linux EC2 to trigger CloudWatch Alarm
sudo yum install -y stress
stress --cpu 4 --timeout 300s
```

---

## Project 11: Create an Enterprise SNS Real-Time Notification System

```
                         EVENT-DRIVEN ALERTING PIPELINE
  [ CloudWatch Alarm (CPU > 80%) ] ──> [ Amazon SNS Topic ] ──> ├── Email Alerts (SRE Team)
                                                                 ├── SMS Notifications
                                                                 └── Lambda Auto-Remediation
```

---

## Project 12: Build Your First Serverless Function with AWS Lambda

```python
# Lambda Handler (Python 3.12)
import json

def lambda_handler(event, context):
    name = event.get('queryStringParameters', {}).get('name', 'Cloud Engineer')
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'message': f'Hello, {name}! Welcome to Serverless on AWS.'})
    }
```

---

## Project 13: Build a Serverless REST API

```
                               SERVERLESS API FLOW
  [ Client / Postman ] ──(HTTP GET /hello)──> [ API Gateway ] ──> [ AWS Lambda ] ──> [ Response JSON ]
```

---

## Project 14: Build a Serverless Database App (CRUD with DynamoDB)

```python
# Lambda CRUD Handler with Boto3
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('veriqta-tasks')

def lambda_handler(event, context):
    http_method = event.get('httpMethod', 'GET')
    
    if http_method == 'POST':
        body = json.loads(event['body'])
        table.put_item(Item={'taskId': body['taskId'], 'title': body['title'], 'status': 'Pending'})
        return {'statusCode': 201, 'body': json.dumps({'message': 'Task created successfully'})}
        
    elif http_method == 'GET':
        task_id = event['queryStringParameters']['taskId']
        response = table.get_item(Key={'taskId': task_id})
        return {'statusCode': 200, 'body': json.dumps(response.get('Item', {}))}
```

---

## Project 15: Create an Event-Driven S3 File Processing Workflow

```
                         S3 EVENT-DRIVEN PROCESSING
  [ User Uploads image.jpg ] ──> [ S3 Bucket ] ──(s3:ObjectCreated:*)──> [ Lambda Processor ]
                                                                                 │
                                                                                 ▼
                                                                     [ CloudWatch Logs / DB ]
```

---

## Project 16: Host a Private S3 Website Secured with CloudFront OAC

```
                          CLOUDFRONT ORIGIN ACCESS CONTROL (OAC)
  [ User ] ──(HTTPS)──> [ CloudFront CDN ] ──(SigV4 OAC)──> [ S3 Bucket (100% Private!) ]
```

### S3 Bucket Policy for CloudFront OAC
```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "AllowCloudFrontServicePrincipalReadOnly",
    "Effect": "Allow",
    "Principal": { "Service": "cloudfront.amazonaws.com" },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::veriqta-private-website/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EDFDVBD632BHDS5"
      }
    }
  }
}
```

---

## Project 17: Deploy Infrastructure as Code with AWS CloudFormation

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production S3 and EC2 Infrastructure'
Resources:
  ProductionBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub 'app-storage-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256

  WebServerInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0
      Tags:
        - Key: Environment
          Value: Production
```

---

## Project 18: Build an Automated EBS Snapshot & Disaster Recovery Project

```bash
# 1. Create a Snapshot of an EBS Volume
aws ec2 create-snapshot \
  --volume-id vol-0abcd1234efgh5678 \
  --description "Production Backup Snapshot"

# 2. Restore Snapshot to a new EBS Volume in AZ-B
aws ec2 create-volume \
  --snapshot-id snap-0abcd1234efgh5678 \
  --availability-zone us-east-1b \
  --volume-type gp3
```

---

## Project 19: Troubleshoot a Broken AWS Application (6-Layer Matrix)

```
                            THE 6-LAYER AWS TRIAGE PATH
  Layer 1: Symptom ────────> "ERR_CONNECTION_TIMED_OUT" or HTTP 502/504
  Layer 2: Security Groups > Verify Inbound Port 80/443 (Stateful rule check)
  Layer 3: Route Tables ───> Verify 0.0.0.0/0 -> igw-xxxx in Public Subnet
  Layer 4: ALB Health ─────> Verify Target Group Health Check returns HTTP 200
  Layer 5: Application Logs> Inspect /var/log/nginx/error.log or /var/log/httpd/
  Layer 6: Resolution ─────> Apply fix, verify HTTP 200 OK, update monitoring alarms
```

---

## Project 20: Capstone Production Architecture

```
                       ENTERPRISE PRODUCTION ARCHITECTURE
  [ Users ] ──> [ Route 53 ] ──> [ AWS WAF ] ──> [ Application Load Balancer ]
                                                        │
         ┌──────────────────────────────────────────────┴──────────────────────────────────────────────┐
         ▼                                                                                             ▼
  [ AZ-A Public Subnet: NAT GW ]                                                [ AZ-B Public Subnet: NAT GW ]
         │                                                                                             │
         ▼                                                                                             ▼
  [ AZ-A Private Subnet: EC2 ASG ]                                              [ AZ-B Private Subnet: EC2 ASG ]
  ├── Web Application Instance                                                  ├── Web Application Instance
  └── Attached IAM Role (S3/CloudWatch Access)                                  └── Attached IAM Role
         │                                                                                             │
         └──────────────────────────────────────────────┬──────────────────────────────────────────────┘
                                                        ▼
                                       [ Amazon RDS Multi-AZ Database ]
                                       [ Amazon S3 Static/Media Bucket]
```

### 10-Point Capstone Production Checklist
- [x] VPC configured with public, private, and data subnets across $\ge 2$ AZs.
- [x] Internet Gateway attached to public subnets; NAT Gateways attached for private outbound.
- [x] ALB deployed across public subnets with HTTPS TLS 1.2+ certificate.
- [x] EC2 instances isolated inside private subnets managed by Auto Scaling Group.
- [x] Security Group chaining enforced (ALB SG $\rightarrow$ EC2 SG $\rightarrow$ RDS SG).
- [x] Database deployed on Amazon RDS Multi-AZ with automated backups enabled.
- [x] S3 bucket protected with encryption, versioning, and Block Public Access.
- [x] CloudWatch CPU alarms, synthetic health checks, and SNS notifications configured.
- [x] EC2 IAM Roles used for all AWS API calls (zero static access keys).
- [x] Infrastructure defined and deployed via Terraform or CloudFormation.
