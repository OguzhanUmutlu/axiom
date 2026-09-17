# Axiom EDA: Hierarchical Dynamic Energy Treemap & Thermal Architecture

## 1. Overview & Vision

In Vivado, power analysis (`report_power` / UG907) is an isolated, static post-mortem process that outputs dry text reports or spreadsheet exports. Hardware engineers cannot see *which* submodules are responsible for power spikes in real time as simulation proceeds.

**Axiom EDA introduces the Hierarchical Dynamic Energy Treemap and Live PDN Analyzer**: a real-time, physics-informed visual instrument that decomposes dissipated silicon energy ($E = \frac{1}{2} \sum C V^2$) and supply rail voltage sag directly as the simulation runs.

---

## 2. Dynamic Energy Treemap Representation

The energy dashboard visualizes total accumulated energy ($\mu\text{J}$) as a 2D squarified treemap where:
- **Rectangle Area**: Proportional to total dynamic energy consumed by that module or bus ($E_m / E_{\text{total}}$).
- **Color Hue / Intensity**: Proportional to instantaneous switching activity rate ($\alpha$, toggles per nanosecond), graded from Cool Slate (quiescent) to Blazing Orange / Neon Red (heavy switching).

```
+-------------------------------------------------------------------------+
| HIERARCHICAL SILICON ENERGY TREEMAP                                     |
+---------------------------------------------+---------------------------+
| Module: `top.u_core` (74.2% of Total Energy)| Module: `top.u_interconnect|
|                                             | (18.1% of Total Energy)   |
| +---------------------+-------------------+ |                           |
| | Sub: `u_alu_32bit`  | Sub: `u_regfile`  | | +-----------------------+ |
| | (42.8% Energy)      | (21.4% Energy)    | | | `axi_crossbar`        | |
| | Hot: 32b Multiplier | 32 x 32b FlipFlops| | | (12.4% Energy)        | |
| +---------------------+-------------------+ | +-----------------------+ |
| | Sub: `u_branch_pred` (10.0% Energy)     | | `u_dma_engine` (5.7%)   | |
+---------------------------------------------+---------------------------+
| Module: `top.u_uart` (4.5%)  | Module: `top.u_spi` (3.2%)               |
+-------------------------------------------------------------------------+
```

When simulation transitions from an idle loop into an intensive matrix multiply or cryptographic hash, the treemap tiles dynamically expand and shift color in real time at 60 FPS, providing immediate visual intuition into algorithmic energy efficiency.

---

## 3. Simultaneous Switching Noise (SSN) & PDN Droop Analyzer

High-speed parallel buses (e.g. 64-bit memory buses or 32-bit ALUs) toggling simultaneously draw high instantaneous current spikes ($\frac{di}{dt}$), causing voltage drop across power distribution network (PDN) package parasitics:

$$V_{\text{droop}}(t) = I(t) \cdot R_{\text{package}} + L_{\text{package}} \cdot \frac{dI}{dt}$$

Axiom models the multi-rail power distribution network in real time:
- **Core Voltage Rail ($V_{\text{core}} = 0.90\text{V}$)**: Monitors internal logic cell switching.
- **Auxiliary Voltage Rail ($V_{\text{aux}} = 1.80\text{V}$)**: Monitors clock buffers and PLLs.
- **I/O Rail ($V_{\text{io}} = 3.30\text{V}$)**: Monitors high-drive I/O pads.

### 3.1. Automated Voltage Droop Hazard Flags
If a simultaneous bus transition causes rail voltage to sag below the specified noise margin (e.g. $V_{\text{core}} < 0.85\text{V}$, $>5\%$ droop), Axiom automatically flags an **SSN Hazard Warning** in the telemetry panel and places a persistent hazard marker on the waveform timeline.
