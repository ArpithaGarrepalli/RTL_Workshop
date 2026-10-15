# 🔧 VSDBabySoC-3 — Synthesis Recap, Static Timing Analysis, and Physical Design

<p>
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/Tool-OpenSTA-blue" alt="OpenSTA">
  <img src="https://img.shields.io/badge/Tool-OpenROAD-purple" alt="OpenROAD">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the RTL Design and Synthesis Workshop series.

## 📖 Overview

This document covers two connected stages of the VSDBabySoC work. The first half recaps what synthesis does to an RTL design and why a gate-level netlist can look very different from the RTL it came from. The second half moves on to static timing analysis with OpenSTA across multiple PVT corners, in both a VM (pre-route) and a Codespace (post-route), followed by a walkthrough of the OpenROAD physical design flow for the same design.

| | |
|---|---|
| 🛠️ **Tools used** | Yosys, OpenSTA, OpenROAD |
| 🧩 **Example design** | VSDBabySoC on the SKY130 HD platform |
| 📋 **Prerequisites** | Familiarity with RTL synthesis, gate-level simulation, and the Linux terminal |

## 📑 Table of Contents

- 1. Why Internal Waveforms Can Differ
- 2. RTL vs Gate-Level
- 3. Role of the SKY130 Standard-Cell Library
- 4. Important Observations
  - 4.1 RTL Coding Matters
  - 4.2 Synthesis Changes Structure
  - 4.3 Optimization Changes the Implementation
  - 4.4 Technology Libraries Matter
  - 4.5 GLS Is Important
  - 4.6 Internal Signals Can Change
- 5. Complete Synthesis Command Flow
- 6. Static Timing Analysis Using OpenSTA
  - 6.1 Introduction
  - 6.2 Important Timing Parameters
  - 6.3 OpenSTA Commands
- 7. Static Timing Analysis on VSDBabySoC — VM
  - 7.1 Overview
  - 7.2 VM Run TCL Script
  - 7.3 VM Timing Reports — Representative Corners
  - 7.4 VM WNS and TNS Graphs
  - 7.5 Observations
- 8. SKY130 Timing Libraries — PVT Corners
  - 8.1 Introduction
  - 8.2 SKY130 HD Timing Libraries
  - 8.3 Process Corner Categories
- 9. PVT Corner-Based STA — Codespace
  - 9.1 Overview
  - 9.2 Analysis Setup
  - 9.3 Codespace TCL Script
  - 9.4 Timing Report Commands
  - 9.5 PVT Corner Coverage
  - 9.6 Timing Reports — Four Representative Corners
  - 9.7 Summary Report
  - 9.8 WNS and TNS Graphs
  - 9.9 Observations
- 10. Physical Design Flow Using OpenROAD
  - 10.1 Introduction
  - 10.2 Synthesis
  - 10.3 Floorplanning
  - 10.4 Placement
  - 10.5 Routing
  - 10.6 Observations
- 11. Takeaways
- Author

---

## 1️⃣ Why Internal Waveforms Can Differ

Synthesis may perform:

- Logic optimization
- Constant propagation
- Logic merging
- Redundant logic removal
- Technology mapping
- Hierarchy flattening
- Internal signal renaming

Because of this, the internal gate-level structure does not have to be identical to the RTL structure. A different internal waveform does not automatically indicate a functional error — the important comparison is the behaviour of the relevant observable signals and outputs.

---

## 2️⃣ RTL vs Gate-Level

| RTL | Gate-Level |
|---|---|
| Higher abstraction | Lower abstraction |
| Behaviour-oriented | Structure-oriented |
| Easier to read | More detailed |
| Technology independent | Technology mapped |
| Describes intended functionality | Describes cell connectivity |
| Used for functional verification | Used for post-synthesis verification |

---

## 3️⃣ Role of the SKY130 Standard-Cell Library

The standard-cell library determines which technology-specific cells can be used during mapping. The relationship is:

```text
RTL
 ↓
Generic Logic
 ↓
Technology Mapping
 ↓
SKY130 Standard Cells
```

Examples of standard-cell types include:

```text
INV
NAND
NOR
AND
OR
XOR
MUX
D Flip-Flop
```

The selected technology library therefore influences the structure of the final gate-level netlist.

---

## 4️⃣ Important Observations

### 4.1 RTL Coding Matters

The MUX experiments demonstrated that RTL coding style influences how synthesis interprets the design.

### 4.2 Synthesis Changes Structure

