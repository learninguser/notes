# Terraform: tfvars, Multienvironment, intro to modules

Date: 11-09-2023

## tfvars

- To maintain consistency across multi ENV, we can use same code but with different variables
- For e.g.:
  - We need 10 instances in DEV and 10 instances in PROD
  - Route53 records for DEV and PROD
  - For e.g. mongodb-dev.learninguser.online for dev and mongodb-prod.learninguser.online
- Values mentioned in `variables.tf` are default values which can always be overwritten using `terraform.tfvars` which will be loaded automatically
- But it is not mandatory to define default values
- We create separate .tfvars file for DEV and PROD for e.g. dev.tfvars, prod.tfvars
- This information can be passed onto terraform using: `terraform plan -var-file=DEV/dev.tfvars`
- Usually in PROD, we expect huge traffic and therefore, we choose larger instance tyes
- If a variable is defined but is not initialised with a default value anywhere in the .tfvars or variables.tf, terraform prompts the user to enter a value at the time of planning or applying
- Once the dev instances are created and after that if we want to create the PROD infrastructure, terrform always replaces the DEV infra and with the PROD infra using the current approach as we are saving the remote state with the same name in S3

## Multi Environment

### Approach 1: Different repos

- Maintain different repos i.e. roboshop-DEV and roboshop-PROD
- But the disadvantage with this approach is that, the code is duplicated and we need to work with different repos
- Preferred for projects with larger infrastructure such as Banking

### Approach 2: Using tfvars

- When using the same code snippets for different envrionments, we need to have different tfvars file and remote state location i.e. s3 bucket
- The remote state file location information can be specified by defining `backend.tf` file for each environment
- We can initialise it at the time of dev environment using `terraform init -reconfigure -backend-config=DEV/backend.tf`, `terraform plan -var-file=DEV/dev.tfvars`, , `terraform destroy -var-file=DEV/dev.tfvars` and when using the same script for production, we should reconfigure using: `terraform init -reconfigure -backend-config=PROD/backend.tf`, `terraform plan -var-file=PROD/prod.tfvars`, `terraform destroy -var-file=PROD/prod.tfvars`
- If we use the same statefile for different environments, Terraform deletes the resources and creates new resources for that environment
- This is because Terraform tries to validate the Terraform scripts with remote tfstate file
- Preferred for projects with smaller infrastructure

### Approach 3: Terraform workspaces

- Terraform Workspace: Using this, we can use single code to manage multiple environments
- To get the list of workspaces: `terraform workspace list`
- By default, terraform creates a **default** workspace
- Only some backends are supported by workspaces
- To create a workspace, we can use: `terraform workspace new <name of the workspace>`
- By default, terraform provides a variable called: `terraform.workspace`
- Workspace also creates different buckets based on the environment in the Remote backend such as S3
- To switch between workspaces, we can run: `terraform workspace select dev`

## Provisioners

- There are useful to integrate with configuration management tools such as Ansible
- The main input for configuration management is that, the server should be up and running
- As Ansible doesn't maintain the state of the infrastructure it created, it is intended for infrastructure creation
- There are 2 types of provisioners
  1. local-exec: The code block will run in the machine where terraform command is executed
      - Works only once
      - Creation Time: local-exec will run at the time of server creation
      - Destroy time: local-exec will be destroyed at the time of server termination `when = destroy`
      - on_failure: Handling errors for e.g. continue the execution of the rest blocks `on_failure = continue`

      `ec2.tf`

      ```hcl
      resource "aws_instance" "example" {
        ami           = "ami-03265a0778a880afb"
        instance_type = "t2.micro"

        tags = {
          "Name" = "test"
        }

        provisioner "local-exec" {
          # self = aws_instance.example
          command = "echo The server's IP address is ${self.private_ip}"
        }
      }
      ```

  2. remote-exec
      - This will run inside the server
      - To run, we should first connect to the server only then we can run something on the server

      `ec2.tf`

      ```hcl
      resource "aws_instance" "example" {
        ami           = "ami-03265a0778a880afb"
        instance_type = "t2.micro"
        vpc_security_group_ids = [aws_security_group.roboshop-all.id]

        tags = {
          "Name" = "test"
        }

        connection {
          type = "ssh"
          user = "centos"
          password = "DevOps321"
          host = self.public_ip
        }

        provisioner "remote-exec" {
          inline = [
            "sudo yum install nginx -y",
            "sudo systemctl start nginx",
          ]
        }
      }
      ```

      `sg,tf`

      ```hcl
      resource "aws_security_group" "roboshop-all" {
        name        = "provisoner"
        description = "provisoner"

        ingress {
          description      = "Allow SSH"
          from_port        = 22
          to_port          = 22 
          protocol         = "tcp"
          cidr_blocks      = ["0.0.0.0/0"]
        }

        ingress {
          description      = "Allow HTTP"
          from_port        = 80
          to_port          = 80 
          protocol         = "tcp"
          cidr_blocks      = ["0.0.0.0/0"]
        }
        
        egress {
          from_port        = 0
          to_port          = 0
          protocol         = "-1"
          cidr_blocks      = ["0.0.0.0/0"]
        }

        tags = {
            Name = "provisoner"
        }
      }
      ```

## Modules

- Using modules, we can maintain DRY principle
- We can reuse it when ever we want
