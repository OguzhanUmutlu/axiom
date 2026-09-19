# Axiom Desktop & Web Application Architecture (Tauri v2 + React 19 + PostCSS + Vite)

## 1. Overview and Design Philosophy

The Axiom user interface is a desktop application that runs independently of the host operating system (Linux, macOS, Windows) and can also run directly inside standard web browsers.

### Design Principles:
1. **Clean, Modern, Dark-Themed Aesthetic**: Minimalist, distraction-free engineering environment (inspired by Linear, Obsidian, and modern VS Code).
2. **High-Performance Waveform Rendering**: 60+ FPS digital waveform scrolling and zooming using Canvas 2D / WebGL, capable of rendering 100,000+ transitions without frame drops.
3. **Integrated Dual-Domain Visualization**: Logic waveforms and analog voltage/power curves rendered synchronously side by side.
4. **Instant Zero-Config Web Mode**: Runs via Vite in development, compiling down to a static WebAssembly SPA for web deployment.

---

## 2. Technology Stack

- **Desktop Framework**: **Tauri v2** (Rust-based native shell, uses OS webview, < 20 MB binary footprint, 30x lighter than Electron).
- **UI Library**: **React 19** with **TypeScript** for strict type safety.
- **Styling**: **PostCSS** with modern CSS Variables, nesting, and CSS modules (clean, fast, zero CSS-in-JS runtime overhead).
- **Build & Dev Tool**: **Vite 6** (sub-second HMR for rapid frontend development).
- **Waveform Engine**: Custom WebGL / Canvas 2D hardware-accelerated time-series renderer.
- **State Management**: **Zustand** (lightweight, minimal re-renders).
- **Code Editor**: **Monaco Editor** or **CodeMirror 6** with custom syntax highlighting for Verilog/SystemVerilog.

---

## 3. Desktop Application Layout & Component Hierarchy

```
+-------------------------------------------------------------------------------+
| Header: [Project Name] [Compile] [Play/Pause] [Step 1ns] [Step Delta] [Reset] |
+---------------+-----------------------------------------------+---------------+
| Hierarchy /   | Waveform & Inspection Viewport                | Telemetry &   |
| Scope Tree    |                                               | Power Graph   |
|               | +-------------------------------------------+ |               |
| > tb_top      | | clk     _-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-| | Dynamic Power |
|   > u_cpu     | | rst_n   ________/-------------------------| | ~ 14.2 mW     |
|     > u_alu   | | pc      [0000][0004][0008][000C]          | |               |
|     - din     | | din     [  00  ][  FF  ][  AA  ]          | | Vcore Sag:    |
|     - dout    | +-------------------------------------------+ | 0.982 V       |
|               | | Delta Zoom: [d0: 0] -> [d1: 1] (Glitch!)  | |               |
|               | +-------------------------------------------+ | Current (mA): |
|               | Synchronized Analog Voltage & Current Curve:  | /\    /\  /\  |
|               |  Vcore: ------------------------------------  |   \__/  \/    |
|               |  Icore: ___/\____/\___/\____/\______________  |               |
+---------------+-----------------------------------------------+---------------+
| Bottom Panel: [Console / Tcl Output] [Errors & Diagnostics] [SAIF / VCD Export]|
+-------------------------------------------------------------------------------+
```

---

## 4. Hardware-Accelerated Waveform Visualizer

Rendering thousands of digital signal transitions in the DOM via SVGs causes catastrophic browser lag. Axiom implements a virtualized **Canvas 2D / WebGL Time-Series Canvas**:

### Key Features:
- **Viewport Virtualization**: Only signals and time ranges currently visible in the scroll window are computed and rendered.
- **LOD (Level of Detail) Aggregation**: When zoomed far out, dense high-frequency transitions are aggregated into hatched transition blocks, avoiding sub-pixel overdraw.
- **Sub-Cycle Delta Mode**: Clicking a transition expands a magnification window revealing every intermediate delta cycle ($\delta_0 \to \delta_1 \to \delta_2$) with color-coded hazard highlights.
- **Bus Grouping & Radix Switching**: Expand multi-bit buses into individual wires or format as Hexadecimal, Binary, Unsigned, Signed, or ASCII.

---

## 5. Dual-Target Build Architecture (Desktop & Web)

```
                       axiom-ui (React + Vite)
                                    |
                    +---------------+---------------+
                    |                               |
                    v (Target: Desktop)             v (Target: Web SPA)
           +-----------------+             +-----------------+
           | Tauri v2 Shell  |             | Browser Window  |
           | - Native Rust   |             | - Web Worker    |
           |   JIT Backend   |             | - WASM Engine   |
           | - Native IPC    |             | - Shared Memory |
           +-----------------+             +-----------------+
```

- In **Desktop mode** (`npm run tauri dev`), the UI calls Tauri IPC commands that invoke the native Cranelift JIT engine.
- In **Web mode** (`npm run dev`), the UI instantiates `axiom-wasm` inside a Web Worker and communicates via typed message channels.
- Zero conditional code in UI components—both modes share the identical React frontend code.

