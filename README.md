# Tarot DevZone Documentation

> Comprehensive documentation for Tarot DevZone — A web-based game data editor for Tarot Battlegrounds

[![Status](https://img.shields.io/badge/Status-Ready%20to%20Deploy-green)]()
[![React](https://img.shields.io/badge/Frontend-React%20+%20Vite-blue)]()
[![Express](https://img.shields.io/badge/Backend-Express%20+%20Lambda-orange)]()
[![AWS](https://img.shields.io/badge/Infra-AWS%20SAM-yellow)]()

## About

Tarot DevZone is a web-based editor that allows non-developers to edit cards, synergies, balance parameters, and upload card art for the Tarot Battlegrounds game — all from a browser. Changes are pushed to S3 and loaded by the game at startup.

## Architecture

```
[DevZone React App]  ->  [Express API on Lambda]  ->  [S3: game-data JSON + images]
                                                             |
[Unity WebGL Game]   <---- fetches cards.json, synergies.json, config.json at startup
```

## Features

- Card CRUD editor with tribe/ability/tier configuration
- Synergy editor with tiered thresholds per tribe
- Balance tuner for economy, pool sizes, shop sizes
- Theme editor with color pickers and UI text
- Version management with snapshots, diff, rollback
- One-click deploy to live (S3 + CloudFront invalidation)
- Image upload for card art (S3 presigned URLs)
- Role-based auth (admin, editor, viewer)
- Draft/live separation — edit safely, publish when ready

## Documentation

- **[Developer](developer/)** — Architecture, API, frontend, backend, deployment
- **[Product](product/)** — Features, user guide, roadmap
- **[Learn](learn/)** — Getting started
- **[Resources](resources/)** — Tech stack, environment variables, data formats

## Quick Start

### For Developers
1. [Getting Started](learn/getting-started.md)
2. [Setup Guide](developer/setup-guide.md)
3. [Architecture](developer/architecture.md)
4. [Deployment](developer/deployment.md)

### For Agents
1. [Architecture](developer/architecture.md) — system overview
2. [API Reference](developer/api-reference.md) — all endpoints
3. [Frontend](developer/frontend.md) — React components
4. [Backend](developer/backend.md) — Express services

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite + TypeScript + Tailwind CSS |
| Backend | Express + TypeScript + serverless-http |
| Auth | JWT tokens + bcrypt passwords |
| Validation | Zod (shared schemas) |
| Storage | S3 (JSON + images), DynamoDB (users + versions) |
| CDN | CloudFront (game data + DevZone frontend) |
| Compute | Lambda (API) + API Gateway (HTTP) |
| IaC | AWS SAM (CloudFormation) |

## Related Repos

| Repo | Description |
|------|-------------|
| TarotBattlegrounds-POC | Unity game (consumes game data from S3) |
| TarotBattlegrounds-docs | Game documentation |
| tarot-devzone | This project's source code |

---

Last Updated: February 7, 2026
