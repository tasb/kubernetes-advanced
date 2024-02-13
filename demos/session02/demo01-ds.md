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

## Check DaemonSet manifest

- On this manifest, mention nodeSelector that makes a pod to be deployed on every node that match nodeSelector

## Navigate to LoadBalancer

- Check service IP

```bash
kubectl get svc
```

- Get External IP and navigate on browser and check node name changes

## Clean up namespace

```bash
kubectl ns default

kubectl delete ns ds-ns
```
