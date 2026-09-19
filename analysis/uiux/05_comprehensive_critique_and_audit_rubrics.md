# Comprehensive UI/UX Critique Rubrics, Usability Heuristics & Anti-Pattern Catalog

## 1. Executive Summary & Audit Methodology

Engineering software and CAD/EDA tools require a rigorous, objective evaluation framework. Traditional consumer usability heuristics often misdiagnose intentional density as "clutter" or fail to account for domain-specific realities such as zero-time delta cycles, multi-radix buses, and sub-millisecond execution loops.

This manual provides an **Industry-Grade Audit Framework** composed of:
1. **Nielsen's 10 Usability Heuristics** adapted specifically for Technical CAD & Developer IDEs.
2. **Shneiderman's 8 Golden Rules of Interface Design** applied to desktop engineering applications.
3. **The 50-Point UI/UX Verification Checklist**.
4. **The Engineering Tool Anti-Pattern Catalog**.
5. **Defect Severity Classification Matrix**.

---

## 2. Adapted Usability Heuristics for Technical CAD & EDA

```
┌──────────────────────────────────────┬─────────────────────────────────────────────────────────┐
│ Heuristic (Jakob Nielsen)            │ Technical CAD / EDA Application Mandate                 │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 1. Visibility of System Status       │ Real-time display of physical simulation time (ps/ns),   │
│                                      │ active delta cycle (δ), JIT compiler status, PDN rail   │
│                                      │ voltage, and active running/paused state.               │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 2. Match Between System & Real World │ Virtual Lab instruments must mirror physical FPGA boards │
│                                      │ (DIP switches, 7-seg displays, LEDs). Electrical logic  │
│                                      │ states must strictly adhere to IEEE 1800 (0, 1, X, Z).  │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 3. User Control & Freedom            │ Instant reset (t=0), checkpoint rollback, panel         │
│                                      │ maximization/restoration, and unconditional modal       │
│                                      │ dismissal via Escape key.                               │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 4. Consistency & Standards           │ Strict compliance with Vivado file sets (`sources_1`,   │
│                                      │ `sim_1`, `constrs_1`), standard Verilog Monarch grammar,│
│                                      │ and consistent border/surface elevation tokens.         │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 5. Error Prevention                  │ Active warning when closing an unsaved project. Disable │
│                                      │ stepping controls when HDL has uncompiled syntax errors.│
│                                      │ Prevent invalid radix input (e.g. 'G' in Hex).          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 6. Recognition Rather Than Recall    │ Omnibar fuzzy finder (`Ctrl+K`), in-RAM autocomplete    │
│                                      │ snippets (`always_ff`, `case`), hover tooltips with     │
│                                      │ port bit-widths, and cross-probing halos.               │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 7. Flexibility & Efficiency of Use   │ Full keyboard accelerator parity (F5, F10, F11),        │
│                                      │ multi-radix bus switching (Hex, Bin, Dec, ASCII), and   │
│                                      │ quick starter hardware templates on the launchpad.      │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 8. Aesthetic & Minimalist Design     │ Dual-pane studio layout eliminating quad-split cramping.│
│                                      │ Collapsible 28px bottom dock and 38px sidebar strip.   │
│                                      │ High density without visual clutter or heavy shadows.   │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 9. Help Users Recognize Errors       │ Linter squiggles paired with plain-English descriptions,│
│                                      │ rule codes (`AXIOM_W001`), line/col coordinates, and    │
│                                      │ 1-click jump-to-source navigation.                      │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 10. Help & Documentation             │ Context-aware tooltips, empty-state onboarding cards,    │
│                                      │ quick-start hardware templates, and direct links to      │
│                                      │ architecture documentation.                             │
└──────────────────────────────────────┴─────────────────────────────────────────────────────────┘
```

---

## 3. Shneiderman's 8 Golden Rules Applied to EDA Tools

