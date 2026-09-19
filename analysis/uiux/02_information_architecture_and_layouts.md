# Axiom Information Architecture, Layout Ergonomics & De-Cramping

## 1. Information Architecture & Viewport Anatomy

Hardware Description Language (HDL) design environments are among the most information-dense software applications in existence. An engineer actively manipulates three distinct mental representations of hardware simultaneously:
1. **Symbolic RTL Text**: Behavioral Verilog / SystemVerilog code, parameters, directives, and syntax trees.
2. **Topological Hardware Netlist**: Gate-level Directed Acyclic Graphs (DAG), module boundaries, ports, registers, and interconnects.
3. **Temporal Waveform Dynamics**: Digital logic transitions over picoseconds, clock edges, setup/hold slack windows, and zero-time delta cycles.

The primary challenge of EDA Information Architecture is organizing these three orthogonal representations so the user can transition between them seamlessly without cognitive friction or layout claustrophobia.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ Global Engineering Header (40px)                                                      │
├─────────────┬────────────────────────────────────┬─────────────────────────────────────┤
│ Collapsible │ Monaco HDL Code Editor             │ Visualizer Container                │
│ Sidebar     │ (Active File, Breadcrumb, Markers) │ (Schematic DAG / Virtual Lab Rack / │
│ (38px or    │                                    │  Waveforms / Timing Radar)          │
│  228px)     │                                    │                                     │
│             │                                    │                                     │
│             │                                    │                                     │
│             │                                    │                                     │
├─────────────┴────────────────────────────────────┴─────────────────────────────────────┤
│ Unified Bottom Dock (Collapsed 28px Status Bar / Expanded 240px Drawer)               │
│ [Console & REPL]   [Problems & Linter (0)]   [Power & PDN]   [Glitches (0)]            │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Layout Archetypes: Why Quad-Split Fails & Dual-Pane Excels

### 2.1. The Legacy Quad-Split Dilemma
Legacy tools like Vivado, ModelSim, and older IDEs divide the screen into four static quadrants:
- Top-Left: Sources Explorer
- Top-Right: Code Editor
- Bottom-Left: Netlist / Console
- Bottom-Right: Waveforms / Schematics

**Why Quad-Split Fails**:
- On typical $13''$ to $15''$ laptop screens ($1440 \times 900$ or $1920 \times 1080$), each quadrant receives barely $600\text{px} \times 350\text{px}$.
- Code lines wrap prematurely or require endless horizontal scrolling.
- Waveform signal gutters consume half the window, leaving only 2–3 clock periods visible.
- Schematic gates clip into unreadable clusters.
- The interface feels claustrophobic, overwhelming users with visual noise ("quad-split paralysis").

### 2.2. Axiom's Spacious Dual-Pane Architecture
Axiom replaces static quadrants with a **Spacious Dual-Pane Studio**:
- **Left Pane**: Monaco HDL Editor dedicated to full-height source code editing (adjustable from 18% to 75% width).
- **Right Pane**: Full-height, full-width Visualizer Container equipped with an ergonomic tab switcher:
  - `⚡ Schematic DAG`
  - `🎛 Virtual Lab Rack`
  - `📈 Waveform Viewer`
  - `⏱ Timing & Energy Radar`
- **Optional Stacked Waveform Toggle (`[+ Waveforms]`)**: When simultaneous code and waveform inspection is required, the right pane can cleanly split vertically (Visualizer on top, Waveforms on bottom), controlled by a dedicated vertical splitter.
- **1-Click Fullscreen Maximization (`⛶`)**: Any pane can instantly expand to 100% full screen, hiding all splitters and chrome for concentrated debugging.

