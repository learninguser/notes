# Docker Continued

## Volumes

- Container in docker is ephemeral i.e. data will be deleted once the container is removed
- If we run DBs as container, by default the data will be removed once the container is deleted
- Therefore, an alternative would be: **Docker Volumes**
- There are 2 kinds of volumes:
  1. Unnamed volumes
  2. Named volumes

### Unnamed volumes

- We can mount a volume at the time of container creation itself using: `docker run -d -p 8080:80 -v <host-path>:<container-path> nginx`
- Docker creates a temporary volume for storing the data of the container in `/var/lib/docker/overlay2` directory
- These kind of volumes are said to be as unnamed volumes
- This is not preferred usually as we need to manage it completely i.e. docker commands will not work to manage this volume type

### Named Volumes

- Functionality wise it is very similar to unnamed volumes
- To create a named volume: `docker volume create <volume-name>`
- In this case, the volume is created under: `/var/lib/docker/volumes/<volume-name>/_data`
- This way, we can manage the created volume using docker commands
- Once the named volume is created, we can mount it at the time of container creation using: `docker run -d -p 8080:80 -v <volume-name>:<container-path> nginx`

### How to use with docker compose?

```Dockerfile
volumes:
  mongodb:

services:
  mongodb:
    image: joindevops/mongodb:v1
    container_name: mongodb
    networks:
      - roboshop
    volumes:
    - source: mongodb
      target: /data/db
      type: volume
```

## Best Practices to implement in Docker

### Reduce Docker image size

- Reduce the size of the docker image which can be done by using a smaller base image such as **alpine**
- This conclusion should only made after performing proper testing of the whole application
- With python based applications, just by changing the image will not work rather we need to also add commands to install necessary dependencies for the application to work properly

### Multi-stage build

### Limit Privileges

- Avoid running containers as a Root user whenever possible
- We need to create system group and system user using:

```Dockerfile
RUN addgroup -S cart && adduser -S cart cart
```
