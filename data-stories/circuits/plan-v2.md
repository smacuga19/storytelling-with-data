# Plan — DC Circuit Simulator
**Version 2.1** *(adds ADR-07 and ADR-08)*

This document records the architectural decisions that diverged from Plan v1.0 during implementation. It is not a full restatement of the original plan — read this alongside plan.md. Sections not listed here are unchanged from v1.0.

---

## ADR-01: Manual Simulation Trigger

**Decision:** `runSimulation()` is no longer called from `dispatch()`. Simulation runs only when the user explicitly presses ▶ Run or Enter.

**Rationale:** Auto-simulation on every mutation produced confusing error messages mid-edit (e.g. "floating node" while the user was still placing components). Errors should only appear when the user has declared the circuit ready.

**Consequences for plan.md §8 (Simulation Orchestrator):**

`dispatch()` no longer ends with `runSimulation()`. Instead it ends with:
```javascript
APP.simResult = null;
APP.simError  = null;
APP.simErrorIds = [];
updateProbeReadouts();
render();
```

`runSimulation()` is wired directly to the ▶ Run button and the Enter key. `rotateComponent()` follows the same pattern.

**New top-bar button:** `#btn-run` — styled in green to stand out as the primary action. Keyboard shortcut: Enter (guarded against active text inputs).

---

## ADR-02: Explicit Wire Tool

**Decision:** The Wire tool is a first-class persistent tool, activated by **W** / toolbar button, not implicitly triggered by clicking a terminal.

**Rationale (two problems solved):**
1. In the original plan, clicking near a terminal in Select mode silently switched to Wire mode. This made it impossible to select a component whose terminal was near the click. An explicit tool eliminates the ambiguity.
2. With an explicit wire tool, the user can start a wire from any grid point, not just component terminals — enabling free-form wiring and junction-to-junction connections.

**New functions (additions to plan.md §15.2):**

```javascript
findWireStartPoint(sx, sy)
// Priority: terminal snap > wire endpoint snap > free grid point
// Returns a {x,y} grid coordinate to begin the wire from

tryLandWire(sx, sy)
// Called on each click while a wire is in progress
// Priority: terminal snap > wire endpoint snap > wire mid-segment (T-junction)
// Calls commitWire() on success; returns boolean

commitWire(last, target, wip)
// Appends one straight segment from last to target, dispatches ADD_WIRE

nearestWireEndpoint(sx, sy)
// Returns closest wire first/last point within TERMINAL_HIT px, or null

nearestWireSegment(sx, sy)
// Returns { wireId, gx, gy } for the closest mid-segment snap point
// within 8px screen space, axis-locked to segment, excluding endpoints
```

**Tool state machine:**

```
          W / Wire btn          S / Esc (not drawing)
Select ──────────────► Wire ◄──────────────────────────
  ▲                     │  \
  │    S / P             │   Esc (while drawing)
  │                     │    cancels wire, stays in Wire
  └─────────────────────┘
         P / Probe btn
Select ──────────────► Probe
  ▲                     │
  └─────────────────────┘
         S / W / Esc
```

**`onMouseDown` restructure:** The original plan had a single hit-test cascade for all cases. The new version branches first on `APP.activeTool`:

```
onMouseDown:
  if probe: probeClick(); return
  if pan condition: begin pan; return
  if wire tool:
    if drawing: tryLandWire() OR plant axis-locked waypoint
    else: findWireStartPoint(), begin wire
    return
  // select tool:
  hitTest cascade: component → wire → rubber-band
```

**`onMouseUp` simplification:** Wire completion no longer happens on mouseup. `tryLandWire()` is called on mousedown clicks instead, making the wire model click-to-click rather than press-and-release.

---

## ADR-03: Multi-Wire Node Connections

**Decision:** Wires can start and land on existing wire endpoints and mid-segments, not only on component terminals.

**Rationale:** The original spec and plan only allowed terminal-to-terminal connections. This made it impossible to build circuits with T-junctions without first having a component at the junction, which is not how real schematic editors work. Multiple wires must be able to meet at a node.

**Implementation:**

