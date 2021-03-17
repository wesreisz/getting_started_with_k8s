---
title: "Exposing the App"
full_title: "Lesson 3: Exposing the Sample Application to Traffic"
time_estimate: "20 Minutes"
weight: 3
date: 2020-10-23T1:00:00-00:98
draft: true
---

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).

![Services](/getting_started_with_k8s/images/lesson3/services.svg)

There are three service types that we'll be concerned about:
* clusterip
* nodeport
* loadbalancer

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic. 

**NOTE: All pods deployed on a node can more horizontally to another pod. These are not isolated unless they're in namespaces.**

Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:
* Designate objects for development, test, and production
* Embed version tags
* Classify an object using tags


### Key Takeaways
1. Learn how services are used to expose your services to traffic
1. Learn about the different service types, specifically clusterip, nodeport, and loadbalancer
1. See how to get logs from the pods.
1. Execute commands against and inside the pods.
1. Learn more about how the overlay networks work in Kubernetes.

### Services and Labels

Explore the IPs that were created. Let's first look at the nodes:
```bash
kubectl get nodes -o wide
```
This shows the node IPs. This is the infra that your nodes are running on. If you were to take a VM created in vSphere or provisioned on a cloud provider to install k8s on it, this is the accessible IP. It is required that your nodes need to be able to communicate between the control plan and the worker nodes. In the example below, I have a single node that is both the control plane and the worker node.

```
NAME                 STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                     KERNEL-VERSION      CONTAINER-RUNTIME
test-control-plane   Ready    master   34m   v1.19.1   172.22.0.2    <none>        Ubuntu Groovy Gorilla (development branch)   4.19.121-linuxkit   containerd://1.4.0
```

Now that we've looked at the nodes, let's look at the pod IPs.
```bash
kubectl get pods -o wide
```

In my case, I see:
```
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
hello-kubernetes-8448b6db7-27dlc   1/1     Running   0          37m   10.244.0.7   test-control-plane   <none>           <none>
hello-kubernetes-8448b6db7-knn2l   1/1     Running   0          37m   10.244.0.5   test-control-plane   <none>           <none>
hello-kubernetes-8448b6db7-x8m2d   1/1     Running   0          37m   10.244.0.6   test-control-plane   <none>           <none>
```

Notice that the IPs here are 10.244.*.* addresses. Why? 

These are the pod IPs. These are only routable (right now) inside the cluster. Give it a try. Ping one of the IPs from your host machine and then shell into one of your pods and ping it again. 


```bash
# from your host OS
ping 10.244.0.7

#shell into a pod and ping from inside the cluster
kubectl exec -it hello-kubernetes-8448b6db7-27dlc  -- bash
ping 10.244.0.7
```
These addresses are only routable "individually" inside the cluster, not outside the cluster. To address these two things, we use services. A service can make a VIP and/or expose that VIP outside the cluster. Here are the three we'll discuss 

Services can be exposed in different ways by specifying a type in the ServiceSpec:

