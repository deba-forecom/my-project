# Visual Summary — The Last Village

Scope: visual quality and presentation only. Gameplay/balance is out of scope for this audit. Findings are based on the running game (`index.html`) at its current MVP state — flat vector shapes, no external image/audio/font assets, single self-contained HTML file.

## 1. Overall visual impression

The game reads as a **functional wireframe / grey-box prototype**, not as a finished product. Every gameplay element (player, enemies, towers, core, forge, gold, projectiles) is a flat, single-color circle or triangle with a solid fill and a stroke outline. There is no shading, no texture, no depth cue beyond a couple of glow/shadow effects on the forge and projectiles. Nothing in the world looks "built" — the "town" is legible only through UI badges and icon glyphs (⚑, ⚒), not through actual town architecture.

That said, the game is **readable and internally consistent** — every clickable object uses the same rounded-pill cost badge, colors map consistently to meaning (blue = tower, orange = forge/danger, gold = currency, green = player/health-good, red = health-bad), and the HUD/overlay typography (from the shared sibling-game convention) already looks intentional and reasonably premium. The gap is entirely in the **game-world rendering**, not the UI chrome.

**What already looks good:**
- Start/Win/Lose overlay cards — consistent glowing-gold typography, good spacing, matches the "premium arcade" tone the project is going for.
- HUD layout (gold/timer/core bar) — clean, minimal, legible at a glance.
- Cost badges (`badge()` in `index.html:653`) — one visual language for "this is interactive and costs gold," used consistently across core/forge/towers.
- Tower range ring on hover — a nice, cheap piece of feedback that's already in place.

**What should be improved first:** the game world itself (background, entities) — see Section 6 and the priority plan. This is where "cheap/placeholder" reads strongest, and where the biggest perceived-quality jump is available without a redesign.

## 2. Art style

There currently isn't a defined art style beyond "flat colored primitives on a dark grid." Circles-with-emoji (core = ⚑, forge = ⚒) and circles-with-a-number (towers) don't cohere into a single visual language — they read as three different shorthand systems bolted together rather than one town rendered in one style.

All entities share the same weight of stroke and flat fill, which is visually consistent but also visually *undifferentiated* — a runner enemy, the player, and a tower spot are all "a circle," distinguished only by color and size. Nothing is stretched, blurry, or mismatched in resolution (because nothing is a raster asset), so there's no "quality mismatch" problem in the traditional sense — the problem is uniform low elaboration everywhere.

**Recommended direction:** stay 100% vector/procedural (no external assets — this project intentionally has zero network dependencies and should stay that way), but move from "flat primitive" to "layered primitive with light/shadow and silhouette variety" — i.e., build every entity out of 2–4 stacked shapes with a gradient and a shadow instead of one flat circle. This is a style, not an asset pipeline, and is achievable entirely in Canvas 2D.

## 3. Color and lighting

The palette itself (gold `#ffd166`, blue `#5eb1ff`, orange `#ff9a4d`, green `#4ade80`, red `#ff5470` on a `#141b28` night background) is attractive and has good hue separation — this part is already solid and matches the sibling reference games' palette conventions.

The problem is **lighting, not hue**: almost everything is drawn with a flat fill and a single-width stroke (`ctx.fillStyle`, `ctx.stroke()`), so nothing has gradient shading, rim light, or a cast shadow. The only elements with any glow/shadow treatment are the forge (`shadowBlur` in `drawForge()`, `index.html:709`) and the player bullets (`drawProjectiles()`, `index.html:802`). Everything else — core, towers, enemies, player, gold — sits flush on the background with no visual weight or grounding.

Contrast is functionally fine for readability (light objects on a dark ground), but there's no focal hierarchy beyond size: the eye has no lighting cue telling it "the core is the most important object on screen" beyond its central position and the ambient radial glow (`drawBackground()`, `index.html:675`).

## 4. UI quality

The DOM-based HUD (gold count, timer, core HP bar, legend) is clean, uses consistent type scale and letter-spacing, and is the strongest visual layer in the game — it doesn't need a redesign, only refinement (see quick wins: a coin glyph next to the gold number, small transition/pulse on value change).

The in-canvas badges are good UI design (one badge style, consistently applied) but visually plain: flat dark pill, thin gold border, no shadow/depth, no icon — just text like "25g." A small coin icon and a subtle drop shadow would make these read as "game UI" rather than "debug label."

Overlay cards (start/win/lose) are the most polished screens in the game already — good hierarchy, good use of glow on headlines, clear CTA buttons. No major complaints here.

## 5. Animation and visual feedback

This is the weakest area outside of raw asset quality. Currently:
- Enemies have **no hit-flash** when shot — only a shrinking HP bar communicates damage, which is easy to miss in the heat of combat.
- There is **no screen shake / hit-stop** anywhere, including when the town core takes damage — a moment that should feel impactful currently feels identical to normal idle rendering.
- Death particles (`killEnemy()`, `index.html` update section) are a 6-dot radial burst lasting 0.35s — functional but very small and quick; it reads as a flicker, not a "kill."
- The player and enemies have **zero locomotion animation** — they slide across the screen with no walk-cycle, bob, squash/stretch, or facing change (enemies don't rotate/flip toward their movement direction at all).
- Tower fire is a single straight `beam` line for 0.12s (`updateTowers()`) — very subtle, easy to miss, no muzzle flash or impact spark at the target.
- Gold pickup has motion (magnet pull) but no "pop"/scale-in on collection and no floating "+N" text, so the reward moment is nearly silent visually.

Overall the game currently *works* but doesn't yet *feel* — there's a real opportunity to add "juice" (small, cheap animation/feedback effects) for a large perceived-quality gain.

## 6. Backgrounds and environment

The world background is a flat navy fill with a faint 40px grid and one soft radial gold gradient centered on the core (`drawBackground()`, `index.html:675`). Outside the ~220px glow radius, the map is visually empty — no ground texture, no path/road leading from the spawn edges to the core, no environmental props (rubble, fences, trees, rocks), and no visual distinction between "inside the town" and "the wilderness enemies are marching in from."

This is the single biggest driver of the "cheap/unfinished" impression: a defense game about protecting a *town* currently shows no town — just a core icon and four tower-spot circles floating on an empty grid. Even lightweight procedural detail (a paved ring around the core, simple building silhouettes near the tower spots, worn dirt paths from each spawn edge toward the center) would do more for perceived production value than almost any other single change.
