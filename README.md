# ☁️ AWS Multi-Tier Cloud Architecture

> Designed and deployed a production-grade, scalable multi-tier infrastructure on AWS — supporting analytics workloads with a 99.9% uptime SLA using EC2, VPC, IAM, ELB, and Auto Scaling.

[![AWS](https://img.shields.io/badge/Cloud-Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws)](https://aws.amazon.com)
[![Architecture](https://img.shields.io/badge/Pattern-Multi--Tier-232F3E?style=for-the-badge)]()
[![Uptime SLA](https://img.shields.io/badge/Uptime_SLA-99.9%25-28a745?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Deployed_&_Tested-28a745?style=for-the-badge)]()

---

## 🎯 Project Overview

This project demonstrates the design, configuration, and deployment of a **scalable, secure, highly-available multi-tier architecture on AWS** — the standard infrastructure pattern for production-grade web and analytics applications.

The architecture separates concerns across three tiers: presentation (web), application (logic), and data — each isolated in its own subnet, with traffic flowing only through defined, secured paths.

---

## 🏗️ Architecture Design

```
                        INTERNET
                            │
                    ┌───────▼────────┐
                    │  Route 53 DNS  │
                    └───────┬────────┘
                            │
                    ┌───────▼────────┐
                    │  Internet      │
                    │  Gateway (IGW) │
                    └───────┬────────┘
                            │
    ┌───────────────────────▼──────────────────────────┐
    │                   VPC (10.0.0.0/16)               │
    │                                                    │
    │   ┌────────────────────────────────────────────┐  │
    │   │         PUBLIC SUBNETS (2 AZs)             │  │
    │   │                                            │  │
    │   │  ┌──────────────────────────────────────┐  │  │
    │   │  │   Elastic Load Balancer (ELB)        │  │  │
    │   │  │   • Routes HTTPS traffic             │  │  │
    │   │  │   • Health checks every 30s          │  │  │
    │   │  │   • Terminates SSL                   │  │  │
    │   │  └─────────────┬────────────────────────┘  │  │
    │   │                │                            │  │
    │   │  ┌─────────────▼────────────────────────┐  │  │
    │   │  │   Web Tier (EC2 Auto Scaling Group)  │  │  │
    │   │  │   • t2.micro instances               │  │  │
    │   │  │   • Min: 2 · Max: 6 · Desired: 2    │  │  │
    │   │  │   • Scale out at 70% CPU             │  │  │
    │   │  │   • Scale in at 30% CPU              │  │  │
    │   │  └─────────────┬────────────────────────┘  │  │
    │   └────────────────┼───────────────────────────┘  │
    │                    │                               │
    │   ┌────────────────▼───────────────────────────┐  │
    │   │         PRIVATE SUBNETS (2 AZs)            │  │
    │   │                                            │  │
    │   │  ┌──────────────────────────────────────┐  │  │
    │   │  │   App Tier (EC2)                     │  │  │
    │   │  │   • Business logic processing        │  │  │
    │   │  │   • Isolated from internet           │  │  │
    │   │  │   • Communicates via internal SG     │  │  │
    │   │  └─────────────┬────────────────────────┘  │  │
    │   │                │                            │  │
    │   │  ┌─────────────▼────────────────────────┐  │  │
    │   │  │   Data Tier                          │  │  │
    │   │  │   • RDS (Multi-AZ, automated backup) │  │  │
    │   │  │   • S3 (analytics data lake)         │  │  │
    │   │  │   • No direct internet access        │  │  │
    │   │  └──────────────────────────────────────┘  │  │
    │   └────────────────────────────────────────────┘  │
    └────────────────────────────────────────────────────┘
                            │
                    ┌───────▼────────┐
                    │  CloudWatch    │
                    │  Monitoring +  │
                    │  Alarms        │
                    └────────────────┘
```

---

## 🛠️ AWS Services Used

| Service | Tier | Configuration | Purpose |
|---------|------|--------------|---------|
| **EC2** | Web + App | t2.micro, Amazon Linux 2 | Compute for web and application layers |
| **VPC** | Network | 10.0.0.0/16 CIDR | Isolated network environment |
| **Subnets** | Network | 4 subnets across 2 AZs | Public (web) + Private (app + data) isolation |
| **Internet Gateway** | Network | Attached to VPC | Inbound internet access to public subnets |
| **NAT Gateway** | Network | 1 per AZ | Private subnet outbound access (no inbound) |
| **Route Tables** | Network | 2 (public + private) | Traffic routing rules |
| **Security Groups** | Security | 3 SGs (web, app, db) | Stateful firewall rules per tier |
| **IAM** | Security | Roles + Policies | Least-privilege access for all services |
| **ELB (ALB)** | Compute | Application Load Balancer | HTTPS traffic distribution, health checks |
| **Auto Scaling** | Compute | Min 2, Max 6 | Scale based on CPU utilisation |
| **RDS** | Data | Multi-AZ deployment | Managed relational database with failover |
| **S3** | Data | Versioning enabled | Analytics data lake, static assets |
| **CloudWatch** | Monitoring | Metrics + Alarms | Uptime monitoring, threshold alerting |

---

## 🔐 Security Design

### Principle of Least Privilege (IAM)
```
IAM Roles Created:
  ├── EC2WebRole         → S3 read-only, CloudWatch logs write
  ├── EC2AppRole         → RDS connect, S3 read-write, SSM access
  ├── RDSMonitoringRole  → CloudWatch enhanced monitoring
  └── LambdaRole         → (if applicable) specific service access

IAM Policies:
  → No wildcard (*) actions used
  → Resource-level restrictions on all policies
  → MFA required for all console access
```

### Security Group Rules (Defence in Depth)
```
Web-Tier-SG:
  Inbound:  HTTPS (443) from 0.0.0.0/0
            HTTP (80) from 0.0.0.0/0 (redirect to 443)
  Outbound: All traffic to App-Tier-SG only

App-Tier-SG:
  Inbound:  Port 8080 from Web-Tier-SG ONLY (no internet)
  Outbound: Port 3306 to DB-Tier-SG (MySQL)
            Port 443 to internet (via NAT, for API calls)

DB-Tier-SG:
  Inbound:  Port 3306 from App-Tier-SG ONLY
  Outbound: None
```

### Network Isolation
- Public subnets: web tier + ELB (internet-facing)
- Private subnets: app tier + data tier (no internet route)
- NAT Gateway: allows private instances to initiate outbound (updates, APIs) without being reachable from internet

---

## ⚡ High Availability Design

### Multi-AZ Deployment
Every component that can fail has a parallel in a second Availability Zone:
- ELB spans both AZs automatically
- Auto Scaling maintains instances in both AZs
- RDS has a synchronous standby replica in AZ-2 (automatic failover < 60s)
- NAT Gateways deployed in both AZs (if one AZ fails, private instances in the other still have outbound)

### Auto Scaling Policy
```
Scale-Out Trigger:
  CPU Utilisation > 70% for 2 consecutive 5-min periods
  → Add 2 instances
  → Cooldown: 300 seconds

Scale-In Trigger:
  CPU Utilisation < 30% for 10 consecutive 5-min periods
  → Remove 1 instance
  → Cooldown: 600 seconds (aggressive scale-in protection)

Health Check:
  ELB health check every 30 seconds
  Unhealthy threshold: 2 consecutive failures
  → Instance replaced automatically
```

### Uptime SLA Achievement: 99.9%
The 99.9% SLA comes from the combination of:
- Multi-AZ = survives single AZ outage (AWS SLA: 99.99% per AZ)
- Auto Scaling = survives single instance failure (automatic replacement)
- RDS Multi-AZ = survives database node failure (< 60s failover)
- ELB health checks = routes away from unhealthy instances in < 60s

---

## 📁 Repository Contents

```
aws-multi-tier-architecture/
├── README.md                          ← You are here
├── architecture/
│   ├── architecture-diagram.png       ← Visual AWS diagram
│   └── architecture-decisions.md      ← Why each service was chosen
├── infrastructure/
│   ├── vpc-setup.md                   ← VPC + subnet configuration steps
│   ├── ec2-setup.md                   ← EC2 launch + AMI configuration
│   ├── elb-autoscaling.md             ← Load balancer + ASG setup
│   ├── rds-setup.md                   ← Database configuration
│   └── iam-policies/                  ← IAM policy JSON files
│       ├── ec2-web-role.json
│       ├── ec2-app-role.json
│       └── rds-monitoring-role.json
├── monitoring/
│   ├── cloudwatch-dashboards.md       ← Dashboard setup guide
│   └── alarm-configurations.md        ← All alarm thresholds documented
└── testing/
    ├── load-test-results.md           ← Auto Scaling validation results
    └── failover-test-results.md       ← Multi-AZ failover test log
```

---

## 🧪 Testing & Validation

### Auto Scaling Test
Simulated CPU load using `stress` tool on EC2 instances:
- Ramped CPU to 85% across existing instances
- Auto Scaling triggered within 2 minutes
- 2 new instances launched and passed health checks
- Total capacity restored in **< 5 minutes**

### Failover Test (RDS Multi-AZ)
Forced primary RDS instance failure via AWS console reboot with failover:
- Standby promoted to primary
- Application reconnected after connection retry logic
- Total downtime: **< 45 seconds** (within SLA)

### Load Balancer Health Check
Manually stopped application service on one EC2:
- ELB detected unhealthy in < 30 seconds (2 failed checks × 15s interval)
- Traffic rerouted to healthy instances
- Auto Scaling replaced the unhealthy instance within 4 minutes

---

## 💡 Key Learnings

**1. Security Groups > NACLs for application-layer security**
NACLs are stateless and apply at the subnet level. Security Groups are stateful and apply at the instance level. For fine-grained tier-to-tier access control, Security Groups are the right tool. I learned this the hard way when NACL rules blocked RDS return traffic I forgot to allow (stateless = need explicit inbound AND outbound rules).

**2. NAT Gateway placement matters for cost + availability**
A single NAT Gateway saves cost but creates a single point of failure: if that AZ goes down, all private instances in other AZs lose outbound. For production, one NAT Gateway per AZ is the correct pattern.

**3. Auto Scaling cooldown periods need tuning**
A 60-second cooldown was too short — the system would trigger another scale-out event before newly launched instances had time to pick up load. Extending cooldown to 300 seconds stabilised the scaling behaviour and prevented over-provisioning thrash.

---

## 📬 Author

**BN Pooja** — Product Manager × Cloud & DevOps Practitioner

[![Portfolio](https://img.shields.io/badge/Portfolio-bnpooja12.github.io-c8410a?style=flat-square)](https://bnpooja12.github.io/portfolio/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-BN_Pooja-0077B5?style=flat-square&logo=linkedin)](https://linkedin.com/in/b-n-pooja-13279b280/)
[![Email](https://img.shields.io/badge/Email-bnpooja12%40gmail.com-D14836?style=flat-square&logo=gmail)](mailto:bnpooja12@gmail.com)
