# AWS Infrastructure Migration Proposal

## Region

**Frankfurt (eu-central-1)**

---

# 1. Objective

This proposal outlines the migration of the existing AWS Lightsail infrastructure to a redesigned AWS EC2-based architecture in Frankfurt.
The goal is to improve:

* Security
* Operational visibility
* Reliability
* Performance capacity
* Controlled cost structure

---

# 2. Current Environment

* 2 Lightsail Ubuntu instances (publicly exposed)
* 4 Lightsail MySQL databases
* Limited monitoring and security controls
* No centralized traffic management

---

# 3. Target Architecture Overview

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/cde28e5f-09fe-4694-95ae-85784d9a9e6b" />


## Networking

* 1 VPC in `eu-central-1`
* Public Subnet:

  * Application Load Balancer (ALB)
  * AWS WAF (attached to ALB)
* Private Application Subnet:

  * Main Application Server
  * Standby Application Server (same subnet for simplicity)
* Private Security + Database Subnet:

  * SOC / EDR Server
  * RDS Subnet Group (4 databases)

No NAT Gateway (cost optimization).

---

# 4. Compute Design

## Main Application Server

* Instance Type: **m6a.xlarge**
* 4 vCPU / 16 GB RAM
* Handles primary workload

## Standby Application Server

* Instance Type: **m6a.large**
* 2 vCPU / 8 GB RAM
* Same subnet as main server
* Used if the main server fails

## SOC / EDR Server

* Instance Type: **m6a.large**
* 2 vCPU / 8 GB RAM
* Dedicated for monitoring and endpoint security tooling

---

# 5. Database Layer

* **4 × Amazon RDS MySQL**
* MySQL 8.x
* Single-AZ deployment
* Automated backups enabled
* Only accessible from Application Server Security Group
* SOC/EDR server does NOT connect to databases

Only application servers have database access.

---

# 6. Security Architecture

## AWS WAF

* Single Web ACL attached to ALB
* Protects against:

  * SQL Injection
  * XSS
  * Malicious IPs
  * Abuse patterns

## Security Groups

* ALB: Allow HTTPS (443) from internet
* App Servers: Allow traffic only from ALB
* RDS: Allow MySQL (3306) only from App Server SG
* SOC/EDR: Restricted internal access

No public IPs assigned to EC2 instances.

---

# 7. Monitoring & Automated Diagnostics

## CloudWatch Monitoring

CloudWatch monitors:

* EC2 CPU usage
* EC2 Memory usage (via CloudWatch Agent)
* Disk usage
* ALB health metrics
* RDS performance metrics

---

## Automated High Memory / CPU Detection

To detect high resource consumption:

CloudWatch monitors total memory usage →
If memory exceeds threshold (e.g., 85%) →
CloudWatch Alarm triggers →
Alarm invokes AWS Systems Manager (SSM) →
SSM runs:

ps aux --sort=-%mem | head -n 10

on the EC2 instance →
Output is stored in CloudWatch Logs for investigation.

This enables automated identification of problematic processes without SSH access.

---

# 8. Secure Access to Instances

## Option 1: AWS Systems Manager (SSM) — Recommended

No SSH. No VPN. No port 22 open.

Engineer connects via AWS Console or CLI →
SSM creates secure session →
Direct shell access to private EC2.

✔ No public exposure
✔ No SSH key management
✔ Fully audited
✔ Lower operational complexity

---

## Option 2: AWS Client VPN (If full network access required)

Laptop
↓ (TLS encrypted)
AWS Client VPN Endpoint
↓
VPC
↓
Private EC2 (port 22 allowed from VPN CIDR only)

Security Group Rule:
Allow SSH (22) from VPN CIDR (e.g., 10.200.0.0/22)

---

# 9. Migration Approach

## EC2 Migration Options

Option 1: Lift-and-shift

* Export Lightsail snapshot
* Convert to AMI
* Launch EC2 instance

Option 2: Rebuild on fresh EC2

* Deploy clean instances
* Install dependencies
* Deploy application
* More controlled and secure long-term

Final method to be discussed with application team.

---

## Database Migration

* Lightsail MySQL → Amazon RDS MySQL
* Use AWS DMS (minimal downtime)
* Or mysqldump (simple approach)
* Maintain MySQL version compatibility

---

# 10. Cost Estimation (Frankfurt – On-Demand Pricing)

Based on eu-central-1 On-Demand pricing (730 hours/month):

## EC2

m6a.xlarge = $0.207/hour
m6a.large = $0.1035/hour

Main Server:
0.207 × 730 = **$151.11**

Standby Server:
0.1035 × 730 = **$75.55**

SOC / EDR Server:
0.1035 × 730 = **$75.55**

EC2 Total:
**$302.22/month**

---

## EBS (gp3)

gp3 Frankfurt: $0.0952 per GB/month

Assuming ~320 GB total:
≈ **$30.46/month**

---

## RDS MySQL (Single-AZ)

Using db.t3.medium baseline (~$0.084/hour):

0.084 × 730 = $61.32 per DB

4 databases:
≈ **$245.28/month**

(RDS storage & backups not included — depends on allocated size.)

---

## AWS WAF

Example configuration:

* 1 Web ACL
* ~10 rules
* ~10M requests/month

≈ **$20–25/month**

---

## Application Load Balancer

ALB pricing depends on LCU usage and traffic.
Estimated low-traffic range:

≈ **$25–35/month**

---

# 11. Estimated Total Monthly Cost

EC2: $302.22
EBS: $30.46
RDS: $245.28
WAF: ~$22
ALB: ~$30

Estimated Total:

≈ **$630 – $670 per month**

(Final cost depends on RDS storage size, backup retention, and actual traffic volume.)

---

# 12. Benefits

* No public EC2 exposure
* Centralized WAF protection
* Automated diagnostics via CloudWatch + SSM
* Secure audited access
* Clear separation of application and security roles
* Controlled and predictable cost
* Increased compute capacity compared to Lightsail

---

# Conclusion

This architecture upgrades the current Lightsail deployment into a secure, production-grade AWS design in Frankfurt.
It improves security posture, monitoring visibility, and operational control while maintaining cost efficiency and meeting the application owner's performance requirements.


