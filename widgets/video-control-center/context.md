# video-control-center

## Purpose

Reusable Main UI widget showing 3 clickable camera preview tiles (fed by
go2rtc/Frigate or any similar streaming source). Tapping a tile opens a
popup with an embedded iframe (`oh-webframe`) pointing to a
separately-configured URL — typically Frigate's live camera view — so
the user can drill into the full Frigate UI without leaving openHAB.

## Items / Groups used

None — this widget is purely URL-driven (no openHAB items bound). Each
of the 3 tiles is configured independently via props:

- **Stream URL** — what the tile's live preview displays.
- **Stream type** — `mjpeg` / `hls` / `webrtc`, picked from a dropdown;
  determines which renderer is used for Stream URL (see Layout notes).
- **Click-through URL** — opened in the popup iframe when the tile is
  tapped (e.g. `http://frigate.local:5000/cameras/front_door`).

## Config parameters

| Prop | Type | Default | Notes |
|---|---|---|---|
| `camera1StreamUrl` / `camera2StreamUrl` / `camera3StreamUrl` | TEXT | — | required |
| `camera1StreamType` / `camera2StreamType` / `camera3StreamType` | TEXT (dropdown: `mjpeg`\|`hls`\|`webrtc`) | `mjpeg` | required |
| `camera1RedirectUrl` / `camera2RedirectUrl` / `camera3RedirectUrl` | TEXT | — | required |

Grouped into `camera1`/`camera2`/`camera3` parameter groups so the
widget's configuration screen shows the 3 cameras as separate sections.

## Layout notes

- Root `div` carries the same defensive full-width styling used in
  [swimming-pool-manager](../swimming-pool-manager/widget.yaml)
  (`width: 100%`, `grid-column: 1 / -1`, `flex: 1 1 100%`,
  `box-sizing: border-box`) — applied proactively this time, since that
  widget only got it after hitting narrow-rendering issues inside a
  popup/grid parent (see its `context.md`).
- The 3 tiles sit in a `display: flex; flex-wrap: wrap; gap: 1rem` row
  (`flex: 1 1 260px` each), so they sit side by side and wrap to a new
  row on narrow screens — no fixed/absolute sizing.
- **Dynamic stream rendering**: each tile has 3 sibling media
  components (`img`, `oh-video` with `playerType: videojs`, `oh-video`
  with `playerType: webrtc`), each gated by a `visible` expression
  comparing `props.cameraNStreamType` — only one ever renders at a time.
  This was a deliberate choice (discussed with the user) to support all
  three stream formats from one widget rather than picking one, since
  go2rtc/Frigate setups vary in what they expose (MJPEG snapshot/stream
  endpoint, an `.m3u8` HLS playlist, or go2rtc's native WebRTC).
  `mjpeg` also renders when `camera1StreamType` is unset, so the prop's
  own `defaultValue: mjpeg` isn't the only fallback.
- **Click-to-popup uses `popupOpen`/`popupClose` + an inline `f7-popup`**,
  not `action: popup`/`actionModal`. `actionModal` can only reference an
  *already registered* Page or personal Widget by id (confirmed against
  openHAB's own docs) — it cannot point at an anonymous component
  defined inline in the same file. Since each of the 3 tiles needs a
  *different* iframe target and the whole thing should stay one
  self-contained widget (not 3 extra registered popup widgets), the
  `popupOpen: '.class-name'` / inline `f7-popup` pattern — taken
  directly from openHAB's own
  [Personal Widgets](https://www.openhab.org/docs/ui/personal-widgets.html)
  documentation example — is the only mechanism that fits. Each tile is
  wrapped in an `f7-link` (confirmed in that same doc example to support
  `popupOpen`) rather than a plain `div`, since `popupOpen`/`popupClose`
  are Framework7 Link/Button props, not generic HTML attributes a raw
  `div` would be guaranteed to honor.
- Popup close button uses `oh-link` + `popupClose`, matching the same
  documented example.
- Each popup's `oh-webframe` has `height: 600` (px, the component's own
  unit) — adjust if 600px doesn't suit the popup's actual rendered size
  once tested live (`oh-webframe` config only documents `src`/`height`,
  no percentage/flex sizing confirmed).

## Known issues / TODO

- **Not validated against a running openHAB 5.x instance** — no server
  access from this environment (see root [CLAUDE.md](../../CLAUDE.md)).
  In particular:
  - `playerType: webrtc` on `oh-video`: openHAB's documentation doesn't
    specify the exact signaling protocol/API contract this expects, so
    whether it's compatible with go2rtc's own WebRTC endpoint is
    unconfirmed — test this stream type first if you plan to use it.
  - `popupOpen` on `f7-link` is demonstrated in openHAB's own docs, but
    the resulting *visual* layout (whether the whole tile — not just an
    `f7-link`'s usual text — is properly clickable and correctly
    triggers the popup) hasn't been seen rendered.
- **Frigate may block iframe embedding.** Some Frigate deployments set
  `X-Frame-Options`/CSP headers that prevent being embedded in an
  iframe from a different origin. If the popup shows a blank/refused
  frame, this is a Frigate-side setting to check (not fixable from the
  widget YAML) — a reverse proxy stripping those headers, or navigating
  to Frigate directly (`action: url` instead of the iframe popup), would
  be the workarounds.
- MJPEG via a plain `<img src="...">` only works if the URL serves
  either a static image or a multipart MJPEG stream — an RTSP URL or a
  raw `.m3u8`/WebRTC endpoint will not render this way (that's what the
  Stream type selector is for).
