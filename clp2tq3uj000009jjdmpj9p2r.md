---
title: "EC2 instance - without "hardcoding" AMI"
datePublished: Fri Nov 17 2023 16:17:43 GMT+0000 (Coordinated Universal Time)
cuid: clp2tq3uj000009jjdmpj9p2r
slug: ec2-instance-without-hardcoding-ami

---

What can be the greatest nightmare for any DevOps consultant?

Ans : Ever changing AMI id of EC2 instances and the constant need to change them in the Terraform template.

There is a fix for this too.

All that needs to be done is get the "latest" AMI dynamically into your code base.

```go
data "aws_ami" "amazon-linux-2" {
  most_recent = true

  filter {
    name   = "owner-alias"
    values = ["amazon"]
  }

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64*"]
  }
}

resource "aws_instance" "amzn2_linux_instance" {
  ami           = data.aws_ami.amazon-linux-2.id
  instance_type = "t2.micro"
}
```

Data Source is used to query and select the latest AMI. Let me explain the parameters here

* most\_recent - Returns the most recent AMI, if there are more than one result
    
* filter1 - name is "owner-alias" is "amazon". As Amazon is the owner of Amazon2 Linux owner
    
* filter2 - "name" is the variable for the Amazon AMI name.
    

Pass these filtered values to "aws\_instance" resource and you will have created your ec2 instance without having to hard code the ami.

```plaintext
Note: The AMI will be pulled from the default region 
configured using aws configure or by explicitly 
mentioning them in your providers.tf file
```

You can find more of them at [https://www.hashicorp.com/blog/hashicorp-terraform-supports-amazon-linux-2](https://www.hashicorp.com/blog/hashicorp-terraform-supports-amazon-linux-2)

I will be following this by ways to dynamically bring other Operating Systems AMI.