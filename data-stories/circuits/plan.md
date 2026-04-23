# Plan — DC Circuit Simulator
**Version 1.0**

---

## 1. File Structure

The entire application lives in one HTML file with three clearly delimited sections:

```
index.html
├── <style>        — all CSS, CSS variables, layout
├── <body>         — static HTML shell (palette, topbar, canvas)
└── <script>       — all JavaScript, organized into modules by convention
```

The script section is organized into the following logical modules, each separated by a heading comment:

```
// === CONSTANTS ===
// === STATE ===
// === GEOMETRY ===
// === CIRCUIT GRAPH ===
// === UNION-FIND / NODE RESOLUTION ===
// === MNA SOLVER ===
// === SIMULATION ===
// === SI FORMATTER ===
// === COMPONENT SYMBOLS ===
// === RENDERER ===
// === UNDO / REDO ===
// === PROBE TOOL ===
// === PARAMETER POPOVER ===
// === INTERACTION ===
// === BOOTSTRAP ===
```

---

## 2. Constants

All magic numbers are named constants at the top of the script:

```javascript
const GRID          = 20       // px per grid unit at scale 1.0
const SCALE_MIN     = 0.1
const SCALE_MAX     = 10.0
const TERMINAL_HIT  = 8        // px screen-space snap radius for terminals
const PIVOT_EPSILON = 1e-12    // singularity threshold in Gaussian elimination
const COMP_HALF     = 2        // terminal offset in grid units for standard components
const GROUND_OFFSET = {x:0, y:-1}  // terminal offset for ground component
const GRID_HIDE_ZOOM = 0.4     // hide grid dots below this scale
```

---

## 3. State

A single top-level `APP` object holds all mutable state. No other global mutable variables exist.

```javascript
const APP = {
  // Circuit
  circuit: CircuitGraph,

  // Viewport
  viewport: { tx, ty, scale },

  // Interaction
  selection: Set,              // Set of component/wire ids
  activeTool: 'select',        // 'select' | 'wire' | 'probe'
  drag: null,                  // DragState | null (palette drag or component move)
  wireInProgress: null,        // WireInProgress | null
  hover: null,                 // { type: 'terminal'|'component'|'wire', id, terminal } | null
  spaceDown: false,            // true while Space is held (pan mode)
  mouseScreen: {x:0, y:0},    // last known mouse position in screen space

  // Probe
  probeReadouts: [],           // ProbeReadout[]
  probeAnchor: null,           // { nodeKey, sx, sy } | null — first probe click marker

  // Simulation
  simResult: null,             // SimResult | null
  simError: null,              // string | null
  simErrorIds: [],             // component/node ids to highlight red

  // Undo / Redo
  undoStack: [],               // CircuitGraph[]
  redoStack: [],               // CircuitGraph[]

  // DOM refs (set in bootstrap)
  canvas: null,
  ctx: null,
  popover: null,
  errorBanner: null,
}
```

### CircuitGraph

Plain object, fully JSON-serializable (enabling deep clone via `JSON.parse(JSON.stringify(...))`):

```javascript
{
  components: {
    [id]: {
      id,          // integer
      type,        // 'battery'|'resistor'|'capacitor'|'inductor'|'switch'|'ground'
      x,           // grid units (integer)
      y,           // grid units (integer)
      rotation,    // 0 | 90 | 180 | 270
      params,      // { E, R, C, L, state } — only relevant keys present
    }
  },
  wires: {
    [id]: {
      id,          // integer
      points,      // [{x,y}, ...] grid units, minimum 2 points
    }
  },
  nextId: 1,       // monotonically increasing, never reused
}
```

### WireInProgress

```javascript
{
  points: [{x,y}],          // committed waypoints so far (grid coords)
  startComponentId: id,
  startTerminal: 'A'|'B',
  flipL: false,             // true when user has pressed X (overrides auto)
}
```

### ProbeReadout

