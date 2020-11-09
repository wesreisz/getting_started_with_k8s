---
title: "K8s Concepts"
full_title: "Lesson 1: Kubernetes Background and Concepts"
time_estimate: "20 Minutes"
weight: 1
date: 2020-10-23T1:00:00-00:98
draft: true
---

Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

The name Kubernetes originates from Greek, meaning helmsman or pilot. Google open-sourced the Kubernetes project in 2014. Kubernetes combines over 15 years of Google's experience running production workloads at scale with best-of-breed ideas and practices from the community.

There are a huge number of contributors and [companies contributing to k8s](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1).

Some stats:
* **84 percent**: In its most recent survey, the Cloud Native Computing Foundation (CNCF) found that in 2019 the vast majority of respondents – 84 percent – were running containers in production. 
* **56 percent**: Meanwhile, that usage still appears to be growing considerably. 56 percent of the organizations polled for the 2020 edition of The State of Enterprise Open Source report said they expected their use of containers to increase in the next 12 months.
* **78 percent**: Kubernetes is not the only option for container orchestration, but it has clearly become the go-to choice: more than three in four firms included in the CNCF report said they were using Kubernetes in some form.
* **250+**: In 2019, the percentage of companies running 250 or more containers in production grew by 28 percent, crossing the 50 percent threshold for the first time.
* **8/10**: The ratio (81 percent) of companies that have already adopted public cloud that say they’re working with two or more providers, according to Gartner.
* **6 years old**: All of these numbers are made more impressive by Kubernetes’ relative youth: The project just celebrated its sixth birthday on June 7.



[Kubernetes by the numbers, in 2020: 12 stats to see](https://enterprisersproject.com/article/2020/6/kubernetes-statistics-2020)

### Key Takeaways
1. Containers vs virtual machines
1. Why is Kubernetes needed
1. Understand the Kubernetes Key Components
1. Basic Control Plane / Node Components


### Container Evolution
!["Evolution"](/getting_started_with_k8s/images/lesson1/container_evolution.svg)

**Traditional deployment era:** 

Early on, organizations ran applications on physical servers. There was no way to define resource boundaries for applications in a physical server, and this caused resource allocation issues. For example, if multiple applications run on a physical server, there can be instances where one application would take up most of the resources, and as a result, the other applications would underperform. A solution for this would be to run each application on a different physical server. But this did not scale as resources were underutilized, and it was expensive for organizations to maintain many physical servers.

**Virtualized deployment era:**

As a solution, virtualization was introduced. It allows you to run multiple Virtual Machines (VMs) on a single physical server's CPU. Virtualization allows applications to be isolated between VMs and provides a level of security as the information of one application cannot be freely accessed by another application.

Virtualization allows better utilization of resources in a physical server and allows better scalability because an application can be added or updated easily, reduces hardware costs, and much more. With virtualization you can present a set of physical resources as a cluster of disposable virtual machines.

Each VM is a full machine running all the components, including its own operating system, on top of the virtualized hardware.

**Container deployment era:** 

Containers are similar to VMs, but they have relaxed isolation properties to share the Operating System (OS) among the applications. Therefore, containers are considered lightweight. Similar to a VM, a container has its own filesystem, share of CPU, memory, process space, and more. As they are decoupled from the underlying infrastructure, they are portable across clouds and OS distributions.

Containers have become popular because they provide extra benefits, such as:

* Agile application creation and deployment: increased ease and efficiency of container image creation compared to VM image use.
* Continuous development, integration, and deployment: provides for reliable and frequent container image build and deployment with quick and easy rollbacks (due to image immutability).
* Dev and Ops separation of concerns: create application container images at build/release time rather than deployment time, thereby decoupling applications from infrastructure.
* Observability not only surfaces OS-level information and metrics, but also application health and other signals.
* Environmental consistency across development, testing, and production: Runs the same on a laptop as it does in the cloud.
* Cloud and OS distribution portability: Runs on Ubuntu, RHEL, CoreOS, on-premises, on major public clouds, and anywhere else.
* Application-centric management: Raises the level of abstraction from running an OS on virtual hardware to running an application on an OS using logical resources.
* Loosely coupled, distributed, elastic, liberated micro-services: applications are broken into smaller, independent pieces and can be deployed and managed dynamically – not a monolithic stack running on one big single-purpose machine.
* Resource isolation: predictable application performance.
* Resource utilization: high efficiency and density.


## Why you need Kubernetes and what it can do
Containers are a good way to bundle and run your applications. In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. For example, if a container goes down, another container needs to start. Wouldn't it be easier if this behavior was handled by a system?

That's how Kubernetes comes to the rescue! Kubernetes provides you with a framework to run distributed systems resiliently. It takes care of scaling and failover for your application, provides deployment patterns, and more. For example, Kubernetes can easily manage a canary deployment for your system.

Kubernetes provides you with:

* **Service discovery and load balancing**: Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
* **Storage orchestration**: Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
* **Automated rollouts and rollbacks**: You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.
* **Automatic bin packing**: You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
* **Self-healing Kubernetes**: restarts containers that fail, replaces containers, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
* **Secret and configuration management**: Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.


#### IaaS Providers for k8s
* Alibaba Cloud:	https://www.alibabacloud.com/
* Amazon Web Services:	https://aws.amazon.com/
* Google Cloud Platform:	https://cloud.google.com/
* IBM Cloud:	https://www.ibm.com/cloud
* Microsoft Azure:	https://docs.microsoft.com/en-us/azure/security
* VMWare VSphere:	https://www.vmware.com


### Kubernetes Components

!["Components of Kubernetes"](/getting_started_with_k8s/images/lesson1/components-of-kubernetes.svg)

#### Control Plane Components 
The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine.

**kube-apiserver**

The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.

**etcd**

Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.

You can find in-depth information about etcd in the official documentation.

**kube-scheduler** 

Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.

Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

**kube-controller-manager**

Control Plane component that runs controller processes.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

* Node controller: Responsible for noticing and responding when nodes go down.
Replication controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
* Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
* Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.

**cloud-controller-manager** 

A Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager: lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that just interact with your cluster.

The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager.

As with the kube-controller-manager, the cloud-controller-manager combines several logically independent control loops into a single binary that you run as a single process. You can scale horizontally (run more than one copy) to improve performance or to help tolerate failures.

The following controllers can have cloud provider dependencies:

* Node controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
* Route controller: For setting up routes in the underlying cloud infrastructure
* Service controller: For creating, updating and deleting cloud provider load balancers

#### Node Components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

**kubelet**

An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.

**kube-proxy** 

kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

**Container runtime**

The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).

