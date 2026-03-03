# Project Overview — PONG ROGUE

## What Is This

A **roguelite pong game** — single-player pong with RPG progression. The player picks a paddle class, fights through 12 waves of increasingly difficult AI opponents, and collects upgrades between rounds. Built as a single-page web app with no frameworks.

## Tech Stack

- **Pure vanilla JS** — one file: [`script.js`](script.js) (~1526 lines)
- **Single HTML page** — [`pong-rogue__2_ (5).html`](pong-rogue__2_ (5).html) (artifact name, should be renamed to `index.html`)
- **CSS** — [`styles.css`](styles.css) for menu/card UI, CRT scanline overlay
- **Canvas 2D** — all gameplay rendering via `<canvas id="cv">`
- **Web Audio API** — procedural sound effects (no audio files)
- **Biome** — configured for linting/formatting (tabs, double quotes)
- **No build step** — open the HTML file directly in a browser

## File Structure

| File | Purpose |
|------|---------|
| `pong-rogue__2_ (5).html` | Entry point. Contains all screen layouts (menu, cards, opponent select, game over, victory, dev mode). Should be renamed to `index.html`. |
| `script.js` | All game logic: audio, constants, data, game state, update loop, draw loop, UI builders, input, dev mode |
| `styles.css` | Dark CRT aesthetic. Paddle cards, upgrade cards, opponent cards, buttons, screen transitions |
| `package.json` | Only dependency: `@biomejs/biome` for formatting/linting |
| `biome.json` | Biome config: tabs, double quotes, recommended rules |

## Architecture Pattern

Everything lives in a **single global scope** in `script.js`. There are no modules, classes, or imports. The code is organized into labeled sections:

1. **Audio** (lines 1–4) — Web Audio oscillator-based SFX
2. **Constants** (lines 6–13) — game dimensions, physics values, utilities
3. **Data Tables** (lines 15–156) — difficulty ranks, palettes, paddles, enemies, abilities, upgrades
4. **Card/Upgrade Logic** (lines 158–205) — card pool filtering, wave config generation
5. **Game State** (lines 207–277) — state init, save/load, `newGame()` factory
6. **Update Loop** (lines 282–811) — physics, collision, AI, ability execution per-frame
7. **Draw Loop** (lines 813–1247) — rendering, VFX, HUD
8. **Screen Management** (lines 1249–1326) — UI transitions, upgrade screen builder
9. **Opponent Selection** (lines 1330–1444) — generates 3 difficulty choices per wave
10. **Input** (lines 1447–1453) — keyboard and click handlers
11. **Dev Mode** (lines 1455–1520) — test any paddle vs any enemy
12. **Game Loop** (lines 1522–1526) — `requestAnimationFrame` loop, canvas resize

## Key Global Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `g` | Object \| null | The active game state. `null` when on a menu screen. |
| `curScreen` | string \| null | Current visible screen ID, or `null` when gameplay is active |
| `padId` | string | Selected paddle class ID |
| `wave` | number | Current wave (1–12) |
| `savedState` | Object \| null | Persistent player stats/abilities carried between waves |
| `enemyUps` | Array | Accumulated enemy upgrades across waves |
| `keysDown` | Object | Currently pressed keys |
| `chosenOppCfg` | Object \| null | The opponent config chosen from the selection screen |

## Known Issues / Quirks

- HTML file has artifact name from download: `pong-rogue__2_ (5).html` — should be renamed to `index.html`
- All code is heavily minified-style (single-letter vars, dense one-liners) despite being source code
- `g.phaseNext` is declared twice in `newGame()` (line 235 and 243) — second declaration shadows the first
- The `sp` variable is used for both "speed" and "scanline pattern" in different scopes (lines 370 vs 1241)
- No error boundaries — a crash in update/draw would halt the game loop
