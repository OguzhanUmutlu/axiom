# Vivado Elaboration Subsystem: `xelab`

## 1. Role of Elaboration in Hardware Simulation

In standard EDA flows, HDL source files describe modular, parameterized templates of hardware units (e.g., `module`, `entity`). They do not represent a concrete circuit until they are **elaborated**.

**`xelab`** is Vivado's elaboration engine. It serves as the design linker and topology builder. It takes the parsed ASTs produced by `xvlog` and `xvhdl`, designates a root top-level module (or testbench), recursively instantiates sub-modules, evaluates compile-time parameters, binds signals across module boundaries, unrolls `generate` loops, evaluates conditions, and links the resulting concrete design hierarchy into an executable **simulation snapshot**.

```
Compiled Libraries (xsim.dir/work, etc.)
                   |
                   v
+-----------------------------------------------------------------+
|                       xelab Entry Point                         |
|   Inputs: Top Module Name (e.g. tb_top), Generics, Defines      |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 1. Hierarchical Instantiation & Scope Tree Construction        |
|    - Instantiates top-level module                              |
|    - Recursively traverses sub-module instances                 |
|    - Constructs hierarchical naming paths (e.g. /tb_top/u_alu)  |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 2. Parameter & Generic Propagation                             |
|    - Overrides parameters from parent instantiations            |
|    - Resolves `defparam` statements                             |
|    - Evaluates constant expressions (clog2, bit-widths, arrays) |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 3. Generate Block Elaboration                                   |
|    - Unrolls `generate for` loops based on evaluated limits     |
|    - Evaluates `generate if` and `generate case` branches       |
|    - Creates unique elaborated scopes (e.g. genblk1[0], etc.)   |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 4. Port Binding & Net Resolution                                |
|    - Connects actual nets to formal ports                       |
|    - Checks port directionality (input, output, inout)          |
|    - Verifies width compatibility and performs automatic padding|
|    - Resolves multi-driver contention rules                     |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 5. Process & Continuous Assignment Lowering                     |
|    - Extracts sensitivity lists for always/process blocks       |
|    - Lowers continuous assigns into dataflow network            |
|    - Builds dependency graph for event scheduler                |
+-----------------------------------------------------------------+
                   |
                   v
+-----------------------------------------------------------------+
| 6. Code Generation & Snapshot Assembly                          |
|    - Emits C/C++ or machine code modules                        |
|    - Links with xsim simulation kernel runtime                  |
|    - Output: xsim.dir/<snapshot_name>/xsimk                     |
+-----------------------------------------------------------------+
```

---

## 2. Core Elaboration Mechanics in `xelab`

### 2.1. Parameter Evaluation & Constant Folding
In Verilog and SystemVerilog:
```systemverilog
module fifo #(
    parameter int DATA_WIDTH = 32,
    parameter int DEPTH = 16,
    localparam int ADDR_WIDTH = $clog2(DEPTH)
)(
    input  logic                   clk,
    input  logic [DATA_WIDTH-1:0] din,
    output logic [DATA_WIDTH-1:0] dout
);
```
During elaboration:
1. `xelab` binds the instance-specific parameter values provided by the parent instance:
   `fifo #(.DATA_WIDTH(64), .DEPTH(32)) u_fifo (...)`
2. `xelab` evaluates constant functions (such as `$clog2(32) = 5`).
3. Port widths `[DATA_WIDTH-1:0]` are concretized to `[63:0]`.
4. `ADDR_WIDTH` is sealed as constant `5`.

### 2.2. Generate Blocks Unrolling
Generate blocks allow dynamic hardware generation based on parameter values:
```systemverilog
genvar i;
generate
    for (i = 0; i < NUM_CHANNELS; i = i + 1) begin : gen_channel
        channel_unit #(.ID(i)) u_chan (
            .clk(clk),
            .data_in(data_bus[i*8 +: 8]),
            .data_out(out_bus[i*8 +: 8])
        );
    end
endgenerate
```
`xelab` unrolls this loop at elaboration time into $N$ distinct concrete scopes:
- `tb_top.gen_channel[0].u_chan`
- `tb_top.gen_channel[1].u_chan`
- ...
- `tb_top.gen_channel[N-1].u_chan`

Each scope has its own isolated variables, state registers, and evaluation processes.

### 2.3. Net Resolution & Multi-Driver Conflict
In RTL designs, multiple drivers on a net must be resolved:
- **`wire` nets**: Resolved using 4-state resolution tables (wired-AND, wired-OR, or tri-state `Z` contention resulting in `X` if two active drivers conflict).
- **`logic` / `reg` nets**: In SystemVerilog, variables are restricted to a single continuous assignment or procedural assignment driver. `xelab` strictly validates that no multiple drivers exist on variable types, raising fatal elaboration errors:
  `ERROR: [VRFC 10-718] variable 'dout' driven by multiple drivers`.

---

## 3. `xelab` Snapshot Generation

The final output of `xelab` is a **simulation snapshot** stored inside `xsim.dir/<snapshot_name>/`:
- `xsimk`: The executable or shared object binary. In standard Vivado flows, `xelab` generates C/C++ code representing the design's procedural blocks and net connections, then calls the host C compiler (GCC/Clang or Vivado's embedded compiler) to produce native object code linked against `librdi_simulator_kernel.so`.
- `xsim.cov`: Coverage instrumentation metadata (if `-coverage` is passed).
- `xsim.type`: Metadata describing signal names, types, widths, and hierarchical scopes for debugging in `xsim`.

---

## 4. `xelab` CLI Reference & Options

| Option | Parameter | Description |
| :--- | :--- | :--- |
| `-top` | `<top_name>` | Specifies top-level module/entity. |
| `-s` / `-snapshot` | `<snap_name>` | Assigns the name for the output simulation snapshot. |
| `-generic_top` | `<name>=<val>` | Overrides top-level VHDL generic / Verilog parameter. |
| `-defparam` | `<path>=<val>` | Overrides parameter at hierarchical path. |
| `-timescale` | `<unit>/<prec>` | Default simulation timescale if unspecified in sources. |
| `-relax` | *None* | Relaxes strict type binding rules during elaboration. |
| `-debug` | `typical`/`all` | Embeds signal debug symbols for waveform inspection. |
| `-O0` / `-O2` / `-O3` | *None* | Compiler optimization levels for the generated snapshot. |
| `-mt` | `auto`/`1..8` | Multi-threading level for elaboration linking. |

---

## 5. Axiom Elaboration Innovations

In Axiom:
1. **Direct IR Lowering**: Instead of generating intermediary C/C++ files and invoking GCC (which takes 5–30 seconds), Axiom lowers directly to a flat SSA-like hardware Intermediate Representation (**BIR**).
2. **Instant In-RAM Snapshot**: Elaboration is purely an in-memory transformation. The resulting BIR graph is handed directly to Cranelift for JIT machine-code compilation in milliseconds.
3. **Interactive Hierarchy Inspector**: The hierarchical scope tree built during elaboration is exposed directly to the Tauri frontend via JSON/WASM, enabling live browsing of module trees, port widths, and net connectivity.
