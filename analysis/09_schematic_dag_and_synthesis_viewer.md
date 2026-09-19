# Axiom EDA: GPU-Accelerated Hardware Schematic DAG & Logic Cone Architecture

## 1. Overview & Vision

In traditional EDA tools like AMD Vivado, the schematic viewer is a sluggish, static Java Swing view. Navigating large designs with hundreds of thousands of gates leads to visual stuttering, modal window freezing, and opaque netlist hierarchies. Furthermore, the schematic is isolated from the simulation waveform and HDL source code—tracing a bug across schematic, waveform, and code requires manual search and mental mapping.

**Axiom EDA replaces this with a GPU-accelerated, infinite-canvas Hardware Directed Acyclic Graph (DAG)** engine built with WebGL / HTML5 Canvas. It connects the elaborated **BIR (Axiom Intermediate Representation)** netlist directly to an interactive, 60+ FPS visual representation with **bidirectional cross-probing** and **instant critical logic cone tracing**.

---

## 2. DAG Topology & Representation

The schematic engine consumes the elaborated `BirCircuit` and maps every entity into a graph node:

```
                                    BirCircuit
                                        |
                 +----------------------+----------------------+
                 |                                             |
                 v                                             v
        Structural Hierarchy Nodes                     Primitive Leaf Nodes
   - Modules & Submodule Instances             - State Registers (FDRE / Latch)
   - Clock Domains & Reset Trees               - Combinational Operators (+, -, *, /)
   - Port Envelopes (Inputs/Outputs)           - Multiplexers & Decoders
                                               - Bitwise & Reduction Gates
```

### 2.1. Multi-Level Semantic Zooming

To prevent visual clutter on designs with $>10^5$ nets, the Axiom Schematic Engine employs continuous semantic level-of-detail (LOD):

1. **Macro Level (Zoom 0.05x – 0.3x)**:
   - Displays high-level module hierarchy blocks with aggregate bus widths.
   - Modules are shaded according to dynamic switching activity heatmaps.
2. **Structural Level (Zoom 0.3x – 1.0x)**:
   - Expands submodule boundaries to reveal registers, arithmetic datapaths, and control multiplexers.
   - Multi-bit buses render as thick bundle conduits with width callouts (e.g. `[31:0]`).
3. **Primitive & Gate Level (Zoom 1.0x – 5.0x)**:
   - Displays individual 4-state logic gates (AND, OR, XOR, NOT) and target FPGA primitives (LUT6, FDRE, CARRY4).
   - In-RAM Cranelift JIT machine code operations and truth tables can be inspected on hover.

---

## 3. Bidirectional Cross-Probing & Synchronization

Axiom enforces seamless synchronization across the three primary views of hardware logic:

```
     +-------------------------------------------------------------+
     |                       HDL Source Editor                     |
     |                       (Monaco / Text AST)                   |
     +------------------------------+------------------------------+
                                    ^
                                    | (Byte Span AST Mapping)
                                    v
+-----------------------------------+-----------------------------------+
|     Interactive Schematic DAG     |        Digital Waveform Canvas    |
|     (WebGL / Canvas 2D)           |<------>|        (Time & Delta Steps)      |
+-----------------------------------+        +-----------------------------------+
             (Selected Net)                              (Signal Trace)
```

1. **Schematic -> Waveform**: Clicking any wire or port pin in the schematic automatically brings that signal to the top of the waveform viewer and highlights its trace.
2. **Waveform -> Schematic**: Selecting a signal label in the waveform viewer automatically pans and focuses the schematic camera onto the driving gate with a soft cyan pulsing glow.
3. **Schematic -> Code**: Double-clicking any gate or register jumps directly to the line of Verilog/SystemVerilog in the editor that instantiated or inferred that logic.

---

## 4. 1-Click Critical Logic Cone Slicing

When debugging timing violations or incorrect simulation values, engineers must isolate the exact fan-in logic cone feeding a flip-flop. In Vivado, this requires multiple nested menu clicks and manually unhiding components.

In Axiom, logic cone isolation is an instantaneous $O(V + E)$ graph traversal:

```rust
pub struct LogicCone {
    pub target_net: NetId,
    pub fanin_cells: HashSet<CellId>,
    pub fanin_nets: HashSet<NetId>,
    pub logic_depth: usize,
    pub estimated_delay_ps: u64,
}
```

- **Fan-In Extraction**: Selecting a register input pin and pressing `F` dims all unrelated circuitry into 10% opacity, rendering an illuminated datapath cone tracing back to the primary inputs and source flip-flops.
- **Fan-Out Extraction**: Selecting a clock or enable net highlights its entire downstream load tree, displaying fan-out count and total lumped capacitive load.
- **Timing Heatmap Overlay**: Logic cells along the critical path are shaded on a gradient from Emerald Green (positive slack) to Crimson Red (negative slack).

