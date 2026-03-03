# Upgrades & Card System — PONG ROGUE

## Rarity Tier System

### Tier Hierarchy

Defined in [`TIER_ORDER`](script.js:27) at line 27:

`common → uncommon → rare → epic → legendary → mythical → secret`

### Tier Visuals

Defined at [`script.js:23-26`](script.js:23):

| Tier | Color | Icon | Glow RGB |
|------|-------|------|----------|
| common | `#667788` | ◇ | 102,119,136 |
| uncommon | `#55aa77` | ◈ | 85,170,119 |
| rare | `#5588ff` | ★ | 85,136,255 |
| epic | `#ffaa44` | ✦ | 255,170,68 |
| legendary | `#ff5555` | ❂ | 255,85,85 |
| mythical | `#ff44ff` | ❈ | 255,68,255 |
| secret | `#00ffcc` | ❖ | 0,255,204 |

## Difficulty → Reward Mapping

### Guaranteed Reward Tier

[`DIFF_REWARD`](script.js:20) at line 20 determines what tier label is shown:

| Difficulty | Reward Label |
|------------|-------------|
| F, E | common |
| D, C | uncommon |
| B | rare |
| A | rare |
| S | epic |
| SS | legendary |
| SSS | secret |

### Available Tiers in Pool

[`DIFF_TIERS`](script.js:22) at line 22 determines which card tiers can appear:

| Difficulty | Available Tiers |
|------------|----------------|
| F | common |
| E | common, uncommon |
| D | common, uncommon |
| C | common, uncommon, rare |
| B | uncommon, rare |
| A | uncommon, rare, epic |
| S | rare, epic, legendary |
| SS | epic, legendary, mythical |
| SSS | legendary, mythical, secret |

## Card Selection Logic

### `getCardsForDiff(diff, count)` — [`script.js:177`](script.js:177)

Used by the upgrade screen. Selects `count` (typically 3) cards:

1. Filters `ALL_CARDS` to those whose tier is in `DIFF_TIERS[diff]`
2. Weights cards: highest available tier gets 4× weight, second highest 2×, rest 1×
3. Shuffles weighted pool, deduplicates by `id`
4. Falls back to unweighted pool if not enough unique cards

### `getCardsForTier(maxTier, count)` — [`script.js:158`](script.js:158)

Alternative selection: includes all cards up to `maxTier` in the tier hierarchy. Used less frequently.

## All Upgrade Cards

Defined in [`ALL_CARDS`](script.js:114) array at line 114. Each card has:

| Property | Type | Purpose |
|----------|------|---------|
| `id` | string | Unique identifier |
| `name` | string | Display name |
| `desc` | string | Description text |
| `tier` | string | Rarity tier |
| `type` | string | `"stat"` or `"ability"` |
| `fn` | function | Applied to `savedState` when chosen |

### Common Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `wider` | WIDE PADDLE | stat | `ph *= 1.3` |
| `swift` | QUICK FEET | stat | `pSpd *= 1.3` |
| `heavy` | HEAVY BALL | stat | `bs *= 1.2`, `aiMod *= 0.85` |

### Uncommon Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `life` | EXTRA LIFE | stat | `lives++` |
| `cd` | FAST CHARGE | stat | `cdMul *= 0.65` |
| `edge` | EDGE MASTER | ability | `edge = true` — outer paddle hits send extreme angles |
| `reach` | LONG REACH | stat | `horizMul *= 1.5` |
| `bounceback` | SNAPBACK | ability | `rico = true` — ball speeds up 15% on wall bounce |

### Rare Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `shield` | SAFETY NET | ability | `shields++` — auto-blocks first missed ball each wave |
| `freeze` | FREEZE FRAME | ability | `freeze = true` — enemy freezes 0.8s per player hit |
| `siphon` | SIPHON | ability | `siphon = true` — enemy AI -5% per player goal |
| `turbo` | ADRENALINE | stat | `pSpd *= 1.4`, `bs *= 1.15` |
| `fort` | IRON WILL | stat | `lives += 2`, `ph *= 1.1` |

### Epic Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `magnet` | MAGNETISM | ability | `magnet = true` — ball curves toward paddle when approaching |
| `double` | DOUBLE OR NOTHING | ability | `dblScore = true` — goals score 2 instead of 1 |
| `vampire` | VAMPIRE | ability | `vampire = true` — heal 1 life per goal (max 6) |
| `afterimg` | AFTERIMAGE | ability | `afterimage = true` — delayed ghost ball follows and scores separately |
| `shockwave` | TREMOR | ability | `shockwave = true` — hits push enemy paddle away from ball |
| `gravity2` | DEAD ZONE | ability | `singularity = true` — ball drifts toward enemy goal |

