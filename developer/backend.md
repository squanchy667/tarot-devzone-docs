# Backend

## Overview

Express.js API server that manages game data on S3, user auth via DynamoDB, and CloudFront cache invalidation. Runs locally on port 3001 and deploys to AWS Lambda via `serverless-http`.

## Entry Points

| File | Purpose |
|------|---------|
| `src/index.ts` | Local dev server (Express on port 3001) |
| `src/app.ts` | Express app factory (routes, middleware, CORS) |
| `src/lambda.ts` | Lambda handler via `serverless-http(app)` |

## Routes

All routes are mounted under `/api`:

| File | Prefix | Endpoints |
|------|--------|-----------|
| `routes/auth.ts` | `/api/auth` | login, register, me |
| `routes/cards.ts` | `/api/cards` | CRUD cards |
| `routes/synergies.ts` | `/api/synergies` | list, update synergies |
| `routes/config.ts` | `/api/config` | get/update balance config |
| `routes/theme.ts` | `/api/theme` | get/update theme |
| `routes/images.ts` | `/api/images` | upload, presign, delete images |
| `routes/versions.ts` | `/api/versions` | snapshots, publish, rollback, diff |
| `routes/deploy.ts` | `/api/deploy` | publish to live, check status |

## Services

### S3 Service (`services/s3.ts`)

Operations on the game data S3 bucket:

| Method | Description |
|--------|-------------|
| `getJson<T>(key)` | Fetch and parse JSON from S3 |
| `putJson(key, data)` | Serialize and upload JSON to S3 |
| `deleteObject(key)` | Delete a file from S3 |
| `listObjects(prefix)` | List all keys under a prefix |
| `getUploadUrl(key, type)` | Get presigned upload URL (5 min expiry) |
| `uploadBuffer(key, buf, type)` | Upload binary data directly |

### DynamoDB Service (`services/dynamodb.ts`)

User and version management:

| Method | Description |
|--------|-------------|
| `getUserByEmail(email)` | Lookup user for login |
| `getUserById(userId)` | Get user by ID (for JWT verification) |
| `createUser(user)` | Create new user record |
| `listUsers()` | List all users (password hash excluded) |
| `createVersion(version)` | Store version metadata |
| `getVersion(versionId)` | Get version details |
| `listVersions()` | All versions sorted by timestamp DESC |
| `setVersionLive(versionId)` | Mark version as live, others inactive |

### CloudFront Service (`services/cloudfront.ts`)

CDN cache management:

| Method | Description |
|--------|-------------|
| `invalidateCache(paths)` | Create invalidation for given paths |
| `getInvalidationStatus(id)` | Check invalidation progress |

### Version Service (`services/version.ts`)

Version snapshot and rollback logic:

| Method | Description |
|--------|-------------|
| `createVersionSnapshot(author, desc)` | Copy live/ to versions/vNNN/ |
| `publishVersion(versionId)` | Copy version files to live/ |
| `diffVersion(versionId)` | Compute diff between version and live |

## Middleware

### Auth Middleware (`middleware/auth.ts`)

- Extracts JWT from `Authorization: Bearer <token>` header
- Verifies token with `JWT_SECRET`
- Attaches `req.user` with `userId`, `email`, `role`
- Returns 401 if token missing/invalid

### Validation Middleware (`middleware/validation.ts`)

- Accepts a Zod schema
- Validates `req.body` against schema
- Returns 400 with validation errors if invalid
- Passes through if valid

## Data Storage Pattern

All game data follows the same pattern:

```
GET  /api/{resource}           → s3.getJson("drafts/{resource}.json")
PUT  /api/{resource}           → s3.putJson("drafts/{resource}.json", data)
POST /api/deploy/publish       → copy drafts/*.json → live/*.json
```

- Edits always go to `drafts/` prefix
- Publishing copies `drafts/` to `live/`
- Game only reads from `live/`

## CORS Configuration

In production, the server uses origin reflection (`origin: true`) instead of a static origin list. This is required because `credentials: true` is incompatible with `origin: '*'` per the CORS spec.

```typescript
app.use(cors({
  origin: true,       // reflects request origin
  credentials: true,
}));
```

## Lambda Deployment

The Express app is wrapped with `serverless-http`:

```typescript
// lambda.ts
import serverless from 'serverless-http';
import { createApp } from './app';

const app = createApp();
export const handler = serverless(app);
```

This allows the same Express app to run locally and on Lambda without code changes.

The Lambda package is built using a custom Makefile (see [Deployment](deployment.md#sam-build-workspace-package-handling)) to handle workspace dependency resolution.

## Related Documentation

- [API Reference](api-reference.md) — endpoint request/response details
- [Shared Types](shared-types.md) — Zod schemas used for validation
- [AWS Infrastructure](aws-infrastructure.md) — Lambda, API Gateway config
