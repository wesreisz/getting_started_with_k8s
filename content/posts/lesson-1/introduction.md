---
title: "Course Introduction"
full_title: "Lesson 1: Course Introduction and Logistics"
weight: 1
date: 2020-10-23T1:00:00-00:98
draft: true
---

Containers are the new unit of deployment. The reason is simple: containers are a very powerful tool that can streamline development and ops, save companies money by focusing on deploying a packaged unit, and reduce the friction in delivering software. However, the flip side is they’re a new paradigm to understand, many things are abstracted away, and they require apps to be built with a specific architecture to take full advantage of their features. In this hands-on workshop, you will learn about Docker basic concepts, how to run containers, create your own images, interact with the "Docker Hub", and more.

### Key Takeaways
1. Learn what containers are and how they work
1. Understand the main use cases of containers
1. Understand how to containerize an app

### Audience
Anyone wanting to learn about containers. Will look at things through a developers lens.

### Prerequisites
* Docker client installed or use a Ubuntu student VM (accessed via Workspaces)

**NOTE** Everything can be run locally with your own installation of Docker; however, this is a 3 hour class, I have created Amazon workstation instances that you may access. I ask that you use those instances to minimize the configuration challenges with a large group. You can download the client here: https://clients.amazonworkspaces.com

### Agenda
* [Course Agenda](/getting_started_with_containerization/posts/)

### Environment

Each student has been created a virtual desktop enviornment. Everything can be run on your own machine if you have docker and docker-compose installed. As this is only a short 3-hour course, I request that you use the virtual desktop environment. This will minimize the support time if docker/docker compose is not setup correctly. Feel free to install docker and start locally, and use the VDI only if there's an issue.

To access the Amazon VDI, install workplace on your OS. The clients are available here:
https://clients.amazonworkspaces.com

You should have an [email](/getting_started_with_containerization/images/lesson0/email.png) that has
- link create your profile for the workspace
- the registration code to your specific workspace
- your userid that I created for you

Please ask your instructor for a student email account.

Please install the client, register your Ubuntu profile, and launch your VDI (make sure to use the registration code in your email).

![Your Amazon VDI](/getting_started_with_containerization/images/lesson0/desktop.png "Amazon VDI")



### Validate you are ready to start

Open a terminal and type:
```bash
docker info
```
You should see information about your installation of docker. There should be no errors. If you see a message with a failure in it:
```
ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
errors pretty printing info
```
Make sure docker is installed and the service is running. On your Ubuntu VDI, do this:
```bash
sudo service docker start
```

At this point, on your VDI you can only run docker as root. Let's fix that my running this command.
```bash
sudo usermod -a -G docker $USER
```
It won't work until we logoff and log back in, we'll do that in a minute. For now, use `sudo` when you run docker commands on this VM.

```bash
docker run hello-world
```
You should see something like this:
```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```
If so, you're ready to start! If this doesn't work, please install Docker and configure it for your system:
https://docs.docker.com/get-docker

Before we logoff, let's fix a dns issue with Amazon VDI and Docker. You don't need to do this if you're running locally and not having a dns issue.

```bash
sudo su -
echo "{\"dns\": [\"8.8.8.8\"]}" >>/etc/docker/daemon.json
sudo service docker restart
```


### Networking with Docker in Amazon workspaces
There is sometimes an issue with docker resolving dns on these VDI. The quickest solution is to add 
`-dns 8.8.8.8 ` to your docker command.

By setting /etc/docker/daemon.json with,

```bash
{
  "dns": ["8.8.8.8"]
}
```
and then restarting docker we won't have to keep doing this.

```
sudo service docker restart
```


### Potential Issues
**Amazon Workspace: Unable to run docker without using root**

ref: https://stackoverflow.com/questions/48957195/how-to-fix-docker-got-permission-denied-issue

I've installed docker and configured it. However, for your user you may get this error:
```bash
docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```

If you get this error, do the following to allow your user to run as a non-root user:

Create the docker group if it does not exist
```bash
sudo groupadd docker
```
Add your user to the docker group.
```bash
sudo usermod -aG docker $USER
```
Run the following command or Logout and login again and run (that doesn't work you may need to reboot your machine first)
```bash
newgrp docker
```
Check if docker can be run without root
```bash
docker run hello-world
```
Reboot & log back in if you still get this error
```bash
reboot
```


**Amazon Workspace: pip fails doing install due to network**

`failed to establish a new connection: [Errno -3] Temporary failure in name resolution`

There is an issue with the way that DNS is built with Amazon Linux workspaces

ref: https://stackoverflow.com/questions/28668180/cant-install-pip-packages-inside-a-docker-container-with-ubuntu

Modifying /etc/resolv.conf and adding the following lines at the end

```bash
# Google IPv4 nameservers
nameserver 8.8.8.8
nameserver 8.8.4.4
```

However this change won't be permanent (see this thread). To make it permanent : 
```bash
$ sudo nano /etc/dhcp/dhclient.conf
```
Uncomment and edit the line with prepend domain-name-server : prepend domain-name-servers 8.8.8.8, 8.8.4.4;

Restart dhclient : 
```bash
sudo dhclient
```


**Amazon Workspace: If you get cannot connect to Docker daemon, just start the docker service**
docker run -it --name docker-host --rm --privileged ubuntu:bionic
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```bash
sudo service docker start
```
