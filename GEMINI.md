# Axiom: High-Performance Rust Remake of AMD Vivado's HDL Engine & Studio

## 1. Project Manifesto & Core Mission

**Axiom** is an aerospace-grade, high-performance, cross-platform remake of AMD Vivado's Hardware Description Language (HDL) processing, simulation, and analysis engine, built natively in **Rust** and deployed live at **`https://axiom.aerovex.net`**.

Vivado is the industry standard for FPGA development, yet it suffers from severe legacy bloat: 100+ GB installations, sluggish Java Swing interfaces, multi-stage file-based elaboration pipelines, zero-time delta cycle black-boxing, and zero support for modern platforms like macOS (Apple Silicon) or standard web browsers.

**Axiom eliminates these limitations.** It provides a lightweight (<50 MB), lightning-fast, and deeply introspectable HDL engine paired with an elegant, modern dark-themed desktop (Tauri v2) and web application (WebAssembly).

---

## 2. Current Implementation Status & Production Deliverables

As of **Phase 14.0**, the entire core engine, compiler, scheduler, telemetry system, language server (LSP), linter, primitive library emulation, and modern studio are **fully implemented, tested, and active**:
- **63 / 63 Rust Workspace Tests Passing**: Comprehensive unit, integration, benchmark, conformance, primitive emulation, and linter tests across all crates.
- **Dual-Runtime Execution & WebAssembly LSP**:
  - **Desktop Native**: Native Cranelift JIT compiling Verilog/SystemVerilog directly to x86_64 / AArch64 machine code in RAM with zero disk turnaround.
  - **In-Browser WebAssembly**: Pure client-side `wasm32-unknown-unknown` simulation kernel and in-RAM LSP static analysis linter running 100% in-browser with zero backend dependencies.
- **In-RAM Verilog/SystemVerilog LSP & Static Analysis Linter (`crates/lsp`)**:
  - 10 static design rules: syntax error mapping, blocking assignment in sequential blocks (`AXIOM_W001`), non-blocking in combinational blocks (`AXIOM_W002`), undriven nets (`AXIOM_W003`), unused signals (`AXIOM_W004`), multi-driver net contention (`AXIOM_E002`), transparent latch inference (`AXIOM_W006`), missing case default (`AXIOM_W007`), bit width mismatch (`AXIOM_W008`).
  - Standard JSON-RPC stdio daemon (`axiom lsp`) and colorized CLI reporter (`axiom lint <FILE>`).
- **Monaco Editor Integration with Custom Monarch Verilog Tokenizer**:
  - Dark engineering palette (`axiom-dark`), live debounced squiggly marker underlines (`monaco.editor.setModelMarkers`), hover tooltips with IEEE 1800 AST metadata, and autocompletion snippets/signals.
- **Primary Default Combinational Hardware System (`logic_circuit`)**:
  - Gate-level boolean logic system: `w1 = ~A; w2 = w1 & B; w3 = w2 & C; w4 = ~B; F = w3 | w4;` ($F = ((\neg A \land B) \land C) \lor \neg B$).
  - Configured as the #1 featured template on the Welcome Launchpad with full gate-level schematic DAG (9 cells, 9 nets: `inv1`, `inv2`, `and1`, `and2`, `or1`) and dedicated Virtual Lab bay with tactile switches, gate probes, output LED, and 8-row Truth Table HUD.
- **Spacious Dual-Pane Studio & Scaled Typography**:
  - Redesigned Split Studio to eliminate quad-split cramping: Left = Monaco HDL Editor; Right = Full-height, full-width Visualizer Pane (`Schematic DAG`, `Virtual Lab`, `Waveforms`, `Timing & Energy`) with optional `+ Waveforms` stack toggle and 1-click Maximize.
  - Comfortable, readable typography and touch targets across the entire interface (14px base font, 14px Monaco editor, 52px header, 280px sidebar, 32px collapsed status bar).
