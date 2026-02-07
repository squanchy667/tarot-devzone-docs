# Tech Stack

## Frontend

| Technology | Purpose |
|-----------|---------|
| React 18 | UI framework |
| TypeScript | Type safety |
| Vite | Dev server + build tool |
| Tailwind CSS | Utility-first styling |
| React Router DOM | Client-side routing |
| @tanstack/react-query | Data fetching, caching, mutations |
| lucide-react | Icons |
| react-colorful | Color picker (theme editor) |
| react-dropzone | Drag-and-drop image upload |

## Backend

| Technology | Purpose |
|-----------|---------|
| Express | Web framework |
| TypeScript | Type safety |
| serverless-http | Lambda adapter (same code locally + deployed) |
| jsonwebtoken | JWT auth token signing/verification |
| bcryptjs | Password hashing |
| multer | Multipart file upload handling |
| zod | Request validation (shared with frontend) |

## AWS Services

| Service | Purpose | Cost |
|---------|---------|------|
| Lambda | API compute (Node.js 20.x) | Free tier |
| API Gateway | HTTP routing to Lambda | Free tier |
| S3 | Game data JSON, card images, frontend hosting | ~$0.10/mo |
| DynamoDB | User accounts, version metadata | Free tier |
| CloudFront | CDN for game data + DevZone frontend | Free tier |
| SAM/CloudFormation | Infrastructure as code | Free |

## Shared

| Technology | Purpose |
|-----------|---------|
| Zod | Schema validation (shared between client + server) |
| TypeScript | Shared type definitions |
| npm workspaces | Monorepo dependency management |

## Dev Tools

| Tool | Purpose |
|------|---------|
| npm workspaces | Monorepo management |
| AWS CLI | Infrastructure management |
| AWS SAM CLI | Lambda deployment |
| concurrently | Run server + client in parallel |
| tsx | Run TypeScript scripts directly |
