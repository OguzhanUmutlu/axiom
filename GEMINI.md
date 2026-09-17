# Betterado: A High-Performance Rust Remake of Vivado's HDL Processor & Simulator

## 1. Project Manifesto & Core Mission

**Betterado** is a ground-up, high-performance, cross-platform remake of AMD Vivado's Hardware Description Language (HDL) processing, simulation, and analysis engine, built natively in **Rust**.

Vivado is the undisputed industry standard for FPGA development, yet it suffers from decades of technical bloat: 100+ GB installations, sluggish Java Swing interfaces, multi-stage file-based elaboration pipelines, and a complete lack of support for modern operating systems like macOS (Apple Silicon) or web browsers. Furthermore, Vivado treats zero-time delta cycles as a black box, hiding critical combinational glitches and race conditions from the engineer.

**Betterado dismantles these limitations.** It provides a lightweight (<50 MB), lightning-fast, and deeply introspectable HDL engine paired with an elegant, modern dark-themed desktop and web application.

---

## 2. Core Architecture & Architectural Techniques

As defined by the project specification, Betterado incorporates the following core technical capabilities and architectural techniques:

### 2.1. In-RAM JIT Machine Code Execution Engine
- **End-to-End HDL Understanding**: Full lexical analysis, AST parsing, and semantic elaboration of Verilog and SystemVerilog without relying on external compilers (GCC/Clang) or intermediate C++ file dumps.
- **Direct JIT Compilation to Machine Code**: The engine compiles elaborated hardware netlists and procedural blocks directly into native machine code in RAM using **Cranelift** (x86_64, AArch64).
- **Zero-Disk Turnaround**: The entire parse-elaborate-compile pipeline executes entirely in memory within milliseconds, enabling instant test iteration.
- **Cache-Optimized 4-State State Arena**: Signals and registers are laid out contiguously in memory using dual-vector bit representations (`value` and `mask`) for hyper-efficient 4-state logic (0, 1, X, Z) execution.

### 2.2. Manual Delta-Time Tick & Event Inspection API
- **Caller-Controlled Stepping**: The simulation engine exposes an embeddable API that allows the caller to manually advance simulation time by an arbitrary delta time:
  ```rust
  engine.tick(delta_time);
  ```
- **Granular Delta-Cycle Step**: In addition to physical time advances, the caller can execute single discrete delta cycles ($\delta$-steps) within zero simulation time:
  ```rust
  engine.step_delta();
  ```
- **Event-Driven Introspection**: The engine emits granular event callbacks letting the caller know *what happens when*:
  - Net transitions and signal value changes.
  - Process activations and sensitivity list evaluations.
  - Active, Inactive, and Non-Blocking Assignment (NBA) phase updates.
  - Combinational glitch detection and race-condition warnings.

### 2.3. Real-Time Voltage, Energy & Power Telemetry
- **Physics-Informed Dynamic Power**: Implements dynamic power modeling based on physical capacitance, switching activity, and voltage supply rails:
  $$P_{\text{dynamic}} = \frac{1}{2} C_{\text{net}} V_{\text{dd}}^2 f \alpha$$
- **Instantaneous Energy & Current Spikes**: Calculates instantaneous energy dissipation per event ($E = \frac{1}{2} C V^2$), capturing micro-spikes during clock transitions that Vivado's static reports miss.
- **Live Synchronized Graphing**: Streams real-time telemetry over high-frequency ring buffers to power and voltage visualizers in the UI, perfectly synchronized with digital logic waveforms.

### 2.4. Cross-Platform Desktop & Web Architecture
- **OS-Independent Desktop Application**: Built with **Tauri v2**, running natively on Linux, macOS (Intel & Apple Silicon), and Windows.
- **Modern Dark-Themed UI**: Designed with a clean, organized, dark aesthetic using **React 19**, **TypeScript**, and **PostCSS**.
- **Vite Development & Web Parity**: Built using Vite, allowing instant development in standard browsers.
- **Native WebAssembly (WASM) Engine Support**: The Rust HDL engine compiles to WebAssembly (`wasm32-unknown-unknown`), enabling 100% in-browser simulation and analysis with zero backend server dependencies.

---

## 3. Strict Operating Guidelines & Constraints

> [!CRITICAL]
> **Git Repository Constraint:**
> For now, **DO NOT commit, push, or touch Git in any destructive way whatsoever**, including `git reset`, `git checkout`, `git clean`, `git rebase`, or force operations. All repository files and branch states must remain preserved and non-destructive.

---

## 4. Documentation & Analysis Organization

The project maintains two distinct, highly organized research and architecture domains:

1. **`vivadoanalysis/`**:
   Comprehensive, deep analysis of AMD Vivado's architecture, documentation, and subsystems:
   - `01_vivado_design_suite_overview.md`
   - `02_hdl_parsing_xvlog_xvhdl.md`
   - `03_elaboration_pipeline_xelab.md`
   - `04_simulation_kernel_xsim.md`
   - `05_power_and_voltage_analysis_ug907.md`
   - `06_waveform_and_inspection_subsystem.md`
   - `07_vivado_architecture_critique.md`
   - `08_cli_and_tcl_reference.md`

2. **`analysis/`**:
   In-depth technical architecture and specification for **Betterado itself** (the actual project, not Vivado). The master index is located at:
   - `analysis/analysis.md` (stores the directory tree structure and defines the exact purpose of every markdown file).

3. **`todo.md`**:
   Master task and priority tracker containing:
   - Feature explanation at the top.
   - `## In Progress` section.
   - `## Todo` section with clear priority ratings (`[P0]`, `[P1]`, `[P2]`).
   - `## Completed` section tracking finished items with filled checkboxes `[x]`.

---

## 5. Development Roadmap Summary

- **Phase 1 (Immediate Target)**: Core HDL Engine Foundation (Rust Lexer, Parser, AST, Elaboration & Netlist IR).
- **Phase 2**: JIT Machine Code Compiler (Cranelift in-RAM compilation & WASM codegen).
- **Phase 3**: Stratified Event Scheduler & Manual Delta-Time Tick Engine.
- **Phase 4**: Voltage, Energy & Power Telemetry Engine.
- **Phase 5**: Modern Dark Desktop & Web Application (Tauri v2 + React 19 + TypeScript + PostCSS + WebGL/Canvas Waveform Viewer).
- **Phase 6**: Verification, Conformance Testing vs Vivado, and Benchmarks.
