# swimming-pool-manager

## Purpose

Reusable Main UI widget acting as a control panel for the swimming pool:
special commands (forced filtration / off-season mode), filtration pump
control, and heat pump (PAC) management (power, mode, setpoint, and the two
water temperature readings). Built as a widget with `props` rather than a
standalone page, so it can be dropped into any page/parent widget and
re-pointed at different items if needed (e.g. a second pool).

## Items / Groups used

All items are passed in via props (see below) rather than hardcoded, but the
defaults match the real items currently in use:

- `swimmingPoolFiltrationForcedOperation` — switch, forced vs normal filtration mode
- `swimmingPoolOffSeasonMode` — switch, off-season (winterization) mode
- `Pool_Controller_Swimming_Pool_Pump` — switch, filtration pump start/stop
- `swimmingpoolHeaterPower` — switch, heat pump on/off
- `Poolex_Jetblack_FI_7_mode` — string/enum, heat pump operating mode
- `swimmingPoolWaterTemperatureSetpoint` — number, heat pump target water temperature
- `Pool_Controller_Temperature_Eau_Injection_Piscine` — number (read-only display), water temperature leaving the pool as it enters the filtration circuit (labeled "Température eau provenance piscine" — opposes "Température eau injectée piscine" below)
- `Pool_Controller_Temperature_Eau_Piscine` — number (read-only display), water temperature injected back into the pool

## Config parameters

| Prop | Default | Section |
|---|---|---|
| `itemFiltrationForcedMode` | `swimmingPoolFiltrationForcedOperation` | Commandes spéciales |
| `itemOffSeasonMode` | `swimmingPoolOffSeasonMode` | Commandes spéciales |
| `itemFiltrationPump` | `Pool_Controller_Swimming_Pool_Pump` | Filtration |
| `itemHeaterPower` | `swimmingpoolHeaterPower` | Chauffage |
| `itemHeaterMode` | `Poolex_Jetblack_FI_7_mode` | Chauffage |
| `itemWaterTemperatureSetpoint` | `swimmingPoolWaterTemperatureSetpoint` | Chauffage |
| `itemInputWaterTemperature` | `Pool_Controller_Temperature_Eau_Injection_Piscine` | Chauffage |
| `itemPoolWaterTemperature` | `Pool_Controller_Temperature_Eau_Piscine` | Chauffage |

## Layout notes

- Root is a plain `div` (flexbox column, `gap: 1rem`) holding a widget
  title ("Centre commande piscine", 2026-08-25 — a plain `div` with
  `config.content`, the generic-HTML-tag way to set text content in Main
  UI YAML, styled inline rather than via an `f7-block-title`) followed by
  three `oh-list-card` sections — one per section, as decided: "Commandes
  spéciales", "Filtration", "Chauffage". `oh-list-card` was chosen over
  `f7-card` + manual `div`s because it natively provides the Main UI list
  styling for the row-style item widgets (`oh-toggle-item`,
  `oh-list-item`, `oh-slider-item`, `oh-label-item`) nested in its
  `default` slot — this is the one native-style exception per the project's
  general "prefer divs" rule.
- **Heat pump mode selection** uses `oh-list-item` with `action: options` /
  `actionItem` / `actionOptions` (the documented Main UI equivalent of the
  legacy sitemap `Selection` element) rather than a dedicated select
  component — openHAB's Main UI component library has no standalone
  `oh-select-item`. The currently selected mode is shown via the `after`
  field, computed with an inline JS object-literal lookup against the same
  French label map used for `actionOptions`, falling back to the raw state
  if the value isn't in the map.
- **Setpoint control** (`oh-stepper-item`, replacing an earlier slider —
  see 2026-08-25 note below) defaults to **15–32°C, step 0.5**, with
  `autorepeat`/`autorepeatDynamic` enabled so holding `+`/`-` accelerates
  through a larger change. Adjust `min`/`max`/`step` in `widget.yaml` if
  the real hardware range differs.
- **Temperature readouts** (`oh-label-item`) show `displayState` (falls
  back to raw `state`) so they respect any unit/formatting already defined
  on the item's state description.
- No absolute positioning; all spacing is flexbox `gap` at the root level,
  each `oh-list-card` handling its own internal row layout natively.

## Layout fix — narrow rendering in popup/page (2026-08-17)

