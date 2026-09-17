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
