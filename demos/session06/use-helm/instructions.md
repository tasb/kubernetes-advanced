# How to use Helm

## Add Helm Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo list
```

## Search Helm Repository

```bash
helm search repo
```

## Search Helm Repository with filter

```bash
helm search repo nginx -l
```

## Pull Helm Chart

```bash
helm pull bitnami/nginx --version 15.1.3
```

## Inspect Helm Chart

```bash
helm inspect readme bitnami/nginx
```

## Install Helm Chart

```bash
helm install nginx bitnami/nginx  --version 13.2.16 -n nginx --create-namespace
```

## Check created resources

```bash
kubectl get all -n nginx
```

## List Helm Charts

```bash
helm list
```

## List Helm Charts in specific namespaces

```bash
helm list -n nginx
```

## List Helm Charts in all namespaces

```bash
helm list -A
```

## Upgrade Helm Chart

```bash
helm upgrade nginx bitnami/nginx  --version 15.1.3 -n nginx --set image.tag=latest
```

## Check again created resources

```bash
kubectl get all -n nginx
```

## Generate Helm Chart Values

```bash
helm inspect values bitnami/nginx > values_all.yaml
```

## Upgrade using values.yaml

```bash
helm upgrade nginx bitnami/nginx -n nginx --values values.yaml
```

## Check one more time created resources

```bash
kubectl get all -n nginx
```

## List again Helm Charts

```bash
k ns nginx
helm list
```

## Get Helm Chart History

```bash
helm history nginx -n nginx
```

## Rollback Helm Chart

```bash
helm rollback nginx -n nginx <NUMBER>
```

## Delete Helm Chart

```bash
helm delete nginx  -n nginx
```

## List one last time the Helm Charts

```bash
helm list -A
```

## Check no resources are left

```bash
kubectl get all -n nginx
```

## Delete namespace

```bash
kubectl delete ns nginx
```
