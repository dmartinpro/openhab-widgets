# Diagral Alarm Control Widget: Build Instructions

## 1. Context: what you're building on top of

This widget controls a **Diagral home alarm system** through the `org.openhab.binding.diagral` openHAB binding. The binding is already built, live-tested, and stable. You are not modifying the binding, only building a MainUI widget that talks to the Items it exposes.

**Relevant Things and channels:**

| Thing | Channel | Type | Direction | Values |
|---|---|---|---|---|
| `alarm-system` (one per installation) | `mode-control` | String | Write (command) | `OFF`, `FULL`, `PRESENCE`, `PARTIAL1`, `PARTIAL2` |
| `alarm-system` | `armed-status` | String | Read | Same five values, plus undocumented transitional values (see section 7), purely informational for this widget |
| `group` (one per zone, up to 4) | `active` | Switch | Read/Write | `ON`/`OFF` |
| `group` | `status` | String | Read | `"Active"`/`"Inactive"`, redundant with `active`, ignore it |

Each `group` Thing also carries discovery-time properties: `armModes`, `inputDelay`, `outputDelay` (seconds). Not required for the core interaction, optional to surface as metadata.

**The two control paths:**
- `mode-control` arms/disarms the whole system via one of five fixed, installation-defined modes, each arming whatever fixed set of zones that mode was configured (in the Diagral installer app) to include. It cannot express an arbitrary user-chosen combination of zones.
- A `group`'s own `active` channel arms/disarms that one zone directly, independent of any mode, and works regardless of how the zone got into its current state (a named mode, or a prior direct activation). This is the only mechanism this widget needs, since every action in the state table below reduces to "activate/deactivate this specific set of zones."

**This widget never sends `mode-control` commands.** All activation and deactivation, including the "activate everything" and "deactivate everything" cases, goes through each zone's `active` channel. Confirmed as the intended design: turning `active` `ON` for every zone gives the same practical result as arming `FULL` via `mode-control`, and turning `active` `OFF` for whichever zones happen to be active gives the same result as a global disarm, without needing a second code path.

## 2. Functional requirements

- A central round button, surrounded by one button per configured zone (2 to 4 zones, see section 4), arranged in a ring around it.
- Clicking a zone button toggles its **selection**, a purely client-side UI state, not an openHAB Item. Clicking an already-selected button deselects it. Selection alone never triggers an activate/deactivate action.
- Clicking the central button performs whatever single, unambiguous action it's currently proposing.

**State table for the central button:**

| Selection | Zone states | Central button proposes | Action on click |
|---|---|---|---|
| None selected | All zones inactive | Activate | Send `ON` to every zone's `active` channel |
| None selected | One or more zones active | Deactivate | Send `OFF` to every currently-active zone's `active` channel (only the active ones, inactive zones are left alone) |
| One or more selected | All selected zones inactive | Activate | Send `ON` to each selected zone's `active` channel |
| One or more selected | All selected zones active | Deactivate | Send `OFF` to each selected zone's `active` channel |
| One or more selected | Mixed (some selected active, some not) | Disabled | No-op, not clickable |

Deselecting is implied by "clicking selects it": clicking a currently-selected button removes its selection.

## 3. Visual design spec

Zone button, four states from two independent facts (active/inactive, selected/not):

| Active | Selected | Background | Border |
|---|---|---|---|
| No | No | none | none |
| Yes | No | configurable `activeColor` | none |
| No | Yes | none | configurable `selectedBorderColor` |
| Yes | Yes | configurable `activeColor` | configurable `selectedBorderColor` |

Each zone button also shows a label and an icon representing that real-world zone (both supplied per-zone in configuration, see section 4).

Central button: a single icon whose **tint** changes across its three states (e.g. one color for "will activate," a different color for "will deactivate," and a dimmed/grey tint plus reduced opacity and `cursor: not-allowed` for "disabled/mixed"). No icon swap, no shape change, just color state. Make the two active-state tints configurable colors as well, consistent with how the zone colors are configurable.

## 4. Widget configuration schema

A standalone/reusable openHAB MainUI widget (YAML widget DSL, registered under the UI's Widgets developer section), with:

- **`zones`**: a list of 2 to 4 entries, each with:
  - `activeItem` (string, item picker): the Item linked to that zone's `active` channel
  - `label` (string): display label for that zone
  - `icon` (string): an f7 icon name for that zone
- **`activeColor`** (color picker): background for an active zone button
- **`selectedBorderColor`** (color picker): border for a selected zone button
- **`activateColor`** (color picker, optional): central button tint when proposing activate
- **`deactivateColor`** (color picker, optional): central button tint when proposing deactivate

Cap the `zones` list at 4 entries in the config UI (or just document the cap and let a 5th entry silently overflow the ring layout, your call, but the ring math in section 5 assumes 4 as the max). No `mode-control` or `armed-status` item is required by the core interaction; add `armedStatusItem` only if you also want an optional status label somewhere in the widget, purely cosmetic.

Since `zones` is a small, fixed-size structured list (max 4), check whether your openHAB version's widget config editor handles a nested array-of-objects parameter directly. If not, a `TEXT` parameter holding a JSON array is an acceptable fallback, less friendly to hand-edit but fully sufficient at this size.

## 5. Technical architecture recommendation

Build this as an openHAB MainUI YAML widget (`oh-*`/`f7-*` component DSL), not a bespoke Vue app.

- **Selection state**: hold it in the widget's `vars:` block (widget-instance-local reactive state), e.g. `vars.selected` as an array of selected zone indices. Never back this with a real Item, it's pure UI state.
- **Ring layout**: `oh-repeater` over the configured `zones` list (2 to 4 items). Position each button at an angle of `360deg / count * index` around a fixed-size circular container, via `transform: rotate(angle) translate(radius) rotate(-angle)` on each button's wrapper. The central button sits absolutely centered in the same container. With a max of 4 zones, this stays simple, no need to handle crowding/scaling logic for larger counts.
- **Central button logic**: a computed expression reading `vars.selected` plus the current `active` state of each selected (or, if none selected, each) zone, producing one of three discrete outputs (`activate` / `deactivate` / `disabled`) per the state table, driving both tint and the click action.
- **Actions**: zone buttons mutate `vars.selected` only (no Item command). The central button issues real `ON`/`OFF` commands to the relevant `activeItem`(s) per the state table.

## 6. Interaction details worth getting right

- After the central button is clicked, briefly disable it (or show a pending/spinner treatment) rather than letting it immediately flip to the opposite proposed state, since the real system takes time to respond (section 7). Let it re-enable naturally once the real Item states update.
- Selection state doesn't need to persist beyond the widget instance's lifetime, no need to store it anywhere durable.

## 7. Real-world behavior to design around

This binding was extensively live-tested against a real Diagral installation:

- **Arming/disarming is not instant.** After a command, the real system can take up to a zone's configured exit/entry delay (commonly around 90 seconds, but per-zone configurable) before settling. The binding's own internal state derivation already handles this correctly, you don't need to replicate any of that logic, just don't assume the UI reflects a command's effect immediately.
- **Polling, not push.** The binding polls (default every 60 seconds), with an immediate out-of-cycle poll fired right after any command. Expect state updates within a few seconds after a click, not instantly.
- **Zones armed from elsewhere** (e.g. the official e-ONE app) are picked up correctly on the next poll, same as anything commanded from this widget, nothing special needed.
