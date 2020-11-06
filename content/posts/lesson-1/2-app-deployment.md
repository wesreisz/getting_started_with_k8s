---
title: "Deploying an App"
full_title: "Lesson 2: Deploy and Interact with an Example Application"
time_estimate: "20 Minutes"
weight: 2
date: 2020-10-23T1:00:00-00:98
draft: true
---

Now that we have a basic understanding of what Kubernetes is and how it operates. We're going to deploy a simple application and interact with it. We will write this app locally, buld it, push it to Dockerhub. Then we'll create a docker-compose file and run it. Then we'll use Kompose and discuss how it can help you go from a `docker-compose.yaml` file to a deployment yaml file for Kubernetes. We'll deploy this file with the  `kubectl apply -f` command.  

### Key Takeaways
1. Refresh build and pushing an app to a repo.
1. Understand how to Write and Deploy a sample app to our cluster.
1. See how Kompose can add your development of yaml files.
1. Understand basic commands of interacting with your cluster.
1. Execute commands against and inside the pods.
1. Explore declarative vs imperative in relationship with k8s

### Write and Deploy a simple hello world node app
let's create a new folder to work in:
```bash
mkdir hello-node
cd hello-node
```

Create a Dockerfile from the commandline:
```bash
cat <<EOF > Dockerfile
FROM node:12-stretch
COPY index.js index.js
CMD ["node", "index.js"]
EOF
```

Create an `index.js` file from the commandline:
```javascript
cat <<EOF > index.js
const http = require("http");
  http
   .createServer(function(request, response){
       console.log("request received");
       response.end("Hello World", "utf-8")
   })
   .listen(3000)
  console.log("server running: localhost:3000");
EOF
```

After you have your `Dockerfile` and an `index.js`, let's build and run the app:
```bash
docker build -t wesreisz/hello-node:v1 .
docker image ls
docker run -p 3000:3000 hello-node:v1
```

push it to dockerhub (NOTE: you'll need to replace your username on dockerhub to push it there). However, you can skip this step, and just pull the image from my repo.
```bash
docker login
docker push wesreisz/hello-node:v1
```

Create a docker-compose.yml file
```yaml
cat <<EOF > docker-compose.yml
version: '3'
services:
  hello-node:
    image: wesreisz/hello-node:v1
    container_name: hello-node
    ports:
        - "3000:3000"
EOF
```

Run it
```bash
docker-compose up
```

#### Kompose
kompose is a tool to help users who are familiar with docker-compose move to Kubernetes. kompose takes a Docker Compose file and translates it into Kubernetes resources.

kompose is a convenience tool to go from local Docker development to managing your application with Kubernetes. Transformation of the Docker Compose format to Kubernetes resources manifest may not be exact, but it helps tremendously when first deploying an application on Kubernetes.

https://github.com/kubernetes/kompose

See if it's installed:
```bash
kompose version
```

If not, install it:
```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.22.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```

Convert your `docker-compose.yml` to a kubernetes.yml file for deployment.
```bash
kompose convert -f docker-compose.yaml
```

#### Deploy to your Cluster
A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert -f docker-compose.yml
    kompose.version: 1.22.0 (955b78124)
  creationTimestamp: null
  labels:
    io.kompose.service: hello-node
  name: hello-node
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: hello-node
  strategy: {}
  template:
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose.yml
        kompose.version: 1.22.0 (955b78124)
      creationTimestamp: null
      labels:
        io.kompose.service: hello-node
    spec:
      containers:
        - image: wesreisz/hello-node:v1
          name: hello-node
          ports:
            - containerPort: 3000
          resources: {}
      restartPolicy: Always
status: {}
```

To apply it, run:
```bash
kubectl apply -f hello-node-deployment.yaml 
```

See your deployment:
```bash
kubectl get pods
```

#### Pods

![](/getting_started_with_k8s/images/lesson3/k8s-arch3-thanks-weave.png)

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage/network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain init containers that run during Pod startup. You can also inject ephemeral containers for debugging if your cluster offers this.

Take a look at all of the pods, including the k8s system level pods
```bash
kubectl get pods -A
```

See your deployment:
```bash
kubectl get pods
kubectl get deployment
kubectl describe hello-node-7887c44b77-mnthh
```

### Declarative vs Imperative
Our container orchestrator puts a very strong emphasis on being declarative

Declarative: *I would like a cup of tea.*

Imperative: *Boil some water. Pour it in a teapot. Add tea leaves. Steep for a while. Serve in a cup.* 

Declarative seems simpler at first ...as long as you know how to brew tea

**Reconciling state**

* The spec describes how we want the thing to be
* Kubernetes will reconcile the current state with the spec (technically, this is done by a number of controllers)
* When we want to change some resource, we update the spec
* Kubernetes will then converge that resource


Replace replica count in the deployment descriptor with 10 and then get the pods again. You can also do it directly using a command like:
```bash
kubectl get pods
kubectl get deployment
kubectl describe hello-node-7887c44b77-mnthh
kubectl scale --replicas 1 deployment/hello-node
kubectl get pods -o wide
```

Each node maintains a docker image repo (just like you're used to with docker on your machine). Take a look, by finding the machine and then asking `crictl`
```bash
kubectl get nodes
docker exec -it kind-worker crictl images
```

Let's show k8s keeps things running by killing some pods:
```bash
kubectl delete pod hello-node-7887c44b77-7ckzj
```

Say we have some maintenace that needs to happen on a node or a problem: Let's remove traffic:
 ```bash
kubectl cordon kind3-worker3
kubectl cordon kind3-worker3
 ```
This doesn't remove traffic it marks it unschedulable

Now let's `drain` traffic
```bash
kubectl drain kind3-worker3
```

**DaemonSet?**

DaemonSets are used to ensure that some or all of your K8S nodes run a copy of a pod, which allows you to run a daemon on every node.

Let's reschedule workloads
```bash
kubectl uncordon kind3-worker3
kubectl scale --replicas 2 deployment/hello-node
kubectl scale --replicas 5 deployment/hello-node
```

