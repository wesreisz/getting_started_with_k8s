---
title: "Security"
full_title: "Lesson 5: Security"
time_estimate: "20 minutes"
weight: 7
date: 2020-10-23T1:00:00-00:98
draft: true
---

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage/network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

### Key Takeaways

1. Understand about more complicated configurations for architecture components
1. Explore the CNCF Cloud Native Landscape around Kubernetes

![](/getting_started_with_k8s/images/lesson3/k8s-arch3-thanks-weave.png)

#### Security

https://kubernetes.io/docs/concepts/security/overview/


#### Securing a cluster

https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/