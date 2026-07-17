# AWS Infrastructure

This document describes the AWS architecture this project was designed and deployed against.

## Architecture Diagram (textual)

```
                          Internet
                             |
                      [Internet Gateway]
                             |
                    ---------------------
                    |   Public Subnets   |
                    |  (2 AZs, for HA)   |
                    ---------------------
                             |
                 [Application Load Balancer]
                             |
                    ---------------------
                    |  Private Subnets   |
                    |  (2 AZs, for HA)   |
                    ---------------------
                       |             |
                [EC2 - Auto      [EC2 - Auto
                 Scaling Group]   Scaling Group]
                       |             |
                       -------+-------
                              |
                      [Amazon RDS - MySQL]
                        (Multi-AZ, private subnet)

     [S3 Bucket - Static Frontend Hosting] -----> served independently via S3 static website hosting
```

## Components

| Component | Purpose |
|---|---|
| **VPC** | Custom VPC (10.0.0.0/16) with 2 public + 2 private subnets across two Availability Zones |
| **Internet Gateway** | Provides internet access to the public subnets |
| **Public Subnets** | Host the Application Load Balancer and NAT Gateway |
| **Private Subnets** | Host EC2 instances (backend) and the RDS database — not directly internet-accessible |
| **Application Load Balancer (ALB)** | Distributes incoming traffic across EC2 instances in the Auto Scaling Group |
| **Auto Scaling Group (ASG)** | Maintains 2+ EC2 instances, scales out on high CPU/traffic, replaces unhealthy instances |
| **Amazon RDS (MySQL)** | Managed relational database, deployed in private subnets, not publicly accessible |
| **Security Groups** | ALB SG allows inbound HTTP/HTTPS from internet; EC2 SG allows inbound only from ALB SG; RDS SG allows inbound only from EC2 SG |
| **S3** | Hosts the static frontend (`index.html`) via S3 static website hosting |

## Security Group Rules (summary)

- **ALB-SG**: Inbound 80/443 from `0.0.0.0/0`
- **EC2-SG**: Inbound 8080 from `ALB-SG` only
- **RDS-SG**: Inbound 3306 from `EC2-SG` only

This ensures the database and application servers are never directly reachable from the public internet — all traffic must pass through the load balancer first.

## Deployment Steps

1. Create custom VPC with public/private subnets across 2 AZs
2. Launch RDS MySQL instance in private subnet
3. Build and containerize the Spring Boot backend (`Dockerfile`)
4. Launch EC2 instances (or an Auto Scaling launch template) running the backend container, injecting `RDS_ENDPOINT`, `RDS_USERNAME`, `RDS_PASSWORD` as environment variables
5. Create Target Group + Application Load Balancer pointing to the EC2 instances
6. Configure Auto Scaling Group with min 2 / max 4 instances, scaling policy based on CPU utilization
7. Upload `frontend/index.html` to an S3 bucket, enable static website hosting
8. Update `API_BASE` in `frontend/index.html` to point to the ALB's DNS name

## Environment Variables (backend)

| Variable | Description |
|---|---|
| `RDS_ENDPOINT` | RDS instance endpoint hostname |
| `RDS_USERNAME` | Database username |
| `RDS_PASSWORD` | Database password |

These are injected at runtime (EC2 user-data / launch template) and are never committed to source control.
