# Paddles & Player Abilities — PONG ROGUE

## Paddle Definitions

Defined in [`PADDLES`](script.js:33) array at line 33. Each paddle has:

| Property | Type | Purpose |
|----------|------|---------|
| `id` | string | Internal key, also used as color palette key in `PCOL` |
| `name` | string | Display name |
| `hMul` | number | Paddle height multiplier (applied to `BASE_PAD_H` = 72) |
| `desc` | string | Flavor text shown on selection screen |
| `abil` | string | Ability name displayed in HUD |
| `akey` | string | Key to activate (always `"Q"`, empty for Standard) |
| `ainfo` | string | Short ability description |
| `cd` | number | Base cooldown in seconds |

## All Paddles

| ID | Name | Height Mult | Cooldown | Ability | Effect |
|----|------|------------|----------|---------|--------|
| `classic` | STANDARD | 1.1× (79px) | 999 (none) | NONE | No ability. Slightly larger paddle. |
| `oracle` | ORACLE | 1.0× (72px) | 12s | FORESIGHT | Shows full ball trajectory for 8 seconds |
| `inferno` | PHANTOM | 0.95× (68px) | 6s | PHASE | Next hit pierces through enemy paddle |
| `frost` | GLACIER | 1.05× (76px) | 8s | BLIZZARD | Freezes enemy + slows ball in enemy half for 1.8s |
| `storm` | STORM | 0.9× (65px) | 6s | THUNDER | Next hit spawns a stun bolt that freezes enemy for 0.7s |
| `voidp` | VOID | 1.0× (72px) | 6s | GRAVITY WELL | Places a gravity well in ball path for 3s, warps trajectory |

## Color Palettes

Defined in [`PCOL`](script.js:30) at line 30. Each paddle ID maps to:

| Paddle ID | Color (hex) | RGB tuple | Used for |
|-----------|-------------|-----------|----------|
| `classic` | `#ffffff` | `[1,1,1]` | White — ball, trail, sparks |
| `oracle` | `#44ddff` | `[0.27,0.87,1]` | Cyan |
| `inferno` | `#cc88ff` | `[0.8,0.53,1]` | Purple |
| `frost` | `#88ccff` | `[0.53,0.8,1]` | Light blue |
| `storm` | `#ffee44` | `[1,0.93,0.27]` | Yellow |
| `voidp` | `#ff44aa` | `[1,0.27,0.67]` | Pink |

Each palette object: `{p: hexColor, g: rgbaPrefix, t: [r,g,b]}`
- `p` — solid color for fills
- `g` — partial `rgba(` string, close with `alpha)` when using
- `t` — normalized RGB tuple for spark colors

## Ability Execution

All abilities triggered by [`doAbility()`](script.js:1255) at line 1255.

### FORESIGHT (Oracle)
- Sets `g.foresightT = 8` — timer for 8 seconds
- In [`draw()`](script.js:1005): renders predicted ball path by simulating ball physics forward (600 steps)
- Shows impact marker (crosshair) at predicted landing point
- Eye icon displayed above paddle

### PHASE (Phantom)
- Sets `g.phaseNext = true` — flag consumed on next player hit
- On hit (line 419): sets `piercing = true`, ball passes through enemy paddle
- VFX: purple ring around ball while active

### BLIZZARD (Glacier)
- Sets `g.blizzardT = 1.8` and `g.freezeT = 1.8`
- `freezeT > 0`: enemy paddle cannot move (line 576)
- `blizzardT > 0`: ball speed in enemy half × 0.3 (line 367)
- VFX: ice overlay on right half, snowflake particles

### THUNDER (Storm)
- Sets `g.thunderNext = true` — flag consumed on next player hit
- On hit (line 459): spawns a bolt projectile at ball position
- Bolt moves right with zigzag pattern, hits enemy paddle → freezes for 0.7s
- Bolts fizzle at screen edges — **never score directly**
- VFX: yellow electric arcs around paddle while charged, bolt trails

### GRAVITY WELL (Void)
- Places a well 150px ahead of ball in its trajectory
- `g.gravWell = {x, y}`, `g.gravWellT = 3`
- Physics (lines 308–327):
  - Ball heading toward player: strong pull toward well center, pushes right when passing through
  - Ball heading toward enemy: perpendicular deflection to curve around enemy
  - Anti-stall: forces horizontal when `bvy/bvx` ratio > 3
- VFX: swirling arcs, pulsing center, tether line to ball

## Multicast Interaction

If `g.multicast` is true (from ECHO CAST upgrade), ability fires **twice** with 300ms delay. The cooldown resets to 0 before the second cast. Implemented at line 1274.

## Game State Properties for Abilities

These are set in [`newGame()`](script.js:214) and persisted via [`saveGame()`](script.js:279):

| State Key | Type | Default | Used By |
|-----------|------|---------|---------|
| `foresightT` | number | 0 | Oracle: time remaining |
| `phaseNext` | boolean | false | Phantom: next hit pierces |
| `blizzardT` | number | 0 | Glacier: freeze zone timer |
| `thunderNext` | boolean | false | Storm: next hit spawns bolt |
| `bolts` | Array | [] | Storm: active bolt projectiles |
| `gravWell` | Object\|null | null | Void: well position `{x,y}` |
| `gravWellT` | number | 0 | Void: well duration remaining |
| `abCD` | number | 0 | Cooldown timer (decrements each frame) |
| `cdMul` | number | 1 | Cooldown multiplier (from upgrades) |

## How to Add a New Paddle

1. Add entry to [`PADDLES`](script.js:33) array with unique `id`
2. Add color palette to [`PCOL`](script.js:30) matching the `id`
3. Add ability logic case to [`doAbility()`](script.js:1255) switch statement
4. Add any new state properties to [`newGame()`](script.js:214) and [`saveGame()`](script.js:279)
5. Add VFX in [`draw()`](script.js:816) under the ability VFX section (lines 894+)
6. Add any per-frame update logic in [`update()`](script.js:282)
