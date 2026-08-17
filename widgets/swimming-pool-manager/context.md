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
- **Setpoint slider** (`oh-slider-item`) defaults to **15–32°C, step 0.5**,
  `releaseOnly: true` (command sent only when the user releases the
  handle, to avoid flooding the heat pump with commands while dragging).
  Adjust `min`/`max`/`step` in `widget.yaml` if the real hardware range
  differs.
- **Temperature readouts** (`oh-label-item`) show `displayState` (falls
  back to raw `state`) so they respect any unit/formatting already defined
  on the item's state description.
- No absolute positioning; all spacing is flexbox `gap` at the root level,
  each `oh-list-card` handling its own internal row layout natively.

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
