# 🔍 Module 3 — Combinational and Sequential Optimization

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the [RTL Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop) series.

## 🎯 Objectives

- Understand the concept of logic optimization in digital circuits.
- Study combinational and sequential logic optimization techniques.
- Perform synthesis using the Yosys synthesis tool.
- Map Verilog designs to the SKY130 standard-cell library.
- Simulate Verilog designs using Icarus Verilog.
- Verify circuit behavior using GTKWave.
- Analyze optimized gate-level netlists generated after synthesis.

## 🛠️ Tools and Technologies

| | |
|---|---|
| **HDL** | Verilog |
| **Simulator** | Icarus Verilog |
| **Waveform Viewer** | GTKWave |
| **Synthesis Tool** | Yosys |
| **PDK** | SKY130 |
| **Operating System** | Linux (Ubuntu) |

## 📚 Table of Contents

- 1. Introduction to Logic Optimization
- 2. Combinational Logic Optimization
- 3. Sequential Logic Optimization
  - 3.1 Sequential Optimization of D Flip-Flop
- 4. Constant Propagation
- 5. Unused Output Optimization
- 6. State Optimization
- 7. Logic Cloning
- 8. Retiming
- 9. Optimization Passes Performed in Yosys
- 10. Laboratory Exercises
  - 10.1 AND Gate Optimization (opt_check)
  - 10.2 OR Gate Optimization (opt_check2)
  - 10.3 Three-Input AND Gate Optimization (opt_check3)
  - 10.4 Sequential Constant Propagation — dff_const1
  - 10.5 Sequential Constant Propagation — dff_const2
  - 10.6 D Flip-Flop with Synchronous Reset (dff_const3)
  - 10.7 Counter Optimization (counter_opt)
- 11. Laboratory Summary
- 12. Takeaways
- Author

---

## 1️⃣ Introduction to Logic Optimization

Once RTL has been mapped to logic gates, synthesis doesn't stop there — the tool still walks back through the circuit looking for logic that can be simplified, dropped, or restructured without changing what the design actually does. The goal is a smaller, cleaner gate-level implementation that still behaves identically to the original RTL, while reducing area, power, and delay.

This module looks at the specific optimization behaviors Yosys applies, split across combinational and sequential circuits, using the SKY130 standard-cell library throughout.

---

## 2️⃣ Combinational Logic Optimization

Combinational optimization trims logic that isn't pulling its weight — **the circuit's behavior stays exactly the same**, only the implementation gets leaner. Yosys works through the Boolean expressions in the design and strips out anything redundant, arriving at a smaller gate count than a literal translation of the RTL would produce.

**🎯 What it's aiming for**

- Fewer logic gates overall.
- Simpler Boolean expressions.
- Smaller silicon area.
- Faster propagation through the circuit.
- Lower power draw.

---
<img width="1254" height="660" alt="Screenshot 2026-08-22 183837" src="https://github.com/user-attachments/assets/9cd8f384-c8fc-49cc-a7ce-c0c695ab1e37" />


## 3️⃣ Sequential Logic Optimization

Sequential optimization targets circuits built around memory elements — flip-flops, in particular. The constraint here is stricter than in the combinational case: the tool has to leave the *behavior* of every sequential element untouched, even while it's stripping out registers that turn out to be unnecessary and simplifying the logic feeding them. Techniques like sequential constant propagation, retiming, and state optimization all fall under this umbrella.

**🎯 What it typically does**

- Drops flip-flops that turn out to be redundant.
- Pushes constant values forward through sequential logic.
- Clears out logic that can never actually be reached.
- Tightens timing without altering functional behavior.
<img width="1214" height="651" alt="Screenshot 2026-08-22 184051" src="https://github.com/user-attachments/assets/b4a6ee3d-b644-400a-94a0-b6def0613459" />


### 3.1 Sequential Optimization of D Flip-Flop

**Design — `dff_const1.v`**

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

<img width="1912" height="1016" alt="dff_const1" src="https://github.com/user-attachments/assets/764344e4-3ff5-4a23-b7a6-f266d9459768" />


