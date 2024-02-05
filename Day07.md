# Manual installation of project

Date: 08-08-2023

## 3 Tier Architecture

- Web Tier / Frontend
- App / API / backend
- DB Tier

## AWS

- AWS resources (such as servers, databases, Load Balancer) are mostly region based i.e. one resource cannot be transferred from one region to another region
- There are very few services (such as IAM) are global
- Every cloud as Regions and Availability zones
- N. Virgina (us-east-1) is the cheapest amongst all the regions
- AZs are used for Disaster Recovery purpose and High Availability
  - Therefore, we create our resources in 2 Azs
  - We can move our resources from one AZ to another AZ
- Each region has a minimum of 2 AZs

### What is an AMI?

- Amazon Machine Image
- Use Case: We frequently need NGiNX servers in our projects
- There are multiple steps involved in it such as OS + Install Package + Configuring service/package
- Rather than repeating these steps, we can create an image from the server i.e. capturing everything inside the server
- If we take an image from an instance when it is in running state, there is a possibility for altering its state
- Therefore its always better to stop the instance and create an image from the instance
- Actions -> Image and Templates -> Create image

## Who choose what?

- Developers and architects choose which programming language, tools and its versions to use in the project ?
- DevOps architects choose which DevOps tools to use and which version to use?

## Course lab

- We use custom AMI's such as Centos-8-DevOps-Practice AMI (AMI ID: ami-03265a0778a880afb) which is present in the community section
- It has password authentication enabled
- The username is centos and password is DevOps321
- Project Documentation: [URL](https://github.com/sivadevopsdaws74s/roboshop-documentation)

## Build process

- Developer writes the code, compiles and builds it to generate a package
- Compile - Check for syntax errors
- Once the build is complete, the package is generated as per the programming language used
  - For e.g. An application written in Java generates a .jar file, other programming languages generate .zip file
- Once the package is generated, we store them in Nexus Repository or S3 etc
- Finally we fetch the packages and deploy them inside the server

## Deployment Process

1. Create server
2. Create environment such as NodeJS, Golang, Java etc
3. Fetch the package and install dependencies
4. Create a service
5. Some Configuration
6. Finally run the application

## NGiNX

- NGiNX is a webserver and serves as a reverse proxy
- The default configuration of NGiNX is present inside `/etc/nginx/nginx.conf`
- User defined configuration can be placed under `/etc/nginx/default.d/` directory
- **When ever we make any change to the configuration file, we should restart our server**
- Logs are present inside `/var/log/nginx`
- To view the real-time logs, we can use: `tail -f /var/log/nginx/access.log`
  - `-f` - follow
- There are 2 types of proxy:
  1. Forward Proxy:
      - E.g. when connecting to the internet, it passes through a proxy server (i.e. on behalf of you) that checks whether the request can be allowed or not, VPN
      - Configuration is client-centric i.e. only client is aware that there is a proxy configured
      - Content filtering and Access control
      - Caching: Frequently accessed content can be cached inside the proxy server of the ISP
  2. Reverse Proxy:
      - Server centric i.e. clients are not aware of the proxies on the server side
      - For e.g. they can only view the web tier but inside that we configure our nginx server to route the traffic to a particular server
      - Load Balancer
      - SSL / HTTPS
      - Security purpose

## MongoDB

- RDBMS: Relation Database Management System
  - SQL based i.e. we have rows and columns
  - E.g. PostgreSQL, MySQL etc
- NoSQL:
  - Stricly no SQL
  - No tables, no rows and colums
  - Rather the data is based on documents and collections which are JavaScript Object Notation (.json) files
  - E.g. MongoDB
- When installing mongodb-org package with yum, it looks for its source from where it can download inside `/etc/yum.repos.d/` directory
- If its unable to find, we need to specify the source file (i.e. .repo file) inside this directory
- After installation, if we want to check if the mongodb installation is successful or not using: `netstat -lntp` or `ps -ef | grep mongo` or `systemctl status mongod`
- MongoDB opens the port 27017
- By default some severs mostly DB servers runs on localhost (127.0.0.1) for security reasons
- But for our practice purpose, we will open it to accept all traffic from any IPv4 by changing the **bindIp** from 127.0.0.1 to 0.0.0.0 inside `/etc/mongod.conf`
- After that we need to restart the mongod service

## Catalogue

- It is microservice developed in NodeJS by the developer
- What it does? It connects to the MongoDB, fetches the products and shows on the frontend
- To work with this NodeJs application, we need a NodeJS environment
- To run the application, we need to install all the necessary dependencies/packages that are necessary for it to run
- Different build tools specify the list of dependencies in different files:
  1. Maven: pom.xml
  2. nodejs/any js: package.json
  3. python: requirements.txt

### Different types of users

- Normal user: Human
- System users (non-human): Instead of running the applications as a root user, applications have dedicated users to run the application for security reasons
