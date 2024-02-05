# Roboshop Implementation

Date: 31-08-2023

- Setting up other components using Ansible

## Difference between Shell and Command module

- With command module, we can only perform simple tasks
- With shell module, we can have access to the environment variables, user variables i.e. as if we're inside the server

## Difference between infrastructure and configuration

- In infrastructure, we create servers, route53 records, VPCs, security groups etc
- In configuration, we install packages and their dependencies
- With configuration management such as Ansible, we can also create EC2 servers but it is not at all recommended.