```javascript
{
  id: integer,
  type: 'voltage'|'current',
  targetA: nodeKey | null,    // for voltage probes
  targetB: nodeKey | null,    // for voltage probes
  componentId: id | null,     // for current probes
  value: number | null,       // null = deleted target
  sx: number,                 // screen x of readout panel anchor
  sy: number,                 // screen y of readout panel anchor
}
```

---

## 4. Geometry Module

Pure functions, no side effects, no DOM access.

```javascript
worldToScreen(wx, wy, vp)         // → {sx, sy}
screenToWorld(sx, sy, vp)         // → {wx, wy}  (wx,wy in px — divide by GRID for grid units)
snapToGrid(wx, wy)                // → {x,y} integer grid units
terminalPositions(component)      // → { A:{x,y}, B:{x,y}|null }  grid units
rotateOffset(dx, dy, rotation)    // → {dx,dy}
segmentHitTest(px,py, ax,ay, bx,by, threshold)  // → boolean  (screen space)
nearestTerminal(sx, sy, circuit, vp)  // → {componentId, terminal, gx, gy} | null
  // returns null if no terminal within TERMINAL_HIT px
lShapePoints(from, to, flipL)     // → [{x,y},{x,y},{x,y}]  grid units (3 pts = 2 segments)
  // auto-direction: if |dx|>=|dy| go H-first; else V-first; flipL inverts
rerouteWireEnd(wire, oldPt, newPt)  // → Wire  re-routes first or last segment as L-shape
splitWireAtPoint(wire, gx, gy)    // → [Wire, Wire]  splits at nearest grid point on segment
midpointOfLongestSegment(wire)    // → {x,y} grid units
```

---

## 5. Circuit Graph Module

Pure functions. Each takes a `CircuitGraph`, returns a new `CircuitGraph`. Deep-clones input before mutating.

```javascript
addComponent(circuit, type, gx, gy, rotation, params)  // → CircuitGraph
deleteComponents(circuit, ids)     // → CircuitGraph  (also deletes connected wires)
moveComponent(circuit, id, gx, gy) // → CircuitGraph  (re-routes attached wires via rerouteWireEnd)
setParam(circuit, id, key, value)  // → CircuitGraph
addWire(circuit, points)           // → CircuitGraph
deleteWires(circuit, ids)          // → CircuitGraph
splitWire(circuit, wireId, gx, gy) // → CircuitGraph  (replaces wire with two)
```

**moveComponent** implementation:
1. Clone circuit
2. Record old terminal positions
3. Update component x, y
4. Compute new terminal positions
5. For each wire whose first or last point matched an old terminal position, call `rerouteWireEnd` to produce a new orthogonal path to the next waypoint
6. Replace wires in cloned circuit

---

## 6. Union-Find / Node Resolution Module

Computes electrical node membership from a `CircuitGraph`. Called before every simulation run and whenever node keys are needed (probe, labels).

```javascript
buildNodeMap(circuit)  // → NodeMap
```

**NodeMap:**
```javascript
{
  nodes: { [canonicalKey]: ["x,y", ...] },  // canonical key → all coords in node
  coordToNode: { ["x,y"]: canonicalKey },   // any coord → its node's canonical key
  termToNode: { ["id_A"]: canonicalKey },   // "componentId_terminal" → canonical key
  groundKey: canonicalKey | null,           // canonical key of ground node, or null
}
```

**Algorithm:**
1. Collect all grid coords: every terminal position + every wire waypoint
2. Initialize union-find: each coord is its own set
3. For each wire: union all adjacent waypoint pairs
4. For each coord: union with any terminal that shares the same grid position
5. Canonical key = lexicographically smallest `"x,y"` string in the set (e.g. `"3,7"` before `"3,12"` before `"10,2"`)
6. Ground key = canonical key of the node containing any ground component's terminal A

**DC connectivity check** (for floating-node validation):
Re-run union-find excluding all capacitor terminal coords. Any non-ground node not reachable from ground in this reduced graph is floating.

---

## 7. MNA Solver Module

Self-contained pure function. No DOM, no side effects.

```javascript
solve(circuit, nodeMap)  // → SimResult | SimError
```

