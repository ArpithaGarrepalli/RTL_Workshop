# 🔧 Module 4 — Blocking Assignments & Synthesis-Simulation Mismatch

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the [RTL Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop) series.

## 📖 Overview

Whether a statement inside an `always` block uses `=` or `<=` isn't a stylistic choice — it changes how the simulator executes that statement, and it can change what hardware synthesis actually builds. This module works through that gap directly: a multiplexer built two ways, one correct and one subtly broken, simulated and synthesized side by side, followed by a closer look at exactly how blocking assignments execute so the failure mode makes sense rather than just being a rule to memorize.

## 🎯 Objectives

- Understand how blocking (`=`) and non-blocking (`<=`) assignments differ in execution.
- Identify what causes synthesis-simulation mismatch and how to spot it in a waveform.
- Build and simulate a multiplexer using the ternary operator.
- Trace blocking-assignment execution order statement by statement.
- Verify RTL behavior using Icarus Verilog and GTKWave.
- Synthesize designs with Yosys and map them onto SKY130 standard cells.
- Compare simulated behavior against synthesized hardware directly.

## 🛠️ Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| Verilog HDL | RTL design |
| Icarus Verilog | Compiling and simulating designs |
| GTKWave | Waveform inspection |
| Yosys | RTL synthesis |
| SKY130 Standard-Cell Library | Technology mapping |
| gVim | Viewing and editing Verilog source |
| Linux Terminal | Command execution |

## 📑 Table of Contents

- 1. Building and Verifying a Correct Multiplexer
  - 1.1 Simulating the Ternary MUX
  - 1.2 Mapping the MUX to Silicon
  - 1.3 Confirming Behavior Across Inputs
- 2. Diagnosing a Broken Multiplexer
  - 2.1 First Signs of Trouble
  - 2.2 Tracing the Latch
- 3. Understanding Blocking Assignment Execution
  - 3.1 Watching Execution Order in Simulation
  - 3.2 Confirming the Synthesized Result
  - 3.3 Same-Block Value Dependency
- 4. Results at a Glance
- 5. Takeaways
- Author

---

## 1️⃣ Building and Verifying a Correct Multiplexer

### 1.1 Simulating the Ternary MUX

A 2:1 multiplexer written as a single ternary expression is about as close to unambiguous combinational logic as Verilog gets — there's no branch that can be left unspecified.

```bash
iverilog -o mux ternary_operator_mux.v tb_ternary_operator_mux.v
gtkwave ternary_operator_mux.vcd
```

<img width="1918" height="1018" alt="ternarymux" src="https://github.com/user-attachments/assets/a3300a2d-10a9-49b6-b774-c727b5e6887f" />

**Observation:** the output tracks the selected input cleanly, confirming the RTL behaves as intended before synthesis even enters the picture.

### 1.2 Mapping the MUX to Silicon

The same design was pushed through Yosys and mapped onto the SKY130 library to see what hardware it actually resolves to.

```bash
yosys
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1918" height="1020" alt="ternaryoperatormux" src="https://github.com/user-attachments/assets/b42e463d-a6ef-440c-9b9b-f5991768a478" />


**Observation:** Yosys mapped the design directly onto a single SKY130 multiplexer cell — no extra logic, no latch, exactly what a fully-specified ternary expression should produce.

### 1.3 Confirming Behavior Across Inputs

To rule out the possibility that the first simulation just got lucky with its test vectors, the design was re-run against a wider set of input/select combinations.

```bash
iverilog -o mux mux_generate.v tb_mux_generate.v
gtkwave mux_generate.vcd
```

<img width="958" height="930" alt="ternarymux1" src="https://github.com/user-attachments/assets/b84afc97-d703-4b47-8051-ca1830e8dfbc" />


**Observation:** every combination tested tracked correctly. This confirms the baseline this module compares everything else against.

---

## 2️⃣ Diagnosing a Broken Multiplexer

### 2.1 First Signs of Trouble

A second multiplexer was written with an incomplete `always` block — the kind of gap that's easy to miss reading through the code, but that a simulator and a synthesis tool won't handle the same way.

```bash
iverilog -o bad_mux bad_mux.v tb_bad_mux.v
gtkwave bad_mux.vcd
```

<img width="1918" height="1020" alt="bad_mux" src="https://github.com/user-attachments/assets/dd5bc0dc-fc6e-4d84-921a-0b361fbce7bb" />


**Observation:** the output no longer follows the select signal cleanly the way Section 1.1's did — the first visible sign that this RTL won't synthesize into what it appears to describe.

### 2.2 Tracing the Latch

Looking more closely at exactly *when* the output fails to update reveals the underlying cause.

```bash
iverilog -o bad_mux bad_mux.v tb_bad_mux.v
gtkwave bad_mux.vcd
```

<img width="958" height="930" alt="badmux" src="https://github.com/user-attachments/assets/6be89683-4f14-4a27-81c9-e4a7ef19417a" />


**Observation:** the output freezes at its previous value exactly where an assignment was skipped in the RTL — the signature of latch inference. During synthesis, this incomplete `always` block resolves into a MUX plus an unintended latch, so the hardware won't behave identically to what the RTL seemed to promise. This is synthesis-simulation mismatch, caught here before it ever reached actual hardware.

---

## 3️⃣ Understanding Blocking Assignment Execution

### 3.1 Watching Execution Order in Simulation

Stepping away from the MUX examples, a standalone design isolates how blocking assignments actually execute, statement by statement.

```bash
iverilog -o blocking blocking_caveat.v tb_blocking_caveat.v
gtkwave blocking_caveat.vcd
```

<img width="958" height="930" alt="blockingwave" src="https://github.com/user-attachments/assets/c4128a95-157a-4613-94be-06c358bf575a" />


**Observation:** each `=` takes effect the instant it executes, and every line after it sees that new value immediately. This sequential, no-delay execution is exactly why blocking assignments suit combinational logic — and exactly why they're risky inside a sequential block, where later logic often needs to see the *old* register value rather than one just written earlier in the same clock cycle.

### 3.2 Confirming the Synthesized Result

The same blocking-assignment design was synthesized to check whether the hardware Yosys builds actually matches what the simulation predicted.

```bash
yosys
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="958" height="907" alt="blockingcircuit" src="https://github.com/user-attachments/assets/b42e7544-8805-4202-a040-c66511d1a2d9" />


