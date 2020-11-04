---
title: "Security"
full_title: "Lesson 7: Security"
time_estimate: "20 minutes"
weight: 7
date: 2020-10-23T1:00:00-00:98
draft: true
---

Kubernetes security is important throughout the container lifecycle due to the distributed, dynamic nature of a Kubernetes cluster. Different security approaches are required for each of the three phases of an application lifecycle: build, deploy, and runtime. Kubernetes provides innate security advantages. For example, application containers are typically not patched or updated — instead, container images are replaced entirely with new versions. This enables strict version control and permits rapid rollbacks if a vulnerability is uncovered in new code.

However, since individual pods are transient and ephemeral, the ever-changing runtime environment can present challenges for IT security professionals, as applications and API links to other applications and services are constantly in flux.

### Key Takeaways

1. Understand about more complicated configurations for architecture components
1. Explore the CNCF Cloud Native Landscape around Kubernetes

#### Security
![4 C's of Cloud Native Security](/getting_started_with_k8s/images/lesson7/4c.png)

Each layer of the Cloud Native security model builds upon the next outermost layer. The Code layer benefits from strong base (Cloud, Cluster, Container) security layers. You cannot safeguard against poor security standards in the base layers by addressing security at the Code level.

**Cloud**

In many ways, the Cloud (or co-located servers, or the corporate datacenter) is the trusted computing base of a Kubernetes cluster. If the Cloud layer is vulnerable (or configured in a vulnerable way) then there is no guarantee that the components built on top of this base are secure. Each cloud provider makes security recommendations for running workloads securely in their environment.

**Cluster**

There are two areas of concern for securing Kubernetes:

* Securing the cluster components that are configurable
* Securing the applications which run in the cluster

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


#### RBAC
Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

RBAC authorization uses the rbac.authorization.k8s.io API group to drive authorization decisions, allowing you to dynamically configure policies through the Kubernetes API.

The RBAC API declares four kinds of Kubernetes object: 
* Role 
* ClusterRole
* RoleBinding
* ClusterRoleBinding. 

You can describe objects, or amend them, using tools such as kubectl, just like any other Kubernetes object.

Role and ClusterRole
An RBAC Role or ClusterRole contains rules that represent a set of permissions. Permissions are purely additive (there are no "deny" rules).

A Role always sets permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.

ClusterRole, by contrast, is a non-namespaced resource. The resources have different names (Role and ClusterRole) because a Kubernetes object always has to be either namespaced or not namespaced; it can't be both.

ClusterRoles have several uses. You can use a ClusterRole to:

1. define permissions on namespaced resources and be granted within individual namespace(s)
1. define permissions on namespaced resources and be granted across all namespaces
define permissions on cluster-scoped resources
1. If you want to define a role within a namespace, use a Role; if you want to define a role cluster-wide, use a ClusterRole.

Grants read permissions to pods (default namespace)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

**RoleBinding and ClusterRoleBinding**
A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. A RoleBinding grants permissions within a specific namespace whereas a ClusterRoleBinding grants that access cluster-wide.

A RoleBinding may reference any Role in the same namespace. Alternatively, a RoleBinding can reference a ClusterRole and bind that ClusterRole to the namespace of the RoleBinding. If you want to bind a ClusterRole to all the namespaces in your cluster, you use a ClusterRoleBinding.

The name of a RoleBinding or ClusterRoleBinding object must be a valid path segment name.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```



**ClusterRole**
A ClusterRole can be used to grant the same permissions as a Role. Because ClusterRoles are cluster-scoped, you can also use them to grant access to:

* cluster-scoped resources (like nodes)
* non-resource endpoints (like /healthz)
* namespaced resources (like Pods), across all namespaces For example: you can use a ClusterRole to allow a particular user to run kubectl get pods --all-namespaces.
Here is an example of a ClusterRole that can be used to grant read access to secrets in any particular namespace, or across all namespaces (depending on how it is bound):

**Example**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

More Details: https://kubernetes.io/docs/reference/access-authn-authz/rbac/


#### Creating the Rights to Access the K8s API

https://medium.com/@nieldw/curling-the-kubernetes-api-server-d7675cfc398c

