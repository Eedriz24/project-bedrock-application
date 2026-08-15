# Retail Store Manifests

Place the upstream `retail-store-sample-app` Kubernetes manifests here
(https://github.com/aws-containers/retail-store-sample-app), split per
service (ui, catalog, orders, carts, checkout, rabbitmq, redis).

Before applying, patch the `catalog` and `orders` Deployments so their DB
env vars are mounted from the Secrets Store CSI Driver (pointing at the
`bedrock/catalog-mysql` and `bedrock/orders-postgres` Secrets Manager
entries) instead of the chart's default in-cluster DB service names.
Patch `carts` to use the DynamoDB table output from
`terraform/modules/dynamodb` via IRSA.

Apply order:
```bash
kubectl apply -f ../namespace.yaml
kubectl apply -f .
kubectl apply -f ../ingress.yaml
```
