# Axiom: High-Performance Rust Remake of AMD Vivado's HDL Engine & Studio

## 1. Project Manifesto & Core Mission

**Axiom** is an aerospace-grade, high-performance, cross-platform remake of AMD Vivado's Hardware Description Language (HDL) processing, simulation, and analysis engine, built natively in **Rust** and deployed live at **`https://axiom.aerovex.net`**.

Vivado is the industry standard for FPGA development, yet it suffers from severe legacy bloat: 100+ GB installations, sluggish Java Swing interfaces, multi-stage file-based elaboration pipelines, zero-time delta cycle black-boxing, and zero support for modern platforms like macOS (Apple Silicon) or standard web browsers.

**Axiom eliminates these limitations.** It provides a lightweight (<50 MB), lightning-fast, and deeply introspectable HDL engine paired with an elegant, modern dark-themed desktop (Tauri v2) and web application (WebAssembly).

---

## 2. Current Implementation Status & Production Deliverables

As of **Phase 12.6**, the entire core engine, compiler, scheduler, telemetry system, language server (LSP), linter, and modern studio are **fully implemented, tested, and active**:
- **55 / 55 Rust Workspace Tests Passing**: Comprehensive unit, integration, benchmark, conformance, and linter tests across all crates.
- **Dual-Runtime Execution & WebAssembly LSP**:
  - **Desktop Native**: Native Cranelift JIT compiling Verilog/SystemVerilog directly to x86_64 / AArch64 machine code in RAM with zero disk turnaround.
  - **In-Browser WebAssembly**: Pure client-side `wasm32-unknown-unknown` simulation kernel and in-RAM LSP static analysis linter running 100% in-browser with zero backend dependencies.
- **In-RAM Verilog/SystemVerilog LSP & Static Analysis Linter (`crates/lsp`)**:
  - 10 static design rules: syntax error mapping, blocking assignment in sequential blocks (`AXIOM_W001`), non-blocking in combinational blocks (`AXIOM_W002`), undriven nets (`AXIOM_W003`), unused signals (`AXIOM_W004`), multi-driver net contention (`AXIOM_E002`), transparent latch inference (`AXIOM_W006`), missing case default (`AXIOM_W007`), bit width mismatch (`AXIOM_W008`).
  - Standard JSON-RPC stdio daemon (`axiom lsp`) and colorized CLI reporter (`axiom lint <FILE>`).
- **Monaco Editor Integration with Custom Monarch Verilog Tokenizer**:
  - Dark engineering palette (`axiom-dark`), live debounced squiggly marker underlines (`monaco.editor.setModelMarkers`), hover tooltips with IEEE 1800 AST metadata, and autocompletion snippets/signals.
- **Problems & Linter Dock**: Dedicated collapsible dock tab with active diagnostic cards and 1-click jump-to-line navigation.
- **Production Web Deployment**: Live at **`https://axiom.aerovex.net/studio/`** (and docs at `https://axiom.aerovex.net/`) served via GitHub Pages with CNAME.
- **Vivado Project Management System & Welcome Launchpad**:
  - Starts cleanly from a "No Project Open" standpoint with zero pre-loaded clutter.
  - Aerospace-grade Welcome Launchpad with hero banner, "Create New Project" wizard card, "Open Project from File" (.json) importer, and 7-item interactive starter templates grid.
  - Authentic Vivado file sets (`sources_1`, `sim_1`, `constrs_1`), multi-file bundling, target FPGA parts (Artix-7, Zynq-7000, Kintex-7, Kintex UltraScale+, Axiom Virtual Silicon), active `[TOP]` module designation, and clean "Close Project" lifecycle.
- **De-Cramped Layout System**: Unified collapsible bottom dock (collapses to a 28px status bar), collapsible 38px sidebar, and 1-click panel maximization (`⛶`).

---

## 3. Repository & Workspace Architecture

The repository is organized as a Cargo multi-crate workspace and modern TypeScript/React frontend:

