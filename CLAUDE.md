# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of browser-based games — single HTML files with no build tools, no dependencies, and no server required. Each file is fully self-contained (HTML + CSS + JS in one file), opened directly in a browser.

## Running the Games

```bash
open tictactoe.html
open treasure_hunter.html
```

No build, no install, no dev server. Changes take effect on browser refresh.

## Git Workflow

After every meaningful change, commit and push immediately:

```bash
git add <file>
git commit -m "message"
git push
```

Remote is `origin` → `https://github.com/Husse06/claude-games` (main branch).

**Commit message rules:**
- Describe *what changed and why* — e.g. `add ambient sparkle particles to treasure hunter`, `fix miner cost scaling formula`
- Never mention Claude, AI, or the assistant in commit messages
- Never reference the conversation, the task, or phrases like "as requested", "per instructions", "implement plan"
- No `Co-Authored-By` trailers

## Code Architecture

### Shared conventions across both files
- **Dark navy palette** — `#0d1117` background, `#161b22` surfaces, `#f5a623` gold accent. New games should match this aesthetic.
- All styling is in a `<style>` block at the top; all logic in a `<script>` block at the bottom.
- CSS custom properties (`--bg`, `--surface`, `--accent`, etc.) defined on `:root` for theming consistency.

### `tictactoe.html`
- Pure vanilla JS, no state management library.
- `board` is a flat `Array(9)` of `''|'X'|'O'`.
- AI uses unoptimised **minimax** (works fine for 3×3, do not use for larger boards).
- Scores persist in memory only (reset on page reload).

### `treasure_hunter.html`
- Single global `gs` object holds all game state; mutate it directly then call `renderAll()` (or `renderStats()` alone during the 100ms tick for performance).
- **Tick loop** runs every 100ms via `setInterval(tick, 100)` — accumulates fractional XP in `xpAccum` to avoid float drift.
- **Auto-save** every 10 s to `localStorage['treasureHunterSave']`; loaded on init with offline earnings (capped 8 h).
- Key formulas:
  - `xpRequired(lvl) = floor(lvl × 100 × 1.15^lvl)`
  - `globalMultiplier = 1 + Σ(accessory.bonus for owned accessories)`
  - Miner cost scaling: `baseCost × 1.15^owned`
  - Treasure roll: weighted random filtered by `unlockLvl ≤ gs.level`
- `renderShop()` is expensive — only call it on purchases or tab switches, not in the tick.
- Particle/float animations are DOM-spawned divs removed on `animationend`.
- Ambient sparkles are spawned every 180 ms into `#sparkle-layer` (fixed, pointer-events none, z-index 0).
