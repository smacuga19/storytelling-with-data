# Specification — DC Circuit Simulator
**Version 1.3** *(updated from v1.2)*

### Changelog from v1.2
- §12: Typography sizes and contrast levels updated to reflect legibility pass

### Changelog from v1.0
- §1: Top bar updated — added Wire tool button, Run button, Help button
- §5.1: Wire tool is now explicit, not auto-activated; wire can start and land on terminals, wire endpoints, and mid-segments
- §5.2: Valid connections expanded to include wire-endpoint-to-wire-endpoint
- §6: Tools section rewritten — three explicit tools with keyboard shortcuts
- §7.1: Simulation is now manual (▶ Run / Enter), not automatic
- §7.5: Floating node error message updated; error now highlights affected elements visually
- §11: Keyboard shortcuts updated — added W, S, Enter
- §12: Visual design updated — error wire highlighting, probe panel details
- §13: New section — Help Panel

---

## 1. Application Shell

The application is a single HTML file. On load it renders regions that together fill the full viewport with no scrollbars:

**Left panel — Component Palette (fixed width: 152px)**
A vertical strip listing every available component. Each entry shows a schematic symbol and a label. The user drags an entry onto the canvas to place it. The palette does not scroll; all components fit in the fixed height.

**Center region — Canvas (fills remaining width)**
An infinite, zoomable, pannable workspace where the circuit is built.

**Right panel — Help Panel (fixed width: 240px, collapsible)**
A collapsible panel containing a quick-start guide and full keyboard reference. See §13.

**Top bar (full width, above all panels, fixed height: 42px)**
Contains, left to right: app title, Undo button, Redo button, a divider, Zoom In button, Zoom Out button, Zoom Reset button, a divider, Probe button, Wire button, a divider, ▶ Run button, tool indicator span, a divider, Help button.

Undo/Redo buttons are grayed out when their respective stacks are empty. The active tool button is highlighted.

---

## 2. Canvas

### 2.1 Grid
A dot grid (not lines) drawn at every logical grid unit (20px at 100% zoom). Dots scale and translate with the viewport. At zoom levels below 40%, the grid is hidden to reduce visual noise. Grid is always one dot per grid unit — no subdivision at high zoom.

### 2.2 Viewport Transform
The canvas maintains a single transform: `{ tx, ty, scale }`. All world-to-screen and screen-to-world coordinate conversions go through this transform exclusively.

- Initial state: scale = 1.0, tx = canvas_width/2, ty = canvas_height/2 (origin centered)
- Scale range: 0.1 – 10.0
- Zoom is applied toward the cursor position (zoom-to-pointer)

### 2.3 Pan
- Middle mouse button drag
- Space + left mouse drag
- Arrow keys pan by one grid unit per keypress (10 grid units if Shift held)

Pan continues to track correctly even when the mouse moves outside the canvas boundary during a drag.

### 2.4 Zoom
- Scroll wheel (zoom to pointer)
- Ctrl+= and Ctrl+− keyboard shortcuts
- Zoom In / Zoom Out / Zoom Reset buttons in top bar

### 2.5 Resize
The canvas element resizes to fill its container. On window resize, the canvas element dimensions update; the viewport transform is preserved.

---

## 3. Component Palette

Components listed top to bottom in the palette:

| Label | Symbol description |
|---|---|
| Battery | Two unequal parallel lines (long = +, short = −) |
| Resistor | IEEE zigzag line |
| Capacitor | Two equal parallel lines with a gap |
| Inductor | Series of humps (3 arcs, IEEE style) |
| Switch | Line with a gap and a tilted segment |
| Ground | Three descending horizontal lines |

Dragging from the palette creates a ghost component that follows the cursor, snapped to the grid. The ghost always starts at 0° (horizontal) regardless of previously placed components. Releasing over the canvas places the component. Releasing over the palette cancels the drag. Pressing **R** during a drag rotates the ghost 90° CW.

---

## 4. Components

