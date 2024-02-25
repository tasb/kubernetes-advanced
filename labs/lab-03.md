# Lab 03 - Kubernetes Scheduling

## Table of Contents

- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Define readiness and liveness probe on StatefulSet](#step-1-define-readiness-and-liveness-probe-on-statefulset)
  - [Step 2: Define readiness and liveness probe on Echo API Deployment](#step-2-define-readiness-and-liveness-probe-on-echo-api-deployment)
  - [Step 3: Use node affinity to schedule StatefulSet pods](#step-3-use-node-affinity-to-schedule-statefulset-pods)
  - [Step 4: Use pod affinity to schedule database backup](#step-4-use-pod-affinity-to-schedule-database-backup)
- [Conclusion](#conclusion)

## Objectives

In this lab, you will:

- Understand how Kubernetes scheduling works
- Define readiness and liveness probes
- Use node and pod affinity to schedule pods

## Prerequisites

- Finish properly [Lab #02](./lab-02.md)
- Having your `minikube` cluster running

## Guide

### Step 1: Define readiness and liveness probe on StatefulSet

Readiness and liveness probes are essential to keep your application running properly. They are used to check if your application is ready to receive traffic and if it's still running.

On StatefulSet, we need to assure that SQL Server is running and can access the database needed by our application.

To do that you will use [sqlcmd](https://learn.microsoft.com/en-us/sql/tools/sqlcmd/sqlcmd-utility) to check if the database is ready and if the SQL Server is still running.

You need to add a new environment variable to your `echo-app-db.yaml` file and include the readiness and liveness probes.

First, let's add the environment variable to the `echo-app-db.yaml` file. Update the file with the following content on the property `spec.template.spec.containers.env`:

```yaml
- name: SQLCMDPASSWORD
  valueFrom:
  secretKeyRef:
    name: echo-api-db-secret
    key: dbpass
```

Pay attention to the indentation and the other properties on the file.

This environment variable will be used to pass the password to the `sqlcmd` command.

Now, let's add the readiness and liveness probes to the `echo-app-db.yaml` file. Update the file with the following content on the property `spec.template.spec.containers`:

```yaml
livenessProbe:
  exec:
    command:
    - sh
    - -c
    - |-
      /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q 'USE [echo-log]; SELECT * FROM EchoLogs'
  initialDelaySeconds: 60
  periodSeconds: 60
  failureThreshold: 3
  timeoutSeconds: 10
readinessProbe:
  exec:
    command:
    - sh
    - -c
    - |-
      /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q 'USE [echo-log]; SELECT * FROM EchoLogs'
  initialDelaySeconds: 30
  periodSeconds: 20
  failureThreshold: 3
  timeoutSeconds: 5
```

Pay attention to the indentation and the other properties on the file.

Take a look on the `initialDelaySeconds`, `periodSeconds`, `failureThreshold`, and `timeoutSeconds` properties. They are used to define when the probes will start, how often they will run, how many times they can fail, and how long they will wait for a response.

Let's update the StatefulSet with the new configuration. Run the following command:

```bash
kubectl apply -f echo-app-db.yaml
```

Now you can check the status of the pods and see how the probes are working. Run the following command:

```bash
kubectl get pods
```

If you want to see what happen when the probes fail, you can edit the `sqlcmd` command to use a wrong database name or a wrong query.

### Step 2: Define readiness and liveness probe on Echo API Deployment

This time you'll use a HTTP request to check if the application is ready to receive traffic and if it's still running.

First, you need to create a manifest file for Echo API Deployment.

Create a new file named `echo-api-dep.yaml` and add the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-api-dep
  namespace: echo-app-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo-app
      tier: back
  template:
    metadata:
      labels:
        app: echo-app
        tier: back
    spec:
      containers:
      - name: echo-api
        image: tasb/echo-api:k8s-v2
        ports:
        - containerPort: 80
        imagePullPolicy: Always
        env:
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: echo-api-db-secret
              key: connString
```

Now let's update with the readiness and liveness probes. Update the file with the following content on the property `spec.template.spec.containers`:

```yaml
readinessProbe:
  httpGet:
    path: /echo/ready
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 5
  failureThreshold: 1
livenessProbe:
  httpGet:
    path: /echo/live
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 5
  failureThreshold: 3
```

On this case, you are using the `httpGet` property to define the path and the port to be used on the probes.

Let's update the Deployment with the new configuration. Run the following command:

```bash
kubectl apply -f echo-api-dep.yaml
```

Now you should see again the status of the pods and see how the probes are working. Run the following command:

```bash
kubectl get pods
```

Again, if you want to see what happen when the probes fail, you can edit the probes to use a wrong path or a wrong port.

### Step 3: Use node affinity to schedule StatefulSet pods

Since our StatefulSet is using SQL Server, you want to assure that the pods are running on nodes with better disk performance.

To do that, you can use node affinity to schedule the pods on nodes with a specific label.

First, let's add a label to the node where you want to schedule the pods. Run the following command:

```bash
kubectl label nodes labs-m02 disk=ssd
```

You can check the label on the node running the following command:

```bash
kubectl get nodes --show-labels
```

If you want to check only the `disktype` label, you can run the following command:

```bash
kubectl get nodes -L disktype
```

Now you need to update the `echo-app-db.yaml` file to include the node affinity configuration.

Edit the `echo-app-db.yaml` file and add the following content on the property `spec.template.spec`:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - "ssd"
          - "nvme"
```

Pay attention to the indentation and the other properties on the file.

Before updating the StatefulSet, confirm in which node the pods are running. Run the following command:

```bash
kubectl get pods -o wide
```

Now you can update the StatefulSet with the new configuration. Run the following command:

```bash
kubectl apply -f echo-app-db.yaml
```

After the pods are running, you can check again where they are scheduled. Run the following command:

```bash
kubectl get pods -o wide
```

You can confirm that the pods are running on the node with the label `disk=ssd`.

### Step 4: Use pod affinity to schedule database backup

Since your backup job is running on a CronJob, you want to assure that the pods are running on nodes with the database pods.

To do that, you can use pod affinity to schedule the pods on nodes with a specific label.

Let's update the `echo-app-backup.yaml` file to include the pod affinity configuration.

You need to add the following content on the property `spec.jobTemplate.spec.template.spec`:

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
      matchExpressions:
      - key: app
        operator: In
        values:
          - echo-app
      - key: tier
        operator: In
        values:
          - db
      topologyKey: "kubernetes.io/hostname"
```

Pay attention to the indentation and the other properties on the file.

Now let's update the CronJob with the new configuration. Run the following command:

```bash
kubectl apply -f echo-app-backup.yaml
```

After the pods are running, you can check again where they are scheduled. Run the following command:

```bash
kubectl get pods -o wide
```

You may need to wait until the next minute to see the pods running.

## Conclusion

In this lab, you learned how to define readiness and liveness probes and how to use node and pod affinity to schedule pods.

You also learned how to use `kubectl` to check the status of the pods and how to update the manifests to include new configurations.
