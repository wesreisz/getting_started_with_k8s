---
title: "Networking"
full_title: "Lesson 5: Networking"
time_estimate: "20 minutes"
weight: 5
date: 2020-10-23T1:00:00-00:98
draft: true
---

### The Kubernetes network model

Every Pod gets its own IP address. This means you do not need to explicitly create links between Pods and you almost never need to deal with mapping container ports to host ports. This creates a clean, backwards-compatible model where Pods can be treated much like VMs or physical hosts from the perspectives of port allocation, naming, service discovery, load balancing, application configuration, and migration.

### Key Takeaways

1. Understand about more complicated configurations for architecture components
1. Explore the CNCF Cloud Native Landscape around Kubernetes

Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies):

* pods on a node can communicate with all pods on all nodes without NAT
agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node
Note: For those platforms that support Pods running in the host network (e.g. Linux):

* pods in the host network of a node can communicate with all pods on all nodes without NAT
This model is not only less complex overall, but it is principally compatible with the desire for Kubernetes to enable low-friction porting of apps from VMs to containers. If your job previously ran in a VM, your VM had an IP and could talk to other VMs in your project. This is the same basic model.

* Kubernetes IP addresses exist at the Pod scope - containers within a Pod share their network namespaces - including their IP address. This means that containers within a Pod can all reach each other's ports on localhost. This also means that containers within a Pod must coordinate port usage, but this is no different from processes in a VM. This is called the "IP-per-pod" model.

How this is implemented is a detail of the particular container runtime in use.

It is possible to request ports on the Node itself which forward to your Pod (called host ports), but this is a very niche operation. How that forwarding is implemented is also a detail of the container runtime. The Pod itself is blind to the existence or non-existence of host ports.

**Pod networking**
Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using localhost. 

When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports). Within a Pod, containers share an IP address and port space, and can find each other via localhost. The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. 

Containers in different Pods have distinct IP addresses and can not communicate by IPC without special configuration. Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

Containers within the Pod see the system hostname as being the same as the configured name for the Pod



Kubernetes defines a network model that helps provide simplicity and consistency across a range of networking environments and network implementations. The Kubernetes network model provides the foundation for understanding how containers, pods, and services within Kubernetes communicate with each other. This guide explains the key concepts and how they fit together.

In this guide you will learn:

The fundamental network behaviors the Kubernetes network model defines.
How Kubernetes works with a variety of different network implementations.
What Kubernetes Services are.
How DNS works within Kubernetes.
What “NAT outgoing” is and when you would want to use it.
What “dual stack” is.
The Kubernetes network model
The Kubernetes network model specifies:

Every pod gets its own IP address
Containers within a pod share the pod IP address and can communicate freely with each other
Pods can communicate with all other pods in the cluster using pod IP addresses (without NAT)
Isolation (restricting what each pod can communicate with) is defined using network policies
As a result, pods can be treated much like VMs or hosts (they all have unique IP addresses), and the containers within pods very much like processes running within a VM or host (they run in the same network namespace and share an IP address). This model makes it easier for applications to be migrated from VMs and hosts to pods managed by Kubernetes. In addition, because isolation is defined using network policies rather than the structure of the network, the network remains simple to understand. This style of network is sometimes referred to as a “flat network”.

Note that, although very rarely needed, Kubernetes does also support the ability to map host ports through to pods, or to run pods directly within the host network namespace sharing the host’s IP address.

Kubernetes network implementations
Kubernetes built in network support, kubenet, can provide some basic network connectivity. However, it is more common to use third party network implementations which plug into Kubernetes using the CNI (Container Network Interface) API.

There are lots of different kinds of CNI plugins, but the two main ones are:

network plugins, which are responsible for connecting pod to the network
IPAM (IP Address Management) plugins, which are responsible for allocating pod IP addresses.
Calico provides both network and IPAM plugins, but can also integrate and work seamlessly with some other CNI plugins, including AWS, Azure, and Google network CNI plugins, and the host local IPAM plugin. This flexibility allows you to choose the best networking options for your specific needs and deployment environment. You can read more about this in the Calico determine best networking option guide.

Kubernetes Services
Kubernetes Services provide a way of abstracting access to a group of pods as a network service. The group of pods is usually defined using a label selector. Within the cluster the network service is usually represented as virtual IP address, and kube-proxy load balances connections to the virtual IP across the group of pods backing the service. The virtual IP is discoverable through Kubernetes DNS. The DNS name and virtual IP address remain constant for the life time of the service, even though the pods backing the service may be created or destroyed, and the number of pods backing the service may change over time.

Kubernetes Services can also define how a service accessed from outside of the cluster, for example using

a node port, where the service can be accessed via a specific port on every node
or a load balancer, whether a network load balancer provides a virtual IP address that the service can be accessed via from outside the cluster.
Note that when using Calico in on-prem deployments you can also advertise service IP addresses, allowing services to be conveniently accessed without going via a node port or load balancer.

Kubernetes DNS
Each Kubernetes cluster provides a DNS service. Every pod and every service is discoverable through the Kubernetes DNS service.

For example:

Service: my-svc.my-namespace.svc.cluster-domain.example
Pod: pod-ip-address.my-namespace.pod.cluster-domain.example
Pod created by a deployment exposed as a service: pod-ip-address.deployment-name.my-namespace.svc.cluster-domain.example.
The DNS service is implemented as Kubernetes Service that maps to one or more DNS server pods (usually CoreDNS), that are scheduled just like any other pod. Pods in the cluster are configured to use the DNS service, with a DNS search list that includes the pod’s own namespace and the cluster’s default domain.

This means that if there is a service named foo in Kubernetes namespace bar, then pods in the same namespace can access the service as foo, and pods in other namespaces can access the service as foo.bar

Kubernetes supports a rich set of options for controlling DNS in different scenarios. You can read more about these in the Kubernetes guide DNS for Services and Pods.

NAT outgoing
The Kubernetes network model specifies that pods must be able to communicate with each other directly using pod IP addresses. But it does not mandate that pod IP addresses are routable beyond the boundaries of the cluster. Many Kubernetes network implementations use overlay networks. Typically for these deployments, when a pod initiates a connection to an IP address outside of the cluster, the node hosting the pod will SNAT (Source Network Address Translation) map the source address of the packet from the pod IP to the node IP. This enables the connection to be routed across the rest of the network to the destination (because the node IP is routable). Return packets on the connection are automatically mapped back by the node replacing the node IP with the pod IP before forwarding the packet to the pod.

When using Calico, depending on your environment, you can generally choose whether you prefer to run an overlay network, or prefer to have fully routable pod IPs. You can read more about this in the Calico determine best networking option guide. Calico also allows you to configure outgoing NAT for specific IP address ranges if more granularity is desired.


To get rerid of the context
```bash
kubectl config unset contexts
```