1. **Strive for Consistency**: Maintain uniform icon meanings across the application (e.g. Play always starts simulation, Cpu always triggers compile, Maximize2 always expands panel).
2. **Enable Frequent Users to Use Shortcuts**: Provide standard single-key accelerators (`F` for fan-in, `O` for fan-out, `F5` for run, `F11` for delta-step) to keep senior engineers in a flow state.
3. **Offer Informative Feedback**: Every click, switch toggle, or step command must produce immediate optical feedback (<100ms) indicating state change.
4. **Design Dialogs to Yield Closure**: Multi-step operations (such as the Vivado Project Wizard or Add Source modal) must have clear Step 1 $\to$ Step 2 $\to$ Complete pathways with a definitive confirmation card.
5. **Offer Simple Error Handling**: When JIT compilation fails, pinpoint the exact syntax error line with red underlines and direct the user to the exact fix.
6. **Permit Easy Reversal of Actions**: Allow resetting forced signals back to dynamic nets, restoring closed tabs, and clearing temporary waveform probes.
7. **Support Internal Locus of Control**: The user must feel in command of simulation time. Avoid unexpected auto-running; give users explicit manual stepping (`tick(ps)` and `step_delta()`).
8. **Reduce Short-Term Memory Load**: Keep current simulation time, rail voltage, and top module permanently visible in the header so the user never has to guess system context.

---

## 4. The 60-Point Industry-Grade UI/UX Audit Checklist

