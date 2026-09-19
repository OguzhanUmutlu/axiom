# Axiom EDA — Implementation Todo & Progress Tracker

> **Feature Overview:**
> **Axiom** (formerly Axiom) is an aerospace-grade, high-performance, cross-platform Electronic Design Automation (EDA) suite and hardware simulation engine built natively in **Rust** under the **Aerovex** platform (`https://axiom.aerovex.net`).
> 
> Axiom eliminates decades of legacy EDA bloat by providing:
> 1. **In-RAM Cranelift JIT Compilation**: Direct translation of Verilog/SystemVerilog into native machine code in milliseconds with zero disk turnaround.
> 2. **Manual Delta-Time Tick & Event Queue Inspection**: Full visibility into zero-time $\delta$-cycles, exposing combinational race conditions and glitches hidden by Vivado.
> 3. **Physics-Informed Silicon Power & Voltage Telemetry**: Live dynamic power ($P = \frac{1}{2} C V^2 f \alpha$) and PDN inductive voltage sag modeling ($V_{sag} = IR + L\frac{di}{dt}$) with VCD & SAIF 2.0 exporters.
> 4. **Dual-Target Desktop & Web UI**: Built with Tauri v2, React 19, TypeScript, PostCSS, and Vite, running seamlessly on Linux, macOS (Apple Silicon), Windows, and in standard web browsers.
> 5. **VitePress Documentation Portal**: Hosted on GitHub Pages with custom domain redirect to `axiom.aerovex.net`.

---
---
## In Progress

*(None currently active - Phase 12.12 complete)*

---

## Todo

### Future Enhancement Roadmap
- [ ] **Phase 13: WebAssembly Standalone Worker Sandbox**: WebWorker multithreaded simulation isolation with SharedArrayBuffer.
- [ ] **Phase 14: Direct Xilinx 7-Series & UltraScale+ Primitive Library Emulation**: Pre-compiled primitives for LUT6_2, DSP48E2, RAMB36E2, and BUFG.

---

## Completed

- [x] **Phase 13.8: Vivado XDC LSP Engine, Fileset Plus (+) Actions, 3-Dot File Menus & Lean Viewport/Scroll Caching - [P0]**
  - [x] **Vivado XDC Constraints LSP Engine & Monaco Monarch Tokenizer**:
    - Built comprehensive Vivado XDC constraint static analysis linter (`XdcLinter`), hover documentation provider (`XdcHover`), and completion provider (`XdcCompletion`) in `crates/lsp/src/xdc.rs`.
    - Validates all core Vivado XDC commands (`set_property`, `create_clock`, `create_generated_clock`, `set_input_delay`, `set_output_delay`, `set_false_path`, `set_max_delay`, `get_ports`, `get_pins`, `get_nets`), physical properties (`PACKAGE_PIN`, `IOSTANDARD`, `DRIVE`, `SLEW`, `PULLUP`, `PULLDOWN`), and IO standards (`LVCMOS33`, `LVCMOS25`, `LVCMOS18`, `LVDS_25`, etc.).
    - Integrated with LSP server (`textDocument/hover`, `textDocument/completion`, `publish_diagnostics`), CLI (`axiom lint`), and WebAssembly (`crates/wasm/src/lib.rs`).
    - Implemented client Monarch tokenizer and language configuration in `ui/src/engine/monacoXdc.ts` supporting `#` line comments and dark engineering syntax highlighting.
    - Updated `HdlEditor.tsx` to automatically switch editor language (`language={isXdc ? "xdc" : "verilog"}`), display "Vivado XDC" breadcrumb badge, and provide dedicated "Check XDC" validation button.
  - [x] **Fileset Header Plus (+) Buttons & 3-Dot File Kebab Context Menu**:
    - In `ProjectManager.tsx`, replaced static `({count})` file badges on **Design Sources**, **Simulation Sources**, and **Constraints** with interactive `<Plus size={13} />` icon buttons.
    - Clicking the fileset `+` button opens `AddSourceModal` with that exact fileset pre-selected and pre-filled with the corresponding template and extension (`sources_1` -> `.v`, `sim_1` -> `_tb.v`, `constrs_1` -> `.xdc`).
    - Replaced the exposed `<Trash2>` button on each file row with a sleek 3-vertical-dot button (`MoreVertical`) that opens a floating context menu.
    - Context menu provides "Set as Top Module" (for eligible design sources) and "Delete" (triggering unified dark-acrylic `confirmDialog`).
    - Corrected `isTop` module detection to strictly target `sources_1` design files (`file.name === "${topModule}.v"`), eliminating false `[TOP]` badges on simulation testbenches (e.g. `tb_logic_circuit.sv`).
  - [x] **Lean Workspace, Viewport & Scroll Position Persistence**:
    - **Open Files & Active Tab**: Fixed `handleSelectFile` and `handleCloseTab` in `App.tsx` to immediately save project state (`saveProjectToStorage(updated)`), ensuring open tabs and active file are never lost across page reloads.
    - **Project Fileset Folders**: Persisted collapsed/expanded state of `sourcesOpen`, `simOpen`, and `constrsOpen` in `localStorage` under `axiom_folders_${project.id}`.
    - **Monaco ViewState & File Scroll Index**: `HdlEditor.tsx` preserves Monaco editor view states across tab switches in-memory and caches scroll index (`top`, `left`, `line`, `col`) in `localStorage` under `axiom_file_scroll_${fileId}` with debounced scroll event listeners, returning users to their exact cursor and scroll location.
    - **Schematic Camera Viewport**: `SchematicViewer.tsx` caches pan coordinates (`offsetX`, `offsetY`) and zoom (`scale`) under `axiom_schematic_cam_${activeDesignId}`, restoring camera viewport on mount or design switch without unwanted auto-fit resets. Also caches `showLiveValues` and `hideClockNets`.
    - **Waveform Viewport**: `WaveformViewer.tsx` caches `timeOffsetPs` and `pixelsPerPs` under `axiom_wave_viewport_${topModule}`.
  - [x] **Simulation Reset Engine Fix & Waveform Graph t=0 Viewport Rewind**:
    - Rewrote `engineBridge.reset()` so that resetting the simulation clock to $t=0$ keeps `compiled = true` and leaves the circuit fully initialized with valid states, eliminating the annoying behavior where the compile button turned on and forced manual recompilation.
    - Dispatches an `axiom_sim_reset` event that triggers `WaveformViewer` to smoothly rewind its viewport back to $t=0$ (`timeOffsetPs = 0`).
  - [x] **Modern Drag-to-Measure Waveform Window Selection & Non-Wrapping HUD**:
    - Replaced the rigid two-click A-B cursor workflow with a modern drag-to-measure window system (inspired by Saleae Logic 2 and Chrome DevTools). Dragging across the graph highlights a translucent shaded measurement window with draggable boundary edges and window panning.
    - Fixed text wrapping on the measurement HUD by enforcing `whiteSpace: "nowrap"`, `flexShrink: 0`, and using compact engineering units (`formatTimeCompact`).
    - Added an intuitive "Zoom into Window" button (`<Search size={11} />`) that fits the selected measurement interval to 100% of the waveform viewport.
    - Added mouse-wheel and trackpad zooming and panning directly on the waveform canvas via `onWheel`.
  - [x] **Verification**: All 55 Rust workspace tests passing (`cargo test --workspace`) and frontend production build verified (`npm run build`).

- [x] **Phase 13.7: Precise Cursor-Tracking Wire & Net Hover Card Positioning - [P0]**
  - [x] **Fixed Broken Wire Hover Tooltip Positioning**:
    - Identified that `tooltipPos` only handled `hoveredNodeId` and `selectedNodeId`, completely omitting `hoveredEdgeId` and falling back to a static `{ x: 20, y: 50 }` window coordinate.
    - Implemented dedicated `edgeTooltipPos` calculating real-time cursor coordinates from `mousePos.x + 16, mousePos.y + 16` with viewport boundary clamping (`maxX = window.innerWidth - 270`, `maxY = window.innerHeight - 160`) to prevent off-screen clipping.
    - Updated wire hover tooltip component to bind directly to `edgeTooltipPos.x` and `edgeTooltipPos.y`.
  - [x] **Hover Priority Resolution & Selection Isolation**:
    - Refactored `activeHoverNode` and `activeHoverEdge` memoizers so that hovering an edge cleanly takes priority over any previously selected node (`hoveredEdgeId` yields `activeHoverNode = null`).
    - Removed `selectedNodeId` suppression from `activeHoverEdge`, ensuring wires can be hovered and inspected even while a cell in the schematic DAG is selected.
  - [x] **Canvas Mouse Leave Cleanup**:
    - Added `handleMouseLeave` to clear `hoveredNodeId` and `hoveredEdgeId` immediately when cursor leaves the canvas, preventing lingering frozen hover cards.
  - [x] **Verification**: All 55 Rust workspace tests passing (`cargo test --workspace`) and frontend production build verified (`npm run build`).

