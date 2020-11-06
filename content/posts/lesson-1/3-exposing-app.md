---
title: "Exposing the App"
full_title: "Lesson 3: Exposing the Sample Application to Traffic"
time_estimate: "20 Minutes"
weight: 3
date: 2020-10-23T1:00:00-00:98
draft: true
---

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic. 

Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:
* Designate objects for development, test, and production
* Embed version tags
* Classify an object using tags


### Key Takeaways
1. Learn how services are used to expose your services to traffic
1. Learn about the different service types
1. See how to get logs from the pods.
1. Execute commands against and inside the pods.
1. Learn more about how the overlay networks work in Kubernetes.

### Services and Labels

Explore the IPs that were created:
```bash
ping 10.244.1.3
kubectl exec -it hello-node-7887c44b77-4bw6v  -- bash
ping 10.244.3.11
```
These addresses are only routable inside the cluster, not outside... services address this.

![Services](/getting_started_with_k8s/images/lesson3/services.svg)

Services can be exposed in different ways by specifying a type in the ServiceSpec:

* **ClusterIP** (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
* **NodePort** - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
* **LoadBalance**r** - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
* **ExternalName** - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of kube-dns.

#### type: clusterip

Let's start by looking at a default standard deployment. Kompose created a service, let's look at it.
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert -f docker-compose.yml
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
ubectl get services
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


#### type: loadbalancer

If you’re using YAML files from your production deployment, be aware that a service of type LoadBalancer isn’t going to work on your local machine. You can create the service, but the External IP address will stay in Pending state indefinitely. This is because LoadBalancer type relies on an external provider (like your cloud service) setting up a load balancer for sending traffic to the service. You’ll need to set the type to NodePort.

Using loadbalancer with any of the cloud providers will working this out. If you're doing this on kind (like we are), you will see an external IP address of pending. You will need to expose a different service or install something like Metallb for this to resolve.

