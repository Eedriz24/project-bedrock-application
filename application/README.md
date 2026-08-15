# Application - Project Bedrock

Everything that runs *inside* (or is invoked by) the platform Terraform
provisions: the retail-store-sample-app Kubernetes workload, its
Ingress and NetworkPolicies, and the source code for the
`bedrock-asset-processor` Lambda.

**This layer owns:** the `retail-app` namespace and its workloads, the
ALB Ingress, NetworkPolicies (bonus 5.4), the Helm values overriding the
data layer, and the Lambda function's Python source.

**This layer does NOT own:** any AWS resource creation (VPC, EKS, RDS,
IAM, S3 bucket, the Lambda *resource* itself) - that all lives in
`../infrastructure`. The Lambda source here is packaged and deployed by
Terraform's `s3-lambda` module; this folder is just where the code
lives and is edited/tested.

## Contents

```
application/
├── kubernetes/     # raw-manifest deploy path (namespace, ingress, network policies)
├── helm/           # Helm-based deploy path (bonus 5.1) - pick this OR kubernetes/
├── lambda/         # bedrock-asset-processor source (packaged by infrastructure/terraform)
└── .github/workflows/
    ├── app-validate.yml   # PR: lints manifests (kubeconform), Helm (helm lint/template), Lambda syntax
    └── app-deploy.yml     # main: deploys to EKS via Helm (or manifests) on merge
```

## Prerequisites

The infrastructure layer must already be applied — you need a running
EKS cluster, the RDS instances, the DynamoDB table, and the AWS Load
Balancer Controller / Secrets Store CSI Driver installed as cluster
add-ons before deploying the app.

```bash
aws eks update-kubeconfig --name project-bedrock-cluster --region us-east-1
```

## Deploy — pick one path

**Raw manifests:**
```bash
cd kubernetes
kubectl apply -f namespace.yaml
kubectl apply -f retail-store/
kubectl apply -f ingress.yaml
kubectl apply -f network-policies/
```

**Helm (bonus 5.1):**
```bash
cd helm/retail-store
kubectl apply -f secretProviderClass.yaml
helm upgrade --install retail-store retail-store/retail-store-sample-app \
  -n retail-app --create-namespace \
  -f values-rds-dynamodb.yaml
```

## Verify

```bash
kubectl get pods -n retail-app
kubectl get ingress -n retail-app -o wide
```

## Lambda code changes

Editing `lambda/bedrock_asset_processor/handler.py` requires a
`terraform apply` in `../infrastructure/terraform` to redeploy — the
function resource and its packaging live in the infrastructure layer,
even though the code is versioned here.

## CI/CD

Copy `.github/workflows/*.yml` into your repo's `.github/workflows/`
alongside the infrastructure layer's workflows. Both are path-scoped so
they only trigger on relevant changes:

- **`app-validate.yml`** — runs on every PR touching `application/**`.
  Lints raw manifests with `kubeconform`, lints/templates the Helm chart
  against the custom values file, and syntax-checks + smoke-tests the
  Lambda handler. Catches broken YAML/Helm config before merge.
- **`app-deploy.yml`** — runs on merge to `main` (and can be triggered
  manually via `workflow_dispatch`). Assumes the cluster and its
  add-ons (AWS Load Balancer Controller, Secrets Store CSI Driver)
  already exist from the infrastructure layer. Auto-detects whether
  the Helm values file is present and deploys via Helm; otherwise falls
  back to `kubectl apply` on the raw manifests. Verifies the rollout by
  listing pods and the Ingress at the end.

### Required repository secrets

Same OIDC role as the infrastructure workflows
(`AWS_OIDC_ROLE_ARN`), granted `eks:DescribeCluster` and enough
Kubernetes RBAC (via an EKS Access Entry, e.g. `AmazonEKSAdminPolicy` or
a scoped custom policy for the `retail-app` namespace) to apply
manifests/Helm releases — broader than `bedrock-dev-view`'s view-only
scope, since this identity needs to deploy, not just read.