- [x] **Phase 13.6: Resizable De-Cramped Sidebar & Non-Wrapping File Tree Architecture - [P0]**
  - [x] **Spacious Resizable Sidebar with Draggable Splitter Handle**:
    - Expanded default sidebar width from cramped `228px` to spacious `285px` (configurable between `220px` and `500px`).
    - Added interactive `ResizableSplitter` handle between the Sidebar and the Center Simulation Workspace.
    - Double-clicking the splitter snaps sidebar width back to the default `285px`.
    - Persists user's preferred sidebar width in `localStorage` (`axiom_sidebar_width`).
    - Automatically updates dynamic editor width percent calculation based on live sidebar width.
  - [x] **De-Cramped Project Meta Card Header**:
    - Removed the redundant, wide `+ Create` text button and `Close` text button that squeezed `project.name` into `logi...`.
    - `project.name` now occupies the full width of the card with `13px` bold typography and clean icon.
    - Close project is a sleek, compact icon button (`<X size={14} />`) with tooltip.
  - [x] **Guaranteed Non-Wrapping Typography & File Trees**:
    - Enforced `whiteSpace: "nowrap"`, `overflow: "hidden"`, and `textOverflow: "ellipsis"` across all file item names and fileset headers.
    - File names (`logic_circuit.v`, `tb_logic_circuit.v`) now render completely without truncation.
    - Section headers (**Design Sources**, **Simulation Sources**, **Constraints**) and count chips render cleanly on a single row without wrapping.
  - [x] **Verification**: All 55 Rust workspace tests passing (`cargo test --workspace`) and frontend production build verified (`npm run build`).

- [x] **Phase 13.5: Toast Notification System, Unified Confirm Dialogs, 3-Dot Project Trashing & Fileset Architecture - [P0]**
  - [x] **Aerospace Toast Notification System (`toast.ts` & `ToastContainer.tsx`)**:
    - Built lightweight reactive singleton toast manager supporting `toast.success`, `toast.error`, `toast.info`, and `toast.warning`.
    - Floating dark-acrylic glassmorphism container fixed at top-right with smooth `toastSlideIn` CSS animation, auto-dismiss, manual dismiss, and distinct status icons.
    - Completely replaced all legacy browser `alert(...)` calls in the application with reactive toasts.
  - [x] **Unified Dark-Acrylic Confirm Dialog (`ConfirmModal.tsx`)**:
    - Created styled modal dialog subsystem with promise-based `confirmDialog(...)` helper replacing all native browser `confirm(...)` dialogs.
    - Supports `variant: "danger" | "primary" | "warning"`, customizable confirm/cancel button text, keyboard shortcuts (`Enter` to confirm, `Escape` to cancel), and auto-focus.
    - Completely eliminated `prompt()` and native `confirm()` across the entire codebase.
  - [x] **Main Page 3-Vertical-Dot Kebab Menu & Trashing Isolation**:
    - Completely removed "Move to Trash" from the in-project header menu (`ProjectDropdown.tsx`), keeping project header focused purely on design workflow.
    - On `WelcomeLaunchpad.tsx`, replaced exposed red trash buttons with a three-vertical-dot button (`MoreVertical`).
    - Clicking the 3-dot button opens a card dropdown menu with "Move to Trash" (with click-outside auto-close and event stop-propagation).
    - Trashing, restoring, and permanent deletion trigger the styled `confirmDialog` with toast confirmations.
  - [x] **Vivado Fileset Label Cleanup & Standardization**:
    - Standardized sidebar fileset headers across all 7 supported languages (`en`, `de`, `es`, `fr`, `ja`, `tr`, `zh`) to clean, unified titles: **Design Sources**, **Simulation Sources**, and **Constraints** (removing confusing `(sim_1)` and `(constrs_1)` suffixes).
    - Updated `AddSourceModal.tsx` to display clean primary labels with technical `sources_1`, `sim_1`, and `constrs_1` identifiers in secondary subtext.
  - [x] **Verification**: All 55 Rust workspace tests passing (`cargo test --workspace`) and frontend production build verified (`npm run build`).

- [x] **Phase 13.4: Lock-Safe Multi-Session Concurrency, Desktop Single-Instance Multi-Window & Welcome Ergonomics - [P0]**
  - [x] **Welcome Launchpad Ergonomics & Direct Project Card Clicking**:
    - Project cards are now directly clickable (`cursor: pointer`, opens project immediately on click).
    - Trash button stops click propagation (`e.stopPropagation()`) so clicking trash does not accidentally open the project.
    - Vertically centered the `FolderOpen` icon with text inside the Open button using standard `.btn .btn-primary` and flex alignment.
    - Updated trash tab action buttons to standard `.btn .btn-danger` and `.btn .btn-cyan`.
  - [x] **Consolidated Single GitHub Footer Mention**:
    - Removed GitHub links from desktop and mobile `Header.tsx` and the Launchpad hero section.
    - Positioned GitHub link exclusively once at the bottom footer of `WelcomeLaunchpad.tsx`.
  - [x] **Web Cross-Tab Lock Safety & Atomic Concurrency (`sessionSync.ts`)**:
    - Implemented `withLock` concurrency coordinator using the modern Web Locks API (`navigator.locks.request`) with an in-memory sequential promise queue fallback for unsupported environments.
    - Implemented `sessionBroadcaster` using `BroadcastChannel("axiom_session_sync")` with cross-window `storage` event fallback.
    - Wrapped all critical project registry mutations (`saveProjectRegistry`, `createAndPersistProject`, `trashProject`, `permanentDeleteProject`) with `withLock`.
    - Wrapped `BrowserIndexedDbFileSystem` file operations (`writeFile`, `deleteFile`, `rmdir`) with resource-scoped locks.
    - Subscribed `App.tsx` to `sessionBroadcaster` to auto-synchronize project lists and reload active project state when modified in other tabs/windows.
  - [x] **Desktop Single-Instance Multi-Window Architecture (`axiom-desktop`)**:
    - Integrated `tauri-plugin-single-instance = "2"` into `crates/desktop/Cargo.toml`.
    - Initialized single-instance plugin in `run_desktop_app()`: secondary process executions notify the primary Rust process to spawn another window via `WebviewWindowBuilder`, maintaining 1 single Rust process for multiple GUI windows.
    - Implemented `MultiEngineManager` in `crates/desktop/src/lib.rs` isolating in-RAM Cranelift JIT simulation sessions per window label.
    - Window close handler dynamically prunes destroyed window simulation engines from memory.
    - Added global thread-safe `FS_MUTEX: Mutex<()>` guarding host filesystem commands (`fs_read_file`, `fs_write_file`, `fs_remove_file`, `fs_list_dir`, `fs_create_dir`, `fs_exists`) against cross-window file race conditions.
  - [x] **Verification**: All 55 workspace tests passing (`cargo test --workspace`) and frontend production build verified (`npm run build`).

- [x] **Phase 13.3: Minimalist Header Controls & Open Source GitHub Integration - [P0]**
  - [x] **Flag-Only Language Selector**:
    - Removed `Globe` icon (`🌐`) and text ("English") from the language selector trigger button in `Header.tsx`.
    - Enhanced `Select.tsx` with `hideChevron`, `buttonClassName`, and `renderTrigger` properties.
    - Rendered the active language's flag emoji (e.g., `🇺🇸`, `🇹🇷`, `🇩🇪`) in a clean, standardized 28x28 square button matching all other header icon buttons.
    - Clicking the flag opens the full language dropdown with flags, native names, localized names, and language codes (`EN`, `TR`, etc.).
  - [x] **Single-Icon Omnibar / Search Button**:
    - Replaced the wide `[ 🔍 Omnibar ⌘K ]` button with a minimalist single-icon button `<Search size={14} />`.
    - Maintained accessible tooltip (`title="Omnibar (Ctrl+K / ⌘K)"`) and keyboard shortcut handlers.
  - [x] **Open Source GitHub Integration**:
    - Created reusable `GithubIcon` SVG component in `ui/src/components/ui/GithubIcon.tsx` and exported via `ui/src/components/ui/index.ts`.
    - Added GitHub repository icon button in `Header.tsx` (desktop and mobile) linked to `https://github.com/OguzhanUmutlu/axiom`.
    - Added Open Source GitHub collaboration badge, repository link card (`OguzhanUmutlu/axiom`), "Star & Collaborate" action, and documentation link in the Welcome Launchpad hero section.
    - Added an open-source collaboration note and PR invitation in the Welcome Launchpad footer.
  - [x] **Verification**: Built UI with 0 errors (`npm run build`) and verified all 55 Rust workspace tests passing (`cargo test --workspace`).

