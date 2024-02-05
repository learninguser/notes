# Manual installation of project

Date: 11-08-2023

## Release another version

1. Stop the server
2. Delete the old package
3. Place the new package
4. Restart the server

- SOP: Standard Operating Procedure
- Each project has a standard documentation called as SOP which has the information on installing the necessary tools

## Redis

- It is a popular cache server and in-memory database
- Querying the database is an expensive operation because no: of steps that it needs to go through, the latency increases.
  1. Connect to DB
  2. Fetch the data
  3. Close the Connection
- To reduce the latency, a cache server is used which stores the data in a key-value pair format after the first request
- RPM: Redhat Package Manager
- All packages in RedHat are RPM's
- What is the difference between YUM and RPM?
  - `yum` helps us with dependency resolution before installing the requested package
- Starting from CentOS 8, Redis provided with a single repo file with which we can either upgrade or downgrade the versions easily by enabling them at the time of installation

## User

- Similar approach like catalogue as its written in NodeJS
- This also depends on MongoDB and Redis

## Cart

- Similar to User and catalogue i.e. written in NodeJS
- Depends on Redis and Catalogue

## MySQL

- We can have a larger instance type i.e. for e.g. t3.micro
- By default, CentOS 8 yum package manager installs MySQL v8 but our project is dependent on MySQL v5.7
- Therefore, we disable it and manually configure MySQL repo

## Shipping

- Depends on MySQL for loading the country and city information
- Maven is build tool for Java
- When we install Maven, it will install Java automatically
- `mvn clean package` - Installs dependencies and creates a .jar package inside **target/** directory
  - .jar - Java Archive file
- Maven installs dependencies that are listed in `pom.xml` file
