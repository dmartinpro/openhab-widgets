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

## Tips and best practices (learned from real widget work)

These are hard-won lessons from building and debugging widgets against a live
openHAB instance. Apply them proactively, not just when a bug matches.

### Expression evaluator limits

- openHAB's Main UI expression evaluator (the `=...` syntax) is **not** full
  JavaScript. Confirmed unsupported: `Object.keys`, `Object.values`, any
  `.members`/`.groupNames`/`.tags`/`.lastUpdate` on `items[x]` (only
  `{state, type, displayState}` is exposed). Confirmed supported:
  `Math.sin`/`cos`/`PI`, template literals, `JSON.stringify`, ternaries.
- There is no reactive clock/timer available to expressions — you cannot
  build a widget-side "N seconds since X" without a backing Item that
  actually changes over time. Don't propose designs that assume one.
- Because of these limits, prefer designs driven by **existing Item state**
  (e.g. comparing a computed command string against a `vars` value) over
  designs that need new supporting Items, timers, or backend rules — but
  don't force it if the evaluator genuinely can't express the logic;
  flag the limitation and propose the smallest addition needed instead.

### CSS / Framework7 gotchas

- `oh-link` renders as `<a class="link">`, and Framework7 ships default
  CSS for `.link`: `display: flex` and `:hover { opacity: 0.65 }`. Both are
  easy to forget and cause silent bugs — an unstyled `oh-link` used as a
  layout container can flex-shrink its children unexpectedly, and any
  `oh-link` without an explicit `opacity` will visibly fade on hover.
  **When using `oh-link` as a pure layout/click wrapper (not a real link
  affordance), explicitly set `opacity` and consider `display` rather than
  relying on the element having no default styling.**
- `box-sizing: border-box` on a bordered flex container shrinks the
  *content* box by the border width on each side. A child sized with an
  absolute unit (e.g. `6rem`) can then get silently flex-shrunk on one axis
  only, producing an asymmetric (oval, not circular) shape. Prefer sizing
  inner layers with `width: 100%; height: 100%` (relative to the actual
  parent content box) rather than a hardcoded absolute size duplicated
  across stacked layers.
- `clip-path: path("...")` on an SVG-positioned element works for both
  paint and accurate hit-testing in this stack — clicks outside the clipped
  shape but inside the bounding box do not trigger the element's action.
  Useful for non-rectangular click targets (e.g. pie/donut wedges).
- `oh-repeater` wraps its output in a `<div>` by default. Set
  `fragment: true` to suppress that wrapper — required when repeating
  inside `<svg>` (a stray `<div>` breaks SVG content), and whenever the
  repeated children need to individually participate in a shared CSS Grid
  stack (see below).

### CSS Grid overlay stacking

- To stack multiple elements on top of each other with CSS Grid, every
  element that should overlap must have its own `grid-column`/`grid-row`
  set directly — a wrapper `div` that positions *itself* inside an outer
  grid does **not** automatically make its own children grid-aware. Give
  the wrapper `display: grid` too, or set the placement directly on the
  elements that need to overlap.

### Multi-layer click tricks

- When stacking multiple `oh-link`/interactive layers to attach different
  side effects (e.g. one layer dispatches a real command, another only
  mutates a `vars` variable) to a single visual click target, remember the
  bubble order: the innermost element with `pointer-events: auto` fires
  **first**; each `pointer-events: none` ancestor's own `action` fires in
  turn as the event bubbles up, with the **outermost firing last**. Order
  variable-mutation layers with this in mind — a layer that must read a
  variable's pre-mutation value has to fire before the layer that mutates it.
- For debounce/anti-double-send logic, compare the **exact resolved
  target** (e.g. `"${targetItem}|${targetIds}"`), not just a coarse label
  like `activate`/`deactivate` — two different selections can otherwise
  collapse to the same label and get incorrectly deduplicated.

### YAML authoring

- Any `=`-prefixed expression containing a bare `: ` (e.g. inside a ternary
  with string literals) must be quoted, or YAML will parse the `:` as a new
  mapping key and fail with `mapping values are not allowed here`. Single-quote
  the whole expression and double up internal single quotes:
  `filter: '=cond ? ''value1'' : ''value2'''`.
- Before deploying, verify locally that the YAML parses and that every
  `=`-prefixed expression has balanced parens/quotes/ternaries. After
  deploying, diff the deployed JSON's expressions against the local file's
  expressions to confirm they match exactly (transcription/escaping errors
  are easy to introduce when hand-copying large expressions — prefer
  extracting them programmatically from the source of truth rather than
  retyping).
- A `#` preceded by whitespace starts a YAML comment even mid-line in an
  unquoted plain scalar — not just at line start. Any style value that
  embeds a hex color after a space silently loses everything from the `#`
  onward: `border: 1px solid #c7c7c7` parses as `1px solid` (color
  dropped), and `background: var(--f7-card-bg-color, #ffffff)` parses as
  `var(--f7-card-bg-color,` (truncated, syntactically broken CSS). This
  passes a plain `yaml.safe_load` structural check — the YAML is valid,
  just silently wrong — so it won't be caught by parse validation alone;
  grep for `' #[0-9a-fA-F]'` across style values and quote the whole value
  whenever a hex color follows a space (`border: '1px solid #c7c7c7'`).

### Diagnosing "is it the widget or the backend?"

- When a user reports behavior that looks like a widget bug, check
  `events.log` before assuming the widget is at fault: an `ItemCommandEvent`
  (source `org.openhab.ui=>...`) shows what the UI actually sent, while a
  later `ItemStateChangedEvent` (source `org.openhab.core.thing$<channel>`)
  shows what the binding subsequently reported. These can diverge — a
  binding can send/report a different state than what was commanded. Don't
  "fix" a backend bug by faking or overriding display logic in the widget;
  identify the actual source and say so, even if it means the fix is out of
  scope.

## Version control

The project is tracked with Git. Commit widget additions/changes with
messages that name the affected widget(s); avoid bundling unrelated widgets
in the same commit.
