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
- There should be a file called `docker-compose.yaml` for it to work
- Docker compose can create a network, volumes, containers attach them together

  `roboshop-docker/docker-compose.yaml`

  ```yaml
  networks:
    roboshop:
        driver: bridge
  services:
    mongodb:
      image: mongodb:v1
      container_name: mongodb
      networks:
        - roboshop
      catalogue:
        image: catalogue:v1
        container_name: catalogue
        networks:
          - roboshop
        depends_on:
          - mongodb
      web:
        image: web:v1
        container_name: web
        networks:
          - roboshop
        ports:
          - "80:80"
        depends_on:
          - catalogue
  ```

- We should first build the build the images and then only we can use them inside docker compose using `docker compose up -d`
  - `-d` to run it in a detached mode

## MySQL Component

- What ever the scripts that are present inside `/docker-entrypoint-initdb.d` will be loaded into MySQL

  `roboshop-docker/mysql/Dockerfile`

  ```Dockerfile
  FROM mysql:5.7

  ENV MYSQL_ALLOW_EMPTY_PASSWORD=yes \
      MYSQL_USER=shipping \
      MYSQL_PASSWORD=RoboShop@1

  COPY scripts/* /docker-entrypoint-initdb.d/
  ```

## Shipping component

- As its based on Java, every Java application converts the application into Bytecode called as `.jar` file at the time of compilation
- Once we have the Jar file, we can run the application
- For this, we can make use of the **Multi-stage build** concept
- At the time of developing the java application, we need JDK (Java Development Kit) and Maven to build the application
- But when running the application, we just need the JAR file and a run-time environment called JRE (Java Runtime environment) to run it
- JDK = JRE + Development packages because of which JDK is heavy in memory

  `roboshop-docker/shipping/Dockerfile`

  ```Dockerfile
  # Build
  FROM maven AS build
  WORKDIR /app
  COPY pom.xml /app/
  RUN mvn dependency:resolve
  COPY src /app/src/
  RUN mvn package


  # Run

  FROM openjdk:8
  EXPOSE 8080
  WORKDIR /app
  ENV CART_ENDPOINT=cart:8080
  ENV DB_HOST=mysql
  COPY --from=build /app/target/shipping-1.0.jar shipping.jar
  CMD [ "java", "-Xmn256m", "-Xmx768m", "-jar", "shipping.jar" ]
  ```
