# Axiom: High-Performance Rust Remake of AMD Vivado's HDL Engine & Studio

## 1. Project Manifesto & Core Mission

**Axiom** is an aerospace-grade, high-performance, cross-platform remake of AMD Vivado's Hardware Description Language (HDL) processing, simulation, and analysis engine, built natively in **Rust** and deployed live at **`https://axiom.aerovex.net`**.

Vivado is the industry standard for FPGA development, yet it suffers from legacy bloat: 100+ GB installations, sluggish Java Swing interfaces, multi-stage file-based elaboration pipelines, zero-time delta cycle black-boxing, and zero support for modern platforms like macOS (Apple Silicon) or standard web browsers.

**Axiom eliminates these limitations.** It provides a lightweight (<50 MB), lightning-fast, and deeply introspectable HDL engine paired with an elegant, modern dark-themed desktop (Tauri v2) and web application (WebAssembly).

### Core Technological Pillars:
1. **In-RAM Cranelift JIT Compilation**: Direct compilation of Verilog/SystemVerilog into native machine code in RAM in milliseconds with zero disk turnaround.
2. **Stratified Event Queue & Delta-Cycle Introspection**: Full visibility into zero-time delta cycles (Active -> Inactive -> NBA -> Observed -> Reactive), exposing combinational race conditions and glitches concealed by legacy tools.
3. **Physics-Informed Silicon Power & Voltage Telemetry**: Live dynamic power ($P = \frac{1}{2} C V^2 f \alpha$) and PDN inductive voltage droop modeling ($V_{sag} = IR + L\frac{di}{dt}$) with standard VCD and SAIF 2.0 exporters.
4. **Dual-Runtime Desktop & In-Browser Execution**: Native Cranelift JIT on x86_64 / AArch64 desktops via Tauri v2, and pure client-side `wasm32-unknown-unknown` simulation kernel with Web Worker sandbox in browsers.
5. **Modern Engineering Studio**: Split dual-pane workspace with Monaco Verilog/SystemVerilog/XDC editor, IEEE gate-level schematic DAG, virtual FPGA hardware bay (Digilent Basys 3 / Nexys A7), high-density digital/analog waveform viewer with drag-to-measure window, static timing analysis radar, FSM bubble diagrams, and protocol decoders.

---

## 2. Agent Standard Operating Procedure & Phase Workflow

All AI agents working on this repository must strictly adhere to the standardized 5-step development lifecycle:

```
[ Step 1: Todo Future Phase ]
           |
           v
[ Step 2: Formulate Implementation Plan ]
           |
           v
[ Step 3: Wait for User Acceptance ]
           |
           v
[ Step 4: Apply Plan Across Codebase ]
           |
           v
[ Step 5: Verify Quality Gate & Move to Completed ]
```

### Step 1: Todo Future Phase
- Consult `todo.md` before planning architectural changes, new subsystems, or feature enhancements.
- Identify the active phase or establish a new phase under the `## Todo` section in `todo.md`.
- Break down the phase into explicit, actionable task checkboxes with target files specified.

### Step 2: Formulate Implementation Plan
- Formulate an in-depth implementation plan before making source code edits or running mutating operations.
- Write the plan to the implementation plan artifact (`implementation_plan.md`) with:
  - Background context and problem statement.
  - User review items (highlighting any breaking changes, architectural pivots, or design decisions).
  - Proposed changes categorized logically by crate, component, or file.
  - Verification plan covering automated tests and manual UI/UX verification steps.
- Set `request_feedback = true` and `user_facing = true`.

### Step 3: Wait for User Acceptance
- Stop calling tools and pause execution.
- Present a concise, high-level summary of the implementation plan pointing to the artifact.
- **Do NOT proceed to execution until the user explicitly reviews and accepts the plan.**

### Step 4: Apply Plan Across Codebase
- Execute the approved changes cleanly across affected Rust crates, UI components, tests, and documentation.
- Maintain code cleanliness, documentation integrity, and preserve existing comments.
- Synchronize all user-facing UI strings across all 7 supported language dictionaries (`en`, `tr`, `de`, `ja`, `zh`, `es`, `fr`).

