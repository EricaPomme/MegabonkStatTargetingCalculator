# Megabonk Stat Targeting Calculator

A single-file HTML/JS companion tool that helps a player decide which stat buffs (from items, shrines, etc.) to prioritize based on their currently-equipped weapons.

## Quick start

Open `index.html` in any modern browser. No build step, no dependencies, no server.

## What it does

Given up to 4 weapons, the tool:

1. **Tallies each stat** — counts how many of the equipped weapons benefit from it
2. **Splits stats into buckets** by commonality:
   - **Common Stats** — 2+ weapons benefit (the strongest targets)
   - **Situational Stats** — exactly 1 weapon benefits (niche)
   - **Dead Stats** — no weapon benefits (don't bother)
3. **Lists complementary weapons** — every other weapon in the game, ranked by the total weight of its benefiting stats, excluding dead stats
4. **Applies user weights** — each stat can be weighted 0.0+ (default 1.0); higher weights boost a stat's ranking in all three places it appears

## Data model

Two hardcoded JS objects drive the entire tool:

### `ALL_STATS`

The canonical list of every stat a weapon can benefit from. Used as iteration order for all stat displays.

```
"Attack Speed", "Crit Chance", "Crit Damage", "Damage", "Duration",
"Knockback", "Projectile Bounces", "Projectile Count",
"Projectile Speed", "Size"
```

### `WEAPONS`

Lookup table mapping each weapon name to the array of stats it benefits from. Currently 28 weapons. Add a new weapon by appending one entry.

**Important game-modeling note:** Weapons do not *add* stats. They *benefit from* stats provided by items and buffs. The tool models this correctly — picking up a stat buff benefits whichever weapons are in your loadout.

## UI structure

Three-panel layout with sticky side panels on desktop, stacked vertically below 960px viewport.

```
┌──────────────┬─────────────────────────────┬──────────────┐
│ WEAPONS      │                             │ STAT WEIGHTS │
│              │   Common Stats              │              │
│ [Slot 1 ▼]   │   ─────────────             │ Atk Spd  1.0 │
│ [Slot 2 ▼]   │   ...                       │ Crit Ch 1.0  │
│ [Slot 3 ▼]   │                             │ ...          │
│ [Slot 4 ▼]   │   Situational Stats         │              │
│              │   ...                       │ [Reset all]  │
│ [Reset]      │                             │              │
│              │   Dead Stats                │              │
│              │   ...                       │              │
│              │                             │              │
│              │   Complementary Weapons     │              │
│              │   ...                       │              │
└──────────────┴─────────────────────────────┴──────────────┘
   240px sticky    flexible (scrolls)         260px sticky
```

### Why three panels?

- **Weapons (left)** — the input, always in reach while reviewing results
- **Middle** — the actual answers, takes most of the space, scrolls naturally as it grows
- **Weights (right)** — a tuning surface, can be tweaked without scrolling

Sticky positioning (`position: sticky; top: 2rem`) keeps both side panels visible while the middle scrolls.

## Algorithm details

### Common / Situational / Dead bucketing

For each stat `s`:

| Bucket | Condition | Sort |
|---|---|---|
| **Common** | `count > 1 AND weight > 0` | `count * weight` DESC, α |
| **Situational** | `count == 1 AND weight > 0` | `count * weight` DESC, α |
| **Dead** | `count == 0 OR weight == 0` | α |

`count` is the number of selected weapons that list the stat. `weight` defaults to 1.0; setting it to 0 opts the stat out (moves to Dead regardless of count).

The middle-ground **Situational** bucket was added later because single-weapon stats are a real category — useful info, but not the same priority as 2+ weapon stats.

### Complementary weapons

Hidden when all 4 slots are filled. Otherwise, for every weapon NOT in the current loadout:

```
score = sum of WEIGHT for each stat in candidate.weapon
        where counts[stat] > 0 AND weights[stat] > 0
```

Dead stats (`count == 0` or `weight == 0`) are filtered out and contribute 0. Situational stats (`count == 1`) and common stats (`count > 1`) both keep their full weight.

Sort by score DESC, α tiebreaker.

Display: just the score (no label). When all weights are 1.0 and no stats are dead, score equals the raw stat count; with non-default weights or dead stats, decimals appear or the weapon drops toward the bottom.

### Weights affect everything

The same `weights` object influences all four displays:

- **Common / Situational lists** — multiplied into the sort key
- **Dead list** — `weight == 0` moves a stat here regardless of count
- **Complementary list** — weighted sum of shared stats

Default of 1.0 makes weights a no-op multiplier, so the tool behaves identically to a pre-weights version when weights are untouched.

### Duplicate prevention

Weapons already selected in any slot are marked `disabled` in all other slots (rebuilt on every change). Same weapon cannot be picked twice.

## Design choices and rationale

### "Simple, clean" visual design

- System font stack — uses whatever the OS provides
- Warm stone/neutral palette (light and dark variants)
- Minimal accent color — only used for focus rings
- No icons, no images, no heavy shadows
- Tabular numerics for all numeric displays (alignment)

### Dark mode via `prefers-color-scheme`

CSS variables on `:root` define the light palette; `@media (prefers-color-scheme: dark)` overrides them. Browser handles live theme tracking — switching the OS theme mid-session updates the page instantly. No JS, no toggle UI.

### No persistence

Weights and weapon selections reset on refresh. This was a deliberate choice — keeps the tool stateless and predictable, no storage code to maintain. If persistence is desired later, localStorage is a one-line add.

### Minimal stat display

The stat rows show just the count (e.g., `2/4`) — no per-row weight badges. Weights influence ordering silently. Reasoning: showing 10 weight values on every stat row would add visual noise without much value; users can see the effect of their weights by watching the list reorder.

### Number inputs, not sliders

Native `<input type="number">` with `step="0.5"` gives precise control via arrow keys AND direct typing. Sliders would force the user to drag to set values, which is worse for numeric input. No defined ceiling — users can type any positive value.

### Single-file architecture

Everything inlined in one `index.html`:
- HTML structure
- CSS (with dark-mode media query)
- JS data tables
- JS logic

Trade-offs: not modular, can't tree-shake, harder to test in isolation. Benefits: trivial to share, no build pipeline, no server needed, opens from disk. Right call for a personal companion tool.

## Adding new weapons or stats

### New weapon

Append to the `WEAPONS` object — alphabetized for predictable dropdown order:

```js
"New Weapon Name": ["Stat 1", "Stat 2", "Stat 3"]
```

### New stat

1. Add to `ALL_STATS` (keep it ordered; the order appears in the weight panel)
2. Reference it in any `WEAPONS` entries that should benefit from it

## Future ideas (not implemented)

- localStorage persistence for weights and selections
- Reverse lookup — pick stats you have, see which weapons benefit
- Per-weapon icons / images
- Mobile-optimized layout for narrower viewports (currently collapses below 960px)

## File layout

```
index.html      Single-file app (HTML + CSS + JS, all inlined)
AGENTS.md       This file
```