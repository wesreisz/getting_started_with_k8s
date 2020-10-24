---
title: "Networking"
full_title: "Lesson 6.1: Networking"
weight: 7
date: 2020-10-23T1:00:00-00:96
draft: true
---
Docker includes support for networking containers through the use of network drivers. By default, Docker provides two network drivers for you, the bridge and the overlay drivers. You can also write a network driver plugin so that you can create your own drivers but that is an advanced task.

Every installation of the Docker Engine automatically includes three default networks. You can list them:

```bash
 $ docker network ls
```
 ```
 NETWORK ID          NAME                DRIVER
18a2866682b8        none                null
c288470c46f6        host                host
7b369448dccb        bridge              bridge
```

The network named bridge is a special network. Unless you tell it otherwise, Docker always launches your containers in this network. Try this now:
```bash
docker run -itd --name=networktest ubuntu
```
Inspecting the network is an easy way to find out the container’s IP address.
```bash
docker network inspect bridge
```

**Network Example**

Create two containers one with the mongo db and the other using the mongo db client
create a new network:
```bash
docker network create -d bridge app-net
```

db:
```bash
docker run -d --network=app-net -p 27017:27017 --name=db mongo:3
```

client:
```bash
docker run -it --rm --network=app-net mongo:3 mongo --host db
```

mongo:
```mongo
db.someCollection.insertOne({x: 1})
show collections
```