### Step 5: Verify Quality Gate & Move Phase to Completed
- Execute the full verification suite (automated tests, strict clippy, TypeScript build, zero-emoji audit).
- Verify that changes fulfill all acceptance criteria without regressions.
- Update `todo.md`: move the completed phase and its subtasks from `## Todo` to `## Completed`.
- Commit and push changes with clear, descriptive commit messages following the repository Git policy.

---

## 3. Repository & Workspace Architecture

The repository is structured as a high-performance Cargo multi-crate workspace paired with a modern React 19 / TypeScript studio and VitePress documentation portal:

```
axiom/
|-- Cargo.toml                          # Master workspace configuration
|-- GEMINI.md                           # Single-source-of-truth project context for AI agents
|-- todo.md                             # Master task & milestone priority tracker
|-- install.sh / install.ps1            # Universal single-line installation scripts
|
|-- crates/                             # Rust Simulation & HDL Workspace
|   |-- core/                           # Four-state logic (0,1,X,Z), SimTime, diagnostics & spans
|   |-- syntax/                         # Zero-copy lexer, preprocessor, and Pratt AST parser
|   |-- ir/                             # Elaborator, symbol tables, module hierarchy, BIR netlist & primitives
|   |-- jit/                            # Cranelift in-RAM JIT backend & 4-state memory manager
|   |-- sim/                            # Stratified event queue, delta-cycle loop, glitch detector & protocols
|   |-- telemetry/                      # Dynamic power, PDN inductive sag ($V_{sag} = IR + L di/dt$), VCD & SAIF
|   |-- lsp/                            # In-RAM Verilog/SystemVerilog/XDC LSP server & static analysis linter
|   |-- desktop/                        # DesktopEngine library and native Tauri v2 IPC handlers
|   |-- cli/                            # Unified CLI & in-RAM GUI server (embedded Zstd UI bundle)
|   `-- wasm/                           # wasm-bindgen WebAssembly wrapper for in-browser simulation & LSP
|
|-- ui/                                 # React 19 + TypeScript + PostCSS Web & Desktop Studio
|   |-- src/
|   |   |-- engine/                     # Simulation bridges, worker client, project model, timing & VFS
|   |   |-- components/                 # Workspace views: Monaco editor, Schematic DAG, Lab Rack, Waves, FSM
|   |   |-- i18n/                       # 7 language dictionaries (en, tr, de, ja, zh, es, fr)
|   |   `-- styles/theme.css            # Dark engineering aesthetic design system
|   `-- dist/                           # Production web bundle (pre-compiled into CLI)
|
|-- docs/                               # VitePress Documentation Portal (59 comprehensive guides)
|-- analysis/                           # Axiom Architectural Blueprints (14 detailed specs + uiux/)
`-- vivadoanalysis/                     # AMD Vivado Reverse-Engineering & Architecture Critique
```

---

## 4. Strict Agent Operating Policies

All agents operating in this codebase must adhere to the following non-negotiable rules:

### 4.1. Strict No-Emoji Policy
> [!IMPORTANT]
> **Do NOT use emojis anywhere in the codebase, UI/UX, or agent markdown files.**
> Always use clean SVG icons (such as Lucide React icons in the frontend) or concise text labels instead of emojis. Never include emojis in menu titles, buttons, tabs, tooltips, dialogs, console messages, commit messages, or documentation.

### 4.2. Continuous Verification Quality Gate
Before concluding any task or moving a phase to completed, all three quality gates must pass cleanly:
1. **Rust Workspace Tests**:
   ```bash
   cargo test --workspace
   ```
   *(All 160+ unit, integration, primitive emulation, protocol decoding, and synthesis tests must pass with 0 failures)*
2. **Rust Strict Clippy**:
   ```bash
   cargo clippy --workspace --all-targets -- -D warnings
   ```
   *(Must compile with 0 warnings)*
3. **Frontend Production Build**:
   ```bash
   npm --prefix ui run build
   ```
   *(Must complete TypeScript checks and Vite bundling with 0 errors)*

### 4.3. Git Workflow & Commit Policy
> [!NOTE]
> **You can commit and push after every change.**
> Always ensure that changes are verified, clean, and committed with clear descriptive messages.
> Destructive Git operations (such as `git reset --hard`, `git clean -f`, force-pushing `git push --force` on shared branches, or destructive rebasing) remain strictly forbidden to preserve repository history integrity.

