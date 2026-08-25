# video-control-center

## Purpose

Minimal reusable Main UI widget embedding the Frigate web UI directly
in openHAB via a single `oh-webframe` (iframe) component. Simplified
from an earlier design (3 independent go2rtc/Frigate preview tiles,
each opening its own popup) down to just showing Frigate itself.

## Items / Groups used

None — purely URL-driven, no openHAB items bound.

## Config parameters

| Prop | Type | Default | Notes |
|---|---|---|---|
| `frigateUrl` | TEXT | — | required. URL of the Frigate web UI to embed (e.g. `http://frigate.local:5000`) |

## Layout notes

- The widget's root component is `oh-webframe` itself (no wrapping
  `div`) — nothing else needs to be laid out.
- `height: 800` (px, the component's own unit) is a fixed starting
  guess sized for viewing a dashboard-like page rather than a single
  camera row; adjust in `widget.yaml` once seen live. `oh-webframe`'s
  documented config only covers `src`/`height` (no percentage/flex
  sizing confirmed), so height is a hardcoded pixel value rather than a
  prop — add one later if it needs to vary per placement.
- `style: {width: 100%}` so the frame still fills whatever width its
  parent container gives it (a page, a card, a popup, etc.).

## Known issues / TODO

- **Not validated against a running openHAB 5.x instance** — no server
  access from this environment (see root [CLAUDE.md](../../CLAUDE.md)).
- **Frigate may block iframe embedding.** Some Frigate deployments set
  `X-Frame-Options`/CSP headers that prevent being embedded in an
  iframe from a different origin. If the frame shows blank/refused
  content, that's a Frigate-side setting to check — a reverse proxy
  stripping those headers would be the workaround (not fixable from
  this widget's YAML).
