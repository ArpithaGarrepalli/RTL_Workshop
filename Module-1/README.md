# 🔧 Module 1 — Introduction to Verilog RTL Design & Synthesis

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the [RTL Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop) series.

## 📖 Overview

This document covers Day 1 of the RTL Design Workshop, focused on the fundamentals of Verilog RTL design — simulating designs with Icarus Verilog (`iverilog`), analyzing waveforms in GTKWave, and an introduction to logic synthesis with Yosys.

| | |
|---|---|
| 🛠️ **Tools used** | Icarus Verilog, GTKWave, Yosys |
| 🧩 **Example design** | 2-stage shift register (`good_shift_reg`) |
| 📋 **Prerequisites** | Basic familiarity with digital logic and Linux terminal |

## 📑 Table of Contents

- 1. Introduction to the Open-Source Simulator Iverilog
  - 1.1 Core Concepts: Simulator, Design, and Testbench
  - 1.2 How Iverilog Fits In
- 2. Labs: Iverilog and GTKWave
  - 2.1 Lab 1 — Environment Setup
  - 2.2 Lab 2 — Compile and Simulate
  - 2.3 Lab 3 — Waveform Analysis
  - 2.4 Verilog Code Analysis
- 3. Introduction to Yosys and Logic Synthesis
  - 3.1 What Synthesis Does
  - 3.2 Lab: Synthesizing the Design
- 4. Takeaways
- Author

---

## 1️⃣ Introduction to the Open-Source Simulator Iverilog

### 1.1 Core Concepts: Simulator, Design, and Testbench

| Term | Description |
|---|---|
| 🖥️ **Simulator** | A tool that checks whether a digital circuit behaves the way it's supposed to, without needing to build the hardware first. |
| 📐 **Design** | The Verilog code itself — it describes the logic and behavior the hardware needs to implement. |
| 🧪 **Testbench** | A separate piece of code that feeds various input patterns into the design and confirms the outputs come out as expected. |

**How a Simulator Works**

The simulator monitors the input signals of the design. Whenever an input changes, it re-evaluates the design's logic and updates the output accordingly. If none of the inputs change, the output stays the same — the simulator only reacts to changes in input, not to the passage of time by itself.

<img width="1812" height="978" alt="Screenshot 2026-08-07 065921" src="https://github.com/user-attachments/assets/456bd11f-67f0-4831-bc71-cc0ed4877319" />

### 1.2 How Iverilog Fits In

Iverilog is a free, open-source simulator for Verilog designs. The overall flow looks like this:

```
Design + Testbench  →  Iverilog  →  VCD File  →  GTKWave
```

The design and testbench are compiled together by iverilog, which produces a VCD (waveform) file that GTKWave can then display.

<img width="1847" height="922" alt="Screenshot 2026-08-07 065944" src="https://github.com/user-attachments/assets/537f946d-1db3-4ec1-9f8e-9d1cd4384cf9" />

---

## 2️⃣ Labs: Iverilog and GTKWave

### 2.1 Lab 1 — Environment Setup

Install the required tools:

```bash
sudo apt install iverilog
sudo apt install gtkwave
sudo apt install yosys
```

### 2.2 Lab 2 — Compile and Simulate

Compile the design and testbench together:

```bash
iverilog good_shift_reg.v tb_good_shift_reg.v
```

Run the compiled simulation:

```bash
./a.out
```

### 2.3 Lab 3 — Waveform Analysis

Open the generated waveform in GTKWave:

```bash
gtkwave tb_good_shift_reg.vcd
```

<img width="1917" height="1002" alt="gtkwave1" src="https://github.com/user-attachments/assets/d335a648-fd61-4410-bbb1-526935753b6f" />

### 2.4 Verilog Code Analysis

View the design and testbench files side by side:

```bash
gvim good_shift_reg.v -o tb_good_shift_reg.v
```

<img width="1917" height="927" alt="codevlsi" src="https://github.com/user-attachments/assets/cf52333c-f340-4d37-8651-b37da17c33df" />