### 4.1 Placement
When placed, a component is assigned:
- A unique integer ID
- A world-space grid position (origin point, always on-grid)
- A default orientation: horizontal (0°)
- Default parameter values (see 4.3)

### 4.2 Terminals
Every component (except Ground) has exactly two terminals: **A** and **B**.
Ground has one terminal: **A**.

Terminal world positions are derived from the component's grid position and orientation:

| Component | Terminal A offset | Terminal B offset |
|---|---|---|
| Battery | (−2, 0) grid units | (+2, 0) grid units |
| Resistor | (−2, 0) | (+2, 0) |
| Capacitor | (−2, 0) | (+2, 0) |
| Inductor | (−2, 0) | (+2, 0) |
| Switch | (−2, 0) | (+2, 0) |
| Ground | (0, −1) | — |

Terminal A is the positive terminal for Battery. For all other two-terminal components, terminals are symmetric. Ground's terminal is at the top of the symbol; the descending-lines symbol hangs below. Wires connect to Ground from above.

### 4.3 Default Parameters

| Component | Parameter | Default | Unit | Valid Range |
|---|---|---|---|---|
| Battery | Voltage (E) | 9 | V | any real number |
| Resistor | Resistance (R) | 1000 | Ω | > 0 |
| Capacitor | Capacitance (C) | 100 | µF | > 0 |
| Inductor | Inductance (L) | 10 | mH | > 0 |
| Switch | State | Open | — | Open / Closed |
| Ground | — | — | — | — |

### 4.4 Orientation
Components can be rotated in 90° increments (0°, 90°, 180°, 270°). Rotation is applied around the component origin. Terminal offsets rotate accordingly. Rotation is triggered by pressing **R** while a component is selected or while a drag-ghost is active.

### 4.5 Parameter Editing
Double-clicking a placed component opens a small inline popover (not a modal) anchored to the component. The popover contains:
- A labeled numeric input per parameter
- A unit label (non-editable)
- For Switch: a toggle button (Open / Closed)
- Confirm on Enter or clicking away; cancel on Escape

Capacitor and inductor popovers show their parameter inputs with no special messaging about DC behavior. Invalid values (out of range) show a red border and block confirmation. The popover closes on Escape without saving.

### 4.6 Selection
Selection is only available in the Select tool (§6.1).
- Single click selects one component or one wire segment
- Shift+click toggles selection of additional items
- Click on empty canvas deselects all
- Rubber-band drag (left mouse drag on empty canvas) selects all items whose bounding boxes intersect the drag rectangle
- Selected components show a highlight rectangle around them
- Selected wire segments highlight in accent color

### 4.7 Moving
- Drag a selected component to move it; it snaps to the grid
- Moving a component with connected wires: the wire endpoints connected to that component's terminals move with it; attached wire segments are automatically re-routed as orthogonal segments from the new terminal position to the next waypoint
- The re-route uses the same axis-lock logic as wire drawing (greater displacement axis goes first)

### 4.8 Deletion
- Delete or Backspace key deletes all selected components and wires
- Deleting a component also deletes all wires connected to its terminals
- Deletion is a single undoable action

---

## 5. Wires

### 5.1 Drawing
The Wire tool (**W** key or **✎ Wire** toolbar button) must be active to draw wires.

**Starting a wire:** Click on any of the following — in priority order:
1. A component terminal (within 10px screen space, snaps to terminal grid position)
2. An existing wire endpoint (within 10px screen space)
3. Any free grid point

**While drawing — straight-segment model:**
Each click lays exactly one straight segment, either horizontal or vertical. The user controls every bend by clicking intermediate waypoints before landing on the target. There is no automatic L-shape routing between start and end.

- The in-progress wire displays all committed segments plus a dashed preview of the next single segment from the last waypoint to the axis-locked cursor position
- The segment axis is auto-selected: horizontal if |Δx| ≥ |Δy|, vertical otherwise
- **X** toggles between horizontal-first and vertical-first for the current segment
- Clicking on empty canvas plants a waypoint at the axis-locked position and continues the wire from there
- Pressing Escape cancels the in-progress wire; the Wire tool remains active