```
Category 1: Visual Design & Design Tokens (Items 1–8)
[ ] 01. Are all colors derived from centralized CSS variables in theme.css?
[ ] 02. Does background layering follow a logical elevation gradient without muddy grays?
[ ] 03. Are borders subtle (1px var(--border-subtle)) rather than heavy diffuse drop shadows?
[ ] 04. Are semantic signal colors (0, 1, X, Z, Bus, Glitch) universally consistent?
[ ] 05. Is the interface typography strictly partitioned between sans-serif (UI) and monospace (data)?
[ ] 06. Do all numerical readouts use tabular figures (tabular-nums) to prevent layout wobble?
[ ] 07. Are border radii uniform across components (--radius-sm: 4px, --radius-md: 6px)?
[ ] 08. Is pure black (#000000) avoided on large surface areas to prevent OLED smearing?

Category 2: Information Architecture & Layout Density (Items 9–16)
[ ] 09. Does the layout avoid static quad-split crowding, favoring a flexible dual-pane studio?
[ ] 10. Can the editor and visualizer widths be resized smoothly via a draggable splitter?
[ ] 11. Does the splitter enforce safe percentage boundaries (18% minimum, 75% maximum)?
[ ] 12. Can individual panes be maximized to 100% full screen with a single click (⛶)?
[ ] 13. Does the left sidebar collapse into an ultra-compact icon rail (<= 38px)?
[ ] 14. Does the bottom dock collapse into a functional status bar (<= 32px height)?
[ ] 15. Are project sources partitioned according to standard file sets (sources_1, sim_1, constrs_1)?
[ ] 16. Is the top-level module clearly distinguished with an active [TOP] badge?

Category 3: Interaction Ergonomics & State Feedback (Items 17–24)
[ ] 17. Do interactive buttons and toggles provide visual feedback in under 100ms?
[ ] 18. Are disabled controls clearly styled with explanatory tooltips indicating why they are locked?
[ ] 19. Does the header simulation clock update smoothly without text flickering?
[ ] 20. Is the global Omnibar (Ctrl+K) accessible from any view and dismissible via Escape?
[ ] 21. Does the Escape key dismiss nested UI layers in a predictable, priority-ordered stack?
[ ] 22. Are primary simulation shortcuts (F5 Run, F10 Step, F11 Delta, Shift+F5 Reset) functional?
[ ] 23. Does clicking on a schematic gate or timing violation jump directly to the code line?
[ ] 24. Do open file tabs display dirty/modified indicators when unsaved edits exist?

Category 4: Complex Data & Hardware Visualizations (Items 25–32)
[ ] 25. Does the Monaco editor provide syntax highlighting for IEEE 1800 Verilog/SystemVerilog?
[ ] 26. Are in-RAM linter squiggles debounced (200ms) to avoid distracting flicker while typing?
[ ] 27. Does the Schematic DAG automatically compute bounds and center the graph on screen?
[ ] 28. Does the Schematic DAG support semantic Level-of-Detail (LOD) zoom scaling?
[ ] 29. Can engineers isolate critical timing paths using 1-click Logic Cone Slicers (Fan-in / Fan-out)?
[ ] 30. Does the Waveform viewer maintain 60 FPS scrolling and zooming without DOM lag?
[ ] 31. Can multi-bit vector buses be exploded into individual bits and formatted in multiple radixes?
[ ] 32. Are zero-time delta-cycle glitches visually flagged with interactive hazard ribbons?

Category 5: Accessibility & Contrast (Items 33–40)
[ ] 33. Does primary body text achieve a contrast ratio >= 7:1 against its background (WCAG AAA)?
[ ] 34. Does secondary text and metadata achieve a contrast ratio >= 4.5:1 (WCAG AA)?
[ ] 35. Are electrical logic states distinguished by position and texture, not color alone?
[ ] 36. Do interactive elements have distinct :focus-visible outlines for keyboard users?
[ ] 37. Are touch targets on mobile viewports sized at or above 44px x 44px?
[ ] 38. Are screen reader aria-label attributes present on icon-only buttons?
[ ] 39. Is layout readability preserved when browser text zoom is increased to 150%?
[ ] 40. Are destructive operations protected by confirmation dialogs or instant undo?

Category 6: Responsive & Mobile Viewport Ergonomics (Items 41–50)
[ ] 41. Does the interface detect viewports <= 768px and disable desktop splitters completely?
[ ] 42. Does the mobile mode ensure document.body.scrollWidth === window.innerWidth (0px overflow)?
[ ] 43. Is navigation handled via a smooth off-canvas sliding drawer (MobileDrawer.tsx)?
[ ] 44. Does the off-canvas drawer feature a darkened backdrop blur filter?
[ ] 45. Does the mobile view render 1 tool at a time at 100% width and height without clipping?
[ ] 46. Is there a fixed bottom navigation bar with thumb-friendly touch tabs?
[ ] 47. Does the mobile bottom bar include dynamic notification badges for errors and glitches?
[ ] 48. Are mobile drawer selections automatically closed upon navigation?
[ ] 49. Is safe-area inset padding respected for mobile device notches and home bars?
[ ] 50. Does the Welcome Launchpad render clean single-column cards on narrow screens?

Category 7: Primitive Componentization & Zero Native Bleed (Items 51–60)
[ ] 51. Are 100% of interactive controls composed from abstract UI primitives (ui/src/components/ui/)?
[ ] 52. Are raw browser-native <select> tags completely eliminated in favor of custom Popover Selects?
[ ] 53. Do all dropdowns support standardized density tiers: xs (24px), sm (28px), and md (34px)?
[ ] 54. Do dropdowns support categorized option groups (SelectGroup) for logical grouping (e.g. Inputs vs Outputs)?
[ ] 55. Do select options support rich metadata (custom left icons, country flags, right-hand bit-width badges)?
[ ] 56. Do custom popovers support viewport-safe boundary alignment (align="left" | "right")?
[ ] 57. Is full keyboard parity implemented for custom dropdowns (ArrowUp/ArrowDown, Enter, Space, Escape)?
[ ] 58. Do text input fields provide inline clearable action buttons (✕) and left icon slots?
[ ] 59. Are custom scrollbars (.custom-scrollbar) applied to all popovers and overflow containers?
[ ] 60. Do button primitives provide tactile haptic micro-scaling (:active: scale(0.97)) and semantic color tokens?
```

---

## 5. Engineering Tool Anti-Pattern Catalog