Reported: when opened (via a group/equipment default-widget popup or page),
the three cards rendered in a narrow ~250px column with a lot of empty
space to the right, and the title text wrapped awkwardly inside that
narrow column — a strong sign of a **fixed-width parent grid cell**
(e.g. a CSS `grid-template-columns: repeat(auto-fill, minmax(200px, 1fr))`
pattern, which is how openHAB's auto-generated equipment/group overview
grids commonly lay out single default widgets: with `auto-fill`, unused
grid tracks stay reserved as empty space instead of letting the one
item stretch, unlike `auto-fit`).

Fix applied to the root `div`'s `style`:
- `grid-column: 1 / -1` — makes the widget span every column if its
  parent turns out to be a CSS grid (a no-op, harmless, if the parent
  isn't a grid).
- `flex: 1 1 100%` — same defensive idea for a flex-row parent.
- `width: 100%` / `max-width: 100%` / `box-sizing: border-box` — belt and
  braces for a plain block parent.
- Each `oh-list-card` also got an explicit `style: {width: 100%}`, since
  card components can carry their own intrinsic/shrink-to-fit width.
- **Follow-up (2026-08-25):** that `width: 100%` combined with the F7
  card's own default horizontal margin caused the opposite problem — the
  card rendered slightly *wider* than the popup, producing a horizontal
  scrollbar. Added `margin: 0` alongside `width: 100%` on each
  `oh-list-card`'s `style` to remove that default margin; vertical
  spacing between cards still comes entirely from the root `div`'s
  `gap: 1rem`, so nothing is lost.

Also fixed a **duplicated temperature value** on the setpoint slider
(`29 °C29 °C`): `oh-slider-item` already displays its own current value
next to the slider — the extra `after` expression I'd added duplicated
it. Removed; only the `oh-label-item` rows (which do *not* auto-display
their state — confirmed in the official docs) still set `after` manually.

If cards are still narrow after this, the constraint is most likely on
the **caller's side** rather than fixable from inside this widget — e.g.
an explicit column/width set on whatever list/grid/popup is invoking
`widget:swimming_pool_manager`. In that case the next thing to check is
how the widget is being opened (equipment/group default widget vs. a
manual `action: popup` on a link/list item) so the parent's own
width/column config can be adjusted instead.

## Setpoint control: slider → stepper (2026-08-25)

Replaced `oh-slider-item` with `oh-stepper-item` (`-` / value / `+`) for
`itemWaterTemperatureSetpoint`, per user request for discrete +/- control
instead of a drag slider. Decisions validated with the user beforehand:
- **Replace, not add** — a single control per item, no redundant slider
  left alongside it.
- **Value shown inline** between the buttons (component default) rather
  than `buttonsOnly` + a separate `after` field.
- **`autorepeat` + `autorepeatDynamic`** enabled — holding a button
  accelerates the change, useful for larger adjustments (e.g. +3°C)
  without many individual taps.

`oh-stepper-item` has no `unit`/`releaseOnly` config (unlike
`oh-slider-item`) — commands are sent immediately per step/repeat tick,
and the displayed value's formatting comes from the item's own state
description, not a widget-level unit override.

## Label clarification: "provenance piscine" vs "injectée piscine" (2026-08-25)

`itemInputWaterTemperature` was originally labeled "Température eau entrée
PAC", which was misleading: this item actually reads the water temperature
as it **leaves the pool**, before entering the filtration circuit — not a
heat-pump-specific reading. Renamed to **"Température eau provenance
piscine"**, chosen to read as the counterpart of the other label,
**"Température eau injectée piscine"** (`itemPoolWaterTemperature`): one
is water leaving the pool, the other is water returning to it.

## Tap-to-analyze on the two temperature readouts (2026-08-25)

Both temperature `oh-label-item` rows ("Température eau provenance
piscine" / "Température eau injectée piscine") now open the built-in
Main UI **Analyzer** (historic chart) for their respective item on tap,
via:
```yaml
action: analyzer
actionAnalyzerItems: =props.item...
```
No `actionAnalyzerChartType`/`actionAnalyzerCoordSystem`/
`actionAnalyzerAggregation` were set, so the analyzer opens with its own
defaults (dynamic period, time coordinate system, no aggregation) — the
same as opening it manually from an item's context menu. Add those keys
in `widget.yaml` if a specific default view (e.g. last 7 days, averaged)
turns out to be preferable once tested live.

## Filtration pump becomes read-only under forced filtration (2026-08-25)

When `itemFiltrationForcedMode` is `ON`, "Pompe de filtration" should
still show the pump's state but no longer be user-controllable. Main
UI's `oh-toggle-item` has **no native `disabled`/`readonly` config** (not
documented on the component, and no such generic property exists across
widgets — only `visible`, `visibleTo`, `class`, `style`), so this is
implemented as **two mutually-exclusive sibling widgets bound to the same
item**, gated by the officially documented `visible` expression:
- `oh-toggle-item` (interactive) — `visible` when
  `itemFiltrationForcedMode` state is **not** `ON`.
- `oh-label-item` (read-only, with a "Verrouillée (filtration forcée
  active)" subtitle so the change in behavior is self-explanatory) —
  `visible` when it **is** `ON`.

This is the safer, documented approach compared to e.g. a CSS
`pointer-events: none` hack on the toggle, which isn't a confirmed/
supported pattern for this widget.

## Section icons via embedded base64 images (2026-08-25)

Added an icon next to the widget title and next to the "Filtration" and
"Chauffage" section titles, per user request, using **base64 data URIs
embedded directly in `widget.yaml`** rather than external image files —
this was a deliberate choice (discussed and confirmed with the user)
over two alternatives: (a) deploying PNGs to a static folder on the
openHAB server and referencing them by URL, or (b) registering them as
a custom classic icon set. Both alternatives would have split the
widget across two things to deliver/keep in sync; embedding keeps
`widget.yaml` fully self-contained and copy-paste portable.

**Source files** are kept at
[assets/](assets/) (`swimming-pool.png`, `water-pump.png`,
`air-source-heat-pump.png`, 512×512 originals) — keep these around as
the source of truth if the icons ever need to be re-cropped or
re-encoded at a different size; they are not referenced by the widget
directly (git-tracked for provenance only, not consumed at runtime).

**Encoding process:** resized each source PNG to **64×64** with `sips`
(macOS) before base64-encoding, to keep the embedded strings small
(~6–10 KB each, ~24 KB total added to the YAML) — full-resolution
512×512 originals would have made the file unnecessarily large for
what only ever renders as a small inline icon (~28–32px).

**Placement mechanics:**
- The widget title is just our own root `div` — turned into a small
  flex row (`display:flex; align-items:center; gap:0.5rem`) with an
  `img` (the data URI) followed by the text `div`.
- "Commandes spéciales" was left as a plain `title:` string — no icon
  was requested for that section.

### Fix (2026-08-25): `oh-list-card`'s `header` slot doesn't actually work

The first version of this feature put the Filtration/Chauffage icon+text
row in the card's `header` slot (reasoning: `oh-card.vue`'s own template
has `<slot name="header"><f7-card-header v-if="config.title">...`, so a
supplied `header` slot should override the plain-text title). **This
broke in the real UI: both section titles disappeared entirely — no
icon, no text.**

