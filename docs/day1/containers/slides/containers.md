###  Containers

A container is an isolated and limited context, that contain an app and all its dependencies to be executed.
- It generates a new namespace in the OS kernel.
- It has a host OS shared kernel.
- It has own resources of cpu, memory, network and storage.
- It executes only one app, ideally.
- It contains all needed (basic os and dependencies) to execute the app.
- It would die once the principal app get stopped.

---

####  Container runtime
Principal container runtime in the market

<img src="day1/containers/slides/images/containers.jpg", style="width:700; height:auto; background-color:white; float:center;"/>

---

####  Containers vs VMs

- Container = context (shared kernel) vs VM = virtualization (hypervisor)
- Container has near native performance. Almost NO overhead.
- Containers have all software requirement and dependencies needed to run the app.
- Containers are portable, versionable and immutable.

---

### Docker
Docker is a container ecosystem used to develop, deploy and execute dockerized app's. 
- Docker Engine: API and client docker components.
- Docker Trusted Registry: Docker repository where save, push and pull docker images.
- Docker Hub: Docker registry official service. Could code autobuilds from github and bitbucket.
- Docker Machine: Tool to deploy hosts in distinct providers.
- Docker Compose: Tool to deploy multi docker services.

---

#### First steps

- Docker service can run in every linux distro (debian, ubuntu, coreos, centos, rancheros,...)
- Docker daemon listen by default in a unix socket /var/run/docker.sock. Also can listen in a network ports 2375 and/or 2376 (ssl).
- Local: Executing in a machine running the daemon.
- Remote: Executing from a remote machine not running the daemon.  

```
export DOCKER_HOST=tcp://<host>:<port>; docker <command> || 
docker -H tcp://<host>:<port> <command>
```

---

#### Commands

- Life cycle : create, rename, run, rm, update, images, import, build, commit, rmi, load, save......
- State : start, stop, restart, pause, unpause, wait, kill, attach...
- Information : ps, logs, inspect, events, port, top, stats, diff, tag....
- Import/Export : cp, export...
- Execution : exec...

---

#### Dockerfiles

- A Dockerfile is the script to build a docker image. 
- Docker images are cumulative and incremental. 
- Every Dockerfile has a FROM section, to specify what’s the "base" image it comes from. 
- Many distributions and applications, already have docker images published in dockerhub. 
- We could use them as FROM to add or customize for our use. 

---

#### Images

Image workflow:

Dockefile -> docker build -> docker tag -> docker push -> docker rmi

- Building a Dockerfile, generates a docker image. 
- Every build, generates a new and unique docker image. 
- Every image should be tagged and published in a docker registry (public or private), to be used. 

---

#### Dockers

docker workflow:

docker pull -> docker run -> docker stop -> docker rm 

- A docker container is an deployed instance of a docker image. 
- Pulling an image from a docker registry, download the docker image to your host.
- Running an image, start an instance of your docker image.
- Every single deployed docker is unique and has state (started, stopped, created, error,...)

---

### Examples

---

#### Docker as client app

- A user wants to use curl in his computer.
- Curl package for his specific OS doesn’t exist.
- The user could execute dockers.

We could build a docker that executes curl as a client. 

---

<!-- .slide: style="text-align: left;"> --> 
- Generate a Dockerfile

```
FROM docker.io/alpine:3.5   # Dockers hierarchy

#Install curl and some basic packages, and remove packages cache
RUN apk add --update bash libressl curl && \ 	 
    rm -rf /var/cache/apk/* 

# Default command when docker run: $ENTRYPOINT + $CMD
ENTRYPOINT [“/usr/bin/curl”]	
```

---

<!-- .slide: style="text-align: left;"> --> 
- Build and tag the docker image. Generate a unique, portable and reproducible version.

```
# Generates image with a uuid
docker build  -f Dockerfile-curlApp .   
docker tag <IMAGE_UUID> <MY_DOCKERHUB_USER>/curlApp:<VERSION>
```

- Or in one command

```
docker build -t <MY_DOCKERHUB_USER>/curlApp:<VERSION> \
  -f Dockerfile-curlApp .
```

---

<!-- .slide: style="text-align: left;"> --> 
- Login to docker hub with your user/pass if needed.

```
docker login -u <MY_DOCKERHUB_USER> -p <MY_DOCKERHUB_PASS>
```

- Push the image to a docker registry to make it available.

```
docker push <MY_DOCKERHUB_USER>/curlApp:<VERSION>
```

---

<!-- .slide: style="text-align: left;"> --> 
- Launch an instace of our dockerized curl app. $CMD would be passed as curl args.

```
docker run -it --rm <MY_DOCKERHUB_USER>/curlApp:<VERSION> \
  -L www.google.com
```

- Making it nicer: Set an alias in the user computer and he/she could launch regular curl.

```
alias curl='docker run -it --rm \
  <MY_DOCKERHUB_USER>/curlApp:<VERSION> '

curl -L www.google.com
```