Even though `q` toggles between two different values depending on `reset`, the synthesized circuit still comes out leaner than a naive reading of the RTL might suggest — unnecessary sequential logic gets stripped while the original behavior is fully preserved.

<img width="1912" height="1011" alt="Screenshot 2026-08-22 053736" src="https://github.com/user-attachments/assets/0fd6f6a5-501f-463e-8232-1274d0841128" />


---

## 4️⃣ Constant Propagation

If a signal is always going to carry the same fixed value no matter what, there's no reason to build logic to compute it — Yosys just substitutes the constant directly wherever that signal is used, then removes whatever logic becomes redundant as a result.

**✅ Why it's worth doing**

- Cuts down on logic complexity.
- Uses less hardware overall.
- Helps timing.
- Reduces power consumption.

<img width="1771" height="835" alt="count" src="https://github.com/user-attachments/assets/cddcdd00-a958-469c-97e1-8ca63e8b0d76" />



The synthesized netlist shows exactly this — constant-valued signals get carried through the logic directly, and whatever gates were only there to compute that already-known value disappear.

---

## 5️⃣ Unused Output Optimization

Any signal or output that nothing downstream actually reads gets flagged by Yosys as having zero impact on the final result — and it gets stripped out entirely.

This keeps the gate count from ballooning with hardware that would never actually do anything, and it's a good illustration of a broader point: synthesis only builds hardware for logic that genuinely reaches an output. The counter example in Section 10.7 demonstrates this directly.

<img width="890" height="826" alt="Screenshot 2026-08-22 175140" src="https://github.com/user-attachments/assets/4908022b-0bd3-4ddb-a0ee-2368dbb0b2b8" />

The optimized netlist ends up with noticeably fewer gates than the unoptimized version, while still producing identical results.

---

## 6️⃣ State Optimization

Finite State Machines often carry states that turn out to be equivalent to each other, or that never actually get reached. During optimization, Yosys can merge or drop these, cutting the hardware needed to implement the FSM without changing how it behaves.

**🎯 What this usually involves**

- Merging states that behave identically.
- Choosing a more efficient state encoding.
- Simplifying the next-state logic.
- Trimming overall hardware complexity.

---

## 7️⃣ Logic Cloning

Logic cloning takes the opposite approach from most optimizations — instead of removing hardware, it **duplicates** a cell that's driving too many downstream loads.

Splitting one heavily-loaded gate into several copies, each driving a smaller share of the fan-out, cuts the delay on whichever timing path was suffering because of that single overloaded gate.

---

## 8️⃣ Retiming

Retiming shifts flip-flops around within the combinational logic surrounding them, without touching what the circuit as a whole actually computes.

The point is to rebalance how much delay sits in each pipeline stage, which in turn raises the maximum clock frequency the design can run at. Unlike the other optimizations here, retiming only ever moves **where registers sit** — it doesn't change the logic itself.

---

## 9️⃣ Optimization Passes Performed in Yosys

Synthesis in Yosys runs through several distinct optimization passes automatically, each targeting a different kind of redundancy.

| Optimization Pass | Purpose |
|---|---|
| Constant propagation | Replace known-constant signals directly |
| Dead logic elimination | Remove logic with no effect on outputs |
| Boolean simplification | Reduce Boolean expressions |
| Removal of unused wires | Remove unreferenced signals |
| Removal of unused cells | Remove unreferenced gates/cells |
| Expression simplification | Simplify equivalent expressions |
| Resource sharing | Reuse hardware across similar operations |

Taken together, these passes are what turns a literal gate-for-gate translation of the RTL into a genuinely efficient netlist.

---

## 🔟 Laboratory Exercises

### 10.1 AND Gate Optimization (opt_check)

```verilog
module opt_check (
    input a,
    input b,
    output y
);

assign y = a & b;

endmodule
```

**Yosys Commands**

