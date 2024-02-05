# Manual installation of project

Date: 14-08-2023

## RabbitMQ

- Its one of the popular Messaging Queue Software
- ActiveMQ, Kafka are some other message queue softwares
- AMQP: Advanced Messaging Queue Protocol

### Synchronous and Asynchronous communication

- In Synchronous communication, we get an immediate response
- When a HTTP request is sent and if an immediate response is exepected to receive, then it is said to be Synchronous in nature
- What if the server is down? at that point we will not get any response from the server
- Lets consider the case of what happends with WhatsApp?
- If the other person is unavailable, the message that we send will be lost incase of HTTP Synchronous request
- To overcome this issue, WhatsApp implemented a **MQ Sever**
- Any message that is sent is first stored in the MQ Server
- Any user that is registered with WhatsApp is also subscribed the MQ Server
- A MQ Server consists of two things.
  1. Queue: Point-to-point communication (One to One)
  2. Topic: Publish and subscribe (One to Many)

#### Queue

- Each user has his own queue inside the MQ Server and is subscribed to that queue
- When a message is sent to a user, it will be stored inside his respective queue
- If the user is online, the messages will be send to him
- If he is offline, the messages are stored in his queue for certain number of days (e.g. 30 days) and will be delivered to him once he is online

#### Topic

- Let's understand the analogy using Youtube
- When a user is subscribed to a channel, he gets notified once a new content is published to that channel
- Within Youtube server, there is topic for each channel and each and every user subscribe to that channel are subscribed to that topic
- Any content that is published on the channel will trigger a message to the topic with which all of its subscribers are notified immediately
- All subscribed users are actively listening to the topic continously because of which a notification is sent to them at a very fast pace

## Payment

- Written in Python
- Dependent on Cart, User and Rabbitmq

## Dispatch

- Always listens to Payments Queue for product and shipping information
- Developed in Golang

## Disadvantages with Manual installation

1. Everytime we need to configure same steps of creating the microservice (Ansible)
2. We should map the IP address to a Domain name as the Public IP changes when we restart the server
3. What if the traffic is more? We need to scale the severs ---> Autoscaling
4. We need to monitor for errors. System resources should be monitored
5. Resource Utilisation --> Min. servers but more performance ---> Monitoring
6. Automatic Deployment (CI/CD) -> Whenever there is a new release, we need to:
    - Compile
    - Package
    - Scan
    - Remove old version
    - Release new version
7. Automate the infrastructure creation ---> Using terraform
8. We should install HTTPS certificate for our domain names

## How DNS works?

- When we enter an website address in a browser, it will follow this process for finding the IP address of that website
  1. Check the browser cache if is already present or not
  2. Check in OS Cache and OS is configured with the ISP information
  3. Check with ISP through Modem, if its present in its DNS server and its DNS Cache
  4. ISPs are configured with Root servers and therefore they reach out to them for the IP address of the website
     - Root servers are managed by independent organisations which is called as ICANN (Interent Coorporation of Assigned Names and Numbers)
     - ICANN manages the DNS Root servers
     - IANA: Internet Assigned Numbers Authority
     - Root servers contains top level domains (TLDs) such as .com, .us, .in, .net etc
     - When we purchase a domain from a domain broker, its their responsiblity to update root servers about this domain
     - In addition that, a domain name is also allocated with Name servers i.e. who is managing the domain
     - Root servers provide the information about the name servers that are linked with the website address
  5. Finally ISP contacts the Nameservers for the IP address and sends it back to the client
- We need to always create records for associating an IP address to a web address
- But we need to change the ownership of the Name Servers to AWS for automation purpose using Route53 service in AWS
- Once we made the change, hostinger will connect with the root severs and updates the change in name servers
- TTL: Time to Leave is similar to Cache i.e. how long the information related to the record should be there

## Types of Records

- A Record: domain name pointing to a IPv4 address
-