**Example — wiring a corner:** To connect a battery terminal to a resistor terminal that is both to the right and below, click once on empty canvas at the turn point, then click the destination terminal. The first click plants a horizontal segment; the second click plants a vertical segment to the target.

**Landing a wire:** Click on any of the following — in priority order:
1. A component terminal (within 10px screen space)
2. An existing wire endpoint (within 10px screen space)
3. A wire mid-segment (within 8px screen space) — automatically splits the segment and forms a T-junction (see §5.5)

A wire is not committed if the landing point is identical to the starting point.

### 5.2 Valid Connections
A wire may connect:
- Terminal to terminal
- Terminal to existing wire endpoint
- Terminal to wire mid-segment (T-junction)
- Wire endpoint to wire endpoint
- Wire endpoint to wire mid-segment (T-junction)

A wire may not connect a point to itself.

### 5.3 Wire Data Model
A wire is a polyline: an ordered list of grid-coordinate waypoints. Minimum two distinct points. Every consecutive pair of waypoints lies on the same horizontal or vertical line (orthogonal invariant).

### 5.4 Node Merging
All terminals and wire waypoints that share the same grid coordinate belong to the same electrical node. Node membership is computed by union-find over all coincident coordinates each time the circuit graph changes. A node's canonical key is the lexicographically smallest `"x,y"` string in its connected set.

### 5.5 T-Junctions
Landing a wire on the mid-segment of an existing wire splits that segment at the nearest grid point on the segment's axis, inserts a waypoint, and forms a T-junction. The original wire is replaced by two wires sharing that waypoint.

### 5.6 Visual Style
- Default wire color: neutral mid-tone (dark gray)
- Selected wire segments: accent color
- In-progress wire: dashed line in accent color (single-segment preview only)
- Error-highlighted wires: red stroke

---

## 6. Tools

Three explicit tools are available. The active tool is shown in the top bar indicator and its toolbar button is highlighted.

### 6.1 Select Tool (default)
**Activated by:** **S** key, or Escape when not drawing a wire.

Behaviour:
- Left click → select component or wire segment
- Shift+click → add to / remove from selection
- Drag on selected component → move it
- Drag on empty canvas → rubber-band select
- Double-click on component → open parameter popover

### 6.2 Wire Tool
**Activated by:** **W** key or **✎ Wire** toolbar button.

Behaviour: see §5.1. Each click either starts a wire, plants a waypoint (one straight segment), or lands the wire on a valid target. The tool remains active after a wire is committed. Escape cancels an in-progress wire but keeps the Wire tool active. Pressing **S** or **P** switches away.

### 6.3 Probe Tool
**Activated by:** **P** key or **⊕ Probe** toolbar button.

Behaviour: see §8. Pressing **S** or **W** switches away; existing readouts remain open.

---

## 7. Simulation

### 7.1 Trigger
Simulation runs **only when explicitly requested** — it does not run automatically on circuit mutations. Any circuit mutation (place, delete, move, wire, parameter edit) immediately clears any existing simulation results so stale labels do not persist.

Simulation is triggered by:
- Clicking the **▶ Run** button in the top bar
- Pressing **Enter** (when focus is not in a text input)

### 7.2 MNA Construction
For a circuit with N non-ground nodes and M voltage sources (batteries + closed switches + inductors), the MNA system is:

```
[ G  B ] [ v ]   [ i ]
[ C  D ] [ j ] = [ e ]
```

Where:
- `G` is the (N×N) nodal conductance matrix
- `B` is the (N×M) voltage-source incidence matrix
- `C = Bᵀ`
- `D` is the (M×M) zero matrix (for ideal sources)
- `v` is the vector of unknown node voltages
- `j` is the vector of unknown source currents
- `i` is the vector of known current injections
- `e` is the vector of known source voltages

Ground node is eliminated (referenced as node 0, voltage = 0 V).

### 7.3 MNA Stamps

**Resistor (R) between nodes p and q:**
```
G[p][p] += 1/R
G[q][q] += 1/R
G[p][q] -= 1/R
G[q][p] -= 1/R
```

