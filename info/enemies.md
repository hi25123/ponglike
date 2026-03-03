# Enemies & Enemy Abilities — PONG ROGUE

## Enemy Definitions

Defined in [`ENEMIES`](script.js:43) array at line 43. Each entry:

| Property | Type | Purpose |
|----------|------|---------|
| `id` | string | Unique key, used in `ENEMY_ABIL_MAP` |
| `name` | string | Display name |
| `tag` | string | Short visual indicator shown in HUD |
| `diff` | string | Difficulty rank: F through SSS |
| `mod` | function | Mutates the wave config object (adjusts AI stats, enables flags) |

## All Enemies by Tier

### F-Tier (Easiest)
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `basic` | ROOKIE | *(none)* | No changes (baseline) |
| `sleepy` | DROWSY | zz | `aiSpd *= 0.85` |

### E-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `wide` | WALL | = | `aiSpd *= 0.9` |
| `stubborn` | BRICK | [] | `aiSpd *= 0.85` |

### D-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `fast` | SPEEDSTER | >> | `aiSpd *= 1.3` |
| `jitter` | TWITCHER | ~~ | `aiSpd *= 1.15`, enables `jitter` flag |

### C-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `tricky` | TRICKSTER | ~ | Enables `trickAng` (wider return angles) |
| `mimic` | MIMIC | <> | `trickAng` + `aiSpd *= 1.1` |

### B-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `ghost` | PHANTOM | ? | Enables `ghost` (paddle flickers invisible) |
| `blinker` | BLINKER | *? | `ghost` + `aiSpd *= 1.1` |

### A-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `tank` | FORTRESS | ## | `aiSpd *= 0.85` |
| `jugg` | JUGGERNAUT | ### | `aiReact += 0.08` (capped at 0.95) |

### S-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `sniper` | SNIPER | + | `aiReact += 0.14` (capped at 0.97) |
| `hunter` | HUNTER | >+ | `aiReact += 0.1` (capped 0.96) + `aiSpd *= 1.2` |

### SS-Tier
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `accel` | CHAOS | !! | Enables `chaos` + `aiSpd *= 1.1` |
| `warden` | WARDEN | !# | `aiReact += 0.15` (capped 0.97) + `chaos` + `trickAng` |

### SSS-Tier (Boss-only)
| ID | Name | Tag | Modifiers |
|----|------|-----|-----------|
| `apex` | APEX | ☠ | `chaos` + `trickAng` + `aiReact += 0.2` (capped 0.98) + `aiSpd *= 1.3` |
| `void` | THE VOID | █ | `ghost` + `chaos` + `trickAng` + `aiReact += 0.18` (capped 0.98) + `aiSpd *= 1.25` |

## Modifier Flags

These flags are set on the wave config by enemy `mod()` functions:

| Flag | Effect |
|------|--------|
| `trickAng` | Enemy returns at wider angles (0.45 vs 0.35 spread) |
| `ghost` | Enemy paddle flickers between 15%–80% opacity |
| `chaos` | Ball base speed increases by 4% each enemy hit (capped at 600) |
| `jitter` | Adds `sin(t*18)*2.5` oscillation to enemy paddle position |

## Enemy Abilities

Defined in [`E_ABILS`](script.js:77) at line 77 and mapped via [`ENEMY_ABIL_MAP`](script.js:92) at line 92.

| Ability ID | Name | Icon | Cooldown | Duration | Description |
|------------|------|------|----------|----------|-------------|
| `none` | — | — | — | — | F/E tier enemies have no ability |
| `dash` | DASH | ↠ | 4s | 0.5s | Surges to ball at 3.5× AI speed |
| `spin` | SPIN RETURN | ↻ | 6s | 0s | Queues curve on next enemy hit. Ball curves mid-flight |
| `blink` | BLINK | ☇ | 5s | 0s | Instant teleport to ball Y position |
| `bulk` | BULK UP | ⬛ | 8s | 2.5s | Paddle grows to 1.4× height |
| `pull` | GRAVITY PULL | ⊙ | 7s | 2s | Decelerates ball + pulls toward enemy |
| `lightning` | LIGHTNING | ⚡ | 10s | 1.2s | Channels for 0.7s, stuns ball, launches at 2.5× speed |
| `voidpulse` | VOID PULSE | ⦿ | 9s | 0.3s | Reverses ball at 2× speed, pushes player paddle, freezes briefly |
| `rampage` | RAMPAGE | ☢ | 10s | 2.5s | Fires 2 phantom balls at player (can score if they pass) |
| `accel` | OVERDRIVE | » | 8s | 2s | Ball accelerates to 2× speed |
| `clone` | CLONE | ≡ | 12s | 3s | Spawns a phantom copy of paddle that also blocks |

