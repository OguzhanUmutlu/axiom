# Axiom Complex EDA Visualizations & Hardware Ergonomics

## 1. Executive Summary & Multi-Domain Visual Ergonomics

Electronic Design Automation (EDA) applications present unique visual design challenges. A single hardware bug may originate in behavioral RTL code, manifest structurally across an elaborated gate-level netlist DAG, produce a glitch hazard in a digital waveform transition, and cause an inductive voltage drop on the silicon power distribution network (PDN).

Axiom coordinates **five synchronized visual domains**:
1. **Monaco HDL Code Editor** (Textual RTL & In-RAM LSP Linter).
2. **GPU / Canvas Schematic DAG Viewer** (Topological Netlist & Logic Cone Slicer).
3. **Digital Waveform Viewer** (Temporal Transitions, Multi-Radix Vectors & Delta Cycles).
4. **Virtual Lab Hardware Rack** (Tactile Breadboard Controls & Protocol Analyzers).
5. **Physics-Informed Silicon Telemetry** (Power Sweep Dial, PDN Sag & Slack Waterfall).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 SYNCHRONIZED MULTI-DOMAIN VISUALIZATION                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ HDL Editor:     assign w2 = w1 & B;   <--- Line Highlight / Cross-Probe    │
│                                            ▲                                │
│                                            │                                │
│ Schematic DAG:  [NOT w1] ---> [AND w2] ----+ <--- Gate Selection Halo       │
│                                 ▲          │                                │
│                                 │          ▼                                │
│ Waveforms:      w2: __________/-----\_____ <--- Trace Transition Glitch    │
│                                            │                                │
│                                            ▼                                │
│ Virtual Lab:    Input B: [SW1 ON]  =====> LED Output F: [GLOWING AMBER]     │
│                                            │                                │
│                                            ▼                                │
│ Telemetry:      P_dynamic: 14.2 mW | V_sag: 0.982 V (-18 mV Spike)          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Monaco HDL Editor Ergonomics & In-RAM LSP Feedback