The gate-level netlist can be structurally very different from the RTL.

### 4.3 Optimization Changes the Implementation

Redundant or unnecessary logic may be simplified or removed while preserving functionality.

### 4.4 Technology Libraries Matter

The available technology library determines the cells used during technology mapping.

### 4.5 GLS Is Important

RTL simulation verifies the original RTL, while GLS verifies the synthesized gate-level implementation.

### 4.6 Internal Signals Can Change

Synthesis may rename, optimize, or remove internal signals, so functional comparison should focus on meaningful signals and outputs.

---

## 5️⃣ Complete Synthesis Command Flow

```bash
yosys

read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
read_verilog -I src/include src/module/clk_gate.v

read_liberty -lib lib/avsdpll.lib
read_liberty -lib lib/avsddac.lib
read_liberty -lib lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

dfflibmap -liberty lib/sky130_fd_sc_hd__tt_025C_1v80.lib

opt

abc -liberty lib/sky130_fd_sc_hd__tt_025C_1v80.lib

check

show vsdbabysoc

flatten
setundef -zero
clean -purge
rename -enumerate

write_verilog -noattr netlist/babysoc_netlist_new.v
```

---

## 6️⃣ Static Timing Analysis Using OpenSTA

### 6.1 Introduction

Static Timing Analysis (STA) evaluates the timing behaviour of a digital circuit without requiring exhaustive functional simulation. It analyses signal propagation delays and checks whether timing paths satisfy the specified timing constraints.

OpenSTA is an open-source static timing analysis tool. In this work, it was used to examine timing paths in VSDBabySoC using SKY130 HD standard-cell timing libraries. The analysis involves loading the design netlist, timing constraints, and libraries, generating timing reports, and evaluating timing behaviour across different Process, Voltage, and Temperature (PVT) conditions.

### 6.2 Important Timing Parameters

| Parameter | Description |
|---|---|
| Startpoint | Beginning of a timing path |
| Endpoint | End of a timing path |
| Arrival time | Time at which a signal reaches the endpoint |
| Required time | Time by which the signal must arrive |
| Slack | Difference between required time and arrival time, interpreted according to the timing check |
| WNS | Worst Negative Slack; the worst reported timing slack |
| TNS | Total Negative Slack; accumulated negative slack across violating paths |

Positive slack generally indicates that a path meets its timing requirement. Negative slack indicates a timing violation.

### 6.3 OpenSTA Commands

Check whether OpenSTA is available in the VM:

```bash
which sta
sta -version
```

Inspect minimum- and maximum-delay timing paths:

```tcl
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout}
```

The report provides timing-path details, including net information, capacitance, slew, input pins, and fanout.

---

## 7️⃣ Static Timing Analysis on VSDBabySoC — VM

### 7.1 Overview

Static Timing Analysis was performed on the synthesized VSDBabySoC design in the virtual machine using OpenSTA and SKY130 HD timing libraries. This analysis focuses on the **pre-route** design. The results help identify timing violations and understand how timing behaviour changes across PVT corners.

The VM screenshots are stored in the `images/` directory. The original filenames contain `(report checks)`.

### 7.2 VM Run TCL Script

The two parts of the VM `run.tcl` file:

**Part 1**

<img width="1547" height="803" alt="Screenshot 2026-10-15 214859" src="https://github.com/user-attachments/assets/4ed1a843-2eb6-4ee6-870f-46ba8af2ff52" />


**Part 2**

<img width="1103" height="797" alt="Screenshot 2026-10-15 215034" src="https://github.com/user-attachments/assets/672740ac-4b01-4082-8fbe-d61fbfcecbda" />


### 7.3 VM Timing Reports — Representative Corners

Each selected corner has two screenshot parts.

**Corner 1: SS, -40°C, 1.28 V**

This corner produced the following pre-route timing results:

| Metric | Value |
|---|---:|
| WNS | -62.56 ns |
| TNS | -13957.59 ns |

<img width="486" height="610" alt="Screenshot 2026-10-15 215142" src="https://github.com/user-attachments/assets/de032a06-99a8-405c-a219-d27d368055ea" />

**Corner 2: SS, -40°C, 1.35 V**

<img width="682" height="737" alt="Screenshot 2026-10-15 215257" src="https://github.com/user-attachments/assets/27bdd98e-b586-4346-8568-54f5ee6ff2c8" />