The landing priority order (terminal > wire endpoint > wire mid-segment) ensures that component terminals are always preferred when ambiguous, while still allowing junction connections. The `nearestWireEndpoint` and `nearestWireSegment` helpers handle the two new connection types.

Wire mid-segment landing delegates to the existing `SPLIT_WIRE` dispatch action before committing the new wire, so the T-junction is formed atomically and is undoable as a single action.

---

## ADR-04: Floating Node Error Highlights Elements, Not Coordinates

**Decision:** When a floating node is detected, `simErrorIds` is populated with the actual component IDs and wire IDs that belong to the floating node, not the node's canonical coordinate string.

**Rationale:** Displaying a coordinate like `"Node at (12, 4) is floating"` requires the user to mentally locate that position on the canvas. Highlighting the actual wires and components in red is immediate and unambiguous.

**New function (addition to plan.md §8):**

```javascript
resolveNodeToIds(circuit, nodeMap, nodeKey)
// Input:  a node's canonical "x,y" key
// Output: array of component IDs and wire IDs (plain integers)
//         whose terminals or waypoints belong to that node
// Used:   to populate APP.simErrorIds before calling setSimError()
```

**Wire renderer fix:** The original plan called for `APP.simErrorIds.includes(`wire_${wid}`)`. This was corrected to `APP.simErrorIds.includes(parseInt(wid))` so that wire IDs (integers) match the plain integers stored by `resolveNodeToIds`.

---

## ADR-05: Help Panel

**Decision:** A collapsible help panel is added as a third column in the body grid layout.

**Rationale:** The simulator has enough interaction surface (three tools, 15+ shortcuts, DC component behaviours) that new users need in-app guidance. A panel is preferable to a modal because it can stay open while working.

**Layout change (addition to plan.md §17):**

Body grid gains a third column:
```css
grid-template-columns: var(--palette-w) 1fr auto;
grid-template-areas:
  "topbar  topbar  topbar"
  "palette canvas  help";
```

`#help-panel` width transitions between 240px (open) and 0px (collapsed) via CSS `transition: width 220ms`. The canvas `1fr` column fills the remaining space naturally.

**Structure:**
- Quick-start: 5 numbered steps + two tip callouts (Ground/DC path requirement; pan and zoom controls), always visible when panel is open
- Full Reference: collapsed `<div>` toggled by a button within the panel; contains keyboard shortcut tables grouped by category
- Both sections are pure HTML/CSS — no canvas rendering involved

**State:** Panel open/closed state is not part of `APP` or the undo stack. It is purely a DOM class toggle on `#help-panel`.

---

## ADR-06: `setActiveTool` Manages All Tool Button States

**Decision:** A single `setActiveTool(t)` function is the sole place that updates cursor style, tool indicator text, and button highlight states for all three tools.

**Rationale:** In plan v1.0, tool state updates were scattered across event handlers. Centralising them eliminates the possibility of a button appearing active while the tool is not, or vice versa.

**Implementation:**

```javascript
function setActiveTool(t) {
  APP.activeTool = t;
  // update tool indicator text
  // update canvas cursor class
  // toggle .active on #btn-probe, #btn-wire-tool
  // (select tool has no dedicated button to highlight)
}
```

`toggleWireTool()` and `toggleProbeTool()` are thin wrappers that call `setActiveTool` and handle any tool-specific teardown (clearing `APP.wireInProgress`, `APP.probeAnchor`).

---

## ADR-07: Straight-Segment Wire Model Replaces L-Shape Auto-Routing

**Decision:** `lShapePoints()` is replaced by `straightSegmentEnd()`. Each click lays exactly one straight horizontal or vertical segment. The user controls every bend explicitly by planting intermediate waypoints.

**Rationale:** The L-shape auto-router chose a corner direction based on relative cursor displacement, which frequently routed wires through the interior of the circuit rather than around it. This produced the "inward routing" problem visible in the screenshot: a rectangle of components wired with segments that cut through the centre instead of following the perimeter. Straight-segment-per-click is the standard interaction model used by KiCad, Logisim, and most professional schematic editors. It gives the user full geometric control and requires no heuristics.

