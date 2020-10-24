---
title: "Dockerfile"
full_title: "Lesson 4: Dockerfile"
weight: 4
date: 2020-10-23T1:00:00-00:96
draft: true
---

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using docker build users can create an automated build that executes several command-line instructions in succession.
[Example Dockerfile](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)

### Learning Objectives
* *Understand* how the Dockerfile works
* *Build* containers
* *Use* containers we built.


Here is the format of the Dockerfile:

`INSTRUCTION arguments`

**NOTE**: The capitalized INSTRUCTION is more convention than anything else. Your editors may warn you and you'll rarely see the instruction not capitalized, but it will work if it's lowercase. 

After we create the file, we'll use `docker build .` to create an image. When we build an image, it's stored in a local copy of your image repository (if you're used to maven, similar to that).

Let's go through a simple example. Create a new working direction and `cd` into it. This snippet just writes some text into a file.
```bash
cat <<EOF > Dockerfile
FROM ubuntu:latest
RUN echo "Hello World"
EOF
```

After you have your `Dockerfile`, build it with:
`docker build .`

Let's go through a more complete example and build a node server.

Here's the Dockerfile.
```Dockerfile
FROM node:12-stretch
COPY index.js index.js
CMD ["node", "index.js"]
```

Let's create index.js
```javascript
const http = require("http");
  http
   .createServer(function(request, response){
       console.log("request received");
       response.end("Hello World", "utf-8")
   })
   .listen(3000)
  console.log("server running: localhost:3000");
```

build and tag it
```bash
docker build -t hello-node .
```

run it
```bash
docker run hello-node
```

This won't run because while the port is open in the container, it's not exposed outside the container. We have to open it. Kill it and let's try again. `--publish` or `-p`
```bash
docker run --publish 3000:3000 hello-node
```

**NOTE:** 

cntrl-c doesn't work... because we're sending cntrl-c to docker and it's not being passed to node. There is a more correct way to capture the signal; however, if you use `--init` for the time being when you run it. It will exit normally. To close the process, open another terminal windows and find the process with `ps aux` and kill that pid.

**Container Naming**

An image name is made up of slash-separated name components, optionally prefixed by a registry hostname. The hostname must comply with standard DNS rules, but may not contain underscores. If a hostname is present, it may optionally be followed by a port number in the format :8080. If not present, the command uses Docker’s public registry located at registry-1.docker.io by default (this is where it will push/pull from). Name components may contain lowercase letters, digits and separators. A separator is defined as a period, one or two underscores, or one or more dashes. A name component may not start or end with a separator.
```bash
docker image ls
```
```
REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
hello-node                                       latest              0810ad403c0f        37 hours ago        918MB
mcr.microsoft.com/dotnet/core/sdk                3.1                 c4155a9104a8        6 weeks ago         708MB
azurealpharegistry.azurecr.io/azure-vote-front   v1                  5d0b975ec2b3        2 months ago        944MB
registry.tkg.vmware.run/kind/node                v1.18.6_vmware.1    62aa5847174e        3 months ago        1.42GB
tiangolo/uwsgi-nginx-flask                       python3.6           42c10a01539f        4 months ago        944MB
```
&nbsp; 

**Running as Root**
In this node example, we're running as root. That's not really desired. It's more of an attack surface. Let's create and run as a less priveledged user. Update the Dockerfile with this
```Dockerfile
FROM node:12-stretch
USER node 
COPY --chown=node:node index.js index.js
CMD ["node", "index.js"]
```

Now let's build, run, and exec into the container. What user is it running as?
```bash
docker build -t hello-node:v1 .
docker image ls
docker run -dit --rm -p 3000:3000  hello-node:v1
docker ps
docker exec -it 499f587a0691 bash
```

ADD vs COPY
ADD and COPY are doing the same thing. ADD has the ability to do a few different things like go out to the network and download a file from the network. In general doing less, is safer. So use the least priveledged "thing" you can. Doing less is safer. Add will automatically unzip/untar as well (a copy would require that extra step).

```Dockerfile
...
WORKDIR /home/node/src
COPY --chown=node:node index.js index.js
...
```
Create a working directory... we put it in home because we know for sure the node user owns it's own home directory.  If it doesn't exist, it will create it for you. Build it and run it again. Then look at the directory where the source code is:
```bash
...
docker exec -it 2e91baefe0 pwd
```

**Docker Layers** 
Docker containers are building blocks for applications. Each container is an image with a readable/writeable layer on top of a bunch of read-only layers.
These layers (also called intermediate images) are generated when the commands in the Dockerfile are executed during the Docker image build.
For example, here is a Dockerfile for creating a node.js web app image . It shows the commands that are executed to create the image. Layers are neat because they can be re-used by multiple images saving disk space and reducing time to build images while maintaining their integrity. Order matters... and each layer is cached. So you may have to change a Dockefile at a certain line to get it to rebuild. You may also reorganize when you're doing installs so it doesn't install everytime. We'll see more about this in a bit.
```
wesleyreisz@Wesleys-MacBook-Pro hello-node % docker build .
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM node:12-stretch
 ---> 1f560ce4ce7e
Step 2/5 : USER node
 ---> Using cache
 ---> 372366a9e39a
Step 3/5 : WORKDIR /home/node/src
 ---> Using cache
 ---> a99e8cb0772f
Step 4/5 : COPY --chown=node:node index.js index.js
 ---> Using cache
 ---> 9546865b3c50
Step 5/5 : CMD ["node", "index.js"]]
 ---> Using cache
 ---> 4ec64400f39d
Successfully built 4ec64400f39d
```

**CMD and ENTRYPOINT**

What is the difference between CMD and ENTRYPOINT? You cannot override the ENTRYPOINT instruction by adding command-line parameters to the docker run command. By opting for this instruction, you imply that the container is specifically built for such use.