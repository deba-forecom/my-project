# Visual Priority Plan — The Last Village

How to sequence the fixes in `visual_problems.md`, `quick_wins.md`, and `high_impact_upgrades.md` for the best perceived-quality gain per hour spent, without a full redesign.

## What should be changed first

**Phase 1 — Combat feedback (do this first, highest ROI):**
Hit-flash on enemies, a screen shake on core damage, and a bigger/longer death-particle burst. These are all quick wins (see `quick_wins.md` #1, #4, and the additional death-particle item), touch only the existing `update*`/`draw*` functions, and fix the problem that hurts moment-to-moment feel the most: combat currently has almost no impact feedback. Do this before anything else — it's cheap and it's the difference most playtesters will notice first.

**Phase 2 — Light and shadow on existing shapes:**
Radial gradients + drop shadows on every entity (quick wins #2 and #3). This is a mechanical, low-risk change (swap `fillStyle` calls, add one shadow-ellipse draw per entity) that immediately makes every object look "lit" instead of flat, without touching layout, gameplay, or silhouette shapes. Do this right after Phase 1 since it touches the same draw functions and is easy to batch into one pass.

**Phase 3 — The town environment:**
Build the procedural ground detail (paved ring, building silhouettes, dirt paths) described in `high_impact_upgrades.md` #1. This is the single biggest fix identified in the audit ("the town doesn't look like a town") but takes longer than the Phase 1–2 items, so it's sequenced after the cheaper wins are banked. It only touches `drawBackground()` and doesn't risk destabilizing gameplay code.

**Phase 4 — UI icon/badge polish:**
Coin icon in the HUD, "+N" floating gold text, icon-prefixed cost badges (`quick_wins.md` #5 and the badge item from `high_impact_upgrades.md` #5). Do this after the world-facing work because it's more visually isolated (DOM/badge code only) and lower urgency — the UI already reads as the most finished part of the game per this audit.

## What can wait until later

- **Layered entity silhouettes / shape-language redesign** (`high_impact_upgrades.md` #2) — valuable, but it's a bigger, riskier change (touches every draw function's actual geometry, not just fill style) and the return is incremental once gradients/shadows (Phase 2) and hit-feedback (Phase 1) are already in. Good candidate for a follow-up pass once the game is otherwise feature-complete.
- **Locomotion animation** (walk-wobble, player lean/recoil) — nice-to-have juice, but lower priority than combat-hit feedback since it's about idle/movement polish rather than the core shoot-and-defend loop.
- **Tower tier silhouette growth** — depends on the layered-silhouette work above being done first, so it's naturally a later-phase item.
- **Ambient background motion, time-of-day tint, weather** — genuinely nice "premium" touches but purely atmospheric; they add the least toward fixing the "looks unfinished" impression compared to Phases 1–3.
- **Sound design** — explicitly out of scope for this visual audit, but flagged in `high_impact_upgrades.md` as the natural next investment after visuals are addressed.

## Rationale for this order

The sequencing follows a simple rule: **fix feedback before fixing decoration, and fix decoration before fixing geometry.** Players notice unresponsive hits (Phase 1) faster than they notice flat shading (Phase 2), notice an empty world (Phase 3) faster than they notice plain badges (Phase 4), and all four of those are noticed faster than "the enemy silhouettes could be more distinct" (deferred items). This order also keeps early phases cheap and isolated (small diffs to existing functions) before committing to the larger, riskier structural changes later.