Root cause, found by reading `oh-list-card.vue`'s source on GitHub:
`oh-list-card` is not a passthrough wrapper — its template is
```vue
<oh-card :context="context">
  <template #content>
    <oh-list :context="cardChildContext(context.component)" />
  </template>
</oh-card>
```
It only ever forwards the **`content`** slot to the underlying `oh-card`.
Any `header` (or `footer`) slot defined in the widget YAML under an
`oh-list-card` is never read by anything — Vue slots have to be
explicitly relayed by each wrapper component, and this one only relays
`content`. So the custom header slot was silently dropped, and since we
had also removed the card's `title:` config (in favor of the — dead —
header slot), `oh-card`'s own fallback (`v-if="config.title"`) had
nothing to render either. Both mechanisms failed at once, hence a
completely empty header.

**Actual fix:** stopped relying on any card-internal header mechanism.
Each of "Filtration" and "Chauffage" is now a plain wrapper `div`
(`flex-direction: column`, `gap: 0.5rem`) containing two siblings:
1. The icon+text row (identical pattern to the widget title), sitting
   visually above the card.
2. The `oh-list-card` itself, with no `title:` at all (so `oh-card`
   renders no header block, avoiding a redundant/empty one) — just its
   list of `oh-toggle-item`/`oh-label-item`/etc. rows in `content`.

The wrapper's own `gap: 0.5rem` keeps the label tight against its card,
while the root widget's `gap: 1rem` still separates whole sections from
each other. This only touches presentation — no config parameter names,
item bindings, or `visible` logic changed.

## Background temperature trend on the two readouts (2026-08-25)

Both temperature `oh-label-item` rows now show a subtle historic trend
line behind their text, using the System widget **`oh-trend`**
(`trendItem`, low `opacity`) — the component openHAB documents
specifically as "designed to render as a background visualization"
behind other content, not a standalone chart (that role is already
covered by the tap-to-open Analyzer added earlier).

