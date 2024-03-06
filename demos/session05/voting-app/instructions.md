# NetworkPolicies Demo

## Create a new namespace

```bash
kubectl create ns vote-ns

kubectl ns vote-ns
```

## Create deployments

```bash
kubectl apply -f voting-app-back.yml

kubectl apply -f voting-app-front.yml
```

## Check you can reach REDIS from front

```bash
kubectl exec -it voting-app-front-XXX -- bash

> redis-cli -h voting-app-back-svc
```

## Check you can reach REDIS from other pod

```bash
kubectl run redis --image=redis

kubectl exec -it redis -- bash

> redis-cli -h voting-app-back-svc
```

## Apply netpol

```bash
kubectl apply -f network-policies
```

## Check the access to REDIS from both pods

```bash
kubectl exec -it voting-app-front-XXX -- bash

> redis-cli -h voting-app-back-svc

kubectl exec -it redis -- bash

> redis-cli -h voting-app-back-svc
```
