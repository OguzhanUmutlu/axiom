# Axiom EDA — Implementation Todo & Progress Tracker

> **Feature Overview:**
> **Axiom** (formerly Betterado) is an aerospace-grade, high-performance, cross-platform Electronic Design Automation (EDA) suite and hardware simulation engine built natively in **Rust** under the **Aerovex** platform (`https://axiom.aerovex.com`).
> 
> Axiom eliminates decades of legacy EDA bloat by providing:
> 1. **In-RAM Cranelift JIT Compilation**: Direct translation of Verilog/SystemVerilog into native machine code in milliseconds with zero disk turnaround.
> 2. **Manual Delta-Time Tick & Event Queue Inspection**: Full visibility into zero-time $\delta$-cycles, exposing combinational race conditions and glitches hidden by Vivado.
> 3. **Physics-Informed Silicon Power & Voltage Telemetry**: Live dynamic power ($P = \frac{1}{2} C V^2 f \alpha$) and PDN inductive voltage sag modeling ($V_{sag} = IR + L\frac{di}{dt}$) with VCD & SAIF 2.0 exporters.
> 4. **Dual-Target Desktop & Web UI**: Built with Tauri v2, React 19, TypeScript, PostCSS, and Vite, running seamlessly on Linux, macOS (Apple Silicon), Windows, and in standard web browsers.
> 5. **VitePress Documentation Portal**: Hosted on GitHub Pages with custom domain redirect to `axiom.aerovex.com`.

---

## In Progress

- [ ] **Brand Identity & Project Renaming to Axiom - [P0]**
  - [x] Rename GitHub repository from `betterado` to `axiom` (`OguzhanUmutlu/axiom`).
  - [x] Update GitHub description, topic tags, and homepage URL (`https://axiom.aerovex.com`).
  - [ ] Update workspace manifest, crate naming, and internal module paths to `axiom-*`.
  - [ ] Update UI branding tokens, headers, and metadata to Axiom EDA.
  - [ ] Overhaul `README.md` with executive-grade architecture diagrams, benchmarks, and badges.
- [ ] **VitePress Documentation Portal & GitHub Pages CI/CD (`axiom.aerovex.com`) - [P0]**
  - [ ] Initialize `docs/` VitePress suite with Obsidian dark theme matching Axiom UI.
  - [ ] Configure `docs/public/CNAME` with `axiom.aerovex.com`.
  - [ ] Author documentation chapters: Guide, Core Architecture, Vivado Feature Parity, and CLI Reference.
  - [ ] Implement `.github/workflows/deploy-docs.yml` automated GitHub Pages deployment.

---

## Todo

### Vivado Next-Generation Features Roadmap
- [ ] **[P1] Interactive RTL Schematic & Netlist DAG Graph Viewer**: Hardware-accelerated schematic DAG rendered directly from elaborated BIR netlist with bidirectional waveform cross-probing.
- [ ] **[P1] Static Timing Analysis (STA) & SDC/XDC Constraint Engine**: Setup/hold slack calculation, Worst Negative Slack (WNS), Total Negative Slack (TNS), and critical timing path pin-to-pin delay tracing.
- [ ] **[P2] Logic Synthesis & FPGA Technology Mapping**: Combinational logic mapping into 6-input LUTs (`LUT6`), flip-flops (`FDRE`), and fast carry chains (`CARRY4`/`CARRY8`).
- [ ] **[P2] Interactive Scriptable Tcl Shell & Embedded REPL**: Headless and in-UI command console supporting standard Tcl automation (`run`, `step_delta`, `get_nets`, `report_power`).
- [ ] **[P2] Virtual I/O (VIO) & Software Integrated Logic Analyzer (ILA)**: Virtual lab bench with interactive switches, pushbuttons, LEDs, 7-segment displays, and triggerable software logic analyzer.
- [ ] **[P2] VHDL-2008 Front-End Parser (`xvhdl` Parity)**: Native IEEE 1076-2008 parser enabling mixed-language Verilog/VHDL elaboration.

---

## Completed

- [x] **Phase 6: Verification, Benchmarking & Tooling Parity - [P2]**
  - [x] Standalone Headless CLI Driver (`crates/betterado-cli`): In-RAM compilation (`compile`), headless simulation (`run`), batch IEEE 1364 VCD & SAIF 2.0 dumping, and benchmark command.
  - [x] Vivado Output Conformance Test Suite: Multi-fixture validation (ALU, Counter, Hierarchy, FIFO) verifying deterministic 4-state logic, glitch hazards, and SAIF toggle activity against Vivado reference patterns.
  - [x] In-RAM Performance Benchmark Suite: End-to-end latency measurement across Lexing, AST Parsing, Elaboration, Cranelift JIT compilation (sub-3 ms total JIT turnaround), and simulation throughput (>780,000 events/sec).

