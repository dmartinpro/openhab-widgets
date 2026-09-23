# navimow-widget-compact

## Purpose

Lightweight companion of [`navimow-widget`](../navimow-widget/): a small
card (same shape/style as [`tempo-widget`](../tempo-widget/)) showing the
mower image, current activity and battery level. Clicking it opens an
*extended* widget in a popup — by default the main Navimow widget, but the
target is a configuration parameter so other widgets can be used.

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
| `modalWidget` | TEXT (`context: widget`) | `widget:navimow_mower_card` | Widget opened on click; value format `widget:<uid>` |

## Layout notes

- Root `div` copies the `oh-label-cell` card style exactly as `tempo-widget`
  does (fixed 120px height, margin, card background/radius/shadow) — see
  that widget's context.md for the measurements.
- Inside the root, an `oh-link` fills the card and carries the click action
  (`action: popup`, `actionModal`, `actionModalConfig`). Because the
  `oh-link` is a `.link` element it gets an explicit `opacity: 1` (no hover
  fade), `text-decoration: none`, `color: inherit`.
- Row: mower `img` (`flex: 1 1 0; min-width: 0; max-width: 4.5rem`) on the
  left, a column on the right with the activity (colored icon + label) on
  top and a horizontal battery bar with the percentage underneath. `min-width: 0`
  on the image avoids the flex overflow bug seen in the main widget.
- Activity table (`vars.activityMeta`), battery thresholds (≤20 red, ≤50
  amber, else green) and the image lookup (first 2 characters of the model →
  `vars.modelImages`, else `vars.defaultImage`) are copies of the main
  widget's.

## Popup mechanism

Main UI's `oh-link` modal actions (`popup`/`popover`/`sheet`) require
`actionModal` to be `page:<uid>` or `widget:<uid>` and pass
`actionModalConfig` as the target's props. The widget passes the four item
names under the same prop names as the main Navimow widget
(`activityItem`, `controlItem`, `batteryItem`, `modelItem`), so any target
widget that uses these prop names works. The `modalWidget` picker uses
`context: widget` (Main UI's parameter editor offers all widgets, values in
`widget:<uid>` form).

## Model images

Embedded as base64 like the main widget, but downscaled to 110px wide
(~10–13KB each) in `assets/`: `H2`, `i1`, `i2`, `X3`, `X4` plus
`default-model.png`. Widgets cannot share assets, so these are duplicated
from the main widget; if the main widget gets new categories, add them here
too (resize with `sips -Z 110`, base64-encode, add to `vars.modelImages`).

## Known issues / TODO

- The target widget's props are fixed to the four item names above; a
  target with different prop names is not supported.
- Verified on the live instance (dark theme) that the card renders and the
  popup opens the main widget with all four items working. The `context:
  widget` picker was verified in Main UI's source but not clicked through.
