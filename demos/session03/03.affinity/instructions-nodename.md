# Nodename demo

## Create namespace for demos

```bash
kubectl create ns affinity

kubectl ns affinity
```

## Create deployment

```bash
kubectl apply -f nodename.yaml
```

## Check the pods weren't scheduled

```bash
kubectl get pods -o wide
```

## Verify why the pods are pending

```bash
k describe pod <POD_NAME>
```

## Update deployment

- Change nodename on manifest file

```bash
kubectl apply -f nodename.yaml
```

## Check where pods were scheduled

```bash
kubectl get pods -o wide
```
