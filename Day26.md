# Terraform: Foreach, Dynamic Blocks, statefile

Date: 08-09-2023

## For each

- To access provider documentation: terraform.io -> registry -> (provider) -> Documentation
- Both count based or for each based allows us to create more resources
- E.g. To create 10 instances using For-each loop
- When iterating over a map, it gives us each element inside: **each** variable

`foreach/variables.tf`

```hcl
variable "ami_id" {
  type = string # this is the data type
  default = "ami-03265a0778a880afb" # this is the default value
}

variable "instances" {
  type = map
  default = {
    MongoDB = "t3.medium"
    MySQL = "t3.medium"
    Redis = "t2.micro"
    RabbitMQ = "t2.micro"
    Catalogue = "t2.micro"
    User = "t2.micro"
    Cart = "t2.micro"
    Shipping = "t2.micro"
    Payment = "t2.micro"
    Web = "t2.micro"
  }
}

variable "zone_id" {
  default = "Z0308214GYCUYHGJHT8R"
}

variable "domain" {
  default = "joindevops.online"
}
```

`foreach/ec2.tf`

```tf
resource "aws_instance" "roboshop" {
  for_each = var.instances
  ami = var.ami_id
  instance_type = each.value
  tags = {
    Name = each.key
  }
}

# if web, give public IP else you give private IP
resource "aws_route53_record" "www" {
  for_each = aws_instance.roboshop
  zone_id = var.zone_id
  name    = "${each.key}.${var.domain}"
  type    = "A"
  ttl     = 1
  records = [ each.key == "Web" ? each.value.public_ip : each.value.private_ip ]
}

# output "aws_instance_info" {
#   value = aws_instance.roboshop
# }
```

## Dynamic Block

- E.g. Creating Security Group on AWS using Terraform
- In production environment we need to specify which port needs to be opened
- Count and for each based loops are used for creating entire resource itself
- Whereas with Dynamic block, we can include repeatable nested blocks (for e.g. defining multiple ports in the security group) within the **resource definiton**

`foreach/variables.tf`

```hcl
variable "ingress_rules" {
  type = list
  default = [
    {
        from_port = 80
        to_port = 80
        description = "allowing PORT 80 from public"
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    },
    {
        from_port = 443
        to_port = 443
        description = "allowing PORT 443 from public"
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    },
    {
        from_port = 22
        to_port = 22
        description = "allowing PORT 22 from public"
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

`foreach/sg.tf`

```tf
resource "aws_security_group" "roboshop" {
  name        = "roboshop"
  description = "Allow HTTP HTTPS SSH"

  dynamic ingress {
    for_each             = var.ingress_rules
    iterator             = abc # here you will get a variable ingress (default)
    
    content {
        description      = abc.value["description"]
        from_port        = abc.value.from_port
        to_port          = abc.value.to_port
        protocol         = abc.value.protocol
        cidr_blocks      = abc.value.cidr_blocks
    }
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "roboshop"
  }
}
```

## State file

- Terraform keeps a track of the resources that it created also known as **state** using the **terraform.tfstate** file.
- Therefore terraform's takes the responsibility to ensure that the information in the .tf files and .tfstate file is the same
- When terraform is performing its validations and operations, it creates and applies a lock on terraform.tfstate file called **.terraform.tfstate.lock.info** to avoid editing the tfstate file
- Once the operations are peformed, terraform releases the tfstate file.
- We cannot delete a certain block within the resource rather we should add everything inside the resource itself
- To avoid unneccessary edits to the file from multiple sources, we add the tfstate files to the gitignore
- In addition there is a possiblity for creating duplicate infrastructure if the track of tfstate file is lost
- To do so, we store the tfstate file in a remote location such as S3 with DynamoDB and apply lock (LockID) to it rather than storing it locally

`foreach/provider.tf`

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.15.0"
    }
  }

  backend "s3" {
    bucket         = "learninguser"
    key            = "tfstate_lock"
    region         = "us-east-1"
    dynamodb_table = "terraform_lock"
  }
}
```

- We should run `terraform init -reconfigure` to configure the S3 to store the remote state file