**Voltage source (E) from node p (+) to node q (−), source index k:**
```
B[p][k] += 1
B[q][k] -= 1
C[k][p] += 1
C[k][q] -= 1
e[k]     = E
```

Batteries, closed switches (E=0), and inductors (E=0) all use this stamp.

**Capacitor:** No stamp (open circuit in DC).
**Open switch:** No stamp.

### 7.4 Solver
Gaussian elimination with partial pivoting on the assembled MNA matrix. Returns:
- Node voltage map: `{ canonicalKey → voltage (V) }`
- Branch current map: `{ componentId → current (A) }`, defined as current flowing from terminal A to terminal B

### 7.5 Pre-solve Validation
Before matrix assembly, the following are checked in order. Only the first error encountered is reported:

1. **No ground node** → error: `"No ground node. Add a ground component."`
2. **Floating node** (DC connectivity check — capacitors treated as open circuits) → error: `"Floating node detected — highlighted in red. All nodes must connect to ground."` All components and wires belonging to the floating node are highlighted red on the canvas.
3. **No components** → simulation skipped silently
4. **Singular matrix** (pivot < 1×10⁻¹²) → error: `"Circuit has a loop or conflict that cannot be solved (singular matrix)."`

### 7.6 Error Display
The first error appears as a dismissible banner below the top bar. All components and wire segments belonging to the offending node or branch are highlighted with a red stroke on the canvas. Node-to-element resolution is performed by `resolveNodeToIds()`, which maps a node's canonical key to the concrete component and wire IDs in that node.

### 7.7 Result Display
After a successful solve:
- Each electrical node is labeled once with its voltage, placed at the midpoint of the longest wire segment in that node's wire network
- Each component is labeled below its symbol (screen space, regardless of rotation) with its branch current
- Labels use auto-scaled SI prefixes (see §10)
- Label font size is fixed in screen space (unaffected by zoom)
- All results are cleared immediately when any circuit mutation occurs

---

## 8. Probe Tool

### 8.1 Activation
The Probe tool is activated by pressing **P** or clicking **⊕ Probe** in the top bar. The cursor changes to a crosshair. The active button is highlighted.

### 8.2 Node Voltage Probe
Click on any wire or terminal → sets probe point A; a persistent crosshair marker appears at that location. Click a second wire or terminal → sets probe point B. The probe readout displays `V_A − V_B` in a floating readout panel near the second click point.

### 8.3 Component Current Probe
Click directly on a component body (not a terminal) → the probe readout displays the branch current through that component (A to B direction is positive). No anchor step required; the readout appears immediately.

### 8.4 Readout Panel
A small floating panel (rendered on the canvas, not as a DOM element) shows:
- Probe type label (V or I)
- Numeric value with SI prefix and unit, or `—` if target is deleted or sim not yet run
- A close (✕) button

Multiple probe readouts can be open simultaneously.

### 8.5 Staleness
Probe readouts persist through undo/redo and re-evaluate against the restored circuit. If the probed node or component is deleted, the readout displays `—`. When simulation results are cleared by a circuit mutation, readouts display `—` until the next Run.

### 8.6 Exit
Press **S**, **W**, or **Escape** to exit the Probe tool without closing existing readouts.

---

## 9. Undo / Redo

### 9.1 Actions That Are Recorded
Place component, delete component(s), move component, edit parameter, toggle switch, place wire, delete wire(s), rotate component.

### 9.2 Actions That Are Not Recorded
Pan, zoom, probe, selection changes, popover open/close, run simulation.

### 9.3 Mechanics
- Each recorded action pushes a deep-cloned snapshot of the full `CircuitGraph` onto the undo stack before the mutation is applied
- Undo: pops from undo stack, pushes current state to redo stack, restores popped state, clears sim results
- Redo: pops from redo stack, pushes current state to undo stack, restores popped state, clears sim results
- Any new mutation clears the redo stack
- History stack depth is unbounded within a session

