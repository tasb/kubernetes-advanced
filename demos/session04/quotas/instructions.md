# Quotas

## Create namespace

```bash
k create ns quotas-ns
k ns quotas-ns
```

## Create quotas

```bash
k apply -f pod-quota.yml
k apply -f resources-quota.yml
```

## List quotas

```bash
k get resourcequota
```

## Create the resource quota deployment

```bash
k apply -f resources-quota-demo.yml
```

## List the pods

```bash
k get pods
```

- Check that only one pod is running

## Describe ReplicaSet

```bash
k describe rs resources-quota-demo-XXXX
```

- Check that new pods are not being created due to the resource quota

## Create pod quota deployment

```bash
k apply -f pod-quota-demo.yml
```

- Check no pods running

## Get error on ReplicaSet

```bash
k describe rs pod-quota-demo-XXXX
```

- Check the error is caused because no resources defined on deployment

## Update deployment with resources

- uncomment on manifest

```bash
k apply -f pod-quota-demo.yml
```

- Check that you have one pod running

## Review ReplicaSet to check the reason

```bash
k describe rs pod-quota-demo-XXXX
```

## Cleaup namespace

```bash
k ns default

k delete ns quotas-ns
```
