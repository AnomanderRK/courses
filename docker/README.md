# docker lifecycle
## Commands:
	- docker ps -> see containers
	- docker inspect --format '{{.State.Pid}}' [name] -> check pid for container [name] 
	- kill -9 [id]	-> kill process in ubuntu


- Run container

	As long as the main process is running in the container, the container will be alive
	docker run --name alwaysup -d ubuntu tail -f /dev/null
		-d -> detached
		tail -f /dev/nell -> command
- Execute command in a running container

	docker exec -it alwaysup bash

# Exposing containers
## Commands
	- docker run -d --name nginx -> run ngix on the background [-d]
	- docker stop [name]	-> stop container
	- docker rm [name]		-> remove docker container (even if it has status = Exited(0))
	- docker logs [name]	-> see logs from container
		- docker logs -f [name]	-> see logs as they come
		- docker logs --tail [num] [name]	-> see just [num] of logs


- map ports from containers to host ports
							   host:container
	docker run -d --name proxy -p 8080:80 nginx

# Bind mounts
Allow apps running in docker containers to access exernal files

    mkdir mongodata	-> directory to store data
    pwd -> see current directory
    docker run -d --name [container] -v [local path]:[container path] [image]

# Volumes
Another way, more secure, of sharing data within containers. 
The data stored in volumes is only accessable by containers but it is permanent. If we stored something there it will persist even if we delete the container and it will be there if we, later atach another container

    docker volume ls 						-> see volumes
    docker volume create [vol_name]	-> create volume
    docker run -d --name [name] --mount src=[vol_name],dst=[dir in container] [image]	-> attach volume
    docker exec -it [name] bash

# Extract data from containers
We can extract/insert data from/to a container even if we are not using bind mounts or volumes

## Example
### from local machine to container
    touch test.txt
    docker run -d --name copytest ubuntu tail -f /dev/null
    docker exec -it copytest bash
    mkdir testing_dir
    exit
    docker cp test.txt copytest:/testing_dir/test.txt
    docker exec it copytest bash
    cd testing
    ll 	-> test.txt should be here!
    exit  -> exit container

### From container to local machine
    docker cp copytest:/testing_dir local_dir -> copy to local_dir from copytest [container]/folder to local_dir
    ll -> new local_dir should be here
 
 # Images
 Solves build and distribution of software
 It contains all that is needed for a container to be executed
 Templates to creae new containers

     docker image ls -> See images
     docker pull [image] -> pull image from docker hub (default)

## Build our own image
We need a config file which describes how to build the image

### Create and share images
    mkdir images
    cd images
    touch Dockerfile	# Create config file
    code . [open directory with vs code - optional]

Docker file content example

````yaml
FROM ubuntu:latest	# We need a base. In this case ubuntu
RUN touch /usr/src/hola-platzi.txt	# Run command in build time

````

    docker build -t ubuntu:platzi .
        build image ubuntu[name]:platzi[tag]  using current directory
    docker run -it ubuntu:platzi -> (optional) create container 
    docker login -> Access with docker hub credentials
    docker tag ubuntu:platzi [user]/[name]:[version]
        We cannot push directly to ubuntu because the repo is not ours. We need to tag it with our user
        class example: docker tag ubuntu:platzi gvilarino/ubuntu:platzi
    docker push [user]/[name]:[version]