**Design — `good_shift_reg.v`**

```verilog
module good_shift_reg (input clk, input reset, input d, output reg dout);
reg q1;

always @ (posedge clk, posedge reset)
begin
    if(reset)
    begin
        q1 <= 1'b0;
        dout <= 1'b0;
    end
    else
    begin
        dout <= q1;
        q1 <= d;
    end
end
endmodule
```

**Testbench — `tb_good_shift_reg.v`**

```verilog
module tb_good_shift_reg;
// Inputs
reg clk, reset, d;
// Outputs
wire dout;

// Instantiate the Unit Under Test (UUT)
good_shift_reg uut (
    .clk(clk),
    .reset(reset),
    .d(d),
    .dout(dout)
);

initial begin
$dumpfile("tb_good_shift_reg.vcd");
$dumpvars(0,tb_good_shift_reg);
// Initialize Inputs
clk = 0;
reset = 1;
d = 0;
#3000 $finish;
end
endmodule
```

**🔌 Ports**

| Signal | Direction | Description |
|---|:---:|---|
| `clk` | 🟢 Input | Clock signal |
| `reset` | 🟢 Input | Active-high synchronous reset |
| `d` | 🟢 Input | Serial data input |
| `dout` | 🔵 Output | Registered output |

**Internal signal**

| Signal | Description |
|---|---|
| `q1` | Internal register that holds the data for one clock cycle before it reaches `dout` |

**⚙️ Logic**

The design uses a single `always` block sensitive to `posedge clk` and `posedge reset`:

- When `reset` is high, both `q1` and `dout` are cleared to `0`.
- Otherwise, on every rising edge of `clk`, `dout` takes the previous value of `q1`, and `q1` takes the current value of `d`.

This creates a **2-stage shift** — the input `d` takes two clock cycles to appear on `dout` (`d → q1 → dout`), which is visible in the GTKWave output as `dout` trailing one clock cycle behind `q1`, and `q1` trailing one cycle behind `d`.

---

## 3️⃣ Introduction to Yosys and Logic Synthesis

### 3.1 What Synthesis Does

Simulation confirms that a design *behaves* correctly, but it doesn't produce hardware. Synthesis is the step that translates the Verilog RTL into a **gate-level netlist** — a description of the design built entirely out of standard cells (AND, OR, flip-flops, muxes, etc.) taken from a technology library. Yosys is the open-source synthesis tool used for this step; it reads the `.lib` file for the target technology (SKY130 in this workshop) along with the Verilog source, and maps the RTL onto the available standard cells.

<img width="1893" height="726" alt="Screenshot 2026-08-08 180322" src="https://github.com/user-attachments/assets/35c90da4-c64f-4f47-86d5-b39cef54a11d" />


### 3.2 Lab: Synthesizing the Design

Launch Yosys:

```bash
yosys
```

Read the SKY130 liberty file:

```bash
read_liberty -lib my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the design:

```bash
read_verilog good_shift_reg.v
```

Run synthesis, pointing to the top module:

```bash
synth -top good_shift_reg
```

Map the synthesized design onto the SKY130 standard cells:

```bash
abc -liberty my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

View the resulting gate-level schematic:

```bash
show
```

<img width="1600" height="859" alt="shiftreg" src="https://github.com/user-attachments/assets/a0d6a9cf-44de-4215-b166-295c256b7d04" />


Write out the gate-level netlist:

```bash
write_verilog -noattr good_shift_reg_netlist.v
```

---

## 4️⃣ Takeaways

- ✅ Built a working understanding of RTL design fundamentals in Verilog.
- ✅ Learned how a Simulator, Design, and Testbench fit together.
- ✅ Simulated a shift register design using Iverilog.
- ✅ Analyzed the resulting waveform in GTKWave.
- ✅ Synthesized the design into a gate-level netlist using Yosys and the SKY130 PDK.

---

## 👤 Author

**Arpitha Garrepalli**
[github.com/ArpithaGarrepalli/RTL_Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop)
