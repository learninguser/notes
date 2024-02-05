# Terraform VPC

Date: 12-09-2023

## Modules

- When defining the module, **provider** information is not necessary
- Only when consuming the module, we need to provide this information

## VPC

- Virtual Private Cloud
- IPv4 Address --> 32 bits --> 4 octets --> each octet consists of 8 bits
- In binary system if you have n bits, you can generate 2^n combinations
- 32 bits --> 2^32 --> 4.3 billion
- All the devices that are connected to the Modem are assigned with Private IP address where as a modem is assigned with both Public and Private IP
- If the communication is going outside of the network, then it uses Public IP address and this is called as NAT (Network Address Translation)
- Ideally every network is isolated and it can contain any number of devices connected to it
- If we select 1 bit out of 2 bits, then how many parts can I divide the entire network? Ans: 2
- If we have k bits and out of this if we select n bits, we can have
  - parts = 2^n
  - each part contains --> 2^n - 1
- For e.g. Total number of bits, we have is 3 (i.e. k = 3). Out of which we select 2 bits (n = 2)
  - This means, we can have 2 ^ 2 = 4 parts and each part consists of 2^(k - n) = 2^(3 - 2) = 2
- CIDR: Classless Inter Domain Routing
- An IP address is a combination of Network ID + Host ID

### Private IP ranges

1. 10.0.0.0 to 10.255.255.255 (10.0.0.0/8)
2. 172.16.0.0 to 172.31.255.255 (172.16.0.0/12)
3. 192.168.0.0 to 192.168.255.255 (192.168.0.0/16)

- Lets consider a CIDR block: 10.0.0.0/8
  - In this the first 8 bits are reserved for Network, remaining 24 bits are for Host
  - Network bits are not allowed to use
- Our task is to split this CIDR block into 2 parts, therefore we need 1 extra bit
- The 1st subnet will be: 10.0.0.0/9 i.e. 10.0.0.0 - 10.127.255.255
- The 2nd subnet will be: 10.128.0.0/9 i.e. 10.128.0.0 - 10.255.255.255
- AWS will only allow 16 - 28 as CIDR notation
- By default, every subnet has its route table set to local i.e. all the devices within the subnet can communicate with each other
- We have 2 types of routes:
  - Public Route
  - Private Route
- A route table consists of routes listed in it
- Each route table should be associated with atleast one subnet
- For e.g. Public Route table is assocated with public subnet
