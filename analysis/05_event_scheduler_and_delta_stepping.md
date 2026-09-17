# Betterado Stratified Event Scheduler & Manual Delta-Time Tick API

## 1. Executive Summary

The simulation kernel in `betterado-sim` is responsible for driving simulated physical time and executing digital transitions with mathematical determinism.

Betterado implements:
1. Full compliance with the **IEEE 1800 Stratified Event Queue** specification.
2. A breakthrough **Manual Delta-Time Tick & Step API**, giving callers total control over physical time increments ($\Delta t$), discrete delta cycles ($\delta$), and individual event firings.

---

## 2. Event Queue Architecture

The simulation engine maintains a priority queue of scheduled events sorted by:
$$\text{EventKey} = (\text{SimTime}, \ \text{RegionPriority}, \ \text{DeltaCycleId})$$

```
+------------------------------------------------------------------+
|                    Priority Event Queue (Min-Heap)               |
+------------------------------------------------------------------+
| [Time: 10.000 ns, Region: Active, Delta: 0] -> Eval clk posedge |
| [Time: 10.000 ns, Region: NBA,    Delta: 0] -> Update q1 <= d    |
| [Time: 15.000 ns, Region: Active, Delta: 0] -> Toggle reset      |
+------------------------------------------------------------------+
```

### 2.1. Stratified Execution Pipeline

Within any given time step $t$:
1. **Active Region**:
   - Executes all continuous assignment equations.
   - Executes blocking procedural statements.
   - Evaluates Right-Hand-Side (RHS) of non-blocking assignments (NBAs).
2. **Inactive Region**:
   - Executes explicit `#0` delay statements.
3. **NBA Region**:
   - Updates Left-Hand-Side (LHS) variables for non-blocking assignments.
   - **Critical Transition**: If an NBA update changes a signal's value, it checks the sensitivity graph (`sensitivity_map`) and enqueues newly triggered processes into the **Active Region** of the **Next Delta Cycle** ($\delta + 1$) at the same time $t$.
4. **Postponed Region**:
   - Samples values for waveform tracing, VCD dumps, and telemetry calculations.

---

## 3. The Manual Delta-Time Tick & Step API

Unlike Vivado (which only exposes coarse-grained `run 100ns` commands), Betterado exposes an embeddable, fine-grained control API:

```rust
pub struct BetteradoSimulator {
    state: SimStateArena,
    circuit: BirCircuit,
    event_queue: StratifiedEventQueue,
    current_time: SimTime,
    current_delta: u32,
    listeners: Vec<Box<dyn SimEventListener>>,
}

impl BetteradoSimulator {
    /// Advances physical simulation time by a specified delta duration (e.g. 10ns).
    /// Executes all scheduled events and delta cycles within this duration.
    pub fn tick(&mut self, delta_time: SimTime) -> SimResult {
        let target_time = self.current_time + delta_time;
        while let Some(event) = self.event_queue.peek() {
            if event.time > target_time {
                break;
            }
            self.step_event()?;
        }
        self.current_time = target_time;
        self.current_delta = 0;
        Ok(SimResult::AdvancedTo(self.current_time))
    }

    /// Advances simulation by exactly ONE discrete delta cycle (zero physical time).
    /// Allows inspecting intermediate glitches before combinational logic settles!
    pub fn step_delta(&mut self) -> SimResult {
        let current_delta = self.current_delta;
        
        // Process Active region for current delta
        self.process_active_region()?;
        
        // Process NBA region (applies flip-flop outputs)
        let changed_nets = self.process_nba_region()?;
        
        // Trigger downstream sensitivity
        for net in changed_nets {
            self.trigger_sensitive_processes(net);
            self.notify_signal_change(net);
        }
        
        self.current_delta += 1;
        Ok(SimResult::DeltaStepped {
            time: self.current_time,
            delta: self.current_delta,
        })
    }

    /// Advances to the very next scheduled event in the queue.
    pub fn step_event(&mut self) -> SimResult {
        // Pops and executes single lowest-timestamp event
        ...
    }
}
```

---

## 4. Deep Inspection & Glitch Detection

Because Betterado exposes `step_delta()`:
- The UI can step through a clock edge one delta cycle at a time.
- If a signal toggles multiple times within the same time step ($0 \to 1 \to 0$), the engine detects this **combinational glitch**.
- The engine computes the glitch energy dissipation and alerts the user with a visual indicator in the waveform viewer.

---

## 5. Event Callbacks & Hooks

Callers can register zero-cost hooks:
```rust
pub trait SimEventListener: Send + Sync {
    fn on_signal_change(&mut self, net: NetId, new_val: Logic4, time: SimTime, delta: u32);
    fn on_delta_cycle_finished(&mut self, time: SimTime, delta: u32);
    fn on_glitch_detected(&mut self, net: NetId, time: SimTime);
}
```
This powers the live waveform streaming and telemetry graphing engines without polling.
