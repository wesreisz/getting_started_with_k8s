---
title: "Ingress"
full_title: "Lesson 6: Ingress"
time_estimate: "20 minutes"
weight: 6
date: 2020-10-23T1:00:00-00:98
draft: true
---


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