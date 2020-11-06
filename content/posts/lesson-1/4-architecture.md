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

Let's deploy a new app. First, we'll tear down the kind server
```bash
kubectl config get-clusters
kind get clusters
kind delete cluster --name kind3
docker ps
```

Note: you can also run something like: `kubectl delete --all namespaces` to delete all namespaces (and the objects in there) in the system. It will not delete system namespaces. So it has the effect of cleaning the cluster. With kind, it's just easier for me to recreate the cluster.


Let's create a new cluster. Remember, we created a new yml file before. We can use it for the new cluster. 
```yaml
cat <<EOF > singlenode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
EOF
```

Build the cluster.
```bash
kind create cluster --config singlenode.yaml
```

In our new cluster, let's create two namespaces. We'll call one `dev` and another `prod`
```bash
kubectl create namespace dev
kubectl create namespace prod
```

#### Namespace
Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. Namespaces are intended for use in environments with many users spread across multiple teams, or projects.

Namespaces provide:
* A scope for Names.
* A mechanism to attach authorization and policy to a subsection of the cluster.

Get the namespaces on your cluster. Note: this will be all namespaces including the ones that run pods kubernetes uses.
```bash
kubectl get namespaces
```
```
[CORP\student2@a-3ronkozj3vcn8 hello-node]$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   8m9s
dev               Active   116s
kube-node-lease   Active   3m10s
kube-public       Active   8m10s
kube-system       Active   8m10s
prod              Active   113s
```

You can also see them if you use:
```bash
kubectl get pods -A
```

You can also use yaml and `kubectl apply -f ` it.
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: test
  labels:
    name: test
```

Let's clone an different app and load into the dev and prod namespace.
```bash
git clone https://github.com/wesreisz/azure-voting-app-redis.git
kubectl apply -f azure-vote-all-in-one-redis.yaml --namespace=dev
kubectl apply -f azure-vote-all-in-one-redis.yaml --namespace=prod
```

Now type: `kubectl get pods` Where is your deployment?

By default, `get pods` is only showing the default namespace so you'll need to add `-A`
```bash
kubectl get pods -A
```

In addition, the namespace is "part" of the name now. It will fail if you, don't include it:
Try these:
```bash
kubectl exec pods nginx --bash
kubectl exec pods -n dev nginx --bash
kubectl exec pods -n prod nginx --bash
```

...and, of course, you can also set this up in yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: prod
  labels:
    name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
```

To list only the pods in a given namespace, try this:
```bash
kubectl get pods --namespace=dev
```

Let's scale up production to five nodes:
```bash
kubectl scale --replicas 5 -n prod          deployment/azure-vote-front
```

Another way we can do this is to use contexts. Try these steps:

find the cluster and user name by running:
```bash
kubectl config view
```

This create two contexts
```bash
kubectl config set-context dev --namespace=dev --cluster=kind-kind --user=kind-kind
kubectl config set-context prod --namespace=prod --cluster=kind-kind --user=kind-kind
```

Now you can view and then apply the context like this:
```bash
kubectl config view
kubectl config use-context prod
```

### Deeper into K8s Architecture

![](/getting_started_with_k8s/images/lesson3/k8s-arch4-thanks-luxas.png)

#### Controller Manager
Controllers are the core abstraction used to build Kubernetes. Once you’ve declared the desired state of your cluster using the API server, controllers ensure that the cluster’s current state matches the desired state by continuously watching the state of the API server and reacting to any changes. 

The Kubernetes controller manager is a daemon that embeds the core control loops shipped with Kubernetes. In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

Two methods to access:
* kubectl proxy --port=8080
* Direct Access using Tokens and Certificates

**Example Using kubectl proxy**
In one terminal, start the proxy
```bash
kubectl proxy --port=8080
```

Either send that process to the background or open another terminal:
```bash
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-86d687ddfb-d56dp   1/1     Running   0          5h26m
```
Call the api
```bash
curl localhost:8080/api/v1/namespaces/default/pods/hello-node-86d687ddfb-d56dp | jq
```

