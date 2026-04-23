# Tasks — DC Circuit Simulator
**Version 1.3** *(adds T89)*

### Legend
- ✅ Completed
- 🔄 Revised from original during implementation
- ➕ Added post-v1.0

---

## Phase 1: HTML Shell & Layout

**T01** ✅ Create the single HTML file with a `<!DOCTYPE html>` skeleton. Add the `<style>`, `<body>`, and `<script>` block stubs with section-header comments for all 15 modules.

**T02** ✅ Write the CSS layout: top bar (42px fixed height, full width), palette (152px fixed width), help panel (240px, right side), canvas (fills remaining width). Use CSS Grid on `<body>` with three columns. No scrollbars on any element. `box-sizing: border-box` globally.

**T03** ✅ Define all CSS custom properties (variables) for the color palette: background, grid dot color, wire color, accent color, component stroke color, voltage label color, current label color, error color, panel background, text color, probe color.

**T04** ✅ 🔄 Write the static HTML shell: `#topbar` with title, undo/redo buttons, zoom buttons, probe button, wire tool button, run button, tool indicator span, help button; `#palette` with one `.palette-item` div per component; `#canvas` element; `#error-banner` div; `#popover` div; `#help-panel` div.

**T05** ✅ Write `resizeCanvas()` and the `DOMContentLoaded` bootstrap stub that gets DOM refs, calls `resizeCanvas()`, and attaches the resize listener.

---

## Phase 2: Constants & State

**T06** ✅ Write the `// === CONSTANTS ===` block with all named constants: `GRID`, `SCALE_MIN`, `SCALE_MAX`, `TERMINAL_HIT`, `PIVOT_EPSILON`, `COMP_HALF`, `GROUND_OFFSET`, `GRID_HIDE_ZOOM`, `ZOOM_FACTOR`.

**T07** ✅ Write the `// === STATE ===` block: declare the `APP` object with all fields initialized to their default values. Include `deepClone(obj)` and `emptyCircuit()` utilities.

---

## Phase 3: Geometry Module

**T08** ✅ Write `worldToScreen`, `screenToWorld`, `snapToGrid`, `gridToWorld`, `screenToGrid`, `gridToScreen`.

**T09** ✅ Write `rotateOffset(dx, dy, rotation)` for 0/90/180/270 degree cases. Write `terminalPositions(component)` using `rotateOffset` and the per-type offset table from the spec.

**T10** ✅ Write `segmentHitTest(px, py, ax, ay, bx, by, threshold)` using point-to-segment distance formula.

**T11** ✅ Write `nearestTerminal(sx, sy, circuit, vp)`: iterate all components, compute terminal screen positions, return closest within `TERMINAL_HIT` px or null.

**T12** ✅ 🔄 Write `straightSegmentEnd(from, to, flipL)`: returns the single axis-locked endpoint one segment from `from` toward `to`. H-first if `|dx| >= |dy|`, V-first otherwise; `flipL` swaps the axis. **Replaces `lShapePoints` from plan v1.0.**

**T13** ✅ 🔄 Write `rerouteWireEnd(wire, oldPt, newPt)`: identifies whether `oldPt` matches the wire's first or last point, recomputes the end segment using `straightSegmentEnd`. Returns a new wire object.

**T14** ✅ Write `splitWireAtPoint(wire, gx, gy)`: finds the segment closest to `(gx, gy)`, snaps split point to segment axis, splits into two Wire objects. Returns `[wireA, wireB]`.

**T15** ✅ Write `midpointOfLongestSegment(wire)`: iterate segments, find longest by Euclidean length, return its midpoint in grid units.

**T16a** ✅ ➕ Write `nearestWireEndpoint(sx, sy)`: iterate all wire first/last points, return the closest within `TERMINAL_HIT` px in screen space, or null.

**T16b** ✅ ➕ Write `nearestWireSegment(sx, sy)`: iterate all wire segments, return the closest mid-segment snap point (axis-locked grid position) within 8px, excluding segment endpoints. Returns `{ wireId, gx, gy }` or null.

---

## Phase 4: Circuit Graph Module

**T16** ✅ Write `addComponent(circuit, type, gx, gy, rotation, params)`.

**T17** ✅ Write `deleteComponents(circuit, ids)`.

**T18** ✅ Write `moveComponent(circuit, id, gx, gy)`: uses `rerouteWireEnd` (now straight-segment based).

