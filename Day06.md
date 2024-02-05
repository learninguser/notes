# Manual installation of project

Date: 07-08-2023

- Every application these days follow 3 tier architecture i.e. Web Tier, Application Tier and Database Tier
- Web Tier includes Load Balancer component as well

## Public IP, Private IP and How Internet works

- Each and every device that is connected to internet has public and private IP
- Till 1995, we only had Public IP
- After that we have Network Address Translation (NAT)
- Within a network, each device is assigned with a Private IP by the Network modem
- Devices that are connected with in a network can communicate with each other
- A modem is assigned with a Public IP by the ISP
- A DNS (Domain Name System) contains a list of domain names linked with its IP address
- ISP is responsible for resolving the IP address of a domain
- When a request is raised by the user, it goes through the following steps:
  1. Requests the modem
  2. Reaches the ISP provider
  3. ISP checks with all the available home networks
  4. If not, it will reach a **peering/exchange point** to check with all the ISP providers
      - All the ISP providers in the country are connected using an exchange point
  5. If not, it will reach other peering points in the world to fetch the IP address
  6. If not, there is no IP address linked with the domain name that we are trying to reach for

## Monolithic vs Microservices architecture

### Monolithic

- Earlier everything used to be present in one server
- A simple change also need to be deployed again
- Maintainence and monitoring is a big headache
- Each application that is developed is released as an **.ear** file i.e. enterprise archive on the server
- For every new change, the entire application needs to be changed and a new .ear file is generated
- This leads to downtime of the application
- Migration is very difficult in monolithic
- All the components should be programmed in one programming language
- Scaling is very difficult at the time of more traffic

### API based

- All the components are communicated using API calls
- Frontend always communicates with the backend through API and backend responds to the request with a JSON response
- Now, The applications are deployed using .war file i.e Web Archive
- To overcome the disadvantages with the Monolithic based architecture, Microservices were introduced
- With microservices, the communication is established using Private IPs and API calls
- Each component can be programmed in any programming language and expose its functionality using APIs
- Entire application is developed by the developers and they provide us with the documentation in which the dependencies information is mentioned

## Security Group

- A Security group is a firewall
- Here we have Inbound and outbound traffic information
- In inbound, we specify the rules for incoming traffic
- Similarly in outbound, we specify the rules for outgoing traffic