```
┌──────────────────────────────┬─────────────────────────────────────────────────────────┐
│ Left Pane: Code Focus        │ Right Pane: Visualizer Focus (100% Height & Width)      │
│                              │ [⚡ Schematic DAG]  [🎛 Virtual Lab]  [📈 Waveforms]     │
│                              ├─────────────────────────────────────────────────────────┤
│                              │                                                         │
│                              │                                                         │
│                              │                                                         │
│                              │                                                         │
└──────────────────────────────┴─────────────────────────────────────────────────────────┘
```

---

## 3. Resizable Splitter Physics & De-Cramping Engine

### 3.1. Percentage-Based Dynamic Clamping
Fixed pixel layouts fail across varied monitor DPIs and aspect ratios. Axiom implements zero-dependency draggable splitters (`ResizableSplitter.tsx`) utilizing **percentage-based geometry** with strict safety clamps:

$$\text{Editor Width \%} = \text{clamp}\left(18\%, \; \frac{\Delta x}{W_{\text{viewport}} - W_{\text{sidebar}}} \times 100\%, \; 75\%\right)$$

```typescript
// App.tsx: Resizing calculation with strict boundary preservation
const handleEditorResize = useCallback((deltaPx: number) => {
  const totalWidth = window.innerWidth - (isSidebarCollapsed ? 38 : 228);
  if (totalWidth <= 0) return;
  const deltaPct = (deltaPx / totalWidth) * 100;
  setEditorWidthPercent((prev) => 
    Math.max(18, Math.min(75, Math.round((prev + deltaPct) * 10) / 10))
  );
}, [isSidebarCollapsed]);
```

### 3.2. Splitter Affordance & Dragging Physics
- **Resting State**: 4px transparent hit strip with a 1px border line (`var(--border-subtle)`).
- **Hover State**: Hit strip expands to 8px; divider shifts to `var(--accent-blue)` with cursor `col-resize` or `row-resize`.
- **Active Drag State**: Global user-select is disabled (`user-select: none`); an invisible full-screen overlay prevents iframe/canvas mouse loss; splitter handle glows cyan.
- **Double-Click Reset**: Double-clicking the splitter handle instantly resets the layout to the balanced golden ratio preset (42% / 58%).

---

## 4. Sidebar Architecture & Wayfinding

The left sidebar (`Sidebar.tsx`) handles project organization and elaborated netlist exploration. To prevent horizontal space theft, it features a **Dual-State Collapsible Strip**:

```
State A: Expanded (228px)                  State B: Collapsed (38px)
┌──────────────────────────────────────┐   ┌──────┐
│ [Sources]  [Netlist Hierarchy]   [<] │   │ [>]  │
├──────────────────────────────────────┤   ├──────┤
│ ▾ Design Sources (sources_1)         │   │ [📁] │
│   📄 logic_circuit.v           [TOP] │   │      │
│ ▾ Simulation Sources (sim_1)         │   │ [🌳] │
│   📄 tb_logic_circuit.sv             │   │      │
│ ▾ Constraints (constrs_1)            │   │ [⚙]  │
│   📄 timing.xdc                      │   │      │
└──────────────────────────────────────┘   └──────┘
```

### 4.1. Vivado Project File Sets
Files are partitioned strictly according to authentic Vivado conventions:
- `sources_1`: Verilog / SystemVerilog synthesizeable RTL. The designated top module carries a cyan `[TOP]` badge with a 1-click star trigger to change the hierarchy root.
- `sim_1`: Testbenches, verification suites, and stimulus generators.
- `constrs_1`: Timing constraints and pin maps (`.xdc`).

### 4.2. Breadcrumb Navigation & File Tabs
In `HdlEditor.tsx`, the tab strip displays open file tabs with close buttons (`✕`) and dirty indicators. Below the tabs, an active breadcrumb trail confirms the hierarchical location:
$$\text{project} \;\;>\;\; \text{sources\_1} \;\;>\;\; \text{logic\_circuit.v} \;\;>\;\; \text{module logic\_circuit}$$

---

## 5. Unified Bottom Dock: Ergonomic Status Bar Integration

