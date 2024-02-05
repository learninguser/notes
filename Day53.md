# Containerization: Docker (Continued)

## Further instructions

- **USER**:
  - By default docker containers gets the root access to the underlying server which is not good
  - This can be restricted and secure our containers using the USER instruction
  - First we should create the user in the docker container
  - After that, we can add `USER <username>` instruction so that all the other steps would be executed with that user privilege

  ```Dockerfile
  FROM almalinux:8
  RUN useradd roboshop
  USER roboshop
  RUN echo "Hello Docker" > /tmp/hello.txt
  ```

- **WORKDIR**:
  - When we add the instruction
  
  ```Dockerfile
  FROM almalinux:8
  RUN cd /tmp
  RUN echo "Hello Docker" > hello.txt
  ```
  
  - When building the image using the above Dockerfile, docker changes the directory but the file will be created in the `/` path
  - Rather, we should use `WORKDIR /tmp` as it sets the current working directory and creates the file in the desired location
  - Also when we enter the container, Docker by default enters into that path defined in WORKDIR

- **ARG**:
  - It supplies variables at **build time** like input arguments to a function
  - In a Dockerfile, **FROM** is the first instruction with an exception that **ARG** can also be the first instruction
  
  ```Dockerfile
  ARG version
  FROM almalinux:${version}
  ```

  - ARG variable before FROM version, can only be used for FROM instruction only
  - In this case, we should pass the value to the variable from the terminal: `docker build -t arg:v1 --build-arg version=9 .`
  - We can also specify default values to the arguments using `:-8` as follows:

  ```Dockerfile
  ARG version
  FROM almalinux:${version:-8}
  ```

  - In this case, if the user doesn't provide a value at the build time almalinux v8 will be installed by default
  - Difference between **ARG** and **ENV**: ARG varaibles will work only at build time whereas ENV variables can work at build and runtime as well
  - If we want to pass the ARG variables to the ENV variables, we can do it as follows:

  ```Dockerfile
  ARG version
  FROM almalinux:${version}
  ARG COURSE
  ENV COURSE=${COURSE}
  ```

- **ONBUILD**:
  - Using this, we can add some instructions to the users who are using our build images
  - ONBUILD instructions will not execute at the time of image build

  ```Dockerfile
  FROM almalinux:8
  RUN yum install nginx -y
  RUN rm -rf /usr/share/nginx/html/index.html
  ONBUILD ADD index.html /usr/share/nginx/html/index.html
  CMD [ "nginx", "-g", "daemon off;" ]
  ```

  - With this as an image developer, we build the image using: `docker build -t onbuild:v1 .`
  - When someone wants to use it, they can do it as follows:

  ```Dockerfile
  FROM onbuild:v1
  ```

  - Only at this time, the ONBUILD instructions are executed
  - They should ensure that they have all the necessary files for building their images successfully on top of our base image

## Roboshop on Docker

### Web

- We should use the official base image when available for e.g. NGiNX so that we don't need to manage it when ever there are any security updates and fixes

`roboshop-docker/web/Dockerfile`

```Dockerfile
FROM nginx
RUN rm -rf /usr/share/nginx/html/*
COPY static /usr/share/nginx/html/
COPY roboshop.conf /etc/nginx/nginx.conf
```

### MongoDB

- We can use the official Mongo image
- By default docker can initialise and load the .js files that are present inside the `/docker-entrypoint-initdb.d/` directory

`roboshop-docker/catalogue/Dockerfile`

```Dockerfile
FROM mongo:5
COPY *.js /docker-entrypoint-initdb.d/
```

### Catalogue

- We just need 2 files i.e. `package.json` and `server.js` to build the catalogue image

`roboshop-docker/catalogue/Dockerfile`

```Dockerfile
FROM node:14
EXPOSE 8080
WORKDIR /opt/server
COPY package.json .
COPY server.js .
RUN npm install
ENV MONGO=true
CMD ["node", "server.js"]
```

## Docker Networking

- There are 2 kinds of networks that Docker offers
  - Host
  - Bridge (default)
- **docker0** network interface is created by default at the time of docker installation
- Disadvantage with default bridge network i.e. eth0 is:
  - The docker containers cannot communicate with each other i.e. with names
- Therefore we need to create our own network
- This can be done using: `docker network create roboshop`
- To create docker containers with a network: `docker run -d --name mongodb --network=roboshop mongodb:v1`
