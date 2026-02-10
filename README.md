# AWS Infrastructure Migration Proposal

## Target Region
**US East (N. Virginia – `us-east-1`)**

---

## Objective
Migrating the current AWS Lightsail infrastructure to a standard AWS EC2-based architecture in order to improve **security, availability, and scalability**, while keeping the overall solution **cost-effective and simple to operate**.

---

## Current State
- 2 AWS Lightsail Ubuntu instances with public IP addresses
- 4 AWS Lightsail MySQL databases (production and development)
- Application servers are directly exposed to the internet
- Limited scalability and limited security controls

---

## Target Architecture (After Migration)

### Overview
- Single **VPC** in `us-east-1`
- **Public Subnets**
  - Application Load Balancers (one per application)
- **Private Subnets**
  - EC2 Auto Scaling Groups (no public IPs)
- **Private Database Subnets**
  - Amazon RDS MySQL (Single-AZ)
- One **AWS WAF** shared across both Application Load Balancers
- No NAT Gateway (cost-optimized design)

### Architecture Diagram
> Add the architecture image here

![Uploading ChatGPT Image Feb 10, 2026, 06_51_21 AM.png…]()

---

## Compute Layer
- Two applications
- Each application runs in an **Auto Scaling Group**
  - Minimum of **2 EC2 instances** for high availability
  - Instance type: **t3.large (2 vCPU, 8 GB RAM)**
- EC2 instances run only in **private subnets**

---

## Database Layer
- 4 × **Amazon RDS MySQL** databases
- MySQL version 8.x
- **Single-AZ** deployment (cost-optimized)
- Automated backups enabled
- Databases accessible only from application servers

---

## Application Server Migration Options

### Option 1: Lift-and-Shift (Lightsail → EC2)
- Export existing Lightsail instance snapshots
- Convert snapshots to AMIs
- Launch EC2 instances from the AMIs

**Pros**
- Faster initial migration
- Minimal application changes

**Cons**
- Legacy configurations are preserved
- Less flexibility for long-term scaling and improvements

---

### Option 2: Rebuild on EC2
- Deploy applications on clean EC2 instances
- Install dependencies and redeploy application code
- Configure Launch Templates and Auto Scaling Groups

**Pros**
- Cleaner and more secure environment
- Better long-term maintainability
- Easier scaling and operations

**Cons**
- Requires application redeployment and testing

**Note:**  
The final migration approach should be discussed with the application team based on application complexity, deployment readiness, and acceptable risk.

---

## Database Migration
- Databases will be migrated from Lightsail MySQL to Amazon RDS MySQL using **AWS Database Migration Service (DMS)**
- Full initial data load with ongoing replication
- Minimal downtime during cutover
- Same MySQL major version maintained for compatibility

---

## Security Improvements
- No public EC2 instances
- Centralized protection using AWS WAF
- Strict security group rules between ALB, EC2, and RDS
- Reduced attack surface compared to the current setup

---

## Benefits
- Improved security posture
- High availability at the application layer
- Easy horizontal scaling
- Better operational control compared to Lightsail
- Cost-optimized design using a single region and Single-AZ databases

---

## Estimated Monthly Cost (us-east-1)

| Service | Approximate Cost |
|------|----------------|
| EC2 (4 × t3.large) | ~$240 |
| EBS Storage | ~$40–50 |
| RDS MySQL (4 × Single-AZ) | ~$320–$400 |
| Application Load Balancers (2) | ~$30–$50 |
| AWS WAF (1) | ~$5–$10 |
| **Estimated Total** | **~$635–$750 / month** |

> Costs are based on on-demand pricing in `us-east-1` and may vary depending on usage.

---

## Conclusion
This migration replaces the existing Lightsail-based infrastructure with a **secure, scalable, and AWS best-practice architecture**, while keeping operational complexity and cost under control. The proposed setup supports current workloads and allows future growth without significant redesign.


