# Megabonk Stat Targeting

A small companion tool that helps you decide which stat buffs (from items,
shrines, and the like) to chase based on the weapons you're currently
running.

No install, no server, no build step. Just open `index.html` in any modern
browser.

## What it does

Pick up to four weapons. The tool tallies which stats each one benefits
from and groups them so you can see at a glance:

- **Common Stats** — two or more of your weapons benefit from these. The
  strongest targets for any stat buff you find.
- **Situational Stats** — exactly one weapon benefits. Useful, but not the
  same priority as common stats.
- **Dead Stats** — none of your weapons benefit. Skip these.
- **Complementary Weapons** — every other weapon in the game, ranked by
  how well its stats line up with the stats your current loadout already
  cares about. Hidden once all four slots are full.

Stats come from items and buffs, not from weapons themselves. The tool
models this correctly: picking up a stat buff benefits whichever weapons
are in your loadout.

## Using it

1. **Choose weapons.** The left panel has four dropdowns. Pick a weapon in
   each slot you want to fill. The same weapon can't go in two slots —
   already-selected weapons are dimmed in the other dropdowns.
2. **Read the middle panel.** Common Stats goes first because that's where
   you should focus. Situational next, Dead at the bottom. Each stat shows
   a count like `2/4` (number of weapons that benefit out of the total
   selected).
3. **Tune the weights (optional).** The right panel has one weight per
   stat, defaulting to `1.00`. Higher weights boost that stat's ranking in
   the common and situational lists, and make complementary weapons that
   share that stat score higher. Setting a weight to `0` opts a stat out
   entirely — it moves to the Dead list regardless of how many weapons
   benefit from it.
4. **Reset.** Two buttons: "Reset weapons" clears your loadout;
   "Reset all" returns every weight to `1.00`.

## Working with weights

Each weight row has a slider and a number input that stay in sync — drag
the slider for quick adjustments, or type a value directly for precision.

The number input accepts basic arithmetic, so you can type things like
`3/2`, `(2+3)*0.4`, or `1.25` instead of doing the math elsewhere. Results
are clamped to `0` and below; anything that doesn't parse cleanly
resets to `1.00` when you commit the field (blur or Enter).

The slider's range stretches automatically to fit the highest weight
you've set, so you never run out of headroom while dragging.

## Notes

- Selections and weights reset on page refresh. There's no save state by
  design — keeps the tool stateless and predictable.
- The tool follows your OS theme (light or dark) automatically.
- Below a 960px viewport the panels stack vertically.
</content>
</invoke>