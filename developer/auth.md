# Authentication

## Overview

DevZone uses JWT-based authentication with role-based access control. Passwords are hashed with bcrypt and stored in DynamoDB.

## Auth Flow

```
1. User submits email + password on LoginPage
2. POST /api/auth/login
3. Server looks up user in DynamoDB by email
4. bcrypt.compare(password, storedHash)
5. If match → sign JWT with { userId, email, role }, return token
6. Client stores token in localStorage
7. All subsequent requests include Authorization: Bearer <token>
8. Auth middleware verifies token, attaches req.user
9. On 401 → client redirects to /login
```

## Roles

| Role | Permissions |
|------|------------|
| `admin` | Full access: CRUD all data, register users, deploy, manage versions |
| `editor` | Edit cards, synergies, config, theme. Create versions. Cannot register users. |
| `viewer` | Read-only access to all data. Cannot edit or deploy. |

## JWT Token

- **Algorithm**: HS256
- **Expiry**: 24 hours
- **Secret**: `JWT_SECRET` environment variable
- **Payload**: `{ userId, email, role }`

## DynamoDB Users Table

| Field | Type | Description |
|-------|------|-------------|
| userId | String (PK) | UUID |
| email | String | Unique login identifier |
| passwordHash | String | bcrypt hash |
| role | String | admin, editor, or viewer |
| createdAt | String | ISO timestamp |

## Seeding Admin User

```bash
npx tsx scripts/seed-db.ts
```

Creates the initial admin user. Edit the script to set email/password.

## Auth Middleware

```typescript
// middleware/auth.ts
export function requireAuth(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'No token' });

  const decoded = jwt.verify(token, JWT_SECRET);
  req.user = decoded;
  next();
}
```

Applied to all routes except `POST /api/auth/login`.

## Client-Side Auth

The `useAuth` hook manages token state:

```typescript
const { user, login, logout, isAuthenticated } = useAuth();
```

- Token stored in `localStorage` under key `devzone-token`
- `api.ts` reads token and attaches to all requests
- On 401 response, clears token and redirects to `/login`

## Related Documentation

- [API Reference](api-reference.md) — auth endpoints
- [Backend](backend.md) — middleware details
- [Environment Variables](../resources/environment-vars.md) — JWT_SECRET config