---

## 5. Production Implementation & Studio Architecture

Axiom EDA implements this architecture natively in TypeScript, Canvas 2D, and React 19:

- **Graph & Slicing Model (`ui/src/engine/schematicModel.ts`)**:
  - Sugiyama layered DAG placement algorithm with Manhattan orthogonal wire channel routing.
  - **Zero-Turn Datapath Pin Alignment (`fixedY`)**: Precise horizontal alignment of connected pins ($Y_{\text{out}} = Y_{\text{in}}$) eliminating unnecessary bends. For example, in `logic_circuit`, 5 of 9 connections ($A \to \text{inv1}$, $B \to \text{and1.in2}$, $\text{and1.out} \to \text{and2.in1}$, $\text{and2.out} \to \text{or1.in1}$, $\text{or1.out} \to F$) are rendered as 100% straight horizontal lines with **0 turns**.
  - **Multi-Layer Destination Stepping**: For wires spanning multiple layers ($dx \ge 150\text{px}$, such as $C \to \text{and2}$ and $\text{inv2} \to \text{or1}$), the router keeps the wire running horizontally at its low source track across intermediate layers, and executes the vertical step in the open channel immediately preceding the destination node ($dstX - 28$).
  - **Generous Inter-Layer Padding (`layerSpacingX = 92px`)**: Spacious 92px routing channels provide clean corridors for vertical drops, eliminating cramped wire bundles.
  - **Obstacle-Aware Orthogonal Channel Router (`routeOrthogonalEdge`)**: Dynamically computes `KeepOutBox` clearance bounding boxes for all intermediate nodes (including a top margin of $y - 18$ to protect instance labels). Vertical trunks and horizontal corridors automatically jog around intermediate obstacles into open channels before or after the node.
  - $O(V + E)$ breadth-first critical fan-in and fan-out cone extractors.
  - Real-time setup slack calculation: $T_{\text{slack}} = T_{\text{clk}} - \sum t_{\text{cell}} - \sum t_{\text{net}} - t_{\text{setup}}$.
- **Hardware Schematic Viewer (`ui/src/components/SchematicViewer.tsx`)**:
  - 60+ FPS Canvas 2D engine with smooth mouse-wheel centered zooming (0.2x to 3.5x) and drag panning.
  - **Solid Background Text Knockout Plates**: Renders solid `#0c1017` protective plates behind instance labels (`inv1`, `inv2`, `and1`, `or1`) using `ctx.fillRect(textX - w/2 - 4, textY - 10, w + 8, 14)`, completely eliminating text/wire collisions and matching Vivado's clean visual hierarchy.
  - Semantic LOD: Macro blocks with heatmaps $\to$ Structural datapath MUXes/adders/registers $\to$ Primitive gates with Cranelift JIT machine instructions (`iadd`, `isub`, `band`, `icmp eq`).
  - Wire callout badges displaying real-time logic values from `SimulationState.signals` (toggled via `⚡ Live Values`, configured **off by default** for clean, clutter-free gate readability).
  - High-Precision Auto-Fit Framing: Exact geometric bounding box calculation across all cells and routed nets, dynamically auto-scaling and symmetrically centering the circuit to fill the visualizer pane cleanly with comfortable margins.
  - **Dynamic Midpoint Camera Anchoring & Continuous Canvas Resizing**: Continuous `ResizeObserver` on the container dynamically updates the canvas pixel buffer resolution (`canvas.width = Math.round(newW * dpr)`) on every frame of dragging the middle splitter handle. To prevent horizontal stretching or squishing, the world-space camera midpoint is mathematically shifted by half the dimension delta ($\Delta \text{offsetX} = \Delta W / 2$, $\Delta \text{offsetY} = \Delta H / 2$) while preserving constant zoom scale, keeping the circuit locked to the center of the visualizer pane.
  - Interactive Minimap camera viewport navigator.
  - 1-Click Cone Slicing with `[F]` (Fan-In), `[O]` (Fan-Out), and `[Esc]` (Clear) shortcuts, dimming unrelated logic to 12% opacity.
- **Split Studio Workspace (`ui/src/App.tsx`)**:
  - Studio view switcher: `[ 📈 Waveforms ]` | `[ 🔀 Schematic DAG ]` | `[ ◫ Split Studio ]`.
  - 3-way bidirectional cross-probing: selecting a gate/net highlights its trace in `WaveformViewer.tsx` and scrolls to/highlights its exact source lines in `HdlEditor.tsx`.


