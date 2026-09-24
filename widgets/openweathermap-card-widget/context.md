# Widget context: openweathermap-card-widget

## Purpose

Weather card for the OpenWeatherMap binding (`weather-and-forecast` Thing).
Top: current icon, big current temperature, one line with condition, today's
min/max, wind, rain and rain probability. Below: one column per following day
with localized weekday, date, icon, max/min temperature, rain probability and
rain amount (mm).

## Items / Groups used

Nothing is passed item by item: the widget builds names from a prefix,
`<prefix>_<group>_<channel with - replaced by _>` (default prefix `OWM`).
All Number items must be quantity types; temperatures are read as °C.

- `OWM_current_`: `condition`, `icon` (Image), `temperature`, `apparent_temperature`, `humidity`, `wind_speed`, `rain`
- `OWM_forecastToday_`: `min_temperature`, `max_temperature`, `precip_probability`, `rain`, `snow`, `uvindex`, `humidity`, `wind_speed`, `sunrise`, `sunset`, `time_stamp`, `icon`, `condition`
- `OWM_forecastTomorrow_`, `OWM_forecastDay2_`, `OWM_forecastDay3_`, `OWM_forecastDay4_`: `time_stamp`, `condition`, `icon`, `min_temperature`, `max_temperature`, `precip_probability`, `rain`, `snow`

Source: `openweathermap:onecall:*` Thing (One Call 3.0), groups `forecastToday`
… `forecastDay4`. The `current` items are linked to the `weather-and-forecast`
Thing. The `weather-and-forecast` daily groups need the paid "Daily Forecast
16 days" API and stay NULL on a One Call plan, so do not link them.
Units seen: rain in `m` (widget converts to mm), `precip-probability` a
fraction 0..1 (widget converts to %), `time-stamp` ISO string.

## Config parameters

| Name | Type | Default | Notes |
|---|---|---|---|
| `prefix` | TEXT | `OWM` | item name prefix |
| `days` | INTEGER | 5 | including today, max 5 until more items exist |
| `unit` | TEXT | `C` | `C` or `F` (converted in the widget) |
| `locale` | TEXT | `fr` | `fr`, `en`, `de`: weekday names and labels |
| `title` | TEXT | localized | header text |
| `maxWidth` | TEXT | `28rem` | card max-width |

## Layout notes

- Plain flex divs, inline styles, no f7 layout components.
- Weekday is computed from the `time_stamp` date string (Sakamoto formula):
  the evaluator's `Date`/`Intl` support is unverified, so they are avoided.
- Icons use `oh-image` with the Image item (`items[x].state` in expressions is
  only a text description of the image, not a usable `src`).
- Evaluator has no `isNaN`/`parseFloat`/`parseInt`: numbers via unary `+`,
  NaN check via `v !== v`. `oh-context` variables do not refresh with item
  state, so date/number expressions are inlined. `oh-repeater` takes `in`
  (not `list`).
- Today card has an extras row: UV, humidity, sunrise, sunset (labels per locale).
- Widgets must be placed in `oh-grid-row`/`oh-grid-col` or `oh-grid-cells`
  on a page; directly in an `oh-block` they do not resolve.

## Known issues / TODO

- Verified live on Main UI (dark theme, `locale: de`). Only `weather-and-forecast`-linked items feed `current`.
- Rain assumes `mm`; snow items exist but are not displayed.
