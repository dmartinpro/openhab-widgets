# Label-Card-Background

## Purpose

Tile with an image and a label under it; a tap opens the group popup of
`targetItem`. Used on the Overview page for Maison / Jardin / Voiture.

## Items / Groups used

- `targetItem` — group (Location/Equipment) opened via `action: group`

## Config parameters

`imagepath`, `desc`, `desccolor` (default theme text color), `targetItem`,
`height` (default `9rem`), `labelSize` (default `1.05rem`).

## Layout notes

Rewritten from nested `f7-card` + absolutely positioned overlay link: the
whole tile is now one `oh-link` (opacity 1, flex column). Image is a
`background-size: contain` flex child (`flex: 1 1 0; min-height: 0`), label
below it. Old version: 180px fixed, label above, `background-size: 90%`.

## Known issues / TODO

Live `overview` page passes no `height`, so it renders at 9rem. Original definition (180px, nested f7-card, absolute overlay link) was replaced on the server 2026-09-23. Note: `text:` does not render on a `div` here, use `content:`.