```bash
yosys
read_verilog opt_check.v
synth -top opt_check
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1912" height="1025" alt="opt_check" src="https://github.com/user-attachments/assets/758cfa83-9708-4c4a-b3d9-8e1d60604b17" />


**Result:** the AND gate synthesized cleanly and mapped directly onto a single SKY130 `and2` standard cell.

### 10.2 OR Gate Optimization (opt_check2)

```verilog
module opt_check2 (
    input a,
    input b,
    output y
);

assign y = a | b;

endmodule
```

**Yosys Commands**

```bash
yosys
read_verilog opt_check2.v
synth -top opt_check2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1915" height="1015" alt="opt_check2" src="https://github.com/user-attachments/assets/debda19d-e03d-42e0-b903-82b4d46716e3" />

**Result:** the OR gate mapped onto a single SKY130 `or2` standard cell, just as directly as the AND gate did.

### 10.3 Three-Input AND Gate Optimization (opt_check3)

```verilog
module opt_check3 (
    input a,
    input b,
    input c,
    output y
);

assign y = a & b & c;

endmodule
```

**Yosys Commands**

```bash
yosys
read_verilog opt_check3.v
synth -top opt_check3
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1916" height="1017" alt="opt_check3" src="https://github.com/user-attachments/assets/139f6627-60eb-48cd-b81d-1a7397d44939" />

**Result:** extending the AND to three inputs, Yosys mapped it onto a single SKY130 `and3` cell rather than chaining two `and2` cells together.

### 10.4 Sequential Constant Propagation — dff_const1

This pair of flip-flop designs (`dff_const1` and `dff_const2`) demonstrates sequential constant propagation — both assign constant values to their output, giving Yosys the opportunity to optimize the resulting hardware differently depending on how constant the behavior actually is.

```bash
vim dff_const1.v
```

<img width="1918" height="1016" alt="dffconst1and2code" src="https://github.com/user-attachments/assets/afc1eba2-d24b-4f9c-b4de-c7d5d26650e9" />


Simulate `dff_const1`, whose output still depends on `reset`:

```bash
iverilog -o dff_const1.out dff_const1.v tb_dff_const1.v
gtkwave tb_dff_const1.vcd
```

<img width="1912" height="1011" alt="Screenshot 2026-08-22 053736" src="https://github.com/user-attachments/assets/0fd6f6a5-501f-463e-8232-1274d0841128" />

Synthesize it and inspect the netlist before optimization collapses anything further:

```bash
yosys
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1912" height="1016" alt="dff_const1" src="https://github.com/user-attachments/assets/764344e4-3ff5-4a23-b7a6-f266d9459768" />

### 10.5 Sequential Constant Propagation — dff_const2

`dff_const2` pushes the same idea further — `q` is assigned `1'b1` on *both* branches, so its value never actually depends on `reset` at all:

```bash
iverilog -o dff_const2.out dff_const2.v tb_dff_const2_.v
gtkwave tb_dff_const2_.vcd
```

<img width="1918" height="1022" alt="dff_const2" src="https://github.com/user-attachments/assets/6ea7c5cd-7546-4df8-b78b-10a295ecc131" />


```bash
yosys
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

<img width="1917" height="1017" alt="gtkwavedffconst2_1" src="https://github.com/user-attachments/assets/97d05fa9-af67-4421-8eec-15dcba996191" />


**Result:** since `q` never changes regardless of `reset`, Yosys strips out the redundant reset logic that `dff_const1` still needed, showing a visibly leaner circuit here than in 10.4.

### 10.6 D Flip-Flop with Synchronous Reset (dff_const3)

```verilog
module dff_const3(input clk, input reset, output reg q);