### 7.1 Setup

1. Enumerate non-ground nodes → array `nodeKeys[]`, assign integer index 0..N-1
2. Enumerate voltage sources: batteries + closed switches + inductors → `vsources[]`, assign index 0..M-1
3. Allocate augmented matrix `A` of size (N+M) × (N+M+1), all zeros

### 7.2 Stamping

For each component, look up its terminal nodes via `nodeMap.termToNode`. If a terminal is the ground node, treat it as the reference (no row/column in matrix).

Use helper:
```javascript
function nodeIndex(key) {
  if (key === nodeMap.groundKey) return -1  // ground = reference
  return nodeKeys.indexOf(key)
}
```

Apply stamps per Spec §7.3. Skip ground rows/columns (index === -1).

### 7.3 Gaussian Elimination with Partial Pivoting

```
for col = 0 to N+M-1:
  find row >= col with max |A[row][col]|
  if max < PIVOT_EPSILON: return SimError("singular matrix")
  swap rows
  normalize pivot row
  eliminate all other rows
extract solution vector x from last column
```

### 7.4 Result Extraction

- `v[i]` = solution at index i = voltage of `nodeKeys[i]`
- `j[k]` = solution at index N+k = current through voltage source k (into + terminal)
- Battery/switch/inductor branch current = `j[k]` for their source index k
- Resistor branch current = `(V_A - V_B) / R`
- Capacitor branch current = 0 (open circuit)
- Open switch branch current = 0

```javascript
SimResult = { ok: true, nodeVoltages: Map, branchCurrents: Map }
SimError  = { ok: false, message: string, highlightIds: [] }
```

---

## 8. Simulation Orchestrator

```javascript
function runSimulation() {
  const circuit = APP.circuit
  const nodeMap = buildNodeMap(circuit)

  // Pre-solve validation (first error only)
  if (!nodeMap.groundKey) {
    setError("No ground node. Add a ground component.", [])
    return
  }
  const floaters = findFloatingNodes(circuit, nodeMap)
  if (floaters.length > 0) {
    const {x,y} = floaters[0]
    setError(`Node at (${x}, ${y}) is floating. All nodes must connect to ground.`, [floaters[0].key])
    return
  }
  if (Object.keys(circuit.components).length === 0) {
    APP.simResult = null; APP.simError = null
    updateProbeReadouts(); render(); return
  }

  const result = solve(circuit, nodeMap)
  if (!result.ok) {
    setError(result.message, result.highlightIds)
    return
  }

  APP.simResult = result
  APP.simError = null
  APP.simErrorIds = []
  updateProbeReadouts()
  render()
}

function setError(msg, ids) {
  APP.simError = msg
  APP.simErrorIds = ids
  APP.simResult = null
  updateProbeReadouts()
  render()
}
```

`runSimulation()` is called at the end of every `dispatch()` call.

---

## 9. SI Prefix Formatter

```javascript
function formatSI(value, unit)  // → string, e.g. "3.14 mA", "12.0 kΩ"
```

Prefix ladder: `['p',1e-12], ['n',1e-9], ['µ',1e-6], ['m',1e-3], ['',1], ['k',1e3], ['M',1e6], ['G',1e9]`

Select the largest prefix whose factor ≤ |value|. Display mantissa with `toPrecision(3)`. Handle zero as `"0 " + unit`.

---

## 10. Component Symbols Module

One draw function per type. Each draws in **local component space**: origin at (0,0), facing right (0° orientation). Size is expressed in grid units. The renderer applies the world transform and rotation before calling these.

```javascript
drawBattery(ctx, params)     // long line (+), short line (−), lead lines
drawResistor(ctx)            // IEEE zigzag: 6 peaks over 4 grid units
drawCapacitor(ctx)           // two parallel vertical lines, lead lines
drawInductor(ctx)            // 3 semicircular humps above center line
drawSwitch(ctx, isOpen)      // isOpen: gap with angled line; closed: straight line
drawGround(ctx)              // vertical lead + 3 descending horizontal lines
```

