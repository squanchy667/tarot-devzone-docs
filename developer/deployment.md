# Deployment

## Prerequisites

- AWS CLI configured with admin credentials
- AWS SAM CLI installed (`brew install aws-sam-cli`)
- Node.js 20+
- The game's CloudFront distribution ID (for cache invalidation)

## First-Time Deployment

### 1. Build All Packages

```bash
cd tarot-devzone
npm install          # installs all workspace deps
npm run build -w shared
npm run build -w server
```

### 2. Deploy Backend with SAM

```bash
cd aws
sam build --template template.yaml
sam deploy \
  --stack-name "tarot-devzone-prod" \
  --resolve-s3 \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides "Environment=prod JwtSecret=<your-secret>" \
  --no-confirm-changeset
```

SAM creates: S3 buckets, DynamoDB tables, Lambda function, API Gateway, IAM roles.

### 3. Build and Deploy Frontend

After backend is deployed, get the API URL from CloudFormation outputs:

```bash
API_URL=$(aws cloudformation describe-stacks \
  --stack-name "tarot-devzone-prod" \
  --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' \
  --output text)

cd ../client
VITE_API_URL="${API_URL}/api" npm run build

FRONTEND_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name "tarot-devzone-prod" \
  --query 'Stacks[0].Outputs[?OutputKey==`DevZoneFrontendBucket`].OutputValue' \
  --output text)

aws s3 sync dist/ "s3://$FRONTEND_BUCKET" --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --exclude "index.html"
aws s3 cp dist/index.html "s3://$FRONTEND_BUCKET/index.html" \
  --cache-control "public, max-age=0, must-revalidate"
```

### 4. Seed the Admin User

```bash
cd tarot-devzone
npx tsx scripts/seed-db.ts
```

Edit the script first to set your desired admin email/password.

### 5. Export Initial Game Data

```bash
npx tsx scripts/export-game-data.ts
```

This generates `cards.json`, `synergies.json`, `config.json`, `theme.json` from the Unity CardDatabase and uploads them to both `live/` and `drafts/` prefixes on S3.

### 6. Automated Full Deploy

All of the above is automated in a single script:

```bash
cd aws
./deploy-devzone.sh prod
```

The script: builds shared + server, runs `sam build && sam deploy`, gets the API URL from CloudFormation, builds the client with `VITE_API_URL`, then syncs to S3.

## Redeployment

After making code changes:

```bash
cd tarot-devzone/aws
./deploy-devzone.sh prod
```

This rebuilds everything and redeploys.

## SAM Build: Workspace Package Handling

The project uses npm workspaces with a `@tarot-devzone/shared` package. SAM's default Node.js builder runs `npm install` in isolation, which fails because the shared package isn't in the npm registry.

**Solution**: The SAM template uses `BuildMethod: makefile` with `CodeUri: ../` (repo root). The root `Makefile` handles packaging:

1. Copies pre-built `server/dist/` to the Lambda artifacts directory
2. Strips the `@tarot-devzone/shared` workspace dependency from `package.json` using `sed`
3. Runs `npm install --omit=dev` in the artifacts directory (isolated from workspace)
4. Manually copies `shared/dist/` into `node_modules/@tarot-devzone/shared/`

This ensures the Lambda deployment package has all dependencies without relying on npm workspace resolution.

## Deployment Checklist

- [x] AWS CLI configured (`aws sts get-caller-identity`)
- [x] SAM CLI installed (`sam --version`)
- [x] Environment variables set in SAM template
- [x] Admin user seeded in DynamoDB
- [x] Initial game data exported to S3 (30 cards, 4 synergies, config, theme)
- [x] Frontend deployed to S3 with public access policy
- [x] API Gateway URL injected into client build via VITE_API_URL
- [x] CORS configured (origin reflection mode for credentials support)
- [x] Test login at DevZone URL
- [ ] Test card edit -> publish -> game loads new data (Unity loader not yet integrated)

## Environment Configuration

### Local Development
```
API: http://localhost:3001
Client: http://localhost:5173 (proxies /api to :3001)
```

### Deployed
```
API: https://{api-id}.execute-api.{region}.amazonaws.com/api
Frontend: http://tarot-devzone-frontend-{env}.s3-website-{region}.amazonaws.com
```

The frontend is served via S3 static website hosting. CloudFront can optionally be added in front for HTTPS and caching.

## Monitoring

- **Lambda logs**: CloudWatch Logs (`/aws/lambda/tarot-devzone-api-{env}`)
- **API Gateway**: CloudWatch metrics
- **S3**: Access logs (if enabled)
- **DynamoDB**: CloudWatch metrics for read/write capacity

## Troubleshooting

### SAM build fails with "not in this registry"
The `@tarot-devzone/shared` package is a workspace package, not published to npm. Ensure you're using the Makefile build method (see "Workspace Package Handling" above).

### CORS errors in browser
The server uses `origin: true` (reflects the request origin) with `credentials: true`. If you see CORS errors, check that the Lambda function has the latest code deployed.

### Lambda cold starts
The first request after deployment may take 3-5 seconds (cold start). Subsequent requests are fast. If you get intermittent 401 errors right after deploy, wait a moment and retry.

### Frontend shows blank page
Ensure `VITE_API_URL` was set correctly during the client build. Check the browser's Network tab — API calls should go to the API Gateway URL, not a relative `/api` path.

## Rollback

If a deployment breaks something:

1. **Lambda**: SAM keeps previous versions; redeploy from last known good commit
2. **Game data**: Use DevZone's version manager to rollback to a previous version
3. **Frontend**: Redeploy previous build to S3

## Related Documentation

- [AWS Infrastructure](aws-infrastructure.md) — resource details
- [Setup Guide](setup-guide.md) — local development
- [Environment Variables](../resources/environment-vars.md) — all config
