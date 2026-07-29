# Visual Problems — The Last Village

Concrete, specific issues, grouped by area. Each entry names what's wrong and why it reads as low-end. File/function references point at `index.html`.

## Top 5 visual problems (highest severity)

1. **The town doesn't look like a town.** Outside the core's icon and the tower-spot circles, there is no architecture, no walls, no roads, no props — just an empty dark grid (`drawBackground()`, ~line 675). For a game literally named after defending a village, the world reads as an empty test level.
2. **No hit feedback on enemies.** Bullets and tower beams reduce an HP bar but the enemy sprite itself doesn't react (no flash, no knockback, no squash) — combat feels muted and it's hard to tell your shots are landing without watching a tiny bar.
3. **Every entity is a flat, unshaded circle/triangle.** Core, towers, forge, enemies, player, gold are all single-fill-color primitives with a stroke (`drawCore()`, `drawTowers()`, `drawEnemies()`, `drawPlayer()`, `drawGold()`, lines ~692–823). No gradient, no shadow, no light direction — nothing has visual weight or dimension.
4. **No impact moment when the core is hit.** Losing town-core HP is the central lose-condition stakes of the game, yet the only feedback is a brief red ring (`flash()` particle) — no screen shake, no color flash on the HUD bar, no sound cue. The most important negative event in the game currently has the least visual weight.
5. **Tower and player combat animation is nearly invisible.** Tower attacks are a 0.12s straight line (`updateTowers()`); player bullets are small dots with a soft glow. In a busy scene with several enemies, it's easy to lose track of who's shooting what.

## By category

### Art style
- Three different visual metaphors for interactive objects (emoji icon on core/forge, numeral on towers, plain circle on gold) don't cohere into one system.
- No silhouette variety — every unit type is "a circle of a different color/size," so at a glance runners and brutes are only distinguishable by color, not shape/read.
- Tower tiers are communicated only by a numeral (1–4) inside the circle (`drawTowers()`, ~line 758) — no structural growth (e.g., the tower doesn't get visibly bigger/more elaborate as it upgrades), so upgrades don't feel like progression, just a stat change.

### Color and lighting
- Flat fills throughout; only the forge and bullets use `shadowBlur`/glow. Inconsistent use of the effect makes the forge/bullets look like they're from a different, more finished pass than everything else.
- No cast shadows under player/enemies/towers to ground them on the play surface — objects feel like they're floating over the grid rather than standing on it.
- Core HP bar color-shifts (green → yellow → red) are a good idea but happen only on the thin DOM bar at the top of the screen — there's no matching in-world cue (e.g., the core itself glowing red when critical), so the danger state isn't visible where the action is happening.

### UI quality
- Cost badges (`badge()`, line 653) are plain text pills with no icon — "25g" is legible but generic; nothing distinguishes it as "premium game UI" versus a debug overlay.
- The HUD gold counter is a bare number with no coin icon, despite gold being the central resource of the game.
- No visual "can't afford" feedback beyond a dimmed badge — clicking an unaffordable object should probably shake or briefly redden the badge for clearer feedback (ties into animation gap below).

### Animation and visual feedback
- No hit-flash / hit-stop anywhere in the game.
- No screen shake on core damage or big kills.
- Death particle burst is small (6 dots, radius 3px, 0.35s life) and easy to miss.
- No idle/walk animation on player or enemies — pure linear translation, which reads as sliding rather than moving.
- No "+N gold" floating text on pickup, despite gold collection being one of the core feedback loops (per the design brief: "shoot enemies → collect gold").
- Tower fire and player fire look visually identical in weight/color family in a busy fight, making it hard to tell which damage source is doing the work.

### Backgrounds and environment
- The map is ~95% empty dark grid outside a single soft radial glow around the core.
- No path/road geometry showing where enemies come from and where they're headed — currently enemies just appear at a screen edge and beeline for the core with no environmental storytelling.
- No differentiation between "inside town" and "outside town" — no wall, fence, or boundary ring, which undercuts the "defend the town" fantasy since there's no visual town to defend.
- No parallax, vignette, or ambient motion (e.g., drifting dust/embers) to make the background feel alive rather than static.
