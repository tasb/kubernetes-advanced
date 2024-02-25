# Init Container Demos

## Create namespace and change kubectl

```bash
kubectl create ns init-ns

kubectl ns init-ns
```

## Apply file with all resources

```bash
kubectl apply -f init-container.yml
```

## Check all the changes on pods

```bash
kubectl get pods -n init-ns --watch
```

## Navigate to service

```bash
kubectl get svc -n init-ns
```

## Enable tunnel to service

```bash
minikube tunnel -p labs
```

1. Get External IP on services list
2. Curl to http://EXTERNAL_IP:12000
