---
title: "Ingress"
full_title: "Lesson 7: Ingress-Controller"
time_estimate: "20 minutes"
weight: 8
date: 2020-10-23T1:00:00-00:98
draft: true
---

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.

### Key Takeaways

1. Understand how to use a tool like Metallb to get an exteralip in your local cluster.
1. See an ingres in action.
1. Understand a bit about Helm as a package manager for k8s.


There are several popular implementations of ingress controller:
* NGINX, Inc. offers support and maintenance for the NGINX Ingress Controller for Kubernetes.
* Contour is an Envoy based ingress controller provided and supported by VMware.
* Ambassador API Gateway is an Envoy based ingress controller with community or commercial support from Ambassador Labs (previously Datawire).
* others: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/


This is what the ingress resource looks like.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```


NOTE: It still needs something like the cloud or metallb to give us an ip. 

Let's go through install metallb and then install an nginx ingress-controller.
**Let's Install It**
Create a cluster
```bash
cat <<EOF > singlenode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
EOF
```
```

There's some configuration we have to tell metallb. It has to know what ip's it can give out. I'm going to do a simple, demo configuration:
```bash
docker network ls
docker network inspect kind | jq .[0].IPAM
```
```json
{
  "Driver": "default",
  "Options": {},
  "Config": [
    {
      "Subnet": "172.22.0.0/16",
      "Gateway": "172.22.0.1"
    },
    {
      "Subnet": "fc00:f853:ccd:e793::/64"
    }
  ]
}
```

"Subnet": "172.22.0.0/16" on the network. So I'm going to claim 100 ip's in the range or 172.22.255.1-172.22.255.250. There better ways to config this.

metallb configuration
```yaml
cat <<EOF > metallb.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.22.255.150-172.22.255.250
EOF
```

BTW... this is a configmap. A ConfigMap is an API object that lets you store configuration for other objects to use. Unlike most Kubernetes objects that have a spec , a ConfigMap has data and binaryData fields. These fields accepts key-value pairs as their values

```bash
kubectl apply -f metallb.yaml
```

Now deploy our azure app:
```bash
git clone https://github.com/wesreisz/azure-voting-app-redis.git
kubectl apply -f azure-vote-all-in-one-redis.yaml --namespace prod
```

Take a look at services now. This is what it looks like when you deploy a loadbalancer service on a public cloud like AKS or EKS
```bash
kubectl get services
```
```
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
azure-vote-back    ClusterIP      10.96.127.21   <none>           6379/TCP       3m51s
azure-vote-front   LoadBalancer   10.96.89.216   172.22.255.150   80:30087/TCP   3m51s
kubernetes         ClusterIP      10.96.0.1      <none>           443/TCP        9m35s
```

Let's install the nginx ingress-controller now, but let's use Helm.

#### Helm
Helm is the best way to find, share, and use software built for Kubernetes. It's a package manager for k8s.

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

Helm is a graduated project in the CNCF and is maintained by the Helm community.


First install Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

Then we can install inginx ingress-controller:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/nginx-ingress-controller
```

Then apply an ingress yaml to configure the route into the service:
```yaml
cat <<EOF > ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: azure-vote
            port:
              number: 80   
EOF
```
Apply it
```bash
kubectl apply -f ingress.yaml
```

```
NAME                                                  TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
azure-vote-back                                       ClusterIP      10.96.127.21    <none>           6379/TCP                     9m52s
azure-vote-front                                      LoadBalancer   10.96.89.216    172.22.255.150   80:30087/TCP                 9m52s
kubernetes                                            ClusterIP      10.96.0.1       <none>           443/TCP                      15m
my-release-nginx-ingress-controller                   LoadBalancer   10.96.13.103    172.22.255.151   80:30086/TCP,443:32473/TCP   8s
my-release-nginx-ingress-controller-default-backend   ClusterIP      10.96.156.159   <none>           80/TCP                       8s
```

