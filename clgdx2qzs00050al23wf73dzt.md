---
title: "Target Resource Creation"
datePublished: Wed Apr 12 2023 16:39:39 GMT+0000 (Coordinated Universal Time)
cuid: clgdx2qzs00050al23wf73dzt
slug: target-resource-creation
tags: aws, terraform, iac

---

You are a Platform Engineer and are responsible to maintain the Cloud Infrastructure with many limitations, with one being not to destroy the current infrastructure. What would you do if you have been asked to modify or create a new resource **BUT** without creating a new terraform template???  
The answer is **TARGET** resource creation.

All you have to do is add the resource code and use the format mentioned in the sections below to run plan/apply or destroy.

```go
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "4.60.0"
    }
  }
}

provider "aws" {
  region  = var.region
  profile = var.profile
}
```

```go
# variables.tf
variable "region" {}
variable "profile" {}

variable "ec2-role-name" {}
```

```go
# iam.tf
resource "aws_iam_role" "ec2-iam-role" {
    name = "my-ec2-iam-role"
    assume_role_policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
            {
                Action = "sts:AssumeRole"
                Effect = "Allow"
                Sid = ""
                Principal = {
                    Service = "ec2.amazonaws.com"
                }
            }
        ]
    })  
}
```

```go
# target_iam.tfvars
profile = "aws-terraform"
region  = "us-east-1"

ec2-role-name = "my-ec2-iam-role"
```

Syntax:

```powershell
$ terraform apply -var-file <file>.tfvars -target="<AWS_Terraform_Resource_name>.<Resource>" --auto-approve
```

Example:

```powershell
$ terraform apply -var-file target_iam.tfvars -target="aws_iam_role.ec2-iam-role" --auto-approve
```

Any resource can be selectively destroyed **provided** it does not have any dependencies or the related dependencies are also deleted.

```powershell
$ terraform destroy -var-file target_iam.tfvars -target="aws_iam_role.ec2-iam-role" --auto-approve
```

> **NOTE:** It is *not recommended* to use `-target` for routine operations, since this can lead to undetected configuration drift and confusion about how the true state of resources relates to configuration.
> 
> Ref: [https://developer.hashicorp.com/terraform/cli/commands/plan](https://developer.hashicorp.com/terraform/cli/commands/plan)