```
axiom/
├── Cargo.toml                          # Master workspace configuration
├── GEMINI.md                           # Single-source-of-truth project context for AI agents
├── todo.md                             # Master task & milestone priority tracker
├── install.sh / install.ps1            # Universal single-line installation scripts
│
├── crates/                             # Rust Simulation Engine Workspace
│   ├── core/                           # Four-state logic (0,1,X,Z), SimTime, diagnostics & spans
│   ├── syntax/                         # Streaming zero-copy lexer, preprocessor, and Pratt AST parser
│   ├── ir/                             # Elaborator, symbol tables, module hierarchy & BIR (IR)
│   ├── jit/                            # Cranelift in-RAM JIT backend & 4-state memory manager
│   ├── sim/                            # Stratified event queue, delta-cycle loop, glitch detector
│   ├── telemetry/                      # Dynamic power, PDN inductive sag ($V_{sag} = IR + L di/dt$), VCD/SAIF
│   ├── lsp/                            # In-RAM Verilog/SystemVerilog LSP 3.17 server & static analysis linter
│   ├── desktop/                        # DesktopEngine library and native Tauri v2 IPC handlers
│   ├── cli/                            # Unified CLI & In-RAM GUI server (embedded Zstd UI bundle)
│   └── wasm/                           # wasm-bindgen WebAssembly wrapper for in-browser simulation & LSP
│
├── ui/                                 # React 19 + TypeScript + PostCSS Web & Desktop Studio
│   ├── src/
│   │   ├── engine/
│   │   │   ├── engineBridge.ts         # Dual-runtime bridge (auto-detects Tauri IPC vs WebAssembly)
│   │   │   ├── projectModel.ts         # Vivado project model (sources_1, sim_1, constrs_1, templates)
│   │   │   ├── monacoVerilog.ts        # Monarch Verilog tokenizer, axiom-dark theme & LSP providers
│   │   │   ├── schematicModel.ts       # Synthesizes hardware netlist DAG for schematic viewer
│   │   │   ├── timingModel.ts          # Static timing analysis, slack radar, CDC matrix, energy treemap
│   │   │   └── sampleDesigns.ts        # 7 production Verilog hardware systems
│   │   ├── components/
│   │   │   ├── Header.tsx              # Execution control (Run, Step, +1ns, +100ps, Step δ, Reset)
│   │   │   ├── Sidebar.tsx             # Collapsible sidebar (Sources explorer vs Netlist hierarchy)
│   │   │   ├── ProjectManager.tsx      # Vivado file sets explorer, top-module picker, templates
│   │   │   ├── NewProjectModal.tsx     # Vivado project creation wizard with target FPGA part selection
│   │   │   ├── AddSourceModal.tsx      # Add source dialog for sources_1, sim_1, constrs_1
│   │   │   ├── HdlEditor.tsx           # Monaco multi-tab editor with live markers, [TOP] tag, breadcrumb
│   │   │   ├── UnifiedBottomDock.tsx   # Collapsible dock: Console & REPL, Problems & Linter, Power, Glitches
│   │   │   ├── WaveformViewer.tsx      # Multi-radix traces, dual cursors, delta-cycle accordion
│   │   │   ├── SchematicViewer.tsx     # GPU-accelerated netlist DAG, semantic LOD, logic cone slicer
│   │   │   ├── VirtualLabRack.tsx      # DIP switches, buttons, 7-seg LEDs, UART/SPI/PWM/RISC-V bays
│   │   │   ├── TimingRadarViewer.tsx   # Slack waterfall, CDC matrix, hierarchical energy treemap
│   │   │   ├── OmnibarModal.tsx        # Ctrl+K Spotlight-style command palette & fuzzy finder
│   │   │   └── ResizableSplitter.tsx   # Zero-dependency draggable splitter handles
│   │   └── styles/theme.css            # Dark engineering aesthetic design system
│   └── dist/                           # Production web bundle (pre-compiled into CLI)
│
├── docs/                               # VitePress Documentation Portal (https://axiom.aerovex.net)
├── analysis/                           # Axiom Architectural Blueprints (14 detailed specs)
└── vivadoanalysis/                     # AMD Vivado Reverse-Engineering & Architecture Critique
```

---

## 4. Key Architectural Subsystems

### 4.1. In-RAM JIT Machine Code & WebAssembly Engine
- **No External Compilers**: Verilog/SystemVerilog parsing and elaboration occur entirely in Rust.
- **Cranelift Native JIT**: Directly emits native machine instructions into executable memory in RAM in milliseconds.
- **WebAssembly Client**: Compiles to `wasm32-unknown-unknown`, allowing browser simulation without server computation.
- **Contiguous 4-State Arena**: All signal states reside in contiguous `values: Box<[u64]>` and `masks: Box<[u64]>` memory, guaranteeing L1/L2 cache locality.

### 4.2. Stratified Event Scheduler & Delta-Cycle Introspection
- **Full IEEE 1800 Compliance**: Implements Stratified Event Queue phases: Active $\rightarrow$ Inactive $\rightarrow$ NBA (Non-Blocking Assignments) $\rightarrow$ Observed $\rightarrow$ Reactive.
- **Granular Stepping APIs**:
  - `engine.tick(delta_ps)`: Advances physical simulation time by arbitrary picoseconds.
  - `engine.step_delta()`: Advances a discrete zero-time $\delta$-cycle within current time.
- **Glitch & Hazard Radar**: Automatically intercepts and flags zero-time combinational glitches (static-0, static-1, dynamic hazards) that Vivado conceals.

