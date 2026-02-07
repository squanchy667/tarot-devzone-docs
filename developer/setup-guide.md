# Setup Guide

## Prerequisites

- Node.js 20+
- npm 9+
- AWS CLI configured with credentials (for S3/DynamoDB access)
- Git

## Clone and Install

```bash
git clone <repo-url> tarot-devzone
cd tarot-devzone
npm install
```

This installs dependencies for all three workspaces (shared, server, client).

## Build Shared Types

```bash
npm run build --workspace=shared
```

Must be done before running server or client, as both depend on `shared`.

## Environment Variables

Create `.env` in the `server/` directory:

```bash
cp .env.example server/.env
```

Required variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `AWS_REGION` | AWS region | `us-east-1` |
| `S3_BUCKET` | Game data bucket name | `tarot-battlegrounds-data-dev` |
| `DYNAMODB_USERS_TABLE` | Users table name | `devzone-users-dev` |
| `DYNAMODB_VERSIONS_TABLE` | Versions table name | `devzone-versions-dev` |
| `CLOUDFRONT_DISTRIBUTION_ID` | Data CDN distribution | `E1234567890` |
| `JWT_SECRET` | Secret for signing tokens | `your-secret-here` |
| `GAME_URL` | Live game URL | `https://dui22oafwco41.cloudfront.net` |

## Run Locally

### Both server and client (recommended)

```bash
npm run dev
```

This starts:
- Server on `http://localhost:3001`
- Client on `http://localhost:5173` (with `/api` proxy to server)

### Server only

```bash
npm run dev:server
```

### Client only

```bash
npm run dev:client
```

## Seed Database

Create an initial admin user:

```bash
npx tsx scripts/seed-db.ts
```

This creates an admin user with credentials defined in the script.

## Export Game Data

Generate initial JSON from the game's CardDatabase:

```bash
npx tsx scripts/export-game-data.ts
```

This creates `cards.json`, `synergies.json`, `config.json`, and `theme.json` and uploads them to S3.

## Build for Production

```bash
npm run build
```

Builds all three workspaces:
1. `shared` — compiles TypeScript to `shared/dist/`
2. `server` — compiles to `server/dist/`
3. `client` — builds to `client/dist/`

## Project Structure

```
tarot-devzone/
├── package.json           # Root workspace config
├── .env.example           # Environment variables template
├── shared/                # Shared types (npm workspace)
│   ├── enums.ts           # TribeType, AbilityTrigger, etc.
│   ├── types.ts           # CardData, SynergyData, etc.
│   └── schemas.ts         # Zod validation schemas
├── server/                # Express API
│   └── src/
│       ├── index.ts       # Local dev entry (port 3001)
│       ├── app.ts         # Express app + routes
│       ├── lambda.ts      # Lambda handler
│       ├── routes/        # API route handlers
│       ├── services/      # S3, DynamoDB, CloudFront, versioning
│       └── middleware/    # Auth, validation
├── client/                # React frontend
│   └── src/
│       ├── App.tsx        # Router + layout
│       ├── components/    # Page components
│       ├── hooks/         # React Query hooks
│       └── services/      # API client
├── scripts/               # Seed DB, export data
└── aws/                   # SAM template + deploy scripts
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `shared` import errors | Run `npm run build --workspace=shared` |
| 401 on API calls | Check JWT_SECRET matches, token not expired |
| S3 access denied | Check AWS credentials and bucket policy |
| DynamoDB table not found | Run `aws/setup-infra.sh` or check table names in `.env` |
| Vite proxy not working | Ensure server is running on port 3001 |

## Related Documentation

- [Architecture](architecture.md) — system overview
- [Environment Variables](../resources/environment-vars.md) — all env vars explained
- [Deployment](deployment.md) — deploying to AWS