**Replaced function:**
```javascript
// REMOVED:
function lShapePoints(from, to, flipL, exitAxis)
// returned [from, corner, to] — three points, two segments, auto-routed

// REPLACED WITH:
function straightSegmentEnd(from, to, flipL)
// returns one endpoint: the axis-locked position one segment from `from` toward `to`
// H-first if |dx| >= |dy|, V-first otherwise; flipL swaps the axis
```

**Wire preview:** The in-progress wire renderer now shows only the committed waypoints plus one straight dashed segment from the last waypoint to the axis-locked cursor position. No second segment is shown speculatively.

**Waypoint planting:** Clicking on empty canvas while drawing plants a waypoint at `straightSegmentEnd(last, cursor, flipL)` — the axis-locked position — not the raw grid position. This ensures every stored waypoint lies on a horizontal or vertical line from its predecessor.

**`commitWire` change:** Instead of appending an L-shape (two segments), `commitWire` appends `straightSegmentEnd(last, target)` and then `target`, deduplicating if they are the same point (i.e. the target was already axis-aligned).

**`rerouteWireEnd` change:** When a component moves, the rerouted wire end is computed with `straightSegmentEnd` rather than `lShapePoints`, maintaining the one-segment-per-section invariant.

**`findWireStartPoint` simplification:** The `exitAxis` field that previously forced the first segment to leave along the terminal's natural axis is removed. It was a heuristic that conflicted with the straight-segment model. Users explicitly lay the first segment in the direction they choose.

**X key behaviour unchanged:** X still toggles `wip.flipL`, swapping between H-first and V-first for the current segment. The live preview updates immediately.

---

## ADR-08: Mouse Event Listeners Moved to Window for Reliable Pan

**Decision:** `mousemove` is attached to `window` instead of the canvas element. `onMouseMove` and `onMouseUp` use `clientX/clientY` corrected by the canvas bounding rect instead of `offsetX/offsetY`.

**Rationale:** When `mousemove` was on the canvas, dragging the mouse outside the canvas boundary during a pan caused events to stop firing, freezing the viewport mid-drag. This is a fundamental browser constraint: `offsetX/offsetY` events are only delivered to the registered element, and `offsetX/offsetY` is only reliable when the event target is the element itself.

**Changes to plan.md §16 (Bootstrap):**

```javascript
// BEFORE:
APP.canvas.addEventListener('mousemove', onMouseMove);
window.addEventListener('mouseup', onMouseUp);

// AFTER:
window.addEventListener('mousemove', onMouseMove);
window.addEventListener('mouseup', onMouseUp);
```

**Coordinate correction in `onMouseMove` and `onMouseUp`:**
```javascript
// BEFORE (unreliable when mouse outside canvas):
const sx = e.offsetX, sy = e.offsetY;

// AFTER:
const rect = APP.canvas.getBoundingClientRect();
const sx = e.clientX - rect.left;
const sy = e.clientY - rect.top;
```

**Guard for hover/wire-preview logic:** Non-pan logic (hover state update, wire-in-progress preview) is skipped when the mouse is outside the canvas bounding rect. Only pan continues to work outside the canvas bounds, which is the intended behaviour.

---

## Module Order (updated)

The build order from plan.md §18, updated through v2.1:

1. Constants
2. State
3. Geometry *(includes `straightSegmentEnd`, `nearestWireEndpoint`, `nearestWireSegment`; `lShapePoints` removed)*
4. Circuit Graph
5. Union-Find / Node Resolution *(includes `resolveNodeToIds`)*
6. MNA Solver
7. Simulation Orchestrator *(no longer calls `runSimulation` from `dispatch`)*
8. SI Prefix Formatter
9. Component Symbols
10. Renderer *(wire preview shows single straight segment)*
11. Undo / Redo
12. Probe Tool
13. Parameter Popover
14. Interaction *(includes `findWireStartPoint`, `tryLandWire`, `commitWire`, `toggleWireTool`; waypoints axis-locked on plant)*
15. Bootstrap *(`mousemove` on `window`; wires ▶ Run, Wire tool, Help panel toggle)*

---

*End of Plan v2.1*
