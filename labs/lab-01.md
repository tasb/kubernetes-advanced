# Lab 01 - Review of Kubernetes Main Concepts

## Table of Contents

- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Guide](#guide)
  - [Step 1: Install Minikube and kubectl](#step-1-install-minikube-and-kubectl)
  - [Step 2: Create a Kubernetes cluster with 2 nodes using Minikube](#step-2-create-a-kubernetes-cluster-with-2-nodes-using-minikube)
  - [Step 3: Enable ingress and metrics server addons](#step-3-enable-ingress-and-metrics-server-addons)
  - [Step 4: Deploy a sample application to the cluster](#step-4-deploy-a-sample-application-to-the-cluster)
  - [Step 5: Test the application and prove that cluster is working](#step-5-test-the-application-and-prove-that-cluster-is-working)
- [Conclusion](#conclusion)

## Objectives

In this lab, you will:

- Install Minikube and kubectl
- Create a Kubernetes cluster with 2 nodes using Minikube
- Enable ingress and metrics server addons
- Deploy a simple application to the cluster
- Test the application and prove that cluster is working

## Prerequisites

- A Ubuntu 22.04 LTS machine with at least 4GB of RAM and 2 CPU cores
- Docker installed and running

## Guide

### Step 1: Install Minikube and kubectl

To install Minikube, run the following commands:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

To install kubectl, run the following commands:

```bash
curl -LO https://dl.k8s.io/release/v1.29.1/bin/linux/amd64/kubectl
sudo install kubectl /usr/local/bin/kubectl
```

Now, let's install kubectl autocompletion by running the following commands:

```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

Then, let's use a trick that helps to type `k` instead of `kubectl`:

```bash
echo "alias k=kubectl" >> ~/.bashrc
source ~/.bashrc
```

Finally, let's install `krew` to manage kubectl plugins:

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

Let's add krew to the PATH:

```bash
echo "export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"" >> ~/.bashrc
source ~/.bashrc
```

Install two useful kubectl plugins:

```bash
kubectl krew install ctx
kubectl krew install ns
```

### Step 2: Create a Kubernetes cluster with 2 nodes using Minikube

To create a Kubernetes cluster with 2 nodes using Minikube, run the following command:

```bash
minikube start -p labs --nodes 2
```

The `--nodes 2` flag tells Minikube to create a cluster with 2 nodes.

The `-p labs` flag tells Minikube to create a profile named `labs`.

Wait for the cluster to be created.

Then let's check the status of the cluster:

```bash
k get nodes
```

You should get and output similar to the following:

```bash
NAME       STATUS   ROLES           AGE     VERSION
labs       Ready    control-plane   3m8s    v1.28.3
labs-m02   Ready    <none>          2m46s   v1.28.3
```

Check that you are using `k` instead of `kubectl` because of the alias we created.

You can automatically use `kubectl` with the new cluster because Minikube sets the context for you.

Check the context using `kubectl ctx` plugin:

```bash
k ctx
```

### Step 3: Enable ingress and metrics server addons

To enable metrics server addon, run the following command:

```bash
minikube addons enable metrics-server -p labs
```

You should see get an output similar to the following:

```bash
💡  metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.6.4
🌟  The 'metrics-server' addon is enabled
```

To enable ingress addon, run the following command:

```bash
minikube addons enable ingress -p labs
```

You should see get an output similar to the following:

```bash
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.9.4
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

To check the status of the addons, run the following command:

```bash
minikube addons list -p labs
```

You should see a list of all addons available on minikube and their status.

To get only the enabled addons, run the following command:

```bash
minikube addons list -p labs | grep enabled
```

You should see get an output similar to the following:

```bash
| ingress                     | labs    | enabled ✅   | Kubernetes                     |
| metrics-server              | labs    | enabled ✅   | Kubernetes                     |
```

### Step 4: Deploy a sample application to the cluster

Now let's deploy a simple application to the cluster.

First, let's create a namespace for the application:

```bash
k create ns echo-app-ns
```

Then, let's deploy the application.

First, take a look at the application manifest file. Navigate to the following URL: [echo-app-full.yml](https://raw.githubusercontent.com/tasb/kubernetes-advanced/main/labs/lab01/echo-app-full.yml).

Check that the file have all the resources needed to deploy the application.

Then, deploy the application by running the following command:

```bash
k apply -f https://raw.githubusercontent.com/tasb/kubernetes-advanced/main/labs/lab01/echo-app-full.yml
```

You should see get an output similar to the following:

```bash
deployment.apps/echo-api-dep created
deployment.apps/echo-webapp-dep created
service/echo-api-svc created
service/echo-webapp-svc created
ingress.networking.k8s.io/echo-app-ingress created
```

Using the `kubectl ns` plugin, let's switch to the namespace of the application:

```bash
k ns echo-app-ns
```

Let's check the status of all resources in the namespace:

```bash
k get all
```

You should see get a list of deployments, services, and pods.

### Step 5: Test the application and prove that cluster is working

Since the application is configured to use an ingress, we need to find the IP address of the ingress controller.

To get the IP address of the ingress controller, run the following command:

```bash
k get ingress
```

You should see get an output similar to the following:

```bash
NAME       CLASS    HOSTS   ADDRESS        PORTS   AGE
echo-app   <none>   *       X.X.X.X       80      2m
```

Replace `X.X.X.X` with the IP address of the ingress controller.

Now let's update the `/etc/hosts` file to add the IP address of the ingress controller and the hostname of the application.

To do that, run the following command:

```bash
echo "X.X.X.X echo-app.ingress.test" | sudo tee -a /etc/hosts
```

Replace `X.X.X.X` with the IP address of the ingress controller.

Now let's test the application by running the following command:

```bash
curl http://echo-app.ingress.test
```

You should get an HTML response body,

Then to test the API endpoint, run the following command:

```bash
curl http://echo-app.ingress.test/api/echo/kubernetes
```

You should get `kubernetes` as the response body.

Let's test with another word:

```bash
curl http://echo-app.ingress.test/api/echo/advanced
```

Now you should get `advanced` as the response body.

This API have database connection to keep logs of all `echo` requests. To get a list of all logs, run the following command:

```bash
curl http://echo-app.ingress.test/api/logs
```

You should get a list of all logs on JSON format.

Depending on your environment, if you have a desktop environment, you can also open a web browser and navigate to `http://echo-app.ingress.test` to see the application.

## Conclusion

In this lab, you installed Minikube and kubectl, created a Kubernetes cluster with 2 nodes using Minikube, enabled ingress and metrics server addons, deployed a simple application to the cluster, and tested the application to prove that the cluster is working.