### Legendary Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `homing` | HUNTER BALL | ability | `homing = true` — ball curves toward gaps in enemy defense |
| `timewarp` | TIME WARP | ability | `timewarp = true` — ball slows near enemy, speeds near player |
| `multicast` | ECHO CAST | ability | `multicast = true` — paddle ability fires twice |
| `unstop` | JUGGERNAUT | stat | `lives += 3`, `ph *= 1.3`, `pSpd *= 1.3`, `bs *= 1.15` |
| `sec_mirror` | MIRROR MATCH | ability | `mirrorMatch = true` — enemy copies player Y with 0.4s delay |

### Mythical Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `godmode` | TRANSCENDENCE | ability | `transcend = true` — ball phases through enemy, every hit scores |
| `clone` | DOPPELGANGER | ability | `doppel = true` — phantom paddle mirrors player on right side |
| `immortal` | UNDYING | stat | `lives += 5`, `shields += 1` |
| `warpspd` | LIGHTSPEED | stat | `bs *= 1.5`, `pSpd *= 1.8`, `cdMul *= 0.5` |
| `sec_berserk` | BERSERKER | ability | `berserker = true` — each hit +8% ball speed, enemy score resets |
| `sec_master` | MASTER OF SKILL | ability | Strips ALL abilities. Ball 50% faster. Player hits = 3× speed. Each rally = permanent +20% base speed |

### Secret Tier

| ID | Name | Type | Effect |
|----|------|------|--------|
| `sec_storm` | STORM CALLER | ability | `stormCaller = true` — every hit fires a stun bolt |
| `sec_phantom` | PHANTOM STRIKE | ability | `phantomStrike = true` — ball teleports 20% closer to goal on hit |
| `sec_over` | OVERCHARGE | ability | `overcharge = true` — every 3rd consecutive hit, ball pierces |
| `sec_abs` | THE ABSOLUTE | ability | Enables almost everything: stormCaller, phantomStrike, berserker, overcharge, doppel, homing, multicast, magnet, dblScore, vampire, freeze, shockwave, edge, rico. `lives += 5`, huge stat boosts |

## State Properties Set by Upgrades

All stored in `savedState` and persisted via [`saveGame()`](script.js:279). These are the properties the upgrade `fn` callbacks can modify:

| Property | Type | Default | Category |
|----------|------|---------|----------|
| `ph` | number | `BASE_PAD_H * pad.hMul` | Paddle height |
| `pSpd` | number | 420 | Paddle speed |
| `bs` | number | 340 | Ball base speed |
| `cdMul` | number | 1 | Ability cooldown multiplier |
| `horizMul` | number | 1 | Horizontal move range multiplier |
| `lives` | number | 3 | Remaining lives |
| `shields` | number | 0 | Auto-block charges |
| `aiMod` | number | 1 | AI speed multiplier (lower = weaker enemy) |
| `edge` | boolean | false | Edge angle enhancement |
| `rico` | boolean | false | Wall bounce acceleration |
| `magnet` | boolean | false | Ball attraction |
| `dblScore` | boolean | false | Double scoring |
| `vampire` | boolean | false | Life steal |
| `freeze` | boolean | false | Freeze on hit |
| `afterimage` | boolean | false | Ghost ball on hit |
| `shockwave` | boolean | false | Push enemy on hit |
| `homing` | boolean | false | Ball seeks gaps |
| `timewarp` | boolean | false | Speed zones |
| `multicast` | boolean | false | Double ability use |
| `transcend` | boolean | false | Pierce all |
| `doppel` | boolean | false | Mirror paddle |
| `singularity` | boolean | false | Gravity drift |
| `siphon` | boolean | false | Degrade enemy AI |
| `stormCaller` | boolean | false | Auto-bolt on hit |
| `phantomStrike` | boolean | false | Teleport ball forward |
| `mirrorMatch` | boolean | false | Enemy mirrors player |
| `berserker` | boolean | false | Stacking speed |
| `overcharge` | boolean | false | Periodic pierce |
| `masterSkill` | boolean | false | Strip-and-boost mode |

## How to Add a New Upgrade Card

1. Add entry to [`ALL_CARDS`](script.js:114) with unique `id`, `tier`, `type`, `fn`
2. If it introduces a new boolean ability:
   - Add the property to [`newGame()`](script.js:214) defaults (lines 219–228)
   - Add to [`saveGame()`](script.js:279) serialization
   - Add gameplay logic in [`update()`](script.js:282)
   - Add VFX in [`draw()`](script.js:816)
   - Add HUD indicator in passives list (line 1216)
3. If stat-only: just ensure the `fn` modifies properties that already exist in `savedState`
