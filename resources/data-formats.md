# Data Formats

## cards.json

Array of 35 card definitions. Stored at `live/cards.json` and `drafts/cards.json` on S3. The `live/` path is publicly readable (the Unity game fetches it at startup).

```json
[
  {
    "id": "flame-dancer",
    "cardName": "Flame Dancer",
    "tier": 2,
    "attack": 3,
    "health": 2,
    "tribes": ["Wands"],
    "ability": "OnAttack: Deal 1 extra damage",
    "abilityTrigger": "OnAttack",
    "abilityEffect": "BuffAttack",
    "abilityValue": 1,
    "effectType": "",
    "buyCostModifier": 0,
    "sellValueModifier": 0,
    "imageUrl": ""
  },
  {
    "id": "crystal-guardian",
    "cardName": "Crystal Guardian",
    "tier": 3,
    "attack": 2,
    "health": 6,
    "tribes": ["Cups", "Pentacles"],
    "ability": "Battlecry: Gain Aegis",
    "abilityTrigger": "Battlecry",
    "abilityEffect": "GainAegis",
    "abilityValue": 1,
    "effectType": "Guardian",
    "buyCostModifier": 1,
    "sellValueModifier": 0,
    "imageUrl": "https://cdn.../images/crystal-guardian.png"
  }
]
```

## synergies.json

Array of tribe synergy definitions:

```json
[
  {
    "tribeType": "Swords",
    "tribeName": "Swords",
    "description": "Aggressive attackers",
    "themeColor": "#DC2626",
    "tiers": [
      {
        "threshold": 2,
        "trigger": "StartOfCombat",
        "effect": "BuffAttack",
        "target": "AllTribeMembers",
        "value": 1,
        "description": "+1 Attack to all Swords"
      },
      {
        "threshold": 4,
        "trigger": "StartOfCombat",
        "effect": "BuffAttack",
        "target": "AllFriendly",
        "value": 1,
        "description": "+1 Attack to all friendly"
      },
      {
        "threshold": 6,
        "trigger": "StartOfCombat",
        "effect": "BuffAttack",
        "target": "AllTribeMembers",
        "value": 2,
        "description": "+2 Attack to all Swords"
      }
    ],
    "combo": {
      "tribe": "Wands",
      "threshold": 2,
      "effect": "BuffAttack",
      "value": 1,
      "description": "Swords + Wands: +1 Attack to all"
    }
  }
]
```

## config.json

Game balance configuration:

```json
{
  "tierCopies": { "1": 16, "2": 15, "3": 13, "4": 11, "5": 9, "6": 7 },
  "shopSizes": { "1": 3, "2": 4, "3": 4, "4": 5, "5": 5, "6": 6 },
  "baseBuyCost": 3,
  "goldenMultiplier": 2,
  "recruitTimerSeconds": 35,
  "startingGold": 3,
  "goldPerTurn": 1,
  "maxGold": 10,
  "tavernUpgradeCosts": { "2": 5, "3": 8, "4": 11, "5": 11, "6": 11 },
  "startingHealth": 40
}
```

## theme.json

Visual theme configuration:

```json
{
  "gameName": "Tarot Battlegrounds",
  "tribes": {
    "Pentacles": {
      "name": "Pentacles",
      "description": "Masters of coin and commerce",
      "color": "#F59E0B"
    },
    "Cups": {
      "name": "Cups",
      "description": "Healers and protectors",
      "color": "#3B82F6"
    },
    "Swords": {
      "name": "Swords",
      "description": "Aggressive warriors",
      "color": "#DC2626"
    },
    "Wands": {
      "name": "Wands",
      "description": "Mystic enchanters",
      "color": "#8B5CF6"
    }
  },
  "colors": {
    "primary": "#1E293B",
    "secondary": "#334155",
    "accent": "#F59E0B",
    "positive": "#22C55E",
    "negative": "#EF4444"
  },
  "uiText": {
    "shopTitle": "Tavern",
    "handTitle": "Hand",
    "boardTitle": "Board",
    "buyButton": "Buy",
    "sellButton": "Sell"
  }
}
```

## Version Metadata

Stored in DynamoDB (`devzone-versions` table) and as `versions/vNNN/metadata.json` on S3:

```json
{
  "versionId": "v003",
  "timestamp": "2026-02-07T12:00:00.000Z",
  "author": "admin@example.com",
  "description": "Balance update: nerfed Swords tier 3",
  "isLive": true
}
```

## S3 Key Structure

```
live/cards.json          — active card data (public, game reads this)
live/synergies.json      — active synergy data
live/config.json         — active balance config
live/theme.json          — active theme
live/images/*.png        — card art images

drafts/cards.json        — work-in-progress edits
drafts/synergies.json
drafts/config.json
drafts/theme.json

versions/v001/cards.json — snapshot for rollback
versions/v001/metadata.json
versions/v002/...
```

## Related Documentation

- [Shared Types](../developer/shared-types.md) — TypeScript interfaces
- [API Reference](../developer/api-reference.md) — endpoints that return these formats
- [Runtime Data Loading](https://github.com/TarotBattlegrounds-docs/developer/runtime-data.md) — how the game loads this data
