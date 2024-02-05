# Manual installation of project

Date: 10-08-2023

## 3 Tier Architecture

- DB Tier
  - Also called as stateful
  - State represents the data
  - The data should be persistent i.e. it should be securly stored, taken backups regularly etc
- App / API / backend
  - Performs CRUD operations on DB Tier which are Stateless in nature
- Web Tier / Frontend
  - Fetch data from API Tier
  - Also Stateless in nature

## Systemd service

- Linux looks for the services inside this directory at the time of start: `/etc/systemd/system/`
- If a service is placed in this directory, we can enable or disable it using `systemctl` command options
- The applications that are created by `yum` package manager has the service file inside this directory
- We need to create service files inside this directory for our user-defined applications such catalogue, user etc inorder to manage it as native systemctl service

## Catalogue

- DB Tier are heavy applications, suggested to use t3.micro
- For any 3 Tier application, we have an admin site to make entries inside the database
- Developer also provides with the schema and sample data inside the schema/catalogue.js file
- Mongodb-org is the server and mongodb-org-shell is the client to load the data into the server
- `mongo --host <IP-address> < <location-of-schema-file>`
- nodejs usually runs on port 8080

## Status codes

- 2** - Success
- 3** - redirection
- 4** - client side error: Not giving proper URL
- 5** - server side error: There is something wrong inside server/code
