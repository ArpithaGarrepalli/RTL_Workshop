# 🔧 Session 2 — Debugging a Bad MUX, Yosys Synthesis Walkthroughs, and BabySoC Pre-Synthesis Simulation

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

## 📖 Overview

The morning of this session started with a debugging exercise — a MUX that looked reasonable but had a subtle coding mistake, found by comparing simulation behavior against what the code actually says. Once fixed, the same MUX became the reference example for a full Yosys synthesis walkthrough, which was then repeated on a counter and a multi-module design to compare hierarchical and flattened synthesis results. The afternoon moved to a different project entirely — cloning and exploring VSDBabySoC, then running its pre-synthesis simulation.

| | |
|---|---|
| 🛠️ **Tools used** | Icarus Verilog, GTKWave, Yosys |
| 🧩 **Example designs** | `bad_mux`, `good_mux`, `good_counter`, `multiple_modules`, VSDBabySoC |
| 📋 **Prerequisites** | Basic familiarity with digital logic and Linux terminal |

## 📑 Table of Contents

- 1. Debugging bad_mux
  - 1.1 Spotting the Mistake
  - 1.2 The Fix
- 2. Yosys Synthesis of good_mux
- 3. Applying the Same Flow to good_counter
- 4. Applying the Same Flow to multiple_modules
  - 4.1 Hierarchical View
  - 4.2 Flattened View
- 5. BabySoC — Cloning, Exploring RVMYTH, and Pre-Synthesis Simulation
  - 5.1 Cloning the Repository
  - 5.2 Exploring the RVMYTH Core
  - 5.3 Running the Pre-Synthesis Simulation
- 6. Takeaways

---

## 1️⃣ Debugging bad_mux

### 1.1 Spotting the Mistake

```verilog
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else
		y <= i0;
end
endmodule
```

<img width="1148" height="1280" alt="WhatsApp Image 2026-08-30 at 11 32 03 PM" src="https://github.com/user-attachments/assets/46a8c04b-4b06-4e34-814d-8d703d938ec9" />

At first glance this looks fine — both branches of the `if` are assigned, so there's no missing-else latch issue. The actual mistake is more subtle: it uses **non-blocking assignments (`<=`)** inside a **combinational** `always @(*)` block. Non-blocking assignments schedule their update to happen at the end of the current simulation time step rather than immediately, which is exactly the right behavior for sequential (clocked) logic — but inside combinational logic it can introduce a one-delta-cycle lag that doesn't match what a synthesis tool will actually build.

```bash
iverilog -o bad_mux bad_mux.v tb_bad_mux.v
gtkwave bad_mux.vcd
```

<img width="720" height="1280" alt="WhatsApp Image 2026-08-30 at 11 32 02 PM" src="https://github.com/user-attachments/assets/7e0c3812-cb37-49b0-9039-7215bdfd5c49" />

The waveform (`i0=1`, `i1=0`, `sel=0`, `y=0`) shows the output lagging behind what the select and data lines say it should be at that instant — the visible symptom of using `<=` where `=` belongs.

### 1.2 The Fix

```verilog
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y = i1;
	else
		y = i0;
end
endmodule
```

Swapping `<=` for `=` is the entire fix. Blocking assignments execute immediately and are read immediately by whatever comes next in the same block — the correct behavior for combinational logic, where there's no clock edge to wait for.

---

## 2️⃣ Yosys Synthesis of good_mux

With the corrected `good_mux` design in hand, this became the reference walkthrough for the rest of the day's synthesis work:

```bash
yosys
read_verilog good_mux.v
synth -top good_mux
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog good_mux_net.v
```

<img width="1280" height="655" alt="WhatsApp Image 2026-08-30 at 11 32 03 PM (1)" src="https://github.com/user-attachments/assets/6b694de7-2086-446d-89e3-40767992b2d1" />

<img width="555" height="911" alt="WhatsApp Image 2026-08-30 at 11 32 03 PM (2)" src="https://github.com/user-attachments/assets/f23161ed-df08-42a7-8e4f-740d593134bd" />

---

## 3️⃣ Applying the Same Flow to good_counter

The identical compile → synthesize → inspect flow was repeated on a counter design:

```bash
yosys
read_verilog good_counter.v
synth -top good_counter
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr good_counter_netlist.v
```
<img width="1280" height="720" alt="WhatsApp Image 2026-08-30 at 11 32 04 PM (1)" src="https://github.com/user-attachments/assets/e911d24f-47fd-41e8-816c-5be2639279e4" />
<img width="1280" height="720" alt="WhatsApp Image 2026-08-30 at 11 32 04 PM (2)" src="https://github.com/user-attachments/assets/d33d9ae3-3209-4408-a7e4-0486f7ac896e" />


The counter's two bits (`cnt[0]`, `cnt[1]`) each map onto their own `$_DFF_PP0_` flip-flop, with `sky130_fd_sc_hd__nor2_1` and `nor2b_1` cells forming the next-state logic feeding them.

Reading the generated netlist directly confirms what the schematic shows: two separate `always @(posedge clk, posedge reset)` blocks, one per bit of `cnt`, each driven by its own NOR-based next-state logic rather than one shared 2-bit register update.

