# Probes Demos

## Create namespace and change kubectl

```bash
kubectl create ns probes-ns

kubectl ns probes-ns
```

## Open cluster info

```bash
kubectl port-forward svc/clusterinfo 5252:5252 -n clusterinfo --address 0.0.0.0
```

## Prepare your environment

Open 3 consoles:

1. To check pod logs
2. To check service endpoints
3. To check pods status
4. To run additional commands

## Create resources and open logs from first pod

```bash
kubectl apply -f probes.yaml

POD_NAME=$(kubectl get pod  -o jsonpath="{.items[0].metadata.name}")

kubectl logs -f $POD_NAME
```

## Check endpoints and pods

```bash
kubectl get endpoints --watch
```

```bash
kubectl get pods --watch
```

## Check probes working

1. First startup is finished. After that, pod status changed
2. Wait until "App not ready!" log and check the change on endpoints
3. Wait until "App not ready!" stops and check endpoints update
4. Wait until "App not running!" shows and count 5 instances. Then pod is restarted

## Update deployment

- Change image to "latest" image
- Check the behaviour on ClusterInfo
