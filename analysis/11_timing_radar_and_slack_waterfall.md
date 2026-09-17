# Axiom EDA: Timing Radar, Slack Waterfall & CDC Matrix Architecture

## 1. Overview & Vision

Static Timing Analysis (STA) in AMD Vivado is notoriously difficult to navigate. Timing violations are presented as massive, text-heavy tables (`report_timing_summary`) with cryptic acronyms (WNS, TNS, WHS, THS). Correlating a violating path with physical logic gates and clock domain crossings requires switching between multiple modal windows and cross-referencing log files.

**Axiom EDA reimagines STA with the Timing Radar and Slack Waterfall**: an interactive, visual timing intelligence suite providing real-time setup/hold margin visualization, critical path delay breakdowns, and automated Clock Domain Crossing (CDC) synchronizer verification.

---

## 2. Slack Radar & Timing Metrics

Axiom models both Setup (Maximum Delay) and Hold (Minimum Delay) timing constraints:

$$\text{Setup Slack} = T_{\text{required}} - T_{\text{arrival}} = (T_{\text{clk2}} + T_{\text{dest\_skew}} - T_{\text{setup}}) - (T_{\text{clk1}} + T_{\text{source\_skew}} + T_{\text{clk-q}} + T_{\text{datapath}})$$

$$\text{Hold Slack} = T_{\text{arrival}} - T_{\text{required}} = (T_{\text{clk1}} + T_{\text{source\_skew}} + T_{\text{clk-q}} + T_{\text{datapath}}) - (T_{\text{clk2}} + T_{\text{dest\_skew}} + T_{\text{hold}})$$

```
                                  TIMING RADAR
         +-------------------------------------------------------------+
         | Setup Margin: +1.42 ns (PASS)  | Hold Margin: +0.28 ns (PASS)|
         | Target Clock: clk_50m (20.0 ns)| Max Frequency: 68.4 MHz     |
         +-------------------------------------------------------------+
                         Interactive Slack Histogram
         Counts
            |               ■■■■■
            |             ■■■■■■■■■
            |         ■■■■■■■■■■■■■■■
            |       ■■■■■■■■■■■■■■■■■■■
            +-------+-------+-------+-------+-----> Slack (ns)
                   -0.5     0.0    +0.5   +1.0
                  [VIOLATION]    [MET SPEC]
```

---

## 3. Interactive Critical Path Waterfall Visualizer

Instead of static ASCII tables, violating and near-critical paths are rendered as an interactive, proportional segment waterfall:

```
[ Launch Clk ]  |== Clock Tree Skew: 0.35ns ==|
[ Flip-Flop  ]                                |== Tco: 0.42ns ==|
[ Logic Gate ]                                                  |=== LUT 1 (4-input MUX): 0.65ns ===|
[ Wire Net   ]                                                                                      |=== Wire RC: 0.88ns ===|
[ Logic Gate ]                                                                                                              |=== LUT 2 (Adder): 0.72ns ===|
[ Capture FF ]                                                                                                                                            | Setup Window: 0.30ns |
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------->
Timeline (ns)  0.0                                          1.0                                         2.0                                         3.0                 4.0ns
```

### 3.1. Interactive Exploration Features:
- **Cell vs. Interconnect Breakdown**: Proportional visual comparison showing whether a violation is caused by excessive logic depth (too many LUTs) or wire congestion (long routing delay).
- **1-Click Buffer Insertion Recommendation**: Automated suggestion of pipeline register insertion points to balance path latency.
- **Cross-Probing**: Clicking any segment in the waterfall highlights the physical cell in the Schematic DAG and the exact statement in the HDL source code.

---

## 4. Clock Domain Crossing (CDC) Verification Matrix

Unsynchronized clock domain crossings cause metastability and intermittent silicon failures. Axiom scans the elaborated netlist to automatically construct a CDC Matrix:

| Source Clock Domain | Destination Clock Domain | Status | Required Protection | Detected Synchronizer |
| :--- | :--- | :--- | :--- | :--- |
| `clk_100m` | `clk_100m` | Synchronous | None (Single Domain) | Direct Net |
| `clk_100m` | `clk_fast_200m`| Related (2:1) | Multi-cycle constraint | Phase-Aligned BUFG |
| `clk_100m` | `clk_uart_115k`| **Asynchronous** | **2-FF Synchronizer** | **2-Stage DFF Detected (PASS)** |
| `clk_uart_115k` | `clk_100m` | **Asynchronous** | **2-FF Synchronizer** | ⚠️ **UNSYNCHRONIZED (CRITICAL)** |

Violations are flagged with high-visibility amber warning badges directly on the schematic and waveform timeline.