- [x] **Phase 13.2: Industry-Grade Project Lifecycle, Regex Naming & UI Alignment - [P0]**
  - [x] **Project Registry & Lifecycle Engine (`projectRegistry.ts`)**: Built persistent project registry tracking active vs trashed projects, with dual-runtime synchronization between `localStorage` and FileSystem `/projects/registry.json`.
  - [x] **Strict Regex Project & Directory Naming**: Enforced strict `[a-zA-Z0-9_.-]+` naming regex with live sanitization and folder preview (`/projects/{name}/`). Enforced matching ID and folder names with collision prevention.
  - [x] **Disallow Temporary Phantom Projects**: Eliminated unpersisted/transient projects. Creating any project requires user naming and immediately persists to the active FileSystem. Invalid or trashed `?project=` URL parameters clean up gracefully without fabricating phantom projects.
  - [x] **Trashing (Soft Delete) & Permanent Deletion (Hard Delete)**:
    - Soft delete: Moving a project to Trash hides it from active views, sets `isTrashed: true` and `trashedAt` timestamp, and cleanly unloads it if currently open.
    - Restoration: 1-click restoration of any trashed project back to Active state.
    - Hard delete: Permanently wipes the `/projects/{id}` directory from the FileSystem and removes it from registry and cache, with confirmation prompt.
    - Empty Trash: 1-click batch permanent deletion of all trashed projects.
  - [x] **"Create >" Template Actions**: Changed template card action button from "Open >" to "Create >" across all 7 supported languages. Clicking a template card opens `NewProjectModal` with that template pre-selected, prompting the user for a valid project name.
  - [x] **"Your Projects" Launchpad Management**: Added dedicated "Your Projects" view on the Welcome Launchpad with Active and Trash tabs, project metadata cards, Open/Trash actions, and Empty Trash controls.
  - [x] **Project Dropdown "Move to Trash" Action**: Added a confirmation-guarded "Move to Trash" action to the top-left Project Header Menu.
  - [x] **Sources Panel Empty State Matching Netlist**: Redesigned the Sources tab empty state in `ProjectManager.tsx` to match the exact design of the Netlist panel (quiet centered icon, title, description, and no button).
  - [x] **Header Empty State De-Cluttering**: In `Header.tsx`, removed the redundant and cramped `New Project` button next to `No Project Open`, keeping the header clean and focused.
  - [x] **Verification**: Verified zero TypeScript/Vite bundling errors (`npm --prefix ui run build` passed) and 55 / 55 passing Rust workspace tests (`cargo test --workspace`).

- [x] **Phase 13.1: Left Panel Empty State Sanitization & Redundancy Removal - [P0]**
  - [x] **Left Panel Starter Templates Removal**: Removed redundant vertical starter templates list from the "No Project Open" empty state in `ProjectManager.tsx`. When no project is open, the left panel presents a focused, clean "No Project Open" card with "+ Create New Project", directing template discovery exclusively to the center Welcome Launchpad.
  - [x] **Unused Exports & Types Cleanup**: Cleaned up unused imports (`PROJECT_TEMPLATES`, `createProjectFromTemplate`, `Sparkles`) and aliased optional props for strict TypeScript compliance.
  - [x] **Verification**: Verified zero TypeScript/Vite bundling errors (`npm --prefix ui run build` passed) and 55 / 55 passing Rust workspace tests.

- [x] **Phase 13.0: Professional Project Lifecycle, Dual-Runtime FileSystem & Dynamic Canvas Centering - [P0]**
  - [x] **Header State Partitioning**: Completely eliminated premature simulation controls when `project === null` on the Welcome screen. Header now cleanly partitions: Welcome mode renders brand, version tag (`v0.1.0-jit`), subtle "No Project Open" badge, and language selector. Active project mode renders the full simulation control ribbon (`Run`, `Pause`, `+1 ns`, `+100 ps`, `Step δ`, `Reset`), sim time, and PDN telemetry meters.
  - [x] **Vivado-Grade Project Header Menu (`ProjectDropdown.tsx`)**: Replaced the tiny ghost `X` close button with an aerospace-grade Project Menu badge in the top-left showing project identity, target FPGA chip, active top module `[TOP]`, file count, manual save (`Ctrl+S` with instant badge), project bundle export (`.json`), Add Source modal trigger, New Project wizard trigger, and a clean Close Project lifecycle.
  - [x] **Template Lifecycle Cleanup**: Eliminated the confusing mid-project "Load Template" section from `ProjectManager.tsx`. Templates are strictly project initialization blueprints residing on the Welcome Launchpad and inside `NewProjectModal.tsx`.
  - [x] **Dynamic Midpoint Camera Anchoring & Continuous Canvas Resizing**: Overhauled `ResizeObserver` in `SchematicViewer.tsx` to continuously update canvas resolution (`canvas.width = Math.round(newW * dpr)`) and mathematically lock the world-space camera midpoint to the visualizer pane center ($\Delta \text{offsetX} = \Delta W / 2$, $\Delta \text{offsetY} = \Delta H / 2$) during middle splitter dragging, completely eliminating all horizontal squishing while keeping zoom scale 100% constant.
  - [x] **Dual-Runtime FileSystem Abstraction (`ui/src/engine/fs/`)**: Built abstract `FileSystem` class, `BrowserIndexedDbFileSystem` using `idb` for in-browser virtual VFS + `localStorage` caching, and `TauriIpcFileSystem` using `@tauri-apps/api/core` `invoke` to communicate with native Rust backend commands (`fs_read_file`, `fs_write_file`, `fs_remove_file`, `fs_list_dir`, `fs_create_dir`, `fs_exists`) in `crates/desktop/src/lib.rs`.
  - [x] **Immediate Auto-Saving & Shortcuts**: Integrated automatic persistence on project creation, file editing, source additions, and global `Ctrl+S` / `Cmd+S` keyboard shortcuts.
  - [x] **Verification**: Verified zero TypeScript/Vite bundling errors (`npm --prefix ui run build` passed) and 55 / 55 passing Rust workspace tests (`cargo test --workspace`).

- [x] **Phase 12.15: Zero-Turn Datapath Pin Alignment & Multi-Layer Destination Stepping - [P0]**
  - [x] **Zero-Turn Pin Alignment (`fixedY`)**: Added `fixedY?: number` to `SchematicNode` and aligned connected pin heights ($Y_{\text{out}} = Y_{\text{in}}$) across the entire circuit. Completely eliminated micro-jogs for 5 major connections ($A \to \text{inv1}$, $B \to \text{and1.in2}$, $\text{and1.out} \to \text{and2.in1}$, $\text{and2.out} \to \text{or1.in1}$, $\text{or1.out} \to F$), rendering them as 100% straight horizontal lines with **zero turning movements**.
  - [x] **Multi-Layer Channel Destination Stepping**: Enhanced `routeOrthogonalEdge` so wires traversing multiple layers ($dx \ge 150\text{px}$, such as $C \to \text{and2}$ and $\text{inv2} \to \text{or1}$) maintain horizontal momentum across intermediate layers and execute their vertical jog in the dedicated open channel immediately before the destination ($dstX - 28$), preventing wire bends from cluttering intermediate gates.
  - [x] **Spacious Inter-Layer Padding**: Increased `layerSpacingX` from 72px to 92px in `layoutAndRouteGraph`, providing wide inter-layer corridors for vertical drops with generous clearance around all gates and labels.
  - [x] **Equidistant Primary Input Ports**: Positioned input ports $A$, $B$, and $C$ at exactly 75px vertical intervals ($Y = 47, 122, 197$), providing visual balance and clarity.
  - [x] **Continuous Documentation Currency**: Updated `analysis/09_schematic_dag_and_synthesis_viewer.md` and `GEMINI.md`.
  - [x] **Verification**: Verified zero TypeScript/Vite bundling errors (`npm --prefix ui run build` passed) and 55 / 55 passing Rust workspace tests (`cargo test --workspace`).

