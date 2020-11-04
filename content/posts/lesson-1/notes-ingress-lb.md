---
title: "Notes"
full_title: "Lesson 2: Example Application: DockerCoin"
time_estimate: "60 Minutes"
weight: 13
date: 2020-10-23T1:00:00-00:98
draft: true
---

The person I first took a container workshop with was Jerome Pettazoni. He's been a trusted trainer for QCon and me personally for at least four years now. He uses an application in his Kubernetes training course that we're going to use to in the next sections.

### Key Takeaways
1. Understand the sample app
1. Understand some of the terminology k8s users to talk about applications


spin up with docker compose

talk about kompose
conf



### Create a new Multi-node kind cluster
Kubernetes in Docker supports multi-node clusters. We'll use yaml to create one.
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
    protocol: udp # Optional, defaults to tcp
- role: worker
- role: worker    
```

Scale out the app on across the existing nodes.
```
kubectl scale --replicas 10 deployment/azure-vote-front
```

once we did the apply each container pulls the image to it's own local repo just like docker goes. 
get nodes -o wide will tells us where each container is deployed?
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

In fact, let's look at:
```
kubectl get pods -o wide
docker exec -it kind-worker crictl images
```

Let's get to an app running in kind:
port forward to a specific app instance

Port forwarding lets you access a specific pod’s container port directly. If you’ve got just one pod (perhaps because it’s a deployment with a single replica), this is a straightforward way to access it. It’s also the approach to use if you have reason to access a particular pod.
```bash
kubectl get pods
kubectl port-forward azure-vote-front-674f8559d4-c552s 8080:80
```

You can tail a specific instance of the logs in the app by:
```bash
kubectl logs -f azure-vote-front-674f8559d4-c552s
```
thx to liz rice: 
https://medium.com/@lizrice/accessing-an-application-on-kubernetes-in-docker-1054d46b64b1


If you’re using YAML files from your production deployment, be aware that a service of type LoadBalancer isn’t going to work on your local machine. You can create the service, but the External IP address will stay in Pending state indefinitely. This is because LoadBalancer type relies on an external provider (like your cloud service) setting up a load balancer for sending traffic to the service. You’ll need to set the type to NodePort.


you can curl a cluster ip from within the cluster... so if you copy the cluster ip then
log into one of the pods then you can curl the endpoint
```bash
kubectl expose deployment azure-vote-front --type=NodePort --name=vote-service
kubectl get pods
kubectl exec -it azure-vote-back-7dd7659996-7qntq -- bash
cat /etc/issue
apt-get update -y
apt-get install curl
curl 10.96.150.77
```


You can also create a 
```bash
kubectl expose deployment azure-vote-front --type=NodePort --name=azure-vote
kubectl get services
kubectl port-forward svc/azure-vote 8080:80
firefox 
```

Can exec into the backend and run redis-cli
```bash
kubectl get pods
kubectl exec -it azure-vote-back-7dd7659996-vcx4k -- bash
redis-cli
GET Dogs
```



```bash
- create a new cluster
- deploy azure app
- see pending state... discuss lb implementations
- we can fix by installing our own metalb
- install metalb
- docker network ls
- docker network inspect kind
- find cidr for pods and use some of them... this is a work around.
- configure metalb to use that range
- check your services
- firefox 172.20.255.1
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.20.255.1-172.20.255.250
```


Maybe install the k8s dashboard



https://mauilion.dev/posts/kind-metallb/


### Start Kubernetes Dashboard 
[Launch his slide deck where he discusses the app](https://pycon2019.container.training/#28)







#### Running the control plane on special nodes

It is common to reserve a dedicated node for the control plane (Except for single-node development clusters, like when using minikube).

This node is then called a "master" (Yes, this is ambiguous: is the "master" a node, or the whole control plane?)

Normal applications are restricted from running on this node (By using a mechanism called "[taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)")

When high availability is required, each service of the control plane must be resilient. The control plane is then replicated on multiple nodes(This is sometimes called a "multi-master" setup)


#### Running the control plane outside containers

The services of the control plane can run in or out of containers

For instance: since etcd is a critical service, some people deploy it directly on a dedicated cluster (without containers)

In some hosted Kubernetes offerings (e.g. AKS, GKE, EKS), the control plane is invisible (we only "see" a Kubernetes API endpoint). In that case, there is no "master node". For this reason, it is more accurate to say "control plane" rather than "master."

#### How many nodes should a cluster have?
There is no particular constraint (no need to have an odd number of nodes for quorum)

A cluster can have zero node (but then it won't be able to start any pods)

For testing and development, having a single node is fine

For production, make sure that you have extra capacity (so that your workload still fits if you lose a node or a group of nodes)

Kubernetes is tested with up to 5000 nodes (however, running a cluster of that size requires a lot of tuning).