Auxiliary tools (REPL Console, Problems & Linter, Silicon Power Telemetry, Glitch Hazard Detector) consume significant screen space if left open permanently. 

Axiom replaces bulky stacked panels with a **Collapsible Bottom Dock** (`UnifiedBottomDock.tsx`):

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ Status Bar (Collapsed: 28px Height)                                                    │
│ [>_ Console]  [⚠ Problems (0)]  [⚡ Power: 14.2 mW]  [Rail: 0.982 V]  [Glitches: 0]   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### Key Dock Interactions
- **1-Click Expansion**: Clicking any status pill or the dock tab expands the drawer to its last resized height (default 240px).
- **Smooth Drag Resizing**: An integrated top handle allows dragging height from 120px to 80% of viewport.
- **1-Click Maximize (`⛶`)**: Expands the dock to 100% of the screen for comprehensive telemetry graph analysis or extensive REPL scripting sessions.
- **Intelligent Error Pulsing**: When a compilation error or zero-time glitch occurs while the dock is collapsed, the corresponding status badge pulses in rose/amber, alerting the engineer without jarringly ripping the current view open.

---

## 6. Responsive Mobile Viewport Architecture ($\le 768\text{px}$)

High-density engineering interfaces notoriously fail on mobile viewports due to horizontal overflow, unreadable microscopic fonts, and unusable resizable splitters.

Axiom incorporates a **Dedicated Responsive Mobile Engine**:

```
Desktop Mode (Width > 768px)          Mobile Mode (Width <= 768px)
┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│ Sidebar │ Editor │ Visualizer   │   │ Mobile Header (42px) [☰] [12ns] │
│         │        │              │   ├─────────────────────────────────┤
│         │        │              │   │ Single Panel at a Time (100%)   │
│         │        │              │   │ - Monaco Code Editor            │
│         │        │              │   │ - Schematic DAG Canvas          │
├─────────┴────────┴──────────────┤   │ - Virtual Lab Breadboard        │
│ Bottom Dock (Collapsed / Drawer)│   │ - Waveform Viewer Canvas        │
└─────────────────────────────────┘   ├─────────────────────────────────┤
                                      │ Mobile Bottom Bar (52px)        │
                                      │ [Code] [DAG] [Lab] [Wave] [Term]│
                                      └─────────────────────────────────┘
```

### 6.1. Complete Splitter & Sidebar Disabling
On mobile viewports, multi-pane splitters, resizable handles, and desktop sidebars are **completely removed from the DOM**. The application locks into a single, focused viewport where `document.body.scrollWidth === window.innerWidth` (0px horizontal overflow).

### 6.2. Off-Canvas Left Drawer (`MobileDrawer.tsx`)
Clicking the header hamburger icon (`☰`) smoothly slides in an off-canvas drawer from the left (`translateX(-100%)` to `translateX(0)`) backed by a 6px blurred dark backdrop:
- Switch active studio tool with 1 tap.
- Browse authentic Vivado project file sets (`sources_1`, `sim_1`, `constrs_1`).
- Trigger instant JIT compilation or inspect simulation clock status.
- Drawer auto-closes automatically upon file or tool selection.

### 6.3. 1-Tap Thumb Bottom Navigation Bar (`MobileBottomBar.tsx`)
A fixed 52px–56px navigation bar stays anchored to the bottom of the screen, equipped with notch safe-area padding (`env(safe-area-inset-bottom)`):
- 5 thumb-friendly touch targets ($\ge 48\text{px} \times 44\text{px}$):
  1. `Code` (Blue `#3b82f6`)
  2. `Schematic` (Cyan `#06b6d4`)
  3. `Lab` (Amber `#f59e0b`)
  4. `Waves` (Emerald `#10b981`)
  5. `Console` (Purple `#8b5cf6`)
- Dynamic badges showing real-time LSP diagnostic errors and simulation glitches directly on the icons.