```
┌──────────────────────────────────────┬─────────────────────────────────────────────────────────┐
│ Anti-Pattern                         │ Cause, Manifestation & Axiom Architectural Fix          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 1. Mystery Meat Navigation           │ Icons without text labels or tooltips. Users are forced │
│                                      │ to guess what an abstract symbol does.                  │
│                                      │ Fix: Every icon button must have a concise, descriptive │
│                                      │ title attribute and tooltip showing shortcut keys.      │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 2. Quad-Split Claustrophobia         │ Forcing code, waveforms, netlists, and consoles into    │
│                                      │ four tiny static boxes simultaneously.                  │
│                                      │ Fix: Spacious Dual-Pane Studio with 1-click Maximize    │
│                                      │ (⛶) and collapsible drawer docks.                       │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 3. Contrast Starvation               │ Using dark-gray-on-black text (#475569 on #0d0f12) for  │
│                                      │ fine technical metadata, causing severe eye strain.     │
│                                      │ Fix: Strict WCAG AA enforcement; secondary labels must  │
│                                      │ maintain >= 4.5:1 relative contrast (#94a3b8).          │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 4. Modal Traps                       │ Dialog windows that ignore Escape, trap tab focus, or   │
│                                      │ lack prominent close buttons.                           │
│                                      │ Fix: Unified stacked Escape handler and backdrop clicks │
│                                      │ that unconditionally restore the underlying workspace.  │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 5. Dead-End Empty States             │ Blank white/gray canvas when no project is loaded.      │
│                                      │ Fix: Aerospace-grade Welcome Launchpad with hero banner,│
│                                      │ project wizard card, and 1-click starter templates.     │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 6. The Number Jitter Wobble          │ Using proportional fonts for rapidly incrementing time  │
│                                      │ counters or registers, causing horizontal layout shake. │
│                                      │ Fix: Mandatory font-variant-numeric: tabular-nums on    │
│                                      │ all monospace clocks, addresses, and measurements.      │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 7. Accidental Viewport Scrolling     │ Unpinned body elements allowing entire IDE window to    │
│                                      │ scroll off-screen on touch devices or trackpads.        │
│                                      │ Fix: html, body, #root fixed at 100vw, 100dvh, overflow:│
│                                      │ hidden with internal virtualized scroll containers.     │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 8. The Leaky Native Control          │ Rendering un-styled, default browser-native <select> or │
│                                      │ raw <input> tags that leak bright white OS menus,       │
│                                      │ clunky platform chrome, and break dark mode immersion. │
│                                      │ Fix: Encapsulate all interactive materials into         │
│                                      │ abstract primitives (ui/src/components/ui/Select.tsx)   │
│                                      │ with dark acrylic glassmorphism and custom scrollbars.  │
├──────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 9. Concrete Primitive Sprawl         │ Inlining repetitive custom <div> and <button> styling   │
│                                      │ across disparate components instead of composing from   │
│                                      │ centralized design system primitives (<Button>, <Card>).│
│                                      │ Fix: Enforce strict single-source-of-truth UI primitive │
│                                      │ imports from ui/src/components/ui/.                     │
└──────────────────────────────────────┴─────────────────────────────────────────────────────────┘
```

---

## 6. Defect Severity Classification Matrix

When auditing technical applications, classify all discovered UI/UX issues into one of five standardized severity tiers:

```
Severity Level         Impact on Engineer                      Remediation SLA
───────────────────────────────────────────────────────────────────────────────
P0 — Blocker           Prevents hardware design, simulation,   Immediate hotfix.
                       or compilation. Critical layout crash.
                       
P1 — Critical          Severe usability failure: unreadable    Must fix before
     Usability         contrast (<3:1), broken mobile layout,  production release.
                       or irreversible project data loss.
                       
P2 — Moderate          Workflow friction: cramped splitter,   Target for next sprint
     Friction          missing tooltips, small touch hitboxes, or milestone phase.
                       or confusing disabled button feedback.
                       
P3 — Minor             Visual polish: 1px alignment offsets,   Polish backlog.
     Polish            subtle hover timing, missing dirty
                       save indicator, or tab overflow menus.
                       
P4 — Cosmetic          Micro-aesthetic nuances: icon weight    Opportunistic fix.
                       balance or gradient saturation tweaks.
───────────────────────────────────────────────────────────────────────────────
```