All use `ctx.beginPath()` / `ctx.stroke()`. No fills. Stroke style set by caller.

Symbol extents (in grid units, from origin):
- All standard components: ±2 on the terminal axis, ±0.75 perpendicular
- Ground: terminal at (0,−1), symbol extends to (0,+1.5) downward

---

## 11. Renderer

Single entry point called after every state change:

```javascript
function render()
```

Drawing order:

1. **Clear** — `ctx.clearRect` + fill background color
2. **Grid** — if `APP.viewport.scale >= GRID_HIDE_ZOOM`, draw dot grid across visible area
3. **Wires** — iterate `circuit.wires`; draw polylines; selected wires in accent color; error-highlighted wires in red
4. **Wire in progress** — dashed polyline: committed points + live L-shape preview to current mouse
5. **Components** — for each component:
   a. Save ctx, apply viewport transform + component position + rotation
   b. Call symbol draw function in local space
   c. Draw terminal dots (filled circles, ~4px radius at scale 1)
   d. If selected: draw highlight rectangle
   e. If in error: overlay red stroke
   f. Restore ctx
6. **Simulation labels** — drawn at screen coordinates (no viewport scale on font):
   a. Node voltage labels: one per node, at screen position of `midpointOfLongestSegment`
   b. Component current labels: below each component in screen space
7. **Probe markers** — draw anchor crosshair at `APP.probeAnchor` if set; draw readout panels
8. **Rubber-band selection rect** — if drag-selecting
9. **Palette drag ghost** — if `APP.drag.type === 'palette'`
10. **Error banner** — update DOM element (not canvas)

**Viewport transform helper:**
```javascript
function applyViewport(ctx, vp) {
  ctx.setTransform(vp.scale, 0, 0, vp.scale, vp.tx, vp.ty)
}
```

**Label drawing** resets transform to identity, converts grid coords to screen coords manually for positioning.

---

## 12. Undo / Redo Module

```javascript
function pushUndo() {
  APP.undoStack.push(deepClone(APP.circuit))
  APP.redoStack = []
  updateUndoButtons()
}

function undo() {
  if (!APP.undoStack.length) return
  APP.redoStack.push(deepClone(APP.circuit))
  APP.circuit = APP.undoStack.pop()
  runSimulation()
  updateUndoButtons()
}

function redo() {
  if (!APP.redoStack.length) return
  APP.undoStack.push(deepClone(APP.circuit))
  APP.circuit = APP.redoStack.pop()
  runSimulation()
  updateUndoButtons()
}

function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj))
}

function updateUndoButtons() {
  // gray out / enable undo and redo buttons
}
```

---

## 13. Probe Tool Module

```javascript
function probeClick(sx, sy)      // handle a probe tool click
function closeReadout(id)        // remove a readout panel
function updateProbeReadouts()   // re-evaluate all readouts against APP.simResult
```

**probeClick logic:**
1. Hit-test: terminal or wire → node probe candidate; component body → current probe
2. If no anchor set: set `APP.probeAnchor`, draw marker, return
3. If anchor set and same type: complete voltage probe, create ProbeReadout, clear anchor
4. Component body click at any point: create current ProbeReadout immediately (no anchor needed)

**updateProbeReadouts:**
- For voltage readouts: look up `nodeVoltages.get(targetA)` and `nodeVoltages.get(targetB)`; compute difference; set `value = null` if either key missing
- For current readouts: look up `branchCurrents.get(componentId)`; set `value = null` if missing

Readout panels are rendered on the canvas in the renderer (step 7). They are HTML-like boxes drawn with `ctx.fillRect` + `ctx.fillText`, not DOM elements, so they zoom/pan correctly. The close button is a hit-tested region in `onMouseDown`.

---

## 14. Parameter Popover

A single `<div id="popover">` in the HTML body, `position: absolute`, hidden by default (`display: none`).

```javascript
function openPopover(componentId, sx, sy)
function closePopover(save)
```

**openPopover:**
1. Look up component in `APP.circuit`
2. Build input fields dynamically based on component type
3. Position the div near (sx, sy), clamped to stay within viewport
4. Show div, focus first input

