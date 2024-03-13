# Author your chart

## Create a new chart

```bash
helm create my-app
```

- Show the chart structure

## Review voting-app chart

- Show the chart structure
- Show the values.yaml file
- Show the templates directory

## Lint the chart

```bash
helm lint voting-app/
```

## Generate local manifest

```bash
helm template --debug voting-app/
```

## Do a dry-run install

```bash
helm install --dry-run --debug --generate-name voting-app
```

## Install the chart

```bash
helm install voting-app voting-app/
```

## Check the release

```bash
helm list
```

## Check the pods

```bash
kubectl get pods
```

## Check other resources

```bash
kubectl get all
```

## Get the manifest

```bash
helm get manifest voting-app
```

## Upgrade the chart

- Change service type to ClusterIP in the values.yaml file
- Change the version in the Chart.yaml file

```bash
helm upgrade voting-app voting-app/
```

## Check release notes

```bash
helm history voting-app
```

## Test the chart

```bash
helm test voting-app
```

## Clean up

```bash
helm uninstall voting-app
```
