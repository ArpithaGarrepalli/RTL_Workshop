# 🔧 Module 2 — Timing Libraries, Synthesis Approaches & Flip-Flop Coding

<p align="center">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf?style=for-the-badge" alt="Verilog">
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-2f6fed?style=for-the-badge" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-e67e22?style=for-the-badge" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-27ae60?style=for-the-badge" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red?style=for-the-badge" alt="SKY130">
</p>

<p align="center"><em>Part of the <a href="https://github.com/ArpithaGarrepalli/RTL_Workshop">RTL Workshop</a> series</em></p>

---

## 🎯 Objectives

> Understand **timing libraries** and the **SKY130 PDK**, explore **hierarchical vs. flattened synthesis**, study different **flip-flop coding styles**, walk through the complete **RTL simulation and synthesis flow**, and observe how **Yosys optimizes RTL** into efficient gate-level hardware.

| | |
|---|---|
| 🛠️ **Tools used** | Icarus Verilog · GTKWave · Yosys |
| 📚 **PDK** | SKY130 (`sky130_fd_sc_hd__tt_025C_1v80.lib`) |
| 🧩 **Example designs** | D flip-flops (async reset/set, sync reset), `mul2`, `mult8` |

---

## 📑 Table of Contents

| # | Section |
|---|---|
| 1 | [Timing Libraries](#1️⃣-timing-libraries) |
| 2 | [Hierarchical and Flattened Synthesis](#2️⃣-hierarchical-and-flattened-synthesis) |
| 3 | [Flip-Flop Coding Styles](#3️⃣-flip-flop-coding-styles) |
| 4 | [RTL Simulation and Synthesis Flow](#4️⃣-rtl-simulation-and-synthesis-flow) |
| 5 | [Interesting Optimization](#5️⃣-interesting-optimization) |
| 6 | [Overall Result](#-overall-result) |
| 7 | [Conclusion](#-conclusion) |

---

## 1️⃣ Timing Libraries

### 📘 1.1 SKY130 PDK

The **SKY130 PDK** contains the technology information and standard-cell libraries required to design and synthesize digital circuits using **130 nm CMOS technology**.

The timing library used during the workshop was:

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

### 🔍 1.2 Understanding `tt_025C_1v80`

| Term | Meaning |
|:---:|---|
| 🟢 **tt** | Typical process corner |
| 🌡️ **025C** | Temperature of 25°C |
| ⚡ **1v80** | Supply voltage of 1.8 V |

### 📂 1.3 Exploring the `.lib` File

The `.lib` file contains information about standard cells — including their timing, power characteristics, and operating conditions — used during synthesis and technology mapping.

<p align="center">
<img width="800" alt="SKY130 timing library file" src="https://github.com/user-attachments/assets/ff0007e7-02cf-4d85-9d2e-5e1e4330d753" />
</p>

**Figure 1:** SKY130 timing library file.

> ✅ **Result:** The SKY130 timing library was successfully opened, and its library and operating-condition information was examined.

---

## 2️⃣ Hierarchical and Flattened Synthesis

### 🗂️ 2.1 Hierarchical Synthesis

Hierarchical synthesis keeps the **original module structure** of the RTL design intact. Individual modules remain separate, making the design easier to understand, organize, and debug.

<img width="1912" height="997" alt="vlsi1" src="https://github.com/user-attachments/assets/24d239bb-1302-4cf2-8e9d-dac04aaadbe2" />


**Figure 2:** Hierarchical synthesized design.

> ✅ **Result:** The synthesized multi-module design retained its original module structure and connections.

### 🌐 2.2 Flattened Synthesis

Flattened synthesis combines all modules into a **single design structure**, allowing the synthesis tool to optimize across module boundaries.

```text
flatten
```

<img width="1907" height="1007" alt="vlsi4" src="https://github.com/user-attachments/assets/1d5ee905-e098-42fb-b27d-4e2416dcad49" />

**Figure 3:** Flattened synthesized design.

### ⚖️ 2.3 Comparison

<table>
<tr><th>Feature</th><th>🟦 Hierarchical Synthesis</th><th>🟧 Flattened Synthesis</th></tr>
<tr><td>Module structure</td><td>Preserved</td><td>Removed</td></tr>
<tr><td>Optimization</td><td>Limited between modules</td><td>Possible across the complete design</td></tr>
<tr><td>Debugging</td><td>Easier</td><td>More difficult</td></tr>
<tr><td>Design structure</td><td>Modular</td><td>Single structure</td></tr>
</table>

> ✅ **Result:** The difference between hierarchical and flattened synthesis was studied, especially in terms of module structure, optimization, and debugging.

---

## 3️⃣ Flip-Flop Coding Styles

Flip-flops are sequential logic elements used to store binary information. Three different D flip-flop coding styles were studied.

### 🔴 3.1 Asynchronous Reset D Flip-Flop

An asynchronous reset changes the output **immediately** when the reset signal becomes active, without waiting for the clock edge.

```verilog
module dff_asyncres (
    input clk,
    input async_reset,
    input d,
    output reg q
);

always @(posedge clk, posedge async_reset)
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;

endmodule
```

**⚙️ Working:** When `async_reset` is active, `q` immediately becomes `0`. When inactive, `d` is transferred to `q` on the rising edge of `clk`.

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```

<img width="1917" height="982" alt="dff_asyncres" src="https://github.com/user-attachments/assets/3f0465b2-b915-429c-a83b-3a5db8275f5c" />


**Figure 4:** Simulation waveform of the asynchronous-reset D flip-flop.

> ✅ **Result:** The asynchronous-reset D flip-flop was successfully simulated. The GTKWave waveform verified the relationship between clock, reset, input, and output.

### 🟢 3.2 Asynchronous Set D Flip-Flop

An asynchronous set forces the output to logic **`1`** as soon as the set signal becomes active, without waiting for a clock edge.

```verilog
module dff_async_set (
    input clk,
    input async_set,
    input d,
    output reg q
);

always @(posedge clk, posedge async_set)
    if (async_set)
        q <= 1'b1;
    else
        q <= d;

endmodule
```

**⚙️ Working:** When `async_set` is active, `q` becomes `1` immediately. When inactive, `d` is captured on the rising edge of `clk`.

<p align="center">
<img width="500" alt="Async set DFF waveform" src="https://github.com/user-attachments/assets/0429b814-9eb7-4415-9d54-3361713e12cd" />
</p>

**Figure 5:** Simulation waveform of the asynchronous-set D flip-flop.

> ✅ **Result:** The asynchronous-set D flip-flop was implemented and its operation was studied through simulation.

### 🔵 3.3 Synchronous Reset D Flip-Flop

A synchronous reset affects the output **only** when the active clock edge occurs.

```verilog
module dff_syncres (
    input clk,
    input async_reset,
    input sync_reset,
    input d,
    output reg q
);

always @(posedge clk)
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;

endmodule
```

**⚙️ Working:** When `sync_reset` is active at the rising clock edge, `q` becomes `0`. Otherwise, `d` is transferred to `q` at the clock edge.

<img width="1917" height="1012" alt="dff_syncres" src="https://github.com/user-attachments/assets/108eda0b-bbb8-4aef-b4cd-3bd725bd28e9" />


**Figure 6:** Simulation waveform of the synchronous-reset D flip-flop.

> ✅ **Result:** The synchronous-reset D flip-flop was successfully simulated and its waveform behavior was observed.

---

## 4️⃣ RTL Simulation and Synthesis Flow

The RTL designs were simulated using **Icarus Verilog** and **GTKWave**, then synthesized using **Yosys** and mapped to the SKY130 standard-cell library.

### 🧪 4.1 Simulation Using Icarus Verilog

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```

> ✅ **Result:** The simulation was successfully completed. The GTKWave waveform was used to verify the relationship between clock, reset, input, and output.

### ⚡ 4.2 Synthesis Using Yosys

<details>
<summary>📜 <b>Full Yosys command sequence</b></summary>

```text
yosys
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog /path/to/dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

</details>

<img width="1917" height="992" alt="vlsi5" src="https://github.com/user-attachments/assets/846ab85b-c601-4239-ab1d-ed1b0309c3f1" />


**Figure 7:** Synthesized gate-level representation.

Also synthesized and mapped using the flip-flop flow:

<p align="center">
<img width="800" alt="Flip-flop synthesized gate-level view" src="https://github.com/user-attachments/assets/567ea919-704c-48f4-92c0-6835be3f2f9a" />
<img width="700" alt="Flip-flop synthesized gate-level detail" src="https://github.com/user-attachments/assets/e829b4a2-7a52-4a02-9e21-2b5da865fb70" />
</p>

**Figure 8:** Synthesized gate-level representation of the flip-flop design.

> ✅ **Result:** The RTL design was successfully synthesized and mapped to standard cells from the SKY130 library. The flip-flop RTL design was also synthesized and technology-mapped, with the resulting gate-level representation viewed using Yosys.

---

## 5️⃣ Interesting Optimization

RTL synthesis tools optimize a design while preserving its required functionality. This section shows how Yosys simplifies **constant multiplication** into efficient hardware.

### ✖️ 5.1 `mul2` Optimization

```verilog
module mul2 (
    input [2:0] a,
    output [3:0] y
);

assign y = a * 2;

endmodule
```

The input `a` is multiplied by `2` and assigned to output `y`.

```text
yosys
read_verilog mul2.v
prep -top mul2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr mul2_net.v
gvim mul2_net.v
```

<img width="1910" height="1021" alt="vlsi7" src="https://github.com/user-attachments/assets/db5c19fa-8e02-462d-b07e-97e9cdd2852c" />

**Figure 9:** Yosys synthesis and optimization result for `mul2`.

> ✅ **Result:** The `mul2` design was successfully synthesized. Yosys optimized the multiplication into pure wiring (a left shift) and generated the corresponding synthesized Verilog netlist.

### ✖️ 5.2 `mult8` Optimization

```verilog
module mult8 (
    input [2:0] a,
    output [5:0] y
);

assign y = a * 9;

endmodule
```

The input `a` is multiplied by `9` and assigned to output `y`.

```text
yosys
read_verilog mult8.v
prep -top mult8
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr mult8_net.v
gvim mult8_net.v
```
<img width="1912" height="1015" alt="vlsi9" src="https://github.com/user-attachments/assets/61ae3bea-16f5-46ee-a095-562d9be30c19" />


**Figure 10:** Yosys synthesis and optimization result for `mult8`.

> ✅ **Result:** The `mult8` design was successfully synthesized and optimized. The generated netlist shows the optimized hardware representation obtained from the original RTL code.

### 📄 5.3 Generated Synthesized Netlists

```text
write_verilog -noattr mul2_net.v
write_verilog -noattr mult8_net.v
gvim mul2_net.v
gvim mult8_net.v
```

<p align="center">
<img width="800" alt="Generated synthesized netlist" src="https://github.com/user-attachments/assets/38ec40fb-813f-444a-9e2c-0501a0047cd4" />
</p>

**Figure 11:** Generated synthesized Verilog netlist.

> ✅ **Result:** The synthesized Verilog netlists were successfully generated and examined, showing how the original RTL code was converted into an optimized hardware representation.

---

## 🏁 Overall Result

- ✅ The SKY130 timing library was explored and its operating conditions understood.
- ✅ Hierarchical and flattened synthesis approaches were studied and compared.
- ✅ Three D flip-flop coding styles (async reset, async set, sync reset) were implemented and verified.
- ✅ RTL designs were simulated using Icarus Verilog and GTKWave.
- ✅ Designs were synthesized and technology-mapped using Yosys with the SKY130 standard-cell library.
- ✅ Constant-multiplication optimizations (`×2`, `×9`) were shown to resolve into pure wiring instead of multiplier hardware.

---

## 📌 Conclusion

Module 2 provided practical experience with timing libraries, synthesis techniques, flip-flop coding styles, RTL simulation, waveform analysis, and technology mapping — building a clear understanding of how an RTL design is converted into an optimized gate-level implementation using standard-cell libraries.

---

## 👤 Author

**Arpitha Garrepalli**
📎 [github.com/ArpithaGarrepalli/RTL_Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop)