**closePopover(true):**
1. Read all input values
2. Validate each (range check)
3. If all valid: `dispatch({ type: 'SET_PARAM', id, params })`, hide div
4. If any invalid: show red borders, do not close

**closePopover(false):** hide div, discard changes.

Popover is closed by: Enter key, click outside, Escape (cancel).

---

## 15. Interaction Module

### 15.1 dispatch(action)

```javascript
function dispatch(action) {
  pushUndo()
  switch(action.type) {
    case 'ADD_COMPONENT':  APP.circuit = addComponent(APP.circuit, ...); break
    case 'DELETE':         APP.circuit = deleteComponents(APP.circuit, ...); break
    case 'MOVE':           APP.circuit = moveComponent(APP.circuit, ...); break
    case 'SET_PARAM':      APP.circuit = setParam(APP.circuit, ...); break
    case 'ADD_WIRE':       APP.circuit = addWire(APP.circuit, ...); break
    case 'DELETE_WIRES':   APP.circuit = deleteWires(APP.circuit, ...); break
    case 'SPLIT_WIRE':     APP.circuit = splitWire(APP.circuit, ...); break
  }
  runSimulation()
}
```

Non-circuit mutations (pan, zoom, selection, hover) bypass dispatch and mutate `APP` directly, then call `render()`.

### 15.2 Event Handlers

**onMouseDown(e):**
```
if probe tool active:
  probeClick(e.offsetX, e.offsetY); return

if space held or middle button:
  begin pan; return

terminal = nearestTerminal(e.offsetX, e.offsetY, ...)
if terminal found:
  begin wire draw (set APP.wireInProgress); return

hit = hitTestComponentBody(e.offsetX, e.offsetY)
if hit:
  if not selected: select it (clear others unless Shift)
  begin component move drag; return

hit = hitTestWire(e.offsetX, e.offsetY)
if hit:
  select wire; return

// empty canvas
if wireInProgress: place waypoint; return
begin rubber-band drag
```

**onMouseMove(e):**
```
update APP.mouseScreen
if panning: update viewport tx,ty; render(); return
if component dragging: update ghost position; render(); return
if palette dragging: update ghost position; render(); return
update hover state (nearest terminal or component)
render()
```

**onMouseUp(e):**
```
if panning: end pan; return
if component dragging:
  snap to grid; dispatch MOVE; end drag; return
if palette dragging:
  if over canvas: dispatch ADD_COMPONENT; end drag; return
  cancel drag; return
if wire drawing and click on terminal:
  check for T-junction on existing wire; if so dispatch SPLIT_WIRE first
  dispatch ADD_WIRE with APP.wireInProgress.points + target terminal position
  APP.wireInProgress = null; return
```

**onDblClick(e):**
```
hit = hitTestComponentBody(e.offsetX, e.offsetY)
if hit: openPopover(hit.id, e.offsetX, e.offsetY)
```

**onWheel(e):**
```
e.preventDefault()
zoomToPointer(e.offsetX, e.offsetY, e.deltaY)
render()
```

**onKeyDown(e):**
```
R        → rotate selected component(s) or drag ghost
X        → toggle wireInProgress.flipL
P        → toggle probe tool
Escape   → cancel wire / close popover / exit probe / deselect
Delete   → dispatch DELETE for selection
Ctrl+Z   → undo()
Ctrl+Y   → redo()
Ctrl+0   → reset zoom
Ctrl+=   → zoom in
Ctrl+-   → zoom out
Arrows   → pan (×10 if Shift)
```

### 15.3 Hit Testing

**hitTestComponentBody(sx, sy):**
For each component, transform (sx, sy) into component local space accounting for viewport + component position + rotation. Test against a bounding box of ±2 grid units on terminal axis, ±1 grid unit perpendicular.

**hitTestWire(sx, sy):**
For each wire, test each segment using `segmentHitTest` with a threshold of 5px screen space.

**hitTestProbeClose(sx, sy):**
For each probe readout, test against the close button region (rendered as a small rect).

