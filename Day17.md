# IAM, Ansible

Date: 24-08-2023

## IAM: Identity access management

### Authentication & Authorization

- With Username and password -> authentication
- As AWS has 1000's of services, if a new user is created he will not have admin access rather he has limited access i.e based on the permissions assigned to the user
- Each user can have certain permissions
- In every software system, each user has the following:
  - Username and Password
  - User Group
  - User Permissions
- AWS services also should have certain permissions, so that they can't create resources automatically
- Using AWS Command line utlility, we can create and manage resources on our AWS account
- Roles: Its for non humans, which should be attached to the AWS services so that they can create resources based on our request.
- When creating role, we need to choose the following:
  1. For which service --> EC2
  2. What are its permissions --> AdministratorAccess

## Scripts for creating instances automatically and updating Route53 records

- We can create EC2 instances using AWS CLI: `aws ec2 run-instances --image-id ami-03265a0778a880afb --instance-type t2.micro --security-group-ids sg-039d04b205a0ea7db`
- We can perform query on this using JSON Query (jq) to fetch the Private IP address of the instances: `aws ec2 run-instances --image-id ami-03265a0778a880afb --instance-type t2.micro --security-group-ids sg-039d04b205a0ea7db | jq -r '.Instances[0].PrivateIpAddress'`
- [Script](https://github.com/sivadevopsdaws74s/roboshop-shell/blob/master/create-ec2.sh)

## Disadvantanges with Shell Scripting

1. Homogenous i.e. script is dependent on distribution of OS
2. Validation is necessary at each step
3. If we have 1k servers, we need to login and run the script manually
4. Imperative approach

### Imperative vs Declarative

- Imperative code focuses on writing an explicit sequence of commands to describe how you want the computer to do things
  - E.g. Shell Scripting
- Declarative code focuses on specifying the result of what you want, simpler syntax
  - E.g. Ansible, Terraform

## Push vs Pull Mechanism

- With Push mechanism, server pushes the configuration to the nodes
- With Pull mechanism
  - Nodes connect with the server (called as Configuration server) and pull the configuration from the server
  - Schedule how frequent the nodes should connect to the server for which we need to have an **agent** software installed on the nodes
  - Even though change in configuration is very rare, node needs to stay in connected with the server due to which the resources are wasted

## Ansible

- With Ansible, we can connect to any number of servers without explicit login
- We have one main server (called as Configuration server) and all the other servers we call them as Nodes on which we can execute our code
- Ansible connects to the nodes remotely using SSH mechanism
- We can also connect with SSH to the remote server using: `sshpass -p 'DevOps321' ssh centos@<IP address>`
  - In addition to that, we can also create files on the remote server without logging in: `sshpass -p 'DevOps321' ssh centos@<IP address> touch /tmp/test`
- Ansible works on Push Mechanism where as Chef, Puppet works on Pull Mechanism
- We can install Ansible using: `sudo yum install ansible -y`
- The instance on which Ansible is installed is called **Ansible server**
- We can use command line utility to execute commands on our nodes for e.g. `ansible -i inv all -e ansible_user=centos -e ansible_password=DevOps321 -b -m ansible.builtin.yum -a "name=nginx"` can be used to install NGiNX on remote node present inside inv file
- Similar to Shell Script, if we have all the ansible related commands inside a file, it is called ansible playbook
- Ansible playbooks are written using YAML (Yet Another Markup language)
