---
title: 'Using S3 bucket as Terraform Remote Backend'
date: 2019-03-16
permalink: /posts/2019/03/terraform-s3-backend/
tags:
  - iaac
  - terraform
  - s3
---

If you are familiar with IaC then you must have heard about terraform, right? Terraform is an awesome tool letting you to create, destroy, provision and configure your infrastructure as code. When terraform is instructed to run against some of its configuration then it creates a state file in the developer machine. This state file contains infrastructure configuration as applied by Terraform. But if you want to work on the terraform codebase as a team allowing terraform to create state file locally then every member will be in a great dilemma. Its because the other developers would be also requiring access to this file. Well you might think that we can use git, right? But one problem with that is some sensitive fields such as credentials might get exposed and this is not what you want. One solution is to use terraform S3 backend for storing infrastructure state in S3 bucket.

## Requirements

- AWS CLI Access
- Terraform
- Make (build tool)

## Procedure

At first, export AWS access key id and secret key to environment variables which will be used by Terraform.

```bash
$ export AWS_ACCESS_KEY_ID="<access-key>"
$ export AWS_SECRET_ACCESS_KEY="<secret-key>"
```

Since S3 is our choice for using remote backend so create an S3 bucket for storing terraform remote state file.

```bash
$ aws s3api create-bucket --bucket webapp-terraform-state --region us-east-1
```

Next, clone this repository and get into specific source code repository.

```bash
$ git clone https://github.com/shudarshon/challenge-terraform.git
$ cd challenge-terraform/6-tf-s3-backend/vpc
```

From the source code we can see that there are 4 different AWS resources terraform configuration categorized into specific folder by resource name. Lets find out how a terraform remote backend configuration (vpc/backend.tf) looks like:

```hcl
terraform {
  backend "s3" {
    bucket  = "webapp-terraform-state"
    key     = "vpc/vpc.tfstate"
    region  = "us-east-1"
    encrypt = true
  }
}
```

In above configuration we have specified the bucket name along with the S3 key. We need to specify the region of S3 bucket also. Generally, it is a good practice to isolate the resource specific state file into separate folder under S3 bucket.

Writing the backend configuration is not enough. We need also save the important resource properties in remote state file. For instance, suppose we need to allow VPC CIDR block traffic in a specific security group then we need to use `terraform output` to save that configuration in remote state:

```hcl
output "vpc_id" {
  value = "${aws_vpc.vpc.id}"
}

output "vpc_cidr" {
  value = "${aws_vpc.vpc.cidr_block}"
}
```

Now we need to initialize this configuration with terraform and make it create a VPC resource in AWS:

```bash
$ make init
$ make apply
```

Now lets create security group in that specific VPC. For that, we need to introduce VPC remote state file in security group remote backend configuration and use that backend information as a data source:

```hcl
data "terraform_remote_state" "shared_vpc_local" {
  backend = "s3"
  config {
    bucket = "webapp-terraform-state"
    key    = "vpc/vpc.tfstate"
    region = "us-east-1"
  }
}
```

Then use that data source for retrieving `vpc_cidr` property from remote state file:

```hcl
egress {
  from_port   = 80
  to_port     = 80
  protocol    = "tcp"
  cidr_blocks = ["${data.terraform_remote_state.shared_vpc_local.vpc_cidr}"]
}
```

Use `make apply` to run the configuration and you will find that security groups got created with desired configuration. Get the source code [here](https://github.com/shudarshon/challenge-terraform/tree/master/6-tf-s3-backend).

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-terraform/issues).
