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

  - If ARG variable is defined before FROM version, it can be used only for FROM instruction only
  - In this case, we should pass the value to the variable from the terminal: `docker build -t arg:v1 --build-arg version=9 .`
  - We can also specify default values to the arguments using `:-8` as follows:

  ```Dockerfile
  ARG version
  FROM almalinux:${version:-8}
  ```

  - In this case, if the user doesn't provide a value at the build time almalinux v8 will be installed by default
  - **Difference between ARG and ENV**: ARG variables will work only at build time whereas ENV variables can work at build and runtime as well
  - Therefore, the best way is to use a combination of both i.e. we can pass values to the ENV variables using ARG variables as follows:

  ```Dockerfile
  ARG version
  FROM almalinux:${version}
  ARG COURSE
  ENV COURSE=${COURSE}
  ARG TRAINER
  ENV TRAINER=${TRAINER}
  RUN echo "Course: ${COURSE}, Trainer: ${TRAINER}"
  ```

  - We can pass the variables at build time using: `docker build -t arg:v1 --build-arg version=9 --build-arg COURSE=DevOps --build-arg TRAINER=Sivakumar .`
  - Similarly we can also create a user at the container runtime

  ```Dockerfile
  FROM almalinux:8
  ARG username
  RUN useradd ${username}
  USER ${username}
  ```

  - Now, we should build the image using: `docker build -t arg:v1 --build-arg username=pavan .`

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

  `roboshop-docker/mongodb/Dockerfile`

  ```Dockerfile
  FROM mongo:5
  COPY *.js /docker-entrypoint-initdb.d/
  ```

- We can build the image using: `docker build -t mongodb:v1 .`
- Once the image is build, we can use: `docker run -d --name mongodb mongodb:v1` to create the container
- Once the container is created, we can check the logs using: `docker logs mongodb`

  ```text
  {"t":{"$date":"2024-02-12T16:40:32.558+00:00"},"s":"I",  "c":"NETWORK",  "id":23016,   "ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}
  ```

- If we see an output similar to the above with the message: `"Waiting for connections"`, it means its working perfectly fine.

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

- We can build the image using: `docker build -t catalogue:v1 .`
- Once the image is build, we can use: `docker run -d --name catalogue catalogue:v1` to create the container
- Once the container is created, we can check the logs using: `docker logs catalogue`
- When we check the logs, we got the following message:

  ```text
  {"level":"error","time":1707776727871,"pid":1,"hostname":"e20bf67a46d8","msg":"ERROR {\"name\":\"MongoNetworkError\"}","v":1}
  ```

- This means catalogue component is unable to communicate with MongoDB

## Docker Networking

- **docker0** network interface is created by default at the time of docker installation
- When we inspect both the containers, we see that the docker0 has assigned:
  - 172.17.0.2 to MongoDB
  - 172.17.0.3 to Catalogue
  - Gateway IP of 172.17.0.1
- Disadvantage with default bridge network i.e. eth0 is: containers cannot communicate with each other i.e. with names
- There are 2 kinds of networks that Docker offers
  - Host
  - Bridge (default)
- Therefore we need to create our own network
- This can be done using: `docker network create roboshop`
- When we inspect this new network, a Gateway IP of 172.18.0.1 is assigned to it
- To create docker containers with a network:
  - For mongodb: `docker run -d --name mongodb --network=roboshop mongodb:v1`
  - For catalogue:  `docker run -d --name catalogue --network=roboshop catalogue:v1`
- As they both are in the same network, catalogue is able to communicate with MongoDB successfully as shown below:

  ```text
  {"level":"info","time":1707777534788,"pid":1,"hostname":"54753a9c85fe","msg":"Started on port 8080","v":1}
  {"level":"info","time":1707777534798,"pid":1,"hostname":"54753a9c85fe","msg":"MongoDB connected","v":1}
  ```