### 4.3. Physics-Informed Voltage, Energy & Power Telemetry
- **Dynamic Power**: Modeled from net capacitance, voltage rails, and toggle rate:
  $$P_{\text{dynamic}} = \frac{1}{2} C_{\text{net}} V_{\text{dd}}^2 f \alpha$$
- **Inductive PDN Droop**: Models power distribution network inductance ($V_{\text{sag}} = IR + L \frac{di}{dt}$), flagging micro-spikes during clock transitions.
- **Export Standards**: Generates standard Value Change Dump (VCD) and Switching Activity Interchange Format (SAIF 2.0).

### 4.4. Vivado Project Management & De-Cramped Ergonomic Studio
- **Authentic Vivado File Sets**:
  - `Design Sources (sources_1)`: Verilog / SystemVerilog RTL modules.
  - `Simulation Sources (sim_1)`: Testbenches with stimulus generators.
  - `Constraints (constrs_1)`: Timing & pin constraints (`timing.xdc`).
- **Target Silicon Devices**: Artix-7 (`xc7a35t`, `xc7a100t`), Zynq-7000 (`xc7z020`), Kintex-7 (`xc7k325t`), Kintex UltraScale+ (`xcku5p`), Axiom Virtual Silicon.
- **Ergonomic De-Cramping**:
  - **Unified Bottom Dock**: Collapsible to a 28px status bar, liberating ~250px of vertical space.
  - **Collapsible Sidebar**: Shrinks to a 38px vertical icon strip, liberating 222px of horizontal space.
  - **1-Click Panel Maximization (`⛶`)**: Expands Editor, Waveforms, Schematic DAG, or Virtual Lab to 100% full screen.

---

## 5. Strict Operating Guidelines & Git Workflow Policy

> [!NOTE]
> **Git Workflow Policy:**
> **You can commit and push after every change.**
> Always ensure that changes are verified (e.g., `npm run build`, `cargo test`), clean, and committed with clear descriptive messages.
> Destructive Git operations (such as `git reset --hard`, `git clean -f`, force-pushing `git push --force`, or destructive rebasing) remain strictly forbidden to preserve repository history integrity.

---

## 6. Build, Test & Verification Commands

When working in the repository, verify changes using these standard commands:

- **Build Rust Workspace & Test Suite**:
  ```bash
  cargo test --workspace
  ```
  *(Ensures all 48 unit, integration, benchmark, and conformance tests pass)*

- **Build TypeScript / React UI**:
  ```bash
  npm --prefix ui run build
  ```
  *(Runs TypeScript `tsc` check and Vite production bundling)*

- **Run UI Development Server**:
  ```bash
  npm --prefix ui run dev
  ```
  *(Launches Vite dev server on `http://localhost:3000/`)*

- **Build Rust Desktop / CLI Binary**:
  ```bash
  cargo build --release --bin axiom
  ```

- **Run Cli In-RAM GUI Server**:
  ```bash
  cargo run --bin axiom -- gui
  ```

---

## 7. Master Documentation & Analysis Index

For deep architectural and implementation specifications, refer to:

1. **`analysis/analysis.md`**: Master index of the 14 comprehensive technical blueprints:
   - `01_engine_architecture.md`: Multi-crate workspace & zero-allocation memory model.
   - `02_lexer_parser_ast.md`: Zero-copy lexer, preprocessor & resilient AST parser.
   - `03_elaboration_and_netlist_ir.md`: Elaboration pipeline & BIR netlist intermediate representation.
   - `04_jit_machine_code_compiler.md`: Cranelift in-RAM JIT & WebAssembly codegen.
   - `05_event_scheduler_and_delta_stepping.md`: Stratified IEEE 1800 event queue & delta API.
   - `06_voltage_energy_telemetry_model.md`: Physics-informed dynamic power & PDN droop.
   - `07_api_and_runtime_interface.md`: Embeddable Rust, C-ABI & WASM API.
   - `08_desktop_and_web_ui.md`: Tauri v2 + React 19 + PostCSS desktop and web studio.
   - `09_schematic_dag_and_synthesis_viewer.md`: GPU-accelerated hardware DAG & cone slicer.
   - `10_virtual_lab_and_stimulus_rack.md`: Zero-JTAG virtual instruments & stimulus injection.
   - `11_timing_radar_and_slack_waterfall.md`: Setup/hold slack waterfall & CDC matrix.
   - `12_hierarchical_energy_treemap_and_thermal.md`: Silicon energy treemap & SSN model.
   - `13_omnibar_and_scripting_repl.md`: Ctrl+K Omnibar palette & interactive shell.

2. **`vivadoanalysis/`**: Deep comparative analysis of AMD Vivado's internal tools (`xvlog`, `xelab`, `xsim`, `ug907`, TCL CLI).

3. **`todo.md`**: Full milestone tracker detailing completed phases and future enhancements.
