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
| `clk_uart_115k` | `clk_100m` | **Asynchronous** | **2-FF Synchronizer** | ️ **UNSYNCHRONIZED (CRITICAL)** |

Violations are flagged with high-visibility amber warning badges directly on the schematic and waveform timeline.

---

## 5. Production Implementation & Studio Architecture

Axiom EDA implements this architecture natively in TypeScript, SVG, and React 19:

- **Static Timing Analysis Model (`ui/src/engine/timingModel.ts`)**:
  - Full path delay accumulation ($\sum t_{\text{cell}} + \sum t_{\text{net}} + t_{\text{setup}}$).
  - Setup Slack Calculation: $T_{\text{slack}} = T_{\text{clk}} - T_{\text{arrival}}$.
  - Hold Slack Calculation ($T_{\text{hold}} = 280\text{ ps}$ safe margin).
  - Maximum Achievable Frequency: $f_{\text{max}} = 1 / (T_{\text{clk}} - \text{WNS})$.
  - Slack Distribution Histogram: 5 proportional bins classifying path margins from negative violations ($< 0\text{ ps}$) up to healthy margins ($> +500\text{ ps}$).
- **Timing Radar & Waterfall Viewer (`ui/src/components/TimingRadarViewer.tsx`)**:
  - **Constraint Wizard Toolbar**: Fast toggle presets for 50 MHz (20 ns), 100 MHz (10 ns), 200 MHz (5 ns), and 300 MHz (3.33 ns).
  - **Slack Radar Metric Cards**: Real-time display of WNS, TNS, WHS, and Fmax with status badges (`MET` / `VIOLATION`).
  - **Interactive Critical Path Waterfall**: Color-coded proportional timeline bars:
    - Launch Clock tree skew (Cyan)
    - Source Flip-Flop $T_{\text{co}}$ (Indigo)
    - Combinational Logic Gates (Emerald)
    - Interconnect Wire Routing (Amber)
    - Capture Flip-Flop Setup Window (Purple)
  - **Logic Gate vs Wire Breakdown**: Proportional calculation (e.g. 63% Cell Delay vs 37% Routing Delay) with automated pipeline advice.
  - **Automated CDC Verification Matrix Table**: Source/Dest domain mapping, frequency ratios, protection schemes, and verified status pills.
- **Studio Integration (`ui/src/App.tsx`)**:
  - Dedicated `[ ⏱️ Timing & Energy ]` studio view tab for in-depth timing exploration alongside HDL source code.