- **Problems & Linter Dock**: Dedicated collapsible dock tab with active diagnostic cards and 1-click jump-to-line navigation.
- **Production Web Deployment**: Live at **`https://axiom.aerovex.net/studio/`** (and docs at `https://axiom.aerovex.net/`) served via GitHub Pages with CNAME.
- **Vivado Project Management System & Welcome Launchpad**:
  - Starts cleanly from a "No Project Open" standpoint with zero pre-loaded clutter.
  - Aerospace-grade Welcome Launchpad with hero banner, "Create New Project" wizard card, "Open Project from File" (.json) importer, and 7-item interactive starter templates grid.
  - Authentic Vivado file sets (`sources_1`, `sim_1`, `constrs_1`), multi-file bundling, target FPGA parts (Artix-7, Zynq-7000, Kintex-7, Kintex UltraScale+, Axiom Virtual Silicon), active `[TOP]` module designation, and clean "Close Project" lifecycle.
- **Full Mobile Studio Support & Off-Canvas Drawer Architecture**:
  - Responsive viewport detection (`<= 768px`) completely disabling multi-pane cramping and resizable splitters.
  - Off-canvas left drawer (`MobileDrawer.tsx`) with smooth slide-out (`translateX(-100%)` to `translateX(0)`), dark blur backdrop, full Vivado project file set explorer, 1-panel-at-a-time switcher, and simulation controls.
  - 1-Panel-at-a-Time Viewing: full 100% width and 100% height single-panel rendering for Monaco HDL Code Editor, IEEE Gate Schematic DAG, Virtual Lab Rack, Waveforms, Timing Radar, or Console & REPL.
  - Tactile Mobile Bottom Bar (`MobileBottomBar.tsx`): 5 thumb-friendly tabs (`Code`, `Schematic`, `Lab`, `Waves`, `Console`) with live diagnostic and glitch badges.
- **Vivado-Grade Collision-Free Wire Routing, Zero-Turn Pin Alignment & Text Knockout Plates**:
  - Grid datapath row alignment and precision pin alignment (`fixedY` on `SchematicNode`), mathematically matching connected pin heights ($Y_{\text{out}} = Y_{\text{in}}$) to render major connections ($A \to \text{inv1}$, $B \to \text{and1}$, $\text{and1} \to \text{and2}$, $\text{and2} \to \text{or1}$, $\text{or1} \to F$) as 100% straight horizontal lines with **zero turning movements**.
  - Multi-layer channel destination stepping: wires spanning multiple layers ($dx \ge 150\text{px}$) run cleanly along their source horizontal track and execute their vertical jog in the dedicated open channel immediately before the destination ($dstX - 28$).
  - Spacious inter-layer padding (`layerSpacingX = 92px`) providing wide routing channels and eliminating visual cramping.
  - Obstacle-aware orthogonal channel routing (`routeOrthogonalEdge`) dynamically avoiding `KeepOutBox` clearance bounding boxes ($y - 18$ top margin) to detour around intermediate gates and labels.
  - Solid `#0c1017` protective background knockout plates behind all gate instance labels (`inv1`, `inv2`, `and1`, `or1`) in `SchematicViewer.tsx` guaranteeing 100% collision-free text rendering without wire overlap.
- **Phase 13.0: Professional Project Lifecycle, Dual-Runtime FileSystem & Dynamic Canvas Centering**:
  - **Header State Partitioning**: Distraction-free header when no project is open (`!project`) displaying only brand, version tag (`v0.1.0-jit`), subtle "No Project Open" badge, and language selector. Full simulation control ribbon (`Run`, `Pause`, `+1 ns`, `+100 ps`, `Step δ`, `Reset`), sim time, and PDN telemetry HUD dynamically appear only when an active project is open.
  - **Vivado-Grade Project Header Menu (`ProjectDropdown.tsx`)**: Prominent top-left project badge menu displaying target FPGA device, top module `[TOP]`, file count, manual save (`Ctrl+S`) with instant save indicator, JSON bundle export (`.json`), Add Source modal trigger, New Project wizard, and clean Close Project lifecycle.
  - **Template Lifecycle Sanitization**: Completely removed the confusing mid-project "Load Template" section from `ProjectManager.tsx`. Templates are strictly project initialization blueprints residing on the Welcome Launchpad and inside `NewProjectModal.tsx`.
  - **Dynamic Midpoint Camera Anchoring & Continuous Canvas Resizing**: Overhauled `ResizeObserver` in `SchematicViewer.tsx` to continuously update canvas resolution (`canvas.width = Math.round(newW * dpr)`) and mathematically lock the world-space camera midpoint to the visualizer pane center ($\Delta \text{offsetX} = \Delta W / 2$, $\Delta \text{offsetY} = \Delta H / 2$) during middle splitter dragging, eliminating all horizontal squishing while keeping zoom scale 100% constant.
  - **Dual-Runtime FileSystem Abstraction (`ui/src/engine/fs/`)**: Abstract `FileSystem` class implemented with `BrowserIndexedDbFileSystem` (`idb` virtual VFS + `localStorage` caching) for web browsers and `TauriIpcFileSystem` delegating to native Rust host OS filesystem commands (`fs_read_file`, `fs_write_file`, `fs_remove_file`, `fs_list_dir`, `fs_create_dir`, `fs_exists`) for the desktop application. Projects auto-save immediately to disk/IndexedDB on creation and edits.
