# Betterado Architecture Analysis: Master Index & Tree Structure

## 1. Directory Tree Structure

The `analysis/` folder contains the comprehensive architectural blueprint, technical specifications, and system design documents for **Betterado** (the actual project). 

```
analysis/
├── analysis.md                             # [Master Index] Tree structure and document directory
├── 01_engine_architecture.md               # Rust workspace layout, memory model & performance principles
├── 02_lexer_parser_ast.md                  # HDL lexing, preprocessor, resilient parsing & AST model
├── 03_elaboration_and_netlist_ir.md        # Elaboration pipeline, parameter propagation & BIR (IR)
├── 04_jit_machine_code_compiler.md         # Cranelift JIT compilation in RAM & WebAssembly target
├── 05_event_scheduler_and_delta_stepping.md# Stratified event queue, delta cycles & manual tick API
├── 06_voltage_energy_telemetry_model.md    # Physical power model, capacitance & real-time telemetry
├── 07_api_and_runtime_interface.md         # Embeddable Rust API, C-ABI, WASM bindings & inspection hooks
└── 08_desktop_and_web_ui.md                # Tauri v2, React 19, PostCSS, Vite & WebGL waveform viewer
```

---

## 2. Document Directory & Functional Responsibilities

| File Name | Primary Topic | Detailed Responsibilities & Scope |
| :--- | :--- | :--- |
| **`01_engine_architecture.md`** | **Core Engine Architecture** | • Multi-crate Cargo workspace organization.<br>• Zero-allocation memory models and arena allocation.<br>• Diagnostic reporting engine (`miette`/`codespan`).<br>• Thread safety and multi-threaded design. |
| **`02_lexer_parser_ast.md`** | **Front-End HDL Parser** | • Streaming zero-copy lexer with UTF-8 byte span tracking.<br>• Preprocessor directives (` `define `, ` `ifdef `, ` `include `).<br>• AST node definitions for Verilog & SystemVerilog.<br>• Error recovery and resilient parsing. |
| **`03_elaboration_and_netlist_ir.md`** | **Elaboration & Netlist IR** | • Recursive instantiation and hierarchical scope resolution.<br>• Compile-time parameter evaluation and `generate` expansion.<br>• Port binding and multi-driver resolution.<br>• Specification of Betterado Intermediate Representation (BIR). |
| **`04_jit_machine_code_compiler.md`** | **In-RAM JIT Compilation** | • Cranelift integration for native x86_64 & AArch64 machine code.<br>• 4-state logic (0, 1, X, Z) 2-bit dual-vector bit twiddling.<br>• Memory layout of design state in executable RAM.<br>• Direct WebAssembly (WASM) codegen for web browsers. |
| **`05_event_scheduler_and_delta_stepping.md`** | **Event Scheduler & Tick API** | • Stratified IEEE 1800 event queue implementation.<br>• Delta cycle resolution loop and glitch tracking.<br>• Manual `tick(delta_time)` and `step_delta()` APIs.<br>• Breakpoints, triggers, and state checkpoints. |
| **`06_voltage_energy_telemetry_model.md`** | **Power & Telemetry Engine** | • Physics-informed dynamic power ($P = \frac{1}{2} C V^2 f \alpha$).<br>• Instantaneous energy dissipation and current surge modeling.<br>• Lock-free ring buffer streaming for 60 FPS telemetry.<br>• SAIF and VCD export capabilities. |
| **`07_api_and_runtime_interface.md`** | **API & Interoperability** | • Public Rust API for external embedding.<br>• C-ABI dynamic library (`.so`, `.dylib`, `.dll`).<br>• `wasm-bindgen` interface for browser JavaScript execution.<br>• Signal probing, forcing, and event subscription hooks. |
| **`08_desktop_and_web_ui.md`** | **Desktop & Web Application** | • Tauri v2 desktop shell and cross-platform native bridge.<br>• React 19 + TypeScript + PostCSS dark-themed user interface.<br>• 60+ FPS Canvas 2D/WebGL virtualized digital waveform viewer.<br>• Synchronized power/voltage graphs and delta-step zoom view.<br>• Vite configuration for dual-target desktop and web builds. |
