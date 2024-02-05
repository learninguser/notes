# Terraform continued: VPN and VPC

## Concepts

1. Variables
2. Data Types
3. Conditions
4. Loops
5. Functions
6. Count and Count Index
7. Outputs
8. Data Sources
9. Locals

- Using these concepts, we understand the service and provision it using Terraform
- Without having the knowledge on how service works, its not possible to provision it using Terraform even though we can fetch it from the documentation

## Securing Roboshop

- VPC is the base for any project either in cloud or on-premise
- Its a general concept and not specific to AWS
- Within VPC, we should have network security
- Therefore we have:
  - public and private subnets
  - public and private route tables
  - Internet gateway
  - NAT gateway as a route to private subnet
    - Ingress is not allowed
    - Egress traffic is always through NAT
  - Route table associations with subnets
- We have created a VPC for our Roboshop project
- We have a Default VPC configured with our AWS account with a Default Public Subnet
- With in the Default VPC, we will create an EC2 instance and install a VPN software in it
- As a user, we should connect to the Roboshop component servers through VPN only

### How can we establish peering between two VPCs?

- We establish a VPC peering connection between Default and Roboshop VPC
  - Requestor: Default VPC
  - Acceptor: Roboshop VPC
- In Default VPC route table, we should have an entry for Roboshop CIDR and in Roboshop VPC, we should have Default VPC CIDR in order to enable VPC peering between both these VPCs.
- Usually there are several teams that manage the e-commerce application. For e.g.:
  - VPC is handled by Networking team and stores the values in the AWS SSM parameter store
  - VPN team will access and store the parameters to the same central place
  - Similarly other teams also perform the same action
- Each team will only provide a document with the information on how to use it as it is not necessary to understand all the details such as AMI, versions they're using etc
- They should be able to receive and process the requests and return response to them
- Terraform has access to the current folder only. It cannot fetch values from different projects
- When we have a big infrastructure, parsing of tf files and refreshing takes lot of time if we maintain it in a single repo
- If we maintain it in a different repo, the read operations will be much faster
- As a module developer, we are setting the is_vpc_peering_required to false by default

## Connecting to instances in private subnet

- There are two ways in which we can acheive this
- Method 1: Using Jump/Bastion host
- Method 2: Using VPN
- For e.g. when we want to connect to MongoDB instance that is in database subnet, we need to first add the route in MongoDB security group to accept connections from VPN
- Installation using scripts: [Source](https://github.com/angristan/openvpn-install)
