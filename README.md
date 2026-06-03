# AWS Three-Tier Infrastructure with Terraform

## Overview

This project demonstrates a highly available AWS infrastructure deployed using Terraform. It includes a custom VPC, public and private subnets, an Application Load Balancer, Auto Scaling EC2 instances, and an RDS MySQL database deployed in private database subnets.

## Architecture

```text
Internet
   ↓
Application Load Balancer
   ↓
Auto Scaling Group
   ↓
EC2 App Servers
   ↓
RDS MySQL Database
```

## Services Used

- AWS VPC
- Public and private subnets
- Internet Gateway
- NAT Gateway
- Route tables
- Security groups
- EC2
- Launch Templates
- Auto Scaling Group
- Application Load Balancer
- Target Groups
- RDS MySQL
- CloudWatch
- SNS
- IAM
- S3 remote Terraform state
- DynamoDB state locking

## What This Project Demonstrates

This project shows the ability to design and deploy a production-style AWS network and compute architecture using Infrastructure as Code.

Key capabilities demonstrated include:

- Building a custom VPC across multiple Availability Zones
- Separating public, application, and database layers
- Deploying EC2 instances into private subnets
- Exposing traffic through an Application Load Balancer
- Using Auto Scaling Groups for self-healing infrastructure
- Deploying RDS MySQL in private database subnets
- Restricting database access using security groups
- Using CloudWatch and SNS for monitoring and alerting
- Managing infrastructure with Terraform
- Storing Terraform state remotely in S3
- Using DynamoDB for state locking

## Terraform Structure

```text
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── .terraform.lock.hcl
```

## Key Learning Outcomes

Through this project, I gained hands-on experience with:

- AWS networking fundamentals
- High availability design
- Infrastructure as Code
- Load balancing
- Auto Scaling
- Database subnet isolation
- Cloud monitoring
- Remote Terraform state management

## Validation

The architecture was tested by terminating EC2 instances in the Auto Scaling Group and confirming that replacement instances were launched automatically while the application remained available through the load balancer.

The RDS database was also accessed from an EC2 application server to confirm private database connectivity.