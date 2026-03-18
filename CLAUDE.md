# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of browser-based games — single HTML files with no build tools, no dependencies, and no server required. Each file is fully self-contained (HTML + CSS + JS in one file), opened directly in a browser.

## Running the Games

```bash
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

### `treasure_hunter.html` (Mineral Depths — fully overhauled)
- **Theme**: Deep Cave palette — `#0a0a0f` bg, neon purple `#9b59ff`, glowing green `#39ff14`, gold `#f5a623`.
- **Layout**: CSS Grid 3-column (310px | 1fr | 295px); glassmorphism panels (`backdrop-filter: blur(14px)`).
- Single global `gs` object holds all game state; mutate it directly then call `renderAll()` (or `renderStats()` → alias for `renderLeft()` during the 100ms tick).
- **Tick loop** runs every 100ms via `setInterval(tick, 100)` — accumulates fractional XP in `xpAccum` to avoid float drift.
- **Auto-save** every 10 s to `localStorage['treasureHunterSave']`; loaded on init with offline earnings (capped 8 h). Old-format saves (with `coins` key) are auto-migrated via `migrateOldSave()`.
- **Currencies**: `gs.minerals` (primary, earned by clicking/passive), `gs.gems` (premium, from geode cracks).
- **The Forge**: Two upgrades (`clickPower`, `passiveRate`), each 5 levels (up to 32× multiplier). Costs/unlock levels defined in `FORGE_UPGRADES`. Only call `renderMiddle()` on purchases or tab switches, not in the tick.
- **Geodes**: Three tiers (common/rare/epic), cracked via `crackGeode(tier)` using `weightedRoll()` + `adjustQuality()` (Geomancer's Eye effect). Bought from Prospector shop with minerals.
- **Artifacts**: 5 items (`dwarvenGear`, `luckyCoin`, `geomancersEye`, `dragonClaw`, `ancientCompass`), obtainable only from geode cracks. Effects computed in `computeMults()`.
- **Pets**: 3 pets costing gems; orbit the vein via CSS `rotate(θ) translateX(106px) rotate(-θ)` double-rotation trick. Bonuses included in `computeMults()`.
- Key formulas:
  - `xpRequired(lvl) = floor(lvl × 100 × 1.15^lvl)`
  - `clickYield() = BASE_CLICK(5) × FORGE_UPGRADES.clickPower.multipliers[level] × mults.click`
  - `passiveYield() = BASE_PASSIVE(0.5) × FORGE_UPGRADES.passiveRate.multipliers[level] × mults.passive + (autoMiner ? 2 : 0)`
  - Geode drop per click: base 5%, +10% with Treasure Hound pet
- Particle/float animations are DOM-spawned divs removed on `animationend`.
- Ambient sparkles are spawned every 180 ms into `#sparkle-layer` (fixed, pointer-events none, z-index 0).
