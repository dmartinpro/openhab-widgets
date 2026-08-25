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

## Dedicated full-screen Page (2026-08-25)

Added [page.yaml](page.yaml), a standalone Main UI **Page** (not a
widget) that embeds this widget via `component: widget:video_control_center`.

**Schema correction (2026-08-25):** the first version of this file used
`uid` / `tags` / `component: oh-layout-page` / `slots.default` at the
top level — a structure inferred from a community post about the
*file-provisioning* format (`conf/pages/*.yaml`, wrapped in a
`version`/`pages` map). The actual single-Page YAML shown by Main UI's
own code editor (confirmed live by the user) has **no `uid`/`component`
at the top at all** — a page's id lives outside its YAML, managed by
the Pages list itself. Instead the real top-level shape is:
```yaml
config:
  label: ...
  icon: ...
blocks: []
masonry: []
grid: []
canvas: []
```
— one array per layout mode Main UI supports (stacked blocks, masonry
grid, fixed grid, freeform canvas), all present even when unused. Our
content (the `oh-block` wrapping `widget:video_control_center`) now
lives under `blocks`, the standard "Responsive Layout" mode; the other
three arrays are left empty.

This page still exists for the same reason as before: a Page opened via
a Group/Equipment's automatic "tap to open
default widget" is wrapped in a **popup**, and Main UI popups are
capped at a fixed ~630×630px dialog on tablet/desktop screens (full
screen only on phones) — far too small for browsing Frigate's own
dashboard, and not something fixable from inside a widget's own YAML
(it would take either a global CSS override affecting every popup in
the install, or hand-rolling a custom-sized `f7-popup`, abandoning the
standard popup mechanism). A full Page has no such size cap — it's
always full-viewport on every device — so it's the simpler fix for this
specific "show a whole embedded dashboard" use case.

**How to use it:** paste `page.yaml`'s content into a new Page's YAML
code editor in Main UI settings (Pages are provisioned the same
copy-paste way as this project's widgets — see root
[CLAUDE.md](../../CLAUDE.md)), *after* `video_control_center` has
already been created as a widget with that exact uid, since the page
references it by `widget:video_control_center`. Update the
placeholder `frigateUrl` value in `page.yaml` (currently
`http://frigate.local:5000`) to the real Frigate address before use —
it's set directly in the page's widget instantiation, not re-prompted
at runtime.

**Where this file lives:** kept inside `widgets/video-control-center/`
rather than under `sitemaps/` — the root CLAUDE.md reserves `sitemaps/`
for compositions that reference *multiple* widgets, whereas this page
exists solely as a full-screen wrapper for this one widget. If more
standalone Pages get added later across the project, consider
promoting this to a dedicated top-level `pages/` folder instead for
consistency.

## Known issues / TODO

- **Not validated against a running openHAB 5.x instance** — no server
  access from this environment (see root [CLAUDE.md](../../CLAUDE.md)).
  `page.yaml`'s top-level shape (`config`/`blocks`/`masonry`/`grid`/
  `canvas`) has since been confirmed against a live Page code editor
  (see the 2026-08-25 schema correction above), but the `oh-block` +
  `widget:video_control_center` content nested inside `blocks` hasn't
  been seen rendered yet.
- **Frigate may block iframe embedding.** Some Frigate deployments set
  `X-Frame-Options`/CSP headers that prevent being embedded in an
  iframe from a different origin. If the frame shows blank/refused
  content, that's a Frigate-side setting to check — a reverse proxy
  stripping those headers would be the workaround (not fixable from
  this widget's YAML).
