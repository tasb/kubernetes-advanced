# Affinity demo

## Create namespace for demos

```bash
kubectl create ns affinity

kubectl ns affinity
```

## nodeName sample

- Create deployment

```bash
kubectl apply -f nodename.yaml
```

- Check where pods were scheduled

```bash
kubectl get pods -o wide
```

## nodeAffinity sample

- Check nodes tags

```bash
kubectl get nodes -L parity -L disktype
```

- Create deployment

```bash
kubectl apply -f node-affinity.yaml
```

- Check where pods were scheduled

```bash
kubectl get pods -o wide
```

## podAffinity sample

- Create deployments

```bash
kubectl apply -f pod-affinity-store.yaml

kubectl apply -f pod-affinity-webstore.yaml
```

- In the following example Deployment for the Redis cache, the replicas get the label app=store. The podAntiAffinity rule tells the scheduler to avoid placing multiple replicas with the app=store label on a single node. This creates each cache in a separate node.

- The following example Deployment for the web servers creates replicas with the label app=web-store. The Pod affinity rule tells the scheduler to place each replica on a node that has a Pod with the label app=store. The Pod anti-affinity rule tells the scheduler never to place multiple app=web-store servers on a single node.

- Check where pods were scheduled

```bash
kubectl get pods -o wide
```
