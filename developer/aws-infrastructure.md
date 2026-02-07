# AWS Infrastructure

## Overview

DevZone uses AWS SAM (Serverless Application Model) for infrastructure-as-code. All resources are defined in `aws/template.yaml`.

## Architecture

```
                    Internet
                       │
              ┌────────▼────────┐
              │  API Gateway     │
              │  (HTTP API)      │
              │  /api/* routes   │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │  Lambda Function │
              │  Node.js 20.x   │
              │  256MB / 30s     │
              └──┬──────┬──────┬┘
                 │      │      │
        ┌────────▼┐  ┌──▼──┐  ┌▼──────────┐
        │DynamoDB  │  │ S3  │  │CloudFront  │
        │(2 tables)│  │     │  │(invalidate)│
        └──────────┘  └──┬──┘  └────────────┘
                         │
              ┌──────────▼──────────┐
              │ S3: Game Data       │
              │ live/, drafts/,     │
              │ versions/, images/  │
              └─────────────────────┘
```

## Resources

### S3 Buckets

| Bucket | Purpose |
|--------|---------|
| `tarot-battlegrounds-data-{env}` | Game data JSON + card images |
| `tarot-devzone-frontend-{env}` | DevZone React app static files |

### DynamoDB Tables

| Table | PK | Billing | Purpose |
|-------|-----|---------|---------|
| `devzone-users-{env}` | userId | On-demand | User accounts |
| `devzone-versions-{env}` | versionId | On-demand | Version metadata |

### Lambda Function

| Setting | Value |
|---------|-------|
| Name | `tarot-devzone-api-{env}` |
| Runtime | Node.js 20.x |
| Memory | 256 MB |
| Timeout | 30 seconds |
| Handler | `dist/lambda.handler` |

**IAM Policies:**
- S3: Read/write to data bucket
- DynamoDB: CRUD on both tables
- CloudFront: CreateInvalidation

### API Gateway

- Type: HTTP API (cheaper than REST API)
- Routes: `ANY /api/{proxy+}` → Lambda
- CORS: Configured for DevZone frontend origin

## SAM Template

The `aws/template.yaml` defines all resources with parameterized environment names:

```yaml
Parameters:
  Environment:
    Type: String
    Default: prod
  JwtSecret:
    Type: String
    NoEcho: true
  GameDistributionId:
    Type: String
    Description: CloudFront distribution ID for game data
    Default: none
```

### Lambda Build Method

The Lambda function uses `BuildMethod: makefile` with `CodeUri: ../` (repo root) to handle npm workspace dependencies. SAM's default Node.js builder can't resolve workspace packages, so a custom `Makefile` in the repo root handles packaging:

```makefile
build-DevZoneApi:
    cp -r server/dist $(ARTIFACTS_DIR)/dist
    sed '/@tarot-devzone\/shared/d' server/package.json > $(ARTIFACTS_DIR)/package.json
    cd $(ARTIFACTS_DIR) && npm install --omit=dev
    mkdir -p $(ARTIFACTS_DIR)/node_modules/@tarot-devzone/shared/dist
    cp shared/package.json $(ARTIFACTS_DIR)/node_modules/@tarot-devzone/shared/
    cp -r shared/dist/* $(ARTIFACTS_DIR)/node_modules/@tarot-devzone/shared/dist/
```

### Frontend Bucket Policy

The frontend S3 bucket has public access enabled for static website hosting:

```yaml
PublicAccessBlockConfiguration:
  BlockPublicAcls: false
  BlockPublicPolicy: false
  IgnorePublicAcls: false
  RestrictPublicBuckets: false
```

With a bucket policy allowing `s3:GetObject` for all principals.

## Deploy Scripts

### Full Deployment

```bash
cd aws
./deploy-devzone.sh prod
```

This runs:
1. Build shared types (`cd shared && npm run build`)
2. Build server (`cd server && npm run build`)
3. `sam build && sam deploy` — creates/updates all AWS resources
4. Gets API URL from CloudFormation outputs
5. Build client with `VITE_API_URL="${API_URL}/api" npm run build`
6. `aws s3 sync client/dist/ s3://tarot-devzone-frontend-{env}/`

## Cost Estimate

| Service | Estimated Cost |
|---------|---------------|
| Lambda | Free tier (1M requests/mo) |
| API Gateway | Free tier (1M requests/mo) |
| DynamoDB | Free tier (25 RCU/WCU) |
| S3 (data) | ~$0.05/month |
| S3 (frontend) | ~$0.05/month |
| CloudFront | Free tier (1TB/mo) |
| **Total** | **~$0.10/month** |

## CORS Configuration

The game data CloudFront distribution must allow requests from the game's origin:

```
Access-Control-Allow-Origin: https://dui22oafwco41.cloudfront.net
```

The DevZone API Gateway must allow requests from the DevZone frontend:

```
Access-Control-Allow-Origin: https://{devzone-frontend-cloudfront}.cloudfront.net
```

## Related Documentation

- [Deployment](deployment.md) — step-by-step deploy guide
- [Environment Variables](../resources/environment-vars.md) — all config vars
- [Architecture](architecture.md) — system overview
