# Infrastructure as a Code using Terraform

Date: 06-09-2023

## Infrastructure

- Not only servers but also security group, IAM, VPC, databases, load balancers, Route53 records etc
- Creating manually creates lot of problems i.e. if something goes wrong, we need to terminate and recreate them once again
- Therefore, we use Infrastructure as a Code (IaaC/IaC)
- Ansible: Configuration as a Code

## Terraform

- Terraform is a popular IaaC tool
- 90% of the companies are using Terraform in their DevOps lifecycle
- Terraform is very crucial for a DevOps Engineer
- It is a Hybrid IaaC i.e. with which we can create resources on many platforms
- Terraform uses declarative syntax for Infrastructure automation
- Terraform scripts are written using Hashicorp Configuration Language (HCL)

### Advantanges

1. **Version Control**: Since Terraform is based on scripting, it should be maintained i.e. the history should be preserved and revert to a previous version if something goes wrong
2. **Consistent Infrastructure**:
    - Usually due to inconsistent infrastructure, we have issues in different environments
    - Therefore, we use same code in different environments
3. **CRUD** operations: We can perform CRUD operations on our infrastructure without logging into console
4. **Inventory Management**: Just by looking at the code, we can say which services and resources are being used for a particular project
5. **Cost Management**: When in need we create and when not, we can delete the resources
6. **Dependency Management**: When we need to create an EC2 instance through AWS console, we need to create a security group first
7. **Modules**: Maintain DRY principle i.e. make use of modules in different projects when ever required
8. **Hybrid Cloud**: Can be integrated with many cloud providers

## Environment setup

1. [Terraform installation](https://developer.hashicorp.com/terraform/downloads?product_intent=terraform)
2. [AWS CLI v2](https://awscli.amazonaws.com/AWSCLIV2.msi) for authentication with AWS
3. Create a new user on AWS for command line purpose to authenticate with AWS
4. Run `aws configure` command on Terminal and provide the Access Key ID and Secret Key information in that

- Syntax
  
  ```hcl
  resource "what-resource" "name-your-resource-your-wish"{
    arguments / options / parameters
  }
  ```

- what-resource -> you need to get it from terraform documentation
- For a particular resource type, some arguments are mandatory and some are optional
- If we don't provide values to the optional arguments, default values that are in our AWS account will be choosen automatically by Terraform
  - For e.g. at the time of EC2 instance creation, a default security group is assigned if we don't select a security group

## Creation of any resource with Terraform

- **provider** i.e. we need a provide the information of the provider on which we want to create a resource using `provider.tf`

  `provider.tf`

  ```hcl
  terraform {
    required_providers {
      aws = {
        source = "hashicorp/aws"
        version = "5.16.2"
      }
    }
  }

  provider "aws" {
    # Configuration options
    region = "us-east-1"
  }
  ```

- We can specify aws_access_key_id and aws_secret_access_key information in provider but for security reasons we don't specify it
- **resource** i.e. we need to provide the information about the resource which we would like to create for e.g. an ec2 instance inside `ec2.tf`

  `ec2.tf`

  ```hcl
  resource "aws_instance" "example" {
    ami           = "ami-03265a0778a880afb"
    instance_type = "t2.micro"

    tags = {
      Name="example"
    }
  }
  ```

- Ideally we can have everything in one single .tf file but it would be very difficult to maintain
- Therefore, we split it into seperate files so that we can easily maintain and others can also understand it easily

## Commands to execute

- terraform init --> this will intialize terraform
- terraform plan --> terraform will tell us what are the resources it is going to create
- terraform apply --> it will create the resources
- terraform destroy --> to destroy the resources created by terraform
- Provider SDKs are downloaded at the time of terraform initilisation and are stored in `.terraform` directory
- We shouldn't push the SDK's to GitHub. Therefore we add it to .gitignore file
- **Commenting a code block inside .tf file, indicates to delete that particular resource**

## Variables and Datatypes

### Datatypes

1. string e.g. "hello"
2. number e.g. "203185"
3. bool e.g. true, false
4. list e.g. ["us-east-1a", "us-west-1c"]
5. map e.g. {name = "Mabel", age = 52}

### Variables

- Instead of storing everything in one single file, we split them into multiple files for better readability and code maintainence
- This is where variables comes in handy to store and access data
- type: indicates the data type
- Syntax
  
  ```hcl
  variable "var_name"{
    type    = datatype
    default = "default-value" # can be overriden
  }
  ```

- For e.g. rather than hardcoding the AMI information, we can store inside a variable and refer it where ever we want

  `variables.tf`

  ```hcl
  variable "ami_id"{
    type    = "string"
    default = "ami-03265a0778a880afb"
  }


  variable "instance_type"{
    type    = "string"
    default = "t2.micro"
  }
  ```

  `ec2.tf`

  ```hcl
  resource "aws_instance" "example" {
    ami           = var.ami_id
    instance_type = var.instance_type
  }
  ```

- Similarly if we want to attach a security group to the instance we created, we can create a security group

  `sg.tf`

  ```hcl
  resource "aws_security_group" "allow_all" {
    name        = var.sg_name
    description = "Allowing All ports"

    ingress {
      description      = "Allowing all inbound traffic"
      from_port        = 0
      to_port          = 0
      protocol         = "tcp"
      cidr_blocks      = var.sg_cidr
    }

    egress {
      from_port        = 0
      to_port          = 0
      protocol         = "-1"
      cidr_blocks      = ["0.0.0.0/0"]
    }

    tags = {
      Name = "terraform"
    }
  }
  ```

  `variables.tf`

  ```hcl
  variable "sg_name" {
    default = "allow-all"
  }

  variable "sg_cidr" {
    type = list
    default = ["0.0.0.0/0"]
  }
  ```

  `ec2.tf`

  ```hcl
  resource "aws_instance" "example" {
    ami           = var.ami_id
    instance_type = var.instance_type
    security_groups = [ aws_security_group.allow_all.id ]
  }
  ```
