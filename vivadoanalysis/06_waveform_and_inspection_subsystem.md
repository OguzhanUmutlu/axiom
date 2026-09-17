# Vivado Waveform & Signal Inspection Subsystem

## 1. Overview

Waveform analysis is the primary debug interface for digital hardware engineers. When running logic simulation in Vivado, the simulation kernel (`xsim`) captures digital signal transitions across time and records them into waveform storage databases for visualization in the Vivado Waveform Viewer.

Vivado supports two primary waveform formats:
1. **WDB (Waveform Database)**: AMD's proprietary, high-speed indexed binary database format (`.wdb`).
2. **VCD (Value Change Dump)**: The IEEE 1364 standard ASCII text-based dump file format (`.vcd`).

---

## 2. Vivado Waveform Architecture

```
                 +-----------------------------------+
                 |           xsim Kernel             |
                 +-----------------+-----------------+
                                   |
                   +---------------+---------------+
                   |                               |
                   v                               v
    +------------------------------+   +------------------------------+
    | WDB Engine (Binary Indexed)  |   | VCD Engine (ASCII Stream)    |
    | - Logs to xsim.dir/<snap>.wdb|   | - Logs to dump.vcd           |
    | - Direct Vivado GUI integration| | - Portable to GTKWave/Surfer |
    | - Signal hierarchy & types   |   | - Large file size overhead   |
    +--------------+---------------+   +------------------------------+
                   |
                   v
    +-----------------------------------------------------------------+
    | Vivado Waveform Viewer (Java / Swing UI Component)              |
    | - Signal Pane (Hierarchical net names, radix: Hex, Bin, Dec)   |
    | - Waveform Canvas (Digital square waves, analog bus plots)      |
    | - Cursors & Markers (Time deltas, frequency measurement)        |
    +-----------------------------------------------------------------+
```

---

## 3. Waveform Tracing Commands & Operations

In Vivado `xsim`, wave logging is controlled via Tcl commands:

### 3.1. Logging to WDB:
```tcl
# Add top-level signals and all sub-module signals recursively
log_wave -r /tb_top/*

# Add specific bus with custom display radix
add_wave -radix hex /tb_top/u_cpu/data_bus

# Run simulation
run 10us
```

### 3.2. Logging to Standard VCD:
```tcl
open_vcd "simulation_dump.vcd"
log_vcd -level 0 /tb_top/*
run 10us
close_vcd
```

---

## 4. Signal Radix and Bus Formatting

Vivado's waveform viewer allows multiple representations for multi-bit buses:
- **Binary**: Raw bit sequence `8'b10100111`.
- **Hexadecimal**: Compact byte representation `8'hA7`.
- **Unsigned Decimal**: Positive magnitude `167`.
- **Signed Decimal**: Two's-complement integer `-89`.
- **ASCII**: Decoded characters for string buses.
- **Analog Waveform**: Interprets binary integers as continuous analog voltages or currents, rendering continuous curves. Useful for DSP filters, DAC/ADC models, and PWM modulators.

---

## 5. Architectural Flaws in Vivado's Inspection Engine

Despite basic functionality, Vivado's waveform inspection suffers from critical limitations:

1. **Delta Cycles are Collapsed and Hidden**:
   - In physical digital circuits, combinational logic gates experience transient glitches (e.g. static-1 hazard) within a zero-time interval before settling.
   - Vivado's `.wdb` viewer only stores the final settled value at the end of each simulation time step. All intermediary delta cycles are completely lost.
   - If an engineer has a race condition or a combinational hazard that violates an asynchronous reset or latch, Vivado's waveform viewer cannot display the glitch unless explicit non-zero `#1` delays are manually injected into the RTL code.

2. **Heavy File Footprint & Disk Choke**:
   - Multi-millisecond simulations generate massive multi-gigabyte `.wdb` files.
   - Loading or scrubbing through a 10 GB `.wdb` file freezes the Vivado Java GUI for minutes, exhausting system RAM.

3. **Inability to Inject Interactive Stimulus on the Fly**:
   - In Vivado, altering input signals requires halting the simulation, issuing `add_force` commands, and re-running. You cannot drag or toggle pins interactively on a virtual schematic or front-panel UI.

4. **Zero Live Synchronized Telemetry**:
   - Waveforms display only logical values (`0`, `1`, `X`, `Z`). They do not show dynamic current spikes, power dissipation per net, or instantaneous supply rail noise alongside the digital traces.

---

## 6. How Axiom Redefines Hardware Inspection

Axiom fundamentally redesigns the inspection paradigm:
1. **Delta-Cycle Scrubbing**: The simulation engine records delta micro-steps. The user can expand any timestamp to view every delta transition ($\delta_0, \delta_1, \delta_2...$).
2. **Streaming Ring Buffer in RAM**: Real-time signal updates are cached in memory-mapped ring buffers and streamed directly to a 60+ FPS Canvas/WebGL renderer in the frontend.
3. **Co-located Voltage & Energy Traces**: Every digital net can display an synchronized analog current/power graph beneath it.
4. **Interactive Pin Forcing**: Signals can be toggled interactively with immediate re-evaluation of downstream logic.
