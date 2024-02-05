# Service Management and 3 Tier architecture

Date: 03-08-2023

## Service Management

- There are 65535 ports on a server
- When ever a service is started, it will get assigned with a port number in this range
- To start a service: `systemctl start <servicename>`
- To check the status of a service: `systemctl status <servicename>`
- To enable the service at time of boot: `systemctl enable <servicename>`
- To check whether a service is working or not: `netstat -lntp`
  - Along with this, we also get the PID, port number information
- To check the memory usage on the server: `free -m`
- Check the harddisk utilisation: `df -hT`
- To check if two servers are communicating with each other or not: `telnet <IP address> <PORT>`

## 3 Tier architecture

- Web Tier: Delivers the frontend content E.g. HTML, CSS, JavaScript
- Application Tier: Code related to that application E.g. Java, .Net, Python, Go
- Database Tier: MySQL, PostgreSQL etc
  - Only application can communicate with this component
- In addition to this, we have a load balancer which manages the traffic and routes it to the available free server
- Load Balancer also comes under Web Tier
