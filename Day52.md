# Containerization: Docker (Continued)

## How to create our own Docker Images

- **FROM**: Fetch the Base OS
  - `FROM almalinux:8`
- **RUN**: To install the packages or configure the Base OS
  - **It runs at image creation**
- **CMD**: To run the container for infinite time
  - Without a CMD instruction, a container cannot be created
  - A command should run in the foreground, attached to the screen and then send it to the background
- **LABEL**:
  - These are useful for filtering purpose
  - Adds metadata to the image
  
  ```Dockerfile
  FROM  almalinux:8
  LABEL Course=DevOps \
        Trainer=SivaKumar \
        Duration=100HRS
  ```

- For e.g. `docker images --filter label=Course=DevOps`
- To check the metadata of the image, we can use: `docker inspect <image-id>`
- **EXPOSE**: We can specify the ports that the container is using
  - By default, the ports that are being used within the container can't be seen even with `docker inspect <image-id>`
  - Using **EXPOSE**, the user can see the list of exposed ports by inspecting it
  - It doesn't add any functionality rather can be used as a documentation
- **ENV**: Environment Variables

  ```Dockerfile
  FROM  almalinux:8
  ENV   COMPONENT=MONGODB \
        USERNAME=roboshop
  ```

- To view the list of environment variables available with in a image: `docker run <image-id> env`
  - This command lists all the environment variables in the image and exits
- Whereas `docker run almalinux ping www.google.com` runs for infinte time until it is terminated
- To send only 10 packages with ping: `ping -c 10 www.google.com`
- **COPY** - To copy files from local machine to container
  - **Files should be present within the same location of Dockerfile**

  ```Dockerfile
  FROM  nginx
  RUN   rm -rf /usr/share/nginx/index.html
  COPY  index.html /usr/share/nginx/index.html
  ```

- **ADD** is similar to **COPY** i.e. copy files from local machine to container
- But ADD offers 2 extra options
  - Feature 1: It can fetch the content from internet
  - Feature 2: It can unzip the archive

  ```Dockerfile
  FROM almalinux:8
  ADD  hello.txt /tmp/
  ADD  https://raw.githubusercontent.com/sivadevopsdaws74s/dockerfiles/master/commands.MD /tmp/commands.MD
  ADD  sample-1.tar /tmp/
  ```

- **ENTRYPOINT**: It does the similar task as CMD i.e. runs at the time of Container creation

  ```Dockerfile
  FROM almalinux:8
  RUN yum install nginx -y
  RUN rm -rf /usr/share/nginx/html/index.html
  RUN echo "Hello, Welcome to Dockerfile. A way of creating own images" > /usr/share/nginx/html/index.html
  ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
  ```

- **CMD** instruction can be **overridden** whereas with **ENTRYPOINT** it **cannot** be overridden
- If we try to execute a command at the time of creating the container, it appends to the instruction in the ENTRYPOINT
- Its always best to use a combination of both CMD and ENTRYPOINT

  ```Dockerfile
  FROM       almalinux:8
  CMD        [ "google.com" ]
  ENTRYPOINT [ "ping", "-c5" ]
  ```

- If the user is not providing any argument at container runtime, CMD appends the instructions to the ENTRYPOINT

## Push image to Docker Hub

- First we should login into dockerhub: `docker login`
- Next we should tag the image `docker tag <image>:<version> <URL>/<username>/<image>:<version>`
- Finally, `docker push <username>/<image>:<version>`
