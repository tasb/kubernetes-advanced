# Taint & Toleration Demo

## Create namespace

```bash
kubectl create ns taints

kubectl ns taints
```

## Create a taint

```bash
kubectl taint nodes tiberna-rasp-001 PriorityLevel=Low:NoSchedule
kubectl taint nodes tiberna-rasp-002 PriorityLevel=Normal:NoSchedule
kubectl taint nodes tiberna-rasp-003 PriorityLevel=Critical:NoSchedule
```

- Check taints on your nodes

```bash
kubectl get nodes -o json | jq ".items[]|{name:.metadata.name, taints:.spec.taints}"
```

- Create deploys

```bash
kubectl apply -f taints/
```

- Check pods nominated nodes

```bash
kubectl get pods -o wide
```

## Create NoExecute taints

- Add pods to be evicted

```bash
kubectl apply -f nodename.yaml
```

- Add NoExecute taint

```bash
kubectl taint nodes tiberna-hp not-ready=true:NoExecute
```

- Check some pods starting to be in Terminating status

```bash
kubectl get pods -o wide -A | grep tiberna-hp
```

- Calico and Clusterinfo don't terminated because have tolerations. Let's check.

```bash
kubectl get ds calico-node -n kube-system -o json | jq ".spec.template.spec.tolerations"
```

## Delete taints

```bash
kubectl taint nodes tiberna-rasp-001 PriorityLevel=Low:NoSchedule-
kubectl taint nodes tiberna-rasp-002 PriorityLevel=Normal:NoSchedule-
kubectl taint nodes tiberna-rasp-003 PriorityLevel=Critical:NoSchedule-
kubectl taint nodes tiberna-hp not-ready=true:NoExecute-
```
