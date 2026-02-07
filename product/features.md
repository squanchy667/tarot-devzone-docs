# Features

## Card Editor

Full CRUD management for all game cards:

- **List view** with search by name, filter by tribe and tier
- **Edit form** with all card fields: name, tier, attack, health, tribes, ability, costs
- **Create/delete** cards with validation
- **Image upload** for card art via S3 presigned URLs
- **Multi-tribe support** — cards can belong to multiple tribes

## Synergy Editor

Configure tribe synergy bonuses:

- **Per-tribe panels** with expandable accordion UI
- **Tiered thresholds** — define bonuses at 2/4/6 tribe members
- **Effect configuration** — trigger, effect type, target, value
- **Cross-tribe combos** — define bonus when two tribes are both active

## Balance Tuner

Adjust game economy and pool settings:

- **Economy**: base buy cost, starting gold, gold per turn, max gold, starting health
- **Card pool**: copies per tier (how many of each card exist)
- **Shop sizes**: cards offered per tier level
- **Tavern upgrades**: cost to upgrade to each tier

## Theme Editor

Customize visual appearance:

- **Game name** — title shown in-game
- **Tribe colors** — color picker for each tribe's theme
- **Tribe names/descriptions** — rename or describe tribes
- **UI text** — shop title, hand title, board title, buy/sell button labels

## Version Management

Snapshot and rollback system:

- **Create snapshots** — save current state with description
- **Version history** — timestamped list with author
- **Publish** — set any version as live
- **Rollback** — revert to a previous version
- **Diff view** — see what changed between versions

## Deploy

One-click publishing:

- **Publish to live** — copies draft data to live S3 prefix
- **CloudFront invalidation** — automatically clears CDN cache
- **Status tracking** — polls invalidation progress
- **Live game link** — quick link to the deployed game

## Authentication

Role-based access:

- **Admin** — full access, can register users
- **Editor** — edit all game data, cannot manage users
- **Viewer** — read-only access

## Draft/Live Separation

Safe editing workflow:

- All edits save to `drafts/` on S3
- Live game reads from `live/` on S3
- Publishing copies `drafts/` to `live/`
- Editors can't accidentally break the live game