- **Phase 13.4: Lock-Safe Multi-Session Concurrency & Desktop Single-Instance Multi-Window**:
  - **Welcome Launchpad Ergonomics**: Direct click-to-open on project cards (`cursor: pointer`), event-stopped trash buttons, vertically centered `FolderOpen` icon in standard `.btn .btn-primary` actions, and consolidated single GitHub mention in the launchpad footer.
  - **Web Locks API & Cross-Session Synchronization (`sessionSync.ts`)**: Atomic concurrency coordinator using `navigator.locks` (with sequential promise queue fallback) and cross-session broadcasting via `BroadcastChannel("axiom_session_sync")`, guarding project registry mutations and file operations across concurrent browser tabs.
  - **Desktop Single-Instance Multi-Window (`axiom-desktop`)**: Tauri v2 single-instance plugin (`tauri-plugin-single-instance = "2"`) intercepting secondary process starts to spawn additional windows on the primary process via `WebviewWindowBuilder`, paired with `MultiEngineManager` isolating Cranelift JIT sessions per window label and thread-safe `FS_MUTEX` locking on host filesystem I/O.
- **Phase 13.5: Toast Notification System, Unified Confirm Dialogs, 3-Dot Project Trashing & Fileset Architecture**:
  - **Aerospace Toast Notification System (`toast.ts` & `ToastContainer.tsx`)**: Lightweight reactive toast manager (`toast.success`, `toast.error`, `toast.info`, `toast.warning`) with auto-dismiss, smooth slide-in animations, and dark acrylic styling, completely replacing legacy browser `alert(...)`.
  - **Unified Confirm Dialog Subsystem (`ConfirmModal.tsx`)**: Dark acrylic modal dialog with promise-based `confirmDialog(...)` helper replacing all native browser `confirm(...)` dialogs across the app, with keyboard shortcuts (`Enter`/`Escape`) and zero usage of `prompt()`.
  - **3-Vertical-Dot Kebab Trashing on Main Page**: Removed "Move to Trash" from in-project header menu (`ProjectDropdown.tsx`), keeping project header clean and focused. On the main page (`WelcomeLaunchpad.tsx`), project cards house "Move to Trash" inside a 3-vertical-dot button (`MoreVertical`) with click-outside auto-close and confirmation.
  - **Vivado Fileset Label Cleanup**: Standardized sidebar fileset headers across all 7 supported languages to clean, unified titles: **Design Sources**, **Simulation Sources**, and **Constraints**, eliminating confusing `(sim_1)` and `(constrs_1)` suffixes.
- **Phase 13.7: Precise Cursor-Tracking Wire & Net Hover Card Positioning**:
  - **Fixed Wire Hover Positioning**: Resolved bug where hovering a wire in `SchematicViewer.tsx` triggered a fallback to `{ x: 20, y: 50 }` (rendering fixed at the top-left screen corner). Replaced with `edgeTooltipPos` calculating live viewport cursor coordinates (`mousePos.x + 16, mousePos.y + 16`) with viewport boundary clamping (`maxX = window.innerWidth - 270`, `maxY = window.innerHeight - 160`) preventing viewport overflow.
  - **Hover vs. Selection Priority**: Refactored `activeHoverNode` and `activeHoverEdge` memoizers so wire hovering cleanly takes priority over previously selected nodes and works even while a gate is active.
  - **Canvas Pointer Exit Cleanup**: Implemented `handleMouseLeave` on `<canvas>` clearing `hoveredNodeId` and `hoveredEdgeId` so transient hover cards disappear immediately when pointer leaves the canvas.
