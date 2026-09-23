# Widget context: navimow-widget

## Purpose

Single-mower control card for the `navimow` binding (Segway Navimow robotic
lawn mower). Shows battery level, current activity, a default product
silhouette, and a 5-button control pad (START/STOP/PAUSE/RESUME/DOCK).
Designed to be repeated once per mower — not a multi-mower dashboard.

## Items / Groups used

- `activityItem` (config param) — String item bound to the mower's
  `activity` channel. Read-only. One of `idle`/`mowing`/`paused`/`docked`/
  `charging`/`returning`/`error`/`unknown`.
- `controlItem` (config param) — String item bound to the mower's `control`
  channel. Write-only; widget sends `START`/`STOP`/`PAUSE`/`RESUME`/`DOCK`.
- `batteryItem` (config param) — **plain `Number`** item (not
  `Number:Dimensionless` — see binding pitfall below) bound to
  `battery-level`. 0–100.
- `modelItem` (config param) — String item bound to the mower's `model`
  channel, e.g. `"X430"` or `"i110"`. Drives the card title and the
  per-category image lookup.

No Group binding required; the widget takes four independent item
references.

## Config parameters

| Name | Type | Required | Notes |
|---|---|---|---|
| `activityItem` | TEXT (item) | yes | String item, `activity` channel |
| `controlItem` | TEXT (item) | yes | String item, `control` channel |
| `batteryItem` | TEXT (item) | yes | Number item, `battery-level` channel |
| `modelItem` | TEXT (item) | yes | String item, `model` channel |
| `maxWidth` | TEXT | no | CSS max-width of the card, default `20rem`; the compact widget sets `32rem` so the card fills its popup |
| `title` | TEXT | no | Overrides header text; defaults to `"Navimow " + modelItem state`, or `"Mower"` if unavailable |

## Layout notes

- Root is a `div` card (flex column, inline styles only, no `f7-block`), per
  house convention. `max-width: 20rem; margin: 0 auto` keeps it centered when
  its container is wider (e.g. when opened in a popup from the compact widget).
- Header row: title (left) + activity icon/label chip (right), colored per
  `activity` via a static `vars.activityMeta` lookup table (icon, color,
  label per one of the 8 known activity values, with an `unknown` fallback
  entry also used for any value the lookup doesn't recognize).
- Body row: product image (flex, fills available width) + a small vertical
  battery gauge (fixed-width track, inner fill sized by `height: N%` off the
  actual numeric battery value, color thresholded ≤20% red / ≤50% amber /
  else green — no dependency on `batteryTier`).
- Control block: 2-column button grid — row 1 START/STOP, row 2
  PAUSE/RESUME, row 3 DOCK spanning full width — per user-approved layout
  (not the initial single-column proposal).
- Each button is an `oh-link` with **explicit** `opacity`/`cursor`/
  `display: flex` (per the `oh-link` default-styling gotcha — never relying
  on the element's unstyled default here).
- Enable/highlight rules are static lookup tables (`vars.enabledMap`,
  `vars.highlightMap`) keyed by `activity`, not per-button conditionals
  duplicated ad hoc — see table below.

### Activity → button state

| activity | enabled | highlighted |
|---|---|---|
| `idle` | START | — |
| `mowing` | PAUSE, STOP, DOCK | START |
| `paused` | RESUME, STOP, DOCK | PAUSE |
| `docked` | START | DOCK |
| `charging` | START | DOCK |
| `returning` | STOP | DOCK |
| `error` | STOP | — |
| `unknown` | none | — |

Disabled buttons are shown greyed (`opacity: 0.35`, `cursor: not-allowed`,
`action: none`) rather than hidden, so the button grid never reflows.
`STOP` and `PAUSE` both settle to the same `paused` activity value (binding
limitation) — the widget doesn't try to distinguish which was sent.

## Model image

- `widgets/navimow-widget/assets/default-model-source.png` — original
  870×481 PNG supplied by the user (670KB), kept for future re-edits.
- `widgets/navimow-widget/assets/default-model.png` — resized 220px-wide,
  43KB version, base64-embedded into `widget.yaml` as `vars.defaultImage`.
- **Per-category switching is implemented and populated.**
  `vars.modelImages` is a lookup table keyed by the first 2 characters of
  `modelItem`'s state (e.g. `X4` for `X430`/`X450`, `i1` for `i110`). Image
  `src` expression: `vars.modelImages[modelItem.state.slice(0,2)] ||
  vars.defaultImage`. Currently populated with 5 categories, each resized
  to 220px-wide before base64-embedding (matching `default-model.png`'s
  treatment): `H2` (39KB), `i1` (41KB), `i2` (38KB), `X3` (38KB), `X4`
  (32KB). Live-verified against the deployed `X430` mower → resolves to
  `X4` → correct image rendered.
  **Note**: the original higher-res source PNGs the user provided for
  these 5 (`*-nobg.png`) were deleted after resizing instead of being kept
  like `default-model-source.png` — only the 220px versions
  (`{category}-model.png`) survive on disk. Re-export at higher resolution
  from the original source if these need re-editing later.

## Known issues / TODO

- Categories beyond the 5 provided (`H2`, `i1`, `i2`, `X3`, `X4`) still
  fall back to `vars.defaultImage`. Add more `vars.modelImages` entries the
  same way as new categories are needed.
- **No offline/"unreachable" state.** Thing online/offline status
  (`statusInfo.status`) is still only reachable via
  `this.$oh.api.get('/rest/things/' + thingUID)` — that part of the
  original REST-based plan still stands, and is still unverified/deferred.
  Until wired in, the widget trusts `activity` at face value even if the
  mower is actually unreachable.
- **No `batteryTier` color hint.** Battery gauge color is purely
  threshold-based off the numeric `battery-level` (≤20/≤50/else). The spec
  allows `batteryTier` as a coarser hint but its value set beyond `"HIGH"`
  is unconfirmed, and it's a Thing property (same REST caveat as above) —
  skipped for v1.
- No spin/pulse animation on the `mowing` icon — Main UI widget YAML has no
  clean place to declare `@keyframes`; color + icon swap only for now.

## Deployment

Live on the user's local Docker openHAB (5.2.1) instance, pushed via the
`/rest/ui/components/ui:widget/navimow_mower_card` REST endpoint (not
copy-pasted through the Main UI code editor — the base64 image payload is
too large to type/paste reliably through browser automation without risking
corruption from editor auto-bracket-closing). Items backing the deployed
instance live in `/openhab/conf/items/navimow-test.items` in the container:
`Navimow_Activity`, `Navimow_Control`, `Navimow_Battery`, `Navimow_Model`
(all linked to `navimow:mower:<account>:<mower>`). Placed on the
Overview page (`overview`) via the same REST mechanism against
`/rest/ui/components/ui:page/overview`.
