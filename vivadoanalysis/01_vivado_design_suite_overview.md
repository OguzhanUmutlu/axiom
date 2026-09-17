# Vivado Design Suite: Architectural Overview & HDL Subsystem Analysis

## 1. Executive Summary

The **AMD Vivado™ Design Suite** (formerly Xilinx Vivado) is an industry-standard Electronic Design Automation (EDA) platform for synthesizing, implementing, and simulating Hardware Description Language (HDL) designs targeted at AMD FPGAs, SoCs, and ACAPs. Introduced in 2012 as the replacement for the legacy ISE Design Suite, Vivado was architected around an in-memory data model, a unified Tcl scripting interface, and an updated simulation and synthesis pipeline.

Despite its industry dominance, Vivado's HDL processing pipeline suffers from severe architectural burdens:
- **Monolithic Installation Size**: Exceeds 60–110 GB on disk.
- **High Compilation Latency**: Relies on multi-stage file-based intermediate representations (`xvlog`/`xvhdl` $\to$ library database $\to$ `xelab` $\to$ snapshot binary $\to$ `xsim`).
- **Heavy UI Overhead**: Java/Swing-based desktop frontend with heavy memory consumption, slow rendering of large waveforms, and no web/cloud capability.
- **Opaque Delta Cycles**: Inability to visually single-step or inspect discrete delta cycles in zero simulation time.

Understanding Vivado's internal stages is vital for building **Betterado**—a modern, ultra-fast, cross-platform remake written in Rust that compiles HDL directly to machine code in RAM and provides delta-time tickable simulation and real-time telemetry.

---

## 2. Vivado Architecture Decomposition

Vivado's simulation architecture is composed of distinct modular sub-executables coordinated either via the Vivado GUI or headless Tcl scripts:

```
+-----------------------------------------------------------------------+
|                         Vivado Design Suite                           |
+-----------------------------------------------------------------------+
|  Frontend UI (Java / Swing) & Tcl Shell Engine (C / Tcl 8.5)          |
+-----------------------------------+-----------------------------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
            v                                               v
+-----------------------+                       +-----------------------+
|   Synthesis / Netlist |                       | Simulation Subsystem  |
|   (Vivado Synth,      |                       | (xsim Architecture)   |
|    RTL Elaboration)   |                       +-----------------------+
+-----------------------+                                   |
            |                                               v
            |                         +-----------------------------------+
            |                         | 1. Front-End Parsers:             |
            |                         |    xvlog (Verilog/SystemVerilog)  |
            |                         |    xvhdl (VHDL)                   |
            |                         +-----------------+-----------------+
            |                                           |
            |                                           v
            |                         +-----------------------------------+
            |                         | 2. Elaboration Linker:            |
            |                         |    xelab                          |
            |                         |    - Parameter propagation        |
            |                         |    - Hierarchy flattening         |
            |                         |    - Generates simulation snapshot|
            |                         +-----------------+-----------------+
            |                                           |
            |                                           v
            |                         +-----------------------------------+
            |                         | 3. Simulation Kernel:             |
            |                         |    xsim / xsimk (Shared object)   |
            |                         |    - Stratified event scheduler   |
            |                         |    - Delta cycle loop             |
            |                         |    - Waveform database (WDB/VCD)  |
            |                         +-----------------+-----------------+
            |                                           |
            +-----------------------+-------------------+
                                    |
                                    v
            +---------------------------------------------------+
            | 4. Telemetry & Verification:                      |
            |    - report_power (UG907 Power Engine)            |
            |    - SAIF (Switching Activity Interchange Format) |
            |    - Static & Dynamic Power Estimation            |
            +---------------------------------------------------+
```

---

## 3. The Three-Stage Simulation Pipeline