**T19** ✅ Write `setParam(circuit, id, key, value)`.

**T20** ✅ Write `addWire(circuit, points)`.

**T21** ✅ Write `deleteWires(circuit, ids)`.

**T22** ✅ Write `splitWire(circuit, wireId, gx, gy)`.

---

## Phase 5: Union-Find / Node Resolution

**T23** ✅ Write the union-find data structure: `makeUF`, `ufFind`, `ufUnion`.

**T24** ✅ Write `buildNodeMap(circuit, excludeCapacitors)`.

**T25** ✅ Write `findFloatingNodes(circuit, fullNodeMap)`.

**T25a** ✅ ➕ Write `resolveNodeToIds(circuit, nodeMap, nodeKey)`.

---

## Phase 6: MNA Solver

**T26** ✅ Write `makeMatrix(rows, cols)`.

**T27** ✅ Write `gaussianElim(A)`.

**T28** ✅ Write stamping logic inside `solve`.

**T29** ✅ Write `solve(circuit, nodeMap)`.

**T30** ✅ Verify solver against 9V battery + 1kΩ resistor: 9V node voltage, 9mA current.

---

## Phase 7: Simulation Orchestrator & SI Formatter

**T31** ✅ Write `formatSI(value, unit)`.

**T32** ✅ 🔄 Write `runSimulation()` and `setSimError(msg, ids)`. **Not called from `dispatch()`.**

**T32a** ✅ ➕ Update `dispatch()` to clear sim results and render without simulating.

---

## Phase 8: Component Symbols

**T33** ✅ Write `drawBattery(ctx, params)`.

**T34** ✅ Write `drawResistor(ctx)` — IEEE zigzag.

**T35** ✅ Write `drawCapacitor(ctx)`.

**T36** ✅ Write `drawInductor(ctx)` — IEEE humps.

**T37** ✅ Write `drawSwitch(ctx, isOpen)`.

**T38** ✅ Write `drawGround(ctx)`.

---

## Phase 9: Renderer

**T39** ✅ Write `applyViewport(ctx, vp)` and `render()` skeleton.

**T40** ✅ Write the grid renderer.

**T41** ✅ 🔄 Write the wire renderer: plain integer ID comparison for error highlight (not `wire_${wid}`).

**T42** ✅ 🔄 Write the wire-in-progress renderer: **shows one dashed straight segment** from last waypoint to axis-locked cursor position. No L-shape preview.

**T43** ✅ Write the component renderer.

**T44** ✅ Write the simulation label renderer.

**T45** ✅ Write the probe marker renderer.

**T46** ✅ Write the rubber-band selection rect renderer.

**T47** ✅ Write the palette drag ghost renderer.

---

## Phase 10: Undo / Redo

**T48** ✅ Write `pushUndo()`, `undo()`, `redo()`, `updateUndoButtons()`.

---

## Phase 11: Probe Tool

**T49** ✅ Write `probeClick(sx, sy)`.

**T50** ✅ Write `closeReadout(id)`.

**T51** ✅ Write `updateProbeReadouts()`.

---

## Phase 12: Parameter Popover

**T52** ✅ Write `openPopover(componentId, sx, sy)`.

**T53** ✅ Write `closePopover(save)`.

---

## Phase 13: Interaction

**T54** ✅ 🔄 Write `dispatch(action)`: clears sim results, does not call `runSimulation()`.

**T55** ✅ Write `hitTestComponentBody(sx, sy)`.

**T56** ✅ Write `hitTestWire(sx, sy)`.

**T57** ✅ Write `hitTestProbeClose(sx, sy)`.

**T58** ✅ 🔄 Write `onMouseDown(e)`: branches on active tool; wire tool waypoints are planted at `straightSegmentEnd(last, cursor)`, not raw grid position.

**T59** ✅ 🔄 Write `onMouseMove(e)`: uses `clientX/clientY` corrected by canvas bounding rect; guards hover/preview logic when mouse is outside canvas.

**T60** ✅ 🔄 Write `onMouseUp(e)`: uses `clientX/clientY` corrected by canvas bounding rect.

**T61** ✅ Write `onDblClick(e)`.

**T62** ✅ Write `onWheel(e)` and `zoomToPointer(sx, sy, delta)`.

**T63** ✅ 🔄 Write `onKeyDown(e)`: W (wire tool), S (select tool), Enter (run simulation). X toggles H/V axis for straight-segment wire drawing.

**T64** ✅ Write `initPalette()`.

