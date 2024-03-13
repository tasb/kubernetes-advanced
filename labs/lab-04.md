# Secure your pods connections

On this lab you'll add network policies to your Echo App solution to only allow needed connections from all your pods.

## On this lab

- [Prepare your machine](#prepare-your-machine)
- [Create testing pods](#create-testing-pods)
- [Deny all communication using Network Policies](#deny-all-communication-using-network-policies)
- [Open communication between API and database](#open-communication-between-api-and-database)
- [Open communication between Webapp and API](#open-communication-between-webapp-and-api)
- [Open communication from ingress controller](#open-communication-from-ingress-controller)

## Prepare your machine

To complete this lab you need to have a different network plugin enabled on your minikube cluster.

You'll use [Calico](https://www.tigera.io/project-calico/) plugin that is one of the most common network plugins used with Kubernetes.

This plugin will implement all your network configurations and additionally have the implementation of network policies.

If you have your minikube cluster running, you need to stop it.

```bash
minikube stop -p labs
```

To start the cluster and enable Calico plugin, you need to execute the following command.

```bash
minikube start --nodes 2 -p labs --network-plugin=cni --cni=calico
```

To check if Calico is running properly, execute the following command.

```bash
kubectl get pods -l k8s-app=calico-node -A
```

And you should get an output similar with this, stating `1/1` on `READY` column.

```bash
NAMESPACE     NAME                READY   STATUS    RESTARTS   AGE
kube-system   calico-node-qhmdv   1/1     Running   0          118s
```

Let's start to create network policies.

## Create testing pods

A best practice about network policies says you should always start by denying all traffic and then open when needed.

First let's create a new namespace and a pod on that namespace to make some tests.

```bash
kubectl create ns test-ns 
```

Now let's create a pod using `nginx` image.

```bash
kubectl run test-pod --image=nginx -n test-ns
```

Check when your pod is ready and then let's run a `curl` command to make a request to Echo API service on namespace `echo-app-ns`.

```bash
kubectl exec -it test-pod -n test-ns -- curl -w '\n' http://echo-api-svc.echo-app-ns.svc.cluster.local:8080/hostname
```

You should get a reply like this.

```bash
"echo-api-dep-7d7cf89799-474d7"
```

Let's create a pod inside `echo-app-ns`, to make the same test.

```bash
kubectl run test-pod --image=nginx -n echo-app-ns
```

Execute the `curl` command directly to check you get a reply from the Echo API Service.

```bash
kubectl exec -it test-pod -n echo-app-ns -- curl -w '\n' http://echo-api-svc.echo-app-ns.svc.cluster.local:8080/hostname
```

On this request, you should get a reply like this.

```bash
"echo-api-dep-7d7cf89799-474d7"
```

## Deny all communication using Network Policies

Now, let's add a network policy to deny this communication.

Save the following manifest to `deny-all.yaml`:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
  namespace: echo-app-ns
spec:
  podSelector: {}
  ingress: []
```

And apply to the cluster.

```bash
kubectl apply -f deny-all.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns default-deny-all
```

Now let's go back to the testing pods to check if you can reach the service on other namespace.

```bash
kubectl exec -it test-pod -n test-ns -- curl -w '\n' http://echo-api-svc.echo-app-ns.svc.cluster.local:8080/hostname
```

Now you should not get any reply and stay blocked when running the `curl` command.

To proceed, you need to press `Ctrl+C` to break the request.

Let's try on `echo-app-ns` namespace too.

```bash
kubectl exec -it test-pod -n echo-app-ns -- curl -w '\n' http://echo-api-svc.echo-app-ns.svc.cluster.local:8080/hostname
```

And you should not get any reply either.

## Open communication between API and database

To make your solution to work, you need to allow communication between components.

You'll start by opening communication between Echo API and Echo database.

Create a manifest file named `allow-api-to-database.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-api-to-database
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: back
  egress: 
  - to:
    - podSelector:
        matchLabels:
          app: echo-app
          tier: db
    ports:
      - port: 1433
        protocol: TCP
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
      - port: 53
        protocol: UDP
```

On this manifest you specify a network policy that will be applied to pods that match these labels:

```yaml
podSelector:
  matchLabels:
    app: echo-app
    tier: back
```

And for these pods you define an outbound rule to allow communication on port `1433` for pods with following labels:

```yaml
podSelector:
  matchLabels:
    app: echo-app
    tier: db
```

Let's apply this network policy to the cluster.

```bash
kubectl apply -f allow-api-to-database.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-api-to-database
```

You opened the outbound (egress) connectivity from API pods to Database pods but you need to open inbound (ingress) connectivity.

Create a manifest file named `allow-database-from-api.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-database-from-api
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: db
  ingress: 
  - from:
    - podSelector:
        matchLabels:
          app: echo-app
          tier: back
    ports:
      - port: 1433
        protocol: TCP
```

Let's apply this network policy to the cluster.

```bash
kubectl apply -f allow-database-from-api.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-database-from-api
```

## Open communication between Webapp and API

Like you've done for database, you need to open ingress connectivity on API.

Create a manifest file named `allow-api-from-webapp.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-api-from-webapp
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: back
  ingress: 
  - from:
    - podSelector:
        matchLabels:
          app: echo-app
          tier: front
    ports:
      - port: 80
        protocol: TCP
```

Let's apply this network policy to the cluster.

```bash
kubectl apply -f allow-api-from-webapp.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-api-from-webapp
```

Then you need to enable outbound connectivity on Webapp to allow it to reach API.

Create a manifest file named `allow-webapp-to-api.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-webapp-to-api
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: front
  egress: 
  - to:
    - podSelector:
        matchLabels:
          app: echo-app
          tier: back
    ports:
      - port: 80
        protocol: TCP
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
      - port: 53
        protocol: UDP
```

Let's apply this network policy to the cluster.

```bash
kubectl apply -f allow-webapp-to-api.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-webapp-to-api
```

Now you can try to execute the application again on your browser. But, what you'll get is a HTTP 502 Bad Gateway error.

This happened because you need to open the communication from ingress controller pods to your pods.

## Open communication from ingress controller

Create a manifest file named `allow-ingress-to-webapp.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-ingress-to-webapp
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: front
  ingress: 
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
      podSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
    ports:
      - port: 80
        protocol: TCP
```

Then apply to your cluster.

```bash
kubectl apply -f allow-ingress-to-webapp.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-ingress-to-webapp
```

Check this manifest and see that your are open communication for your pods with following labels:

```yaml
podSelector:
  matchLabels:
    app: echo-app
    tier: front
```

And allowing to receive requests, since you're defining an `ingress` rule, from pods on namespace `ingress-nginx` with label `app.kubernetes.io/name=ingress-nginx`.

Finally, let's do the same for API pods.

Create a manifest file named `allow-ingress-to-api.yaml` with following content.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-ingress-to-api
  namespace: echo-app-ns
spec:
  podSelector:
    matchLabels:
      app: echo-app
      tier: back
  ingress: 
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
      podSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
    ports:
      - port: 80
        protocol: TCP
```

Then apply to your cluster.

```bash
kubectl apply -f allow-ingress-to-api.yaml
```

You can check the details of you network policy with the following command.

```bash
kubectl describe netpol -n echo-app-ns allow-ingress-to-api
```

Now you can test your application using your browser and navigate to <http://echo-app.ingress.test>.

Congratulations! You have set network policies on your Echo App and make it more secure for not expected requests.
