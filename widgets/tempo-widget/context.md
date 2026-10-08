# tempo-widget

## Purpose

Compact display of the EDF Tempo day colors: two rounded tiles side by side
("Today" and "Tomorrow"), each filled with the day's color (BLUE / WHITE /
RED) and showing the label top-left and the color name bottom-left, with a
single line underneath showing when the data was last updated. Any value other
than those three (error value, `NULL`, `UNDEF`, unbound item) is shown as a
grey tile reading "N/A".

Display-only: no commands are sent.

## Items / Groups used

All passed in via `props`, nothing hardcoded:

- `todayItem` (required) — String item, today's color: `BLUE`, `WHITE`, `RED`, or an error value.
- `tomorrowItem` (required) — String item, tomorrow's color, same values.
- `timestampItem` (required) — DateTime item, last successful update.

## Config parameters

| Prop | Type | Default | Notes |
|---|---|---|---|
| `todayItem` | TEXT (item picker) | — | Required |
| `tomorrowItem` | TEXT (item picker) | — | Required |
| `timestampItem` | TEXT (item picker) | — | Required |
| `todayLabel` | TEXT | `Today` | Optional. Label on the left tile; empty falls back to `Today` |
| `tomorrowLabel` | TEXT | `Tomorrow` | Optional. Label on the right tile; empty falls back to `Tomorrow` |
| `blueLabel` | TEXT | `Blue` | Optional. Text printed on a tile whose state is `BLUE` |
| `whiteLabel` | TEXT | `White` | Optional. Same for `WHITE` |
| `redLabel` | TEXT | `Red` | Optional. Same for `RED` |
| `naLabel` | TEXT | `N/A` | Optional. Same for any other state (error, `NULL`, `UNDEF`, unbound) |

Colors are fixed in the `tempo` variable of the root `oh-context`: per state,
`bg` (tile fill), `fg` (text color), `name` (the built-in English text, "Blue" /
"White" / "Red" / "N/A"), `prop` (the name of the prop that overrides `name`,
e.g. `blueLabel`) and `ring` (an inset 1px grey outline, used only on WHITE so the
tile stays visible on a light card). Fills are BLUE `#1976d2`, WHITE
`#f5f5f5`, RED `#d32f2f`, N/A `#5f6368`; the darker blue/red keep white text
above 4.5:1 contrast. Edit them there if needed; they were not exposed as
props on purpose (only the printed names are, see above). The color *name* is printed on the tile so the state never
relies on color alone (color-blind readable).

## Layout notes

- The root `div` copies the computed style of a native `oh-label-cell`
  (`.card.oh-cell.label-cell`), measured on the live instance (2026-09-21):
  fixed `height: 120px`, `margin: 10px 5px` (a grid slot is 171px wide on a
  phone, so the card is 161px wide), `background: var(--f7-card-bg-color)`,
  `border-radius: var(--f7-card-border-radius)` (8px),
  `box-shadow: 0px 5px 10px #00000026`, `position: relative`,
  `overflow: hidden`, `user-select: none`, `font-size: 16px`. A property-by-
  property comparison of the live widget against a native cell showed no
  remaining difference in those. The fixed 120px comes from openHAB's
  `.oh-cell { height/min/max-height: 120px }`; the margin from
  `--f7-card-expandable-margin-*` and the shadow from
  `--f7-card-expandable-box-shadow`, which are only defined on `.oh-cell`
  itself, hence the hardcoded values. If openHAB changes those in a future
  release, update them here.
- The widget sits inside an `oh-cell-container` (`display: block`), which
  gives it no height of its own — that's why `height: 100%` did not work and
  an explicit 120px is needed.
- The root is a flex column with 8px padding: a tile row (`flex: 1 1 0`,
  `gap: 9px`) and a fixed 26px footer holding the timestamp. At the usual
  161px card width this gives two 68×78px tiles, matching the approved
  "Tiles" mock-up; at other widths the tiles just flex (`flex: 1 1 0`,
  `min-width: 0`).
- Each tile is a flex column with `justify-content: space-between`: the label
  (11px, 80% opacity, ellipsized if a long custom label is set) on top, the
  color name (17px, bold) at the bottom. Text color is set once on the tile and
  inherited, so one lookup into `vars.tempo` drives background, text color,
  outline and name (3 expressions per tile).
- Design history: the first version used glossy 3D "control light" circles
  (commit `ba2cc49` and earlier); replaced with flat tiles on 2026-10-08 as
  the circles looked dated. Gradients/glow were dropped on purpose.
- Translated names: the name text is `props[<state>.prop] || <state>.name` — one
  dynamic `props[...]` lookup keyed by the `prop` field of the matched state,
  so an unset/empty prop falls back to the English default. The state *values*
  (`BLUE`/`WHITE`/`RED`) are never translated, only what is printed.
- Value matching is case-sensitive (`BLUE`, not `Blue`).
- The timestamp is formatted by slicing the ISO state
  (`2026-09-21T14:32:00.000+0200` → `2026-09-21 14:32`) rather than with a
  date library: only `.slice`/ternaries are confirmed in the expression
  evaluator here. It therefore shows the time as openHAB reports it (server
  offset), not converted to the browser's timezone. A non-ISO/`NULL`/`UNDEF`
  state shows `—`.

## Known issues / TODO

- Card shell (size, margin, radius, shadow) was verified against the live
  instance (dark theme, phone width) before the tile redesign. The tile
  layout itself was only rendered locally (headless Chrome, light and dark card
  backgrounds, all four states) from this YAML, not yet in Main UI.
- Not checked on tablet/desktop widths.
