# Axiom Design System & Theming Architecture

## 1. Executive Summary & Design System Foundations

High-density technical tools, Computer-Aided Design (CAD) applications, and Electronic Design Automation (EDA) suites demand a design system fundamentally distinct from consumer web applications or standard SaaS dashboards. Engineers spend 8 to 14 consecutive hours in front of these interfaces analyzing intricate gate schematics, timing slack waterfalls, multi-radix waveform buses, and thousands of lines of hardware description code.

Axiom's Design System is engineered around three core imperatives:
1. **Zero Visual Fatigue**: Deep, low-reflectance, dark-mode surfaces that eliminate eye strain without causing optical smearing or muddy gray hazing.
2. **Sub-Pixel Information Density**: Compact sizing and micro-typography that maximize visible signal bandwidth while maintaining crisp legibility and comfortable click targets.
3. **Rigorous Semantic Color Coding**: Universal, deterministic mapping of hues to physical electrical logic states ($0$, $1$, $X$, $Z$), timing margins, diagnostic severities, and clock domains.

---

## 2. Design Tokens & CSS Variable Architecture

All visual attributes across Axiom are declared as atomic CSS custom properties in `theme.css`. Hardcoded hex values, arbitrary inline pixels, and detached styling are strictly prohibited.

```css
:root {
  color-scheme: dark;

  /* -------------------------------------------------------------------------
     Surface & Background Tokens (Layered Z-Elevation Hierarchy)
     ------------------------------------------------------------------------- */
  --bg-primary: #0d0f12;    /* Master canvas, main application backdrop */
  --bg-secondary: #13171d;  /* Headers, sidebars, toolbars, inactive tabs */
  --bg-tertiary: #1a1f26;   /* Active cards, input fills, dropdown options */
  --bg-elevated: #222933;   /* Modals, popovers, tooltip surfaces */
  --bg-hover: #1f2631;      /* Hover states for interactive items */
  --bg-active: #2a3340;     /* Selected / depressed states */

  /* -------------------------------------------------------------------------
     Border & Divider Tokens (High-Density Spatial Separation)
     ------------------------------------------------------------------------- */
  --border-subtle: #242c38; /* 1px standard structural panel borders */
  --border-strong: #364253; /* Active panel borders, hover dividers */
  --border-focus: #3b82f6;  /* Keyboard focus rings, selected item outlines */

  /* -------------------------------------------------------------------------
     Text & Content Tokens
     ------------------------------------------------------------------------- */
  --text-primary: #f1f5f9;   /* High-emphasis headers, code, active labels */
  --text-secondary: #94a3b8; /* Normal body text, secondary labels, icons */
  --text-muted: #64748b;     /* Inactive tabs, metadata, line numbers */
  --text-inverse: #0f172a;   /* Text on bright saturated badges */

  /* -------------------------------------------------------------------------
     Semantic Brand & Action Accents
     ------------------------------------------------------------------------- */
  --accent-blue: #3b82f6;    /* Primary actions, active focus, editor cursor */
  --accent-cyan: #06b6d4;    /* Clock signals, top module badges, Vivado branding */
  --accent-emerald: #10b981; /* Logic High (1), compile success, positive slack */
  --accent-amber: #f59e0b;   /* High-Z (Z), warnings, clock domain crossing alerts */
  --accent-rose: #f43f5e;    /* Unknown (X), timing violations, compile errors */
  --accent-purple: #8b5cf6;  /* Constraints (XDC), SPI protocol, REPL tokens */

  /* -------------------------------------------------------------------------
     HDL Electrical Logic Signal Colors (IEEE 1800 Standard)
     ------------------------------------------------------------------------- */
  --signal-0: #64748b;       /* Logic Low (GND, 0V) */
  --signal-1: #10b981;       /* Logic High (Vdd, Active Rail) */
  --signal-x: #f43f5e;       /* Unknown / Uninitialized / Contention (X) */
  --signal-z: #f59e0b;       /* High-Impedance / Tri-State Floating (Z) */
  --signal-bus: #38bdf8;     /* Multi-bit vector envelope */
  --signal-glitch: #ec4899;  /* Zero-time combinational dynamic hazard */

  /* -------------------------------------------------------------------------
     Typography Tokens
     ------------------------------------------------------------------------- */
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace, ui-monospace;
  --font-sans: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
  --font-size-xs: 10.5px;
  --font-size-sm: 11.5px;
  --font-size-base: 12px;
  --font-size-lg: 13.5px;
  --font-size-code: 13px;

  /* -------------------------------------------------------------------------
     Compact Engineering Density Geometry
     ------------------------------------------------------------------------- */
  --density-header-h: 40px;
  --density-tab-h: 28px;
  --density-toolbar-h: 26px;
  --density-breadcrumb-h: 22px;
  --density-dock-header-h: 28px;

  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
}
```

