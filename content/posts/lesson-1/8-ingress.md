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


We can install an ingress-controller
```bash
kubectl expose deployment azure-vote-front --type=NodePort --name=azure-vote
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

Then apply this yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: azure-vote
          servicePort: 80
```

```bash
kubectl get ingress
```