<img width="540" height="737" alt="Screenshot 2026-10-15 215610" src="https://github.com/user-attachments/assets/3870ee5e-7ac0-48ff-8c0c-5a6eabf93933" />

**Corner 3: SS, -40°C, 1.44 V**

<img width="525" height="563" alt="Screenshot 2026-10-15 215704" src="https://github.com/user-attachments/assets/db70267a-3c26-4478-8238-51d4b1d88191" />

<img width="632" height="686" alt="Screenshot 2026-10-15 215733" src="https://github.com/user-attachments/assets/641f6b58-e43a-4d6f-bf08-c9cf7edf48a7" />

**Corner 4: SS, 100°C, 1.40 V**
<img width="736" height="621" alt="Screenshot 2026-10-15 215932" src="https://github.com/user-attachments/assets/f12d50dd-7905-45fd-8d78-a68259314228" />
<img width="710" height="682" alt="Screenshot 2026-10-15 220005" src="https://github.com/user-attachments/assets/6a82b34f-bdb1-4d18-a482-fa2cca987a6a" />

### 7.4 VM WNS and TNS Graphs

The WNS graph helps identify the worst reported slack across the evaluated corners. The TNS graph shows the accumulated negative slack of violating paths.

**VM WNS graph**

<img width="1492" height="581" alt="Screenshot 2026-10-15 220416" src="https://github.com/user-attachments/assets/e698817f-6ddd-42e6-b22a-2fbebe31f4b4" />

**VM TNS graph**

<img width="1510" height="607" alt="Screenshot 2026-10-15 220445" src="https://github.com/user-attachments/assets/14f7e65b-cc37-464d-90d8-3b34517ebab4" />


### 7.5 Observations

- The VM analysis examines the synthesized design before routing.
- The reported timing results vary across the selected timing libraries.
- The `ss_n40C_1v28` corner shows substantial negative WNS and TNS.
- WNS and TNS provide complementary information about timing violations.
- These results belong to the VM pre-route analysis and must be distinguished from the Codespace post-route results.

---

## 8️⃣ SKY130 Timing Libraries — PVT Corners

### 8.1 Introduction

Timing libraries provide timing and electrical models for standard cells, allowing tools to estimate cell delays under specified operating conditions. PVT stands for:

- **Process:** manufacturing-process variation.
- **Voltage:** supply-voltage condition.
- **Temperature:** operating-temperature condition.

Different PVT conditions can produce different cell delays, so analysing multiple corners helps evaluate timing behaviour across operating conditions.

### 8.2 SKY130 HD Timing Libraries

The SKY130 HD timing-library directory was inspected in the VM. The library filenames identify their process corner, temperature, and supply-voltage conditions.

The libraries were used as inputs to the timing analysis. The GitHub repository documents the library inspection and analysis results; the complete timing-library files do not need to be uploaded.

### 8.3 Process Corner Categories

| Corner | Description |
|---|---|
| FF | Fast-Fast |
| SS | Slow-Slow |
| TT | Typical-Typical |

The VM analysis evaluated 16 corners: 6 FF, 8 SS, and 2 TT.

---

## 9️⃣ PVT Corner-Based STA — Codespace

### 9.1 Overview

A separate multi-corner Static Timing Analysis was performed in Codespace using the VSDBabySoC physical-design environment. Unlike the VM analysis, the Codespace report evaluates the **routed** design using OpenROAD and the associated timing constraints, covering minimum- and maximum-delay timing behaviour across 16 PVT corners.

The Codespace screenshots are stored in `images_codespace/`. Their filenames use `rpt(1)` and `rpt(2)` for the two parts of each corner report.

### 9.2 Analysis Setup

The Codespace report documents the following files and directories:

| Item | Path |
|---|---|
| Routed design database | `results/sky130hd/vsdbabysoc/base/5_route.odb` |
| Timing constraints | `results/sky130hd/vsdbabysoc/base/5_route.sdc` |
| Standard-cell timing libraries | `designs/sky130hd/vsdbabysoc/libs/timing/` |
| PLL timing library | `designs/sky130hd/vsdbabysoc/lib/avsdpll.lib` |
| DAC timing library | `designs/sky130hd/vsdbabysoc/lib/avsddac.lib` |
| Report directory | `reports/sta_minmax_16/` |
| Summary CSV | `reports/sta_minmax_16/minmax_slack_summary.csv` |
| TCL script | `/tmp/sta_minmax.tcl` |

These paths describe the Codespace analysis environment.

### 9.3 Codespace TCL Script

