# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This project is dedicated to advanced configuration of **openHAB 5.x** and to
building **Main UI widgets** (YAML-based, using `f7-*`, `oh-*`, and custom
widget components). There is no separate application code here — the
deliverables are YAML widget definitions, sitemaps, and the documentation
that goes with them.

## Technical context

- **openHAB 5.x**, running on **Java 21**
- UI layer built on **Framework7 v7**
- openHAB's frontend is mid-migration from **Vue 2 to Vue 3** — behavior can
  shift between minor 5.x releases; flag anything version-sensitive
- Widgets are authored in **YAML** using the standard Main UI structure:
  `component` / `config` / `slots`
- No direct network access to a running openHAB server from this
  environment — YAML is authored and reviewed here, then copied manually
  into openHAB by the user. Do not assume the ability to query the REST API,
  validate against live items, or hot-reload a sitemap.

## Layout and styling conventions

These are the house rules for any widget YAML generated or edited here:

- **Minimize `f7-block` / `f7-row` / `f7-col`** — use them only when
  matching the Main UI's native look and feel is the actual goal (e.g.
  standard card padding/spacing). Reach for plain `div` (via
  `component: "oh-container"` or similar) whenever fine-grained control is
  needed.
- **Style inline, not via F7 classes.** Prefer the widget's `style` object
  over Framework7 utility classes — styling should be explicit and
  self-contained in the YAML rather than depending on F7's class behavior.
- **Flexbox is the primary layout tool** for alignment and sizing
  (`display: flex`, `flex-direction`, `justify-content`, `align-items`,
  `gap`, `flex: ...`). Reach for it before anything else.
- **No absolute positioning.** Avoid `position: absolute` / `fixed`; prefer
  relative sizing and `calc()` expressions when a dimension needs to be
  derived (e.g. `calc(100% - 2rem)`).

When generating or fixing widget YAML, apply these rules by default without
being asked each time. If a request conflicts with one of them (e.g. asks
for absolute positioning), flag the conflict and propose the flexbox/`calc()`
equivalent first.

## Repository structure

```
/widgets/<widget-name>/
    widget.yaml       # the widget's component/config/slots definition
    context.md         # what this widget is for — see docs/widget-context-template.md
/sitemaps/
    *.yaml / *.sitemap  # sitemap definitions
/docs/
    widget-context-template.md  # template to copy for each new widget's context.md
CLAUDE.md
```

- **Every widget lives in its own folder** under `widgets/`, alongside a
  `context.md` describing its purpose, the items/groups it binds to, its
  config parameters, and any non-obvious layout decisions. Use
  [`docs/widget-context-template.md`](docs/widget-context-template.md) as
  the starting point for new widgets.
- When creating a new widget, always create both files together —
  `widget.yaml` without a `context.md` is considered incomplete.
- Sitemaps that reference multiple widgets live under `sitemaps/`, not
  inside a widget folder.

## What Claude should do here

- **Generate and fix Main UI widget YAML** following the conventions above.
- **Debug layout issues** — structure, flexbox hierarchy, alignment,
  responsive behavior — by reasoning through the YAML rather than assuming
  visual output; ask the user to describe or screenshot the rendered result
  when behavior is ambiguous, since there's no live server to check against.
- **Propose sitemaps or custom widgets** tailored to the user's items and
  groups, once those are described.
- **Surface openHAB 5.x-relevant changes** (Framework7 v7 quirks, Vue 2→3
  migration fallout, breaking changes in `oh-*` components) when they're
  relevant to the task at hand — this is a fast-moving area and behavior
  isn't guaranteed stable across patch releases.

## Version control

The project is tracked with Git. Commit widget additions/changes with
messages that name the affected widget(s); avoid bundling unrelated widgets
in the same commit.