### 4.4. Continuous Internationalization (i18n) Key Parity
- Whenever a new user-facing UI string is introduced, add it to `ui/src/i18n/en.ts`.
- Immediately synchronize translations across all 6 additional supported languages:
  - Turkish: `ui/src/i18n/tr.ts`
  - German: `ui/src/i18n/de.ts`
  - Japanese: `ui/src/i18n/ja.ts`
  - Chinese: `ui/src/i18n/zh.ts`
  - Spanish: `ui/src/i18n/es.ts`
  - French: `ui/src/i18n/fr.ts`
- Maintain 100% key parity across all language dictionaries with zero missing translation keys.

### 4.5. Single Source of Truth Hierarchy
To prevent documentation bloat and maintain clear authority:
- **`todo.md`**: The exclusive single source of truth for phase tracking, task breakdowns, roadmap milestones, and completion status. Never maintain separate phase checklists in `GEMINI.md`.
- **`analysis/`**: The authoritative source for deep architectural blueprints, internal data structures, and mathematical formulas. Refer to these files instead of duplicating specs in `GEMINI.md`.
- **`docs/`**: The public-facing documentation portal covering user guides, HDL references, and platform tutorials.
- **`GEMINI.md`**: High-density operational rules, workspace architecture, and workflow SOP for AI agents.

---

## 5. Master Subsystem & Reference Index

For deep architectural and implementation specifications, refer directly to the corresponding analysis documents:

| Subsystem Area | Architectural Blueprint | Documentation Guide |
| :--- | :--- | :--- |
| Engine Architecture & Memory Model | `analysis/01_engine_architecture.md` | `docs/architecture/` |
| Lexer, Parser & AST Representation | `analysis/02_lexer_parser_ast.md` | `docs/languages/verilog/` |
| Elaboration & BIR Netlist Intermediate Representation | `analysis/03_elaboration_and_netlist_ir.md` | `docs/architecture/` |
| Cranelift In-RAM JIT & WASM Codegen | `analysis/04_jit_machine_code_compiler.md` | `docs/architecture/` |
| Stratified Event Queue & Delta Stepping | `analysis/05_event_scheduler_and_delta_stepping.md` | `docs/ui/simulation-dock.md` |
| Dynamic Power & Inductive PDN Droop Telemetry | `analysis/06_voltage_energy_telemetry_model.md` | `docs/ui/telemetry-energy.md` |
| C-ABI, WASM & Embeddable Runtime Interface | `analysis/07_api_and_runtime_interface.md` | `docs/architecture/` |
| Tauri v2 Desktop & React 19 Studio UI | `analysis/08_desktop_and_web_ui.md` | `docs/ui/overview.md` |
| Schematic DAG & Technology Mapping | `analysis/09_schematic_dag_and_synthesis_viewer.md` | `docs/ui/schematic-viewer.md` |
| Virtual Lab Hardware Bays & Basys 3 Board | `analysis/10_virtual_lab_and_stimulus_rack.md` | `docs/ui/virtual-lab.md` |
| Static Timing Analysis Radar & Slack Waterfall | `analysis/11_timing_radar_and_slack_waterfall.md` | `docs/ui/timing-radar.md` |
| Energy Treemap & Thermal Distribution | `analysis/12_hierarchical_energy_treemap_and_thermal.md` | `docs/ui/telemetry-energy.md` |
| Omnibar Command Palette & Scripting Shell | `analysis/13_omnibar_and_scripting_repl.md` | `docs/ui/simulation-dock.md` |
| Studio UI Design System & Component Guidelines | `analysis/uiux/` | `docs/ui/` |
| AMD Vivado Comparative Analysis & Reverse Engineering | `vivadoanalysis/` | `docs/vivado/` |

---

## 6. Standard Development & Verification Commands

```bash
# 1. Run full Rust workspace test suite
cargo test --workspace

# 2. Run Rust strict clippy (zero warnings enforced)
cargo clippy --workspace --all-targets -- -D warnings

# 3. Build and check TypeScript / React Studio UI
npm --prefix ui run build

# 4. Launch Studio UI development server
npm --prefix ui run dev

# 5. Build release binaries (CLI & Native Desktop Studio)
cargo build --release --bin axiom --bin axiom-desktop

# 6. Build VitePress documentation portal
npm --prefix docs run docs:build
```
