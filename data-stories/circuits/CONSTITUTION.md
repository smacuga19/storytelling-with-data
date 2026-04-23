# DC Circuit Simulator — Project Constitution

**Version 1.0 — Inviolable Rules**

---

## Article I: Delivery Format

1. **Single HTML file.** The entire application — markup, styles, and logic — lives in one `.html` file. No build step, no bundler, no server, no install. Opening the file in any modern browser is sufficient to run it.
2. **Zero external dependencies.** No frameworks (React, Vue, Angular, etc.), no libraries (jQuery, D3, etc.), no CDN imports, no Web Workers loaded from URLs. Every byte of code is written inline in the file.
3. **No cookies, no localStorage persistence** unless explicitly added in a future revision with a constitutional amendment. The app is stateless on load.

---

## Article II: Physics Correctness

4. **Modified Nodal Analysis (MNA) is the sole simulation engine.** Kirchhoff's Voltage Law and Kirchhoff's Current Law are enforced algebraically through the MNA stamp system. No shortcuts, no heuristics.
5. **The MNA matrix equation `G·x = b` must be solved exactly** using Gaussian elimination with partial pivoting (or equivalent numerically stable method). Iterative approximations are forbidden for DC steady-state.
6. **Ground is the universal reference node at 0 V.** Every circuit must contain exactly one ground node. Simulation is blocked — with a clear error — if no ground node exists.
7. **Floating nodes are forbidden.** Every non-ground node must have a DC path to ground (directly or through a component). If a floating node is detected, simulation halts and highlights the offending node.
8. **Component physics stamps:**
   - **Battery (ideal voltage source):** Adds a row/column to the MNA matrix via the voltage-source stamp. Positive terminal enforces `V_+ − V_− = E`.
   - **Resistor:** Conductance stamp `G = 1/R` applied to the nodal admittance sub-matrix. `R = 0` is forbidden; minimum enforced resistance is 1 µΩ.
   - **Capacitor:** In DC steady-state, treated as an open circuit (infinite impedance). No stamp added; its terminals are left disconnected in the MNA matrix.
   - **Inductor:** In DC steady-state, treated as a short circuit (zero impedance). Modeled as a voltage source of 0 V via the voltage-source stamp.
   - **Switch (open):** Treated as an open circuit — no stamp.
   - **Switch (closed):** Treated as a 0 V voltage source (short) via the voltage-source stamp.
9. **All displayed values are SI units** with automatic prefix scaling (µ, m, k, M). Raw floating-point values are never shown to the user.

---

## Article III: Components

10. **Canonical component set (v1.0):** Battery, Resistor, Capacitor, Inductor, Switch, Ground node. No other components may be added without a constitutional amendment.
11. **Each component has exactly two terminals**, labeled consistently: terminal A (positive/start) and terminal B (negative/end). Ground has one terminal.
12. **Component parameters are editable** by double-clicking the placed component. Each parameter has a validated range enforced on input (e.g., R > 0, C > 0, L > 0, V can be any real number).
13. **Components are immutable in identity** once placed — a resistor cannot become a capacitor. Delete and replace instead.

---

## Article IV: Canvas and Interaction

14. **The canvas is infinite and grid-snapped.** All component terminals and wire endpoints snap to a fixed grid (default: 20 px logical units). This ensures clean connections and prevents floating near-misses.
15. **Wires connect terminals only.** A wire can only be drawn from one component terminal to another. Free-floating wires not connected at both ends are forbidden and automatically discarded.
16. **Connection detection is binary.** A node connection exists if and only if two terminals occupy exactly the same grid coordinate. Proximity without coincidence is not a connection.
17. **Pan** is performed by middle-mouse drag or Space+drag. **Zoom** is performed by scroll wheel, clamped between 10% and 1000% of default scale. The viewport transform is a pure 2D affine (translate + uniform scale).
18. **Component placement** is done by dragging from the component palette onto the canvas. The drag ghost snaps to the grid in real time.
19. **Selection** is a single click on a component or wire. Multi-select is Shift+click or rubber-band drag. Delete key removes selected elements.

---

## Article V: Undo / Redo

20. **Undo/redo covers all state-mutating actions:** place, delete, move, connect wire, disconnect wire, edit parameter, toggle switch. Read-only actions (pan, zoom, probe) are not recorded.
21. **History stack depth is unbounded** within a session (limited only by RAM).
22. **Undo/redo is triggered by Ctrl+Z / Ctrl+Y** (and Cmd+Z / Cmd+Shift+Z on macOS). The UI also provides explicit buttons.
23. **Each history entry is a deep snapshot** of the full circuit graph (components + wires + parameters). Differential patches are an optimization only — correctness of snapshot-based undo is the baseline requirement.

---

## Article VI: Probe Tool

24. **The probe tool measures voltage difference between any two nodes.** Click node A, then node B; the display shows `V_A − V_B` in volts.
25. **Probe readouts are computed directly from the solved MNA node-voltage vector.** They are never estimated or interpolated.
26. **Current through a component** is displayed as a secondary readout when a component (not a node) is probed: click the component body to display the branch current through it, computed from the MNA solution.
27. **Probe values update immediately** after any circuit change that triggers re-simulation.
28. **If the circuit cannot be simulated** (no ground, floating node, singular matrix), the probe displays `—` with the specific error reason.

---

## Article VII: Simulation Lifecycle

29. **Simulation runs automatically** after every state change (component added/removed, wire connected/disconnected, parameter edited, switch toggled). There is no manual "Run" button.
30. **Auto-simulation is synchronous and must complete in < 50 ms** for circuits up to 500 nodes on a modern device. If this bound is exceeded, an async fallback with a spinner is permitted.
31. **Singular matrix** (short circuit, degenerate topology) must be caught gracefully. The UI highlights the problematic loop or branch and displays a human-readable explanation.

---

## Article VIII: Rendering

32. **All rendering is on an HTML5 `<canvas>` element** using the 2D context API. SVG and DOM-overlay rendering of circuit elements is forbidden (though HTML overlays for UI chrome are permitted).
33. **Wire routing is orthogonal** (Manhattan geometry). Diagonal wires are forbidden. Auto-routing may be added in a future amendment; for now, wires are manually placed segment-by-segment.
34. **Animated current flow** (moving dashes along wires) is permitted as a visual affordance but must be clearly distinguished from simulation output. The animation speed may be proportional to current magnitude but is not a simulation value.
35. **Component symbols follow IEEE/IEC schematic conventions** to the extent feasible in pixel art at the canvas resolution.

---

## Article IX: Code Quality

36. **All logic is in one `<script>` block.** CSS lives in one `<style>` block. No inline `style=` attributes except for dynamic values set by JavaScript.
37. **The MNA solver is a self-contained function** with a clear interface: takes a circuit graph, returns a node-voltage map and branch-current map (or an error). It has no side effects on the DOM.
38. **No global mutable state outside the single application state object.** All state transitions go through a single `dispatch(action)` function.
39. **No `eval()`, `new Function()`, or dynamic code execution** of any kind.
40. **Code is written for human readability:** named constants for magic numbers, descriptive variable names, and a comment block at the top of each logical section.

---

## Amendment Procedure

Any departure from the above rules — adding a component, enabling persistence, adding external dependencies, changing the solver — requires a numbered amendment appended to this document, with the rationale stated explicitly. The original articles are never deleted; amendments override or extend them.

---

*This constitution governs all implementation decisions. When in doubt, refer back here.*
