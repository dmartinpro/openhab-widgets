# Widget context: openweathermap-cell-widget

## Purpose

Cell-sized tile (120px high, same box as the Tempo cell) with today's weather
from the OpenWeatherMap binding. Companion of `openweathermap-card-widget`.

## Items / Groups used

Same `OWM_*` items as the card widget (see its context.md), name built from
the prefix:

- `OWM_current_icon` (Image), `OWM_current_temperature`, `OWM_current_condition`
- `OWM_forecastToday_min_temperature`, `OWM_forecastToday_max_temperature`, `OWM_forecastToday_precip_probability`

## Config parameters

| Name | Type | Default | Notes |
|---|---|---|---|
| `prefix` | TEXT | `OWM` | item name prefix |
| `unit` | TEXT | `C` | `C` or `F` (converted in the widget) |
| `locale` | TEXT | `fr` | `fr`, `en`, `de`; passed to the card popup |

## Layout notes

Flex column, `space-between`: row 1 = icon (3rem) + current temperature
(2rem, light); row 2 = condition (one line, ellipsis); row 3 = min / max on the
left, rain probability (blue) on the right. Same evaluator constraints as the
card widget (no `parseFloat`/`isNaN`, `oh-image` for icons). Put it in an
`oh-grid-cells` block.

Click: the whole cell is an `oh-link` (`action: popup`) opening
`widget:openweathermap_card` with `prefix`, `unit`, `locale` and
`maxWidth: 32rem`. The styled `div` must be the root (the grid sizes the root
element) and the link sets `align-items: stretch` because Framework7's `.link`
defaults to `center`, which shrink-wraps the rows.

Test page: `page.yaml` (uid `openweathermap_cell_test`).

## Known issues / TODO

- The popup target is fixed to the card widget (no `modalWidget` parameter).