**Additional STA script**
<img width="1462" height="462" alt="Screenshot 2026-10-15 220756" src="https://github.com/user-attachments/assets/24de5bab-6097-4f2a-8268-e3fb7dc08e32" />


### 9.4 Timing Report Commands

The following commands were used to inspect timing paths and report slack metrics:

```tcl
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout}

report_wns -min
report_wns -max

report_tns -min
report_tns -max
```

Minimum-delay and maximum-delay checks evaluate different timing requirements and must be interpreted separately.

### 9.5 PVT Corner Coverage

| Corner group | Number of corners |
|---|---:|
| FF | 6 |
| SS | 8 |
| TT | 2 |
| **Total** | **16** |

### 9.6 Timing Reports — Four Representative Corners

The post-route report recorded maximum-delay violations in five SS corners. The following four reports show representative results for the corners with the most negative reported WNS values.

**Corner 1: SS, -40°C, 1.28 V**

| Metric | Value |
|---|---:|
| WNS | -20.83 ns |
| TNS | -5815.72 ns |

<img width="1026" height="607" alt="Screenshot 2026-10-15 220949" src="https://github.com/user-attachments/assets/6119d946-cfcd-4b7d-bd12-24a38d92f613" />

<img width="1295" height="737" alt="Screenshot 2026-10-15 221048" src="https://github.com/user-attachments/assets/5460589a-de9c-420a-aff6-f2d5248bb6d0" />

**Corner 2: SS, -40°C, 1.35 V**

| Metric | Value |
|---|---:|
| WNS | -10.66 ns |
| TNS | -2019.57 ns |

<img width="1310" height="686" alt="Screenshot 2026-10-15 221145" src="https://github.com/user-attachments/assets/fbae6a4b-2796-4c99-9826-284a68130865" />

<img width="1361" height="621" alt="Screenshot 2026-10-15 221241" src="https://github.com/user-attachments/assets/55334e02-896b-4b97-b6ef-810e3bea098c" />

**Corner 3: SS, -40°C, 1.40 V**

| Metric | Value |
|---|---:|
| WNS | -6.36 ns |
| TNS | -762.02 ns |

<img width="1333" height="686" alt="Screenshot 2026-10-15 221335" src="https://github.com/user-attachments/assets/be463202-595f-4c03-bdf7-e01e9a9f3076" />

<img width="1318" height="718" alt="Screenshot 2026-10-15 221427" src="https://github.com/user-attachments/assets/caf8e9a4-0e2a-4ee3-a550-214ac2f923eb" />

**Corner 4: SS, -40°C, 1.44 V**

| Metric | Value |
|---|---:|
| WNS | -3.82 ns |
| TNS | -168.54 ns |

<img width="1275" height="682" alt="Screenshot 2026-10-15 221456" src="https://github.com/user-attachments/assets/f208355c-1d27-4591-b806-6edeb31db0ac" />

<img width="1215" height="707" alt="Screenshot 2026-10-15 221538" src="https://github.com/user-attachments/assets/9fae90d4-7078-4ed8-9269-96cb484b728b" />

### 9.7 Summary Report

The summary report provides an overview of the multi-corner timing analysis.