**Observation:** for this design, the synthesized SKY130-mapped circuit matches the simulated behavior — confirming that blocking assignments synthesize predictably as long as the logic they describe stays combinational and fully specified.

### 3.3 Same-Block Value Dependency

One more pass over the same waveform, this time tracking a signal whose value depends on something assigned earlier in the *same* procedural block, on the *same* simulation time step.

```bash
iverilog -o blocking_past blocking_caveat.v tb_blocking_caveat.v
gtkwave blocking_caveat.vcd
```

<img width="958" height="930" alt="blocking1" src="https://github.com/user-attachments/assets/dac8023e-b065-4ac9-9613-c53b27637500" />

**Observation:** the later statement picks up the value its predecessor *just* wrote, not whatever value existed before the block started running. This is the underlying mechanism behind why statement order matters so much with `=`, and it's exactly the behavior that makes blocking assignments a poor fit inside sequential `always` blocks, where the old register value is usually what's actually needed.

---

## 4️⃣ Results at a Glance

| # | What Was Tested | Result | Takeaway |
|---|---|:---:|---|
| 1.1 | Ternary MUX simulation | ✅ | Clean combinational reference behavior |
| 1.2 | Ternary MUX synthesis | ✅ | Maps directly to a SKY130 MUX cell |
| 1.3 | MUX stress test | ✅ | Baseline holds across all input combinations |
| 2.1 | Faulty MUX, first look | ⚠️ | Output stops tracking select signal correctly |
| 2.2 | Faulty MUX, confirmed | ⚠️ | Latch-inference fingerprint identified |
| 3.1 | Blocking assignment sim | ℹ️ | Sequential, immediate-update execution confirmed |
| 3.2 | Blocking assignment synthesis | ✅ | Synthesized circuit matches simulated intent |
| 3.3 | Blocking + same-block value | ℹ️ | Later statements see freshly-updated values, not old ones |

---

## 5️⃣ Takeaways

- ✅ Verified a correctly-written ternary MUX simulates and synthesizes identically, mapping to a single SKY130 cell.
- ✅ Reproduced synthesis-simulation mismatch directly by leaving an `always` block incomplete, and watched it manifest as latch inference.
- ✅ Confirmed blocking assignments update — and get read — immediately, in exact statement order.
- ✅ Verified that fully-specified, combinational blocking-assignment logic synthesizes predictably.

The two multiplexers in this module are nearly identical on paper. Only one survives synthesis with its intended behavior intact, and the difference was never really about `=` versus `<=` in isolation — it came down to whether every output was fully specified for every input condition, and whether the designer accounted for blocking assignments executing immediately, in the order they're written.

That's the connective thread between Sections 2 and 3: the same immediate, in-order execution that makes blocking assignments predictable in combinational logic is exactly what makes them risky the moment they end up in a clocked, sequential context. Mismatch isn't a mysterious tool quirk — it's a direct, traceable consequence of incomplete or order-sensitive RTL.

**Working rule:** blocking (`=`) for combinational always blocks, non-blocking (`<=`) for sequential ones — and treat any unassigned branch as a latch waiting to happen.

---

## 👤 Author

**Arpitha Garrepalli**
[github.com/ArpithaGarrepalli/RTL_Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop)
