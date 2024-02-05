# Infrastructure as a Code using Terraform (continued)

Date: 07-09-2023

## Tags and Variables

- Tags for resources on AWS are used for resource filtering and cost-optimisation
- These tags are simple but very powerful
- Usually we have the following tags:

  ```text
  Name=some-name
  Environment=DEV/QA/PROD
  Terraform=true
  Component=MongoDB
  Project=Roboshop
  ```

- Rather than hardcoding, we can store them in variables and access them

  `variables.tf`

  ```hcl
  variable "tags" {
    type = map
    default = {
      Name = "MongoDB"
        Environment = "DEV"
        Terraform = "true"
        Project = "Roboshop"
        Component = "MongoDB"
    }
  }
  ```
  
  `ec2.tf`

  ```hcl
  resource "aws_instance" "example" {
    ami           = var.ami_id
    instance_type = var.instance_type
    tags          = var.tags
  }
  ```

## Outputs

- Variables are used for providing inputs
- Similarly, we have outputs for e.g. fetch the public IP of the EC2 instance once its created
- When Terraform creates resource on our behalf on the provider, it will also fetch outputs information
- For each resource there are certain outputs that Terraform fetches
- Syntax

  ```hcl
  output "output-variable-name"{
    value = "value"
  }
  ```

- For e.g.

  `outputs.tf`

  ```hcl
  output "public_ip"{
    value = aws_instance.example.public_ip
  }
  ```

## Conditions

- Syntax: `expression ? "this will run if true" : "this will run if false"`
- For e.g.: if MongoDB we are creating t3.medium otherwise t2.micro
- OR is represented using: `||`
- To enter into terraform console, we can use: `terraform console`

  `ec2.tf`

  ```hcl
  resource "aws_instance" "test" {
    ami                    = var.ami_id
    instance_type          = var.instance_name == "MongoDB" ? "t3.small" : "t2.micro"
    vpc_security_group_ids = [aws_security_group.allow_all_1.id]

    tags = {
      Name = "web"
    }
  }
  ```

  `variables.tf`

  ```hcl
  variable "instance_name" {
    type = string
    default = "web"
  }
  ```

## Loops

- There are 3 types of loops
  - Count and Count index
  - For each
  - Dynamic Block

1. Count and Count index
    - To iterate over a list, we use count.index
    - Example:

    `count.tf`

    ```hcl
    resource "aws_instance" "test" {
      count                  = 11
      ami                    = var.ami_id
      instance_type          = var.instance_type
      vpc_security_group_ids = [aws_security_group.allow_all_1.id]

      tags = {
        Name = var.instance_names[count.index]
      }
    }
    ```

    `vars.tf`

    ```hcl
    variable "instance_names" {
      type = list
      default = ["mongodb", "redis", "rabbitmq", "mysql", "catalogue", "user", "cart", "shipping", "payment", "dispatch", "web"]
    }
    ```

2. For each
    - To iterate over map or list
    - We use each special variable to access the values

    `foreach.tf`

    ```hcl
    resource "aws_instance" "web" {
      for_each      = var.instance_names
      ami           = var.ami_id 
      instance_type = each.value
      tags = {
        Name = each.key
      }
    }
    ```

    `variables.tf`

    ```hcl
    variable "instance_names" {
      type = map
      default = {
        mongodb = "t3.small"
        redis = "t2.micro"
      }
    }

    variable "ami_id" {
      default = "ami-03265a0778a880afb"
    }
    ```

## Functions

- Using file function, we can read and load the data in that file at runtime
- Syntax: `file("${path.module}/hello.txt")`, in this hello.txt should be present in the current working directory

  `count.tf`

  ```hcl
  resource "aws_instance" "web" {
    count         = length(var.instance_names)
    ami           = var.ami_id #devops-practice
    instance_type = contains(["mongodb", "shipping", "mysql"], var.instance_names[count.index]) ? "t3.small" : "t2.micro"
    tags = {
      Name = var.instance_names[count.index]
    }
  }
  ```

  `vars.tf`

  ```hcl
  variable "instance_names" {
    type = list
    default = ["mongodb", "redis", "rabbitmq", "mysql", "catalogue", "user", "cart", "shipping", "payment", "dispatch", "web"]
  }
  ```

## Locals

- locals is also a type of variable, **but it can have expressions and functions**
- They can validate expressions and store its result inside a local variable

  `count.tf`

  ```hcl
  resource "aws_instance" "web" {
    count         = length(var.instance_names)
    ami           = var.ami_id
    instance_type = local.instance_type
    tags = {
      Name = var.instance_names[count.index]
    }
  }
  ```

  `vars.tf`

  ```hcl
  variable "instance_name" {
    type = list
    default = mongodb"
  }
  ```

  `locals.tf`

  ```hcl
  local {
    instance_type = contains(["mongodb", "shipping", "mysql"], var.instance_name) ? "t3.small" : "t2.micro"
  }
  ```

## Data Sources

- With Data Sources, we can query the data dynamically from the provider for e.g. AWS
- We have old resources that created manually on AWS.
- Now we are using terraform to create resources and that have dependency on old resources.
- So using data sources, we can fetch the info dynamically

`data.tf`

```hcl
data "aws_ami" "aws-linux-2"{
  owners           = ["amazon"]
  most_recent      = true

  filter {
      name   = "name"
      values = ["amzn2-ami-kernel-5.10-hvm-*"]
  }

  filter {
      name   = "root-device-type"
      values = ["ebs"]
  }

  filter {
      name   = "virtualization-type"
      values = ["hvm"]
  }
}

# Fetch default VPC information
data "aws_vpc" "default" {
  default = true
}
```

`output.tf`

```hcl
output "aws_ami_id" {
  value = data.aws_ami.aws-linux-2.id
}

output "vpc_info" {
  value = data.aws_vpc.default
}
```

`ec2.tf`

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.aws-linux-2.id
  instance_type = "t2.small"
  tags = {
    Name = "data-source"
  }
}
```

- When a change is made to an existing resource AWS can either terminate it and create a new resource or just stop and restart it to update the resource
- For e.g. when we change the instance type from
  - t2.micro -> t3.medium, in this case AWS just updates the exisiting resource
  - t2.micro -> m4.xlarge, in this case AWS terminates the exisiting resource and creates a new one
