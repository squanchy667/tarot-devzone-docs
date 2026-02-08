# Roadmap

## Current Status

DevZone is **fully implemented** and ready for deployment. All core features are complete.

## Completed

- Card CRUD editor with search/filter
- Synergy editor with tiered thresholds
- Balance tuner for economy and pool settings
- Theme editor with color pickers
- Version management with snapshots, publish, rollback, diff
- One-click deploy with CloudFront invalidation
- JWT auth with role-based access (admin/editor/viewer)
- Draft/live data separation
- Image upload via S3 presigned URLs
- AWS SAM infrastructure template
- Deploy scripts (setup + redeploy)
- Shared TypeScript types with Zod validation
- React Query data fetching with cache invalidation

## Completed (Post-Deploy)

- Deployed infrastructure to AWS via SAM
- Seeded admin user (DynamoDB)
- Exported initial game data (30 cards, 4 synergies, config, theme) to S3
- Frontend live on S3 static website hosting
- API live on Lambda + API Gateway (HTTP API)
- Fixed CORS for production (reflect origin mode)
- Fixed SAM build for npm workspace packages (Makefile approach)
- Build-time API URL injection for frontend (VITE_API_URL)
- S3 public access bucket policy for frontend hosting

## Completed (Live Game Integration - Feb 7, 2026)

- Wired Unity game to load data from S3 at startup via RuntimeDataLoader
- Added 5 missing original cards to card pool (30 -> 35 cards)
- Enabled public-read bucket policy on `live/*` prefix for game client access
- Updated SAM template with `GameDataBucketPolicy` resource
- CORS verified for CloudFront game origin
- End-to-end pipeline: edit in DevZone -> publish to live -> game picks up changes on next load

## Next Steps

### Phase 2: Polish
- Add bulk CSV import/export for cards
- Add card preview showing in-game appearance
- Add undo/redo for edits
- Improve error messages and loading states
- Mobile-responsive layout improvements

### Phase 3: Advanced Features
- Audit log (who changed what, when)
- A/B testing support (multiple live configs)
- Card art generation tools
- Real-time collaboration (multiple editors)
- Automated balance testing (simulate games with config changes)

### Phase 4: Operations
- CI/CD pipeline (GitHub Actions)
- Automated backups
- Monitoring and alerting
- Rate limiting on API
- Custom domain for DevZone