#### K8s Architecture

!["K8s Architecture"](/getting_started_with_k8s/images/lesson1/k8s-arch2.png)
(Courtesy of [Imesh Gunaratne](https://medium.com/containermind/a-reference-architecture-for-deploying-wso2-middleware-on-kubernetes-d4dee7601e8e))


### Addons 
Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

Selected addons are described below; for an extended list of available addons, please see Addons.

**DNS**

While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it.

Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.

**Web UI (Dashboard)**

Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

**Container Resource Monitoring**

Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

**Cluster-level Logging**

A cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.

### Explore the Components
How many nodes does your cluster have?
```bash
kubectl config get-contexts
```

Let's create more nodes. First which cluster were you on again?
```bash
kubectl config get-contexts
```

Create a `multinode.yaml` file
```yaml
cat <<EOF > multinode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
EOF
```

Now let's build it: 
```bash
kind create cluster --config ./multinode.yaml  --name kind3
```

Kind is still early alpha and doesn't support adding new nodes to a running cluster (yet).

To configure kind cluster creation, you will need to create a YAML config file. This file follows Kubernetes conventions for versioning etc.


**Nodes**

Nodes are the machines running the Kubernetes cluster. These can be bare metal, virtual machines, or anything else. The word hosts is often used interchangeably with Nodes. I will try and use the term Nodes with consistency but will sometimes use the word Virtual Machine to refer to Nodes depending on context.
```bash
kubectl get nodes
docker ps
docker exec -it kind3-control-plane bash
```

#### How many nodes should a cluster have?
There is no particular constraint (no need to have an odd number of nodes for quorum)

A cluster can have zero node (but then it won't be able to start any pods)

For testing and development, having a single node is fine

For production, make sure that you have extra capacity (so that your workload still fits if you lose a node or a group of nodes)

Kubernetes is tested with up to 5000 nodes (however, running a cluster of that size requires a lot of tuning).

