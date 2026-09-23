# Navimow Mower Compact Card

A small card for a Segway Navimow robotic lawn mower, meant to sit next to
native label cells in an `oh-grid-cells` block (same shape as the
[Tempo widget](../tempo-widget/)). Click it to open a bigger widget — by
default the [Navimow Mower Card](../navimow-widget/).

## What it does

- Shows a small **image of the mower**, picked from its model.
- Shows the current **activity** (idle, mowing, paused, docked, charging,
  returning, error, unknown) as a colored icon and label.
- Shows the **battery level** as a bar and a percentage.
- On click, opens the **extended widget** of your choice in a popup.

## Prerequisites

The `navimow` binding with a mower Thing, and four Items linked to its
channels (see the [main widget's README](../navimow-widget/README.md#item-setup)):
`activity`, `control`, `battery-level` (plain `Number`, **not**
`Number:Dimensionless`) and `model`.

## Widget setup

1. In Main UI, go to **Developer Tools → Widgets → +**, paste the contents of
   [`widget.yaml`](widget.yaml) and save.
2. If you use the default target, also install the main
   [Navimow Mower Card](../navimow-widget/) (uid `navimow_mower_card`).
3. Add the widget as a cell in an `oh-grid-cells` block and set its
   parameters.

## Config parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `activityItem` | Item (String) | yes | `activity` channel |
| `batteryItem` | Item (Number) | yes | `battery-level` channel |
| `modelItem` | Item (String) | yes | `model` channel, selects the image |
| `controlItem` | Item (String) | yes | `control` channel; only forwarded to the extended widget |
| `modalType` | Choice | no | How the extended widget opens: `popup` (default, centered) or `sheet` (from the bottom) |
| `modalWidget` | Widget | no | Widget opened on click. Defaults to `widget:navimow_mower_card` |

## Choosing another extended widget

`modalWidget` accepts any widget. It is opened in a popup and receives four
props: `activityItem`, `controlItem`, `batteryItem` and `modelItem`, each set
to the Item you configured here, plus `maxWidth` (`32rem`, so the card fills
the popup; ignored by widgets without that prop). A custom widget must therefore declare props
with those names to use them.

## Model image

The image is chosen from the first 2 characters of the model (`X430` → `X4`).
Bundled categories: `H2`, `i1`, `i2`, `X3`, `X4`. Any other model shows a
generic default picture.
