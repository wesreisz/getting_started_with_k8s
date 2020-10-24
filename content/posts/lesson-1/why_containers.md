---
title: "Why Containers"
full_title: "Lesson 1.1: Why Containers"
weight: 2
date: 2020-10-23T1:00:00-00:98
draft: true
---
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

Container images become containers at runtime and in the case of Docker containers - images become containers when they run on Docker Engine. Available for both Linux and Windows-based applications, containerized software will always run the same, regardless of the infrastructure. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.

Docker containers that run on Docker Engine:

Standard: Docker created the industry standard for containers, so they could be portable anywhere
Lightweight: Containers share the machine’s OS system kernel and therefore do not require an OS per application, driving higher server efficiencies and reducing server and licensing costs
Secure: Applications are safer in containers and Docker provides the strongest default isolation capabilities in the industry

### Learning Objectives
* *Understand*, at the most basic level, what containers are.
* *Understand* the problem that containers solve.
* *Understand* the relationship between the shipping industry and build software containers.
* *Understand* the basic clone / run workflow for running a project. 
* *Able to run* run a docker/docker compose project.


[![Introduction to Containers](/getting_started_with_containerization/images/lesson1/why-containers.jpg)](https://docs.google.com/presentation/d/16xibcNq2qj8n5_unedSWt1PXymiocNkeHLcWRfIuhbU/edit#slide=id.p)


Deploy Voting App
1. Log into your vm
1. clone repo
```bash
git clone https://github.com/wesreisz/azure-voting-app-redis.git
```
3. Deploy a python app that talks to a redis instance.
```bash
docker-compose up
```
![Docker Compose](/getting_started_with_containerization/images/lesson1/1-1-example.png "Docker Compose")

