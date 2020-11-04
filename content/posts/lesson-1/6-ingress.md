---
title: "Ingress"
full_title: "Lesson 6: Ingress"
time_estimate: "20 minutes"
weight: 6
date: 2020-10-23T1:00:00-00:98
draft: true
---

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut eu sollicitudin purus. Nam efficitur leo a finibus accumsan. Cras finibus tempus leo ac gravida. Suspendisse faucibus hendrerit felis eget sagittis. Cras molestie sollicitudin neque ut porta. Donec volutpat lobortis tempus. Curabitur non nisi placerat, posuere velit ut, porta lacus. Nulla eu congue nulla, id porttitor risus. Vestibulum metus massa, fermentum sagittis enim eget, tempor commodo odio. Nunc suscipit euismod diam sit amet feugiat. Duis feugiat sed nisi eu molestie. Nam aliquam nisi ut luctus pharetra. Sed quis risus ultrices, congue justo in, ornare odio.

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