# Kubernetes Manifests - Project Bedrock

Raw-manifest deployment path for the retail-store-sample-app (alternative
to the Helm chart in `../helm`).

## Apply

```bash
aws eks update-kubeconfig --name project-bedrock-cluster --region us-east-1
kubectl apply -f namespace.yaml
kubectl apply -f retail-store/
kubectl apply -f ingress.yaml
kubectl apply -f network-policies/    # bonus 5.4
```

## Verify

```bash
kubectl get pods -n retail-app
kubectl get ingress -n retail-app
```