### 2.1. Monarch Verilog / SystemVerilog Tokenizer
Monaco Editor (`HdlEditor.tsx`) implements a custom Monarch grammar (`monacoVerilog.ts`) tailored specifically to IEEE 1800-2017:
- **Keywords**: Cyan (`#38bdf8`) for structural declarations (`module`, `endmodule`, `input`, `output`, `wire`, `reg`, `logic`).
- **Control Flow**: Blue (`#60a5fa`) for procedural blocks (`always_comb`, `always_ff`, `case`, `if`, `else`).
- **Directives & Macros**: Pink (`#f472b6`) for compiler directives (`` `timescale``, `` `define``, `` `ifdef``).
- **Literals & Sized Radixes**: Purple (`#c084fc`) for sized constants (`8'hFF`, `1'b0`, `32'd1000`).
- **Operators**: Emerald (`#34d399`) for non-blocking (`<=`) and blocking (`=`) assignments.

### 2.2. Debounced In-RAM Linter Squiggles (200ms)
Unlike external compilers that run in batch scripts, Axiom runs an AST linter directly in Rust in RAM:
- **Debounce Window**: 200ms after the last keystroke, preventing editor flicker while typing.
- **Marker Types**:
  - **Red Squiggle (Error)**: Syntax errors (`AXIOM_E001`), multi-driver net contention (`AXIOM_E002`).
  - **Amber Squiggle (Warning)**: Blocking assignments in sequential blocks (`AXIOM_W001`), transparent latch inference (`AXIOM_W006`), bit-width mismatch (`AXIOM_W008`).
  - **Blue Squiggle (Info)**: Unused nets (`AXIOM_W004`), undriven signals (`AXIOM_W003`).
- **Interactive Marker Pill**: The tab bar displays an active status pill (`✓ Clean` in green, or `2 Warnings` in amber). Clicking it jumps directly to the Problems & Linter dock tab.

---

## 3. GPU / Canvas Schematic DAG & Logic Cone Slicing

Hardware netlists grow into thousands of nodes. Rendering thousands of DOM nodes causes severe browser lag; Axiom utilizes an **HTML5 Canvas 2D DAG Engine** (`SchematicViewer.tsx`):

### 3.1. Semantic Level-of-Detail (LOD)
The camera viewport dynamically adjusts rendering detail across three discrete zoom scales:
1. **Macro Overview ($< 0.4\times$)**:
   - Simplifies complex gate clusters into module boundary envelopes.
   - Renders thick bus pipes instead of individual signal wires.
   - Eliminates pin labels and text to avoid sub-pixel clutter.
2. **Structural Topology ($0.4\times - 1.2\times$)**:
   - Renders standard IEEE logic gate symbols (AND, OR, NOT, XOR, NAND, NOR, MUX).
   - Draws orthogonal net routing paths with clean right-angle bends.
   - Shows instance names (`inv1`, `and1`, `or1`) and wire labels (`w1`..`w4`).
3. **Gate Inspection ($> 1.2\times$)**:
   - Expands pin connection terminals with live digital value probes (`0`, `1`, `Z`, `X`).
   - Renders boolean algebraic notation and JIT operator mappings on hover tooltips.

```
Macro LOD (< 0.4x)         Structural LOD (0.4x - 1.2x)       Gate LOD (> 1.2x)
┌──────────────────────┐   ┌──────────────────────────────┐   ┌──────────────────────────────┐
│  ┌────────────────┐  │   │      ┌─────┐                 │   │    A:0  ┌─────┐              │
│  │ logic_circuit  │  │   │  A ──┤ NOT ├── w1 ┐          │   │  ───────┤ NOT ├── w1:1 ┐       │
│  │ (9 Cells)      │  │   │      └─────┘      │          │   │         └─────┘      │       │
│  │                │  │   │      ┌─────┐      ▼ ┌─────┐  │   │         ┌─────┐      ▼ ┌─────┐
│  └────────────────┘  │   │  B ──┤ AND ├───────►┤ OR  │  │   │    B:1 ─┤ AND ├────────►┤ OR  │
└──────────────────────┘   │      └─────┘        └─────┘  │   │         └─────┘        └─────┘
                           └──────────────────────────────┘   └──────────────────────────────┘
```

### 3.2. 1-Click Critical Logic Cone Slicing
When debugging timing slack violations or logic errors, engineers must isolate the causal path from the surrounding sea of logic:
- **Fan-in Slicer (`F` Key)**: Isolates the backward combinational dependency cone driving the selected node back to primary inputs or register boundaries.
- **Fan-out Slicer (`O` Key)**: Isolates the forward tree driven by the selected node out to primary outputs or register data pins.
- **Background Attenuation**: Unrelated nodes and nets are dimmed to 12% opacity, immediately highlighting the critical path with a glowing cyan halo.

### 3.3. Ergonomic Camera Framing & Clean Default State
- **Live Values Off by Default**: Live signal value callouts along wire paths are disabled by default, ensuring that initial schematics present clean, high-contrast, uncluttered gate topologies without distracting badges. When deep logic probing is desired, engineers toggle ` Live Values` with a single click.
- **High-Precision Auto-Fit Geometry**: The camera framing engine computes the exact tight geometric bounding box across all cells and routed nets, dynamically scaling up to 1.35x and symmetrically centering the circuit horizontally and vertically with comfortable 36px–40px margins.
- **Calibrated Layer Spacing**: Layer horizontal pitch is calibrated to 64px (eliminating excessive empty wire stretches), allowing full combinational systems to occupy the viewport at 100%+ scale.

---

## 4. Digital Waveform Engine & Zero-Time Delta Introspection

The digital waveform viewer (`WaveformViewer.tsx`) delivers high-throughput hardware tracing capable of rendering 100,000+ transitions at 60 FPS.

### 4.1. Multi-Radix Bus Vector Exploder
Multi-bit vectors (such as `instr[31:0]`, `pc[31:0]`, or `alu_result[7:0]`) can be viewed either as unified hexagonal transition envelopes or exploded into individual bit lanes:
- **Radix Switching**: Individual signals can be formatted on-the-fly without re-simulating:
  - `Hex` (`0x5A`)
  - `Binary` (`8'b01011010`)
  - `Unsigned Decimal` (`90`)
  - `Signed Decimal` (`+90`)
  - `ASCII` (`'Z'`)
- **Bus Expansion**: Clicking the vector chevron expands the bus into indexed sub-rows (`[7]`, `[6]`, ..., `[0]`), allowing bit-level inspection of carry propagation and register loading.

### 4.2. Zero-Time Delta Cycle Accordion Viewer
In standard simulators (Vivado `xsim`), multiple signal transitions occurring within the same simulation timestamp are collapsed into a single final value, concealing hazardous intermediate races.

Axiom introduces the **Delta-Cycle Accordion**:
- When multiple delta cycles ($\delta_0 \to \delta_1 \to \delta_2$) occur at time $t$, a distinctive amber hazard ribbon is rendered across the trace.
- Clicking the hazard expands an inline horizontal sub-scale revealing the internal zero-time transitions.
- **Hazard Classification Badges**:
  - *Static-0 Hazard*: $0 \to 1 \to 0$ transition where output should have remained stably Low.
  - *Static-1 Hazard*: $1 \to 0 \to 1$ transition where output should have remained stably High.
  - *Dynamic Hazard*: Multiple oscillations before settling.

```
Timeline (ns):        10.00 ns                      10.05 ns
Waveform Trace:       ___________/¯¯¯¯¯¯\___________/¯¯¯¯¯¯¯¯¯¯¯¯¯
Delta Accordion:                 [ δ0: 0 | δ1: 1 | δ2: 0 ]  <-- Static-0 Hazard Ribbon!
```

---

## 5. Virtual Lab Rack & Hardware Breadboard Skeuomorphism

Hardware engineers rely heavily on tactile prototyping before silicon fabrication. `VirtualLabRack.tsx` provides an interactive virtual instrument rack that bridges behavioral RTL with physical FPGA hardware.

### 5.1. Instrument Bay Catalog
- **8-Bit Tactile DIP Switch Bank**: Realistic SPST rocker switches with LED indicator pills and batch operations (`All 0`, `All 1`, `Invert`).
- **Rotary Quadrature Encoder Dial**: Continuous stimulus injector with tactile detents and step buttons, ideal for PWM duty modulation or motor speed control.
- **Dual 7-Segment Hex Display**: Authentic LED segment layout with realistic optical bloom filters decoding active 8-bit bus signals in real time.
- **Protocol Analyzer Bays**:
  - *UART Console*: Full-duplex ASCII terminal with baud generator, framing status, and transmission trigger.
  - *SPI Bus Monitor*: Mode 0–3 CPOL/CPHA configuration with live MOSI/MISO byte inspector.
  - *PWM Inverter Bay*: Duty cycle percentage gauge, dead-time clock cycle adjustment, and complementary gate drive monitors (`Gate High` / `Gate Low`).
  - *RISC-V SoC Register File*: Live 32-bit register table (x0–x7 in hex and decimal) with single-instruction stepping.
  - *Combinational Truth Table HUD*: Real-time 8-state truth table (`000` through `111`) highlighting the currently active row based on live switch inputs.

---

## 6. Physics-Informed Power & Voltage Telemetry Ergonomics

In sub-micron semiconductor design, dynamic power dissipation and power distribution network (PDN) voltage droop dictate whether a chip functions or fails. Axiom integrates physics-informed telemetry directly alongside digital logic:

### 6.1. Dynamic Power Sweeping Analog Dial
- **Mathematical Basis**: Evaluated per net toggle using lumped wire, pin, and fanout capacitance:
  $$P_{\text{dynamic}} = \frac{1}{2} C_{\text{net}} V_{\text{dd}}^2 f \alpha$$
- **Analog Sweeping Meter**: A circular analog gauge with a sweeping gradient arc (Emerald $\to$ Amber $\to$ Rose) displaying instantaneous dissipation in milliwatts (mW).
- **Activity Indicator**: Switching activity factor ($\alpha$) gauge indicating the percentage of design nets toggling per clock cycle.

### 6.2. Inductive PDN Voltage Droop Graphing
- **Mathematical Basis**: Modeled from power supply loop inductance and transient switching spikes:
  $$V_{\text{sag}} = I R + L \frac{di}{dt}$$
- **Synchronized Voltage Graph**: Plotted directly in the bottom dock, flagging micro-spikes where heavy concurrent switching causes the core rail ($1.0\text{V}$) to dip below operational margins ($<0.95\text{V}$), exposing supply hazards before tape-out.
