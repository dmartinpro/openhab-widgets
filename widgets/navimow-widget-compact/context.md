# navimow-widget-compact

## Purpose

Lightweight companion of [`navimow-widget`](../navimow-widget/): a small
card (same shape/style as [`tempo-widget`](../tempo-widget/)) showing the
mower name/status on top, its image centered, and a battery bar on the
bottom. Clicking it opens an *extended* widget in a popup — by default the
main Navimow widget, but the target is a configuration parameter so other
widgets can be used.

## Items / Groups used

All passed in via `props`:

- `activityItem` — String, `activity` channel (read).
- `batteryItem` — plain `Number` (not `Number:Dimensionless`), `battery-level` channel.
- `modelItem` — String, `model` channel (e.g. `X430`), drives the image.
- `controlItem` — String, `control` channel. **Not used by the card itself**;
  it is only forwarded to the extended widget so its buttons work.

## Config parameters

| Prop | Type | Default | Notes |
|---|---|---|---|
| `activityItem` | TEXT (item) | — | Required |
| `batteryItem` | TEXT (item) | — | Required |
| `modelItem` | TEXT (item) | — | Required |
| `controlItem` | TEXT (item) | — | Required, forwarded to the popup widget |
| `title` | TEXT | — | Overrides the top-left name; defaults to `"Navimow " + modelItem state`, or `"Mower"` if unavailable |
| `modalType` | TEXT (options) | `popup` | `popup` / `sheet`, used as the `oh-link` action |
| `modalWidget` | TEXT (`context: widget`) | `widget:navimow_mower_card` | Widget opened on click; value format `widget:<uid>` |

## Layout notes

- Root `div` copies the `oh-label-cell` card style exactly as `tempo-widget`
  does (fixed 120px height, margin, card background/radius/shadow) — see
  that widget's context.md for the measurements.
- Inside the root, an `oh-link` fills the card, carries the click action
  (`action: popup`, `actionModal`, `actionModalConfig`), and is itself the
  flex column laying out the three rows (`flex-direction: column;
  justify-content: space-between`). Because the `oh-link` is a `.link`
  element it needs explicit `align-items: stretch`: Framework7's `.link`
  defaults to `align-items: center`, which without the override shrinks
  every row (the battery bar included) to its content width instead of the
  full card — this bit the first version of this redesign, caught only by
  measuring `getBoundingClientRect()` in the browser, not by eye. Also
  explicit `opacity: 1` (no hover fade) and `text-decoration: none; color:
  inherit`, same as the other `oh-link`-as-container widgets in this repo.
- **Row 1 (top, `flex: none`):** name on the left (`flex: 1 1 0; min-width:
  0`, ellipsis) — `props.title`, else `"Navimow " + modelItem state`, else
  `"Mower"` — and the activity status on the right (`flex: none`: colored
  icon + label, same `vars.activityMeta` table as before).
- **Row 2 (middle, `flex: 1 1 auto; min-height: 0`):** mower image centered
  both ways in a `display:flex; align-items:center; justify-content:center;
  overflow:hidden` box, sized with `max-width/max-height: 100%` so it never
  overflows whatever height rows 1 and 3 leave it (roughly 45–55px). Image
  lookup unchanged: first 2 characters of the model → `vars.modelImages`,
  else `vars.defaultImage`.
- **Row 3 (bottom, `flex: none`):** the battery bar, a `display: grid`
  two-layer stack — a fill `div` (`grid-area: 1 / 1`, width = clamped
  battery %, colored by the same thresholds as before: ≤20 red, ≤50 amber,
  else green) under the percentage text (`grid-area: 1 / 1; justify-self:
  center; align-self: center` — centered in the bar, not right-aligned like
  the pre-redesign layout or the openweathermap-cell-widget's rain row).

## Popup mechanism

Main UI's `oh-link` modal actions (`popup`/`popover`/`sheet`) require
`actionModal` to be `page:<uid>` or `widget:<uid>` and pass
`actionModalConfig` as the target's props. The widget passes the four item
names under the same prop names as the main Navimow widget
(`activityItem`, `controlItem`, `batteryItem`, `modelItem`), so any target
widget that uses these prop names works. The `modalWidget` picker uses
`context: widget` (Main UI's parameter editor offers all widgets, values in
`widget:<uid>` form).