- [x] **Phase 12.14: Vivado-Grade Collision-Free Wire Routing, Datapath Alignment & Text Knockout Plates - [P0]**
  - [x] **Root Cause Analysis & Vivado Reverse Engineering**: Identified that the wire from input $B$ to gate `and1` in `logic_circuit` passed horizontally through the instance label `inv2` because Layer 1's 2 nodes were centered vertically as a cluster right across Row 1, and `routeOrthogonalEdge` lacked intermediate obstacle awareness.
  - [x] **Vivado-Grade Datapath Grid Alignment (`gridRow`)**: Added `gridRow?: number` to `SchematicNode` interface and updated `layoutAndRouteGraph` in `ui/src/engine/schematicModel.ts` to support both explicit grid datapath row positioning and automatic vertical centering.
  - [x] **Harmonious `logic_circuit` Datapath Grid Layout**: Assigned explicit datapath rows matching Vivado's standard layout:
    - Primary datapath: Input A (Row 0) $\to$ `inv1` (Row 0) $\to$ `and1` (Row 0.55) $\to$ `and2` (Row 0.75) $\to$ `or1` (Row 0.95) $\to$ Output F (Row 1.15).
    - Input B (Row 1): Runs horizontally through an empty 130px open corridor in Layer 1 directly to `and1.in2` with zero obstacles.
    - Input C (Row 2): Runs horizontally through Layer 1 & 2 before stepping up to `and2.in2`.
    - Gate `inv2` (Row 2.8): Placed at the bottom datapath row, fed from B via the first inter-layer channel and feeding `or1.in2` along the bottom corridor.
  - [x] **Obstacle-Aware Orthogonal Channel Routing (`routeOrthogonalEdge`)**:
    - Pre-calculates `KeepOutBox` clearance bounding boxes for all graph nodes (including $y - 18$ margin above nodes to protect instance labels).
    - Added horizontal and vertical segment collision detection (`getHCollision`, `getVCollision`).
    - When a vertical trunk or horizontal segment intersects an intermediate node, router automatically shifts the trunk to an open channel or jogs around the obstacle.
  - [x] **Solid Background Text Knockout Plates**: In `ui/src/components/SchematicViewer.tsx`, added solid `#0c1017` protective plates behind instance labels (`inv1`, `inv2`, `and1`, `or1`) using `ctx.fillRect(textX - w/2 - 4, textY - 10, w + 8, 14)`, completely eliminating wire/text collisions.
  - [x] **Continuous Documentation Currency**: Updated `analysis/09_schematic_dag_and_synthesis_viewer.md` and `GEMINI.md`.
  - [x] **Verification**: Verified zero TypeScript/Vite bundling errors (`npm --prefix ui run build` passed) and 55 / 55 passing Rust workspace tests (`cargo test --workspace`).

- [x] **Phase 12.13: Schematic Ergonomics: Clean Default State & Auto-Fit Zoom Framing - [P0]**
  - [x] Configured `showLiveValues` to `false` by default in `SchematicViewer.tsx` for clean, clutter-free gate-level reading.
  - [x] Re-calibrated layer horizontal pitch to 64px in `schematicModel.ts` (eliminating long empty horizontal wire stretches).
  - [x] Calculated tight, exact geometric bounding boxes across all cells and routed wire points in `schematicModel.ts`.
  - [x] Upgraded `fitToScreen` algorithm in `SchematicViewer.tsx`: symmetrically centers the circuit horizontally and vertically on screen and zooms in up to 1.35x to fill the viewport cleanly.
  - [x] Added `ResizeObserver` on `containerRef` to ensure auto-fit executes once real layout dimensions are present.
  - [x] Updated minimap coordinate framing to use tight bounds relative offsets.
  - [x] Updated living documentation in `analysis/09_schematic_dag_and_synthesis_viewer.md` and `analysis/uiux/04_complex_eda_visualization_ergonomics.md`.
  - [x] Verified build (`npm run build` - 0 errors) and test suite (`cargo test` - 55/55 passed).

- [x] **Phase 12.12: UI Materials Componentization & Custom Aerospace Dropdown Architecture - [P0]**
  - [x] Upgraded `ui/src/components/ui/Select.tsx` with density sizes (`xs`, `sm`, `md`), alignment (`left`, `right`), option groups (`groups`), keyboard navigation, and dark acrylic popover.
  - [x] Upgraded `ui/src/components/ui/Input.tsx` with density sizes (`xs`, `sm`, `md`) and clearable button support.
  - [x] Added `.custom-scrollbar` and custom dropdown popover animations in `ui/src/styles/theme.css`.
  - [x] Replaced native `<select>` in `Header.tsx` (desktop & mobile language selectors) with `<Select size="xs" />`.
  - [x] Replaced native `<select>` in `VirtualLabRack.tsx` (DIP switch & 7-seg port selectors) with `<Select size="xs" />` using grouped options.
  - [x] Replaced native `<select>` in `StimulusPainterModal.tsx` (clock frequency selector) with `<Select size="sm" />`.
  - [x] Updated living documentation in `analysis/uiux/01_design_system_and_theming.md` and `analysis/08_desktop_and_web_ui.md`.
  - [x] Formally integrated primitive abstraction & componentization into UI/UX Skillset:
    - `analysis/uiux/SKILL.md`: Added Principle 6 ("Strict Primitive Abstraction & Zero Native Bleed") and Pillar 7 ("Primitive Componentization & Design System Encapsulation").
    - `analysis/uiux/05_comprehensive_critique_and_audit_rubrics.md`: Expanded to 60-Point Audit Checklist (Category 7: Primitive Componentization & Zero Native Bleed, Items 51–60) and added Anti-Pattern 8 ("The Leaky Native Control") and Anti-Pattern 9 ("Concrete Primitive Sprawl").
    - `analysis/uiux/06_axiom_studio_audit_and_design_critique.md`: Added Section 2.11 for UI Primitives and marked port group selector issue as resolved.
    - `analysis/uiux/03_interaction_design_and_haptics.md`: Added Section 7 detailing popover mechanics, keyboard parity, density tiers, and clearable actions.
    - `analysis/analysis.md`: Updated master blueprint tree to register the full `analysis/uiux/` architectural suite.
  - [x] Verified build via TypeScript compile (`npm run build` - 0 errors) and Rust workspace test suite (`cargo test` - 55/55 passed).

- [x] **Phase 12.11: White Katana Slash Cursor Effect in Monaco HDL Editor - [P0]**
  - [x] Master i18n translations across 7 locales (`editor.katanaSlash`, `editor.katanaSlashTooltip`).
  - [x] Canvas overlay component (`ui/src/components/KatanaCursorOverlay.tsx`) with high-DPI retina support and self-sleeping RAF loop.
  - [x] Frame-rate independent exponential follower physics ($\lambda \approx 18\text{ s}^{-1}$) for a smooth $\sim 90\text{ms}$ delay.
  - [x] Tapered katana blade polygon geometry with radiant white core, silver-white glow, and razor-sharp cutting edge.
  - [x] *Sori* curved slash strike arcs on line jumps ($d > 12\text{px}$) with rapid 180ms exponential dissolve and microscopic glints.
  - [x] `Swords` toggle button in `HdlEditor.tsx` actions strip with `localStorage` persistence.
  - [x] Living documentation updates in `analysis/08_desktop_and_web_ui.md` and `analysis/uiux/03_interaction_design_and_haptics.md`.
  - [x] Verification via TypeScript build (0 errors) and Rust test suite (55/55 passed).

- [x] **Phase 12.10: Ergonomic Studio De-Cramping, Unified Theme System, Precision Iconography & Global Multi-Language (i18n) Engine - [P0]**
  - [x] **Unified Theme Management & Subtle Transitions (`theme.css`)**:
    - Centralized reusable button variants (`.btn`, `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-danger`, `.btn-success`, `.btn-cyan`, `.btn-icon`).
    - Standardized 150ms cubic-bezier transitions, subtle `:active` scale (`scale(0.97)`), and `:focus-visible` high-contrast outline rings.
    - Centralized badge/chip classes (`.badge-cyan`, `.badge-emerald`, `.badge-rose`, `.badge-amber`, `.badge-purple`).
    - Standardized `.axiom-card` with border transitions and `.mono-num` (`tabular-nums lining-nums`).
    - Refined spatial density tokens (header 42px, tabs 30px, tree items 28px) eliminating cramping.
  - [x] **Aerospace-Grade Precision Vector Logo (`logo.svg`)**:
    - Designed modern faceted delta/lambda chevron mark in electric cyan (`#00f2fe`) and cobalt blue (`#2563eb`).
    - Crisp vector rendering optimized across 16px, 18px, 24px, 32px, and 512px viewports in both `ui/public/logo.svg` and `docs/public/logo.svg`.
  - [x] **Global Multi-Language (i18n) Engine (`ui/src/i18n/`)**:
    - Master type-safe dictionary schema (`types.ts`) guaranteeing 100% compile-time key safety.
    - Master English locale catalog (`locales/en.ts`) containing all UI strings.
    - React context (`i18nContext.tsx`) with browser auto-detection (`navigator.language`) and localStorage persistence.
    - Enhanced dual-mode `TFunction`: supports both callable path lookup `t("category.key")` and strongly-typed direct property access `t.category.key`.
    - Wired throughout all studio components: `Header`, `Sidebar`, `ProjectManager`, `HdlEditor`, `SchematicViewer`, `VirtualLabRack`, `WaveformViewer`, `TimingRadarViewer`, `UnifiedBottomDock`, `MobileDrawer`, `MobileBottomBar`, `NewProjectModal`, `AddSourceModal`, `OmnibarModal`, `WelcomeLaunchpad`.
    - Full multi-language translations across 7 production locales:
      - English (`en.ts` - Master)
      - Turkish (`tr.ts` - Türkçe natively translated with standard digital electronics terms)
      - German (`de.ts` - Deutsch)
      - Spanish (`es.ts` - Español)
      - French (`fr.ts` - Français)
      - Japanese (`ja.ts` - 日本語)
      - Simplified Chinese (`zh.ts` - 简体中文)
  - [x] **Component De-Cramping & UI Refinement**:
    - Header: Relaxed spacing, clear dividers, language dropdown selector, responsive collapse below 1200px.
    - Sidebar: 6px tree padding, netlist search inline `✕` clear button.
    - HdlEditor: Tab height 30px, visual "dirty" dot indicator for unsaved changes.
    - VirtualLabRack: Grouped signal selector dropdown, mobile DIP switch hitboxes >= 40px.
    - UnifiedBottomDock: Pulsing error badge when collapsed, 28px status strip.
    - SchematicViewer: Toolbar de-cramping and "Hide Global Clock/Reset Nets" filter.
  - [x] **Verification & Workspace Tests**:
    - Built UI bundle cleanly with Vite and `tsc` (0 type errors, exit code 0).
    - Passed all 55/55 Cargo workspace unit, integration, benchmark, and conformance tests.
  - [x] **Continuous Documentation Synchronization**:
    - Kept `analysis/08_desktop_and_web_ui.md` and `analysis/uiux/01_design_system_and_theming.md` architectural files continuously updated.