### 9.4 UI
- Undo button is grayed out when the undo stack is empty
- Redo button is grayed out when the redo stack is empty
- Keyboard: Ctrl+Z (undo), Ctrl+Y or Ctrl+Shift+Z (redo)

---

## 10. SI Prefix Formatter

Used by simulation result labels and probe readouts.

Prefix table (in order): p (10⁻¹²), n (10⁻⁹), µ (10⁻⁶), m (10⁻³), (none), k (10³), M (10⁶), G (10⁹).

Selects the prefix that keeps the mantissa between 1.000 and 999.9. Displays 3 significant figures.

Examples: `0.00314 A → "3.14 mA"`, `12000 Ω → "12.0 kΩ"`, `0.000000047 F → "47.0 nF"`.

---

## 11. Keyboard Shortcuts

| Key | Action |
|---|---|
| S | Switch to Select tool |
| W | Switch to Wire tool |
| P | Toggle Probe tool |
| Enter | Run simulation |
| R | Rotate selected component (or drag ghost) 90° CW |
| X | Toggle H/V axis for current wire segment |
| Escape | Cancel in-progress wire / close popover / exit probe tool |
| Delete / Backspace | Delete selected items |
| Ctrl+Z | Undo |
| Ctrl+Y / Ctrl+Shift+Z | Redo |
| Ctrl+= | Zoom in |
| Ctrl+− | Zoom out |
| Ctrl+0 | Zoom reset |
| Space+drag | Pan |
| Arrow keys | Pan by 1 grid unit |
| Shift+Arrow keys | Pan by 10 grid units |

---

## 12. Visual Design

- **Background:** deep dark tone (#0d1117)
- **Grid dots:** subtle, low-contrast against background; hidden below 40% zoom
- **Components:** clean off-white or light gray stroke (#e6edf3), IEEE symbols
- **Wires:** medium gray default (#8b949e); accent color when selected (#58a6ff); single dashed segment preview when in-progress; red when error-highlighted (#f85149)
- **Terminal dots:** small filled circles; highlighted on hover
- **Voltage labels:** soft green (#3fb950)
- **Current labels:** soft amber (#d29922)
- **Error highlights:** red stroke (#f85149) on all components and wires belonging to the offending node
- **Probe marker:** crosshair at probe point A, purple tint (#bc8cff)
- **Probe readout panels:** rendered on canvas, dark background, purple border
- **Palette:** slightly lighter panel than canvas background (#161b22)
- **Top bar:** same tone as palette, thin bottom border; buttons use full text color (#e6edf3) at 12px; active tool button highlighted in accent color; disabled buttons at 35% opacity
- **Help panel:** same tone as palette, right side, 240px wide, collapsible with smooth CSS transition
- **Font:** monospace throughout (Courier New); numeric readouts always monospace
- **Help panel typography:** step text and keyboard reference at 12px in full text color (#e6edf3); tip callouts at 11px in muted color (#8b949e); section category labels at 10px uppercase in muted color; all sizes chosen for legibility at panel width

---

## 13. Help Panel

A collapsible panel on the right side of the layout (240px wide when open, 0px when collapsed). Toggled by the **? Help** button in the top bar.

### 13.1 Quick-Start Section
Always visible when the panel is open. Contains:
- Five numbered steps: place a component, enter wire mode, run the simulation, edit values, use the probe tool
- A tip callout about the Ground requirement and the need to press Run
- A tip callout showing pan and zoom controls: `Space + drag` or middle-mouse to pan; scroll wheel to zoom

### 13.2 Full Reference Section
Collapsed by default within the panel, expanded by clicking "Full Reference ▾". Contains keyboard shortcut tables grouped by: Canvas Navigation, Components, Tools, Wires, Simulation, Probe Tool, History, DC Behaviour.

### 13.3 Behaviour
- Panel width transitions smoothly on open/close (CSS transition, 220ms)
- Panel inner content is scrollable if viewport height is insufficient
- The panel does not affect canvas width calculations — it occupies its own grid column

---

*End of Specification v1.2*
