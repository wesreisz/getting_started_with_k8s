---
title: "Course Introduction"
full_title: "Lesson 0: Course Introduction and Logistics"
time_estimate: "10 Minutes"
weight: 1
date: 2020-10-23T1:00:00-00:98
draft: true
---

Kubernetes (K8s) is a very powerful tool, primarily concerned with orchestrating and automating the deployment and management of networked applications.

Understanding how Kubernetes works makes you an effective power user for building higher-level platforms. This workshop is aimed at technologists and team leads looking to lift up the hood and understand what are all the major parts of k8s in a fast-paces half-day session. 

### Key Takeaways
1. Understand the major goals and objectives of kubernetes
1. Learn how kubernetes is architected
1. Understand how to deploy and operate a service on kubernetes

### Audience
You should have a familiarity with containers and want to learn more about their orchestration on top of Kubernetes

### Prerequisites
* Familiarity with containers (don't need to be an expert)
* SSH Terminal and basic Linux commandline knowledge
* Amazon Workspace Client installed (https://clients.amazonworkspaces.com)

### Questions we'll answer
Attend if you want to answer the questions below.

* What is the overall architecture of K8s?
* What problems does K8s solve out of the box vs. expecting users to bring a solution for?
* What are the defacto industry standards that Kubernetes has catalyzed?
* What is a K8s controller and how does it work?  
* What kinds of controllers are there, and what roles do they play?  
* What processes run on a K8s master node?
* What processes run on a K8s worker node?
* Exactly what is a pod, or a container in a pod?
* What is a container runtime and what role does it play within a k8s cluster?
* How does K8s schedule and launch a container?
* How does pod security and role-based access control (RBAC) work in Kubernetes?
* How does networking in K8s work?  
* How does K8s integrate with load balancers? Or other cloud resources?

### Agenda

* [Course Agenda](/getting_started_with_k8s/posts/)


### Environment

Each student has been created a virtual desktop enviornment. Everything can be run on your own machine if you have docker, docker-compose, kind, and kubectl installed. As this is only a short 3-hour course, I request that you use the virtual desktop environment if you have any problems locally. This will minimize the support time if things are not setup correctly. Feel free to install and run locally, and use the VDI only if there's an issue.

To access the Amazon VDI, install workplace on your OS. The clients are available here:
https://clients.amazonworkspaces.com

You should have an [email](/getting_started_with_k8s/images/lesson0/email.png) that has
- link create your profile for the workspace
- the registration code to your specific workspace
- your userid that I created for you

Please ask your instructor for a student email account.

Please install the client, register your Ubuntu profile, and launch your VDI (make sure to use the registration code in your email).

![Your Amazon VDI](/getting_started_with_k8s/images/lesson0/desktop.png "Amazon VDI")


### Initial Setup
NOTE: If you ran your workstation in the container workshop, the steps in Initial Setup should already be done. Jump to Configure your system for k8s.

When you start your workspace, Docker most likely will not be running. Check that the daemon is running and if not start it:
```bash
docker info
```
If you see an error, start docker:
```bash
sudo service docker start
```
This takes a minute or two.

At this point, you should be able to run `docker info` and not see an error. If you still see an error, run `groups` and validate that your user is in the docker ground. If it's not, run this command. If this isn't your issue, let the instructor know.
```bash
sudo usermod -a -G docker $USER
```
It won't work until we logoff and log back in, we'll do that in a minute. For now, use `sudo` when you run docker commands on this VM.

```bash
docker run hello-world
```
You should see something like this:
```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```
If so, you're ready to start! If this doesn't work, please install Docker and configure it for your system:
https://docs.docker.com/get-docker


### Configure your system for k8s
We will be using Kind for this workshop.

[kind](https://github.com/kubernetes-sigs/kind) is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

To install kind on this workstation (you can also [install on your local machine](https://kind.sigs.k8s.io/docs/user/quick-start/)), run:
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-linux-amd64
chmod +x kind
mv kind /usr/local/bin/
kind version
```

We will talk about this more later, but to interact with a kubernetes cluster (whether that's a cloud managed cluster, a kind instance, or your own custom built k8s cluster) you will use `kubectl`. Let's install it now as well.

kubectl is a stand alone binary that only needs to be downloaded for the correct operating system and placed in the path. It will look for configuration in the user's home directory on the target OS. 


The process is the same for installing `kubectl`. [If you want to install locally](https://kubernetes.io/docs/tasks/tools/install-kubectl/); otherwise, run:
```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```
```
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"darwin/amd64"}
```

At this point, you should have docker, docker-compose, kubectl, and kind installed. We're ready to begin the course.

Versions:
* kubectl version: v1.19.3 `kubectl version --client`
* Kubernetes version: 1.19.1 `kubectl version`
* docker-compose version: 1.27.4 `docker-compose version`
* Docker version: 19.03.6-ce '19.03.6-ce'

**Kubernetes versioning and cadence**

Kubernetes versions are expressed using semantic versioning (a Kubernetes version is expressed as MAJOR.MINOR.PATCH)

There is a new patch release whenever needed (generally, there is about 2 to 4 weeks between patch releases, except when a critical bug or vulnerability is found: in that case, a patch release will follow as fast as possible)

There is a new minor release approximately every 3 months

At any given time, 3 minor releases are maintained (in other words, a given minor release is maintained about 9 months)

Notes:
* "Validates" = continuous integration builds with very extensive (and expensive) testing
* The Docker API is versioned, and offers strong backward-compatibility (if a client uses e.g. API v1.25, the Docker Engine will keep behaving the same way)

### Attribution

Much of this workshop is heavily influenced by the awesome container and kubernetes workshops from Jerome Petazzoni. While I take a different approach, the material is none-the-less heavily influenced by his container.training material. [He and his contributors](https://github.com/jpetazzo/container.training/graphs/contributors) maintain an [Apache 2 licensed Open Source](https://github.com/jpetazzo/container.training/blob/master/LICENSE) project of his [training material](https://github.com/jpetazzo/container.training/) on github. 

Many additional sources influenced the shape of this project. Those items will be identified throughout.
 
### Just for fun

Just to validate everything is working correctly, let's spin up a cluster:
```bash
kind create cluster
``` 

Why kind? 
* kind supports multi-node (including HA) clusters
* kind supports building Kubernetes release builds from source
* support for make / bash / docker, or bazel, in addition to pre-published builds
* kind supports Linux, macOS and Windows
* kind is a CNCF certified conformant Kubernetes installer

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind
```
Success!!!

Do it again:
```bash
kind create cluster --name kind-2
```
Let's look through the [documentation of kind](https://kind.sigs.k8s.io/) while this cluster is creating. 

When it's done, type:
```
kind get clusters
```
You should see the two clusters we just created. If you run
```bash
cat ~/.kube/config 
```
You will see the kubeconfig file information. Use kubeconfig files to organize information about clusters, users, namespaces, and authentication mechanisms. The kubectl command-line tool uses kubeconfig files to find the information it needs to choose a cluster and communicate with the API server of a cluster.

NOTE: Notice that kind prefaces the cluster name with "kind-", so your first cluster is "kind-kind".

We can switch between the two clusters using `--context`
```bash
kubectl cluster-info --context kind-kind
```

or until we change it again with:
```bash
kubectl config kind-kind
```

Congrats, you've created two clusters with K8s.