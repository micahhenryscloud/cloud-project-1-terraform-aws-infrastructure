# Troubleshooting Notes

This document records real issues encountered while building the AWS three-tier infrastructure project, how they were diagnosed, and how they were fixed.

## Issue 1: Terraform reported “No configuration files”

### Problem

Terraform returned:

```text
Error: No configuration files
```

### Investigation

The Terraform command was being run from the wrong folder.

Terraform only reads `.tf` files from the current working directory.

### Root Cause

The terminal was not inside the `terraform` directory containing `main.tf`.

### Fix

I navigated into the correct folder:

```powershell
cd C:\Users\Gillian\aws-high-availability-web-architecture\terraform
```

Then reran:

```powershell
terraform plan
```

### Lesson Learned

Terraform commands must be run from the directory containing the Terraform configuration files.

---

## Issue 2: Terraform block definition/newline error

### Problem

Terraform returned an error similar to:

```text
A block definition must end with a newline.
```

### Investigation

The Terraform file had two blocks joined together without a newline.

Example:

```hcl
}resource "aws_vpc" "main" {
```

### Root Cause

Terraform resources were pasted too closely together, causing invalid HCL syntax.

### Fix

I separated the blocks with a blank line:

```hcl
}

resource "aws_vpc" "main" {
```

### Lesson Learned

Terraform configuration is sensitive to block structure. Each resource or provider block should be clearly separated.

---

## Issue 3: Duplicate provider configuration

### Problem

Terraform reported a duplicate AWS provider configuration.

### Investigation

The `main.tf` file contained more than one default AWS provider block.

### Root Cause

A second provider block had been pasted into the file accidentally.

### Fix

I removed the duplicate provider block and kept a single AWS provider configuration:

```hcl
provider "aws" {
  region = "eu-west-2"
}
```

### Lesson Learned

Terraform should only have one default provider configuration unless aliases are intentionally being used.

---

## Issue 4: AWS credentials failed during Terraform plan

### Problem

Terraform failed while validating AWS provider credentials.

The error mentioned AWS STS and invalid credentials.

### Investigation

I checked AWS CLI authentication using:

```powershell
aws sts get-caller-identity
```

### Root Cause

The AWS CLI credentials had been entered incorrectly or were malformed.

### Fix

I reran:

```powershell
aws configure
```

Then verified the configured identity with:

```powershell
aws sts get-caller-identity
```

### Lesson Learned

Before running Terraform against AWS, AWS CLI credentials should be verified with STS.

---

## Issue 5: ALB target group showed instances as unhealthy

### Problem

The Application Load Balancer target group showed the EC2 instances as unhealthy.

### Investigation

I checked the EC2 system logs and found that the user-data script had failed.

The log showed errors like:

```text
yum: command not found
httpd.service not found
```

### Root Cause

The EC2 instances were running Ubuntu, but the user-data script was written for Amazon Linux.

The script used:

```bash
yum install -y httpd
systemctl start httpd
```

Ubuntu uses `apt` and `apache2`, not `yum` and `httpd`.

### Fix

I updated the user-data script to use Ubuntu-compatible commands:

```bash
apt update -y
apt install -y apache2
systemctl start apache2
systemctl enable apache2
```

### Lesson Learned

User-data scripts must match the operating system of the EC2 AMI.

---

## Issue 6: Session Manager could not connect to private EC2 instances

### Problem

AWS Session Manager could not connect to the EC2 instances.

### Investigation

The Session Manager page showed that the instance did not have a valid IAM instance profile attached.

### Root Cause

The EC2 instances did not have an IAM role with Systems Manager permissions.

### Fix

I added:

- An IAM role for EC2
- The `AmazonSSMManagedInstanceCore` policy
- An IAM instance profile
- The instance profile reference in the launch template

After replacing the Auto Scaling Group instances, Session Manager worked.

### Lesson Learned

Private EC2 instances can be accessed securely with Session Manager, but they need the correct IAM role and SSM permissions.

---

## Issue 7: CloudWatch alarm did not trigger

### Problem

A CloudWatch CPU alarm did not trigger even when CPU load was generated.

### Investigation

The alarm was checked against the current EC2 instances in the Auto Scaling Group.

The alarm was monitoring an old instance ID that had already been replaced.

### Root Cause

The alarm was created for a specific EC2 instance instead of the Auto Scaling Group.

Because Auto Scaling replaces instances, instance-level alarms can become stale.

### Fix

I created a new alarm using the Auto Scaling Group CPU metric instead of an individual EC2 instance metric.

### Lesson Learned

For Auto Scaling environments, alarms should usually monitor the Auto Scaling Group rather than individual disposable instances.

---

## Issue 8: CloudWatch alarm stayed OK during CPU test

### Problem

The CloudWatch alarm stayed in the OK state even after CPU load was generated.

### Investigation

The CPU graph showed activity, but the value did not exceed the alarm threshold.

### Root Cause

The threshold was set to 80%, but the test generated a lower average CPU value across the Auto Scaling Group.

### Fix

I temporarily lowered the alarm threshold to 10% to test the monitoring pipeline.

After confirming the SNS email notification worked, the threshold could be restored to a production-appropriate value.

### Lesson Learned

Monitoring should be tested deliberately. Alert thresholds must be realistic for the metric and test conditions.

---

## Issue 9: RDS creation took several minutes

### Problem

Terraform appeared to hang while creating the RDS instance.

### Investigation

Terraform repeatedly showed:

```text
aws_db_instance.mysql: Still creating...
```

### Root Cause

RDS creation, especially with Multi-AZ enabled, takes several minutes because AWS provisions the database, standby instance, networking, storage and endpoint.

### Fix

I waited for the creation process to complete.

### Lesson Learned

Some AWS managed services take longer to provision than EC2 or networking resources. RDS creation can take 5–15 minutes or more.

---

## Summary

The main troubleshooting themes in this project were:

- Running Terraform from the correct directory
- Fixing Terraform syntax issues
- Verifying AWS credentials
- Matching user-data scripts to the operating system
- Understanding ALB health checks
- Using IAM roles for Session Manager
- Creating reliable CloudWatch alarms for Auto Scaling environments
- Understanding managed service provisioning times

These issues helped turn the project from a simple build exercise into a realistic cloud troubleshooting experience.