- **Phase 13.8: Vivado XDC LSP Engine, Fileset Plus (+) Actions, Lean Caching & Modern Drag-to-Measure Waveforms**:
  - **Vivado XDC Constraints LSP & Syntax Engine (`crates/lsp/src/xdc.rs`, `ui/src/engine/monacoXdc.ts`)**:
    - Full static analysis linter, hover docs, and autocompletions for Vivado XDC physical and timing constraints (`set_property`, `PACKAGE_PIN`, `IOSTANDARD`, `create_clock`, etc.) with zero false-positive errors on standard constraint files like `timing.xdc`.
    - Integrated with Monaco tokenizer supporting `#` comments, breadcrumb badges, and on-demand XDC validation.
  - **Fileset Plus (+) Action Buttons & 3-Dot Kebab File Context Menus**:
    - Replaced static file count badges with interactive `+` buttons on **Design Sources**, **Simulation Sources**, and **Constraints** opening `AddSourceModal` with that exact fileset pre-selected.
    - Replaced exposed file trash buttons with 3-vertical-dot kebab (`MoreVertical`) dropdowns offering "Set as Top Module" and styled "Delete" dialogs.
    - Fixed top-module tagging to prevent testbenches (`tb_*.v`, `*.sv`) from falsely inheriting `[TOP]` badges.
  - **Lean Viewport, Tab & Scroll Position Persistence ("Don't Overcache")**:
    - Persists active tabs and active file on selection/closure across page reloads.
    - Persists fileset folder expand/collapse state per project in `localStorage`.
    - Monaco editor scroll coordinates (`top`, `left`, `line`, `col`) debounced and restored per file.
    - Schematic camera pan (`offsetX`, `offsetY`), zoom (`scale`), live values, and clock nets persisted per design.
  - **Simulation Reset Rewind Fix (`engineBridge.ts`)**:
    - Resolved bug where clicking Reset uncompiled the circuit; Reset now rewinds to $t=0$, resets signal states to initial vectors, and keeps `compiled: true` so the user can immediately step or run without recompilation.
    - Dispatches `axiom_sim_reset` event.
  - **Modern Drag-to-Measure Waveform Window System & Reset-to-Zero Viewport**:
    - Replaced clunky single-click A-B cursor workflow with a modern Saleae Logic 2 / Chrome DevTools drag-to-measure window: dragging across the graph highlights a shaded measurement window with $[A, B]$ boundary handles, draggable edges, and window sliding.
    - Fixed text wrapping on the measurement HUD by enforcing `whiteSpace: "nowrap"`, compact engineering units (`formatTimeCompact`), and added a 1-click "Zoom into Window" button.
    - Attached `onWheel` to the waveform canvas for trackpad/mouse-wheel zooming and horizontal panning.
    - Added an automatic listener on `axiom_sim_reset` that rewinds the waveform graph viewport back to $t=0$ (`timeOffsetPs = 0`).
