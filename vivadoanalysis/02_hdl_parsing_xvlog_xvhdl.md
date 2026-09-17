# Vivado HDL Parsing Subsystem: `xvlog` and `xvhdl`

## 1. Introduction and Scope

In AMD Vivado, front-end compilation is divided into two primary standalone CLI tools:
- **`xvlog`**: The Verilog and SystemVerilog compiler.
- **`xvhdl`**: The VHDL compiler.

These tools are invoked during the "Analyze" phase of the simulation flow. Their responsibility is to ingest raw HDL source code, perform pre-processing (macro expansion, conditional inclusion), parse the grammar according to IEEE standards, construct an Abstract Syntax Tree (AST), perform preliminary semantic checks, and serialize the parsed intermediate representation into compiled design libraries on disk (by default, `xsim.dir/work` or `xsim.dir/xil_defaultlib`).

---

## 2. Standards Compliance & Language Support

### 2.1. `xvlog` Language Standards
`xvlog` supports:
- **IEEE 1364-1995** (Verilog-95)
- **IEEE 1364-2001** (Verilog-2001)
- **IEEE 1364-2005** (Verilog-2005)
- **IEEE 1800-2012 / IEEE 1800-2017** (SystemVerilog) via the `-sv` switch.

#### Supported Synthesizable & Behavioral Constructs:
1. **Module & Interface Declarations**:
   - `module ... endmodule`, `interface ... endinterface`.
   - ANSI and non-ANSI port declarations (`input`, `output`, `inout`, `ref`).
2. **Data Types**:
   - 4-state types: `wire`, `reg`, `logic`, `integer`, `time`.
   - 2-state types: `bit`, `byte`, `shortint`, `int`, `longint`.
   - Multi-dimensional packed and unpacked arrays.
   - User-defined types (`typedef struct`, `typedef enum`, `typedef union`).
3. **Continuous Assignments**:
   - Explicit `assign` statements with optional drive strengths and delays.
4. **Procedural Blocks**:
   - `always`, `initial`, `always_comb`, `always_ff`, `always_latch`, `final`.
   - Blocking (`=`) and non-blocking (`<=`) procedural assignments.
5. **Flow Control**:
   - `if-else`, `case`, `casex`, `casez`, `unique case`, `priority case`.
   - Loops: `for`, `while`, `repeat`, `forever`, `do-while`.
6. **Pre-processor Directives**:
   - `` `define ``, `` `undef ``, `` `ifdef ``, `` `ifndef ``, `` `else ``, `` `elsif ``, `` `endif ``.
   - `` `include `` (file inclusion from include directories specified via `-i`).
   - `` `timescale <unit> / <precision> `` (sets default simulation time unit and rounding precision).
   - `` `default_nettype ``, `` `line ``, `` `resetall ``.

### 2.2. `xvhdl` Language Standards
`xvhdl` supports:
- **IEEE 1076-1993** (default)
- **IEEE 1076-2002**
- **IEEE 1076-2008** via the `-2008` switch.

#### Key Features:
- Entities, Architectures, Packages, and Configurations.
- Standard IEEE libraries (`ieee.std_logic_1164`, `ieee.numeric_std`, `ieee.std_logic_arith`).
- Concurrent statements (`process`, concurrent signal assignment, component instantiation).

---

## 3. The Front-End Parsing Architecture

Vivado's front-end parsing operates through traditional multi-pass phases:

```
Source Code (.v / .sv)
       |
       v
+-------------------------------------------------------+
| 1. Preprocessor Engine                                |
|    - Macro expansion (`define)                        |
|    - Conditional evaluation (`ifdef/`ifndef)          |
|    - File inclusion (`include)                        |
|    - Strips comments (// and /* ... */)               |
+-------------------------------------------------------+
       |
       v Token Stream
+-------------------------------------------------------+
| 2. Lexical Analyzer (Scanner)                         |
|    - Identifies identifiers, keywords, numbers        |
|    - Evaluates sized integer literals: 32'hDEAD_BEEF  |
|    - Preserves source file and line tracking spans    |
+-------------------------------------------------------+
       |
       v Token Stream + Source Map
+-------------------------------------------------------+
| 3. Syntactic Parser (LR / LALR Grammar Engine)        |
|    - Context-free grammar matching                    |
|    - Constructs Abstract Syntax Tree (AST)            |
|    - Syntax error recovery and error reporting        |
+-------------------------------------------------------+
       |
       v Raw AST
+-------------------------------------------------------+
| 4. Semantic Checker & Symbol Table Builder            |
|    - Validates local scope identifiers                |
|    - Checks type consistency and wire width rules     |
|    - Binds compile-time constants (localparam)        |
+-------------------------------------------------------+
       |
       v Validated Intermediate AST
+-------------------------------------------------------+
| 5. Library Serializer                                 |
|    - Dumps compiled AST / symbol tables to disk       |
|    - Directory: xsim.dir/<library_name>/              |
+-------------------------------------------------------+
```

---

## 4. `xvlog` Command-Line Interface & Configuration Options

| Option | Argument | Description |
| :--- | :--- | :--- |
| `-sv` | *None* | Enables SystemVerilog parsing mode (default is Verilog-2001). |
| `-work` | `<libname>` | Target logical library (defaults to `xil_defaultlib` or `work`). |
| `-i` / `-include` | `<dir_path>` | Adds an include search directory for `` `include `` files. |
| `-d` / `-define` | `<macro>[=<val>]` | Defines a preprocessor macro globally. |
| `-nolog` | *None* | Suppresses generation of `xvlog.log`. |
| `-relax` | *None* | Relaxes strict IEEE compliance checking (e.g. allows lenient port binding). |
| `-sourcelibdir` | `<dir_path>` | Directory to search for unresolved module definitions. |
| `-sourcelibext` | `<ext>` | Extensions for resolving module files (`.v`, `.sv`). |

---

## 5. Architectural Pain Points & Inefficiencies in Vivado's Parsing

1. **Disk I/O Bottleneck**:
   Vivado serializes intermediate parsed symbols directly to disk in `xsim.dir/`. For multi-file designs with hundreds of Verilog modules, disk writes and subsequent disk reads in `xelab` introduce significant latency.
2. **Lack of Incremental In-Memory Caching**:
   Re-running simulation requires either rebuilding the library or relying on coarse-grained file timestamp checking.
3. **Cryptic Diagnostic Messages**:
   Vivado error messages often report parser failures with ambiguous line spans, lacking modern Rust-style annotated code snippets with colorized underline markers.
4. **Thread-Safety & Multi-Threading Limitations**:
   `xvlog` compiles files sequentially or with limited multi-threading, failing to saturate modern multi-core CPUs during parsing of large codebases.

---

## 6. Design Implications for Axiom

In Axiom:
- **Zero Disk Serialization for ASTs**: Parsing occurs in RAM using high-performance arena allocators (`bumpalo`).
- **Parallel Compilation**: Files and modules are parsed concurrently across all CPU threads using `rayon`.
- **Rich Diagnostics**: Diagnostics utilize terminal and web-renderable spans (`miette`-style) with exact source code excerpts and actionable suggestions.
- **Unified Verilog/SystemVerilog Parser**: Clean recursive-descent parser with Pratt parsing for expressions, eliminating heavy LALR parser generator overhead.
