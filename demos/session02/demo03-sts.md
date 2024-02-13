# StatefulSet Demo

## Create namespace and change context

```bash
kubectl create ns sts-ns

kubectl ns sts-ns
```

## Create common deployment to check diferences

```bash
kubectl apply -f hpa-sample.yaml
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

## Run pod to make queries to servers

```bash
kubectl run -it dnsutils --image=ghcr.io/theonorg/dnsutils:ubuntu bash
```

## Run commands inside container

```bash
nslookup hpa-dotnet-svc

nslookup sts-svc

curl sts-sample-0.sts-svc.default.svc.cluster.local
curl sts-sample-1.sts-svc.default.svc.cluster.local
curl sts-sample-2.sts-svc.default.svc.cluster.local

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

- Check that PVs were created dinamically. You can create static PVs to be used by PVCs
- Check `persistentVolumeClaimRetentionPolicy` on k8s manifest

## Delete STS

```bash
kubectl delete sts sts-sample
```

## Clean namespace

```bash
kubectl ns default

kubectl delete ns sts-ns
```
