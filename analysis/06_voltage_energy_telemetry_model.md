# Betterado Voltage, Energy & Power Telemetry Engine

## 1. Overview and Problem Statement

Vivado's power estimation (`report_power`) is disconnected from simulation. It relies on dumping static files (SAIF) and reporting post-simulation averages. It cannot show:
- Real-time voltage rail sag.
- Dynamic current spikes during clock transitions.
- Instantaneous energy dissipation per sub-module.

Betterado embeds a **first-principles physics-based power telemetry model** directly inside the simulation kernel. As signals toggle during `tick(dt)` or `step_delta()`, energy dissipation is accumulated in real time and streamed to the UI at 60 frames per second.

---

## 2. Physics-Based Dynamic Power Formulation

### 2.1. Lumped Net Capacitance Model
Every net $i$ in the circuit is assigned a physical capacitance:
$$C_i = C_{\text{driver\_pin}} + \left(c_{\text{wire\_unit}} \cdot L_{\text{estimated}}\right) + \sum_{j \in \text{sinks}} C_{\text{sink\_pin}, j}$$

- $C_{\text{driver\_pin}}$: Output driver pin capacitance (default: $1.2 \text{ fF}$).
- $C_{\text{wire}}$: Interconnect capacitance estimated from fanout degree ($C_{\text{wire}} \approx \text{fanout} \times 0.8 \text{ fF}$).
- $C_{\text{sink\_pin}}$: Input gate capacitance of connected inputs (default: $0.6 \text{ fF}$).

### 2.2. Event Energy Dissipation
Whenever net $i$ undergoes a logic transition ($0 \to 1$ or $1 \to 0$):
The energy charged into or discharged from the electric field across capacitance $C_i$ is:
$$\Delta E_i = \frac{1}{2} \cdot C_i \cdot V_{\text{rail}}^2$$

For a 32-bit bus toggling from `0x00000000` to `0xFFFFFFFF`:
$$\Delta E_{\text{bus}} = \sum_{k=0}^{31} \frac{1}{2} \cdot C_k \cdot V_{\text{rail}}^2$$

### 2.3. Instantaneous Power & Current Calculation
Over any delta time interval $\Delta t = (t_2 - t_1)$:
$$P_{\text{dynamic}}(t) = \frac{\sum \Delta E}{\Delta t}$$
$$I_{\text{rail}}(t) = \frac{P_{\text{dynamic}}(t)}{V_{\text{rail}}}$$

---

## 3. Power Supply Rails & Voltage Modeling

Betterado allows users to define independent power supply rails (matching FPGA standards):
- **$V_{\text{core}}$ (Core Logic)**: $0.85\text{V} - 1.0\text{V}$ (powers internal flip-flops and logic gates).
- **$V_{\text{aux}}$ (Auxiliary / Clocking)**: $1.8\text{V}$ (powers clock buffers and PLLs).
- **$V_{\text{io}}$ (I/O Banks)**: Configurable per port ($1.2\text{V}, 1.8\text{V}, 2.5\text{V}, 3.3\text{V}$).

### Voltage Sag / Droop Modeling
If a large number of logic gates switch simultaneously on a clock edge (high $di/dt$), Betterado simulates power distribution network (PDN) impedance:
$$V_{\text{droop}}(t) = L_{\text{pdn}} \frac{di}{dt} + R_{\text{pdn}} \cdot i(t)$$
This allows engineers to detect whether high switching activity causes voltage droop that could trigger timing violations.

---

## 4. Lock-Free Telemetry Streaming Architecture

To prevent telemetry calculations from slowing down the high-speed JIT simulation kernel:

```
[Sim Kernel: step_delta()]
           | (Records toggle: NetId, Time)
           v
+-------------------------------------------------------------+
| Lock-Free Ring Buffer (SPSC Atomic RingBuffer)              |
| Memory footprint: 1 MB, zero heap allocations               |
+-------------------------------------------------------------+
           |
           v (Consumed by background Telemetry Worker thread)
+-------------------------------------------------------------+
| Telemetry Aggregator Thread                                 |
| - Computes dE, P(t), I(t) per rail                          |
| - Downsamples to 60 FPS display samples (16.6ms intervals)   |
| - Packages JSON / binary stream packets                     |
+-------------------------------------------------------------+
           |
           v (WebSocket / Tauri IPC / SharedArrayBuffer)
+-------------------------------------------------------------+
| Frontend Telemetry Visualizer (React + Canvas 2D)           |
| - Analog dynamic current wave                               |
| - Instantaneous power graph (mW)                            |
| - Thermal heatmap per module                                |
+-------------------------------------------------------------+
```

---

## 5. Export Formats

In addition to live streaming, `betterado-telemetry` exports:
- **SAIF (Switching Activity Interchange Format)**: 100% compatible with Vivado's `read_saif` command for cross-verification.
- **VCD (Value Change Dump)**: Standard IEEE 1364 format.
- **CSV / Parquet**: Raw numerical time-series of voltage, current, and energy for analysis in Python / Pandas / MATLAB.