---

## 3. Dark Engineering Palette & Contrast Physics

### 3.1. Surface Luminance Layering
Standard consumer web apps use shadows (`box-shadow: 0 10px 25px rgba(...)`) to convey elevation. In high-density CAD and EDA interfaces, soft diffuse drop shadows create blurry visual clutter that bleeds into adjacent waveform traces and fine netlist lines.

Axiom utilizes **Surface Luminance Elevation with 1px Structural Borders**:
- **Layer 0 (Base Canvas)**: `#0d0f12` ($L \approx 0.7\%$)
- **Layer 1 (Structural Sidebars & Header)**: `#13171d` ($L \approx 1.2\%$) with `1px solid var(--border-subtle)`
- **Layer 2 (Interactive Cards & Inputs)**: `#1a1f26` ($L \approx 1.8\%$)
- **Layer 3 (Floating Toolbars & Modals)**: `#222933` ($L \approx 2.7\%$) with `1px solid var(--border-strong)` and `backdrop-filter: blur(8px)`

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 3: Modal Palette / Popover (#222933)                  │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Layer 2: Input Bay / Active Card (#1a1f26)              │ │
│ │ ┌─────────────────────────────────────────────────────┐ │ │
│ │ │ Layer 1: Container / Sidebar / Header (#13171d)     │ │ │
│ │ │ ┌─────────────────────────────────────────────────┐ │ │ │
│ │ │ │ Layer 0: Deep Canvas (#0d0f12)                  │ │ │ │
│ │ │ └─────────────────────────────────────────────────┘ │ │ │
│ │ └─────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 3.2. Contrast Ratio Mathematics (WCAG 2.2 AAA / AA)
Contrast between text/foreground ($L_1$) and surface/background ($L_2$) is calculated using standard relative luminance:

$$L = 0.2126 R_s + 0.7152 G_s + 0.0722 B_s$$

$$\text{Contrast Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05}$$

| Element Pair | Foreground | Background | Calculated Ratio | WCAG 2.2 Compliance | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Primary Text on Base | `#f1f5f9` ($L \approx 0.89$) | `#0d0f12` ($L \approx 0.007$) | **16.5 : 1** | **Pass AAA** | Code, headers, active values |
| Secondary Text on Base | `#94a3b8` ($L \approx 0.36$) | `#0d0f12` ($L \approx 0.007$) | **7.2 : 1** | **Pass AAA** | Labels, breadcrumbs, signals |
| Cyan Accent on Surface | `#06b6d4` ($L \approx 0.40$) | `#13171d` ($L \approx 0.012$) | **7.3 : 1** | **Pass AAA** | Clocks, top modules |
| Emerald Logic 1 on Base | `#10b981` ($L \approx 0.38$) | `#0d0f12` ($L \approx 0.007$) | **7.5 : 1** | **Pass AAA** | High rails, valid flags |
| Rose Logic X on Base | `#f43f5e` ($L \approx 0.23$) | `#0d0f12` ($L \approx 0.007$) | **4.9 : 1** | **Pass AA** | Unknown states, violations |
| Muted Metadata | `#64748b` ($L \approx 0.16$) | `#0d0f12` ($L \approx 0.007$) | **3.6 : 1** | *Caution (AA Large)* | Timestamps, inactive tabs |

> [!TIP]
> **Design Audit Finding & Remediation:**
> To guarantee strict WCAG AA compliance across all viewports, secondary muted labels smaller than 12px should migrate from `#64748b` (3.6:1) to `#94a3b8` (7.2:1), ensuring critical unit indicators (`ns`, `ps`, `mW`) remain readable even on uncalibrated laptop monitors in direct sunlight.

---

## 4. Electrical Signal & Hardware Domain Theming

Unlike software code where colors are largely aesthetic, in hardware design every color maps to an IEEE 1800 logic standard or physical semiconductor constraint:

```
┌──────────────┬──────────────┬────────────────────────┬───────────────────────────────────────────┐
│ Logic State  │ Color Token  │ Hex Value              │ Hardware Engineering Semantics            │
├──────────────┼──────────────┼────────────────────────┼───────────────────────────────────────────┤
│ Logic 0      │ --signal-0   │ #64748b (Slate 500)    │ Ground rail (0V, GND). Clean Low state.   │
│ Logic 1      │ --signal-1   │ #10b981 (Emerald 500)  │ Active Vdd rail. High assertion.          │
│ Logic X      │ --signal-x   │ #f43f5e (Rose 500)     │ Uninitialized net, multi-driver conflict. │
│ Logic Z      │ --signal-z   │ #f59e0b (Amber 500)    │ High-impedance, tri-state disconnected.   │
│ Multi-bit Bus│ --signal-bus │ #38bdf8 (Sky 400)      │ Parallel vector bundle ([31:0], [7:0]).   │
│ Glitch Hazard│ --signal-glitch│ #ec4899 (Pink 500)   │ Zero-time combinational race condition.   │
└──────────────┴──────────────┴────────────────────────┴───────────────────────────────────────────┘
```

### Colorblind & Vision Deficiency Safety (Deuteranopia & Protanopia)
Red-green colorblindness affects roughly 8% of male engineers. To prevent critical confusion between Logic `1` (Emerald) and Logic `0` or Error (Rose), Axiom pairs color with **redundant geometric and textural cues**:
1. **Waveform Levels**: Logic `1` is drawn on the top line ($Y = 4\text{px}$); Logic `0` is drawn on the bottom line ($Y = 24\text{px}$).
2. **Logic X & Z Fill Textures**: Logic `X` lanes feature diagonal red hatching lines (`/ / /`); Logic `Z` lanes feature a centered dashed line (`- - -`).
3. **Diagnostic Badges**: Every diagnostic indicator pairs its color with an explicit icon (Rose `AlertCircle`, Amber `AlertTriangle`, Emerald `CheckCircle2`).

---

## 5. Typography Scale & Monospace Engineering

### 5.1. Dual-Typeface Modular Architecture
- **Primary Interface Font**: `Inter`, `-apple-system`, `system-ui`. Selected for its neutral tone, tall x-height, open apertures, and crisp letterforms at small sizes (10.5px–12px).
- **Technical & Code Font**: `JetBrains Mono`, `Fira Code`. Selected for strict character discrimination (`0` vs `O`, `1` vs `l` vs `I`), distinct programming ligatures (`<=`, `!=`, `==`, `->`), and tabular figures.

### 5.2. Type Scale Table

```
Scale Step     Size    Line Height  Weight   Typical Application
───────────────────────────────────────────────────────────────────────────
Headline       20px    28px         700      Launchpad Hero, Modal Titles
Title          14px    20px         600      Panel Headers, Visualizer Tabs
Base Body      12px    17px         400/500  General UI, Tree items, Menus
Small UI       11.5px  16px         500      Sidebar files, Netlist nodes
Sub-Label      10.5px  14px         600      FPGA Part badges, Pins, Ports
Micro Tag       9.0px  12px         700      [TOP] badge, [100MHz], [Vdd]
───────────────────────────────────────────────────────────────────────────
Code Editor    13.0px  20px         400      Monaco Verilog / SystemVerilog
Code Inline    11.5px  16px         500      Hex values, Bit indices, Signals
Sim Timestamp  11.0px  14px         700      Time counters (12.450 ns, δ=0)
```

### 5.3. Tabular Numerals & Layout Stability
When running simulation stepping or clock pulsing, timestamps and register values update multiple times per second. Proportional numbers cause horizontal jitter ("wobble"), causing eye fatigue:

```css
/* Mandatory for all numerical readouts, hex displays, and clocks */
.axiom-mono-readout {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums lining-nums;
  letter-spacing: -0.01em;
}
```

---

## 6. Spatial Grid & Compact Density Geometry

Consumer design systems often mandate 8px or 16px minimum paddings. In an EDA tool where screen real estate directly limits the number of visible signals or schematic gates, excessive padding is an anti-pattern.

Axiom adheres to a **4px Base Geometry Grid** with standardized component heights:

```
┌──────────────────────────────────────────────────────────┬───────────┐
│ Component Type                                           │ Height    │
├──────────────────────────────────────────────────────────┼───────────┤
│ Master Application Header                                │ 40px      │
│ In-Panel Switcher Tabs (Schematic / Lab / Waves)         │ 28px      │
│ Monaco Multi-File Tab Strip                              │ 28px      │
│ Secondary Toolbar (Zoom, Fit, Lod, Filter)               │ 26px      │
│ Breadcrumb Bar                                           │ 22px      │
│ Unified Dock Header / Collapsed Status Bar               │ 28px–32px │
│ Tree Node Item (Sidebar / Netlist)                       │ 24px      │
│ Mobile Bottom Navigation Bar                             │ 52px–56px │
└──────────────────────────────────────────────────────────┴───────────┘
```

This strict geometry ensures that across a standard $1920 \times 1080$ display, over **88% of vertical viewport pixels** are dedicated exclusively to hardware design visualization and code inspection.

---

## 7. Standardized Micro-Transitions & Motion Curves

Axiom forbids garish, slow, or distracting animations that impede engineering workflow. All interface transitions strictly adhere to:
- **Standard Duration**: `150ms` (imperceptible latency, eliminates optical pop/snap).
- **Standard Curve**: `cubic-bezier(0.16, 1, 0.3, 1)` (snappy ease-out).
- **Tactile Depression**: Interactive buttons use `transform: scale(0.97)` on `:active` with `0.05s ease`.
- **CSS Utility Classes**:
  - `.btn`: Standardized button base with focus rings, hover fills, and active scale.
  - `.badge`: Micro status pill with color variants (`.badge-cyan`, `.badge-emerald`, `.badge-amber`, `.badge-rose`, `.badge-purple`).
  - `.axiom-card`: Interactive panel container with subtle border lighting and 150ms hover glow.

---

## 8. Precision Vector Brand Identity (`logo.svg`)

Axiom's brand identity reflects precision aerospace and digital logic synthesis:
- **Geometry**: Dual-faceted nested chevrons converging toward a central logic core.
- **Color Gradients**: Precision linear gradient from Neon Cyan (`#00f2fe`) to Hyper Blue (`#3b82f6`).
- **Aspect Ratio**: 1:1 square vector with embedded `defs` gradients and drop shadow filters.
- **Assets**: Rendered natively via SVG in both `ui/public/logo.svg` and documentation `docs/public/logo.svg`.

---

## 9. Global Multi-Language Architecture & Browser Locale Detection

Axiom supports seamless multi-lingual engineering environments without external library overhead:
- **TypeScript Contract**: `Translations` interface in `ui/src/i18n/types.ts` defines all UI tokens.
- **Zero-Desync Two-Stage Deployment**:
  1. All component strings cataloged and verified in English (`en.ts`).
  2. Complete, idiomatic, verified translations generated simultaneously for Turkish (`tr.ts`), German (`de.ts`), Spanish (`es.ts`), French (`fr.ts`), Japanese (`ja.ts`), and Simplified Chinese (`zh.ts`).
- **Dual Accessor (`TFunction`)**: Seamlessly supports both `t("namespace.key")` and typed `t.namespace.key`.
- **Automatic Language Detection**: Uses `navigator.language` to match the user's browser locale with persistent storage in `localStorage.axiom_language`.
