# Jobs Demo

## Create namespace and change context

```bash
kubectl create ns jobs-ns

kubectl ns jobs-ns
```

## Create Job

- Mention all fields from manifest

```bash
kubectl apply -f countdown-job.yaml
```

## Check pods created

```bash
kubectl get pods
```

- Check that pod's status is completed

## Get list of jobs

```bash
kubectl get jobs
```

## Create CronJob

- Mention all fields from manifest

```bash
kubectl apply -f sample-cron-job.yaml
```

- Check pods are created, run and final status is 'completed'

## Get list of cronjobs

```bash
kubectl get cronjobs
```

## Get list of jobs created by cronjob

```bash
kubectl get jobs
```