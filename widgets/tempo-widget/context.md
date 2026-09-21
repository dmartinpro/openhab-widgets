# tempo-widget

## Purpose

Compact display of the EDF Tempo day colors: two columns ("Today" and
"Tomorrow"), each a filled circle in the day's color (BLUE / WHITE / RED),
with a single line underneath showing when the data was last updated. Any
value other than those three (error value, `NULL`, `UNDEF`, unbound item) is
shown as a grey circle with "N/A" in it.

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
| `todayLabel` | TEXT | `Today` | Optional. Label under the left circle; empty falls back to `Today` |
| `tomorrowLabel` | TEXT | `Tomorrow` | Optional. Label under the right circle; empty falls back to `Tomorrow` |

Colors are fixed in the `tempo` variable of the root `oh-context`: per state,
`bg` (the gradient stack: highlight, then light/base/dark shades), `shadow`
(inner shading + glow) and `text`. Base shades are BLUE `#1e88e5`, WHITE
`#f0f0f0`, RED `#e53935`, N/A `#9e9e9e`. Edit them there if needed; they were
not exposed as props on purpose.

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
- Content is sized to fit inside 120px with 0.5rem padding: 3.2rem circles,
  0.8rem labels, 0.75rem timestamp.
- Inside the root, a flex row holds the two columns
  (`justify-content: space-around`), each column a flex column (circle,
  label), with the timestamp line below.
- The circle is a fixed 3.2rem `div` with `flex: none` (avoids the
  border-box flex-shrink oval bug), `border-radius: 50%`. Its text is empty
  for the three real colors and "N/A" for the fallback — one lookup into
  `vars.tempo` drives background, box-shadow and text.
- The 3D "control light" look is pure CSS on that single element (no extra
  layers): `background` is two stacked radial gradients (a glossy white
  highlight near the top, over a shaded sphere lit from the upper left), and
  `box-shadow` adds inner shading, a small drop shadow and — for the three
  real colors only — a colored outer glow. N/A has no glow, so it reads as
  an unlit lamp. `background-clip: padding-box` keeps the gradient inside the
  semi-transparent grey rim (`border`), which also keeps the WHITE lamp
  visible on a white/light card.
- Value matching is case-sensitive (`BLUE`, not `Blue`).
- The timestamp is formatted by slicing the ISO state
  (`2026-09-21T14:32:00.000+0200` → `2026-09-21 14:32`) rather than with a
  date library: only `.slice`/ternaries are confirmed in the expression
  evaluator here. It therefore shows the time as openHAB reports it (server
  offset), not converted to the browser's timezone. A non-ISO/`NULL`/`UNDEF`
  state shows `—`.

## Known issues / TODO

- Card styling verified against the live instance (dark theme, phone width)
  by comparing computed styles with the neighbouring label cells. Not
  re-checked on the light theme or on tablet/desktop widths.