**Conflicts with the house "no absolute positioning" rule** ([CLAUDE.md](../../CLAUDE.md)) —
layering a background trend behind foreground text is fundamentally a
stacking problem that flexbox alone can't express (flexbox lays
siblings out in a line, it doesn't overlap them). The usual way to do
this is `position: absolute`, which this project avoids. Instead, each
row is a `div` with `display: grid; grid-template-columns: 1fr;
grid-template-rows: 1fr`, and **both** the `oh-trend` and the
`oh-label-item` are placed in that same single grid cell
(`grid-column: 1; grid-row: 1`) — this achieves the overlap purely
through CSS Grid stacking, with no `position: absolute`/`fixed`
anywhere. `overflow: hidden` on the wrapper keeps the trend line's SVG
(whose exact height isn't configurable — it comes from the underlying
`vue-trend` library default) clipped to the row's bounds.

The `oh-label-item` needed an explicit `style: {background: transparent}`
override — list-item rows have an opaque background by default (matching
the card/page background), which would otherwise fully hide the trend
line sitting behind it.

**Prerequisite:** like the Analyzer action added earlier, `oh-trend`
needs the item to have **persisted history** (a persistence service —
rrd4j, InfluxDB, etc. — configured for `itemInputWaterTemperature` /
`itemPoolWaterTemperature`). Without persisted data the trend line will
simply render empty/flat.

**Not visually verified live** — `trendStrokeWidth: 2` and
`opacity: 0.25` are reasonable starting guesses to keep the line subtle
enough that the temperature text stays legible on top; adjust both (and
optionally `trendGradient`, which defaults to a blue gradient) once seen
on a real instance.

### Fix (2026-08-25): rows rendered much taller than a normal list row

Reported after live testing (persistence confirmed present on both
items): each row's height ballooned well past a normal list-item row.
Cause: the grid wrapper had no explicit `height`, only
`grid-template-rows: 1fr` — with an `auto`-sized container, `1fr` just
means "one row", not a fixed size, so the row grew to fit its tallest
child's *intrinsic* size. `oh-trend`'s underlying `vue-trend` SVG has no
configurable height and defaults to something much taller than a list
row; `height: 100%` on it was meaningless without a definite height on
its parent to be a percentage *of*.

Fixed by giving each wrapper `div` an explicit `height: 3rem` (roughly a
standard list-item row height) alongside its existing `overflow: hidden`
— this gives `height: 100%` on the `oh-trend`/`oh-label-item` children
something concrete to resolve against, and clips whatever the trend
SVG's natural size would otherwise be to that fixed 3rem row. Adjust
`3rem` in `widget.yaml` (both occurrences) if it still doesn't match the
other rows' height exactly once seen live.

### Fix (2026-08-25): trend line was centered, moved to the right edge

Reported after live testing: with `width: 100%` on `oh-trend`, the
sparkline stretched across the entire row, reading visually as
"centered" through the middle of the row rather than sitting behind the
value/icon area on the right. Changed each `oh-trend`'s style to
`justify-self: end` + `width: 50%` (was `width: 100%`, no
`justify-self`) — since both `oh-trend` and `oh-label-item` share the
same grid cell (`grid-column: 1; grid-row: 1`) but `oh-label-item` has
no `justify-self` override, it still stretches to the row's full width
by default, so the label/icon/value all render normally on top while
the trend itself is now confined and right-aligned within that same
cell. Adjust the `50%` in `widget.yaml` (both occurrences) if the
sparkline should cover more or less of the row's right side.

## Known issues / TODO

- **Not validated against a running openHAB 5.x instance** — this
  environment has no server access (see root [CLAUDE.md](../../CLAUDE.md)).
  The YAML follows the documented Main UI component reference and a
  confirmed working community example, but the `oh-list-item` /
  `action: options` block and the JS object-literal expression for `after`
  should be smoke-tested in the widget editor before relying on them.
- Icons (`f7:bolt_fill`, `f7:snow`, `f7:drop_fill`, `f7:flame_fill`,
  `f7:thermometer`) are a reasonable default guess, not a firm requirement
  — swap freely.
- If `Poolex_Jetblack_FI_7_mode` already carries a state description with
  these same value/label mappings (common for heat pump binding channels),
  the explicit `actionOptions` here is redundant but harmless; it exists so
  the widget is self-contained even if that item-level metadata is absent.
- The Filtration/Chauffage icon+title rows (now plain sibling `div`s
  above their `oh-list-card`, see the 2026-08-25 fix note above) haven't
  been visually confirmed live yet either — verify spacing/alignment
  once opened in a real instance.
