# Affinity demo

## Create deployment

```bash
kubectl apply -f node-affinity.yaml
```

## Check pods are not scheduled

```bash
kubectl get pods -o wide
```

## Check details of the pods

```bash
kubectl describe pod <POD_NAME>
```

## Comment nodeSelector from manifest

```bash
kubectl apply -f node-affinity.yaml
```

## Check again that the pods are not scheduled

```bash
kubectl get pods -o wide
```

## Change nodeAfinnity

Change requiredDuringSchedulingIgnoredDuringExecution to preferredDuringSchedulingIgnoredDuringExecution

```bash
kubectl apply -f node-affinity.yaml
```

## Check where pods were scheduled

```bash
kubectl get pods -o wide
```

## Check nodes tags

```bash
kubectl get nodes -L parity -L disktype
```

## Change Affinity on manifest

Change the nodeSelector block to preference block

```bash
kubectl apply -f node-affinity.yaml
```

## Check now where pods were scheduled

```bash
kubectl get pods -o wide
```

## Change weight on preference

Change the weight on preference block

```bash
kubectl apply -f node-affinity.yaml
```

## Check the new pods location

```bash
kubectl get pods -o wide
```