- **Phase 13.9: IEEE 1800 System Tasks ($), Procedural Delays (#), and Default Codebase Lint Sanitization**:
  - **IEEE 1800 System Tasks & Functions (`$`)**: Full lexer and parser support for identifiers starting with `$` (`$dumpfile`, `$dumpvars`, `$finish`, `$stop`, `$display`, `$monitor`, `$time`, `$realtime`, `$random`, `$clog2`) with statement-level `Statement::TaskCall` and expression-level `Expr::Call`.
  - **Procedural Delays & Event Controls**: Full support for `#` delays (`#10;`, `#20 rst_n = 1;`, `always #5 clk = ~clk;`) and event controls (`@(posedge clk);`), with graceful elaborator unwrapping and simulation filtering.
  - **Net Shorthand Initializers & Parameter Ranges**: Continuous assignment initializers on net declarations (`wire [6:0] opcode = instr[6:0];`), unpacked memory arrays (`reg [31:0] regfile [0:7];`), multi-parameter lists, and ranged parameters (`localparam [1:0]`).
  - **Submodule Instance Net Driver Recognition**: Testbench instance port connections (`.F(F)`) properly registered as driven nets to eliminate false `AXIOM_W003_UNDRIVEN_NET` warnings.
  - **100% Zero-Diagnostic Default Codebases**: Complete audit and resolution of all default templates and sample designs in `projectModel.ts`, `sampleDesigns.ts`, and `AddSourceModal.tsx`: added required `default:` branches to finite state machine `case` statements in `uart_transceiver` and `spi_master`, ensuring all default projects start completely clean with 0 warnings and 0 errors.
- **Phase 13.10: WebAssembly Standalone Worker Sandbox & SharedArrayBuffer Simulation Isolation**:
  - **Dedicated Background Web Worker (`ui/src/engine/worker/simWorker.ts`)**: Offloaded in-browser WebAssembly simulation execution and LSP processing from the main UI thread to a dedicated background Web Worker, loading `axiom_wasm_bg.wasm` client-side with zero UI freezing during heavy gate-level simulations or complex elaboration.
  - **High-Speed Autonomous Simulation Loop**: Worker runs autonomous clock ticking and step evaluation at high frequency, batching signal changes and telemetry at 60 FPS (`broadcastBatch`) to eliminate UI thread rendering bottlenecks.
  - **Lock-Free Atomic SharedArrayBuffer Synchronization (`ui/src/engine/worker/simSharedBuffer.ts`)**: Implemented 128-byte shared ring buffer with atomic sequence counter locking (`Atomics.store`, `Atomics.load`) for zero-copy, lock-free telemetry snapshots (`timePs`, `delta`, `isRunning`, `glitchCount`, `powerMw`, `currentMa`, `voltageSagV`, `railVoltageV`). Automatically falls back to structured clone messaging when `crossOriginIsolated` is false.
  - **Watchdog Supervisor & Auto-Respawn Recovery (`ui/src/engine/worker/simWorkerClient.ts`)**: 5,000ms watchdog guarding against infinite zero-time delta loops or unstable combinational oscillation hazards, automatically terminating hung workers, cleanly respawning a fresh sandbox, and re-compiling the active design without crashing the browser tab.
  - **Universal Bridge Integration (`ui/src/engine/engineBridge.ts`)**: Seamless tiered execution hierarchy (Tauri Native JIT IPC $\to$ WebWorker WebAssembly Sandbox $\to$ Main-Thread WASM $\to$ Simulated Fallback) with zero breaking changes to existing UI components.
- **Phase 14.0: Direct Xilinx 7-Series & UltraScale+ Primitive Library Emulation**:
  - **Direct In-Engine Primitive Lowering (`crates/ir/src/primitives/`)**: Full in-RAM catalog (`PrimitiveCatalog`) intercepting unknown hardware modules during AST elaboration without requiring external Verilog library definitions. Cycle- and delta-accurate lowering for `LUT6_2`, `LUT1..6`, `BUFG`, `BUFGCE`, `IBUF`, `OBUF`, `FDRE`, `FDSE`, `FDCE`, `FDPE`, `DSP48E2`, `DSP48E1`, `RAMB36E2`, `RAMB18E2`, `CARRY4`, and `CARRY8`.
  - **Cranelift Native JIT & Portable Evaluator Concat Engine (`crates/jit/src/compiler.rs`, `portable.rs`)**: Extended Cranelift JIT compiler to compile `BirExpr::Concat(items)` directly to native 64-bit machine instructions (`ishl`, `band`, `bor`) with exact IEEE 1800 bit-ordering, and added sub-64-bit masking to `BirExpr::Slice`.
  - **In-RAM LSP Primitive Documentation, Hover & Completion (`crates/lsp/src/primitives_doc.rs`)**: Dedicated markdown documentation, parameter lists, pin descriptions, and completion snippets for all Xilinx primitives. Static linter (`crates/lsp/src/linter.rs`) automatically recognizes primitive driver ports (`O`, `O5`, `O6`, `Q`, `P`, `DOUT...`) for 0 false diagnostics.
  - **Schematic DAG & Production Starter Template (`ui/src/engine/schematicModel.ts`, `sampleDesigns.ts`, `projectModel.ts`)**: Dedicated schematic DAG synthesis for `dsp_bram_mac` with clock buffer, control LUT, 36Kb True Dual-Port BRAM, DSP48E2 slice, and pipeline valid flip-flop. Specialized node badges, colors, and starter template targeted for Kintex UltraScale+ `xcku5p-ffvb676-2-e`.

---

## 3. Repository & Workspace Architecture

The repository is organized as a Cargo multi-crate workspace and modern TypeScript/React frontend:

```
axiom/
├── Cargo.toml                          # Master workspace configuration
├── GEMINI.md                           # Single-source-of-truth project context for AI agents
├── todo.md                             # Master task & milestone priority tracker
├── install.sh / install.ps1            # Universal single-line installation scripts
│
├── crates/                             # Rust Simulation Engine Workspace
│   ├── core/                           # Four-state logic (0,1,X,Z), SimTime, diagnostics & spans
│   ├── syntax/                         # Streaming zero-copy lexer, preprocessor, and Pratt AST parser
│   ├── ir/                             # Elaborator, symbol tables, module hierarchy & BIR (IR)
│   ├── jit/                            # Cranelift in-RAM JIT backend & 4-state memory manager
│   ├── sim/                            # Stratified event queue, delta-cycle loop, glitch detector
│   ├── telemetry/                      # Dynamic power, PDN inductive sag ($V_{sag} = IR + L di/dt$), VCD/SAIF
│   ├── lsp/                            # In-RAM Verilog/SystemVerilog LSP 3.17 server & static analysis linter
│   ├── desktop/                        # DesktopEngine library and native Tauri v2 IPC handlers
│   ├── cli/                            # Unified CLI & In-RAM GUI server (embedded Zstd UI bundle)
│   └── wasm/                           # wasm-bindgen WebAssembly wrapper for in-browser simulation & LSP
│
├── ui/                                 # React 19 + TypeScript + PostCSS Web & Desktop Studio
│   ├── src/
│   │   ├── engine/
│   │   │   ├── worker/                 # Web Worker isolation, SharedArrayBuffer & watchdog supervisor
│   │   │   │   ├── simWorker.ts        # Standalone worker kernel executing WebAssembly in background
│   │   │   │   ├── simWorkerClient.ts  # Client coordinator, request-response router & watchdog supervisor
│   │   │   │   ├── simWorkerProtocol.ts# Strongly typed worker IPC command & event batch message definitions
│   │   │   │   └── simSharedBuffer.ts  # 128-byte SharedArrayBuffer ring buffer with Atomics synchronization
│   │   │   ├── engineBridge.ts         # Dual-runtime bridge (auto-detects Tauri IPC vs Web Worker vs WASM)
│   │   │   ├── projectModel.ts         # Vivado project model (sources_1, sim_1, constrs_1, templates)
│   │   │   ├── packageModel.ts         # Vivado FPGA package definition, BGA pins, die floorplan & XDC
│   │   │   ├── monacoVerilog.ts        # Monarch Verilog tokenizer, axiom-dark theme & LSP providers
│   │   │   ├── schematicModel.ts       # Synthesizes hardware netlist DAG for schematic viewer
│   │   │   ├── timingModel.ts          # Static timing analysis, slack radar, CDC matrix, energy treemap
│   │   │   └── sampleDesigns.ts        # 7 production Verilog hardware systems
│   │   ├── components/
│   │   │   ├── Header.tsx              # Execution control (Run, Step, +1ns, +100ps, Step δ, Reset)
│   │   │   ├── Sidebar.tsx             # Collapsible sidebar (Sources explorer vs Netlist hierarchy)
│   │   │   ├── ProjectManager.tsx      # Vivado file sets explorer, top-module picker, templates
│   │   │   ├── NewProjectModal.tsx     # Vivado project creation wizard with target FPGA part selection
│   │   │   ├── AddSourceModal.tsx      # Add source dialog for sources_1, sim_1, constrs_1
│   │   │   ├── HdlEditor.tsx           # Monaco multi-tab editor with live markers, [TOP] tag, breadcrumb
│   │   │   ├── UnifiedBottomDock.tsx   # Collapsible dock: Console & REPL, Problems & Linter, Power, Glitches
│   │   │   ├── WaveformViewer.tsx      # Multi-radix traces, dual cursors, delta-cycle accordion
│   │   │   ├── SchematicViewer.tsx     # GPU-accelerated netlist DAG, semantic LOD, logic cone slicer
│   │   │   ├── PackageVisualizer.tsx   # FPGA Package BGA ball grid map, silicon die & I/O Ports dock
│   │   │   ├── VirtualLabRack.tsx      # DIP switches, buttons, 7-seg LEDs, UART/SPI/PWM/RISC-V bays
│   │   │   ├── TimingRadarViewer.tsx   # Slack waterfall, CDC matrix, hierarchical energy treemap
│   │   │   ├── OmnibarModal.tsx        # Ctrl+K Spotlight-style command palette & fuzzy finder
│   │   │   └── ResizableSplitter.tsx   # Zero-dependency draggable splitter handles
│   │   └── styles/theme.css            # Dark engineering aesthetic design system
│   └── dist/                           # Production web bundle (pre-compiled into CLI)
│
├── docs/                               # VitePress Documentation Portal (https://axiom.aerovex.net)
├── analysis/                           # Axiom Architectural Blueprints (14 detailed specs)
└── vivadoanalysis/                     # AMD Vivado Reverse-Engineering & Architecture Critique
```

---

## 4. Key Architectural Subsystems

### 4.1. In-RAM JIT Machine Code & WebAssembly Engine
- **No External Compilers**: Verilog/SystemVerilog parsing and elaboration occur entirely in Rust.
- **Cranelift Native JIT**: Directly emits native machine instructions into executable memory in RAM in milliseconds.
- **WebAssembly Client**: Compiles to `wasm32-unknown-unknown`, allowing browser simulation without server computation.
- **Contiguous 4-State Arena**: All signal states reside in contiguous `values: Box<[u64]>` and `masks: Box<[u64]>` memory, guaranteeing L1/L2 cache locality.

### 4.2. Stratified Event Scheduler & Delta-Cycle Introspection
- **Full IEEE 1800 Compliance**: Implements Stratified Event Queue phases: Active $\rightarrow$ Inactive $\rightarrow$ NBA (Non-Blocking Assignments) $\rightarrow$ Observed $\rightarrow$ Reactive.
- **Granular Stepping APIs**:
  - `engine.tick(delta_ps)`: Advances physical simulation time by arbitrary picoseconds.
  - `engine.step_delta()`: Advances a discrete zero-time $\delta$-cycle within current time.
- **Glitch & Hazard Radar**: Automatically intercepts and flags zero-time combinational glitches (static-0, static-1, dynamic hazards) that Vivado conceals.

### 4.3. Physics-Informed Voltage, Energy & Power Telemetry
- **Dynamic Power**: Modeled from net capacitance, voltage rails, and toggle rate:
  $$P_{\text{dynamic}} = \frac{1}{2} C_{\text{net}} V_{\text{dd}}^2 f \alpha$$
- **Inductive PDN Droop**: Models power distribution network inductance ($V_{\text{sag}} = IR + L \frac{di}{dt}$), flagging micro-spikes during clock transitions.
- **Export Standards**: Generates standard Value Change Dump (VCD) and Switching Activity Interchange Format (SAIF 2.0).

### 4.4. Vivado Project Management & De-Cramped Ergonomic Studio
- **Authentic Vivado File Sets**:
  - `Design Sources (sources_1)`: Verilog / SystemVerilog RTL modules.
  - `Simulation Sources (sim_1)`: Testbenches with stimulus generators.
  - `Constraints (constrs_1)`: Timing & pin constraints (`timing.xdc`).
- **Target Silicon Devices**: Artix-7 (`xc7a35t`, `xc7a100t`), Zynq-7000 (`xc7z020`), Kintex-7 (`xc7k325t`), Kintex UltraScale+ (`xcku5p`), Axiom Virtual Silicon.
- **Ergonomic De-Cramping**:
  - **Unified Bottom Dock**: Collapsible to a 28px status bar, liberating ~250px of vertical space.
  - **Collapsible Sidebar**: Shrinks to a 38px vertical icon strip, liberating 222px of horizontal space.
  - **1-Click Panel Maximization (Full Screen)**: Expands Editor, Waveforms, Schematic DAG, or Virtual Lab to 100% full screen.

---

## 5. Strict Operating Guidelines & Git Workflow Policy

> [!IMPORTANT]
> **Strict No-Emoji Policy:**
> **Do NOT use emojis anywhere in the codebase, UI/UX, or agent markdown files.**
> Always use clean SVG icons (such as Lucide React icons in the frontend) or concise text labels instead of emojis. Never include emojis in menu titles, buttons, tabs, tooltips, dialogs, console messages, or documentation.

> [!NOTE]
> **Git Workflow Policy:**
> **You can commit and push after every change.**
> Always ensure that changes are verified (e.g., `npm run build`, `cargo test`), clean, and committed with clear descriptive messages.
> Destructive Git operations (such as `git reset --hard`, `git clean -f`, force-pushing `git push --force`, or destructive rebasing) remain strictly forbidden to preserve repository history integrity.

---

## 6. Build, Test & Verification Commands

When working in the repository, verify changes using these standard commands:

- **Build Rust Workspace & Test Suite**:
  ```bash
  cargo test --workspace
  ```
  *(Ensures all 48 unit, integration, benchmark, and conformance tests pass)*

- **Build TypeScript / React UI**:
  ```bash
  npm --prefix ui run build
  ```
  *(Runs TypeScript `tsc` check and Vite production bundling)*

- **Run UI Development Server**:
  ```bash
  npm --prefix ui run dev
  ```
  *(Launches Vite dev server on `http://localhost:3000/`)*

- **Build Rust Desktop & CLI Binaries**:
  ```bash
  cargo build --release --bin axiom --bin axiom-desktop
  ```

- **Run Native Standalone Desktop Studio (Zero-Port Tauri Window)**:
  ```bash
  ./target/release/axiom-desktop
  # or via CLI launcher:
  cargo run --bin axiom -- gui
  ```

---

## 7. Master Documentation & Analysis Index

For deep architectural and implementation specifications, refer to:

1. **`analysis/analysis.md`**: Master index of the 14 comprehensive technical blueprints:
   - `01_engine_architecture.md`: Multi-crate workspace & zero-allocation memory model.
   - `02_lexer_parser_ast.md`: Zero-copy lexer, preprocessor & resilient AST parser.
   - `03_elaboration_and_netlist_ir.md`: Elaboration pipeline & BIR netlist intermediate representation.
   - `04_jit_machine_code_compiler.md`: Cranelift in-RAM JIT & WebAssembly codegen.
   - `05_event_scheduler_and_delta_stepping.md`: Stratified IEEE 1800 event queue & delta API.
   - `06_voltage_energy_telemetry_model.md`: Physics-informed dynamic power & PDN droop.
   - `07_api_and_runtime_interface.md`: Embeddable Rust, C-ABI & WASM API.
   - `08_desktop_and_web_ui.md`: Tauri v2 + React 19 + PostCSS desktop and web studio.
   - `09_schematic_dag_and_synthesis_viewer.md`: GPU-accelerated hardware DAG & cone slicer.
   - `10_virtual_lab_and_stimulus_rack.md`: Zero-JTAG virtual instruments & stimulus injection.
   - `11_timing_radar_and_slack_waterfall.md`: Setup/hold slack waterfall & CDC matrix.
   - `12_hierarchical_energy_treemap_and_thermal.md`: Silicon energy treemap & SSN model.
   - `13_omnibar_and_scripting_repl.md`: Ctrl+K Omnibar palette & interactive shell.

2. **`vivadoanalysis/`**: Deep comparative analysis of AMD Vivado's internal tools (`xvlog`, `xelab`, `xsim`, `ug907`, TCL CLI).

3. **`todo.md`**: Master milestone and task priority tracker. Must strictly record every change, user request, phase progression, and architectural enhancement.

---

## 8. Continuous Documentation & Task Tracking Policies

1. **Continuous Analysis Documentation Currency**:
   - All architectural blueprints under `analysis/` and `analysis/uiux/` are living documents.
   - Whenever any subsystem, UI/UX layout, theme token, engine API, or interaction pattern is created, modified, or refined, the corresponding architectural documents must be continuously updated in tandem to prevent architectural drift.
2. **Strict Master Task Tracker Policy (`todo.md`)**:
   - `todo.md` is the single source of truth for all milestone progress.
   - Every user request, agent modification, architectural enhancement, and bugfix must be documented in `todo.md` with explicit task breakdowns and completion checkboxes.
3. **Automatic User Feedback & Request Ingestion Policy**:
   - Anytime the user provides a prompt containing feedback, UX/UI critiques, feature adjustments, or bug reports (e.g. context menus, naming, tree styling, selection mechanics), you MUST immediately record all requested items into `todo.md` under an active priority section before proceeding with execution.


