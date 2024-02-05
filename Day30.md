# Terraform VPC module enhancement

Date: 14-09-2023

- We usually have 2 roles i.e. Module Developer and Module user

## Module Developer

- Resource creation/definitions are here
- Variables are must, because diff projects have diff requirements
- data sources, locals, functions to perform certain tasks
- Module developer has to give documentation about the list of inputs and outputs because outputs are used to create other resources such as security group as it needs VPC ID

## Module User

- User should provide values to the variables defined in the module as per the documentation

## Certain enhancements

- Restrict user to create resources in 1a and 1b region only even if he specifies any other regions as values to the variables
- Should only accept 2 public/private/database subnets. If more specifed, throw an error
- Get the project name from the user so that we can some naming convention:
  - `<project-name>-public-<az>`
  - `<project-name>-private-<az>`
  - `<project-name>-database-<az>`
- Values for variables can always be overwritten by user where as the local values cannot be overwritten
- Incoming request is called **Ingress**
- NAT gateway only allows outgoing connections and will not allow any incoming connections
- We can use a module that is published to github using:

  ```hcl
  module "roboshop" {
    source = "git::https://github.com/daws-76s/terraform-aws-vpc.git?ref=main"
  }
  ```

- Once any change is made to the module, they need to pushed to the remote location and need to be fetched by user using: ``terraform init -upgrade` command
- pull = fetch + merge