**T65a** ✅ ➕ Write `findWireStartPoint(sx, sy)`: terminal snap > wire endpoint snap > free grid point. No `exitAxis` field.

**T65b** ✅ ➕ Write `tryLandWire(sx, sy)`: terminal snap > wire endpoint snap > wire mid-segment (T-junction). Calls `commitWire()`.

**T65c** ✅ ➕ Write `commitWire(last, target, wip)`: appends `straightSegmentEnd(last, target)` then `target` (deduped), dispatches `ADD_WIRE`.

---

## Phase 14: Integration & Wiring

**T65** ✅ Wire all top-bar buttons in bootstrap.

**T66** ✅ Wire `#error-banner`.

**T67** ✅ Wire tool indicator span via `setActiveTool()`.

**T68** ✅ 🔄 Wire tool toggles: `setActiveTool`, `toggleWireTool`, `toggleProbeTool`.

**T68a** ✅ ➕ Move `mousemove` listener from canvas to `window` so pan does not freeze when cursor leaves canvas. Use `clientX/clientY` minus canvas bounding rect in `onMouseMove` and `onMouseUp`.

---

## Phase 15: Help Panel

**T69a** ✅ Write help panel CSS.

**T69b** ✅ 🔄 Write help panel HTML: quick-start steps 1–5, Ground tip callout, **pan/zoom tip callout** (Space+drag / middle-mouse / scroll wheel), collapsible full reference.

**T69c** ✅ Wire help panel toggle in bootstrap.

---

## Phase 16: End-to-End Testing

**T70** ✅ **Test: Basic resistor circuit.** 9V + 1kΩ + ground. Press Run. Verify 9V, 9mA.

**T71** ✅ **Test: Voltage divider.** Two resistors in series. Verify midpoint voltage = `E × R2 / (R1 + R2)`.

**T72** ✅ **Test: Switch.** Open switch → floating node error with red highlight. Close → solves correctly.

**T73** ✅ **Test: Inductor.** Treated as short (0V drop).

**T74** ✅ **Test: Capacitor.** Floating node error with red highlight.

**T75** ✅ **Test: Probe tool.** Voltage and current readouts match sim values.

**T76** ✅ **Test: Undo/redo.** Three components → undo ×2 → one remains → redo → two.

**T77** ✅ **Test: Pan and zoom.** Pan outside canvas boundary during drag — viewport continues tracking. Grid hides below 40% zoom.

**T78** ✅ **Test: Wire T-junction.** Land wire on mid-segment → original splits → T-junction forms.

**T79** ✅ **Test: Wire endpoint-to-endpoint.** Two wire endpoints connect as single node.

**T80** ✅ **Test: Multi-wire node.** Three components to same junction. Same simulated voltage.

**T81** ✅ **Test: Component rotation.** R × 3 = 270°. Terminals correct. Wires re-route on move.

**T82** ✅ **Test: No ground error.** Banner message, no crash.

**T83** ✅ **Test: Singular matrix error.** Battery shorted directly.

**T84** ✅ **Test: Stale results cleared.** Labels disappear on circuit mutation without pressing Run.

**T85** ✅ **Test: Floating node highlight.** Error banner + red highlight on affected elements, not just coordinates.

**T86** ✅ ➕ **Test: Straight-segment wire routing.** Wire a rectangle of four components. Verify all segments are horizontal or vertical and none pass through component bodies. No auto-routing inward.

**T87** ✅ ➕ **Test: Waypoint planting.** Start wire, click on empty canvas to plant a turn, then land on terminal. Verify the planted waypoint is axis-locked (same x or same y as the preceding point).

**T88** ✅ ➕ **Test: Pan outside canvas.** Begin Space+drag, move mouse outside canvas boundary, continue moving — verify viewport tracks correctly. Release outside canvas — verify pan ends cleanly.

**T89** ✅ ➕ **Legibility pass.** Audit all UI text for contrast and size. Topbar button color lifted to full text color (#e6edf3), font 11→12px. Help panel step text color lifted to full text color, font 11→12px. Keyboard reference key badges and descriptions 10→11px, description color lifted to full text color. Tip callout color lifted from near-invisible `--text-dim` (#484f58) to `--text-muted` (#8b949e), font 10→11px. Section category labels color lifted from `--text-dim` to `--text-muted`. Palette labels 11→12px.

---

*End of Tasks v1.3 — 89 tasks across 16 phases*