* **ClusterIP** (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
* **NodePort** - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
* **LoadBalance**r** - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
* **ExternalName** - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns.

#### type: clusterip

Let's start by looking at a default standard deployment. Kompose created a service, let's look at it (again, you can look just after this is you want to create the service manually and not use Kompose).
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert -f docker-compose.yaml
    kompose.version: 1.22.0 (955b78124)
  creationTimestamp: null
  labels:
    io.kompose.service: hello-node
  name: hello-node
spec:
  ports:
    - name: "3000"
      port: 3000
      targetPort: 3000
  selector:
    io.kompose.service: hello-node
status:
  loadBalancer: {}
```

**Creating a clusterip service yourself**
```yaml
cat <<EOF > hello-node-service.yaml
# Save to 'hello-node-service.yaml'
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes
spec:
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: hello-kubernetes
EOF
```
The top matter looks similar. It just has a kind of Service (notice if nothing is specified it defaults to clusterip). In the spect section it defines the VIP port to use (in this case 80... or in the case of the Kompose one 3000), and then the port that you exposed your app on inside the container in the pod.



Now let's apply this yaml and test it out.
```bash
kubectl apply -f hello-node-service.yaml
```
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello-node   ClusterIP   10.96.217.162   <none>        3000/TCP   8s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    92m
```

Try to curl the pod. Does it work? Why?
```bash
hello-node]$ curl 10.96.217.162:3000
``` 

Now go into a pod itself. How do you do this in Docker? It's very similar here. Did that work? It should because a ClusterIP exposes the service internal to the cluster. You could use
```bash
kubectl get pods
kubectl exec -it hello-node-86d687ddfb-d56dp -- bash
```

Note: it is possible to expose with a clusterip and then use `kubectl port-forward svc/hello-node 8080:3000` but again this will only work because are k8s server is on the same system as we are. It won't work with a remote cluster.


#### type: nodeport
Now exit that pod and let's expose the app with nodeport.
```bash
kubectl get deployment
kubectl expose deployment hello-node --type=NodePort --name=hello-node-port
kubectl get services
```
```
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node        ClusterIP   10.96.217.162   <none>        3000/TCP         21m
hello-node-port   NodePort    10.96.98.158    <none>        3000:31117/TCP   14s
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP          114m
```
Notice the port number `31117` this is a randomly assigned high port number. If you go to the node ip at that port, you will see the app. NOTE: Not the cluster-ip, the node ip.

Try this:
```bash
kubectl get nodes -o wide
```
Using and of the `kind-worker` internal-ips, curl the the ip using the port `31117`. It should work.

Before we move to the next section, let's get the yaml from `kubectl` for this NodePort. We can use `kubectl` to do that:
```bash
kubectl get services hello-node-port -o yaml  > hello-node-port.yaml
```

NOTE: This is not really for saving off configuration. It's more for getting and grep values if you need it. The yaml this generates is not always small is structured very easily. You can also use `-o json`.


**NOTE** I created the nodeport using kubectl above for variety. You can also create it uing yaml like this:
```yaml
cat <<EOF > node-port-service.yaml
# Save to 'node-port-service.yaml'
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes-nodeport
spec:
  type: NodePort
  selector:
    app: hello-kubernetes
  ports:
  - nodePort: 30163
    port: 80
    targetPort: 3000
EOF
```
Nodeport is the port that will be exposed outside the cluster.
Port is the port that will be used on the VIP inside the cluster.
TargetPort is the port that will be targetted (where the app is deployed).

**NOTE:** One more note. If you're using kind on MacOS or Windows, when you try to access the nodeport outside the cluster it will fail. This fails because we're using docker in kubernetes to access things. It works in linux because (well, because the containers technology is part of linux... not an app that has to expose ports explicity). If you want to make nodeport work explicitly on Windows or MacOS, you need to tell Docker that you want to open the node port that's used when you create the cluster. Below is an example of what it might look like to do that:
```yaml
cat <<EOF > node-port-cluster.yaml
# Save to 'node-port-cluster.yaml'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "0.0.0.0"
  - containerPort: 30163
    hostPort: 30163
    listenAddress: "0.0.0.0"
EOF
```
Creates a two node cluster with port 80 and 30163 mapped

#### type: loadbalancer

If you’re using YAML files from your production deployment, be aware that a service of type LoadBalancer isn’t going to work on your local machine. You can create the service, but the External IP address will stay in Pending state indefinitely. This is because LoadBalancer type relies on an external provider (like your cloud service or vSphere) setting up a load balancer for sending traffic to the service. You’ll need to set the type to NodePort (or use an ingress).

Using loadbalancer with any of the cloud providers will work this out. If you're doing this on kind (like we are), you will see an external IP address of pending. You will need to expose a different service or install something like Metallb for this to resolve.

**NOTE** You can also install Metallb which is a tool that will assign a range of IP address to kind and pretty east to configure if you prefer. I dont cover Metallb here.

