# Environment Variables

## Server (.env)

All variables go in `server/.env`. Copy from `.env.example` in the repo root.

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `AWS_REGION` | Yes | AWS region for all services | `us-east-1` |
| `S3_BUCKET` | Yes | S3 bucket for game data JSON + images | `tarot-battlegrounds-data-dev` |
| `DYNAMODB_USERS_TABLE` | Yes | DynamoDB table for user accounts | `devzone-users-dev` |
| `DYNAMODB_VERSIONS_TABLE` | Yes | DynamoDB table for version metadata | `devzone-versions-dev` |
| `CLOUDFRONT_DISTRIBUTION_ID` | Yes | Game data CloudFront distribution (for cache invalidation) | `E1ABCDEF12345` |
| `JWT_SECRET` | Yes | Secret key for signing JWT tokens | `your-random-secret-here` |
| `GAME_URL` | No | URL to the live game (shown in deploy panel) | `https://dui22oafwco41.cloudfront.net` |
| `PORT` | No | Local dev server port (default: 3001) | `3001` |

## Lambda Environment

When deployed to Lambda, these same variables are set in the SAM template (`aws/template.yaml`) under the function's `Environment.Variables` section. They are populated from SAM parameters during `sam deploy`.

## Client Environment

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `VITE_API_URL` | Build-time | API base URL baked into production build | `https://{api-id}.execute-api.{region}.amazonaws.com/api` |

In local development, Vite's proxy handles API routing (no env var needed). For deployed builds, the deploy script sets `VITE_API_URL` at build time after the API Gateway URL is known from CloudFormation outputs:

```bash
VITE_API_URL="${API_URL}/api" npm run build
```

In client code, the API base URL is resolved as:
```typescript
const API_BASE = import.meta.env.VITE_API_URL || '/api';
```

## Generating a JWT Secret

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Use the output as your `JWT_SECRET`.

## Related Documentation

- [Setup Guide](../developer/setup-guide.md) — where to put .env
- [AWS Infrastructure](../developer/aws-infrastructure.md) — SAM template params
- [Deployment](../developer/deployment.md) — deploy process
