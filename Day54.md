# Roboshop on Docker (Continued)

## User

- Download and copy `package.json` and `server.js` to build the docker image user
- User is dependent on redis, therefore we can first download the redis image and create a container with name redis

## Disadvantages of running docker commands manually

- We need to provide all options through the command line
- We need to ensure that the dependency containers are running first
- We need to start and stop the containers one by one in an order

- To overcome these disadvantages, we can use `docker compose` with which we can start, stop and rebuild services easily
- Docker compose is written in YAML