/*<img width="1566" height="561" alt="Screenshot 2026-10-15 221620" src="https://github.com/user-attachments/assets/97973c70-2704-4edb-b1c4-5dd71be17704" />

### 9.8 WNS and TNS Graphs

**Codespace WNS graph**
<img width="1448" height="427" alt="Screenshot 2026-10-15 225732" src="https://github.com/user-attachments/assets/975ec787-9b55-4c90-91da-ef4b47e47567" />

**Codespace TNS graph**
<img width="1423" height="625" alt="Screenshot 2026-10-15 225716" src="https://github.com/user-attachments/assets/fa455083-2b91-4c54-bc31-81d7f48cf876" />


### 9.9 Observations

- The Codespace analysis evaluates the routed VSDBabySoC design.
- Five SS corners showed maximum-delay timing violations.
- The `ss_n40C_1v28` corner produced the most negative reported WNS.
- WNS indicates the worst reported slack, while TNS represents accumulated negative slack.
- The VM and Codespace results represent different analysis stages and should not be treated as interchangeable.
- The full set of corner reports is retained in the repository as supporting evidence.

---

## 🔟 Physical Design Flow Using OpenROAD

### 10.1 Introduction

The OpenROAD physical-design flow was explored using VSDBabySoC and the SKY130 HD platform. The work includes synthesis, floorplanning, placement, and routing. The design configuration used was:

```bash
./designs/sky130hd/vsdbabysoc/config.mk
```

### 10.2 Synthesis

Synthesis generates a gate-level implementation from the RTL using the selected technology libraries.

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
```

### 10.3 Floorplanning

Floorplanning establishes the initial physical organisation of the design.

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
```

The `gui_floorplan` results were inspected to view the floorplan layout and its associated timing information.

<img width="1482" height="786" alt="Screenshot 2026-10-15 221901" src="https://github.com/user-attachments/assets/96688fb2-de4b-447f-9feb-f4ebc88335db" />
<img width="1517" height="761" alt="Screenshot 2026-10-15 221933" src="https://github.com/user-attachments/assets/caecdf0f-9eee-4a2a-a582-9925e63a4758" />


### 10.4 Placement

Placement assigns physical locations to standard cells within the placement region.

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
```

The `gui_placement` results were inspected to view the placement layout and timing information.

<img width="1482" height="693" alt="Screenshot 2026-10-15 222012" src="https://github.com/user-attachments/assets/651e5a02-3b4c-4c53-bf61-da3315790758" />

<img width="1482" height="563" alt="Screenshot 2026-10-15 222119" src="https://github.com/user-attachments/assets/c46ccfae-f020-47b9-94d3-96eb602f5c4e" />

### 10.5 Routing

Routing connects the placed standard cells through the available metal layers.

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
```

The `gui_routing` results were inspected to view the routed layout and associated timing information.

<img width="1362" height="676" alt="Screenshot 2026-10-15 222230" src="https://github.com/user-attachments/assets/623ab548-2628-4df0-8904-0bf521e26c2f" />
<img width="1386" height="755" alt="Screenshot 2026-10-15 222412" src="https://github.com/user-attachments/assets/0ed0993e-4a67-4969-93f2-894e345166dd" />

### 10.6 Observations

- `gui_floorplan` provides a view of the floorplanned design.
- `gui_placement` provides a view of the placed standard cells.
- `gui_routing` provides a view of the routed design.
- The GUI views help connect the physical layout with timing information reported during the flow.
- The routed design can be used for post-route timing analysis.

---

## 1️⃣1️⃣ Takeaways

The flow studied across this work:

```text
RTL Coding
     ↓
RTL Simulation
     ↓
Functional Verification
     ↓
Synthesis
     ↓
Logic Optimization
     ↓
Sequential Cell Mapping
     ↓
Technology Mapping
     ↓
Gate-Level Netlist
     ↓
Gate-Level Simulation
     ↓
Functional Comparison
     ↓
Static Timing Analysis
     ↓
Physical Design (Floorplan → Placement → Routing)
```

- ✅ Connected RTL coding style to the hardware synthesis infers, using the MUX experiments, and extended that understanding to a larger hierarchical design with VSDBabySoC.
- ✅ Understood how optimization, technology mapping, and the choice of technology library change a gate-level netlist relative to its RTL.
- ✅ Used static timing analysis with OpenSTA, including TCL scripts and timing commands, to inspect timing paths.
- ✅ Applied the timing metrics — arrival time, required time, slack, WNS, and TNS — to real reports.
- ✅ Evaluated timing across 16 PVT corners (6 FF, 8 SS, 2 TT) using the SKY130 HD timing libraries.
- ✅ Distinguished VM pre-route results from Codespace post-route results, rather than treating them as interchangeable.
- ✅ Explored the OpenROAD flow — synthesis, floorplanning, placement, and routing — through its GUI views.
- ✅ Identified the operating conditions under which maximum-delay requirements were not met.

The VM report showed substantial negative slack in the `ss_n40C_1v28` pre-route analysis. The Codespace post-route report also identified five SS corners with maximum-delay violations, with `ss_n40C_1v28.lib` producing the most negative reported WNS. These results show why timing has to be evaluated across multiple PVT conditions and interpreted according to the implementation stage.

More broadly, RTL design is not only about writing code that produces the expected simulation output. It also means understanding how synthesis interprets the RTL, how optimization changes the implementation, how technology libraries influence the mapped hardware, and how timing constraints, standard-cell libraries, PVT conditions, and physical implementation together determine the timing behaviour of a digital circuit.

---

## 👤 Author

**Arpitha Garrepalli**
Department of Electronics and Communication Engineering (ECE)
Anurag University
