# RBAC Demo

## List available resources to use on rules

```bash
kubectl api-resources –o wide
```

## Create Role and ClusterRole

```bash
kubectl apply -f roles/
```

## Get objects details

- Get role details

```bash
kubectl describe role pod-reader -n rbac-ns
```

- List cluster roles

```bash
kubectl get clusterrole
```

- Get your ClusterRole details

```bash
kubectl describe clusterrole deployments-reader
```

- Get `view` ClusterRole details

```bash
kubectl describe clusterrole view
```

- Get `system:discovery` ClusterRole details

```bash
kubectl get clusterroles system:discovery -o yaml
```

## Prepare cluster with resources

```bash
kubectl apply -f ./workloads
```

## Create a new user and set RBAC

- Generate private key and certificate signing request

```bash
openssl genrsa -out dev01.key 2048
openssl req -new -key dev01.key -out dev01.csr -subj "/CN=dev01/O=developers"
```

- Update `csr.yaml` file to update request

```bash
cat dev01.csr | base64 | tr -d "\n"
```

- Create CSR

```bash
kubectl apply -f csr.yaml
```

- Get CSR list

```bash
kubectl get csr
```

- Approve the CSR

```bash
kubectl certificate approve user-request-dev01
```

- Check CSR status

```bash
kubectl get csr
```

- Export certificate

```bash
kubectl get csr user-request-dev01 -o jsonpath='{.status.certificate}'| base64 -d > dev01.crt
```

- Add user config to kubeconfig

```bash
kubectl config set-credentials dev01 --client-key=~/k8s-keys/dev01.key --client-certificate=~/k8s-keys/dev01.crt --embed-certs=true

kubectl config set-context dev01 --cluster=microk8s-cluster --user=dev01
```

## Run kubectl commands with new user

- Change context

```bash
kubectl ctx dev01

kubectl get nodes

kubectl get pods
```

- Check that all operations are forbidden. Let's create role bindings

```bash
kubectl ctx -

kubectl apply -f role-bindings/

kubectl get clusterrolebindings

kubectl get rolebinding
```

- Move to dev01 context

```bash
kubectl ctx dev1

kubectl get pods

kubectl get pods -A

kubectl get deploy

kubectl get deploy -n rbac-ns

kubectl get ds -A

kubectl delete pod deploy-demo-XXX -n rbac-ns

kubectl delete deploy deploy-demo -n rbac-ns
```

## Check permissions

```bash
kubectl auth can-i delete pod

kubectl auth can-i delete pod -n rbac-ns

kubectl auth can-i delete deploy

kubectl auth can-i delete deploy -n rbac-ns

kubectl auth can-i delete deploy --as admin

kubectl ctx -

kubectl auth can-i delete deploy --as dev01

kubectl auth can-i delete deploy --as ops01
```

## Create Service Account and set RBAC

- Check your pods are using default user account

```bash
kubectl get sa

kubectl describe pod deploy-demo-XXX 
```

- Explain volume and volumeMount

- Create a ServiceAccount

```bash
kubectl apply -f sa/sa.yaml
```

- Create deployment with a private image

```bash
kubectl apply -f sa/private-image-deploy.yaml
```

- Check we have an error on image pull

```bash
kubectl describe pod sa-demo-XXXXX
```

## Configure Private Registry credentials

- Create secret with ghcr.io login

```bash
kubectl apply -f ghcr-token.yaml
```

- Update Service Account (uncomment manifest)

```bash
kubectl apply -f sa/sa.yaml
```

- Restart deploymet

```bash
kubectl rollout restart deploy/sa-demo
```

- Check that your deploy ir running

## Use Kubernetes API on your container with SA

- Run bash on your pod

```bash
kubectl exec -it sa-demo-XXXX -- bash
```

- Run script to update kubeconfig

```bash
./kubectl-config.sh
```

- Try to list your pods

```bash
kubectl get pods
```

- You should get a forbidden error

- Create a role binding with service account

```bash
kubectl apply -f sa/cluster-role-binding-sa.yaml
```

- Run you kubectl command again

```bash
kubectl get pods

kubectl get pods -A
```

- Finally check that any other command is forbidden

```bash
kubectl get deploys
```