## Enemy → Ability Mapping

[`ENEMY_ABIL_MAP`](script.js:92) at line 92:

| Enemy ID | Ability |
|----------|---------|
| `basic`, `sleepy`, `wide`, `stubborn` | `none` |
| `fast`, `jitter` | `dash` |
| `tricky`, `mimic` | `spin` |
| `ghost`, `blinker` | `blink` |
| `tank` | `bulk` |
| `jugg` | `accel` (OVERDRIVE) |
| `sniper` | `pull` |
| `hunter` | `lightning` |
| `accel` | `clone` |
| `warden` | `lightning` |
| `apex` | `voidpulse` |
| `void` | `rampage` |

## Enemy AI Decision Logic

Enemy ability casting is handled at [`script.js:688-807`](script.js:688). Each ability has specific trigger conditions:

| Ability | Trigger Condition |
|---------|-------------------|
| `dash` | Ball close (`bx > GW*0.55`) AND enemy far from ball (`> 0.8× paddleH` away) |
| `spin` | Ball heading right, close to paddle (`bx > GW*0.7`), ball within reach |
| `blink` | Ball heading right past 60%, enemy can't reach (`> 0.7× paddleH`) |
| `bulk` | Ball approaching, mid-range (`0.45–0.7 × GW`) |
| `pull` | Ball has passed enemy (`bx > GW*0.75`) |
| `lightning` | Ball safely heading away on player side (`bvx < 0`, `bx < GW*0.35`) |
| `voidpulse` | Ball close and about to pass (`bx > GW*0.7`, within 1.5× paddleH) |
| `rampage` | Ball heading toward player (`bvx < 0`, `bx < GW*0.4`) |
| `accel` | Ball heading toward player (`bvx < 0`, `bx < GW*0.5`) |
| `clone` | Ball approaching enemy (`bx > GW*0.4`) |

## Enemy Upgrades (Between Waves)

Defined in [`E_UPS`](script.js:104) at line 104. One random enemy upgrade is applied each wave after the upgrade card screen:

| ID | Name | Icon | Effect |
|----|------|------|--------|
| `ef` | FASTER | » | `aiSpd *= 1.2` |
| `es` | SMARTER | ◆ | `aiReact *= 1.15` (capped 0.96) |
| `eb` | BALL SPD+ | ● | `bs *= 1.08` |
| `ea` | AGGRO | ∠ | Enables `trickAng` |
| `eg` | GHOST | ? | Enables `ghost` on enemy config |

These accumulate in the `enemyUps` global array across waves and are reapplied each round via [`newGame()`](script.js:214) line 274.

## How to Add a New Enemy

1. Add entry to [`ENEMIES`](script.js:43) array with unique `id` and `diff` rank
2. Add ability mapping in [`ENEMY_ABIL_MAP`](script.js:92)
3. If new ability needed: add to [`E_ABILS`](script.js:77), add trigger logic in AI decision block (line 688+), add execution in cast block (line 740+)
4. Add any new ability VFX in [`draw()`](script.js:816) under enemy ability VFX section (lines 1082+)
5. Add any per-frame state updates in [`update()`](script.js:282)

## How to Add a New Enemy Ability

1. Add to [`E_ABILS`](script.js:77) object with `id`, `name`, `desc`, `icon`, `cd`, `dur`
2. Add state variables to [`newGame()`](script.js:214) 
3. Add trigger condition in AI decision switch at line 694
4. Add execution logic in cast switch at line 740
5. Add per-frame update logic in [`update()`](script.js:612) section
6. Add VFX in [`draw()`](script.js:1082) enemy ability VFX section
7. Map to enemies in [`ENEMY_ABIL_MAP`](script.js:92)
