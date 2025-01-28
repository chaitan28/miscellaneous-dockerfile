#### Dockerfile = Environment+Dependencies+Source Code
################################
### Dockerfile 
- **FROM:** Specifies the base image to use for the Docker image.
```sh
 FROM ubuntu:20.04
```
- **ENV:** Sets environment variables.
```sh
 ENV APP_HOME=/usr/src/app APP_PORT=8080
```
- **LABEL:** Adds metadata to the image in the form of key-value pairs.
```sh
  LABEL maintainer="you@example.com" version="1.0"
```
- **USER:** Sets the user for the subsequent instructions and the CMD instruction.
 ```sh
 USER root
```
- **ARG:** Defines a variable that users can pass at build-time to the builder with the docker build command.
```sh
           FROM eclipse-temurin:17-jdk
           ARG JAR_FILE=target/anewtodo*.jar
           COPY ${JAR_FILE} /tmp/app.jar
           COPY target/index.html /tmp/index.html
           ENTRYPOINT ["java","-jar","/tmp/app.jar"]
```
- **WORKDIR:** Sets the working directory for subsequent instructions.
```sh
WORKDIR $APP_HOME
```
- **COPY:**  Copies files or directories from the host to the container.
```sh
  COPY package.json ./ COPY . .
```
- **ADD:**  Copies files from a URL or tar archives and automatically extracts them.
```sh
 ADD https://raw.githubusercontent.com/user/repo/branch/file.txt /usr/src/app/file.txt
```
- **RUN:**  Executes a command during the image build process.
```sh
RUN apt-get update && apt-get install -y curl git && rm -rf /var/lib/apt/lists/*
```
- **VOLUME:**  Creates a mount point with the specified path and marks it as holding externally mounted volumes.
```sh
VOLUME ["/data"]
```
- **EXPOSE:** Informs Docker that the container listens on the specified network ports at runtime.
```sh
EXPOSE $APP_PORT
```
- **CMD:** Provides the command to run within the container when it starts.
```sh
 CMD ["node", "app.js"]
```
- **ENTRYPOINT:** Configures a container that will run as an executable.
```sh
ENTRYPOINT ["docker-entrypoint.sh"]
```
- **HEALTHCHECK:** Informs Docker on how to test the container to check that it is still working.
```sh
HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl -f http://localhost:$APP_PORT/ || exit 1
```
- **STOPSIGNAL:** Sets the system call signal that will be sent to the container to exit.
```sh
 STOPSIGNAL SIGTERM
```
## CMD Command
#### CMD ["executable", "param1", "param2"]:
CMD ["java", "-jar", "app.jar"]
- executable: This is the command or executable that will run inside the container (e.g., node, python, java, etc.) <br>
- param1 and param2: These are arguments passed to the executable.<br>
  Executable: java
  The container will invoke the java command, which is the Java runtime.<br>
  Arguments:
  -jar tells the Java runtime to treat the file at app.jar as a JAR file and run it.<br>
  app.jar is the path to the actual application JAR file.<br>
  java -jar command to run the java application.<br>

## CMD VS ENTRYPOINT 
- CMD can easily be overridden when running the container with different commands.<br>
```sh
root@ip-172-31-37-89:~/dockerfile/cmd# cat Dockerfile
FROM ubuntu
CMD ["echo", "Hello, world!"]
root@ip-172-31-37-89:~/dockerfile/cmd# docker run cmd echo hello new
hello, new
The container will then execute the command echo "hello, new" instead of the default echo "Hello, World!"
```
- The ENTRYPOINT instruction sets the command that will be executed when the container starts, and it cannot be easily overridden by docker run. 
   However, you can pass additional arguments to it when running the container.<br>
```sh
root@ip-172-31-37-89:~/dockerfile/entry# cat Dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
root@ip-172-31-37-89:~/dockerfile/entry# docker run entry world
Hello world
```

## Best Practices for Docker image creation/Best Practice Writing a Dockerfile <br>

-  Use specific tags:                   Avoid latest to ensure predictable builds and compatibility. <br>
-  Optimize image size:                 Use minimal base images and multi-stage builds to reduce the image footprint. <br>
```sh
                                           ## Mutlistage Dockerfile ###
                                             ###stage1: Build######
                                            FROM schoolofdevops/maven:spring AS build
                                            workdir /app
                                            COPY ..
                                            RUN mvn spring-javaformat:apply && \ mvn package -Dskiptests
                                            ####stage2: deploy ####
                                            FROM openjdk:8u201-jre-alpine3.9 AS Runtime
                                            WORKDIR /run
                                            COPY --from=build /app/target/*.jar /run/petclinic.jar
                                            EXPOSE 8080
                                            CMD ["java","-jar","petclinic.jar"]
#### Explaination: 
  . stage1:  This stage contains maven build tool which is responsible to generate the Jar File. use are using mvn package command to get the Jar file 
  . stage2:   In copy, your copying jar file from build stage to the dest /run dir inside the container 
              . if you observe the stage2 only contains the parent/base image nothing. which only responsible for image size.
              . The result of stage2 will be your dockerimage.
```
```sh                                 
                                        ## Mutlistage Dockerfile ###
                                        # Stage 1: Build
                                        FROM node:14 AS build
                                        WORKDIR /usr/src/app
                                        COPY package*.json ./
                                        RUN npm install
                                        COPY . .
                                        RUN npm run build
                                       # Stage 2: Production
                                        FROM nginx:alpine AS Production 
                                        COPY --from=build /usr/src/app/build /usr/share/nginx/html
                                        EXPOSE 80
                                        CMD ["nginx", "-g", "daemon off;"]
                  Explaination:  In this example, the first stage builds the application, and the second stage uses an Nginx server to serve the built application. This approach keeps the final image lean and efficient.
```
-  Exclude unnecessary files:  Use .dockerignore to avoid sending unneeded files in the building the image.
                                        Ignore Git-related files & log files  <br>
                                              .git/  <br>
                                              .gitignore  <br>
                                              *.log  <br>