- [x] **Phase 12.9: Full Mobile Studio Support & Off-Canvas Drawer Architecture - [P0]**
  - [x] **Responsive Mobile Viewport Engine (`App.tsx`)**:
    - Automatic mobile detection (`window.innerWidth <= 768`) with responsive resize listener.
    - Completely disables desktop splitters, sidebars, and multi-pane crowding on mobile viewports.
  - [x] **Off-Canvas Left Drawer (`MobileDrawer.tsx`)**:
    - Smooth hardware-accelerated drawer sliding from the left (`translateX(-100%)` to `translateX(0)`) with 6px blurred backdrop.
    - Studio Panels selector (view 1 at once: HDL Code Editor, Schematic DAG, Virtual Lab Rack, Waveforms Viewer, Timing & Energy, Console & REPL).
    - Authentic Vivado file sets explorer (`sources_1`, `sim_1`, `constrs_1`) with active `[TOP]` module badge, `+ Add Source`, and `Close Project`.
    - Integrated Simulation Clock display and quick stepping/run controls.
    - Automatic auto-collapse to the left upon panel, file, or template selection.
  - [x] **Single-Panel Full-Screen Layout (`App.tsx`)**:
    - Each tool occupies 100% of the viewport width and height with zero horizontal scrolling or clipping.
    - Full-fidelity rendering for Monaco HDL Editor, GPU/Canvas Schematic DAG, Virtual Lab Rack with Truth Table HUD, Waveforms Viewer, and Console & REPL dock.
  - [x] **Tactile Mobile Bottom Navigation Bar (`MobileBottomBar.tsx`)**:
    - Fixed 56px bottom bar with safe-area notch padding.
    - 5 thumb-friendly tabs: `Code` (blue), `Schematic` (cyan), `Lab` (amber), `Waves` (emerald), `Console` (purple).
    - Dynamic notification badges for active LspDiagnostics and zero-time glitches.
  - [x] **In-Depth Browser QA Mobile Verification**:
    - Validated with browser subagent under iPhone mobile emulation (`390px × 844px`).
    - Captured and verified 6 screenshots: `28_mobile_initial_view.png`, `29_mobile_drawer_open.png`, `30_mobile_schematic_panel.png`, `31_mobile_virtual_lab_panel.png`, `32_mobile_editor_panel.png`, `33_mobile_dock_panel.png`.
    - Confirmed 0px horizontal overflow (`scrollWidth == clientWidth == 390px`).

- [x] **Phase 12.8: Default Combinational Logic Circuit (A, B, C → F) & Spacious Dual-Pane Studio - [P0]**
  - [x] **Default Combinational Logic Hardware System (`sampleDesigns.ts`, `projectModel.ts`)**:
    - Modeled gate-level equation: `F = ((~A & B) & C) | ~B` with intermediate nets `w1 = ~A; w2 = w1 & B; w3 = w2 & C; w4 = ~B; F = w3 | w4;`.
    - Authentic Verilog source (`logic_circuit.v`), SystemVerilog self-checking stimulus testbench (`tb_logic_circuit.sv`), and physical XDC constraints (`timing.xdc`).
    - Configured as the **#1 featured starter template** and primary default system across Axiom Launchpad and Project Manager.
  - [x] **Gate-Level Schematic DAG Synthesis (`schematicModel.ts`)**:
    - Synthesizes 9 cells and 9 nets: primary input ports (`A`, `B`, `C`), inverters (`inv1`, `inv2`), 2-input AND gates (`and1`, `and2`), 2-input OR gate (`or1`), and primary output port (`F`).
    - Live net labeling for intermediate wires `w1`..`w4` and output `F`.
  - [x] **Interactive Virtual Lab Bay & Truth Table HUD (`VirtualLabRack.tsx`)**:
    - Tactile input toggle switches for `Input A (SW0)`, `Input B (SW1)`, and `Input C (SW2)` with high/low state pills, glowing LED indicators, and batch controls (`Cycle +1`, `All 0s`, `All 1s`).
    - Intermediate net probe badges with live operator chips (`NOT w1 = ~A`, `AND w2 = w1 & B`, `AND w3 = w2 & C`, `NOT w4 = ~B`).
    - Primary output F large circular glowing LED indicator with live active state and pin assignment `Pin H17 • LD0`.
    - Live 8-state Truth Table HUD (`000` through `111`) displaying `A`, `B`, `C`, `w1`..`w4`, and `F` with real-time active row highlighting.
  - [x] **Spacious Dual-Pane Studio & De-Cramping (`App.tsx`)**:
    - Redesigned Split Studio from cramped quad-split into high-productivity Dual-Pane Studio:
      - Left Pane: Monaco HDL Editor with active `[TOP]` module tag, breadcrumbs, line numbers, and resizable width handle.
      - Right Pane: Full-height, full-width Visualizer container with ergonomic top switcher tabs (`⚡ Schematic DAG`, `🎛 Virtual Lab`, `📈 Waveforms`, `⏱ Timing & Energy`), optional `[+ Waveforms]` stacked toggle, and 1-click Maximize button.
      - Each tool gets 100% of the right pane width and height, eliminating all clipping and cramped controls.
  - [x] **Typography & Touch Target Scaling (`theme.css`, `Header.tsx`, `Sidebar.tsx`, `ProjectManager.tsx`, `UnifiedBottomDock.tsx`)**:
    - Upgraded base font to 14px (line-height 1.5) in `theme.css`.
    - Monaco editor font size scaled to 14px (`lineHeight: 22`).
    - Header height increased to 52px, simulation stepping buttons scaled to 12.5px–13px with 15px icons.
    - Sidebar width expanded to 280px, project files and netlist items scaled to 12.5px font with comfortable 6px padding.
    - Dock collapsed status bar height increased to 32px, tab headers to 38px with 12px font and clear badges.

- [x] **Phase 12.7: Vivado Welcome Launchpad & Clean No-Project Standpoint - [P0]**
  - [x] **No-Project Open Standpoint (`projectModel.ts`, `App.tsx`)**:
    - Default startup begins from a clean "No Project Open" standpoint (returns `null` instead of forcing hardcoded default project).
    - Added `clearSavedProject()` and `saveProjectToStorage(null)` to cleanly reset session.
  - [x] **Vivado-Style Welcome Launchpad (`WelcomeLaunchpad.tsx`)**:
    - Aerospace-grade hero banner (`Axiom HDL Studio v0.1.0-jit`) with subtitle and architecture summary.
    - Two primary interactive cards: "Create New Project" (launching 3-step Vivado Project Wizard) and "Open Project from File" (one-click JSON project bundle importer).
    - 7-item interactive Quick Start Hardware Templates grid with FPGA target silicon badges and one-click "Open" buttons.
    - Engineering pillars footer highlighting In-RAM Cranelift JIT, Stratified Delta Stepping, and Physics-Informed PDN telemetry.
  - [x] **Context-Aware Header & Sidebar (`Header.tsx`, `Sidebar.tsx`, `ProjectManager.tsx`)**:
    - Header displays "No Project Open" tag, "+ New Project" action button, and cleanly disabled simulation stepping buttons (`Run Free`, `+1 ns`, `Step δ`).
    - Dedicated "Close" button in header and sidebar allowing one-click return to the Welcome Launchpad.
    - Sidebar renders a dedicated empty state card with "+ Create New Project" and template quick-loader list.
    - Netlist hierarchy displays "No Netlist Available" placeholder when no project is loaded.
  - [x] **One-Click Project Loading & Multi-File Studio Transition**:
    - Launching a project immediately initializes the file sets (`sources_1`, `sim_1`, `constrs_1`), triggers In-RAM JIT elaboration, and smoothly transitions to Split Studio (Monaco Editor, Waveforms, Schematic DAG, Virtual Lab).

