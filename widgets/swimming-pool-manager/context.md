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
- `Pool_Controller_Temperature_Eau_Injection_Piscine` — number (read-only display), inlet water temperature at the heat pump
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

- Root is a plain `div` (flexbox column, `gap: 1rem`) holding three
  `oh-list-card` sections — one per section, as decided: "Commandes
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
