# Lab 05: Create your Helm Charts

On this lab you'll create Helm Charts for Echo App and make it simpler to manage Kubernetes manifests.

## Objectives

In this lab, you will:

- Understand how Helm Charts work
- Create Helm Charts for Echo App
- Install Echo App using Helm Charts
- Test Echo App using Helm Charts

## Pre-requisites

First, enable `minikube` cluster on your machine.

Check if you already have Echo App running and if so, let's delete it since we'll recreate everything from the scratch using Helm Charts.

Check if you have `echo-app-ns` namespace on your cluster.

```bash
kubectl get ns
```

If you see `echo-app-ns` on returning list, you should delete it. If not, you may skip this step.

To delete a namespace, you run the following command.

```bash
kubectl delete ns echo-app-ns
```

## Step 01: Install Helm

You may follow the installation steps describe on [Helm Website](https://helm.sh/docs/intro/install/) that have several options, depending of the operating system you're using.

After installation finish with success, you try the following command to check if Helm is working properly.

```bash
helm version
```

## Step 02: Getting Started

Before you start creating your Helm Charts, you need to get the manifest files for the Echo App.

Please use the ones you produced on previous labs or you can download a version from [here](https://github.com/tasb/k8s-advanced-labs/archive/refs/tags/lab04.zip)

In this zip file you'll find all the manifest files you need to create your Helm Charts.

## Step 03: Create Helm Charts

Now, you can start creating your Helm Charts. For this lab, the option is to create a single Helm Chart for all components of Echo App.

First, you need to create a new chart.

```bash
helm create echo-app
```

This command will create a new folder named `echo-app` with all the necessary files to start creating your Helm Chart.

First step is to clean up the folder and remove all files and folders that you won't use.

You can remove all auto generated yaml files and the `templates` folder (or move them to another folder to serve as reference).

Then, you can create 6 new folders inside `echo-app` folder, one for each component of Echo App:

- `echo-app-db`
- `echo-app-db-backup`
- `echo-app-api`
- `echo-app-web`
- `echo-app-common`
- `echo-app-network-policies`

## Step 04: Prepare the Helm Chart

To guide you on Helm Chart creation, you may use the following content as you `values.yaml` file.

```yaml
## Values file for Echo App Chart
namespace: echo-app-ns

echoDb:
  name: echo-db-sts
  image: mcr.microsoft.com/mssql/server:2017-latest
  dbUser: SA
  dbPass: password
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

echoDbBackup:
  name: echo-app-db-backup
  schedule: "* * * * *"
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

echoApi:
  name: echo-api
  image: tasb/echo-api:k8s-v2
  service:
    port: 8080
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

echoWebapp:
  name: echo-webapp
  image: tasb/echo-webapp:k8s-v2
  service:
    port: 9000

ingress:
  enabled: true
  url: echo-app.ingress.test
  ingressClassName: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
```

Additionally, you may use the folowing content on your `_helpers.tpl` file inside the `templates` folder.

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "echo-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "echo-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "echo-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}


{{/*
Selector labels
*/}}
{{- define "echo-app.webapp.labels" -}}
app: {{ .Release.Name }}
tier: front
{{- end }}

{{- define "echo-app.api.labels" -}}
app: {{ .Release.Name }}
tier: back
{{- end }}

{{- define "echo-app.db.labels" -}}
app: {{ .Release.Name }}
tier: db
{{- end }}

{{- define "echo-app.dbBackup.labels" -}}
app: {{ .Release.Name }}
tier: backup
{{- end }}
```

On this file you may find named templates for:

- Getting chart name
- Several labels to be used on your Kubernetes resources

Finally, take a look on your `NOTES.txt` file and check if you need to update it.

## Step 05: Authoring the Helm Chart

Now you can start to create your templates for each manifest for each component.

Take the initial files and start to replace the static values with the dynamic ones.

During this process you may use the following command to validate your Helm Chart.

```bash
helm lint echo-app
```

If you want to generate the manifest files to check if everything is working properly, you can use the following command.

```bash
helm template --debug echo-app
```

You can either use this command to make some validations using your cluster.

```bash
helm install --dry-run --debug --generate-name echo-app
```

## Step 06: Install Echo App using Helm Charts

After you have created your Helm Chart, you can install it on your cluster.

```bash
helm install echo-app echo-app/ -n echo-app-ns --create-namespace --values values.yaml --set "echoDb.dbPass=P@ssw0rd"
```

On this command, you create a new Helm Release named `echo-app` using the helm chart defined on folder `echo-app`.

After you run successfully this command, you should see the contents of `NOTES.txt` file with template blocks replace with values used for these release.

## Step 07: Helm release lifecycle

After you run the previous command, you can check if your release was created successfully.

```bash
helm list -n echo-app-ns
```

And get an output similar with this. Pay attention on the column `STATUS` where you should have a `deployed` value.

```bash
NAME         NAMESPACE   REVISION UPDATED                                 STATUS   CHART              APP VERSION
echo-app     echo-app-ns 1        2023-03-11 00:01:21.940402012 +0000 UTC deployed echo-app-0.5.0  0.1.0
```

Now make a check on your cluster to see if all resources were created successfully.

```bash
kubectl get all -n echo-app-ns
```

Then, change the value of `ingress.url` on your `values.yaml` file and run the following command to upgrade your Helm Release.

```bash
helm upgrade echo-app echo-app/ -n echo-app-ns --values values.yaml
```

After the upgrade, you can check if your release was updated successfully.

```bash
helm list -n echo-app-ns
```

And get an output similar with this. Pay attention on the column `STATUS` where you should have a `deployed` value.

```bash
NAME         NAMESPACE   REVISION UPDATED                                 STATUS   CHART              APP VERSION
echo-app     echo-app-ns 2        2023-03-12 00:06:25.986231865 +0000 UTC deployed echo-app-1.0.0  1.0.0
```

## Step 08: Clean up

After you have finished this lab, you can uninstall your Helm Release.

```bash
helm uninstall echo-app -n echo-app-ns
```

And then, you can delete the namespace you created.

```bash
kubectl delete ns echo-app-ns
```

## Conclusion

On this lab, you have created a Helm Chart for Echo App and installed it on your cluster.

You have learned how to create a Helm Chart, how to install it and how to upgrade it.

You have also learned how to clean up your environment after you have finished your work.
