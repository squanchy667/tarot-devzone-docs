# Architecture

## System Overview

```
                    ┌─────────────────┐
                    │   DevZone Web    │
                    │  (React + Vite)  │
                    └────────┬────────┘
                             │ HTTP + Bearer JWT
                    ┌────────▼────────┐
                    │   API Gateway    │
                    │   (HTTP API)     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │     Lambda       │
                    │ (Express + SH)   │
                    └──┬─────┬─────┬──┘
                       │     │     │
              ┌────────▼┐ ┌─▼───┐ ┌▼─────────┐
              │ DynamoDB │ │ S3  │ │CloudFront │
              │(users,   │ │(JSON│ │(cache     │
              │ versions)│ │+img)│ │ invalidn) │
              └──────────┘ └──┬──┘ └───────────┘
                              │
                    ┌─────────▼───────┐
                    │  Unity WebGL     │
                    │  Game Client     │
                    │  (fetches JSON)  │
                    └─────────────────┘
```

## Monorepo Structure

```
tarot-devzone/
├── shared/          # TypeScript types + Zod schemas (npm workspace)
├── server/          # Express API backend
├── client/          # React + Vite frontend
├── scripts/         # DB seeding, data export
└── aws/             # SAM template + deploy scripts
```

All three packages (`shared`, `server`, `client`) are npm workspaces. `shared` is a dependency of both `server` and `client`.

## Data Flow

### Edit Flow
```
User edits card in browser
  → React mutation (useCards hook)
  → POST /api/cards (Bearer JWT)
  → Express validates with Zod schema
  → S3 putJson("drafts/cards.json", updatedCards)
  → React Query invalidates cache
  → UI refreshes with new data
```

### Publish Flow
```
User clicks "Publish to Live"
  → POST /api/deploy/publish
  → Copy drafts/*.json → live/*.json on S3
  → CloudFront invalidation for /live/*
  → DynamoDB: version record created
  → Unity game fetches updated JSON on next startup
```

### Version Flow
```
User creates version snapshot
  → POST /api/versions
  → Copy live/*.json → versions/vNNN/*.json on S3
  → DynamoDB: version metadata stored
  → Can rollback anytime by copying version → live
```

## S3 Storage Layout

```
s3://tarot-battlegrounds-data-{env}/
├── live/                    # Currently active data (fetched by game)
│   ├── cards.json
│   ├── synergies.json
│   ├── config.json
│   ├── theme.json
│   └── images/              # Card art PNGs
├── drafts/                  # Work-in-progress (edited by DevZone)
│   ├── cards.json
│   ├── synergies.json
│   ├── config.json
│   └── theme.json
└── versions/                # Snapshots for rollback
    ├── v001/
    │   ├── cards.json
    │   ├── synergies.json
    │   ├── config.json
    │   ├── theme.json
    │   └── metadata.json
    └── v002/
```

## DynamoDB Tables

| Table | Partition Key | Attributes |
|-------|--------------|------------|
| `devzone-users-{env}` | userId | email, passwordHash, role, createdAt |
| `devzone-versions-{env}` | versionId | timestamp, author, description, isLive |

## Authentication Model

- JWT tokens with 24h expiry
- Three roles: `admin`, `editor`, `viewer`
- Admin: can register users, full CRUD, deploy
- Editor: can edit cards/synergies/config/theme, create versions
- Viewer: read-only access

## Key Design Decisions

1. **Draft/Live separation** — editors can't accidentally break the live game
2. **Zod schemas shared** — same validation on client and server, single source of truth
3. **serverless-http** — same Express app runs locally (port 3001) and on Lambda
4. **React Query** — automatic cache invalidation after mutations, optimistic updates
5. **S3 as database** — game data is just JSON files, no need for a traditional DB
6. **Presigned URLs** — image uploads bypass Lambda's 6MB limit
7. **Makefile SAM build** — handles npm workspace packages that can't resolve during isolated `npm install` in SAM's build process
8. **Build-time API URL** — frontend bakes the API Gateway URL at build time via `VITE_API_URL`, since the frontend and API are on different origins in production
9. **CORS origin reflection** — uses `origin: true` instead of a static allowlist, required for `credentials: true` compatibility

## Related Documentation

- [Backend](backend.md) — Express routes and services
- [Frontend](frontend.md) — React components and hooks
- [AWS Infrastructure](aws-infrastructure.md) — SAM template details
