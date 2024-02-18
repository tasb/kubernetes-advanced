# DaemonSet Demo

## Create namespace and change context

```bash
kubectl create ns ds-ns

kubectl ns ds-ns
```

## Create DaemonSet and LoadBalancer service

```bash
kubectl apply -f info-pod-ds.yml
```

## Check pods in every node

```bash
kubectl get pods -o wide
```

## Clean up namespace

```bash
kubectl ns default

kubectl delete ns ds-ns
```