-  Keep images secure:                  Regularly update base images to include the latest security patches.
-  Scan for vulnerabilities:            Use tools like Trivy or Docker built-in scan before deployment to ensure security compliance.
-  Try to avoid RUN apt get update -y or yum update -y in the Dockerfile.
-  File which so often change should be written at the end of the file. # example : COPY target/index.html /tmp/index.html


### Dockerfile for Python FrameWork
FROM python:3.8-slim                         # Specifies the base image with Python 3.8.  base image will underlying environment in which your application will run               
WORKDIR /app                                 # Sets the working directory inside the container to /app.
COPY ./python1 /app                          # Copies the content of the local directory /python1 to the containers working directory.
RUN pip install -r requirements.txt          # Installs Python dependencies listed in requirements.txt.
CMD ["python", "app.py"]                     # Uses the exec form to specify that the container should run the Python application app.py by default. 
                                             # python app.py is command to run the python script on ubuntu/centos linux machine directly.
                      
Environment= python:3.8-slim
Dependencies= install -r requirements.txt 
Source Code= ./python1 

### Dockerfile for Java Framework
FROM openjdk:8-jdk-alpine                       #  lightweight OpenJDK 8 Alpine-based image, suitable for running Java applications to keep docker image size minimal.
WORKDIR /app                                    #  /app directory is created in the container and All subsequent commands will be run from this directory.
COPY ./target/*.jar /app/app.jar                #  .jar from the target folder on the local machine is copied inside the /app directory inside the container
CMD ["java", "-jar", "/app.jar"]                #   This specifies the command to run when the container starts.CMD and ENTRYPOINT only excutes when container starts .
                                                # java -jar app.jar is command to run the java script on ubuntu/centos linux machine directly.
###Dockerfile for Nodejs Framework

FROM node:14                           # This specifies the base image. We are  using the official Node.js 14 image here. It includes Node.js and npm pre-installed        
WORKDIR /app                           # This sets the working directory inside the container to /app. All subsequent commands will be run from this directory.
COPY package*.json ./                  # This copies the package.json and package-lock.json files to the container. These files contain information about the project dependencies
RUN npm install                        # This installs the dependencies listed in package.json inside the container
COPY . .                               # This copies the rest of your application code (everything in the current directory) into the /app directory inside the container
EXPOSE 3000                            #  This can be ingored . because we expose the application in particular port number. 
                                         By default, all container ports are accessible from the container itself and can be mapped to any port on the host.
CMD ["npm", "start"]                  # node app.js command used to start the nodejs  application on the local linux machine. 
                                         If your application defines a start script in package.json, you can run it using npm start
                                         >>> package.json 
                                         > {
                                              "scripts": {
                                            "start": "node app.js"
                                            }
                                           }


##################################################################################3
&& To list all files in the directory and display details about myfile.txt, you can use a shell form or exec form in CMD

FROM ubuntu:latest
WORKDIR /app
COPY . /app
CMD ["sh", "-c", "ls -la && ls -la myfile.txt"]

CMD ["executable", "param1", "param2"]:

executable: This is the command or executable that will run inside the container (e.g., node, python, java, etc.).
param1 and param2: These are arguments passed to the executable.
###########################################################
RUN apt-get update && apt-get install -y curl && touch /file2.txt  
    Run instruction in a Dockerfile is used to execute shell commands during the build process of the image.
    You don’t need sudo in a Dockerfile, because commands are typically run as the root user by default in a Docker image.
    RUN command in a Dockerfile creates a new layer in the Docker image. This means that frequent RUN commands can increase the size of your image.
###########################################################
# singlestage Dockerfile
FROM golang:1.23
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go
CMD ["/bin/hello"]

#Multistage Dockerfile
FROM golang:1.23
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go
 ####stage2####
FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]

#Multistage Dockerfile
FROM python:3.10-alpine AS build
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .

FROM python:3.10-alpine 
WORKDIR /code
COPY --from=build /code /code
CMD ["flask", "run", "--debug"]