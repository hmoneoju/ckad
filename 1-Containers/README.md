# What is a Container

* Containers provide a mechanism to run application in full isolation
* Images are obtained from a registry

## Container runtimes

* The program that starts a container
* **runc** is a common container runtime
* To manage a container that runs on a stand-alone computer, a container engine us used on top of a container runtime
* Most well known container engines for running standalone containers are Docker and Podman
* To run containers in the cloud, Kubernetes is used as the container engine 
* Kubernetes is much more than a container engine, it is a container orchestrator

# Container registry

* Is used as a container image store
* Public registries can be used to provide access to container images
* The Docker registry is the most commong registry
* To run container images from specific registries a FQCN (Fully Qualified Container image Name) can be used at all times 

# Starting containers

* Depends on the container being used.

```
# Docker
docker run -d nginx

# Running containers
docker ps

# Interactive shell
docker run -it busybox
```

# Managing containers

* The docker CLI provide many commands for managing containers
```
# Overview of running containers
docker ps 

# Overview of all containers
docker ps -a 

# Stop a running container
docker stop <<name>>

# List of images available
docker images

# Restart a container that have been used before
docker start <<name>>

# Inspect descripton of a container
docker inspect <<name>>

# Remove a container
docker rm <<name>>

# Kill a container (Do not use)
docker kill <<name>>

# Search images where the name or description of the image contains value 
docker search <<value>>
```

# Container logging

* Do not always connect to STDOUT
* Container application output and errors are logged to teh container logs

```
docker run --name=mydb mariadb
docker ps
docker ps -a
docker logs mydb
```