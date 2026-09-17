# Axiom EDA: Unified Omnibar & Scripting REPL Architecture

## 1. Architectural Motivation & Critique of Vivado

In legacy AMD Vivado, user interaction is split between two clunky extremes:
1. **Deeply Nested Menus and Dialogs**: Running a simple simulation step, toggling radix display, or tracing a signal requires navigating cascading multi-level Java Swing menus, right-click context trees, or modal property sheets.
2. **Obscure File-Bound Tcl Console**: The Vivado Tcl console is rigid, lacks modern fuzzy discovery, dumps thousands of lines of unformatted warning text to disk log files (`vivado.log`), and has sluggish terminal response.

**Axiom EDA's Innovation**:
Axiom unifies navigation, simulation control, signal inspection, and batch automation through two complementary high-velocity interfaces:
- **The Unified Omnibar (`Ctrl+K` / `Cmd+K`)**: A Spotlight-style fuzzy search palette providing instant, keyboard-driven access to every command, simulation view, sample design, internal net, and technical documentation page.
- **The In-UI Scripting REPL Shell**: A responsive terminal console embedded directly within the bottom drawer, offering instantaneous execution of simulation commands against the in-RAM JIT state arena with command history, auto-completion, and formatted diagnostics.

---

## 2. Omnibar Architecture & Fuzzy Dispatch

```
[ User Input (Ctrl+K) ]
           │
           ▼
┌────────────────────────────────────────────────────────┐
│               Omnibar Modal Engine                     │
│  - Normalized Case-Insensitive Fuzzy Matcher           │
│  - Weighted Categorical Partitioning:                  │
│    ├── Actions     (step, step delta, reset, compile)  │
│    ├── Views       (waveforms, schematic, lab, timing) │
│    ├── Signals     (direct net selection & probing)    │
│    ├── Designs     (instant fixture switching)         │
│    └── Docs        (deep-links to architectural specs) │
└──────────────────────────┬─────────────────────────────┘
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
[ UI State Transitions ]     [ In-RAM Engine Bridge ]
 (Active tab, cross-probe)    (tick, stepDelta, reset)
```

### 2.1 Index Aggregation

The Omnibar aggregates search indices dynamically from the live simulation state and application catalog:
- **Actions**: Direct dispatchers for physical time advances (`+1 ns`, `+100 ps`), zero-time delta stepping (`step_delta`), JIT re-compilation, and state resets.
- **Views**: 1-key switches to Waveforms, Schematic DAG, Virtual Lab Rack, Timing Radar & Slack Waterfall, and 3-Way Split Studio.
- **Live Netlist Signals**: Dynamically pulls all declared nets and ports from `state.signals` with bit widths and current values (e.g. `alu_8bit.result [7:0] = 0x00`).
- **Fixture Catalog**: Immediate switching between verified hardware designs (8-bit ALU, 32-bit Pipelined ALU, 4-bit Gray Counter, Dual-Clock Asynchronous FIFO).
- **Embedded Docs**: Deep-links to local documentation, math formulas, and GitHub reference guides.

### 2.2 Performance Guarantees

- **Zero Disk Latency**: Indexing is performed entirely in client RAM. Queries execute in sub-millisecond time ($< 0.5 \text{ ms}$ for 1,000+ items).
- **Keyboard-First Ergonomics**:
  - `Ctrl+K` / `Cmd+K`: Open / Close toggle.
  - `ArrowDown` / `ArrowUp`: Move selection cursor with auto-scroll into view.
  - `Enter`: Execute selected item action and auto-close.
  - `Escape`: Cancel and dismiss.

---

## 3. Interactive Scripting REPL Architecture

Located in the bottom console panel under the `REPL` tab, the embedded shell provides engineers with a lightweight, reproducible command-line interface directly bound to the `engineBridge`.

### 3.1 Supported Command Set

| Command | Arguments | Functional Behavior |
| :--- | :--- | :--- |
| `help` | None | Lists all available shell commands and syntax examples. |
| `run` | `[time_ps]` | Advances physical simulation time by specified picoseconds (default: 1000 ps = 1 ns). |
| `step` | None | Steps forward by one default physical clock step (1000 ps). |
| `step delta` | None | Executes a single discrete zero-time delta cycle ($\delta$-step) to inspect race conditions. |
| `reset` | None | Re-initializes simulation state arena to $t = 0 \text{ ps}$, $\delta = 0$. |
| `get` | `<signal_name>` | Reads current 4-state value, width, and bitmask for the specified net. |
| `set` | `<signal_name> <hex_or_bin>` | Injects a new value into the target net and triggers immediate fan-out re-evaluation. |
| `force` | `<signal_name> <val>` | Overrides driver on the net and forces fixed logic state. |
| `release` | `<signal_name>` | Releases manual driver override on the net. |
| `report_timing` | None | Summarizes Worst Negative Slack (WNS), Total Negative Slack (TNS), and $f_{max}$. |
| `report_power` | None | Emits instantaneous dynamic power ($P_{dyn}$), voltage rail droop, and thermal status. |
| `clear` | None | Clears terminal scrollback buffer. |

### 3.2 Terminal Features

- **Command History**: Persistent history buffer navigatable using `Up` and `Down` arrow keys.
- **Tab Auto-Completion**: Typing command prefixes (e.g. `rep` + `Tab` $\to$ `report_timing`) auto-completes the input prompt.
- **Stream Formatter**: REPL outputs are color-coded into distinct semantic types:
  - Cyan (`> command` echo).
  - Green (`success` status and value readouts).
  - Amber (`warning` / glitch notices).
  - Rose (`error` diagnostics with suggestions).
- **Direct Bridge Dispatch**: Every command interacts directly with the in-RAM JIT engine without process spawning, file redirection, or IPC latency.
