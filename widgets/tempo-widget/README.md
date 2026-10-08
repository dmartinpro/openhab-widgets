# Tempo Widget

A Main UI widget showing the day colors of an **EDF Tempo** electricity
contract: today's color, tomorrow's color, and when they were last updated.

## What it does

- Shows two flat rounded tiles side by side, labeled **Today** and
  **Tomorrow** (configurable), filled with the Tempo color of that day:
  **blue**, **white** or **red**. The color name is written on the tile too.
- Shows a grey tile reading **N/A** when the item holds anything else — the
  error value, `NULL`, `UNDEF`, or an item that isn't bound.
- Shows the **last update time** on a single line underneath
  (`YYYY-MM-DD HH:mm`).

The widget is display-only: it sends no commands. It is styled as a card
(theme card background and corner radius) so it sits next to native
`oh-label-cell`s in an `oh-grid-cells` block.

## About Tempo

Tempo is an EDF contract where each day of the year is assigned one of three
colors, each with its own price per kWh (peak and off-peak hours, 06:00–22:00
and 22:00–06:00). Over a Tempo year (1 September to 31 August) there are 300
blue days (cheapest), 43 white days and 22 red days (most expensive).
Red days only occur between 1 November and 31 March, Monday to Saturday.
A Tempo "day" runs from 06:00 to 06:00. The color of the next day is
announced the day before.

Knowing tomorrow's color in advance is the whole point of the contract, hence
the two tiles: they tell you when to shift consumption (heating, hot water,
EV charging, laundry, ...) away from white and especially red days.

## Prerequisites

Three Items, fed by whatever retrieves your Tempo colors (a binding, an HTTP
call plus a rule, ...). The widget doesn't care where the data comes from,
only about the Item types and values:

| Item | Type | Expected state |
|---|---|---|
| Today | `String` | `BLUE`, `WHITE`, `RED`, or any other value on error |
| Tomorrow | `String` | `BLUE`, `WHITE`, `RED`, or any other value on error |
| Last update | `DateTime` | Time of the last **successful** update |

```java
String   Tempo_Today       "Tempo today"
String   Tempo_Tomorrow    "Tempo tomorrow"
DateTime Tempo_LastUpdate  "Tempo last update"
```

Values are matched **case-sensitively** and in upper case exactly as above.
If your source uses another spelling (`Blue`, `bleu`, ...), map it to
`BLUE` / `WHITE` / `RED` before it reaches the Item.

## Widget setup

1. In Main UI, go to **Developer Tools → Widgets → +** and paste in the
   contents of [`widget.yaml`](widget.yaml). Save.
2. Add the widget to a page and set its three Item parameters to the Items
   above. The labels are optional (e.g. `Aujourd'hui` / `Demain`, and `Bleu` / `Blanc` /
   `Rouge` for the colors).

## Config parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `todayItem` | Item (String) | yes | Color for the current day |
| `tomorrowItem` | Item (String) | yes | Color for the next day |
| `timestampItem` | Item (DateTime) | yes | Last successful update |
| `todayLabel` | Text | no | Label on the left tile. Defaults to `Today` |
| `tomorrowLabel` | Text | no | Label on the right tile. Defaults to `Tomorrow` |
| `blueLabel` | Text | no | Text on a `BLUE` tile. Defaults to `Blue` |
| `whiteLabel` | Text | no | Text on a `WHITE` tile. Defaults to `White` |
| `redLabel` | Text | no | Text on a `RED` tile. Defaults to `Red` |
| `naLabel` | Text | no | Text on a grey tile (any other state). Defaults to `N/A` |

## Colors

| State | Tile |
|---|---|
| `BLUE` | Blue, white text "Blue" |
| `WHITE` | Off-white with a thin grey outline (visible on light cards), dark text "White" |
| `RED` | Red, white text "Red" |
| anything else | Grey, "N/A" |

The printed names can be translated per widget instance with the four
`*Label` parameters (e.g. `Bleu` / `Blanc` / `Rouge` / `N/A`); left empty they
fall back to the English names. The shades themselves are set in the `tempo`
variable at the top of [`widget.yaml`](widget.yaml).

## Known limitations

- The timestamp is read straight from the Item's ISO state, so it shows the
  time in the offset openHAB reports (the server's), not converted to the
  browser's timezone.
- No indication of *why* a tile is N/A — an error value and a missing
  update look the same.