#### Docker dive
Tool to see layers from image build
- install from [here](https://github.com/wagoodman/dive) 
- dive [name]:[version]



# Developing with docker
docker file example


```yaml
FROM node:12

COPY ["package.json", "package-lock.json", "/usr/src/"]	# it is a good practice to separate dependencies from code

WORKDIR /usr/src 	# Ser working dir

RUN npm install      # install node dependencies

COPY [".", "/usr/src/"]	# Copy the remaining files after installing dependencies from current directory

EXPOSE 3000   		# Access this container through ´this port

CMD ["node", "index.js"]	# Default command to run when we run the container created from this image
```

    docker build -t examplebuild .
    docker run --rm -p 3000:3000 exampleapp 	# run container and delete it after finish and map ports

## Using layer cache to build images
Docker uses cache from layers to speed up build but if we modify 

# Docker networking
Share information between containers
If we have two services that interact between each other, the ideal case is to run two different containers

    docker network ls 	-> see networks
    docker network create --attachable testnet	-> create testnet and is attachable (containers can be attached)
    docker network inspect testnet -> inspect network
    docker network connect testnet db 	-> connect db container to platzinet
    docker run -d --name app -p 3000:3000 --env VARIABLE=value [image]
        run app with image and set environment variable to value
    docker network connect testnet app -> connect app to testnet 

# Docker compose
Tool that helps us to connect multiple containers easier and faster
Describes architecture of services our app needs in a configuration file

## docker-compose.yml
docker-compose.yml is the configuration file used to define the services we are using in our app

```yaml
version: "3.8"

services:
	service_1:
		image: platziapp
		environment:
			VARIABLE: "somevalue"
		depends_on:
			- service_2
		ports:
			- "3000:3000"
	service_2:
		image: mongo

```

### Run it

    docker-compose up -d	# run compose detached
        Done!! it will take the configuration file to set up the containers
    docker-compose -f logs [service]	-> see logs from services
    docker-compose exec [service_name] [command]
        docker-compose exec app bash
    docker-compose ps -> see services
    docker-compose down 	-> set all services down

## Docker-compose as development tool

Instead of using an image, we can build directly from the configuration file
**Notice build:.**

```yaml

version: "3.8"

services:
	service_1:
		build: .
		environment:
			VARIABLE: "somevalue"
		depends_on:
			- service_2
		ports:
			- "3000-3001:3000"	# range of ports
		volumes:		# update service when a change is detected- 
			- .:/usr/src 	# mount what we have in the current dir to usr/src
			- /usr/src/node_modules	# don't modify this dir in container (where dependencies are installed for node)
		command: npx nodemon index.js # This library will check any change in our code. It is from node. Will it work with a python application? 

	service_2:
		image: mongo

```

### Run it

    docker-compose build
        optional -> we can build specific services
            docker-compose build [service_1]
    docker-compose up -d
    docker-compose ps 	-> see services (optional)

## docker-compose.override.yml
If we need to constantly modify or experiment with our compose configuration but we don't want to have "issues" with git, we can add another file to add this changes while the original compose file remains the same 

- touch docker-compose.override.yml

docker-compose.override.yml example

Modify specific sections

```yaml
version: "3.8"
services:
	service_1: 
		image: platziapp	# change

```

    docker-compose build
    docker-compose up
    docker-compose down

# Managing docker environment

How to manage our environment to have a clean docker

    docker container prune -> Delete all stopped containers
    docker rm -f $(docker ps -aq) -> remove all contianers
        docker ps -aq	-> returns all containers' ids
    docker network prune 	-> delete unused networks
    docker volume prune 	-> delete unused volumes
    docker system prune 	-> delete unused containers, images, volumes, etc

## Set resources for containers

    docker run -d --name app --memory 500m name

Limit memory to 500MB in this case. 
**WARNING**: a container might stop for low memory error!

## Stop containers in the right way

    docker stop [name]	-> send systerm signal to the process. If the process does not respond after some time it sends syskill to delete the process (best)
    docker fill [name]	-> send syskill (not ideal)
    docker rm [name]

# Run containers as binary programs

It can be usefull if want our container to do some tasks just by run it. We can configure our images to receive parameters to make this task easier

modify docker file

```yaml

FROM ubuntu:trusty
ENTRYPOINT ["/bin/ping", "-c", "3"] # send 3 pings to [param from run]
CMD ["localhost"] # if no param is passed take localhost  

```

    docker build -t ping . -> build image ping
    docker run --name pinger ping -> send ping to localhost
    docker rm -f pinger	-> remove container
    docker run --name pinger ping google.com 	-> ping google
