---
title: "Docker"
full_title: "Lesson 3: Docker"
weight: 3
date: 2020-10-23T1:00:00-00:96
draft: true
---

Docker is a software platform for building applications based on containers. It is the defacto standard for doing containerization; however, it is not the only product out there (LXC, Hyper-V containers, podman, containerd, RUNC are all alternatives). However, we will use Docker in this course.

There's alot of different command here. I'll explore some of the ones you'll use the most command and talk through them. We'll also talk about how do get more help and find additional containers.

### Learning Objectives
* *Download* and *run* docker containers.
* *Understand* the difference between `docker run` and `docker exec`.
* *Explore* how to use `docker image` and `docker container` commands.
* *Understand* how to search dockerhub.


Let's explore:

```bash
docker info
docker help
```

Each container has a RUN command that it executes when you run it. However, you can also call your own command explicitly.
```bash
docker run --interactive --tty ubuntu:bionic
cat /etc/issue
docker image ls
docker run --interactive --tty ubuntu:bionic cat /etc/issue
docker run -it ubuntu:bionic ps aux
```

Let's search and then take a tour of dockerhub, open https://hub.docker.com. Let's pull an image to our local image repository.
```bash
docker search ubuntu
docker pull ubuntu:14.04
docker image ls
docker run -it --name my-u ubuntu:14.04
```

```bash
docker run -detach -it jturpin/hollywood
docker ps
docker ps -a
docker run --dit jturpin/hollywood
```

Understand what commands are available
```bash
docker help
docker image help
docker container help
```

Docker run is for creating containers. It is important to differentiate it from exec , which is used to interact with containers that are already running. Attach let's you connect to the same terminal instance.
```bash
docker run -dit --name my-nginx nginx
docker exec -it my-nginx bash
docker exec -it my-nginx ps aux

docker ps
docker container attach my-nginx
```

Start/Stop/Restarting a container
```bash
docker container ls
docker container ls -a
docker start ?????????????????
docker stop ?????????????????
docker restart ?????????????????
docker ps
```



**Tagging** 

Tagging is done to track and differentiate different versions. It's down with `docker tag <IMAGE_ID> <tag>` or by using a `-t` with the during the build. Using this is called aliasing an image

A tag name must be valid ASCII and may contain lowercase and uppercase letters, digits, underscores, periods and dashes. A tag name may not start with a period or a dash and may contain a maximum of 128 characters.
```bash
docker image ls
docker tag 0810ad403c0f wesreisz/hello-node
docker tag 0810ad403c0f wesreisz/hello-node:v1
docker image ls 
```

```
REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
wesreisz/hello-node                              latest              0810ad403c0f        37 hours ago        918MB
wesreisz/hello-node                              v1                  0810ad403c0f        37 hours ago        918MB
hello-node                                       latest              0810ad403c0f        37 hours ago        918MB

```
&nbsp;


**Docker Hub** 

Docker Hub is a hosted repository service provided by Docker for finding and sharing container images with your team. Key features include: Private Repositories: Push and pull container images. Automated Builds: Automatically build container images from GitHub and Bitbucket and push them to Docker Hub
```bash
docker login
docker push wesreisz/hello-node:v1
```

**Pushing Images** 

There are many other registrations gcr, ecr, and acr acr are the cloud providers. Here's what it looks like to upload to Azures container repo (https://portal.azure.com).
```bash
docker pull nginx
docker login azurealpharegistry.azurecr.io
#or az acr login --name azurealpharegistry.azurecr.io --resource-group initial-manually-created-cluster
docker tag nginx azurealpharegistry.azurecr.io/samples/nginx
docker push azurealpharegistry.azurecr.io/samples/nginx
docker run -it --rm -p 8080:80 azurealpharegistry.azurecr.io/samples/nginx
```



Clean up old container or images
```bash
docker image prune
docker container prune
```