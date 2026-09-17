# Axiom Engine Architecture & Workspace Organization

## 1. Architectural Philosophy

The Axiom engine is designed around three foundational engineering principles:
1. **Zero-Overhead Memory Safety**: Built 100% in safe Rust, eliminating memory leaks, buffer overflows, and segmentation faults that plague legacy C/C++ EDA tools.
2. **Instant In-RAM Compilation**: Eliminates external compilers (GCC/Clang) and multi-step disk serialization. All ASTs, netlists, and JIT-compiled machine code reside directly in RAM.
3. **Embeddable & Modular**: The engine operates as an independent, headless library with clean C-ABI and WebAssembly bindings, decoupled from the Tauri desktop UI.

---

## 2. Cargo Multi-Crate Workspace Layout

To maintain strict modularity, high build parallelization, and clean dependency boundaries, the engine is structured as a Cargo workspace:

```
axiom/
├── Cargo.toml                      # Workspace root configuration
├── crates/
│   ├── core/                       # Foundational types (SimTime, 4-state logic, spans, diagnostics)
│   ├── syntax/                     # Streaming lexer, preprocessor, Pratt AST parser
│   ├── ir/                         # Elaborator, symbol tables, and BIR (Axiom IR)
│   ├── jit/                        # Cranelift JIT backend & machine code memory manager
│   ├── sim/                        # Stratified event queue, delta cycle engine & tick API
│   ├── telemetry/                  # Power, energy, voltage calculations, VCD & SAIF exporters
│   ├── desktop/                    # DesktopEngine library & IPC interface
│   └── cli/                        # Unified CLI & In-RAM GUI server (embedded Zstd UI bundle)
├── ui/                             # React 19 + TypeScript + PostCSS interactive studio
└── docs/                           # VitePress documentation portal (axiom.aerovex.net)
```

---

## 3. Memory Model & Zero-Allocation Principles

HDL designs generate millions of tiny AST nodes, net references, and event objects. Traditional heap allocations (`Box`, `Vec`, `malloc`) cause severe memory fragmentation and pointer chasing. Axiom solves this using specialized memory management:

### 3.1. Arena Allocation (`bumpalo`)
- During parsing and elaboration, AST nodes and netlist gates are allocated inside an arena allocator (`bumpalo::Bump`).
- Deallocation of the entire AST or elaboration phase is an instantaneous $O(1)$ pointer reset, completely bypassing individual drop calls.

### 3.2. String Interning (`lasso`)
- Identifiers (signal names, module names, parameters) are interned into compact 32-bit `Spur` tokens via `lasso::Rodeo`.
- Comparing module or net names becomes a single 32-bit integer comparison instead of heap string comparisons (`strcmp`).

### 3.3. Contiguous State Arena for Simulation
- At runtime, the state of all signals, nets, and registers in the elaborated circuit is packed into a contiguous byte array:
  ```rust
  pub struct SimStateArena {
      /// Contiguous buffer holding primary 4-state logic values
      pub values: Box<[u64]>,
      /// Contiguous buffer holding unknown/high-impedance mask bits
      pub masks: Box<[u64]>,
  }
  ```
- This guarantees maximum CPU L1/L2 cache line locality during event evaluation.

---

## 4. Diagnostics & Error Reporting

Axiom replaces cryptic Vivado error messages with rich, modern diagnostics:
- Built with source-span tracking: every token preserves its exact UTF-8 byte offset, line number, and column.
- Colored error messages with underlined offending code snippets, context notes, and actionable suggestions.
- Diagnostic errors serialize cleanly to JSON for display in the Tauri/React IDE code editor.

---

## 5. Concurrency & Multi-Threading Model

- **Parsing & Elaboration**: Parallelized across files and module hierarchies using `rayon`. Independent modules compile in parallel without locks.
- **Simulation Kernel**: Operates on a lock-free, deterministic single-thread event loop for the primary delta cycle, ensuring 100% reproducible bit-level hardware behavior.
- **Telemetry & Rendering**: Offloaded to dedicated background threads via crossbeam channels and lock-free atomic ring buffers (`atomic-ringbuffer`), ensuring that telemetry calculation never throttles the simulation kernel.
