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
openssl genrsa -out user01.key 2048
openssl req -new -key user01.key -out user01.csr -subj "/CN=user01/O=developers"
```

- Update `csr.yaml` file to update request

```bash
cat user01.csr | base64 | tr -d "\n"
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
kubectl certificate approve user-request-user01
```

- Check CSR status

```bash
kubectl get csr
```

- Export certificate

```bash
kubectl get csr user-request-user01 -o jsonpath='{.status.certificate}'| base64 -d > user01.crt
```

- Add user config to kubeconfig

```bash
kubectl config set-credentials dev01 --client-key=./user01.key --client-certificate=./user01.crt --embed-certs=true

kubectl config set-context user01 --cluster=labs --user=user01
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
kubectl ctx dev01

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
