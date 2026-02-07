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
    Default: dev
  JwtSecret:
    Type: String
    NoEcho: true
  CloudFrontDistributionId:
    Type: String
```

## Deploy Scripts

### First-Time Setup

```bash
cd aws
./setup-infra.sh
```

This runs:
1. `sam build` — packages Lambda code
2. `sam deploy --guided` — creates all AWS resources
3. Prompts for JWT secret and CloudFront distribution ID

### Full Deployment

```bash
cd aws
./deploy-devzone.sh
```

This runs:
1. Build shared types (`npm run build --workspace=shared`)
2. Build server (`npm run build --workspace=server`)
3. Build client (`npm run build --workspace=client`)
4. `sam build && sam deploy`
5. `aws s3 sync client/dist/ s3://tarot-devzone-frontend-{env}/`

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
