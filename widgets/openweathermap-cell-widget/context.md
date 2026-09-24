# Widget context: openweathermap-cell-widget

## Purpose

Cell-sized tile (120px high, same box as the Tempo cell) with today's weather
from the OpenWeatherMap binding. Companion of `openweathermap-card-widget`.

## Items / Groups used

Same `OWM_*` items as the card widget (see its context.md), name built from
the prefix:

- `OWM_current_icon` (Image), `OWM_current_temperature`, `OWM_current_condition`
- `OWM_forecastToday_uvindex`
- `OWM_forecastToday_min_temperature`, `OWM_forecastToday_max_temperature`, `OWM_forecastToday_precip_probability`

## Config parameters

| Name | Type | Default | Notes |
|---|---|---|---|
| `prefix` | TEXT | `OWM` | item name prefix |
| `unit` | TEXT | `C` | `C` or `F` (converted in the widget) |
| `locale` | TEXT | `fr` | `fr`, `en`, `de`; passed to the card popup |

## Layout notes

Flex row: icon (6rem, twice the first version) on the left; right column
(`flex: 1 1 0; min-width: 0`) stacks current temperature (2rem, light),
condition (one line, ellipsis), min / max, then a bottom row with the UV
index (colored by WHO level: <3 green, <6 yellow, <8 orange, <11 red, else
violet) on the left and a blue drop icon (`f7:drop_fill`) + rain probability on
the right. Same evaluator constraints as the
card widget (no `parseFloat`/`isNaN`, `oh-image` for icons). Put it in an
`oh-grid-cells` block.

Click: the whole cell is an `oh-link` (`action: popup`) opening
`widget:openweathermap_card` with `prefix`, `unit`, `locale` and
`maxWidth: 32rem`. The styled `div` must be the root (the grid sizes the root
element) and the link sets `align-items: stretch` because Framework7's `.link`
defaults to `center`, which shrink-wraps the rows.

Test page: `page.yaml` (uid `openweathermap_cell_test`).

## Known issues / TODO

- The right column gets ~100px on a 3-cells-per-row tablet layout; on a narrow phone (2 cells per row) it is very tight.
- The popup target is fixed to the card widget (no `modalWidget` parameter).