- [x] **Phase 5: Modern Dark-Themed Desktop & Web Application (Tauri + React + PostCSS) - [P1]**
  - [x] Tauri v2 & Vite Scaffold: Dual-mode build setup (`betterado-desktop` crate + `ui/` React 19 + TypeScript + Vite production bundle).
  - [x] React 19 + TypeScript + PostCSS UI Shell: Sleek, minimalist, dark-themed engineering dashboard (Obsidian / Linear style).
  - [x] High-Performance Waveform Viewer (Canvas 2D): 60+ FPS digital waveform viewer with single-bit levels, bus transition envelopes, time ruler, cursor inspection, and zero-time delta-cycle hazard magnifier.
  - [x] Voltage & Power Telemetry Graphing: Real-time dynamic power (mW), transient current spikes (mA), PDN voltage sag ($V_{sag} = IR + L\frac{di}{dt}$), and summary energy metrics cards.
  - [x] Interactive Simulation Control Console: Step physical time (+1 ns, +100 ps), step discrete delta cycle (`step_delta()`), run free/pause, reset.
  - [x] HDL Code Editor & Hierarchy Explorer: Line-numbered Verilog code editor with live syntax tabs, elaborated netlist hierarchy explorer, preset design fixture switcher (ALU, Counter with Glitches, Hierarchical Core), and IEEE 1364 VCD / SAIF 2.0 exporters.

- [x] **Phase 0: Deep Vivado & HDL Research & Architecture Analysis**
  - [x] Extract and analyze Vivado's HDL simulation pipeline (`xvlog`, `xvhdl`, `xelab`, `xsim`).
  - [x] Analyze Vivado's Power Analysis methodology (UG907, SAIF, vectorless estimation, switching activity).
  - [x] Populate `vivadoanalysis/` with 8 exhaustive reference documentation and architecture breakdowns.
  - [x] Formulate project philosophy and design manifesto in `GEMINI.md` (recording all user techniques and git safety constraints).
  - [x] Author comprehensive internal architecture specifications in `analysis/` with master index `analysis/analysis.md`.
  - [x] Formulate Phase 1 Industrial Implementation Plan.
- [x] **Phase 1: Core HDL Engine Foundation (Rust Core & IR) - [P0]**
  - [x] Setup multi-crate Cargo workspace (`crates/betterado-core`, `crates/betterado-syntax`, `crates/betterado-ir`).
  - [x] Implement fundamental simulation types in `betterado-core`: `SimTime`, IEEE 1800 4-state logic bit representations (`Logic4`, `LogicVector`), source spans, and diagnostic reporter.
  - [x] Implement streaming zero-allocation lexer & preprocessor in `betterado-syntax` for Verilog-2005 & SystemVerilog (`define`, `ifdef`, `include`).
  - [x] Implement AST data structures and Pratt recursive-descent parser with syntax error recovery.
  - [x] Implement hierarchical elaborator in `betterado-ir`: module instantiation traversal, constant folding, parameter resolution, generate block unrolling, and port binding.
  - [x] Formulate Betterado Intermediate Representation (BIR) dataflow graph with explicit sensitivity mapping.
  - [x] Verify complete pipeline against test fixtures (`alu.v`, `counter.v`, `hierarchy.v`) with 100% test pass rate.
- [x] **Phase 2: JIT Machine Code Compiler & In-RAM Execution Engine - [P0]**
  - [x] Setup Cranelift JIT compiler crate `betterado-jit` with in-RAM code emission.
  - [x] Implement contiguous dual-word state arena (`SimStateArena`) with 64-bit aligned value and mask storage for O(1) 4-state logic operations.
  - [x] Implement in-RAM Cranelift JIT codegen compiling continuous assignments and arithmetic/logic expressions directly into executable machine code in memory.
  - [x] Implement portable and WebAssembly (`wasm32-unknown-unknown`) execution path (`PortableEvaluator`) with support for procedural blocks, non-blocking assignments (NBAs), `if/else`, and `case` branches.
  - [x] Verify end-to-end JIT and portable execution against realistic ALU and counter fixtures with 100% test pass rate across the workspace.
- [x] **Phase 3: Stratified Event Scheduler & Delta-Time Stepping API - [P0]**
  - [x] Setup simulation kernel crate `betterado-sim`.
  - [x] Implement IEEE 1800 Stratified Event Queue (`StratifiedEventQueue`) with min-heap ordering by `(SimTime, Delta, Region, SeqId)`.
  - [x] Implement caller-controlled Manual Delta-Time Tick Engine: `tick(delta_time)` and discrete delta cycle stepping: `step_delta()`.
  - [x] Implement zero-overhead combinational glitch detector (`GlitchDetector`) capturing static and dynamic hazards across zero-delay delta cycles.
  - [x] Implement event listener and trace observation framework (`SimEventListener`, `SimTraceRecorder`).
  - [x] Implement fast in-RAM simulation state snapshotting and rollback (`SimSnapshot`, `CheckpointId`) for instant timeline scrubbing.
  - [x] Verify complete event scheduling and delta stepping against clocked counter and ALU fixtures with 100% pass rate (30 workspace tests).
- [x] **Phase 4: Voltage, Energy & Power Telemetry Engine - [P1]**
  - [x] Setup telemetry crate `betterado-telemetry`.
  - [x] Implement lumped physical capacitance model (`NetCapacitanceModel`) capturing pin, wire, and fanout capacitance per net and bit.
  - [x] Implement power distribution network (PDN) impedance model (`PowerRail`, `PdnModel`) simulating dynamic current spikes and voltage sag ($V_{sag} = IR + L\frac{di}{dt}$).
  - [x] Implement physics-informed switching activity and energy accumulator (`TelemetryCollector`) producing real-time `TelemetryFrame` packets for visualizers.
  - [x] Implement IEEE 1364 Value Change Dump (`VcdWriter`) exporter for digital waveform inspection.
  - [x] Implement Switching Activity Interchange Format (`SaifWriter`) exporter for 100% interoperability with AMD Vivado `read_saif`.
  - [x] Verify all telemetry models, voltage sag calculations, and exporter formats with 100% pass rate (36 workspace tests).
