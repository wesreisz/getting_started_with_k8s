---
title: "Security"
full_title: "Lesson 6: Security"
time_estimate: "20 minutes"
weight: 6
date: 2020-10-23T1:00:00-00:98
draft: true
---

Kubernetes security is important throughout the container lifecycle due to the distributed, dynamic nature of a Kubernetes cluster. Different security approaches are required for each of the three phases of an application lifecycle: build, deploy, and runtime. Kubernetes provides innate security advantages. For example, application containers are typically not patched or updated — instead, container images are replaced entirely with new versions. This enables strict version control and permits rapid rollbacks if a vulnerability is uncovered in new code.

However, since individual pods are transient and ephemeral, the ever-changing runtime environment can present challenges for IT security professionals, as applications and API links to other applications and services are constantly in flux.

### Key Takeaways

1. Discuss the 4 C's of Cloud Native Security
1. Review a high-level overview of K8s security
1. Understand a bit about the idea of a software supply chain.

#### Security
![4 C's of Cloud Native Security](/getting_started_with_k8s/images/lesson7/4c.png)

Each layer of the Cloud Native security model builds upon the next outermost layer. The Code layer benefits from strong base (Cloud, Cluster, Container) security layers. You cannot safeguard against poor security standards in the base layers by addressing security at the Code level.

**Cloud**

In many ways, the Cloud (or co-located servers, or the corporate datacenter) is the trusted computing base of a Kubernetes cluster. If the Cloud layer is vulnerable (or configured in a vulnerable way) then there is no guarantee that the components built on top of this base are secure. Each cloud provider makes security recommendations for running workloads securely in their environment.

**Cluster**

There are two areas of concern for securing Kubernetes:

* Securing the cluster components that are configurable
* Securing the applications which run in the cluster
* Look 

**Container**
Container security is outside the scope of this guide. Here are general recommendations and links to explore this topic:

|Area of Concern for Containers   | Recommendation  |
|:-|:-|
|Container Vulnerability Scanning and OS Dependency Security   | As part of an image build step, you should scan your containers for known vulnerabilities.  |
|Image Signing and Enforcement |	Sign container images to maintain a system of trust for the content of your containers.|
|Disallow privileged users |	When constructing containers, consult your documentation for how to create users inside of the containers that have the least level of operating system privilege necessary in order to carry out the goal of the container.|


**Code**

Application code is one of the primary attack surfaces over which you have the most control. While securing application code is outside of the Kubernetes security topic, here are recommendations to protect application code:

|Area of Concern for Containers   | Recommendation  |
|:-|:-|
|Access over TLS only	| If your code needs to communicate by TCP, perform a TLS handshake with the client ahead of time. With the exception of a few cases, encrypt everything in transit. Going one step further, it's a good idea to encrypt network traffic between services. This can be done through a process known as mutual or mTLS which performs a two sided verification of communication between two certificate holding services.|
|Limiting port ranges of communication |	This recommendation may be a bit self-explanatory, but wherever possible you should only expose the ports on your service that are absolutely essential for communication or metric gathering.|
|3rd Party Dependency Security |	It is a good practice to regularly scan your application's third party libraries for known security vulnerabilities. Each programming language has a tool for performing this check automatically.|
|Static Code Analysis |	Most languages provide a way for a snippet of code to be analyzed for any potentially unsafe coding practices. Whenever possible you should perform checks using automated tooling that can scan codebases for common security errors. Some of the tools can be found at: https://owasp.org/www-community/Source_Code_Analysis_Tools |
|Dynamic probing attacks |	There are a few automated tools that you can run against your service to try some of the well known service attacks. These include SQL injection, CSRF, and XSS. One of the most popular dynamic analysis tools is the OWASP Zed Attack proxy tool.|


ref: https://kubernetes.io/docs/concepts/security/overview/

#### What is a high-level Kubernetes security checklist?
Best practice recommendations include:

* Begin with minimal, distro-less images and add only what is absolutely necessary. Smaller is safer.
* Use a minimal host OS, end read-only mounts, and use SELinux options for even more control
* Scan images for vulnerabilities top to bottom, including OS images and outside images of any kind. There is no such thing as an outside trusted source.
* Use namespaces and RBAC to segment the cluster and users in a logical manner. If it’s not needed it shouldn’t be visible.
* Kubernetes networking defaults allow any-to-any communications, so network segmentation should be implemented before production. Carefully define ingress and egress policies to ensure connections are routed properly.
* Keep privileges to a minimum, and never run application processes as root. Using a read-only root filesystem prevents any attack that depends on installing software or modifying the file system.
* Integrate security such as image scanning into the CI/CD pipeline. Even better, run CIS Benchmark security tests.
* Secure the cluster itself. Configure RBAC to limit access to the API server and ensure all etcd communications are secured with TLS encryption. Likewise, lockdown kubelet permissions by configuring RBAC for kubelets.
* Take advantage of built-in controls in Kubernetes, such as configuring security context to limit pod access.
* Proactive security should include monitoring of process activity, communications between services, and communications external to the cluster.


### Software Supply Chain

The supply chain starts with the idea that you can coordinate all production and nonproduction activities as a single system. Supply chain management is often misunderstood as being simply “supplier management,” when really that is only one aspect of supply chain management (though a critical one).
* Planning: In the traditional supply chain, planning activities involve coordinating assets and optimizing the flow of processes to balance supply of materials with demand for products. In the software supply chain, that coordination involves making sure the right code is being developed for the product features that are needed the most. At large scale, with hundreds of applications and thousands of software developers, this is a monumental endeavor.
* Sourcing: Software also relies heavily on sourced components—according to recent research by Sonatype, open source now forms the majority of software products: as much as 80 to 90 percent of the code in modern applications is from open source components. And these components create unique management challenges.
* Distribution: How software is delivered (ci/cd) pipeline. Is it secure through the process.
* Tools: The software supply chain involves a class of tools (variously called ALM tools, life cycle tools, or devops tools) that are used to manage the various stages of software delivery.

https://www.infoworld.com/article/3245800/building-a-model-of-the-software-supply-chain.html



