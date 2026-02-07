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

The client doesn't need environment variables in local development — Vite's proxy handles API routing.

For deployed builds, the API URL is configured in the built JavaScript. The `deploy-devzone.sh` script handles this.

## Generating a JWT Secret

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Use the output as your `JWT_SECRET`.

## Related Documentation

- [Setup Guide](../developer/setup-guide.md) — where to put .env
- [AWS Infrastructure](../developer/aws-infrastructure.md) — SAM template params
- [Deployment](../developer/deployment.md) — deploy process