---

## 4️⃣ Applying the Same Flow to multiple_modules

Repeating the flow once more on a multi-module design (`sub_module1` = AND, `sub_module2` = OR, wired together as `multiple_modules`) surfaced a clear difference depending on whether the design is synthesized hierarchically or flattened first.

### 4.1 Hierarchical View

```bash
read_verilog multiple_modules.v
synth -top multiple_modules
show multiple_modules
```

<img width="720" height="1280" alt="WhatsApp Image 2026-08-30 at 11 32 05 PM (1)" src="https://github.com/user-attachments/assets/0e407f15-bba9-4bca-a255-65df82914311" />

At this level, `sub_module1` and `sub_module2` still appear as their own labeled blocks (`u1`, `u2`) in the schematic — the module boundaries from the RTL are preserved rather than merged.

### 4.2 Flattened View

```bash
flatten
show multiple_modules
```

<img width="1280" height="720" alt="WhatsApp Image 2026-08-30 at 11 32 05 PM (2)" src="https://github.com/user-attachments/assets/e532f87e-c241-41bc-8ff1-5abe30412384" />

After flattening, those same two blocks disappear — `u1.a`, `u1.b`, `u1.y` and `u2.a`, `u2.b`, `u2.y` now sit directly on the gates themselves (`sky130_fd_sc_hd__and2_0` and `sky130_fd_sc_hd__or2_0`), with the `$scopeinfo` markers being the only trace left of the original module names. This is the concrete difference between hierarchical and flat synthesis: same logic, same two gates either way, but the flattened netlist has no sub-module boundaries left to read.

---

## 5️⃣ BabySoC — Cloning, Exploring RVMYTH, and Pre-Synthesis Simulation

### 5.1 Cloning the Repository

The afternoon moved to a different project — VSDBabySoC, a small RISC-V-based SoC design:

```bash
git clone https://github.com/manili/VSDBabySoC
cd VSDBabySoC
```


The `module/` directory contains the SoC's building blocks: `avsddac.v` and `avsdpll.v` (analog DAC and PLL models), `clk_gate.v`, the RVMYTH core (`rvmyth.v`, `rvmyth_gen.v`, `rvmyth.tlv`), and both pre-synthesis and post-synthesis testbenches.

### 5.2 Exploring the RVMYTH Core

<img width="1049" height="659" alt="WhatsApp Image 2026-08-30 at 11 32 06 PM" src="https://github.com/user-attachments/assets/38bb615a-ebc6-4591-8616-07be24e7d26d" />


RVMYTH is a small RISC-V core written in TL-Verilog and generated into plain Verilog (`rvmyth_gen.v`). The comments in the generated file trace out its instruction sequence directly — a small loop built from `ADDI`, `ADD`, `SUB`, and `BNE`/`BEQ` — which is what eventually drives the `OUT` register that feeds the rest of the SoC.

### 5.3 Running the Pre-Synthesis Simulation

<img width="572" height="1075" alt="WhatsApp Image 2026-08-30 at 11 32 06 PM (1)" src="https://github.com/user-attachments/assets/6b0d5054-4388-4b44-9aaf-8363ab99a023" />


The testbench uses a `PRE_SYNTH_SIM` / `POST_SYNTH_SIM` macro switch to include either the original RTL sources or the synthesized netlist plus SKY130 primitives — the same testbench works for both stages just by defining a different macro at compile time.

```bash
iverilog -DPRE_SYNTH_SIM -o pre_synth_sim.out testbench.v
vvp pre_synth_sim.out
gtkwave pre_synth_sim.vcd
```

<img width="1600" height="848" alt="WhatsApp Image 2026-08-30 at 11 32 06 PM (2)" src="https://github.com/user-attachments/assets/b0061a4e-4ceb-4803-a559-a4564d5496b6" />

**Observation:** the waveform shows `CLK` toggling, `RV_TO_DAC[9:0]` counting through values (`334` at the marker), and `OUT` responding as the DAC's analog approximation of that digital value rises and falls — confirming the RVMYTH core, clock gating, and DAC model are all functioning together correctly at the RTL level, before synthesis is involved at all.

---

## 6️⃣ Takeaways

- ✅ Diagnosed a synthesis-simulation risk in `bad_mux` caused by non-blocking assignments inside a combinational `always` block, and fixed it by switching to blocking assignments.
- ✅ Ran a complete Yosys synthesis flow on `good_mux`, down to reading the actual `.lib` cell definition Yosys selected.
- ✅ Repeated the same flow on `good_counter`, confirming each counter bit gets its own flip-flop and NOR-based next-state logic.
- ✅ Repeated the flow again on `multiple_modules`, directly comparing hierarchical synthesis (module boundaries preserved) against flattened synthesis (module boundaries dissolved into raw gates).
- ✅ Cloned and explored VSDBabySoC, including reading through the RVMYTH RISC-V core's generated instruction trace.
- ✅ Ran BabySoC's pre-synthesis simulation and confirmed correct RTL-level behavior across the RVMYTH core, clock gating, and DAC model before moving on to synthesis.
## Author-ArpithaGarrepalli
