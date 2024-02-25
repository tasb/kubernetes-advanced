# Lab 06 - Different Workloads

## Table of Contents

- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Create a StatefulSet to control your database](#step-1-create-a-statefulset-to-control-your-database)
  - [Step 02: Update your secret](#step-02-update-your-secret)
  - [Step 03: Restart API deployment](#step-03-restart-api-deployment)
  - [Step 4: Test your application](#step-4-test-your-application)
  - [Step 5:  Enable database backup](#step-5--enable-database-backup)
  - [Step 6: Check backup data](#step-6-check-backup-data)
- [Conclusion](#conclusion)

## Objectives

On this lab you'll use a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) to control your database and a [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) to backup your database.

## Prerequisites

- Finish properly [Lab #01](./lab-01.md)
- Having your `minikube` cluster running

## Guide

### Step 1: Create a StatefulSet to control your database

You used a deployment as a controller for your database but you should use a StatefulSet since you need to have control about replicas created to handle databases.

At same time, with a StatefulSet you have a full match between a pod and a PersistentVolumeClaim.

Create a file named `echo-app-db.yaml` and add the following content.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: echo-db-sts
  namespace: echo-app-ns
spec:
  selector:
    matchLabels:
      app: echo-app
      tier: db
  serviceName: "echo-db-sts-svc"
  replicas: 1
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Retain
  template:
    metadata:
      labels:
        app: echo-app
        tier: db
    spec:
      containers:
      - name: echo-db
        image: mcr.microsoft.com/mssql/server:2017-latest
        ports:
        - containerPort: 1433
        env:
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: echo-api-db-secret
              key: dbpass
        volumeMounts:
        - name: echo-app-db-pv-claim
          mountPath: /var/opt/mssql/data
  volumeClaimTemplates:
  - metadata:
      name: echo-app-db-pv-claim
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi


---

apiVersion: v1
kind: Service
metadata:
  name: echo-db-sts-svc
  namespace: echo-app-ns
spec:
  ports:
    - port: 1433
      targetPort: 1433
      name: db
  selector:
    app: echo-app
    tier: db
  clusterIP: None

```

On this file you create a StatefulSet resource and the headless service needed to be used by pods to reach each pod of the StatefulSet.

Look that you create a PVC template that will automatically create a PVC and make the link between the pods and PVC.

Using this resource, you have a well defined structure for the names of all resources which allow you to handle more the dynamic behavior of both resources.

Now you need to start change your app. During the next steps your application may me broken and not working properly.

First, delete database deployment.

```bash
kubectl delete deploy echo-db-dep -n echo-app-ns
```

Then, create StatefulSet and Headless Service.

```bash
kubectl apply -f echo-app-db.yaml
```

To confirm that your StatefulSet is running properly, you should list all StatefulSets on your cluster.

```bash
kubectl get sts -n echo-app-ns
```

You should get an output similar with this.

```bash
NAME          READY   AGE
echo-db-sts   1/1     5h15m
```

### Step 02: Update your secret

Now you need to change your secret to reflect the new name of your server.

Using StatefulSet definition, your database is now located on `echo-db-sts-0.echo-db-sts-svc.echo-app-ns.svc.cluster.local`.

Then, you need to encode your connection string to Base64.

```bash
echo "Server=echo-db-sts-0.echo-db-sts-svc.echo-app-ns.svc.cluster.local,1433;Initial Catalog=echo-log;User ID=SA;Password=P@ssw0rd" | base64 -w 0
```

Now, copy to the clipboard the result and let's update the secret already deployed.

```bash
kubectl edit secret echo-api-db-secret -n echo-app-ns
```

This command will open your preferred code editor on your machine. On the editor, you need to change the value of the key `.data.connString` with the value you copied to the clipboard.

When you close the code editor, the new definition is automatically updated on your cluster.

### Step 03: Restart API deployment

Since you mount the secret as an environment variable, you need to restart your pods to reflect the change.

You only need to run the following command for your pods to be recreated using a rolling update strategy.

```bash
kubectl rollout restart deploy/echo-api-dep -n echo-app-ns
```

You may check that your pods were restarted.

```bash
kubectl get pods -n echo-app-ns
```

And you should check that on column `AGE` you have time around few seconds.

### Step 4: Test your application

Now your application should be working properly again.

You can open a browser and navigate to <http://echo-app.ingress.test> and test your Echo App!

### Step 5:  Enable database backup

To backup database data let's create a CronJob to run periodically to copy your database data to a target folder.

Start to create a file named `echo-app-backup.yaml` and add the following content.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: echo-app-db-backup
  namespace: echo-app-ns
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

---

apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-app-db-backup
  namespace: echo-app-ns
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: echo-app
            tier: backup
        spec:
          containers:
          - name: db-backup
            image: busybox
            args:
            - /bin/sh
            - -c
            - cp -r /backup/source/* /backup/target
            - sleep 3
            volumeMounts:
            - name: db-files
              mountPath: /backup/source
            - name: backup-folder
              mountPath: /backup/target
          restartPolicy: OnFailure
          volumes:
          - name: db-files
            persistentVolumeClaim:
              claimName: echo-app-db-pv-claim-echo-db-sts-0
          - name: backup-folder
            persistentVolumeClaim:
              claimName: echo-app-db-backup
```

On this file you create a PVC to handle your target folder to save database files and a CronJob to run periodically and copy files from one folder to another.

The job will run every minute due to `schedule: "* * * * *"`. You can use [crontab guru](https://crontab.guru/#*_*_*_*_*) to generate different schedules.

Let's create the cronjob.

```bash
kubectl apply -f echo-app-backup.yaml -n echo-app-ns
```

Check if your CronJob where created properly.

```bash
kubectl get cj -n echo-app-ns
```

You should have an output similar with this.

```bash
NAME                 SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
echo-app-db-backup   * * * * *   False     0        20s             86m
```

And finally, you should confirm if a pod ran successfully to copy the files.

```bash
kubectl get pods -n echo-app-ns -l tier=backup
```

And you should get a list with at least one pod with status `Completed`.

If you run this command several times, you can confirm that you have an history of 3 pods.

### Step 6: Check backup data

Finally, let's confirm that the backup was made properly.

First, you need to verify in which node the pod is running.

```bash
kubectl get pods -n echo-app-ns -o wide
```

Now, let's list all your PersistentVolumes and find out the name of the PV bound to the PVC named `echo-app-db-backup`.

```bash
PV_NAME=$(kubectl get pv -o jsonpath='{.items[?(@.spec.claimRef.name=="echo-app-db-backup")].metadata.name}')
```

You should get the name of the PV as the output of last command.

Now, you can get the path where the data is stored.

```bash
kubectl describe pv $PV_NAME
```

On the output you get several details from the PV but you should focus on `Path` property.

Finally, let's enter the node and check if the folder is full of database files.

If the pods is running on node `labs`, you should run the following command.

```bash
minikube ssh -p labs
```

If the pod is running on the node `labs-m02`, you should run the following command.

```bash
minikube ssh -p labs -n labs-m02
```

After being inside the node, navigate to the path you got from PV details and check if the folder is full of database files.

To exit from the node, just run the command `exit`.

## Conclusion

You just finish this lab and built a more stable definition of a database running on a Kubernetes Cluster.

On this lab you learned how to use a StatefulSet to control your database and a CronJob to backup your database data.
