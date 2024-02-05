# Terraform: VPC module development

Date: 13-09-2023

- Agenda: Develop VPC module with High availability, so that anyone in our organisation can use this.
- High Availability can be acheived by creating resources in more than 1 AZ

## VPC Module structure

1. IGW and attach to vpc
2. 2 public subnets --> 1 in 1a, 1 in 1b
3. 2 private subnets --> 1 in 1a, 1 in 1b
4. 2 database subnets --> 1 in 1a, 1 in 1b
5. Group our database subnets
6. Create route table for public
7. Create route table for private
8. Create route table for database
9. Route table and subnet associations
10. NAT gateway

- We create 2 separate repositories i.e. one for VPC module and another one for roboshop infra consuming the VPC module

## Terraform naming best practices

- Ref: [Terraform Naming conventions](https://www.terraform-best-practices.com/naming)

1. Use _ (underscore) instead of - (dash) everywhere (in resource names, data source names, variable names, outputs, etc).
2. Prefer to use lowercase letters and numbers
3. If the resource module creates a single resource of this type (eg, in AWS VPC module there is a single resource of type aws_nat_gateway and multiple resources of typeaws_route_table, so aws_nat_gateway should be named this and aws_route_table should have more descriptive names - like private, public, database)

## Developing VPC module

- When developing VPC module, we need to enable DNS Hostnames and DNS support by setting its value to true
- Usually, its the cloud central teams responsibility to develop modules and DevOps engineers to consume it in their projects
- We can merge two map objects using: `merge()` function
- Using module we can acheive the following functionality:
  1. Code reuse
  2. Enforce some best practices and standards according to the company
- We can create our own modules or use Open-source moduels for our project
