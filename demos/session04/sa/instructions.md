# Service Account Demo

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
