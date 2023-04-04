---
title: "Adding multiple emails to sns subscriptions using Terraform"
datePublished: Tue Apr 04 2023 10:45:57 GMT+0000 (Coordinated Universal Time)
cuid: clg24x32o000v09lbe2in00pv
slug: adding-multiple-emails-to-sns-subscriptions-using-terraform

---

We seldom find advanced topics on Terraform templates online and sometimes the information is not fully available on the Official Pages as well.  
I'm bringing to you advanced template examples to be used in your organization's environment.

Bringing the first in the series is adding multiple email ID subscriptions to our SNS Topic.

While you all know the Terraform Basics, I would not go into much detail on them. I have structured the code in a parameterized manner and would be following the same pattern in all my posts.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680603716634/294be43b-b630-4f79-86ce-5e73508154cd.png align="center")

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

variable "sns-topic" {}
variable "sns-topic-subscription" {}
```

```go
# locals.tf
locals {
  emails = ["emailid1@email.com","emailid2@email.com","emailid3@email.com"]
}
```

```go
# sns.tf
resource "aws_sns_topic" "sns_topic" {
  name = var.sns-topic
}

resource "aws_sns_topic_subscription" "sns_topic_subscription" {
  count = length(local.emails)
  topic_arn = aws_sns_topic.sns_topic.arn
  protocol  = "email"
  endpoint = local.emails[count.index]
}
```

```go
# sns.tfvars
sns-topic              = "sns-test-topic"
sns-topic-subscription = "sns-test-subscription"

profile = "aws-terraform"
region  = "us-east-1"
```

The same can be achieved without the use of locals.tf as well. Then we use the "for\_each" functionality and pass the values to it and then refer to the "value" in the endpoint section.

```go
resource "aws_sns_topic_subscription" "sns_topic_subscription" {
  for_each = toset(["emailid1@email.com","emailid2@email.com","emailid3@email.com"])
  topic_arn = aws_sns_topic.sns_topic.arn
  protocol  = "email"
  endpoint = each.value
}
```

And as you already know the drill... Run the Terraform template

```plaintext
$ terraform init
$ terraform plan
$ terraform apply -var-file sns.tfvars --auto-approve
$ terraform destroy -var-file sns.tfvars --auto-approve
```

> Prerequisite:
> 
> 1. Ensure AWS CLI is installed and configured. Installation: [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
>     
>     Configuration: (I have configured using "Short-term credentials" format) [https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
>     
> 2. Ensure Terraform is installed
>     
>     [https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
>