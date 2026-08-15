# Lambda - bedrock-asset-processor

Simple S3-triggered function. Packaged and deployed by Terraform
(`terraform/modules/s3-lambda`), which zips this directory and creates
the function, execution role, and S3 event notification.

## Local test

```bash
python3 -c "
from bedrock_asset_processor.handler import handler
handler({'Records':[{'s3':{'object':{'key':'test.jpg'}}}]}, None)
"
```

## Manual verification after deploy

```bash
aws s3 cp test.jpg s3://bedrock-assets-<student-id>/ --profile bedrock-dev-view
aws logs tail /aws/lambda/bedrock-asset-processor --follow
```
