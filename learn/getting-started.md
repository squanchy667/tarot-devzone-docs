# Getting Started

## What is DevZone?

DevZone is a web-based editor for the Tarot Battlegrounds game. It lets you edit cards, synergies, balance settings, and visual themes — all from a browser. When you publish changes, the game loads the updated data on its next startup.

## Who is it for?

- **Game designers** — balance cards, tune economy, adjust synergies
- **Artists** — upload card art
- **Non-developers** — make game changes without touching Unity or code

## How it works

```
1. You edit game data in DevZone (browser)
2. Changes are saved as drafts (safe, doesn't affect live game)
3. You click "Publish" when ready
4. Game data JSON is updated on S3
5. CloudFront cache is invalidated
6. Players see the new data next time they load the game
```

## Local Development

```bash
# Clone and install
git clone <repo-url> tarot-devzone
cd tarot-devzone
npm install

# Build shared types (required first)
npm run build --workspace=shared

# Start both server and client
npm run dev
```

- Client: `http://localhost:5173`
- Server: `http://localhost:3001`

The client proxies `/api` requests to the server automatically.

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Drafts** | Your work-in-progress edits (saved to S3 drafts/ prefix) |
| **Live** | The data the game actually uses (S3 live/ prefix) |
| **Publish** | Copy drafts to live + invalidate CDN cache |
| **Version** | A snapshot of live data at a point in time (for rollback) |
| **Shared types** | TypeScript enums/interfaces/schemas used by both client and server |

## Project Structure

```
tarot-devzone/
├── shared/     # Types + validation (used by both)
├── server/     # Express API → Lambda
├── client/     # React frontend → S3
├── scripts/    # Seed DB, export data
└── aws/        # SAM template + deploy scripts
```

## Next Steps

- [Setup Guide](../developer/setup-guide.md) — full local dev setup
- [User Guide](../product/user-guide.md) — how to use the editor
- [Architecture](../developer/architecture.md) — system design
- [API Reference](../developer/api-reference.md) — all endpoints