- [x] **Phase 12.6: In-RAM Verilog/SystemVerilog LSP, Static Analysis Linter & Monaco Editor Integration - [P0]**
  - [x] **In-RAM LSP & Static Analysis Linter Crate (`crates/lsp`, `axiom-lsp`)**:
    - High-performance zero-copy linter running directly against AST in milliseconds.
    - 10 static analysis rules: syntax error mapping (`AXIOM_E001`), blocking assignment in clocked sequential blocks (`AXIOM_W001`), non-blocking assignment in combinational blocks (`AXIOM_W002`), undriven nets (`AXIOM_W003`), unused signals (`AXIOM_W004`), multi-driver net contention (`AXIOM_E002`), latch inference from incomplete if (`AXIOM_W006`), missing case default (`AXIOM_W007`), width mismatches & bit truncation (`AXIOM_W008`).
    - Standard JSON-RPC Language Server Protocol 3.17 stdio server (`LspServer`).
    - CLI commands `axiom lsp` (stdio JSON-RPC daemon) and `axiom lint <FILE>` (colorized CLI diagnostic report).
  - [x] **WebAssembly LSP Bindings (`crates/wasm`)**:
    - Exported `lint`, `hover`, and `complete` to JavaScript/TypeScript with full WebAssembly support with zero backend/network overhead.
  - [x] **Monaco Editor HDL Integration (`HdlEditor.tsx`, `monacoVerilog.ts`)**:
    - Replaced simple textarea with Monaco Editor (`@monaco-editor/react`).
    - Custom Monarch Verilog/SystemVerilog tokenizer with dedicated IEEE 1800 syntax tokens, directives, operators, and radix literals.
    - Custom engineering dark palette (`axiom-dark`) with sky-cyan keywords, pink directives, purple literals, and emerald strings.
    - Real-time debounced squiggly marker underlines (`monaco.editor.setModelMarkers`) and status pill (`Clean` / `X Warnings` / `X Errors`).
    - Markdown hover provider (`VerilogHover`) displaying port directions, data types, declaration coordinates, and enclosing scopes.
    - Autocompletion provider (`VerilogCompletion`) with templates, snippets (`always_ff`, `case`, `module`), and in-scope AST signals.
  - [x] **Problems & Linter Dock Panel (`UnifiedBottomDock.tsx`)**:
    - "Problems & Linter" dock tab displaying total issues, errors, warnings, and hints.
    - Interactive diagnostic cards with rule codes, messages, help tips, and clickable line/column navigation that jumps directly to the problem in Monaco.
    - Clean AST state illustration with zero issues detected.

- [x] **Phase 12.5: Vivado Project Management Architecture & De-Cramped Studio Layout - [P0]**
  - [x] **Vivado-Style Project Model & File Sets (`projectModel.ts`)**:
    - Project file sets structure: Design Sources (`sources_1`), Simulation Sources (`sim_1`), Constraints (`constrs_1`).
    - File format classification (`verilog`, `systemverilog`, `xdc`), active top-module designation (`isTop`), and target FPGA silicon device selection (`FPGA_TARGET_DEVICES`: Artix-7, Zynq-7000, Kintex-7, Kintex UltraScale+, Axiom Virtual Silicon).
    - 7 pre-configured multi-file Vivado project templates (RV32I RISC-V SoC, Full-Duplex UART, SPI Master, PWM Inverter, 8-Bit ALU, Glitch Counter, Empty RTL).
    - Multi-file source bundling (`bundleProjectSources`) for monolithic in-RAM JIT elaboration.
  - [x] **Vivado Project Explorer (`ProjectManager.tsx`)**:
    - Project header displaying project name, target device badge (e.g. `Artix-7 xc7a35t`), and active `[Top]` module.
    - Expandable file set folders (`sources_1`, `sim_1`, `constrs_1`) with file icons, cyan `[TOP]` badges, and 1-click "Set as Top" star action.
    - Actions toolbar: `+ Add Sources`, `+ New Project`, and `Export Project JSON`.
    - Project templates quick-loader section with one-click project template switching.
  - [x] **Vivado Project Creation & Source Dialogs**:
    - **New Project Wizard Modal (`NewProjectModal.tsx`)**: Step 1 project name, Step 2 target FPGA silicon selector, Step 3 starter template cards.
    - **Add Source to Vivado Project Dialog (`AddSourceModal.tsx`)**: Target file set picker (`sources_1`, `sim_1`, `constrs_1`), file extension validation (`.v`, `.sv`, `.xdc`), and starter templates (Clocked RTL module, Verilog testbench skeleton, XDC constraints).
  - [x] **Unified Dockable Bottom Drawer (`UnifiedBottomDock.tsx`)**:
    - Replaced statically stacked Telemetry + Console (300px overhead) with a unified, tabbed bottom drawer (`>_ Console & REPL`, `⚡ Power & Telemetry`, `⚠ Glitches & Hazards`, `⏱ Timing Slack`).
    - Collapsible to a 28px sleek status bar displaying live simulation time, rail voltage, instant power, and quick jump buttons, liberating ~250px of vertical space.
    - Resizable height with smooth dragging handle and full Maximize `⛶` / Restore toggle.
  - [x] **Collapsible Sidebar Strip (`Sidebar.tsx`)**:
    - Collapses from 260px down to an ultra-compact 38px icon strip, liberating 222px of horizontal screen width for code and visualizers.
  - [x] **Multi-Tab HDL Editor & Panel Maximization (`HdlEditor.tsx`, `App.tsx`)**:
    - Open file tab strip with active tab highlight, `[TOP]` module tag, close button (`✕`), and quick `+` add source button.
    - Hierarchical breadcrumb navigation (`project > sources_1 > file.v > module`).
    - 1-click panel maximize/restore button (`⛶` / `Minimize2`) across Editor, Waveforms, Schematic DAG, and Virtual Lab.

- [x] **Phase 12.4: Studio Design Evolution — Dynamic Designs, Resizable Architecture & Autonomous Stimulus - [P0]**
  - [x] **7 Production-Grade Dynamic Systems (`sampleDesigns.ts`)**:
    - **32-Bit RISC-V Mini Core Datapath**: RV32I datapath with Program Counter, embedded instruction ROM, 8x32-bit dual-read register file, and single-cycle ALU.
    - **Full-Duplex UART Transceiver**: Configurable baud clock generator, 8-N-1 framing, start/stop bit validation, and shift registers.
    - **SPI Master Controller (Modes 0–3)**: Selectable CPOL/CPHA clock polarity/phase, active-low chip select, and 8-bit full-duplex transfers.
    - **PWM Generator & Power Modulator**: Free-running period counter, duty comparator, and dead-time insertion for half-bridge shoot-through protection.
    - **8-Bit Arithmetic Logic Unit (ALU)**: Multi-function datapath with carry/zero flags.
    - **Synchronous Counter with Glitch Hazards**: Asymmetric path delays exhibiting zero-time delta-cycle hazard tracking.
    - **Hierarchical SoC Subsystem**: Frequency divider and power rail modeling.
  - [x] **Zero-Dependency Draggable Resizable Splitter (`ResizableSplitter.tsx`)**:
    - Smooth horizontal and vertical resizing with active dragging feedback and double-click reset.
    - Integrated across all studio view modes: HDL Editor $\leftrightarrow$ Visualizers, Waveforms $\leftrightarrow$ Schematics/Lab, and Schematics $\leftrightarrow$ Virtual Lab.
    - Quick layout preset buttons in the tab bar: **Balanced (33/67)**, **Code Focus (52/48)**, and **Visual Focus (20/80)**.
  - [x] **Autonomous Multi-Domain Stimulus Engine (`engineBridge.ts`)**:
    - Embedded state machines driving realistic physical transitions: autonomous UART serial frames, SPI bus handshakes, PWM switching ramps, and RISC-V fetch-decode-execute cycles.
    - Operates seamlessly across both in-browser WebAssembly simulation and desktop native IPC.
  - [x] **Protocol & Core Customized Virtual Lab Racks (`VirtualLabRack.tsx`)**:
    - **UART Lab**: ASCII character transmitter with Send Trigger, live RX data hex/ASCII display, and baud status flags.
    - **SPI Lab**: Byte injector, CPOL/CPHA mode selector, chip select monitor, and transfer speed gauge.
    - **PWM Lab**: Interactive duty cycle slider (0–100%), dead-time adjustment (0–15 clock cycles), and complimentary gate drive monitors (Gate High / Gate Low).
    - **RISC-V Lab**: Live 32-bit register file explorer (x0–x7 in hex and decimal), Single-Step instruction execution button, and reset strobe.
    - **Standard Lab**: 8-bit DIP switch bank, tactile pushbuttons, pulse generator, and authentic 7-segment LED display.
  - [x] **Categorized Fixture Explorer & Netlist Filter (`Sidebar.tsx`)**:
    - Filter pills for Cores, Protocols, Power, and Standard designs with color-coded badges and instant fuzzy search.
    - Netlist filter bar in the Scope Hierarchy tab for rapid signal lookup in complex designs.
  - [x] **Physics-Informed Analog Power Dial & PDN Status (`TelemetryViewer.tsx`)**:
    - Sweeping circular analog meter with color gradient arc for instantaneous dynamic power ($P_{\text{dynamic}}$).
    - Switching activity toggle rate ($\alpha$) gauge and real-time PDN rail voltage sag health alerts.

