# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"The Last Village" — a top-down town-defense shooter. The entire game is one self-contained file, `index.html`: inline `<style>`, inline HTML, inline `<script>`. No `package.json`, no build step, no external dependencies (no CDN scripts, no fonts, no images/audio files). It runs by opening `index.html` directly in a browser (`file://`) — this is intentional, not an oversight, and should be preserved. Do not introduce a bundler, module imports, npm dependencies, or split the game into multiple JS/CSS files.

## Commands

There is no build, lint, or test command — there is no tooling in this project at all. To run the game, open `index.html` directly in Chrome/Edge/Firefox (double-click, or drag into a browser tab). A local static server (`npx serve`, VS Code Live Server, etc.) also works but isn't required since the page makes no network requests.

There is no automated test suite. Verify any change by opening the page and playing through the affected mechanic in a browser. Useful things to check after a change:
- Start a game, move (WASD), aim/shoot (mouse), and confirm no console errors.
- Click an empty tower spot / a built tower / the forge / the town core and confirm gold deducts correctly and the action applies (build, upgrade, or repair) — and confirm clicking one of these objects does *not* also fire a shot (see click-priority below).
- Let an enemy reach the core and confirm core HP drops and the lose overlay triggers at 0 HP.
- For balance changes, `SESSION_LENGTH` (currently 600s) can be temporarily lowered to speed up manual playtesting of the difficulty ramp and win condition.

## Architecture

Everything lives in the single `<script>` block in `index.html`, organized top-to-bottom into clearly commented sections:

1. **Config / balancing** — all tunable constants and data tables: canvas size, `SESSION_LENGTH`, fixed world-layout positions (`CORE_START`, `FORGE_POS`, `TOWER_SPOTS_DEF`, `PLAYER_START`), `WEAPON_TIERS`, `TOWER_TIERS`, `ENEMY_BASE` stats, and the difficulty-scaling functions (`spawnInterval`, `bruteChance`, `hpScale`, `speedScale`, `dmgScale`), which are pure functions of `elapsed` (continuous ramp, not discrete waves).
2. **Setup** — DOM element refs, and the mutable game state as flat module-level `let`s (`state`, `elapsed`, `gold`, `core`, `forge`, `towerSpots`, `player`, `enemies`, `projectiles`, `goldDrops`, `particles`, `shake`, `mouse`, `keys`). `freshState()` initializes/resets all of them together and is the single source of truth for "what resets on a new game"; `resetGame()` calls it and flips overlays. There is no nested state object — this flat-variable style is deliberate for a game this size, mirroring the sibling reference games elsewhere in the parent repo.
3. **Input** — keyboard (`keys` map), mouse position (world-space, via `toWorld()` which accounts for the canvas being CSS-scaled to a different size than its logical 960×640 resolution), and `handleWorldClick`. Click handling has a strict **priority order**: on `mousedown`, `handleWorldClick` hit-tests the town core, then the forge, then each tower spot; the first hit consumes the click entirely (build/upgrade/repair, or a no-op if unaffordable) and returns `true`. Only if nothing was hit does the click start player fire. This is what lets "click a building" and "click to shoot" share the same mouse button — preserve this ordering when adding new clickable world objects.
4. **Update** — one `update(dt)` per frame, called only while `state === 'playing'`, running in this order: player → spawner → enemies → projectiles → towers → gold pickups → particles/shake. Entities are plain objects in flat arrays (no classes); a "dead" entity is flagged with `.dead = true` mid-loop and swept with `.filter()` at the end of its update function, so other code in the same frame can still see it before removal.
5. **Render** — one `render()` per frame, always runs (even when not `'playing'`, so overlays sit over a frozen last frame). Canvas draws the game world (background/roads/houses → gold → core → towers → forge → enemies → player → projectiles → particles), all wrapped in a `ctx.save()`/`ctx.translate()`/`ctx.restore()` for screen shake. The DOM HUD (gold, timer, core HP bar) and the start/win/lose `.overlay` panels are separate absolutely-positioned elements updated from `render()`, not drawn on the canvas.
6. **Main loop** — `requestAnimationFrame` with delta-time clamped to 0.05s (`loop()` → `update(dt)` + `render()`).

### Shared logic to reuse, not duplicate

- `nextTowerTier(spot)` / `applyTowerTier(spot, tier)` and `nextWeaponTier()` / `applyWeaponTier(tier)` are the single source of truth for "what does the next upgrade cost and what does it grant." Both the purchase logic (`handleWorldClick`) and the badge/cost display logic (`drawTowers`, `drawForge`) call these — don't re-derive tier/cost logic independently in a new call site.
- `circlePath(x, y, r)`, `radialGrad(x, y, r, light, dark)`, and `shadowEllipse(x, y, rx, ry)` are the shared canvas-drawing primitives used by every entity's draw function for the lit/shaded/grounded look. New entities should use these rather than raw `ctx.arc(...)` calls.
- `badge(text, x, y, affordable)` draws the coin-icon cost pill used above every clickable world object (core repair, forge upgrade, tower build/upgrade) — reuse it for any new purchasable action rather than drawing a new label style.

### Other things worth knowing

- The player has no HP — only the town core does. Enemies never damage the player directly, only the core (via a contact-attack tick, `CORE_ATTACK_TICK` seconds apart, while adjacent to it).
- Towers deal damage via instant hitscan (no projectile entities of their own) — only the player's own shots are travelling projectiles.
- `audit/` contains a visual-polish audit (`visual_summary.md`, `visual_problems.md`, `quick_wins.md`, `high_impact_upgrades.md`, `visual_priority_plan.md`) written against an earlier state of the game. Some of its suggestions have since been implemented (hit-flash, screen shake, gradient/shadow shading, the procedural town background, coin-icon badges); check current `index.html` before assuming a listed item is still outstanding.
