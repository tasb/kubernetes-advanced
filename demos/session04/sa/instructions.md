# Service Account Demo

## Check default Service Account

- Check existing default user account

```bash
kubectl get sa
```

- Check your pods are using default user account

```bash
kubectl get pod deploy-demo-9d586d779-7jhrv -o yaml
```

## Check default volume and volumeMount

- Check your pods are using default volume and volumeMount

```bash
kubectl exec -it deploy-demo-9d586d779-7jhrv -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

- Check content of token

```bash
kubectl exec -it deploy-demo-9d586d779-7jhrv -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

- Check content of ca.crt

```bash
kubectl exec -it deploy-demo-9d586d779-7jhrv -- cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

## Create Service Account and set RBAC

- Create a ServiceAccount

```bash
kubectl apply -f accounts/public-sa.yaml
```

- Create deployment with a public image

```bash
kubectl apply -f deploys/public-image-deploy.yaml
```

- Check the service account of the pod

```bash
kubectl get pod sa-demo-9d586d779-7jhrv -o yaml
```

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
kubectl apply -f roles/cluster-role-binding-sa.yaml
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

## Configure Private Registry credentials

- Create new service account

```bash
kubectl apply -f accounts/private-sa.yaml
```

- Apply deployment with private registry

```bash
kubectl apply -f deploys/private-image-deploy.yaml
```

- Check that pod is not running due to image pull error

```bash
kubectl describe pod private-image-demo-7977cfddcb-9blx6
```

- Check where you get docker login

```bash
cat ~/.docker/config.json
```

- Update JSON file and create secret

```bash
kubectl create secret generic ghcr-io-theonorg --from-file=.dockerconfigjson=./secrets/dockerconfig.json --type=kubernetes.io/dockerconfigjson
```

- Update Service Account (uncomment manifest)

```bash
kubectl apply -f accounts/private-sa.yaml
```

- Restart deploymet

```bash
kubectl rollout restart deploy/private-image-demo
```

- Check that your deploy is running

```bash
kubectl get pods
```
