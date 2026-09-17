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
