# 🔧 Module 5 — Optimization in Synthesis

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the [RTL Workshop](https://github.com/madapaamrutha-svg/RTL_Workshop) series.

## 📖 Overview

This module looks at how Yosys optimizes hardware during synthesis, and — just as importantly — what happens when the RTL doesn't give it enough information to optimize safely. A set of combinational and sequential circuits were simulated and synthesized to see when Yosys collapses logic down to something minimal and efficient, and when incomplete coding forces it to insert extra hardware just to stay correct. The two outcomes turn out to be two sides of the same coin: a synthesis tool can only simplify or share resources when every output is fully defined; the moment a condition is left unhandled, the "optimization" it performs is inserting a latch to preserve state, which is rarely what the designer intended.

## 🎯 Objectives

- Understand what synthesis optimization actually does to an RTL description.
- Observe latch inference caused by incomplete `if` and `case` coding styles.
- Compare an incomplete case statement against a fully-specified one, in the same design.
- Recognize that partial assignment can occur per-signal, even when a branch looks complete.
- Verify RTL behavior in GTKWave before trusting the synthesized result.
- Synthesize each design in Yosys and inspect the resulting schematic/netlist.
- Confirm, using standard combinational blocks, that clean RTL synthesizes to clean hardware.

## 🛠️ Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| Verilog HDL | RTL design |
| Icarus Verilog | Compiling and simulating designs |
| GTKWave | Waveform inspection |
| Yosys | RTL synthesis and optimization |
| SKY130 Standard-Cell Library | Technology mapping |
| Ubuntu Linux | Development environment |

## 📑 Table of Contents

- 1. Incomplete IF Statement
  - 1.1 Simulating the Incomplete IF
  - 1.2 Synthesizing the Incomplete IF
  - 1.3 Closer Look at the Waveform
- 2. Incomplete IF-ELSE Statement
  - 2.1 Synthesizing the Incomplete IF-ELSE
- 3. CASE Statements — Complete vs. Incomplete
  - 3.1 Incomplete CASE
  - 3.2 Complete CASE
  - 3.3 Partial Assignment Inside a CASE
  - 3.4 A Fully-Specified 4-Way CASE
- 4. Combinational Reference Designs
  - 4.1 Multiplexer
  - 4.2 Demultiplexer
  - 4.3 Ripple Carry Adder
- 5. Overall Results
- 6. Conclusion
- Author

---

## 1️⃣ Incomplete IF Statement

### 1.1 Simulating the Incomplete IF

An incomplete `if` statement only assigns its output when a particular condition holds — every other case is left with no assignment at all. Before touching synthesis, the RTL was simulated on its own to see how the output actually behaves.

```bash
iverilog -o incom_if incom_if.v incom_if_tb.v
gtkwave incom_if.vcd
```

<img width="896" height="855" alt="Screenshot 2026-08-22 200856" src="https://github.com/user-attachments/assets/c7ae569e-a8fb-42b7-9e1c-de0f409d5e32" />

**Observation:** the output only updates while `sel` is asserted; the instant `sel` drops, the output freezes at whatever value it last held, rather than changing to reflect the new condition.

**Result:** the waveform confirms the output is undefined for part of the condition space, and simply holds its last value there — a sign of memory behavior that shouldn't exist in a purely combinational block.

### 1.2 Synthesizing the Incomplete IF

The same RTL was pushed through Yosys to see what hardware it actually resolves to.

```bash
yosys
read_verilog incom_if.v
synth -top incom_if
show
```

<img width="888" height="805" alt="Screenshot 2026-08-22 200958" src="https://github.com/user-attachments/assets/e7bc1276-6f9c-4fb7-b5e6-b669d03f109c" />


**Observation:** the schematic contains a latch that Yosys inserted specifically to hold the output steady whenever the `if` condition isn't met.

**Result:** since the RTL never says what to do outside the one specified condition, the only synthesizable option is to remember the previous value — which is exactly what the inferred latch does.

### 1.3 Closer Look at the Waveform

A second, more detailed pass over the same simulation checks whether the freeze behavior lines up precisely with the condition, or whether it's inconsistent.

```bash
iverilog -o incom_if incom_if.v incom_if_tb.v
gtkwave incom_if.vcd
```

<img width="898" height="877" alt="Screenshot 2026-08-22 201100" src="https://github.com/user-attachments/assets/8b28425e-edee-4484-a4bd-4b7c4729e4c7" />


**Observation:** every output edge tracks a transition of the select signal; whenever the select signal falls, the output simply stays where it was.

**Result:** this rules out the freeze being some kind of simulation quirk — it's a direct, repeatable consequence of the missing assignment, matching the schematic in 1.2.

---

## 2️⃣ Incomplete IF-ELSE Statement

### 2.1 Synthesizing the Incomplete IF-ELSE

This design adds an `else` branch to the earlier `if`, but the branch still doesn't account for every reachable condition.

```bash
yosys
read_verilog incom_if2.v
synth -top incom_if2
show
```

<img width="893" height="818" alt="Screenshot 2026-08-22 201146" src="https://github.com/user-attachments/assets/a214bdc7-beaa-4117-a3a6-975df20f1d80" />


**Observation:** even with an `else` present, the schematic still shows a latch wherever coverage is incomplete.

**Result:** simply having an `else` clause isn't the safeguard it looks like — what matters is whether *every* condition ends in an assignment, not whether an `else` exists at all.

---

## 3️⃣ CASE Statements — Complete vs. Incomplete

### 3.1 Incomplete CASE

```verilog
module incomp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
    always @(*) begin
        case(sel)
            2'b00 : y = i0;
            2'b01 : y = i1;
        endcase
    end
endmodule
```

Two of the four possible values of `sel` (`2'b10` and `2'b11`) have no matching branch at all.

<img width="896" height="842" alt="Screenshot 2026-08-22 220632" src="https://github.com/user-attachments/assets/2dd28e2b-e97f-4b38-841c-1a4b29c2386a" />
<img width="941" height="863" alt="Screenshot 2026-08-22 220741" src="https://github.com/user-attachments/assets/5e5793ea-82f0-40ce-b91f-6bd4fcdb62f5" />


**Observation:** the synthesized netlist contains a latch, and the waveform shows the output holding its value for the two unhandled `sel` codes.

**Result:** a `case` statement with missing branches produces exactly the same failure as the incomplete `if` in Section 1 — the mechanism differs, the outcome doesn't.

### 3.2 Complete CASE

```verilog
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
    always @(*) begin
        case(sel)
            2'b00   : y = i0;
            2'b01   : y = i1;
            default : y = i2;
        endcase
    end
endmodule
```

<img width="896" height="843" alt="Screenshot 2026-08-22 220759" src="https://github.com/user-attachments/assets/a987dcf2-4d87-47e2-a8f3-b4d37e49674b" />
<img width="905" height="862" alt="Screenshot 2026-08-22 220812" src="https://github.com/user-attachments/assets/7fdc7f38-0f5f-4db7-a9fa-de7fc71a236c" />


**Observation:** adding a single `default` branch that assigns the output closes every remaining gap; the resulting netlist is pure combinational logic with no latch.

**Result:** this is the direct fix for 3.1 — once every `sel` value maps to an assignment, Yosys has nothing left to "remember" and synthesizes clean.

### 3.3 Partial Assignment Inside a CASE

```verilog
module partial_case_assign (input i0, input i1, input i2, input [1:0] sel,
                             output reg y, output reg x);
    always @(*) begin
        case(sel)
            2'b00 : begin y = i0; x = i2; end
            2'b01 : y = i1;
            default : begin x = i1; end
        endcase
    end
endmodule
```

<img width="899" height="837" alt="Screenshot 2026-08-22 220830" src="https://github.com/user-attachments/assets/342e8f2d-880c-4b26-9a8d-3a4ffb774ec9" />
<img width="943" height="866" alt="Screenshot 2026-08-22 220841" src="https://github.com/user-attachments/assets/a9a10171-d337-4a22-b6c9-e6616c7129ba" />


**Observation:** the netlist infers a latch for `y` because it's never assigned in the `default` branch, and a separate latch for `x` because it's never assigned in the `2'b01` branch. The waveform mirrors this — each signal freezes exactly where its own assignment is missing.

**Result:** this case has a `default` branch and still infers latches, which shows that branch coverage and per-signal coverage are two different checks. Every output needs an assignment in every branch, not just every branch needing *an* assignment.

### 3.4 A Fully-Specified 4-Way CASE

```verilog
module bad_case (input i0, input i1, input i2, input i3, input [1:0] sel, output reg y);
    always @(*) begin
        case(sel)
            2'b00 : y = i0;
            2'b01 : y = i1;
            2'b10 : y = i2;
            2'b11 : y = i3;
        endcase
    end
endmodule
```

<img width="974" height="865" alt="Screenshot 2026-08-22 220853" src="https://github.com/user-attachments/assets/141acaa0-1e50-4b3b-9c1b-005210f318d2" />


**Observation:** despite its name, this module lists all four possible `sel` combinations explicitly, and the waveform shows the output switching correctly for every one of them.

**Result:** correctness here comes from the code, not the label — a reminder that "bad_case" as a name doesn't make the coverage bad, and a design should always be judged by what it actually assigns.

---

## 4️⃣ Combinational Reference Designs

### 4.1 Multiplexer

<img width="990" height="831" alt="Screenshot 2026-08-22 220904" src="https://github.com/user-attachments/assets/096d7cb2-7bfa-4844-bf2c-abb243f9155c" />


**Observation:** the output tracks whichever input the select line points to, across the full sweep of test values.

**Result:** clean, fully-covered combinational behavior — a good baseline for what "no latch" should look like.

### 4.2 Demultiplexer

<img width="989" height="866" alt="Screenshot 2026-08-22 220921" src="https://github.com/user-attachments/assets/7a99225f-908b-477b-8ce5-86626bdf54d0" />

**Observation:** the single input is routed to exactly one output line at a time, and every other output line stays inactive.

**Result:** correct routing with no stray storage — confirms the demux design covers every select condition.

### 4.3 Ripple Carry Adder

<img width="937" height="901" alt="Screenshot 2026-08-22 220932" src="https://github.com/user-attachments/assets/67adb400-387d-4c4f-b33d-b8da02b62e5f" />


**Observation:** the carry signal propagates correctly through each full-adder stage, and the sum/carry outputs match expected binary addition for every tested input pair.

**Result:** the same "fully-specified means clean synthesis" pattern holds even at the scale of a multi-bit arithmetic circuit.

---

## 5️⃣ Overall Results

| # | Design | Result | Takeaway |
|---|---|:---:|---|
| 1.1 | Incomplete IF, sim | ⚠️ | Output freezes when condition is false |
| 1.2 | Incomplete IF, synth | ⚠️ | Latch inferred at the missing branch |
| 1.3 | Incomplete IF, detail | ⚠️ | Freeze behavior confirmed reproducible |
| 2.1 | Incomplete IF-ELSE, synth | ⚠️ | Latch inferred despite else being present |
| 3.1 | Incomplete CASE | ⚠️ | Latch inferred for unhandled sel codes |
| 3.2 | Complete CASE | ✅ | Default branch removes the latch entirely |
| 3.3 | Partial CASE assignment | ⚠️ | Latches inferred per-signal, not per-branch |
| 3.4 | Fully-specified 4-way CASE | ✅ | Every combination assigned, clean switching |
| 4.1 | Multiplexer | ✅ | Clean combinational reference |
| 4.2 | Demultiplexer | ✅ | Clean combinational reference |
| 4.3 | Ripple Carry Adder | ✅ | Clean at multi-bit arithmetic scale |

---

## 6️⃣ Conclusion

Across every design in this module, the pattern held without exception: Yosys could only optimize — simplify logic, share resources, map cleanly onto SKY130 cells — when the RTL left it nothing ambiguous to resolve. The moment an output had no assignment for some reachable condition, whether from a missing `else`, a missing `case` branch, or one signal skipped inside an otherwise-complete branch, the tool's only correct move was to insert a latch and hold state. The multiplexer, demultiplexer, and ripple carry adder confirm the other side of that same rule: fully-specified combinational logic synthesizes exactly as clean as it reads.

**Working rule carried forward from this module:** assign every output, for every branch, for every reachable input — anything less isn't an optimization opportunity, it's a latch waiting to be inferred.

---

## 👤 Author

**Arpitha Garrepalli**
B.Tech – Electronics & Communication Engineering
Anurag University
