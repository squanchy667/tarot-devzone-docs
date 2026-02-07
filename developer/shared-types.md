# Shared Types

## Overview

The `shared/` package contains TypeScript enums, interfaces, and Zod validation schemas used by both the server and client. It's published as an npm workspace dependency.

## Enums

### TribeType
```typescript
enum TribeType {
  None = "None",
  Pentacles = "Pentacles",
  Cups = "Cups",
  Swords = "Swords",
  Wands = "Wands"
}
```

### AbilityTrigger
```typescript
enum AbilityTrigger {
  None = "None",
  Battlecry = "Battlecry",
  Deathrattle = "Deathrattle",
  OnAttack = "OnAttack",
  OnDamaged = "OnDamaged",
  StartOfCombat = "StartOfCombat",
  EndOfTurn = "EndOfTurn"
}
```

### AbilityEffectType
```typescript
enum AbilityEffectType {
  None = "None",
  BuffAttack = "BuffAttack",
  BuffHealth = "BuffHealth",
  BuffStats = "BuffStats",
  BuffAdjacentAttack = "BuffAdjacentAttack",
  BuffAdjacentHealth = "BuffAdjacentHealth",
  BuffAdjacentStats = "BuffAdjacentStats",
  BuffAllFriendlyAttack = "BuffAllFriendlyAttack",
  BuffAllFriendlyHealth = "BuffAllFriendlyHealth",
  GainAegis = "GainAegis",
  GainCoins = "GainCoins",
  DamageRandomEnemy = "DamageRandomEnemy",
  DamageAllEnemies = "DamageAllEnemies",
  BuffRandomFriendly = "BuffRandomFriendly",
  Taunt = "Taunt"
}
```

### SynergyTrigger
```typescript
enum SynergyTrigger {
  Passive = "Passive",
  StartOfCombat = "StartOfCombat",
  EndOfCombat = "EndOfCombat",
  OnSell = "OnSell",
  OnBuy = "OnBuy",
  OnDeath = "OnDeath",
  EndOfTurn = "EndOfTurn"
}
```

### SynergyEffect
```typescript
enum SynergyEffect {
  BuffAttack = "BuffAttack",
  BuffHealth = "BuffHealth",
  BuffStats = "BuffStats",
  BonusGold = "BonusGold",
  ReduceCost = "ReduceCost",
  Shield = "Shield",
  HealFlat = "HealFlat",
  // ... additional effects
}
```

### SynergyTarget
```typescript
enum SynergyTarget {
  AllTribeMembers = "AllTribeMembers",
  AllFriendly = "AllFriendly",
  Adjacent = "Adjacent",
  Random = "Random",
  Self = "Self"
}
```

## Type Interfaces

### CardData
```typescript
interface CardData {
  id: string;                      // unique slug (e.g., "flame-dancer")
  cardName: string;
  tier: number;                    // 1-6
  attack: number;
  health: number;
  tribes: TribeType[];             // multi-tribe support
  ability: string;                 // display text
  abilityTrigger: AbilityTrigger;
  abilityEffect: AbilityEffectType;
  abilityValue: number;
  effectType: string;              // legacy effect system
  buyCostModifier: number;
  sellValueModifier: number;
  imageUrl: string;                // S3 URL to card art
}
```

### SynergyData
```typescript
interface SynergyData {
  tribeType: TribeType;
  tribeName: string;
  description: string;
  themeColor: string;              // hex color
  tiers: SynergyTierData[];
  combo?: SynergyComboData;
}

interface SynergyTierData {
  threshold: number;               // 2, 4, or 6
  trigger: SynergyTrigger;
  effect: SynergyEffect;
  target: SynergyTarget;
  value: number;
  description: string;
}

interface SynergyComboData {
  tribe: TribeType;                // partner tribe
  threshold: number;
  effect: SynergyEffect;
  value: number;
  description: string;
}
```

### GameConfigData
```typescript
interface GameConfigData {
  tierCopies: Record<number, number>;       // {1:16, 2:15, 3:13, 4:11, 5:9, 6:7}
  shopSizes: Record<number, number>;        // {1:3, 2:4, 3:4, 4:5, 5:5, 6:6}
  baseBuyCost: number;                      // 3
  goldenMultiplier: number;                 // 2
  recruitTimerSeconds: number;              // 35
  startingGold: number;                     // 3
  goldPerTurn: number;                      // 1
  maxGold: number;                          // 10
  tavernUpgradeCosts: Record<number, number>; // {2:5, 3:8, 4:11, 5:11, 6:11}
  startingHealth: number;                   // 40
}
```

### ThemeData
```typescript
interface ThemeData {
  gameName: string;
  tribes: Record<TribeType, {
    name: string;
    description: string;
    color: string;                 // hex color
  }>;
  colors: {
    primary: string;
    secondary: string;
    accent: string;
    positive: string;
    negative: string;
  };
  uiText: {
    shopTitle: string;
    handTitle: string;
    boardTitle: string;
    buyButton: string;
    sellButton: string;
  };
}
```

### Other Types
```typescript
interface User {
  userId: string;
  email: string;
  role: "admin" | "editor" | "viewer";
}

interface ApiResponse<T> {
  data?: T;
  error?: string;
}

interface DeployStatus {
  status: "InProgress" | "Completed";
  invalidationId: string;
}

interface VersionDiff {
  added: string[];
  removed: string[];
  modified: { id: string; changes: Record<string, { old: any; new: any }> }[];
}
```

## Zod Schemas

Every type has a corresponding Zod schema for validation:

```typescript
import { cardSchema, synergySchema, gameConfigSchema, themeSchema } from 'shared';
```

These are used by the server's validation middleware to validate request bodies before processing.

## Related Documentation

- [API Reference](api-reference.md) — endpoints using these types
- [Data Formats](../resources/data-formats.md) — JSON examples
