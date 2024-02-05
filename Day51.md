# Containerization: Docker

- In Roboshop project, we don't servers with heavy configuration for deploying user, cart, catalogue, shipping and dispatch components
- Therefore we can pack these individual components and create images from them
- Using these images, we can create containers that are light in weight and faster as well in terms of exeuction
- Bare Metal servers are physical servers
- With hypervisor on the Bare Metal servers, we can have virtualisation i.e. logically divide the server into multiple VMs
- Examples for hypervisor: Oracle Virtual Box, VMWare etc
- Therefore, the technology behind cloud is VM
- Using VMs we can improve resource utilisation

## Installation steps for Docker on CentOS

- Step 1: `sudo yum install -y yum-utils`
- Step 2: `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
- Step 3: After installing Docker, we need to add the current user to docker group that is created at the time of installation `sudo usermod -aG docker $USER`

## Useful commands in Docker

- `docker images` - list down the images on the server
- `docker pull <image-name:version>` - Download the image from Docker Hub i.e. where all the images available
  - E.g.: `docker pull nginx:latest`
    - Pulls the image of the latest version of Nginx
    - The underlying OS can be anything and on top of it NGiNX is installed and published on Docker Hub
- **Container**: A running version of the image
- To **create a container**: `docker create <image-id>`
- To list all the running containers: `docker ps`
- To list all the containers i.e. running, stopped and exited: `docker ps -a`
- To **start** a container: `docker start <container-id>`
- To **stop** a container: `docker start <container-id>`
- Pull + create + start can be performed with one command: `docker run <image-name>`
- To list all the image IDs: `docker images -a -q`
  - `-a`: All the images
  - `-q`: Image IDs only
- To remove a particular image: `docker rmi <image-id>`
- To remove all the images:

    ```bash
    docker rmi `docker images -a -q`
    ```

- To remove all the containers:

    ```bash
    docker rm `docker ps -a -q`
    ```

- To access the NGiNX running inside the container, we need to perform **port forwarding**
- Command: `docker run -p <host-port>:<container-port> <image-id>`
  - But this command runs in the attached mode i.e. foreground
  - To run it in background: `docker run -p -d <host-port>:<container-port> <image-id>`
- To enter into the container: `docker exec -it <container-id> <command>`
- To name a container: `docker run -p --name <name-of-the-container> -d <host-port>:<container-port> <image-id>`
- To check the logs of the container: `docker logs <container-id>`

## How to create our own Docker Images

- This can be done using **Dockerfile** which is a declarative way of creating images
- Using this we can build images and push them to a repository
- **FROM**: Fetch the Base OS
  - `FROM almalinux:8`
- To build a docker image: `docker build -t <image-name>:<version> .`
- **RUN**: To install the packages or configure the Base OS
  - `RUN yum install nginx -y`
  - **It runs at image creation**
- **CMD**: To run the container
  - It runs only at container creation
  - A Dockerfile can have multiple RUN instructions but only ONE CMD instruction
  - **systemctl** commands are not present in container
  - `CMD [ "nginx", "-g", "daemon off;" ]`: To start the NGiNX as a service in the container
    - Using `daemon off`, the container runs in the foreground
  - Without a CMD instruction, a container cannot be created
