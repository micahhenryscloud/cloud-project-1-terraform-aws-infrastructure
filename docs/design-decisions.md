# Design Decisions

This document explains the key design choices made in the AWS three-tier infrastructure project and the reasoning behind them.

## 1. Custom VPC Instead of Default VPC

I used a custom VPC instead of the default VPC so that the network design could be controlled properly.

This allowed me to define:

- The CIDR range
- Public subnets
- Private application subnets
- Private database subnets
- Route tables
- Internet access
- NAT access
- Security boundaries between layers

This reflects how production AWS environments are usually designed, rather than relying on the default networking setup.

## 2. Public Subnets for the Application Load Balancer

The Application Load Balancer was placed in public subnets because it needs to receive traffic from the internet.

The ALB acts as the public entry point into the architecture.

The design is:

    Internet
       ↓
    Public Subnets
       ↓
    Application Load Balancer

The EC2 instances themselves are not publicly exposed. Only the load balancer receives direct public traffic.

## 3. Private Subnets for EC2 Application Servers

The EC2 application servers were placed in private subnets to reduce direct exposure to the internet.

This means users cannot directly reach the EC2 instances. Instead, requests must pass through the Application Load Balancer first.

This improves security because the application servers only need to accept traffic from the ALB security group, not from the entire internet.

## 4. Private Database Subnets for RDS

The RDS MySQL database was placed in private database subnets.

The database is not publicly accessible. It can only be reached from the application server security group.

This follows the principle of least privilege:

    Internet
       ↓
    ALB
       ↓
    EC2 App Servers
       ↓
    RDS Database

The database layer is isolated from public traffic.

## 5. NAT Gateway for Private Subnet Outbound Access

The private EC2 instances still need outbound internet access for updates and package installation.

However, they should not be directly reachable from the internet.

A NAT Gateway solves this by allowing outbound traffic from private subnets while blocking inbound public access.

This allows private instances to reach the internet without receiving direct internet traffic.

## 6. Auto Scaling Group for Self-Healing

The EC2 instances were managed by an Auto Scaling Group.

This was used so the infrastructure could automatically replace unhealthy or terminated instances.

During testing, I terminated an instance and confirmed that the Auto Scaling Group launched a replacement instance.

This demonstrates self-healing infrastructure.

## 7. Application Load Balancer for High Availability

The Application Load Balancer distributes traffic across multiple EC2 instances.

This improves availability because traffic can continue flowing even if one instance becomes unavailable.

The ALB also performs health checks and only sends traffic to healthy targets.

## 8. RDS Multi-AZ for Database Resilience

RDS was deployed with Multi-AZ enabled to improve database availability.

Multi-AZ provides a standby database in another Availability Zone. This helps protect against Availability Zone failure or underlying infrastructure issues.

## 9. Security Groups for Layered Access Control

Security groups were used to control traffic between each layer.

The design used:

- ALB security group allowing HTTP traffic from the internet
- Application server security group allowing HTTP only from the ALB
- Database security group allowing MySQL only from the application servers

This creates a layered security model.

## 10. CloudWatch and SNS for Monitoring

CloudWatch alarms and SNS notifications were used to monitor infrastructure health.

This allows infrastructure issues to generate alerts instead of relying on manual checking.

An alarm was tested by lowering the CPU threshold and confirming that an SNS email notification was received.

## 11. S3 Remote State for Terraform

Terraform state was moved from the local machine to an S3 bucket.

This is important because Terraform state tracks the infrastructure Terraform manages.

Remote state makes the setup more realistic and safer than relying only on a local `terraform.tfstate` file.

## 12. DynamoDB State Locking

DynamoDB state locking was added to prevent multiple Terraform runs from modifying the same infrastructure at the same time.

This is important in team environments where more than one engineer may run Terraform.

## Summary

The architecture was designed around four main principles:

- Security through private subnets and restricted security groups
- High availability through multiple Availability Zones, ALB and Auto Scaling
- Resilience through RDS Multi-AZ and self-healing EC2 instances
- Automation through Terraform, remote state and state locking