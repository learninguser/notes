# Introduction to DevOps

Date: 30-07-2023

## Responsibilites

- Deploy the application as fast as possible with speed and accuracy
- Monitor the applications continuously
- DevOps is a process which can be improved continuously

## Waterfall model

- Clients cannot see the application until it is released
- There used to be only one final release in the PROD stage to clients
- No testing until the whole product is developed
- This is the case until 2012

## Agile model

- Product is dividied into multiple modules and are referred as **Sprint based development**
- Duration for each sprint is less for e.g. One month
- All teams that are involved in the project are part of this sprint planning meetings
- First 2 weeks is the development phase and next 2 weeks is for testing and deployment phase
- With this the client can test the application at their end at the end of each sprint
- Then DevOps came into picture which is again an Agile model but with a simple process change
- After developing each feature, it is tested by the testing team and deployed immediately
- Communication, Collaboration and Co-operation

## Tools and their purpose

- Configuration Management: Ansible
- Continuous Integration: Jenkins (Code Pipeline)
- Source Code Management: GitHub (Code Commit)
- Continuous Deployment: Ansible (Code Deploy)
- Build: Maven, NPM
- Cloud Infrastructure: AWS, Azure, GCP or something else
- Monitoring: Prometheus, ELK, New Relic

## Short intro into Linux

- 96% of world servers are linux
- We use a server for hosting the application
- Unix and Mac OS are locked i.e. with the underlying hardware
- **Linux is a kernel**
- Supports multiple users at the same time

## How we can connect to Linux Server

- Authentication and Authorisation
- What you know? Password
- What you have? RSA Key, Tokens etc
- What you are? fingerprints, palm, retina etc

- In Linux servers, we have public and private key mechanisms
- Linux Box/server/node
- Public Key is present inside the server
- We can generate Public and Private key pairs using: `ssh-keygen -f <filename>`
- We can connect to the linux server using SSH protocol, port-no, IP address, Username, Password/key

## Popular services and their port numbers

- HTTP: 80
- MySQL: 3306
- DNS: 53
- SMTP: 25
- Tomcat: 8080

## Steps to Launch and connect to a server

- Step 1: Create a Firewall (Security Group)
  - Name: it as allow-all
  - Inbound and outbound rules: All TCP and traffic from anywhere in the world
- Step 2: Create a key-pair using `ssh-keygen` command and import the public key
- Step 3: Launch a server (EC2-Instance)
  - OS: AWS Linux 2
  - keypair: choose the one which we created in step 2
  - Security Group: allow-all
- Step 4: Connect to the server using `ssh -i <path-to-pem> <username>@<IP address of the server>`

## Linux Commands

- `command <options> <input>`
- `pwd` -> present working directory
- `#` -> root user
- `$` -> Normal user (default)
- `uname` -> which kernel are we using?
  - `uname -a` -> Detailed information of the kernel
- `ls` -> list all the files and folders in that folder
  - `ls -l` -> long listing of each file and folder
  - `ls -lr` -> Sort files in alphabetical order
  - `ls -lrt` -> Sort files from oldest to newest
    - `d` -> directory
    - `-` -> file
  - `ls -la` -> long listing including hidden files and folders
- `cd <path-of-the-directory>` -> Change directory
- `touch <filename>` -> An empty file is created with the filename
- `mkdir <folder-name>` -> Create a folder
- `cat > <filename>` -> Allows to user to enter the text inside the file
  - Once we are done with it, we need to press `ctrl + d`
  - Removes the older content and stores the new content
  - `>>` -> Appends to the older content in the file
- `cat <filename>` -> To print the contents of the file onto the terminal
- `rm <filename>`  -> To remove the file
- `rmdir <folder name>`  -> To remove an empty folder
- `rm -r <folder name>`  -> To remove a folder and its contents in a recurrsive way
- `cp <source> <destination>` -> To copy a file from one location to another
  - `-r` -> to copy the files in a recurrsive way
- `mv <source> <destination>` -> To move a file from one location to another
