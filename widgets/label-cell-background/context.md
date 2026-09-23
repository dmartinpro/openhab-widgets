# Label-Cell-Background

## Purpose

Same look and size as `oh-label-cell` / the other cell widgets (120px, card
radius and shadow), with a photo as background. Title top-left, optional
state value bottom-left; a tap opens the group popup of `targetItem`.
Alternative to `Label-Card-Background`, which is left untouched by this widget.

## Items / Groups used

- `targetItem` — group opened via `action: group`
- `stateItem` — optional, shown as the value line (`displayState`)

## Config parameters

`desc`, `imagepath`, `targetItem`, `stateItem`, `imageFit` (`cover` default,
`contain` for cut-out PNGs), `overlay` (default 0.45; 0 is treated as unset,
use 0.01 for none).

## Layout notes

One `oh-link` (opacity 1, flex column) carrying the background: a dark
gradient over `url(image)`, so no stacked layers or absolute positioning.
Text is always white, readable thanks to the overlay and a text-shadow.
Meant to be placed inside `oh-grid-cells`.

## Known issues / TODO

Not yet checked on a light theme. `background-color` (theme card color) is the fallback behind transparent PNGs; the Tesla cut-out looks better with
`imageFit: contain`.
