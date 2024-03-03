# Namespace NetworkPolicies

## Create namespaces

```bash
kubectl apply -f namespaces.yaml

kubectl get ns --show-labels
```

## Create deploys

```bash
kubectl apply -f app-deploy.yml -n dev-ns

kubectl apply -f app-deploy.yml -n prod-ns
```

## Check you can access from one namespace to the other

```bash
kubectl exec -it myapp-deploy-XXXX -n dev-ns -- bash

> curl http://myapp-svc:10100
> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
```

## Apply namespace restriction network policies

```bash
kubectl apply -f from-namespace/
```

## Check that access is not allowed now

```bash
kubectl exec -it myapp-deploy-XXXX -n dev-ns -- bash

> curl http://myapp-svc:10100
> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
```

## Check that using other pod allow to use pods on same namespace

```bash
kubectl run -it nginx -n prod-ns --image=nginx bash

> curl http://myapp-svc.prod-ns.svc.cluster.local:10100
> curl http://myapp-svc.dev-ns.svc.cluster.local:10100
```
