---
title: "Random Strings as Parameters"
seoTitle: "Random String in Terraform"
datePublished: Wed Jun 07 2023 09:54:58 GMT+0000 (Coordinated Universal Time)
cuid: clilja0zz000509mld1fzglfr
slug: random-strings-as-parameters
tags: aws, terraform, infrastructure-as-code, hashicorp

---

The most dreaded word that scares the Developer community "Hard Coding Parameters".

Well, in the words of ChatGPT this is why hard coding considered a bad practice

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686125819242/663e0ea4-a23d-43f8-964c-97b51290695e.png align="center")

*Hard coding is considered a bad practice in software development for several reasons:*

1. *Lack of Flexibility: Hard coding refers to embedding specific values or configurations directly into the code, without using variables or configurable options. This approach makes it difficult to modify or adapt the code to different scenarios or requirements. Any changes to the hardcoded values require modifying the source code itself, which can be time-consuming and error-prone. It limits the code's flexibility and makes it less maintainable.*
    
2. *Code Duplication: Hard coding often leads to code duplication, where the same values or configurations are repeated multiple times in different parts of the codebase. This violates the DRY (Don't Repeat Yourself) principle, which promotes reusability and maintainability. Code duplication increases the chances of inconsistencies and bugs, as any modifications or updates must be made in multiple places.*
    
3. *Reduced Readability: Hardcoded values are typically represented as literal values directly in the code, which can make the code more difficult to read and understand. When the purpose or meaning of a particular value is not evident from the code itself, it requires additional effort to comprehend the code's behavior. Using variables or constants with meaningful names improves code readability and makes it easier for developers to understand the intent.*
    
4. *Maintenance Challenges: Hardcoding values makes code maintenance more challenging. When a change is needed, such as updating a configuration or modifying a value, it often requires modifying the code and recompiling the application. This process can be cumbersome, especially in larger projects or when the code is distributed across multiple systems. It is preferable to have configurations stored separately, allowing changes to be made without modifying the source code.*
    
5. *Testing and Debugging Issues: Hardcoding values can introduce difficulties in testing and debugging. For testing purposes, it is often necessary to simulate different scenarios or inputs. Hardcoded values can make it harder to vary inputs or configure the system for different test cases. Additionally, if an error occurs related to a hardcoded value, identifying and fixing the problem becomes more complicated.*
    
6. *Lack of Scalability: Hardcoded values are typically tied to specific contexts or environments. When the code needs to be scaled to handle larger data sets, increased load, or different configurations, hardcoding becomes a significant obstacle. Flexible and configurable code is essential for building scalable applications that can adapt to changing requirements.*
    

*Overall, hard coding limits flexibility, reusability, readability, maintainability, and scalability of the codebase. It is considered a bad practice because it makes code more rigid, error-prone, and difficult to maintain and evolve over time. Instead, it is recommended to use variables, constants, configuration files, or other techniques to decouple values from the code and make it more adaptable and maintainable.*

So what do we do? Do we have a solution for this in Terraform?

The answer is "YES".

Instead of passing hard-coded AWS Resource names, there is a way to generate a random string and then pass on the same to the variables.  

Using ["random provider"](https://registry.terraform.io/providers/hashicorp/random/latest/docs)

As a person who loves their pets, I am drawn towards using the provider "**random\_pet**"

\[Please add your own provider section for this\]

1. provider
    
2. Generate Random parameter: The variable "length" here is - The length (in words) of the pet name. In the below example "length = 2" is the 2 complete words separated by "\_".
    

```go
# ******************************************************
#                    Random Generation
# ******************************************************
resource "random_pet" "name" {
  length  = 2
}

locals {
  prefix = "random-${random_pet.name.id}"
}
```

1. S3 bucket
    

```go
# ******************************************************
#                    S3 Bucket
# ******************************************************

resource "aws_s3_bucket" "demo-s3" {
  bucket = "${local.prefix}-s3"
}
```

1. Lamda function with IAM Role
    
2. ```go
    # ******************************************************
    #                    Lambda functions
    # ******************************************************
    
    data "archive_file" "lambda_zip" {
      type        = "zip"
      source_dir  = "lambda"
      output_path = "lambda.zip"
    }
    
    resource "aws_lambda_function" "lambda" {
      filename         = data.archive_file.lambda_zip.output_path
      function_name    = "${local.prefix}-lambda"
      role             = aws_iam_role.iam_for_lambda.arn
      handler          = "lambda.lambda_handler"
      source_code_hash = data.archive_file.lambda_zip.output_base64sha256
      runtime          = "python3.9"
      timeout          = 900
    }
    
    # ******************************************************
    #                    IAM Role - Lambda
    # ******************************************************
    
    data "aws_iam_policy_document" "assume_role" {
      statement {
        effect = "Allow"
    
        principals {
          type        = "Service"
          identifiers = ["lambda.amazonaws.com"]
        }
    
        actions = ["sts:AssumeRole"]
      }
    }
    
    resource "aws_iam_role" "iam_for_lambda" {
      name               = "${local.prefix}-iam-lambda"
      assume_role_policy = data.aws_iam_policy_document.assume_role.json
    }
    ```
    
    5\. Now lets see before hand what would my "pet" name before we see the name in the AWS Resources.
    

```go
# ***************************************************
#                    Output
# ***************************************************

output "s3_bucket" {
  value = aws_s3_bucket.demo-s3.bucket
}
```

Well the pet name is

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686131006225/a0b8b606-5525-44bb-b1fa-43fbcadd7984.png align="center")

Let's check them out in the AWS Console

S3 Bucket

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686131078368/3a63a349-6d54-4819-8437-686c78edef7c.png align="center")

IAM Role

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686131131591/575347e7-1464-4358-ad26-4caf48b155cf.png align="center")

Lambda function

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686131203226/da7a635a-469f-41ee-a567-4e0b6fd59f4b.png align="center")

As you can see that the pet name "chief-hound" (with length = 2 ) has been added to the names of S3, IAM and Lambda Resources.

The word "random" can be switched with the name that is needed in the project. For eg. Application name.