- [x] **Phase 12.3: Dual-Runtime Simulation Architecture (In-Browser WebAssembly & Native Tauri v2 IPC) - [P0]**
  - [x] **WebAssembly Simulation Kernel (`crates/wasm`)**: Compiled HDL parser, elaborator, 4-state arena (`SimStateArena`), portable evaluator (`PortableEvaluator`), stratified event scheduler, and silicon telemetry engine directly to `wasm32-unknown-unknown` (322 KB, ~102 KB gzipped).
  - [x] **Feature-Gated Cranelift JIT**: Gated Cranelift virtual-memory code generation to `not(target_arch = "wasm32")`, enabling clean compilation across both WebAssembly and native targets with zero host-OS memory protection dependencies.
  - [x] **Tauri v2 Native Desktop Integration (`crates/desktop`)**: Configured native Tauri v2 desktop application (`axiom-desktop`) with high-speed zero-copy IPC handlers (`compile_design`, `step_time`, `step_delta`, `force_signal`, `export_vcd`, `export_saif`) running native Cranelift JIT machine code compilation in RAM.
  - [x] **Universal Dual-Runtime Router (`ui/src/engine/engineBridge.ts`)**: Auto-detects runtime environment (`isTauriRuntime()`); delegates to Tauri native IPC when running on desktop and client-side WebAssembly when running in-browser, with seamless state updates into waveforms, schematic DAG, and telemetry visualizers.
  - [x] **Web Studio Website Deployment**: Integrated `base: './'` asset bundling, deployed interactive Web Studio live to `https://axiom.aerovex.net/studio/`, and linked directly from documentation navbar and homepage hero button (`⚡ Launch Web Studio`).

  - [x] **Zstd In-Binary UI Compression**: Compressed React 19 UI assets (`ui/dist/`) with Zstandard level 19 (~90 KB) and embedded them directly into the `axiom` binary at compile time via `crates/cli/build.rs`.
  - [x] **Embedded In-RAM GUI Server (`gui_server.rs`)**: Zero-async HTTP server (`tiny_http`) serving decompressed UI assets from RAM and providing live simulation engine REST API (`/api/compile`, `/api/step_time`, `/api/step_delta`, `/api/force`, `/api/vcd`, `/api/saif`, `/api/status`) with automatic browser launching (`axiom gui`).
  - [x] **Desktop Application & System Search Integration**:
    - Linux: Created Freedesktop `~/.local/share/applications/axiom.desktop` and installed high-res 512x512 icon in `~/.local/share/icons/hicolor/512x512/apps/axiom.png` for GNOME/KDE/Rofi indexing.
    - macOS: Created `~/Applications/Axiom.app` bundle with `Info.plist`, launcher script, and high-res icon for Spotlight, Raycast, and Alfred.
    - Windows: Created Start Menu and Desktop `.lnk` shortcuts with custom `axiom.ico` for Windows Search.
  - [x] **Universal Pre-Built Distribution (Aerovex CDN & GitHub Releases)**: Automated instant sub-2s installations from `https://axiom.aerovex.net/dist/` and published official `v0.1.0` release assets.
  - [x] **Multi-Architecture Release Workflow**: Configured `.github/workflows/release.yml` with cross-platform build matrix across Linux (`x86_64`, `aarch64`), macOS (`x86_64`, `aarch64` Apple Silicon), and Windows (`x86_64`).

- [x] **Phase 12.1: Universal Single-Line Installers, Versioning, Source Build Driver & Clean Crate Names - [P1]**
  - [x] **Clean Crate Names**: Renamed all crate folders to remove `axiom-` prefix (`crates/core`, `crates/syntax`, `crates/ir`, `crates/jit`, `crates/sim`, `crates/telemetry`, `crates/desktop`, `crates/cli`).
  - [x] **Universal Single-Line Installers**:
    - Linux & macOS: `curl -fsSL https://axiom.aerovex.net/install.sh | bash`
    - Windows: `irm https://axiom.aerovex.net/install.ps1 | iex`
  - [x] **Release Versioning**: Integrated version selection via `AXIOM_VERSION` env var, `--version` flag, and `-Version` PowerShell parameter with GitHub releases API discovery and fallback.
  - [x] **Dedicated Build from Source Driver**: Created `scripts/build_from_source.sh` (Linux/macOS) and `scripts/build_from_source.ps1` (Windows) with CLI-only, custom prefix, and debug/release options.
  - [x] **Documentation Portal**: Updated `docs/index.md` and `docs/guide/quickstart.md` with tabbed single-line installs, versioning guides, and build-from-source instructions.

- [x] **Phase 12: Unified Omnibar (`Ctrl+K`) & Embedded Scripting Shell - [P2]**
  - [x] **Omnibar Command Palette (`Ctrl+K` / `Cmd+K`)**: Instant fuzzy search across signals, netlist hierarchy, actions, fixtures, and documentation (`OmnibarModal.tsx`).
  - [x] **Interactive In-UI Scripting REPL Console**: Responsive terminal console with command history (`Up`/`Down`), tab completion, direct dispatch to `engineBridge` (`run`, `step`, `step delta`, `reset`, `get`, `set`, `force`, `release`, `report_timing`, `report_power`), and syntax colorized output.

- [x] **Phase 11: Timing Radar, Slack Waterfall & Hierarchical Energy Treemap - [P2]**
  - [x] **Visual Timing Constraints Editor**: SDC/XDC constraint wizard for clocks (50 MHz, 100 MHz, 200 MHz, 300 MHz) and path margin calculation.
  - [x] **Setup/Hold Slack Radar & Critical Path Waterfall**: Visual path delay explorer with cell delay vs. interconnect delay breakdowns (e.g. 63% Logic, 37% Routing) and automated pipeline register advice.
  - [x] **Clock Domain Crossing (CDC) Matrix**: Automated metastability risk analysis and 2-FF synchronizer validation across asynchronous clock boundaries.
  - [x] **Hierarchical Silicon Energy Treemap**: 2D squarified treemap visualizer showing dynamic power dissipation ($E = \frac{1}{2} C V^2$) and PDN package supply sag ($V_{sag} = IR + L di/dt$).

- [x] **Phase 10: Virtual Lab & Stimulus Rack (Zero-Boilerplate Hardware Prototyping) - [P1]**
  - [x] **Virtual Instrument Rack**: Front-panel palette with interactive 8-bit DIP switch bank, tactile pushbuttons (RESET strobe & STEP CLK), rotary quadrature hex encoder dial, dual 7-segment LED displays, and 16-bit SMD LED bar graph.
  - [x] **Real-Time In-RAM Stimulus Injection**: Direct state manipulation triggering sub-microsecond Cranelift JIT re-evaluation without physical JTAG hardware.
  - [x] **Waveform Stimulus Painter**: Visual mouse-drawn clock and vector stimulus generator with 1-click synthesizable IEEE 1800-2017 SystemVerilog testbench export (`${topModule}_tb.sv`).

- [x] **Phase 9: GPU-Accelerated Hardware Schematic DAG & Logic Cone Slicer - [P1]**
  - [x] **Interactive Canvas/WebGL Schematic DAG**: Elaborated BIR netlist visualizer (ports, registers, multiplexers, adders, operators) with auto-layout routing, pan/zoom, and minimap navigator.
  - [x] **Semantic Level-of-Detail (LOD)**: Multi-scale zoom transitions from macro module envelopes down to gate-level Cranelift JIT machine operations (`iadd`, `isub`, `band`, `icmp eq`).
  - [x] **Bidirectional Cross-Probing**: Full 3-way synchronization across Schematic DAG $\leftrightarrow$ Waveform Viewer $\leftrightarrow$ HDL Code Editor with line highlights and pulse halos.
  - [x] **1-Click Critical Logic Cone Slicer**: Instant fan-in datapath extraction and fan-out tree isolation (`F` / `O` hotkeys) with timing slack HUD and 12% background dimming.

