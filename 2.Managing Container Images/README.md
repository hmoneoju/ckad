# Container image architecture

* Common system images are the foundation (Containerized Linux distros i.e. busybox)
* In docker file FROM

# Managing Container images

* Before container is started, required images are pulled and stored from the registry
* Prefetched by *docker pull*
* *docker images* for list of stored images
* According to the PullPolicy a new image may be automatically be pulled if it's available
  * Default polociy **Always**
  * In Kubernetes can be set to newer
* To remove unused images *docker image prune*

```
docker images
docker pull alphine
docker image prune
```

# Image create options

* *docker commit* not recommended
* Dockerfile (Containerfile for podman) to build custom image based on components defined in the file
* **builda** advances solution

# Base image

* Minimized image as foundation for building own images
* Fully functional, minimized Linux distro
* Not all tools are always available
  * Common tools like **ps** are often missin, since they are not needed to run the main container application
* Common ones
  * Busybox
  * Alphine
  * Red Hat UBI (Universal Base Image)

# Using Dockerfile

* To build from Docker file **docker build -t imagename .*
* **docker commit** to generate the image from a running container
