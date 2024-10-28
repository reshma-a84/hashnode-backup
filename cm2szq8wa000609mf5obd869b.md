---
title: "Security Group with My IP"
seoTitle: "security group with my IP"
datePublished: Mon Oct 28 2024 12:23:58 GMT+0000 (Coordinated Universal Time)
cuid: cm2szq8wa000609mf5obd869b
slug: security-group-with-my-ip
tags: terraform-aws-infrastructureascode-provisioning-automation-cloudcomputing

---

Security is of paramount importance. How many times have we seen or heard that we should **“NEVER”** allow complete internet access to our security group??

So now you are doing AWS Hands-on and would like to create and attach a security group to your ec2 instance where you can login only from **YOUR IP**. How is that achieved?

Its simple now

Begin with writing a Bash script to read your current IP and lets call it "my-ip.sh”

```bash
#!/bin/bash

# Fetch the public IP address using an external service
ip_address=$(curl -s https://api.ipify.org)

# Output the public IP address
echo "{\"ip\": \"${ip_address}\"}"
```

Don’t forget to make it an executable file

```bash
$ chmod +x my-ip.sh
```

Next lets write simple Terraform code where we create a VPC and add Security Groups with one ingress rule of type SSH and the source to be my IP.

```bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0.0"
    }
  }
}

provider "aws" {
  region  = var.region
  profile = var.profile
}

provider "local" {}
```

Let’s add variables too

```bash
variable "region" {
  type    = string
  default = "us-east-1"
}

variable "profile" {
  type    = string
  default = "default"
}
```

Now adding the data file

```bash
data "aws_caller_identity" "current" {}

data "aws_region" "current" {}

data "aws_vpc" "get_vpc_id" {
  filter {
    name   = "tag:Name"
    values = ["ANW-VPC"]
  }
  depends_on = [aws_vpc.hands_on_VPC]
}

data "external" "my_ip" {
  program = ["bash", "${path.module}/my-ip.sh"]
}
```

Now creating a very simple VPC

```bash
resource "aws_vpc" "hands_on_VPC" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "ANW-VPC"
  }
}
```

Time to create Security Groups of both default and a custom one with my ip as one ingress rule

```bash
resource "aws_default_security_group" "default" {
  vpc_id = data.aws_vpc.get_vpc_id.id
  ingress {
    protocol  = -1
    self      = true
    from_port = 0
    to_port   = 0
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "default"
  }
}

resource "aws_security_group" "sg-ssh" {
  vpc_id = data.aws_vpc.get_vpc_id.id
  name   = "new"
  tags = {
    Name = "new"
  }
}

resource "aws_vpc_security_group_ingress_rule" "ssh" {
  ip_protocol       = "tcp"
  to_port           = 22
  from_port         = 22
  cidr_ipv4         = "${data.external.my_ip.result["ip"]}/32"
  security_group_id = aws_security_group.sg-ssh.id
}
```

Since the IP requires a CIDR notation, we use

```bash
"${data.external.my_ip.result["ip"]}/32"
```

And thats the end of it……. Try it and let me know