- [x] **Phase 8: Advanced Waveform Innovation & Signal Inspection Engine - [P0]**
  - [x] **Multi-Radix Bus Exploder**: Expandable multi-bit vector buses into bit-indexed sub-lanes (`bus[0]..bus[W-1]`) with real-time radix switching (Hex, Binary, Unsigned, Signed Decimal, ASCII).
  - [x] **Dual-Cursor Monotonic Time Measurement**: Cursor A and Cursor B pins with floating HUD badge ($\Delta t$ in ps/ns/$\mu$s, frequency $f = 1/\Delta t$ in MHz/GHz).
  - [x] **Zero-Time Delta Accordion Viewer**: Expandable timeline drawer revealing internal $\delta$-cycles ($\delta_0 \to \delta_1 \to \dots$) with glitch hazard ribbons ($0 \to 1 \to 0$ and $1 \to 0 \to 1$).
  - [x] **Interactive Signal/Pin Forcing & Probing**: Live inspector modal allowing engineers to force `0`, `1`, `X`, `Z` or custom bus values with sub-microsecond in-RAM re-evaluation and release.

- [x] **Phase 7: Documentation Portal, Custom Domain CNAME & GitHub Pages CI/CD - [P0]**
  - [x] Rebranded to **Axiom EDA** under Aerovex (`axiom.aerovex.net`).
  - [x] Generated official vector logo (`logo.svg`), multi-res PNGs (512px, 64px, 32px, favicon.ico), and integrated into docs and UI header.
  - [x] Authored complete VitePress documentation suite in `docs/` with Obsidian dark theme tokens.
  - [x] Fixed and verified native LaTeX MathJax3 formula rendering ($P = \frac{1}{2} C V^2 f \alpha$, $V_{sag} = IR + L\frac{di}{dt}$).
  - [x] Deployed live to `axiom.aerovex.net` via GitHub Actions (`.github/workflows/deploy-docs.yml`) directly on GitHub Pages with CNAME custom domain and automatic SSL.
  - [x] Created `scripts/cloc.sh` to track handwritten code and documentation (11,471 lines across 110 files).

- [x] **Phase 6: Verification, Benchmarking & Tooling Parity - [P2]**
  - [x] Standalone Headless CLI Driver (`crates/axiom-cli`): In-RAM compilation (`compile`), headless simulation (`run`), batch IEEE 1364 VCD & SAIF 2.0 dumping, and benchmark command.
  - [x] Vivado Output Conformance Test Suite: Multi-fixture validation (ALU, Counter, Hierarchy, FIFO) verifying deterministic 4-state logic, glitch hazards, and SAIF toggle activity against Vivado reference patterns.
  - [x] In-RAM Performance Benchmark Suite: End-to-end latency measurement across Lexing, AST Parsing, Elaboration, Cranelift JIT compilation (sub-3 ms total JIT turnaround), and simulation throughput (>780,000 events/sec).

- [x] **Phase 5: Modern Dark-Themed Desktop & Web Application (Tauri + React + PostCSS) - [P1]**
  - [x] Tauri v2 & Vite Scaffold: Dual-mode build setup (`axiom-desktop` crate + `ui/` React 19 + TypeScript + Vite production bundle).
  - [x] React 19 + TypeScript + PostCSS UI Shell: Sleek, minimalist, dark-themed engineering dashboard (Obsidian / Linear style).
  - [x] High-Performance Waveform Viewer (Canvas 2D): 60+ FPS digital waveform viewer with single-bit levels, bus transition envelopes, time ruler, cursor inspection, and zero-time delta-cycle hazard magnifier.
  - [x] Voltage & Power Telemetry Graphing: Real-time dynamic power (mW), transient current spikes (mA), PDN voltage sag ($V_{sag} = IR + L\frac{di}{dt}$), and summary energy metrics cards.
  - [x] Interactive Simulation Control Console: Step physical time (+1 ns, +100 ps), step discrete delta cycle (`step_delta()`), run free/pause, reset.
  - [x] HDL Code Editor & Hierarchy Explorer: Line-numbered Verilog code editor with live syntax tabs, elaborated netlist hierarchy explorer, preset design fixture switcher (ALU, Counter with Glitches, Hierarchical Core), and IEEE 1364 VCD / SAIF 2.0 exporters.

- [x] **Phase 0: Deep Vivado & HDL Research & Architecture Analysis**
  - [x] Extract and analyze Vivado's HDL simulation pipeline (`xvlog`, `xvhdl`, `xelab`, `xsim`).
  - [x] Analyze Vivado's Power Analysis methodology (UG907, SAIF, vectorless estimation, switching activity).
  - [x] Populate `vivadoanalysis/` with 8 exhaustive reference documentation and architecture breakdowns.
  - [x] Formulate project philosophy and design manifesto in `GEMINI.md` (recording all user techniques and git safety constraints).
  - [x] Author comprehensive internal architecture specifications in `analysis/` with master index `analysis/analysis.md`.
  - [x] Formulate Phase 1 Industrial Implementation Plan.
- [x] **Phase 1: Core HDL Engine Foundation (Rust Core & IR) - [P0]**
  - [x] Setup multi-crate Cargo workspace (`crates/axiom-core`, `crates/axiom-syntax`, `crates/axiom-ir`).
  - [x] Implement fundamental simulation types in `axiom-core`: `SimTime`, IEEE 1800 4-state logic bit representations (`Logic4`, `LogicVector`), source spans, and diagnostic reporter.
  - [x] Implement streaming zero-allocation lexer & preprocessor in `axiom-syntax` for Verilog-2005 & SystemVerilog (`define`, `ifdef`, `include`).
  - [x] Implement AST data structures and Pratt recursive-descent parser with syntax error recovery.
  - [x] Implement hierarchical elaborator in `axiom-ir`: module instantiation traversal, constant folding, parameter resolution, generate block unrolling, and port binding.
  - [x] Formulate Axiom Intermediate Representation (BIR) dataflow graph with explicit sensitivity mapping.
  - [x] Verify complete pipeline against test fixtures (`alu.v`, `counter.v`, `hierarchy.v`) with 100% test pass rate.
- [x] **Phase 2: JIT Machine Code Compiler & In-RAM Execution Engine - [P0]**
  - [x] Setup Cranelift JIT compiler crate `axiom-jit` with in-RAM code emission.
  - [x] Implement contiguous dual-word state arena (`SimStateArena`) with 64-bit aligned value and mask storage for O(1) 4-state logic operations.
  - [x] Implement in-RAM Cranelift JIT codegen compiling continuous assignments and arithmetic/logic expressions directly into executable machine code in memory.
  - [x] Implement portable and WebAssembly (`wasm32-unknown-unknown`) execution path (`PortableEvaluator`) with support for procedural blocks, non-blocking assignments (NBAs), `if/else`, and `case` branches.
  - [x] Verify end-to-end JIT and portable execution against realistic ALU and counter fixtures with 100% test pass rate across the workspace.
- [x] **Phase 3: Stratified Event Scheduler & Delta-Time Stepping API - [P0]**
  - [x] Setup simulation kernel crate `axiom-sim`.
  - [x] Implement IEEE 1800 Stratified Event Queue (`StratifiedEventQueue`) with min-heap ordering by `(SimTime, Delta, Region, SeqId)`.
  - [x] Implement caller-controlled Manual Delta-Time Tick Engine: `tick(delta_time)` and discrete delta cycle stepping: `step_delta()`.
  - [x] Implement zero-overhead combinational glitch detector (`GlitchDetector`) capturing static and dynamic hazards across zero-delay delta cycles.
  - [x] Implement event listener and trace observation framework (`SimEventListener`, `SimTraceRecorder`).
  - [x] Implement fast in-RAM simulation state snapshotting and rollback (`SimSnapshot`, `CheckpointId`) for instant timeline scrubbing.
  - [x] Verify complete event scheduling and delta stepping against clocked counter and ALU fixtures with 100% pass rate (30 workspace tests).
- [x] **Phase 4: Voltage, Energy & Power Telemetry Engine - [P1]**
  - [x] Setup telemetry crate `axiom-telemetry`.
  - [x] Implement lumped physical capacitance model (`NetCapacitanceModel`) capturing pin, wire, and fanout capacitance per net and bit.
  - [x] Implement power distribution network (PDN) impedance model (`PowerRail`, `PdnModel`) simulating dynamic current spikes and voltage sag ($V_{sag} = IR + L\frac{di}{dt}$).
  - [x] Implement physics-informed switching activity and energy accumulator (`TelemetryCollector`) producing real-time `TelemetryFrame` packets for visualizers.
  - [x] Implement IEEE 1364 Value Change Dump (`VcdWriter`) exporter for digital waveform inspection.
  - [x] Implement Switching Activity Interchange Format (`SaifWriter`) exporter for 100% interoperability with AMD Vivado `read_saif`.
  - [x] Verify all telemetry models, voltage sag calculations, and exporter formats with 100% pass rate (36 workspace tests).
