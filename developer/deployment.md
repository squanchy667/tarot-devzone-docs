# Deployment

## Prerequisites

- AWS CLI configured with admin credentials
- AWS SAM CLI installed (`brew install aws-sam-cli`)
- Node.js 20+
- The game's CloudFront distribution ID (for cache invalidation)

## First-Time Deployment

### 1. Create AWS Infrastructure

```bash
cd tarot-devzone/aws
./setup-infra.sh
```

This uses SAM to create:
- S3 bucket for game data
- S3 bucket for DevZone frontend
- DynamoDB tables (users, versions)
- Lambda function with API Gateway
- IAM roles and policies

You'll be prompted for:
- **Environment name** (e.g., `dev`, `prod`)
- **JWT secret** — random string for signing tokens
- **CloudFront distribution ID** — from the game's existing CloudFront

### 2. Seed the Admin User

```bash
cd tarot-devzone
npx tsx scripts/seed-db.ts
```

Edit the script first to set your desired admin email/password.

### 3. Export Initial Game Data

```bash
npx tsx scripts/export-game-data.ts
```

This generates `cards.json`, `synergies.json`, `config.json`, `theme.json` from the game's CardDatabase and uploads them to S3.

### 4. Deploy Frontend

```bash
cd aws
./deploy-devzone.sh
```

This builds all three workspaces and deploys:
- Server → Lambda (via SAM)
- Client → S3 (static site)

## Redeployment

After making code changes:

```bash
cd tarot-devzone/aws
./deploy-devzone.sh
```

This rebuilds everything and redeploys.

## Deployment Checklist

- [ ] AWS CLI configured (`aws sts get-caller-identity`)
- [ ] SAM CLI installed (`sam --version`)
- [ ] Environment variables set in SAM template
- [ ] Admin user seeded in DynamoDB
- [ ] Initial game data exported to S3
- [ ] Frontend deployed to S3
- [ ] API Gateway URL noted for client config
- [ ] CORS configured on data CloudFront
- [ ] Test login at DevZone URL
- [ ] Test card edit → publish → game loads new data

## Environment Configuration

### Local Development
```
API: http://localhost:3001
Client: http://localhost:5173 (proxies /api to :3001)
```

### Deployed
```
API: https://{api-id}.execute-api.{region}.amazonaws.com/api
Client: https://{frontend-distribution}.cloudfront.net
```

## Monitoring

- **Lambda logs**: CloudWatch Logs (`/aws/lambda/tarot-devzone-api-{env}`)
- **API Gateway**: CloudWatch metrics
- **S3**: Access logs (if enabled)
- **DynamoDB**: CloudWatch metrics for read/write capacity

## Rollback

If a deployment breaks something:

1. **Lambda**: SAM keeps previous versions; redeploy from last known good commit
2. **Game data**: Use DevZone's version manager to rollback to a previous version
3. **Frontend**: Redeploy previous build to S3

## Related Documentation

- [AWS Infrastructure](aws-infrastructure.md) — resource details
- [Setup Guide](setup-guide.md) — local development
- [Environment Variables](../resources/environment-vars.md) — all config
