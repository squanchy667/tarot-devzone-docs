# User Guide

## Logging In

1. Navigate to the DevZone URL in your browser
2. Enter your email and password
3. Click "Login"

If you don't have an account, ask an admin to create one.

## Dashboard

After login, you'll see the Dashboard with:
- Total card count and breakdown by tribe
- Number of synergy definitions
- Quick action links to common pages

## Editing Cards

1. Go to **Cards** in the sidebar
2. Browse the card list on the left (search/filter by tribe or tier)
3. Click a card to edit it in the right panel
4. Modify fields:
   - **Name**: Display name in-game
   - **Tier**: 1-6 (higher = stronger, unlocked later)
   - **Attack/Health**: Base combat stats
   - **Tribes**: Check one or more (Pentacles, Cups, Swords, Wands)
   - **Ability**: Choose trigger (Battlecry, Deathrattle, OnAttack) + effect + value
   - **Cost modifiers**: Adjust buy/sell price
5. Click **Save**
6. To create a new card, click **Add Card**
7. To delete, click **Delete** (with confirmation)

## Editing Synergies

1. Go to **Synergies** in the sidebar
2. Expand a tribe's accordion panel
3. Edit tier definitions:
   - **Threshold**: Number of tribe members needed (2, 4, or 6)
   - **Trigger**: When the effect fires (StartOfCombat, OnSell, etc.)
   - **Effect**: What happens (BuffAttack, BonusGold, Shield, etc.)
   - **Target**: Who it applies to (AllTribeMembers, AllFriendly, etc.)
   - **Value**: Effect magnitude
4. Click **Save**

## Tuning Balance

1. Go to **Balance** in the sidebar
2. Adjust settings in each section:
   - **Economy**: Buy cost, gold per turn, starting gold, max gold
   - **Pool**: How many copies of each tier exist in the shared pool
   - **Shop**: How many cards the shop offers at each tier
   - **Upgrades**: Cost to upgrade tavern to each tier
3. Click **Save**

## Editing Theme

1. Go to **Theme** in the sidebar
2. Change the game name, tribe colors (click the color swatch), or UI text
3. Click **Save**

## Publishing Changes

Changes you make are saved as **drafts** — they don't affect the live game until you publish.

### Quick Publish
1. Go to **Deploy** in the sidebar
2. Click **Publish to Live**
3. Wait for CloudFront invalidation to complete (usually 30-60 seconds)
4. Click the game link to verify

### Version Snapshots
For safer publishing with rollback:
1. Go to **Versions** in the sidebar
2. Click **Create Snapshot** and add a description
3. The snapshot saves the current live state
4. Click **Publish** on the snapshot to make it live
5. If something breaks, click **Rollback** on a previous version

## Best Practices

- **Always create a version snapshot** before making major changes
- **Test in the live game** after publishing to verify changes
- **Use descriptive version names** (e.g., "Buffed Swords tier 2, nerfed Cups healing")
- **Don't delete cards** that players might already have — it could cause errors
