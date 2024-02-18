# StatefulSet Demo

## Create namespace and change context

```bash
kubectl create ns sts-ns

kubectl ns sts-ns
```

## Create STS and Headless Service

```bash
kubectl apply -f stateful-sets.yml
```

## Check created resources

### Get stateful sets

```bash
kubectl get sts
```

### Get services

```bash
kubectl get svc
```

- Check that headless service don't have any IP

### Get pods

```bash
kubectl get pods -o wide
```

### Delete one pod

```bash
kubectl delete pod sts-sample-0
```

### Check pods again

```bash
kubectl get pods -o wide
```

- Check that another pod was created to replace the deleted one with same name and IP

## Run pod to make queries to servers

```bash
kubectl run -it dnsutils --image=ghcr.io/theonorg/dnsutils:ubuntu bash
```

## Run commands inside container

```bash
nslookup sts-svc

curl sts-svc.sts-ns.svc.cluster.local:8080
curl sts-sample-0.sts-svc.sts-ns.svc.cluster.local:8080
curl sts-sample-1.sts-svc.sts-ns.svc.cluster.local:8080
curl sts-sample-2.sts-svc.sts-ns.svc.cluster.local:8080

exit
```

## Get PVC

```bash
kubectl get pvc
```

## Get PV

```bash
kubectl get pv
```

- Check that PVs were created dynamically. You can create static PVs to be used by PVCs
- Check `persistentVolumeClaimRetentionPolicy` on k8s manifest

## Delete STS

```bash
kubectl delete sts sts-sample
```

## Check PVC and PV again

```bash
kubectl get pvc

kubectl get pv
```

## Clean namespace

```bash
kubectl ns default

kubectl delete ns sts-ns
```