### 3.1. Front-End Parsing (`xvlog` & `xvhdl`)
- **Input**: Raw HDL source files (`.v`, `.sv`, `.vhd`, `.vhdl`).
- **Processing**:
  - Lexical analysis and context-free grammar parsing.
  - Macro expansion (` `define `, ` `ifdef `, ` `include `).
  - Validation of syntax against IEEE standards (IEEE 1364-2001/2005, IEEE 1800-2012/2017, IEEE 1076-1993/2002/2008).
- **Artifacts**: Intermediate parsed design units serialized into disk directories representing HDL logical libraries (e.g. `xsim.dir/work`, `xsim.dir/xil_defaultlib`).

### 3.2. Elaboration (`xelab`)
- **Input**: Compiled design units, top-level module/entity name, parameter overrides (`-generic_top`, `-defparam`).
- **Processing**:
  - Traverses the hierarchy recursively starting from the designated top module.
  - Evaluates compile-time parameters and `generate` statements (`generate if`, `generate for`).
  - Resolves cross-module signal connections, port widths, and array slicing.
  - Performs static semantic checks (e.g., multi-driver conflicts on non-tri nets, width mismatches).
  - Compiles the flattened netlist and behavioral code blocks into an executable simulation snapshot.
- **Output**: A simulation snapshot directory (e.g. `xsim.dir/snapshot/`) containing metadata and the compiled simulation kernel `xsimk`.

### 3.3. Execution Kernel (`xsim`)
- **Input**: Simulation snapshot.
- **Processing**:
  - Initializes state arrays in memory for all nets, registers, and memory blocks.
  - Executes `initial` blocks and evaluates continuous assignments at Time $t = 0$.
  - Drives an IEEE-compliant stratified event queue (Active, Inactive, NBA, Observed, Reactive, Postponed).
  - Advances time and executes delta cycles until design reaches equilibrium or maximum time is reached.
  - Records signal transitions to a binary waveform file (`.wdb`) or text dump (`.vcd`).

---

## 4. Power & Switching Activity Engine (`report_power`)
Vivado incorporates a power estimation engine documented in **UG907**. Power analysis relies on:
1. **Static / Leakage Power**: Baseline quiescent power determined by the target silicon process, junction temperature, and supply voltage rails ($V_{ccint}$, $V_{ccaux}$, $V_{cco}$).
2. **Dynamic Power**: Active power consumed by charging and discharging parasitic and load capacitances when signals toggle:
   $$P_{\text{dynamic}} = \sum_{i \in \text{nets}} \frac{1}{2} C_i \cdot V_{i}^2 \cdot f \cdot \alpha_i$$
   Where:
   - $C_i$ is net capacitance (pin capacitance + interconnect capacitance).
   - $V_i$ is the operational voltage rail for net $i$.
   - $f$ is the global reference clock frequency.
   - $\alpha_i$ is the switching activity / toggle rate (transitions per clock cycle).
3. **Switching Activity Interchange Format (SAIF)**: Vivado dumps SAIF files from simulation runs to capture exact toggle counts and static high probabilities ($T_1 / T_{\text{total}}$) for every net, which are back-annotated into `report_power`.

---

## 5. Motivation for Betterado

| Metric / Dimension | AMD Vivado (xsim/xelab) | Betterado (Target Design) |
| :--- | :--- | :--- |
| **Language & Safety** | Legacy C/C++, Java Swing | Modern 100% Memory-Safe Rust |
| **Install Footprint** | 60 GB – 110 GB | Single self-contained binary (< 50 MB) |
| **Compile & Startup Latency** | 5 – 45 seconds per iteration | Sub-second JIT compilation direct to RAM |
| **Execution Backend** | Interpreted / C-code compiled snapshot | Cranelift JIT machine code (x86_64/ARM64) + WASM |
| **Delta Cycle Visibility** | Black-box zero-time progression | First-class manual delta-cycle stepping & visualizer |
| **Interactive API** | Fragile Tcl stdin/stdout pipes | Direct Rust / C-ABI / WASM API with tick(dt) |
| **Telemetry & Power** | Batch post-simulation SAIF analysis | Real-time live streaming voltage & energy graphing |
| **Platform Support** | RHEL/Ubuntu/Windows only (x86_64) | Linux, macOS (Apple Silicon & Intel), Windows, Web |
| **Frontend Architecture** | Heavy Java/Swing | Tauri v2 + React 19 + TypeScript + PostCSS |
