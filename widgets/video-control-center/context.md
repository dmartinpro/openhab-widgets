# video-control-center

## Purpose

TBD — scaffold created, not yet designed. Fill in once the widget's
purpose is defined (e.g. "control panel for video cameras / NVR").

## Items / Groups used

TBD.

## Config parameters

None yet — `props.parameters` is empty in `widget.yaml`.

## Layout notes

Root is a plain `div` (flexbox column, `gap: 1rem`), mirroring the same
defensive width styling used in
[swimming-pool-manager](../swimming-pool-manager/widget.yaml) (`width:
100%`, `grid-column: 1 / -1`, `flex: 1 1 100%`, `box-sizing:
border-box`) so it renders full-width regardless of whether its parent
turns out to be a CSS grid, flex row, or plain block container. `slots.default`
is currently empty.

## Known issues / TODO

- Not yet designed — no items, sections, or controls defined.
