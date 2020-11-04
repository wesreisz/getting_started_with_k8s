---
title: "22 Essential Concepts"
full_title: "Lesson 5: 22 Essential Kubernetes Concepts"
time_estimate: "10 minutes"
weight: 8
date: 2020-10-23T1:00:00-00:98
draft: true
---

Cloud native represents a paradigm shift in the way infrastructure is deployed. If you are new to cloud native and Kubernetes, there are 22 essential Kubernetes concepts you’ll need to wrap your head around. 

### Key Takeaways

1. Reiterate k8s core concepts

#### Workloads

1. Node - A node may be a virtual or physical machine, depending on the cluster. Each node contains the services necessary to run Pods, managed by the control plane.
1. Cluster - A cluster is the foundation. All your containerized applications run on top of a cluster.
1. Pod - the basic unit of work. Pods are the smallest deployable units of computing that can be created and managed in Kubernetes. Pods are almost never created on their own, instead pod controllers do all the real work.
1. Namespace - Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. Namespaces are intended for use in environments with many users spread across multiple teams, or projects.


#### Pod Controllers
5. Deployment - Most common way to get your app on Kubernetes. Creates a ReplicaSet for each Deployment spec.
5. ReplicaSet -  Creates a stable set of pods. You will almost never create this directly. 
5. DaemonSet - A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created. Common for system process like CNI, Monitor agents, proxies, etc.
5. StatefulSet - StatefulSet is the workload API object used to manage stateful applications with persistent storage. Pod names are persistent and are retained when rescheduled (app-0, app-1). Pods have DNS names, unlike deployments. Storage stays associated with replacement pods. Volumes persist when pods are deleted. Pods are deployed/updated individually and in order.
5. HorizontalPodAutoscaler - The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment, replica set or stateful set based on standard metrics like CPU utilization or custom metrics like request latency.
5. PodDisruptionBudget - the ability to limit the number of Pods that are disrupted during upgrades and other planned events, allowing for higher availability while permitting the cluster administrator to manage the clusters nodes.
5. Job - A Job creates one or more Pods and expects them to successfully terminate.
5. CronJob - create Jobs on a schedule.


#### Configuration
13. ConfigMaps - an API object used to store non-confidential data in key-value pairs.
14. Secrets - let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys.

### Networking
15. Ingress - An API object that manages external access to the services in a cluster, typically HTTP. Ingress may provide load balancing, SSL termination and name-based virtual hosting.
16. Service - an abstract way to expose an application running on a set of Pods as a network service.
17. Network Policy - a specification of how groups of pods are allowed to communicate with each other and other network endpoints.

### Security -  RBAC 

17. ServiceAccount - provides an identity for processes that run in a Pod.
19. Role - a Role is a set of permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.
20. ClusterRole - ClusterRole is exactly the same as a Role except it is a non-namespaced resource. The resources have different names (Role and ClusterRole) because a Kubernetes object always has to be either namespaced or not namespaced; it can't be both.
21. RoleBinding - grants the permissions defined in a Role to some group of Users or ServiceAccounts. It holds a list of subjects (Users, Groups, or ServiceAccounts), and a reference to the Role being granted. A RoleBinding grants permissions within a specific namespace. 
22. ClusterRoleBinding - grants that access cluster-wide.

ref: https://www.fairwinds.com/blog/22-essential-kubernetes-concepts