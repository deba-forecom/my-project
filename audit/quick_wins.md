# Quick Wins — The Last Village

Low-effort, high-visual-return changes. All are achievable with Canvas 2D API calls or small CSS tweaks — no new assets, no new systems, no external dependencies (keeps the single-file/offline-first convention intact). Each should take under an hour to implement individually.

## Top 5 quick wins

1. **Add a hit-flash to enemies on damage.** In the projectile/tower-hit collision handlers, set a short `hitFlash` timer (e.g. 0.08s) on the enemy and draw it with a white/lightened overlay (or `ctx.globalCompositeOperation = 'lighter'` flash) while the timer is active, inside `drawEnemies()` (~line 773). This single change will make combat feel dramatically more responsive.
2. **Add radial gradients to every filled circle** (core, towers, forge, enemies, gold) instead of flat `ctx.fillStyle`. A 2-stop `createRadialGradient` (lighter at the upper-left, darker at the edge) on each entity gives an instant "lit sphere" look with no new drawing logic — just swap the fill style in `drawCore()`, `drawTowers()`, `drawForge()`, `drawEnemies()`, `drawGold()`.
3. **Add soft drop shadows under every entity.** A single `ctx.ellipse` filled with `rgba(0,0,0,0.35)` drawn slightly below each entity (player, enemies, towers, core, forge) before the entity itself, in the existing draw order, grounds everything on the play surface immediately.
4. **Add a screen shake on core damage.** In `updateEnemies()` where `core.hp` is reduced on an attack tick, set a `shakeTime`/`shakeMag` global; in `render()`, apply `ctx.translate(randomOffset)` for the shake duration before drawing. A few lines of code for a big perceived-impact gain on the game's central lose-condition stakes.
5. **Add a floating "+N" gold text and coin icon.** On pickup (`updateGold()`), push a short-lived text particle ("+3") that floats upward and fades; and swap the bare `goldValue` HUD number for a small inline coin glyph (e.g. a CSS-drawn circle or a `●` character in `--gold-400`) next to the number. Reinforces the core "collect gold" feedback loop the design brief calls out explicitly.

## Additional quick wins worth doing in the same pass

- **Bigger, longer death-particle burst** — raise from 6 to ~12 particles and 0.35s to ~0.5s life; add slight gravity/drag so it reads as a "pop" rather than a flicker.
- **Color-tint the core in-world when critical** — draw the core's stroke/fill in red once `core.hp / core.maxHp < 0.25`, mirroring the DOM HP bar's color logic, so danger is visible where the action is happening, not just in the corner HUD.
- **Icon on cost badges** — prefix each `badge()` string with a small coin glyph so "25g" reads visually as currency, not a debug label.
- **Muzzle flash on player fire** — a tiny 2–3 frame bright dot at the gun tip when firing, cheap but makes shooting feel more immediate.
- **Tower beam upgrade** — widen/brighten the beam and add a small impact spark at the target end (a few radiating lines for 1–2 frames) so tower hits are legible in a crowded fight.
- **Vignette overlay on the canvas** — a subtle `radial-gradient` CSS layer (or a canvas draw pass) darkening the corners of the `.stage` adds atmosphere for near-zero cost and is a common "premium feel" trick.
- **Pulse the "can't afford" case** — when a click on a world object is rejected for insufficient gold, briefly flash the badge red instead of doing nothing, closing a current feedback gap.