---

#### Docker as server 

- A user wants a local web server to publish static files.
- The user don't know about install and configure it.
- The user could execute dockers.

- We could build a docker that 
  - executes an nginx server.
  - copy the static files and serve them. 

---

<!-- .slide: style="text-align: left;"> --> 
- Option 1: Build only one docker with nginx and the files to serve.

```
FROM docker.io/alpine:3.5 		  # Dockers hierarchy

#Install nginx and some basic packages and remove packages cache
RUN apk add --update bash libressl nginx && \
    rm -rf /var/cache/apk/*     

# Copy files into the image
ADD <html_files> /var/www/html 	

# Expose the service by a network port
EXPOSE 80		                    

# Default command when docker run: $ENTRYPOINT + $CMD
ENTRYPOINT ["nginx", "-g", "daemon off;”]	
```

---

<!-- .slide: style="text-align: left;"> --> 
- Option 2: Build two docker, to take profit of docker hierarchy

  - Dockerfile for myNginx. Could be used for any other web service.

```
FROM docker.io/alpine:3.5  # Dockers hierarchy

#Install nginx and some basic packages and remove packages cache
RUN apk add --update bash libressl nginx && \
    rm -rf /var/cache/apk/* 	

# Expose the service by a network port
EXPOSE 80		                  

# Default command when docker run: $ENTRYPOINT + $CMD
ENTRYPOINT ["nginx", "-g", "daemon off;”]	
```

---

<!-- .slide: style="text-align: left;"> --> 
- Dockerfile for myServer

```
FROM <MY_DOCKERHUB_USER>/myNginx:<VERSION>  # Dockers hierarchy

# Copy files into the image
ADD <html_files> /var/www/html  
```

---

<!-- .slide: style="text-align: left;"> --> 
- Build and tag the docker images. Generate a unique, portable and reproducible version.
  - Option 1

```
docker build -t <MY_DOCKERHUB_USER>/myServer:<VERSION> \
  -f Dockerfile-myServer1 .
```

  - Option 2: You should build and publish myNginx image before.

```
docker build -t <MY_DOCKERHUB_USER>/myNginx:<VERSION> \
  -f Dockerfile-myNginx .
docker login -u <MY_DOCKERHUB_USER> -p <MY_DOCKERHUB_PASS>
docker push <MY_DOCKERHUB_USER>/myNginx:<VERSION>
docker build -t <MY_DOCKERHUB_USER>/myServer:<VERSION> \
  -f Dockerfile-myServer2 .
```

---

<!-- .slide: style="text-align: left;"> --> 
- Login to docker hub with your user/pass if needed.

```
docker login -u <MY_DOCKERHUB_USER> -p <MY_DOCKERHUB_PASS>
```

- Push the image to a docker registry to make it available.

```
docker push <MY_DOCKERHUB_USER>/myServer:<VERSION>
```

---

<!-- .slide: style="text-align: left;"> --> 
- Launch an instace of our dockerized web server.

```
docker run -td -p 8080:80 <MY_DOCKERHUB_USER>/myServer:<VERSION>
```

- Making it nicer: Set an alias in user computer and he/she could launch his local server. 

```
alias MyServer='docker run -td -p 8080:80 \
  <MY_DOCKERHUB_USER>/myServer:<VERSION>'

MyServer 

curl http://localhost:8080
```

---

### Practices

---

#### HMS docker hierarchy proposed for the workshop

<img src="day1/containers/slides/images/hms-hierarchy.jpg", style="width:auto; height:500; background-color:white; float:center;"/>

---

<!-- .slide: style="text-align: left;"> --> 
JRE8

- Based on rawmind/hms-base https://hub.docker.com/r/rawmind/hms-base/, build and publish a docker with oracle jre8 installed.
- It will be the execution base for ari service.

Maven

- Based on rawmind/hms-jdk8 https://hub.docker.com/r/rawmind/hms-jdk8, build and publish a docker with maven.
- It will be the builder base, to compile and unitary test for ari service.

---

#### Solutions

- JRE8 
  - Generate a Dockerfile that downloads and install oracle jre8
  - Build and publish the docker.
  - https://github.com/rawmind0/hms-jre8

- MAVEN
  - Generate a Dockerfile that downloads and install maven.
  - Build and publish the docker.
  - https://github.com/rawmind0/hms-maven

---

### References

---

#### Web
- Official Documentation - https://docs.docker.com/
- Docker 101 Tutorial - https://blog.docker.com/tag/docker-101/
- Official Docker Training - https://training.docker.com/
- Microservices architecture : http://microservices.io/index.html
- An introduction to microservices : https://opensource.com/resources/what-are-microservices
- Neal Ford slices: http://nealford.com/abstracts.html

---

#### Books
- Docker:  Up & Running - Karl Matthias y Sean P.Kane (O’Reilly)
- Docker in Action - Jeff Nickolofff (Manning)
- Building microservices - Sam Newman (O’Reilly)

