# DC Circuit Simulator

A DC circuit simulator built in a single HTML file. Drag components onto a canvas, wire them together, press Run, and it calculates real voltages and currents using Modified Nodal Analysis.

## Video

https://youtu.be/cw0fujLlzNg

## What it does

- Drag and place components: battery, resistor, capacitor, inductor, switch, ground
- Draw wires by clicking segments (W to enter wire mode, S to select)
- Press ▶ Run to simulate — node voltages appear in green, branch currents in amber
- Probe any node or component for precise measurements
- Unlimited undo / redo
- Pan and zoom an infinite canvas

## How to run

Open `circuit-simulator.html` in any modern browser. Nothing to install.

## Process documents

The simulator was built using the spec-kit workflow — a structured planning process completed before any code was written.

| File | Description |
|---|---|
| `CONSTITUTION.md` | 40 inviolable rules the project could not break |
| `spec.md` | Full functional specification (v1.2) |
| `plan.md` | Technical architecture — 15 modules, build order |
| `plan-v2.md` | Architecture Decision Records — why things diverged from the original plan |
| `tasks.md` | 80 tasks across 15 phases (v1.2, completed tasks marked) |

## Credits

Built with [Claude](https://claude.ai) (Anthropic) using the spec-kit workflow — constitution, specification, clarify, plan, tasks, then implementation. All code, documentation, and process artifacts were produced in a single conversation.

## Physics

The simulator solves DC circuits using Modified Nodal Analysis (MNA). When you press Run, it traces every wire to identify which nodes are connected, assembles a conductance matrix, and solves the system using Gaussian elimination with partial pivoting. Ground is pinned to 0 V; all other node voltages are computed relative to it.

DC steady-state behaviour: capacitors are open circuits, inductors are short circuits.
