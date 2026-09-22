# Axiom EDA: Virtual Interactive Lab & Waveform Stimulus Rack

## 1. Overview & Vision

In traditional FPGA development workflows, verifying hardware requires either:
1. Writing hundreds of lines of tedious, non-synthesizable testbench HDL code with artificial clock toggles (`always #5 clk = ~clk;`) and manual stimulus statements.
2. Synthesizing the design to real FPGA hardware and using AMD Vivado's Virtual Input/Output (VIO) core over a physical JTAG cable to toggle virtual switches and LEDs.

Both approaches impose massive latency. **Axiom EDA introduces the Virtual Interactive Lab**: a pure software, in-RAM front-panel instrument rack that connects directly to the running Cranelift JIT simulation kernel. Engineers can interactively flip switches, press buttons, observe 7-segment displays, and paint waveform stimulus with zero testbench boilerplate and zero hardware JTAG dependencies.

---

## 2. Virtual Instrument Component Palette

The Virtual Lab provides a modular rack of interactive hardware instruments:

```
+-----------------------------------------------------------------------------------------+
| AXIOM VIRTUAL INSTRUMENT RACK                                                           |
+---------------------+-------------------+---------------------+-------------------------+
| 8-Bit DIP Switch    | Momentary Buttons | Dual 7-Segment      | Waveform Stimulus Pad   |
| [o] [x] [o] [o] ... | [ RESET ] [ TICK ]|      [ 8. ] [ E. ]  | ┌─┐ ┌─┐ ┌─┐  clk (50MHz)|
| Pin: `sw[7:0]`      | Pin: `rst_n`, `btn`| Pin: `hex_disp`     | └───┘ └───┘  data_in    |
+---------------------+-------------------+---------------------+-------------------------+
| Virtual UART        | I2C/SPI Protocol  | Virtual Logic State | Rotary Hex Encoder      |
| Terminal (115200)   | Packet Sniffer    | Forcing Matrix      |      ( 0x3F )           |
| "CPU BOOT OK"       | [START][0x42][ACK]| Force: `en <= 1`    | Dial: `mode_sel[3:0]`   |
+---------------------+-------------------+---------------------+-------------------------+
```

### 2.1. Built-in Instrument Primitives:
1. **DIP Switches & Rocker Switches**: 1-bit or $N$-bit bus switches with binary and hex labels, directly driving top-level input ports.
2. **Tactile Pushbuttons**: Momentary active-low or active-high buttons with configurable debouncing filters.
3. **Multi-Digit 7-Segment Displays**: Decodes common-anode or common-cathode multiplexed segment buses (`a..g, dp`) into human-readable hex/decimal characters in real time.
4. **Virtual UART Terminal**: Connects to `tx` and `rx` serial lines, converting 8N1 bitstreams into an interactive ANSI text console.
5. **Rotary Quadrature Encoder**: Simulates 2-phase $A/B$ pulse streams with directional detents.

---

## 3. Real-Time In-RAM Stimulus Injection Architecture

When a virtual instrument is manipulated in the GUI (e.g. flipping `sw[3]` from `0` to `1`):

```
[ User Interaction: Switch Click ]
                | (WebSocket / IPC Event: { net_id: 14, new_val: Logic4::One })
                v
[ Axiom Sim Bridge (engineBridge.ts) ]
                |
                v
[ Cranelift In-RAM Kernel: sim.force_net(net_id, val) ]
                |
                +---> Overwrites SimStateArena memory word (*mut u64)
                +---> Enqueues Downstream Dependent Processes in Active Region
                +---> Re-evaluates BIR continuous assignments in <5 microseconds
                +---> Emits TelemetryFrame & Waveform Transition Event
                |
                v
[ Waveform & 7-Segment Instant Refresh (60 FPS) ]
```

Because Axiom compiles directly to native machine code in RAM without intermediate disk snapshots, stimulus propagation occurs in **single-digit microseconds**, enabling true real-time tactile exploration of digital systems.

---

## 4. Visual Waveform Stimulus Painter

For automated multi-cycle testing without writing HDL testbenches:
1. **Clock Generator**: Specify frequency (e.g. 50 MHz), duty cycle (50%), and phase jitter directly in the UI.
2. **Mouse Stimulus Painting**: Click and drag on the waveform canvas to draw high pulses, low pulses, or random data vectors.
3. **Pattern Sequencer**: Repeat sequences (e.g. Count up, Gray code, PRBS7 pseudo-random) with 1 click.
4. **Testbench Export**: Auto-generate synthesizable IEEE 1800 SystemVerilog testbenches (`tb_generated.sv`) from the recorded stimulus for archival and CI/CD conformance.

---

## 5. Production Implementation & Studio Architecture

Axiom EDA implements this architecture natively in TypeScript, SVG, and React 19:

- **Virtual Instrument Rack (`ui/src/components/VirtualLabRack.tsx`)**:
  - **8-Bit DIP Switch Bank**: Dynamic bit manipulation (`[0]`..`[7]`) driving top-level ports (`a[7:0]`, `b[7:0]`, `opcode[2:0]`, `enable`, `up_down`, `data_in[7:0]`) with live Hex, Binary, and Decimal readouts.
  - **Tactile Pushbuttons**: Active-low `RESET (rst_n)` strobe and `STEP CLOCK (clk)` single-cycle pulsing.
  - **Rotary Quadrature Hex Dial**: Smooth knurled dial sweeping values from `0x00` to `0xFF` with directional stepper buttons (`-16`, `-1`, `+1`, `+16`).
  - **Dual 7-Segment LED Displays**: Authentic SVG 7-segment display decoding registers (`result[7:0]`, `count[7:0]`, `accum_out[15:0]`) with red glowing drop shadows (`#ef4444`).
  - **16-Bit SMD LED Bar Graph**: Discrete surface-mount LED pips displaying bitwise logic states in real time.
- **Waveform Stimulus Painter Modal (`ui/src/components/StimulusPainterModal.tsx`)**:
  - Preset test pattern generator: Linear Ramp, Alternating 0xAA/55, Walking Ones, and PRBS7 pseudo-random.
  - In-RAM multi-cycle execution engine running test sequences without JTAG hardware.
  - 1-Click IEEE 1800-2017 SystemVerilog Testbench Exporter (`${topModule}_tb.sv`) with copy-to-clipboard and file download.
- **In-RAM Stimulus Injection API (`ui/src/engine/engineBridge.ts`)**:
  - `injectStimulus(signalId, value)`: Modifies state memory word and re-evaluates combinational and sequential processes in single-digit microseconds.
  - `pulseSignal(signalId)`: Generates cycle pulses advancing simulation time.
  - `generateSystemVerilogTestbench(topModule)`: Emits clean, synthesizable SystemVerilog testbench.
- **Integrated Studio Workspace (`ui/src/App.tsx`)**:
  - Studio view tabs: `[  Waveforms ]` | `[  Schematic DAG ]` | `[ ️ Virtual Lab ]` | `[ ◫ Split Studio ]`.

