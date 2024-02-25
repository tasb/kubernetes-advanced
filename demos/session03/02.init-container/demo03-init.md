# Init Container Demos

## Apply file with all resources 

```bash
kubectl apply -f init-containers
```

## Check all the changes on pods

```bash
kubectl get pods -n init-ns --watch
```

## Navigate to service

```bash
kubectl get svc -n init-ns
```

1. Get External IP on services list
2. Open browser and navigate to http://EXTERNAL_IP:12000
