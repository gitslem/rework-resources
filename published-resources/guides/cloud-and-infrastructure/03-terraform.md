# Terraform for Beginners: Infrastructure as Code in Practice

## Overview
Terraform lets you define cloud infrastructure as code—reproducible, version-controlled, collaborative.

**Infrastructure as Code (IaC):** Define servers, databases, networks in code files

---

## Part 1: Terraform Basics

### Core Concepts
- **Providers**: AWS, GCP, Azure, etc.
- **Resources**: EC2 instances, S3 buckets, databases
- **State**: Current infrastructure state
- **Plan**: Preview changes before applying

### Basic Syntax
```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket"
}
```

---

## Part 2: Workflow

### 1. Write
Define infrastructure in .tf files

### 2. Plan
```bash
terraform plan  # Preview changes
```

### 3. Apply
```bash
terraform apply  # Deploy to cloud
```

### 4. Manage
```bash
terraform state list  # View resources
terraform destroy     # Delete all
```

---

## Part 3: Best Practices

### Organization
- One module per component
- Separate environments (dev/prod)
- Use variables for customization
- Version control all files

### State Management
- Store state remotely (S3, Terraform Cloud)
- Never commit state to Git
- Enable locking to prevent conflicts
- Backup state regularly

### Security
- Use secrets management (AWS Secrets)
- Restrict state file access
- Audit infrastructure changes
- Use IAM roles, not keys

---

## Summary

**Terraform Advantages:**
- Code-based infrastructure
- Version control
- Team collaboration
- Reproducible deployments
- Easy destruction/recreation

**Workflow:**
Write Code → Plan → Apply → Monitor

**Start:**
1. Install Terraform
2. Define first resource
3. Plan and apply
4. Destroy and recreate

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
