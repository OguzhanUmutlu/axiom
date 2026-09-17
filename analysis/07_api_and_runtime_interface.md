# Axiom API, Runtime Interface & WebAssembly Bindings

## 1. Overview and Integration Scope

Axiom is designed from the core as an **embeddable library**. The simulation engine can be driven via:
1. **Native Rust API**: For maximum performance and integration with Rust tools.
2. **C-ABI Dynamic Library (`libaxiom.so` / `.dll` / `.dylib`)**: For integration with Python, C++, or custom testbenches.
3. **WebAssembly (`wasm-bindgen`)**: For running directly inside modern web browsers with TypeScript/JavaScript.

---

## 2. Public Rust Engine API

```rust
use axiom_core::{Logic4, SimTime};

pub struct AxiomEngine {
    sim: AxiomSimulator,
    telemetry: TelemetryCollector,
}

impl AxiomEngine {
    /// Ingests HDL source code and compiles it directly into executable RAM
    pub fn compile_and_elaborate(source: &str, top_module: &str) -> Result<Self, EngineError>;

    /// Advances physical simulation time by delta duration (e.g. 100ns)
    pub fn tick(&mut self, dt: SimTime) -> Result<TickSummary, EngineError>;

    /// Advances simulation by exactly ONE discrete delta cycle (zero physical time)
    pub fn step_delta(&mut self) -> Result<DeltaSummary, EngineError>;

    /// Queries the current 4-state value of a hierarchical signal path
    pub fn get_signal_value(&self, path: &str) -> Option<LogicVector>;

    /// Forces a signal to a specific value (user stimulus override)
    pub fn force_signal(&mut self, path: &str, value: LogicVector) -> Result<(), EngineError>;

    /// Releases a forced signal override
    pub fn release_signal(&mut self, path: &str) -> Result<(), EngineError>;

    /// Retrieves latest voltage, current, and power telemetry frame
    pub fn poll_telemetry(&mut self) -> TelemetryFrame;

    /// Captures a full snapshot of simulation state in RAM for instant rewind
    pub fn create_checkpoint(&self) -> CheckpointId;

    /// Restores simulation state to an earlier checkpoint
    pub fn restore_checkpoint(&mut self, id: CheckpointId) -> Result<(), EngineError>;
}
```

---

## 3. WebAssembly (WASM) & TypeScript Interface

The `axiom-wasm` crate exposes the engine to the browser via `wasm-bindgen`:

```rust
#[wasm_bindgen]
pub struct WasmAxiomEngine {
    inner: AxiomEngine,
}

#[wasm_bindgen]
impl WasmAxiomEngine {
    #[wasm_bindgen(constructor)]
    pub fn new(hdl_source: &str, top_module: &str) -> Result<WasmAxiomEngine, JsValue> {
        let inner = AxiomEngine::compile_and_elaborate(hdl_source, top_module)
            .map_err(|e| JsValue::from_str(&e.to_string()))?;
        Ok(Self { inner })
    }

    #[wasm_bindgen]
    pub fn tick(&mut self, dt_picoseconds: f64) -> Result<JsValue, JsValue> {
        let dt = SimTime::from_picoseconds(dt_picoseconds as u64);
        let summary = self.inner.tick(dt).map_err(|e| JsValue::from_str(&e.to_string()))?;
        Ok(serde_wasm_bindgen::to_value(&summary)?)
    }

    #[wasm_bindgen]
    pub fn step_delta(&mut self) -> Result<JsValue, JsValue> {
        let summary = self.inner.step_delta().map_err(|e| JsValue::from_str(&e.to_string()))?;
        Ok(serde_wasm_bindgen::to_value(&summary)?)
    }

    #[wasm_bindgen]
    pub fn get_signal(&self, path: &str) -> Option<String> {
        self.inner.get_signal_value(path).map(|v| v.to_string())
    }

    #[wasm_bindgen]
    pub fn get_telemetry_batch(&mut self) -> Result<JsValue, JsValue> {
        let frame = self.inner.poll_telemetry();
        Ok(serde_wasm_bindgen::to_value(&frame)?)
    }
}
```

In the React/TypeScript application:
```typescript
import init, { WasmAxiomEngine } from 'axiom-wasm';

await init();
const engine = new WasmAxiomEngine(verilogCode, "alu_top");

// Step 10 nanoseconds
const summary = engine.tick(10000); 

// Step a single delta cycle
const deltaResult = engine.step_delta();
const powerData = engine.get_telemetry_batch();
```

---

## 4. Telemetry Streaming Protocols

1. **Native Desktop (Tauri v2)**:
   - Uses zero-copy IPC buffers between Rust and Tauri webview.
   - 60 FPS update loop with requestAnimationFrame synchronization.
2. **Web Browser (WASM)**:
   - Runs simulation inside a dedicated `Web Worker` to keep the main UI thread at a silky-smooth 120 FPS.
   - SharedArrayBuffer / postMessage exchange for telemetry transfer.
