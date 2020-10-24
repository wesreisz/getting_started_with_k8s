---
title: "Docker Compose"
full_title: "Lesson 6: Multi-stage & Docker Compose"
weight: 8
date: 2020-10-23T1:00:00-00:95
draft: true
---

### Multistage Build

With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image. To show how this works, let’s adapt the Dockerfile from the previous section to use multi-stage builds.

It is used for great clarity, security, and image size. NOTE: Scatch is the smallest possible container you can start with.


```Dockerfile
FROM golang:1.14

RUN git clone https://github.com/coredns/coredns.git /coredns
RUN cd /coredns && make

FROM scratch
COPY --from=0 /coredns/coredns /coredns

EXPOSE 53 53/udp
CMD ["/coredns"]
```

### Docker Compose
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the list of features.

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in Common Use Cases.

Using Compose is basically a three-step process:

1. Define your app’s environment with a Dockerfile so it can be reproduced anywhere.

1. Define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.

1. Run docker-compose up and Compose starts and runs your entire app.

A docker-compose.yml looks like this:

```yml
version: '3'
services:
  azure-vote-back:
    image: redis
    container_name: azure-vote-back
    ports:
        - "6379:6379"

  azure-vote-front:
    build: ./azure-vote
    image: azure-vote-front
    container_name: azure-vote-front
    environment:
      REDIS: azure-vote-back
    ports:
        - "8080:80"
```

This is a powerful workflow:
```bash
git clone https://github.com/wesreisz/azure-voting-app-redis.git
cd azure-voting-app-redis
docker-compose up
```

### Features
The features of Compose that make it effective are:

* Multiple isolated environments on a single host
* Preserve volume data when containers are created
* Only recreate containers that have changed
* Variables and moving a composition between environments
* Multiple isolated environments on a single host
* Compose uses a project name to isolate environments from each other. 

You can make use of this project name in several different contexts:

* on a dev host, to create multiple copies of a single environment, such as when you want to run a stable copy for each feature branch of a project
* on a CI server, to keep builds from interfering with each other, you can set the project name to a unique build number
* on a shared host or dev host, to prevent different projects, which may use the same service names, from interfering with each other

### Common use cases
Compose can be used in many different ways. Some common use cases are outlined below.

**Development environments**

When you’re developing software, the ability to run an application in an isolated environment and interact with it is crucial. The Compose command line tool can be used to create the environment and interact with it.

The Compose file provides a way to document and configure all of the application’s service dependencies (databases, queues, caches, web service APIs, etc). Using the Compose command line tool you can create and start one or more containers for each dependency with a single command (docker-compose up).

Together, these features provide a convenient way for developers to get started on a project. Compose can reduce a multi-page “developer getting started guide” to a single machine readable Compose file and a few commands.

**Automated testing environments**

An important part of any Continuous Deployment or Continuous Integration process is the automated test suite. Automated end-to-end testing requires an environment in which to run tests. Compose provides a convenient way to create and destroy isolated testing environments for your test suite. By defining the full environment in a Compose file, you can create and destroy these environments in just a few commands:

```bash
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```

**Single host deployments**

Compose has traditionally been focused on development and testing workflows, but with each release we’re making progress on more production-oriented features.

For details on using production-oriented features, see compose in production in this documentation.