---

## 6. Ergonomic De-cramping & Spatial Rhythm (Phase 12.10)

To resolve the dense, claustrophobic nature typical of legacy EDA suites (e.g. Vivado's tiny 11px font sizes and multi-split crowding), Axiom Phase 12.10 enforces clean spatial breathing room:
- **Collapsed Status Bar Mode**: Unified bottom dock collapses from ~280px to a sleek 28px status strip, liberating ~252px of vertical canvas.
- **Collapsible Sidebar**: Shrinks to a 38px icon rail, giving code editors and schematics +242px of uninterrupted horizontal width.
- **Single-Panel Mobile Drawer**: On viewports $\le 768\text{px}$, resizable multi-pane splitters are completely disabled. An off-canvas slide-out drawer (`MobileDrawer.tsx`) and thumb-friendly bottom bar (`MobileBottomBar.tsx`) render 1 full-viewport panel at a time.
- **De-cramped Tree Items & Touch Targets**: File tree items and breadcrumbs maintain $\ge 5\text{px}$ vertical padding and $\ge 28\text{px}$ row height; mobile touch targets enforce $\ge 40\text{px}$ hitbox boundaries.

---

## 7. Unified Engineering Design System & Motion Curves

All UI components adhere to standardized semantic CSS tokens and utility classes defined in `ui/src/styles/theme.css`:
- **Button System**: `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-cyan`, `.btn-danger`, `.btn-icon` with `:active: scale(0.97)` tactile depression.
- **Micro-Transitions**: Strictly capped at `150ms cubic-bezier(0.16, 1, 0.3, 1)` to eliminate sluggishness while preventing abrupt layout jumps.
- **Accessibility & Focus Rings**: Every interactive element features high-contrast `focus-visible: 2px solid var(--accent-blue)` with `outline-offset: 1px`.
- **Aerospace Brand Mark**: Faceted precision chevron vector mark (`ui/public/logo.svg`) featuring cyan-to-blue neon glow and crisp geometry.

---

## 8. Global Multi-Language (i18n) Architecture

Axiom features full internationalization without heavyweight external runtime dependencies:
- **Typed Translation Catalog**: Fully typed `Translations` interface in `ui/src/i18n/types.ts` guaranteeing compile-time detection of missing translation keys.
- **Dual-Mode Accessor (`TFunction`)**: Supports both callable path lookup `t("timing.wns")` and strongly-typed direct property access `t.timing.wns`.
- **Browser Language Auto-Detection**: Reads `navigator.language` on first visit and persists user preferences in `localStorage` under `axiom_language`.
- **7 Supported Production Locales**:
  1. `en`: English (Master Catalog)
  2. `tr`: Türkçe (Turkish - natively translated with standard digital logic terminology)
  3. `de`: Deutsch (German)
  4. `es`: Español (Spanish)
  5. `fr`: Français (French)
  6. `ja`: 日本語 (Japanese)
  7. `zh`: 简体中文 (Simplified Chinese)

---

## 9. White Katana Slash Cursor Effect Architecture

To enhance the kinetic responsiveness and tactile feel of the Monaco HDL Editor without introducing visual clutter or obstruction:
- **Canvas Overlay (`KatanaCursorOverlay.tsx`)**: High-DPI canvas overlay placed directly above the editor text layer (`pointer-events: none`, `z-index: 10`).
- **Frame-Rate Independent Follower Physics**: Exponential decay formula:
  $$\vec{p}_{\text{trail}}(t + \Delta t) = \vec{p}_{\text{target}} + (\vec{p}_{\text{trail}}(t) - \vec{p}_{\text{target}}) \cdot e^{-\lambda \Delta t}$$
  with $\lambda = 18.0\text{ s}^{-1}$, yielding a fluid $\sim 90\text{ms}$ smooth trailing lag across 60Hz, 120Hz, and 144Hz displays.
- **Tapered Katana Blade Polygon**: As the cursor moves, a dynamic polygon connects the trailing needle-point to the leading cutting edge with a radiant pure-white core (`#ffffff`), glowing silver aura (`rgba(255, 255, 255, 0.85)`), and a razor-thin central spine line (*hamon*).
- **Curved Slash Strike Arcs (*Sori*)**: Cursor jumps $> 12\text{px}$ trigger a quadratic bezier slash arc with subtle natural katana curvature and 2–3 microscopic luminous cutting glints, dissolving exponentially in 180ms.
- **Self-Sleeping RAF Pipeline**: When the cursor rests and strikes fade, the render loop automatically stops and clears the canvas, guaranteeing 0% idle CPU and GPU consumption.
- **User Control & Persistence**: Integrated `Swords` button in the editor tab bar allows toggling the effect on or off, with preference persisted in `localStorage.getItem("axiom_katana_cursor")`.
