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

Colors are fixed in the `tempo` variable of the root `oh-context`: per state,
`bg` (the gradient stack: highlight, then light/base/dark shades), `shadow`
(inner shading + glow) and `text`. Base shades are BLUE `#1e88e5`, WHITE
`#f0f0f0`, RED `#e53935`, N/A `#9e9e9e`. Edit them there if needed; they were
not exposed as props on purpose.

## Layout notes

- Root is a flex column; the two columns sit in a flex row
  (`justify-content: space-around`), each column a flex column
  (circle, label).
- The circle is a fixed 5rem `div` with `flex: none` (avoids the
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

- Not yet deployed/verified against a live openHAB instance. The circle look
  was checked by rendering the same CSS in headless Chrome (light and dark
  card backgrounds), not in Main UI itself.
