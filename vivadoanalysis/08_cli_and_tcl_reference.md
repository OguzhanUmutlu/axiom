# Vivado CLI and Tcl Command Reference Manual

## 1. Scope

This document provides a comprehensive command-line and Tcl reference for the key executables and commands that make up Vivado's HDL simulation and power analysis flow (`xvlog`, `xvhdl`, `xelab`, `xsim`, `report_power`). This serves as a functional reference specification for Betterado's CLI interface and automated compatibility layer.

---

## 2. Front-End Parsing Tools

### 2.1. `xvlog` (Verilog / SystemVerilog Compiler)
```bash
xvlog [options] <source_files...>
```
#### Key Flags:
- `-sv`: Enable SystemVerilog syntax parsing mode.
- `-work <lib_name>`: Destination library for compiled units (default: `xil_defaultlib`).
- `-i <dir>` / `-include <dir>`: Include directory path for `` `include `` files.
- `-d <macro>[=<val>]` / `-define <macro>[=<val>]`: Define preprocessor macros.
- `-relax`: Relax strict IEEE rules during port/wire checks.
- `-f <file>`: Read source files and arguments from a command file list.
- `-nolog`: Do not create `xvlog.log`.

### 2.2. `xvhdl` (VHDL Compiler)
```bash
xvhdl [options] <source_files...>
```
#### Key Flags:
- `-2008`: Enable VHDL-2008 standard features.
- `-work <lib_name>`: Destination logical library.
- `-relax`: Relax strict type rules during analysis.
- `-f <file>`: File list input.

---

## 3. Elaboration Linker: `xelab`

```bash
xelab [options] <top_level_module>
```
#### Key Flags:
- `-s <snapshot_name>` / `-snapshot <snapshot_name>`: Name of the generated simulation snapshot (default: `default`).
- `-top <module>`: Top-level module to elaborate.
- `-generic_top <name>=<val>`: Override top-level parameter/generic value.
- `-defparam <hierarchical_path>=<val>`: Override specific hierarchical parameter.
- `-timescale <unit>/<precision>`: Set default timescale (e.g. `1ns/1ps`).
- `-debug <typical|all|off>`: Instrument design with symbol tables for waveform dumping.
- `-O0` / `-O1` / `-O2` / `-O3`: Code generation optimization level.
- `-mt <auto|1..8>`: Number of parallel elaboration threads.
- `-maxdeltaid <count>`: Maximum number of delta cycles permitted within a single time step (default: 10,000).

---

## 4. Simulation Execution: `xsim`

```bash
xsim <snapshot_name> [options]
```
#### Key Flags:
- `-R` / `-runall`: Run simulation to completion (`$finish`) without opening interactive shell.
- `-tclbatch <tcl_file>`: Execute Tcl script commands non-interactively.
- `-gui`: Launch the Java-based Vivado Waveform GUI.
- `-wdb <file.wdb>`: Destination waveform database file.
- `-view <file.wcfg>`: Open saved waveform configuration view.
- `-maxdeltaid <count>`: Overrides max delta cycle threshold.

### 4.1. Core Interactive Tcl Simulation Commands
Within the `xsim%` prompt:
- `run <time>`: Advances simulation by physical time (e.g. `run 500ns`, `run 2us`).
- `run all`: Advances simulation until `$finish` or infinite delta loop.
- `step`: Advances one behavioral statement.
- `restart`: Resets simulation state and registers back to Time 0.
- `get_value <signal_path>`: Prints the current 4-state logic value of a signal.
- `add_force <signal_path> <value> [-radix <radix>]`: Overrides signal value with user stimulus.
- `remove_forces <signal_path>`: Clears forced override.
- `log_wave -r <scope>`: Begins logging transitions for signals under scope.
- `open_vcd <vcd_file>`: Opens a VCD stream file.
- `log_vcd [get_objects <path>]`: Selects signals for VCD dumping.
- `close_vcd`: Flushes and closes the VCD stream file.

---

## 5. Power & Switching Activity Commands

### 5.1. SAIF Generation in `xsim`:
```tcl
# Open SAIF output file
open_saif "activity.saif"

# Log switching activity across entire design hierarchy
log_saif [get_objects -r /tb_top/u_dut/*]

# Run simulation test vectors
run 1ms

# Flush and close SAIF database
close_saif
```

### 5.2. Power Estimation in Vivado (`report_power`):
```tcl
# Load implemented or synthesized netlist
open_run impl_1

# Back-annotate switching activity from simulation SAIF
read_saif "activity.saif"

# Execute power estimation
report_power -file "power_breakdown.rpt" -xpe "power_export.xpe"
```

---

## 6. Mapping Vivado Commands to Betterado Architecture

| Vivado Command / Tool | Betterado Primitive / CLI Equivalent | Betterado Architecture Advantage |
| :--- | :--- | :--- |
| `xvlog`, `xvhdl` | `betterado compile <files>` | In-memory parallel parsing, zero disk serialization |
| `xelab -top <top>` | `betterado elaborate -top <top>` | Direct BIR graph lowering, sub-second latency |
| `xsim -R` | `betterado run` | In-RAM Cranelift JIT machine code execution |
| `xsim (step)` | `sim.step_delta()` / `sim.tick(dt)` | True delta-cycle micro-stepping, glitch inspection |
| `open_saif` / `report_power` | `sim.telemetry_stream()` | Real-time 60 FPS voltage, current, and power telemetry |
| `xsim -gui` | `betterado gui` | Ultra-fast native Tauri v2 + React 19 dark UI |
