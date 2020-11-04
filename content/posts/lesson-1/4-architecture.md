---
title: "K8s Architecture and Components"
full_title: "Lesson 4: K8s Architecture and Components"
time_estimate: "20 Minutes"
weight: 4
date: 2020-10-23T1:00:00-00:98
draft: true
---

This architecture of Kubernetes provides a flexible, loosely-coupled mechanism for service discovery. Like most distributed computing platforms, a Kubernetes cluster consists of at least one master and multiple compute nodes. The master is responsible for exposing the application program interface (API), scheduling the deployments and managing the overall cluster. Each node runs a container runtime, such as Docker or rkt, along with an agent that communicates with the master. The node also runs additional components for logging, monitoring, service discovery and optional add-ons. Nodes are the workhorses of a Kubernetes cluster. They expose compute, networking and storage resources to applications. Nodes can be virtual machines (VMs) running in a cloud or bare metal servers running within the data center.

### Key Takeaways

1. Understand about more complicated configurations for architecture components
1. Explore the CNCF Cloud Native Landscape around Kubernetes

![](/getting_started_with_k8s/images/lesson3/k8s-arch4-thanks-luxas.png)

#### Controller Manager
The Kubernetes controller manager is a daemon that embeds the core control loops shipped with Kubernetes. In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

**Controller pattern**
A controller tracks at least one Kubernetes resource type. These objects have a spec field that represents the desired state. The controller(s) for that resource are responsible for making the current state come closer to that desired state.

The controller might carry the action out itself; more commonly, in Kubernetes, a controller will send messages to the API server that have useful side effects. 

-Kubernetes official documentation, Kube-controller-manager

The vital role of a Kubernetes controller is to watch objects for the desired state and the actual state, then send instructions to make the actual state be more like the desired state.In order to retrieve an object's information, the controller sends a request to Kubernetes API server.

Controllers can track many objects including:

* What workloads are running and where
* Resources available to those workloads
* Policies around how the workloads behave (restart, upgrades, fault-tolerance)

When the controller notices a divergence between the actual state and the desired state, it will send messages to the Kubernetes API server to make any necessary changes.

**Types of Controllers**

* ReplicaSet - A ReplicaSet creates a stable set of pods, all running the same workload. You will almost never create this directly.
* Deployment - A Deployment is the most common way to get your app on Kubernetes. It maintains a ReplicaSet with the desired configuration, with some additional configuration for managing updates and rollbacks.
* StatefulSet - A StatefulSet is used to manage stateful applications with persistent storage. Pod names are persistent and are retained when rescheduled (app-0, app-1). Storage stays associated with replacement pods, and volumes persist when pods are deleted.
* Job - A Job creates one or more short-lived Pods and expects them to successfully terminate.
* CronJob - A CronJob creates Jobs on a schedule.
* DaemonSet - A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Common for system processes like CNI, Monitor agents, proxies, etc.

Kubernetes Controllers allow you to run and manage your applications inside a cluster, and are one of the core concepts you’ll need to understand to be successful with Kubernetes.


### API Server

We will interact with our Kubernetes cluster through the Kubernetes API

The Kubernetes API is (mostly) RESTful. It allows us to create, read, update, delete resources


### Scheduler



### etcd


### CNI


### CRI

### CRI
At the lowest layers of a Kubernetes node is the software that, among other things, starts and stops containers. We call this the “Container Runtime”. The most widely known container runtime is Docker, but it is not alone in this space. In fact, the container runtime space has been rapidly evolving. As part of the effort to make Kubernetes more extensible, we've been working on a new plugin API for container runtimes in Kubernetes, called "CRI".

**containerd**

maintained by Docker, IBM, and community
used by Docker Engine, microk8s, k3s, GKE; also standalone
comes with its own CLI, ctr

**CRI-O**

maintained by Red Hat, SUSE, and community
used by OpenShift and Kubic
designed specifically as a minimal runtime for Kubernetes

And more... We're using docker (in fact, kind standards for Kubernetes in Docker)

### OCI



### Protobug, gRPC, & JSON