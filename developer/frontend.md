# Frontend

## Tech Stack

- **React 18** with TypeScript
- **Vite** for dev server and builds
- **Tailwind CSS** for styling
- **React Router DOM** for client-side routing
- **@tanstack/react-query** for data fetching and cache management
- **lucide-react** for icons
- **react-colorful** for color pickers
- **react-dropzone** for image upload

## Routes

| Path | Component | Description |
|------|-----------|-------------|
| `/login` | LoginPage | Email/password auth form |
| `/` | Dashboard | Stats overview, quick actions |
| `/cards` | CardEditor | Card list + edit form |
| `/synergies` | SynergyEditor | Tribe synergy configuration |
| `/balance` | BalanceTuner | Economy and pool settings |
| `/theme` | ThemeEditor | Colors, names, UI text |
| `/versions` | VersionManager | Snapshot, publish, rollback |
| `/deploy` | DeployPanel | Publish drafts to live |

## Component Tree

```
App.tsx
├── LoginPage (unauthenticated)
└── Layout (authenticated)
    ├── Sidebar — navigation links
    ├── Header — user info, logout
    └── <Outlet> — page content
        ├── Dashboard
        ├── CardEditor
        │   └── CardForm
        ├── SynergyEditor
        ├── BalanceTuner
        ├── ThemeEditor
        ├── VersionManager
        └── DeployPanel
```

## Hooks

### useAuth()
Manages JWT token in localStorage.

```typescript
const { user, token, login, logout, isAuthenticated } = useAuth();
```

- `login(email, password)` — calls `/api/auth/login`, stores token
- `logout()` — clears token from localStorage
- `isAuthenticated` — true if token exists and is valid

### useCards()
React Query hooks for card CRUD.

```typescript
const { data: cards, isLoading } = useCards();
const createCard = useCreateCard();
const updateCard = useUpdateCard();
const deleteCard = useDeleteCard();
```

- Queries auto-refetch on window focus
- Mutations invalidate cache on success

### useSynergies()
React Query hooks for synergy management.

```typescript
const { data: synergies } = useSynergies();
const updateSynergy = useUpdateSynergy();
```

## API Client

`services/api.ts` provides a centralized HTTP client:

```typescript
import { api } from '../services/api';

// All requests include Bearer token automatically
const cards = await api.getCards();
const card = await api.createCard(cardData);
await api.updateCard(id, changes);
await api.deleteCard(id);
```

- Automatically attaches JWT from localStorage
- Redirects to `/login` on 401 responses
- Base URL: `/api` (proxied to server in dev via Vite config)

## Key Components

### CardEditor
Two-panel layout:
- **Left panel**: Searchable card list with tribe/tier filters
- **Right panel**: CardForm with all fields

### CardForm
Full card editor with:
- Name, tier (1-6), attack, health
- Tribe checkboxes (Pentacles, Cups, Swords, Wands)
- Ability: trigger dropdown + effect dropdown + value
- Effect type dropdown (legacy system)
- Cost modifiers (buy/sell)
- Image URL field

### SynergyEditor
Expandable accordion per tribe showing:
- Tier definitions (threshold, trigger, effect, target, value)
- Add/remove tiers
- Cross-tribe combo configuration

### BalanceTuner
Grouped input sections:
- Economy: base buy cost, starting gold, gold per turn, max gold, starting health
- Pool: tier copies per tier (1-6)
- Shop: shop size per tier (1-6)
- Upgrades: tavern upgrade cost per tier (2-6)

### ThemeEditor
- Game name text field
- Tribe configuration: name, description, color (with color picker)
- UI text: shop title, hand title, board title, buy/sell button labels

### VersionManager
- Version history list with timestamps and authors
- "Create Snapshot" button with description input
- "Publish" / "Rollback" buttons per version
- "LIVE" badge on active version

### DeployPanel
- Large "Publish to Live" button
- Status indicator (polls CloudFront invalidation progress)
- Link to live game URL

## Vite Configuration

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:3001'  // Proxy API calls to Express
    }
  }
});
```

## Related Documentation

- [Hooks and API](api-reference.md) — endpoint details
- [Shared Types](shared-types.md) — TypeScript interfaces
- [User Guide](../product/user-guide.md) — how to use the editor
