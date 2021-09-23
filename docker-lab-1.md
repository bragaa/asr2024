Task 10: Setup Docker engine
==============================
 * to get docker running on our linux host, we have to install both the docker engine (daemon program used to manage images and containers) and the docker client tools (used to interact with the docker engine).
    * first, remove packages from your linux distributin using ``` sudo apt-get purge docker-ce docker-ce-cli containerd.io```
    * follow the instructions in "Set up the repository" section (steps 1, 2, 3) in order to add the docker packages repository. ensure you select the right hardware architecture for you (x86_64/amd64) https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository 
    * update the package index ```sudo apt-get update```
    * get and install the packages ```sudo apt-get install docker-ce docker-ce-cli containerd.io```
 * (optional) you may want to follow instructions from https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user, otherwise, you'll need to use ```sudo``` with each docker command line. try to find out why.
 * test your installation using ```docker version```. find out the versions of the engine and the client tools.



Task 30: A typical containers workflow: dockerize a SpringBoot application
=========================================================================
 Assume you have developed a SpringBoot-based application and want to publish it using the Docker technology, which involves *dockerizing* the application, that is, transforming it into a docker image. In this task, you'll learn how to make this happen.
 This tutorial is based on the official Getting Started tutorial of SpringBoot.
 * clone the source code repository (repo) using git as follows:
   git clone https://github.com/spring-guides/gs-spring-boot.git
 * go to directory gs-serving-web-content/complete/, which contains the complete code we want to dockerize (the original aim of the gs tutorial is about SpringBoot)
 * create the java archive (jar file) using ```maven ./mvn package```
 * start the application ```java -jar target/gs-spring-boot-docker-0.1.0.jar``` which then start listening on localhost:8080. try accessing the application using a browser or using curl (a command line http client, install it using ```apt-get install curl```)
 (test this url: localhost:8080/greeting?name=student)

 * As you have seen, the application depends and on a java runtime environment, the jre, (which comprises the java vm and standard and spring web libraries) in order to run. We will dockerize the application, that is, create an image that would *run* on a operating system without installing any of the dependencies; they will be packaged or embedded into the image. The only requirement is to have a special engine to run an application from an image.

 * create the following file gs-serving-web-content/complete/dockerfile with this content
 ```dockerfile
   FROM openjdk:8-jdk-alpine 
   ARG JAR_FILE=target/*.jar
   COPY ${JAR_FILE} app.jar
   ENTRYPOINT ["java","-jar","/app.jar"]
 ```
 * create the image using ```docker  build -t brahim/gs-springboot-webcontent-1  .  ```

 * check the image is created using ```docker image ls```. every image is given an id (computed as a hash function from the image), a name. you can get further information using ```docker inspect brahim/gs-springboot-webcontent-1```

 * now, let's *run* the application. for this purpose, use ```docker run -d brahim/gs-springboot-webcontent-1``` on a machine having the docker engine installed.

 * find out the containers running on your docker engine using ```docker containers ls -a```. what is the id of the container we've just run? find out its IP address using ```docker container inspect XXXX --format '{{ .NetworkSettings.IPAddress }}'``` where XXXX stands to the id of your container. do not worry, your are not required to type the entire id of the container, just enough digits to allow the engine to select a single one with no ambiguity.

 * try accessing the application using ```curl 172.16.0.2:8080/greeting?name=student``` . Replace 172.16.0.2 with the address you find from docker inspect network.

 * how all that happened? the docker engine created a **container** in which it run the command ```java -jar /app.jar```. the resulting java process runs in the container *isolated*, and can be considered as residing on an independent machine. the container has the following properties:
   - the filesystem used within the container is built from the image and is independent from the one on your host system. it contains the java command, the libraries, and the /app.jar. the main goal of the dockerfile was to tell docker engine how to create the image and what it should include. more on this later..
   - docker engine is able to interconnect containers, connects them to the host or to the external networks, and has for this purpose adequate drivers.. in our case, the docker engine created a TCP/IP stack in the container, a virtual ethernet device and connected it to the default virtual switch. it also assigned it an address. more on this later..

 * Following is a short explanation of the dockerfile steps:
 ```dockerfile
   # this is the base image. it contains openjdk, the version 8 based on linux (alpine) system. the image is created by the Docker community.
   FROM openjdk:8-jdk-alpine 
   # definition of a variable named JAR_FILE
   ARG JAR_FILE=target/*.jar
   # the JAR_FILE is copied and renamed to app.jar in the base image.
   COPY ${JAR_FILE} app.jar
   # definition of the default command to run when the container is run.
   ENTRYPOINT ["java","-jar","/app.jar"]
 ```

 * Here is a short explanation of how docker built the image from the dockerfile: (the lines starting with # are my own comments)
 ```bash
   gcr@debian:~/workspace/gs-serving-web-content/complete$ docker  build -t brahim/gs-springboot-webcontent-1  .
   Sending build context to Docker daemon  19.04MB
   # the build context is the directory containing the dockerfile, in this case the current directory .
   Step 1/4 : FROM openjdk:8-jdk-alpine
   8-jdk-alpine: Pulling from library/openjdk
   e7c96db7181b: Pull complete 
   f910a506b6cb: Pull complete 
   c2274a1a0e27: Pull complete 
   Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
   Status: Downloaded newer image for openjdk:8-jdk-alpine
   # now the engine completed downloading the image. the image is the union of the 3 other distinct images (also called layers).
   # this is the id of the base image
    ---> a3562aa0b991
   # the docker engine created a container (5cc6ecf37077). modifications to its file system incurred by step 2/4 were applied, thus obtaining a newer image (dc4e....). the intermediate container were removed (as we already have the newer image)
   Step 2/4 : ARG JAR_FILE=target/*.jar
    ---> Running in 5cc6ecf37077
   Removing intermediate container 5cc6ecf37077
    ---> dc4ece11a040
   Step 3/4 : COPY ${JAR_FILE} app.jar
   # following the same logic, we now have a newer image containing the app.jar
    ---> b8dee25100da
   Step 4/4 : ENTRYPOINT ["java","-jar","/app.jar"]
    ---> Running in 5ac4421ebfb0
   Removing intermediate container 5ac4421ebfb0
    ---> 91213ad44470
   # we obtained the image 9121.. (check you own id), which also named gs-springboot-webcontent-1
   Successfully built 91213ad44470
   Successfully tagged brahim/gs-springboot-webcontent-1:latest
 ```