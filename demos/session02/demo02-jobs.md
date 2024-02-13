# Jobs Demo

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

## Create CronJob

- Mention all fields from manifest    

```bash
kubectl apply -f sample-cron-job.yaml
```

- Check pods are created, run and final status is 'completed'