Another approach is to pass an authentication token:
```bash
# Check all possible clusters, as your .KUBECONFIG may have multiple contexts:
kubectl config view -o jsonpath='{"Cluster name\tServer\n"}{range .clusters[*]}{.name}{"\t"}{.cluster.server}{"\n"}{end}'
# Select name of cluster you want to interact with from above output:
export CLUSTER_NAME="kind-kind"
# Point to the API server referring the cluster name
APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")
# Gets the token value
TOKEN=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode)
# Explore the API with TOKEN
curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```
```json
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

#### API versioning
The JSON and Protobuf serialization schemas follow the same guidelines for schema changes. The following descriptions cover both formats.

The API versioning and software versioning are indirectly related. The API and release versioning proposal describes the relationship between API versioning and software versioning.

Different API versions indicate different levels of stability and support. You can find more information about the criteria for each level in the API Changes documentation.

Here's a summary of each level:

**Alpha:**

* The version names contain alpha (for example, v1alpha1).
* The software may contain bugs. Enabling a feature may expose bugs. A feature may be disabled by default.
* The support for a feature may be dropped at any time without notice.
* The API may change in incompatible ways in a later software release without notice.
* The software is recommended for use only in short-lived testing clusters, due to increased risk of bugs and lack of long-term support.

**Beta:**

The version names contain beta (for example, v2beta3).

The software is well tested. Enabling a feature is considered safe. Features are enabled by default.

The support for a feature will not be dropped, though the details may change.

The schema and/or semantics of objects may change in incompatible ways in a subsequent beta or stable release. When this happens, migration instructions are provided. Schema changes may require deleting, editing, and re-creating API objects. The editing process may not be straightforward. The migration may require downtime for applications that rely on the feature.

The software is not recommended for production uses. Subsequent releases may introduce incompatible changes. If you have multiple clusters which can be upgraded independently, you may be able to relax this restriction.

Note: Please try beta features and provide feedback. After the features exit beta, it may not be practical to make more changes.

**Stable:**

The version name is vX where X is an integer.
The stable versions of features appear in released software for many subsequent versions.
API groups
API groups make it easier to extend the Kubernetes API. The API group is specified in a REST path and in the apiVersion field of a serialized object.

There are several API groups in Kubernetes:

The core (also called legacy) group is found at REST path /api/v1. The core group is not specified as part of the apiVersion field, for example, apiVersion: v1.
The named groups are at REST path /apis/$GROUP_NAME/$VERSION and use apiVersion: $GROUP_NAME/$VERSION (for example, apiVersion: batch/v1). You can find the full list of supported API groups in Kubernetes API reference.
Enabling or disabling API groups
Certain resources and API groups are enabled by default. You can enable or disable them by setting --runtime-config on the API server. The --runtime-config flag accepts comma separated <key>[=<value>] pairs describing the runtime configuration of the API server. If the =<value> part is omitted, it is treated as if =true is specified. 

For example:
* to disable batch/v1, set --runtime-config=batch/v1=false
* to enable batch/v2alpha1, set --runtime-config=batch/v2alpha1


[Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/_

#### Scheduler

A scheduler watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on. 

kube-scheduler is the default scheduler for Kubernetes and runs as part of the control plane. kube-scheduler is designed so that, if you want and need to, you can write your own scheduling component and use that instead.

Node selection in kube-scheduler

kube-scheduler selects a node for the pod in a 2-step operation:
* Filtering
* Scoring

The filtering step finds the set of Nodes where it's feasible to schedule the Pod. For example, the PodFitsResources filter checks whether a candidate Node has enough available resource to meet a Pod's specific resource requests. After this step, the node list contains any suitable Nodes; often, there will be more than one. If the list is empty, that Pod isn't (yet) schedulable.

In the scoring step, the scheduler ranks the remaining nodes to choose the most suitable Pod placement. The scheduler assigns a score to each Node that survived filtering, basing this score on the active scoring rules.

Finally, kube-scheduler assigns the Pod to the Node with the highest ranking. If there is more than one node with equal scores, kube-scheduler selects one of these at random.

```bash
kubectl get pods -o wide
```
```
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
azure-vote-back-7dd7659996-jvbkm    1/1     Running   0          94m   10.244.3.2   kind-worker2   <none>           <none>
azure-vote-front-674f8559d4-62l5p   1/1     Running   0          94m   10.244.1.3   kind-worker3   <none>           <none>
azure-vote-front-674f8559d4-7fpj2   1/1     Running   0          85m   10.244.3.3   kind-worker2   <none>           <none>
azure-vote-front-674f8559d4-852pn   1/1     Running   0          85m   10.244.3.4   kind-worker2   <none>           <none>
azure-vote-front-674f8559d4-d4zk2   1/1     Running   0          85m   10.244.1.4   kind-worker3   <none>           <none>
azure-vote-front-674f8559d4-q25mh   1/1     Running   0          85m   10.244.2.3   kind-worker    <none>           <none>
```

**Taints and Tolerations**
Node affinity, is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

#### etcd
etcd is a consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.

https://etcd.io/docs/


### Cloud Controller Manager (not shown)
Cloud infrastructure technologies let you run Kubernetes on public, private, and hybrid clouds. Kubernetes believes in automated, API-driven infrastructure without tight coupling between components.

The cloud-controller-manager is a Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that just interact with your cluster.

By decoupling the interoperability logic between Kubernetes and the underlying cloud infrastructure, the cloud-controller-manager component enables cloud providers to release features at a different pace compared to the main Kubernetes project.

The cloud-controller-manager is structured using a plugin mechanism that allows different cloud providers to integrate their platforms with Kubernetes.

NOTE: This is how the loadbalancer service obtains an external IP.




#### CNI
CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.

https://kubernetes.io/docs/concepts/cluster-administration/networking/

#### CRI
At the lowest layers of a Kubernetes node is the software that, among other things, starts and stops containers. We call this the “Container Runtime”. The most widely known container runtime is Docker, but it is not alone in this space. In fact, the container runtime space has been rapidly evolving. As part of the effort to make Kubernetes more extensible, we've been working on a new plugin API for container runtimes in Kubernetes, called "CRI".

The most common CRI is Docker; however, there others (including containerd and CRI-O)

* containerd: maintained by Docker, IBM, and community. Used by Docker Engine, microk8s, k3s, GKE; also standalone comes with its own CLI, ctr

* CRI-O: maintained by Red Hat, SUSE, and community. Used by OpenShift and Kubic
designed specifically as a minimal runtime for Kubernetes

And more... 
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

We're using docker  

#### OCI
Open Container Initative: The Open Container Initiative is an open governance structure for the express purpose of creating open industry standards around container formats and runtimes. Runtime specification defining what it means to run an image or container. Enables interoperability between container runtimes.
https://www.youtube.com/watch?v=RyXL1zOa8Bw

#### Protobuf, gRPC, & JSON
* Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler.
* gRPC is an OSS remote procedure call system that uses Protobuf. It's a binary protocol that uses HTTP/2. These are internal APIs
* User facing API uses JSON for easy parsing.


### CNCF Landscape

The CNCF Cloud Native Landscape Project is intended as a map through the previously uncharted terrain of cloud native technologies. This attempts to categorize most of the projects and product offerings in the cloud native space. There are many routes to deploying a cloud native application, with CNCF Projects representing a particularly well-traveled path. 

[CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/)