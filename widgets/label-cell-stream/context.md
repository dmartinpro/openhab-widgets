# Label-Cell-Stream

## Purpose

Cell-sized tile (same metrics as `Label-Cell-Background` / `oh-label-cell`)
whose background is a live low-fps image stream (Frigate MJPEG). Title
top-left; tap navigates to `actionPage`.

## Items / Groups used

None.

## Config parameters

`streamUrl` (required), `desc`, `actionPage`, `overlay` (default 0.35).

## Layout notes

CSS grid stack (no absolute positioning): `img` (`object-fit: cover`), dark
overlay `div`, title `div`, all on `grid-area: 1 / 1`, painted in DOM order.
Root `oh-link` (opacity 1) is the click target. Card background color shows
through if the stream fails to load.

## Known issues / TODO

Needs the browser to be allowed to fetch the stream (Frigate sits behind the
same auth; works when logged in). The stream runs for as long as the page is
open; keep `fps` low in the URL. Do not put real camera hostnames/URLs in this
repo; they are set only in the deployed page.