always @(posedge clk)
begin
    if(reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
```

Note that `reset` here only appears *inside* the clocked block, not in the sensitivity list — this makes it a synchronous reset, unlike the asynchronous reset used in `dff_const1`/`dff_const2`.

```bash
iverilog -o dff_const3.out dff_const3.v dff_const3_tb.v
gtkwave dff_const3.vcd
```

<img width="1918" height="1022" alt="dff_const3" src="https://github.com/user-attachments/assets/c4e60126-3d23-403b-9b41-3fc9f35b185b" />


```bash
yosys
read_verilog dff_const3.v
synth -top dff_const3
show
```

<img width="1918" height="1017" alt="gtkwavedffconst3" src="https://github.com/user-attachments/assets/68e3b604-d529-4d94-a00c-96dc5b719767" />


### 10.7 Counter Optimization (counter_opt)

This example demonstrates unused-output optimization directly: a 3-bit counter is built, but only its least significant bit is ever connected to an output.

```verilog
module counter_opt(input clk, input reset, output q);

reg [2:0] count;

assign q = count[0];

always @(posedge clk, posedge reset)
begin
    if(reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
```

```bash
yosys
read_verilog counter_opt.v
synth -top counter_opt
show
```

<img width="1037" height="701" alt="Screenshot 2026-08-22 183143" src="https://github.com/user-attachments/assets/7b8dd0f8-921d-4d52-b72e-871841630305" />


Since `count[1]` and `count[2]` never reach an output, Yosys has no reason to build flip-flops for them — the schematic shows a 3-bit `count` register collapsing down to a single flip-flop in the actual hardware:

```bash
write_verilog -noattr counter_opt_net.v
gvim counter_opt_net.v
```

<img width="1771" height="835" alt="count" src="https://github.com/user-attachments/assets/db902d0b-353f-4cda-b1ca-63c04cde1fab" />

<img width="890" height="826" alt="Screenshot 2026-08-22 175140" src="https://github.com/user-attachments/assets/f5ed2dd2-3108-450e-9734-7e2ac2b1c096" />


**Result:** even though the RTL describes a full 3-bit counter, only one flip-flop actually gets built, since the other two bits' values never influence any output — a direct, concrete illustration of the unused-output optimization described in Section 5.

---

## 1️⃣1️⃣ Laboratory Summary

| Lab | Focus | Key Result |
|---|---|---|
| 10.1 | AND gate optimization | Mapped directly to a SKY130 `and2` cell |
| 10.2 | OR gate optimization | Mapped directly to a SKY130 `or2` cell |
| 10.3 | Three-input AND optimization | Mapped to a single SKY130 `and3` cell instead of chained `and2`s |
| 10.4 | Sequential constant propagation (dff_const1) | Reset-dependent output retains necessary reset logic |
| 10.5 | Sequential constant propagation (dff_const2) | Reset-independent output allows reset logic to be dropped |
| 10.6 | Synchronous reset flip-flop (dff_const3) | Reset handled without appearing in the sensitivity list |
| 10.7 | Counter optimization (counter_opt) | Only 1 of 3 counter bits synthesized, since the rest were unused |

---

## 1️⃣2️⃣ Takeaways

- ✅ Distinguished how combinational and sequential optimization differ in what they're allowed to change.
- ✅ Saw constant propagation collapse logic that depended on an already-known value, and compared it directly across `dff_const1` and `dff_const2`.
- ✅ Watched unused outputs and dead logic get stripped out automatically during synthesis, concretely demonstrated by `counter_opt` reducing three flip-flops down to one.
- ✅ Compared asynchronous reset (`dff_const1`/`dff_const2`) against synchronous reset (`dff_const3`) coding styles and how each maps to hardware.
- ✅ Verified basic gate optimization (AND, OR, three-input AND) mapping directly onto SKY130 standard cells.
- ✅ Covered state optimization, logic cloning, and retiming as further optimization strategies, even without a dedicated lab for each.
- ✅ Confirmed every optimization claim directly against synthesized netlists, schematics, and simulation waveforms rather than taking them on faith.

Across all of this, the throughline is the same: Yosys isn't just translating RTL into gates one-to-one — it's actively looking for constants to fold in, logic to drop, and registers to eliminate, all while guaranteeing the final circuit behaves exactly like the original description. That balance of **area, timing, and power** is really the whole point of running synthesis in the first place, rather than just hand-mapping RTL to gates directly.

---

## 👤 Author

**Arpitha Garrepalli**
[github.com/ArpithaGarrepalli/RTL_Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop)
