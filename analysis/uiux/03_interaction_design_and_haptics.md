# Axiom Interaction Design, State Machines & Micro-Feedback

## 1. Perceptual Latency Budgets in Hardware Engineering

In hardware simulation and Electronic Design Automation (EDA), the perception of speed is not merely aesthetic—it fundamentally alters how engineers think and debug. 

When tool operations take 30 to 60 seconds (as in AMD Vivado's `xelab` file-based pipeline), engineers adopt batch-oriented, cautious workflows. When operations occur in milliseconds, engineers engage in **continuous exploratory design**, testing hypotheses, toggling input vectors, and inspecting timing consequences in real-time.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PERCEPTUAL LATENCY THRESHOLDS                         │
├───────────────────┬─────────────────────────────────────────────────────────┤
│ Threshold         │ Human Perception & Engineering Tool Requirement        │
├───────────────────┼─────────────────────────────────────────────────────────┤
│ 0 ms – 16 ms      │ Immediate / Continuous. Frame budget for 60 FPS canvas  │
│ (1 Frame @ 60Hz)  │ rendering, waveform zooming, cursor scrubbing, and      │
│                   │ schematic panning. Zero dropped frames permitted.       │
├───────────────────┼─────────────────────────────────────────────────────────┤
│ < 100 ms          │ Instantaneous Response. Limit for tactile button press, │
│                   │ DIP switch toggle, tab switching, and keyboard shortcut.│
│                   │ The user perceives the system as directly responding.   │
├───────────────────┼─────────────────────────────────────────────────────────┤
│ < 300 ms          │ Flow State Preserved. Upper limit for In-RAM Cranelift  │
│                   │ JIT compilation, AST elaboration, and linter squiggles. │
│                   │ User does not lose their train of thought.              │
├───────────────────┼─────────────────────────────────────────────────────────┤
│ > 1000 ms         │ Interruption of Flow. Must render an intermediate       │
│ (1 Second)        │ progress indicator, elapsed time counter, and an active │
│                   │ "Cancel" action to prevent user frustration.            │
└───────────────────┴─────────────────────────────────────────────────────────┘
```

---

## 2. Deterministic Simulation UI State Machine

The interface must faithfully mirror the simulation kernel's internal lifecycle without race conditions or conflicting controls. Axiom models simulation execution as a strict **Finite State Machine (FSM)**:

```mermaid
stateDiagram-v2
    [*] --> Idle: No Project Open
    Idle --> Compiling: Open Template / Project
    Compiling --> Ready: Compilation Success
    Compiling --> Errored: Syntax / Elaborate Failure
    
    Ready --> Running: Click Run / F5
    Running --> Paused: Click Pause / Break
    Paused --> Running: Click Resume
    
    Ready --> Stepping: Click +1ns / +100ps
    Paused --> Stepping: Click +1ns / +100ps
    Stepping --> Paused: Step Complete
    
    Ready --> DeltaStepping: Click Step δ / F11
    Paused --> DeltaStepping: Click Step δ / F11
    DeltaStepping --> Paused: δ-Cycle Settled
    
    Running --> Ready: Click Reset
    Paused --> Ready: Click Reset
    Errored --> Compiling: Edit Code & Re-Compile
```

### UI Control Matrix by State

```
State        Run Button     Step Buttons    Reset Button   Editor Editing   Force Signal
──────────────────────────────────────────────────────────────────────────────────────────
Idle         Disabled       Disabled        Disabled       Disabled         Disabled
Compiling    Disabled       Disabled        Disabled       Read-Only        Disabled
Ready        Play (Green)   Enabled         Enabled        Editable         Enabled
Running      Pause (Rose)   Disabled        Enabled        Editable (Deb)   Disabled
Paused       Play (Green)   Enabled         Enabled        Editable         Enabled
Stepping     Disabled       Disabled        Disabled       Editable         Disabled
Errored      Disabled       Disabled        Disabled       Editable         Disabled
──────────────────────────────────────────────────────────────────────────────────────────
```

> [!IMPORTANT]
> **Defensive Button Feedback:**
> When a button is disabled (e.g., `Step δ` when no project is loaded or compilation failed), clicking or hovering must display an informative tooltip explaining *why* the action is unavailable and *how* to unlock it (e.g., `"Design must compile without syntax errors before stepping simulation"`), rather than silently ignoring clicks.

---

## 3. Direct Manipulation & Tactile Micro-Interactions

### 3.1. Virtual Breadboard DIP Switch Dynamics
In `VirtualLabRack.tsx`, 8-bit DIP switches simulate physical SPST switches found on FPGA evaluation boards (Digilent Basys 3, Nexys A7):
- **Visual Anatomy**: A dark recessed switch well (`#0f1217`) housing a raised tactile rocker lever.
- **Toggle Animation**: Flipping the switch triggers a sub-100ms micro-transition moving the rocker vertically with an subtle optical bevel highlight.
- **Illuminated Status Pill**: An integrated LED badge glows bright emerald (`#10b981`) when closed (Logic `1`) and dim slate (`#475569`) when open (Logic `0`).
- **Immediate In-RAM Stimulus**: Toggling instantly calls `engineBridge.injectStimulus(port, value)`, re-evaluates all dependent combinational gates in RAM, and updates waveform traces without waiting for a clock tick.

### 3.2. Tactical Pushbutton Strobe Dynamics
Physical pushbuttons (such as `RESET` and `STEP CLK`) feature momentary contact mechanics:
- **Depression**: On `mouseDown` / `touchStart`, the button depresses with a 1px offset and glowing cyan border.
- **Strobe Release**: For active-low resets (`rst_n`), pressing immediately forces `0`; a built-in release timer automatically strobes the pin back to `1` after 250ms, simulating a human finger press and preventing the circuit from remaining permanently halted.

### 3.3. Dual Monotonic Waveform Cursor Dragging
In `WaveformViewer.tsx`, timing measurements depend on two independent vertical cursor pins:
- **Cursor A (Cyan `#06b6d4`)** and **Cursor B (Amber `#f59e0b`)**.
- **Edge Snapping**: Dragging a cursor near a signal transition edge snaps within a 6px threshold, allowing exact cycle-accurate setup and hold time measurements.
- **Floating HUD Badge**: As cursors are dragged, a floating badge displays:
  $$\Delta t = |t_B - t_A| = 10.000\text{ ns} \quad \left(f = \frac{1}{\Delta t} = 100.00\text{ MHz}\right)$$

---

## 4. Keyboard Navigation & Omnibar Modality

Power users and FPGA architects navigate primarily via the keyboard. Axiom enforces full keyboard accelerator parity:

### 4.1. Global Keyboard Shortcuts Table

```
Shortcut           Action                        Scope
───────────────────────────────────────────────────────────────────────────────
Ctrl+K / Cmd+K     Open Omnibar Command Palette  Global
F5                 Run / Pause Simulation        Global
F10                Step Physical Time (+1 ns)    Global
F11                Step Discrete Delta Cycle (δ) Global
Shift+F5           Reset Simulation to t=0       Global
Ctrl+S / Cmd+S     Trigger In-RAM JIT Compile    Code Editor
F                  Slice Logic Cone (Fan-in)     Schematic DAG
O                  Slice Logic Cone (Fan-out)    Schematic DAG
Escape             Dismiss Modal / Unfocus       Global (Stacked)
───────────────────────────────────────────────────────────────────────────────
```

### 4.2. Spotlight-Style Omnibar (`OmnibarModal.tsx`)
Triggered via `Ctrl+K` or `Cmd+K`:
- **Fuzzy Matching**: Matches signals, project files, starter templates, and studio commands with sub-10ms query times.
- **Categorized Sections**:
  - `Actions`: Run, Compile, Step, Reset, Export VCD, Export SAIF.
  - `Signals`: Instant inspection and waveform toggle for any net in the elaborated hierarchy.
  - `Files`: Jump directly to any Verilog, testbench, or constraint source.
  - `Templates`: Quick-load industry designs (RV32I, UART, SPI, PWM, ALU).
- **Keyboard Trapping & Arrow Navigation**: Up/Down keys cycle through results with active blue pill highlight; Enter executes and closes; Escape dismisses.

### 4.3. Stacked Escape Key Architecture
The `Escape` key must never cause unpredictable layout shifts. Axiom implements a priority-ordered dismiss stack:
1. **Priority 1**: Dismiss active Autocomplete dropdown or Monaco hover tooltip.
2. **Priority 2**: Dismiss Omnibar / New Project / Add Source Modal.
3. **Priority 3**: Clear active Cross-Probing selection and Logic Cone Slicing halos.
4. **Priority 4**: Restore Maximized Panel (`⛶`) back to split view.
5. **Priority 5**: Collapse expanded Bottom Dock down to status bar.

---

## 5. Optimistic UI vs. Engine Ground Truth

In real-time hardware simulation, UI components must balance responsiveness with mathematical correctness:

```
User Action (Force Signal "0xFF")
       │
       ├─────────────────────────────────────────┐
       ▼ (Instant Optimistic Feedback: <16ms)    ▼ (Engine Ground Truth: ~2ms)
[Update Virtual Lab Hex Display]          [Cranelift JIT In-RAM Re-evaluation]
[Set Gutter Value to "0xFF"]                     │
       │                                         ▼
       └─────────────────────────────────> [Stratified Event Queue Execution]
                                                 │
                                                 ▼
                                          [Waveform Trace Written to Buffer]
                                          [Voltage Sag Computed in Telemetry]
                                          [Cross-Probing Confirmed Across DAG]
```

- **Optimistic Phase**: Input controls (DIP switches, rotary dial, force modal) immediately reflect user intent visually, preventing sluggish tactile feedback.
- **Verification Phase**: The JIT engine elaborates and steps the event queue. Once execution completes, all downstream visualizers (waveforms, schematic wire probes, power dials) receive the validated engine state.
- **Rollback Handling**: If an invalid state or compile error occurs during the operation, the UI rolls back the optimistic control to the last stable checkpoint and displays an inline toast notification.
