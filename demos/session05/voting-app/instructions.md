# NetworkPolicies Demo

## Create namespaces

```bash
kubectl apply -f namespaces.yaml

kubectl get ns --show-labels
```

## Deny all communication between namespaces

- Create deploys

```bash
kubectl apply -f app-deploy.yml -n dev-ns

kubectl apply -f app-deploy.yml -n prod-ns
```

- Check you can access from one namespace to the other

```bash
kubectl exec -it myapp-deploy-XXXX -n dev-ns -- bash

> curl http://myapp-svc:10100
> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
```

- Apply namespace restriction network policies

```bash
kubectl apply -f from-namespace/
```

- Check that access is not allowed now

```bash
kubectl exec -it myapp-deploy-XXXX -n dev-ns -- bash

> curl http://myapp-svc:10100
> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
```

- Check that using other pod allow to use pods on same namespace

```bash
kubectl run -it nginx -n prod-ns --image=nginx bash

> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
> curl http://myapp-svc.dev-ns.svc.cluster.local:10100
```

## Allow communication only from pods from same app

- Create a new namespace

```bash
kubectl create ns vote-ns

kubectl ns vote-app
```

- Create deployments

```bash
kubectl apply -f voting-app/voting-app-back.yml

kubectl apply -f voting-app/voting-app-front.yml
```

- Check you can reach REDIS from front

```bash
kubectl exec -it voting-app-front-XXX -- bash

> redis-cli -h voting-app-back-svc
```

- Check you can reach REDIS from other pod

```bash
kubectl run redis --image=redis

kubectl exec -it redis -- bash

> redis-cli -h voting-app-back-svc
```

- Apply netpol

```bash
kubectl apply -f voting-app/
```

- Check the access to REDIS from both pods

```bash
kubectl exec -it voting-app-front-XXX -- bash

> redis-cli -h voting-app-back-svc

kubectl exec -it redis -- bash

> redis-cli -h voting-app-back-svc
```
