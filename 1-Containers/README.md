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

## Container commands