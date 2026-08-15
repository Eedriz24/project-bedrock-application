# Helm Deployment (Bonus 5.1)

Uses the upstream `retail-store-sample-app` chart with a custom values
file that swaps the in-cluster MySQL/PostgreSQL for RDS and the carts
store for DynamoDB.

## Install

```bash
helm repo add retail-store https://aws-containers.github.io/retail-store-sample-app/
helm repo update

kubectl apply -f secretProviderClass.yaml

helm upgrade --install retail-store retail-store/retail-store-sample-app \
  -n retail-app --create-namespace \
  -f values-rds-dynamodb.yaml
```

Requires the Secrets Store CSI Driver + AWS provider, and the AWS Load
Balancer Controller, to already be installed in the cluster.