---

## 16. Bootstrap

```javascript
window.addEventListener('DOMContentLoaded', () => {
  APP.canvas  = document.getElementById('canvas')
  APP.ctx     = APP.canvas.getContext('2d')
  APP.popover = document.getElementById('popover')
  APP.errorBanner = document.getElementById('error-banner')

  // Set initial circuit
  APP.circuit = { components: {}, wires: {}, nextId: 1 }

  // Set initial viewport (centered)
  APP.viewport = { tx: APP.canvas.width/2, ty: APP.canvas.height/2, scale: 1.0 }

  // Size canvas
  resizeCanvas()
  window.addEventListener('resize', resizeCanvas)

  // Attach all event listeners
  APP.canvas.addEventListener('mousedown', onMouseDown)
  APP.canvas.addEventListener('mousemove', onMouseMove)
  APP.canvas.addEventListener('mouseup',   onMouseUp)
  APP.canvas.addEventListener('dblclick',  onDblClick)
  APP.canvas.addEventListener('wheel',     onWheel, { passive: false })
  window.addEventListener('keydown', onKeyDown)

  // Palette drag init
  initPalette()

  render()
})

function resizeCanvas() {
  APP.canvas.width  = APP.canvas.offsetWidth
  APP.canvas.height = APP.canvas.offsetHeight
  // Re-center viewport only on first load (tracked by a flag)
  render()
}
```

**initPalette():** Attaches `mousedown` listeners to each palette item. On mousedown, creates a palette drag state in `APP.drag` with the component type and initial ghost position.

---

## 17. HTML / CSS Shell

### Body layout (CSS Grid or Flexbox):
```
┌─────────────────────────────────────────┐
│            top bar (40px)               │
├──────────┬──────────────────────────────┤
│ palette  │         canvas               │
│ (160px)  │    (fills remaining)         │
│          │                              │
└──────────┴──────────────────────────────┘
```

### Key DOM elements:
```html
<div id="topbar">
  <span id="app-title">Circuit Simulator</span>
  <button id="btn-undo">Undo</button>
  <button id="btn-redo">Redo</button>
  <button id="btn-zoom-in">+</button>
  <button id="btn-zoom-out">−</button>
  <button id="btn-zoom-reset">100%</button>
  <span id="tool-indicator">Select</span>
</div>
<div id="palette">
  <!-- one .palette-item per component type -->
</div>
<canvas id="canvas"></canvas>
<div id="error-banner" style="display:none"></div>
<div id="popover" style="display:none"></div>
```

Canvas is sized to fill its grid cell via CSS (`width: 100%; height: 100%`). The actual pixel dimensions are set in JavaScript via `resizeCanvas()`.

---

## 18. Build Order

Modules must appear in this order in the `<script>` block:

1. Constants
2. State (APP object declaration and initialization)
3. Geometry
4. Circuit Graph
5. Union-Find / Node Resolution
6. MNA Solver
7. Simulation Orchestrator
8. SI Prefix Formatter
9. Component Symbols
10. Renderer
11. Undo / Redo
12. Probe Tool
13. Parameter Popover
14. Interaction (dispatch + all event handlers)
15. Bootstrap

---

## 19. Coordinate System Reference

| Space | Unit | Origin | Used for |
|---|---|---|---|
| Grid space | grid units (integers) | arbitrary | component positions, wire waypoints, node keys |
| World space | px at scale=1 | same origin as grid | intermediate calculations; `worldPx = gridUnit × GRID` |
| Screen space | px | canvas top-left | mouse events, rendering, hit testing |

Conversion chain:
```
grid → world:   wx = gx * GRID,  wy = gy * GRID
world → screen: sx = wx * scale + tx,  sy = wy * scale + ty
screen → world: wx = (sx - tx) / scale
world → grid:   gx = Math.round(wx / GRID)
```

All stored positions (components, wires) are in **grid units**. Rendering always converts to screen space. Hit testing always converts mouse screen coords to grid or world as needed.

---

*End of Plan v1.0*
