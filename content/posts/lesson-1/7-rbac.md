---
title: "RBAC"
full_title: "Lesson 7: RBAC"
time_estimate: "20 minutes"
weight: 7
date: 2020-10-23T1:00:00-00:98
draft: true
---

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

RBAC authorization uses the rbac.authorization.k8s.io API group to drive authorization decisions, allowing you to dynamically configure policies through the Kubernetes API.

### Key Takeaways

1. Understand the basic idea of how k8s handles authentication and authorization.
1. Understand how Roles/ClusterRoles and RoleBinding/ClusterRoleBinding works.
1. Practice using roles/rolebinding.

### Authorization and Authentication in K8s

Authentication is the act of validating that users are whom they claim to be. This is the first step in any security process. 

Authorization in a system security is the process of giving the user permission to access a specific resource or function. This term is often used interchangeably with access control or client privilege. 

https://kubernetes.io/docs/reference/access-authn-authz/authentication/

#### RBAC

The RBAC API declares four kinds of Kubernetes object: 
* Role 
* ClusterRole
* RoleBinding
* ClusterRoleBinding. 

You can describe objects, or amend them, using tools such as kubectl, just like any other Kubernetes object.

#### Role and ClusterRole
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

#### RoleBinding and ClusterRoleBinding
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

**Lab Exercise**

Let's create a new cluster. Remember, we created a new yml file before. We can use it for the new cluster. 
```yaml
cat <<EOF > singlenode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
EOF
```

Create a new cluster
```bash
kind create cluster --config singlenode.yaml
```

Let's look at what apis are available:
```bash
kubectl api-versions
```

Create an a production namespace. Deploy an app into the default and prod namespace.
```bash
kubectl get namespaces
kubectl create namespace prod

kubectl run nginx --image=nginx --replicas=1
kubectl run nginx --image=nginx --replicas=1 --namespace prod
```

Create a new serviceaccount
```bash
kubectl create serviceaccount udef-pod-reader -n default
kubectl get serviceaccount
```

Create a role that can only read/list/watch the default namespace:
```yml
cat <<EOF > default-pod-role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1 
metadata:
   name: udefreadonly
   namespace: default
rules:
 - apiGroups: [""]
   resources: ["pods"]
   verbs: ["get", "list", "watch"]
EOF
```
```bash
kubectl apply -f default-pod-role.yaml 
```

Create a rolebinding to associate the serviceaccount with the role:
```yaml
cat <<EOF > default-pod-binding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
   name: udefreadonlybinding
   namespace: default
subjects:
 - kind: ServiceAccount
   name: udef-pod-reader
   namespace: default
roleRef:
   kind: Role
   name: udefreadonly
   apiGroup: rbac.authorization.k8s.io
EOF
```

```bash
kubectl apply -f default-pod-binding.yaml
```

Setup your context to access and test.
```bash
TOKEN=$(kubectl describe secrets "$(kubectl describe serviceaccount udef-pod-reader -n default| grep -i Tokens | awk '{print $2}')" -n default | grep token: | awk '{print $2}')
kubectl config set-credentials test-user --token=$TOKEN
kubectl config get-clusters
kubectl config set-context udefpodreader --cluster=kind-kind --user=test-user
kubectl config view
kubectl config get-contexts
kubectl config use-context udefpodreader
```

Now let's test it:
```bash
kubectl get pods
kubectl get pods -n prod
kubectl get pods -A
```


**The key to understanding RBAC in Kubernetes**

In order to fully grasp the idea of RBAC, we must understand that three elements are involved:

* Subjects: The set of users and processes that want to access the Kubernetes API.
* Resources: The set of Kubernetes API Objects available in the cluster. Examples include Pods,  Deployments, Services, Nodes, and PersistentVolumes, among others.
* Verbs: The set of operations that can be executed to the resources above. Different verbs are available (examples: get, watch, create, delete, etc.), but ultimately all of them are Create, Read, Update or Delete (CRUD) operations.

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1 
metadata:
   name: udefreadonly
   namespace: default
rules:
 - apiGroups: [""]
   resources: ["pods"]
   verbs: ["get", "list", "watch"]
```

We want to connect subjects, API resources, and operations. In other words, we want to specify, given a user, which operations can be executed over a set of resources.

