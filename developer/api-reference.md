# API Reference

Base URL: `http://localhost:3001/api` (local) or `https://{api-gateway-url}/api` (deployed)

All endpoints except `/auth/login` require a Bearer JWT token in the `Authorization` header.

## Authentication

### POST /api/auth/login
Login and receive JWT token.

**Request:**
```json
{ "email": "admin@example.com", "password": "secret" }
```

**Response:**
```json
{ "token": "eyJhbG...", "user": { "userId": "...", "email": "...", "role": "admin" } }
```

### POST /api/auth/register
Create a new user. **Admin only.**

**Request:**
```json
{ "email": "editor@example.com", "password": "secret", "role": "editor" }
```

### GET /api/auth/me
Get current authenticated user.

**Response:**
```json
{ "userId": "...", "email": "...", "role": "admin" }
```

---

## Cards

### GET /api/cards
List all cards. Query param `?source=live|drafts` (default: drafts).

**Response:** `CardData[]`

### GET /api/cards/:id
Get single card by ID.

### POST /api/cards
Create a new card. Validated with `cardSchema`.

**Request:** `CardData` object (see [Data Formats](../resources/data-formats.md))

### PUT /api/cards/:id
Update existing card.

**Request:** Partial `CardData` (merged with existing)

### DELETE /api/cards/:id
Delete card by ID.

---

## Synergies

### GET /api/synergies
List all synergy definitions. Query param `?source=live|drafts`.

**Response:** `SynergyData[]`

### PUT /api/synergies/:tribe
Update or create synergy for a tribe (e.g., `Swords`).

**Request:** `SynergyData` object

---

## Config

### GET /api/config
Get game balance configuration. Query param `?source=live|drafts`.

**Response:** `GameConfigData`

### PUT /api/config
Update game configuration.

**Request:** `GameConfigData` object

---

## Theme

### GET /api/theme
Get theme configuration. Query param `?source=live|drafts`.

**Response:** `ThemeData`

### PUT /api/theme
Update theme.

**Request:** `ThemeData` object

---

## Images

### POST /api/images/upload
Upload card image directly. Max 5MB. Uses `multipart/form-data`.

**Request:** Form field `image` with file

**Response:**
```json
{ "url": "https://s3.../live/images/flame-dancer.png" }
```

### POST /api/images/presign
Get presigned upload URL (for large files, bypasses Lambda 6MB limit).

**Request:**
```json
{ "key": "images/flame-dancer.png", "contentType": "image/png" }
```

**Response:**
```json
{ "uploadUrl": "https://s3.../presigned...", "publicUrl": "https://cdn.../live/images/..." }
```

### DELETE /api/images/:key
Delete an image from S3. Key is URL-encoded path.

---

## Versions

### GET /api/versions
List all version snapshots, sorted by timestamp descending.

**Response:**
```json
[
  {
    "versionId": "v003",
    "timestamp": "2026-02-07T12:00:00Z",
    "author": "admin@example.com",
    "description": "Balance update",
    "isLive": true
  }
]
```

### POST /api/versions
Create a new version snapshot (copies current live data to `versions/vNNN/`).

**Request:**
```json
{ "description": "Balance update for swords tribe" }
```

### POST /api/versions/:id/publish
Publish a version to live (copies version files to `live/`).

### POST /api/versions/:id/rollback
Rollback to a previous version (same as publish but for older versions).

### GET /api/versions/:id/diff
Compare version against current live data.

**Response:**
```json
{
  "added": ["new-card-id"],
  "removed": ["old-card-id"],
  "modified": [{ "id": "flame-dancer", "changes": { "attack": { "old": 3, "new": 4 } } }]
}
```

---

## Deploy

### POST /api/deploy/publish
Push draft data to live and invalidate CloudFront cache.

**Response:**
```json
{ "invalidationId": "I1234567", "message": "Published to live" }
```

### GET /api/deploy/status
Check CloudFront invalidation status.

**Query:** `?invalidationId=I1234567`

**Response:**
```json
{ "status": "Completed" }
```

---

## Error Responses

All errors follow this format:

```json
{ "error": "Error message here" }
```

| Status | Meaning |
|--------|---------|
| 400 | Validation error (Zod schema failure) |
| 401 | Missing or invalid JWT token |
| 403 | Insufficient role permissions |
| 404 | Resource not found |
| 500 | Server error |

## Related Documentation

- [Backend](backend.md) — route implementation details
- [Shared Types](shared-types.md) — TypeScript interfaces and Zod schemas
- [Data Formats](../resources/data-formats.md) — JSON schema examples
