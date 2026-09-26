# 🕰️ Module 4 — Clock Tree Synthesis, Timing Analysis & Physical Optimization

<p>
  <img src="https://img.shields.io/badge/Flow-OpenLane%20%2F%20OpenROAD-green" alt="OpenLane / OpenROAD">
  <img src="https://img.shields.io/badge/Synthesis-Yosys-red" alt="Yosys">
  <img src="https://img.shields.io/badge/STA-OpenSTA-orange" alt="OpenSTA">
  <img src="https://img.shields.io/badge/PDK-SKY130-blue" alt="SKY130">
  <img src="https://img.shields.io/badge/Constraints-SDC-yellow" alt="SDC">
  <img src="https://img.shields.io/badge/Env-Docker-lightblue" alt="Docker">
</p>

> Part of the SKY130 Physical Design module series.

## 📖 Overview

This document walks through the physical-implementation stage of the `picorv32a` design on the OpenLane/OpenROAD flow with the SkyWater SKY130 standard-cell library. It covers standard-cell placement, Clock Tree Synthesis (CTS), clock buffering and distribution, setup/hold timing analysis, clock skew, crosstalk, and how the supporting Liberty, LEF, SDC, and Tcl files drive each stage.

|  |  |
|---|---|
| 🛠️ **Tools used** | OpenLane, OpenROAD, Yosys, OpenSTA, SKY130 PDK, Liberty (.lib), LEF, SDC, Tcl, Docker |
| 🧩 **Example design(s)** | `picorv32a` |
| 📋 **Prerequisites** | OpenLane/OpenROAD environment running in Docker, SKY130 PDK installed, a synthesized `picorv32a` netlist, base SDC constraints defined |

## 📑 Table of Contents

1. Technology Libraries and Physical Files
   1.1 Standard-Cell Timing Corners (Fast / Slow / Typical)
   1.2 LEF File
2. Physical Design Flow Overview
3. Synthesis and Placement
   3.1 Running Synthesis
   3.2 Standard-Cell Placement
   3.3 Expanded Placement View
4. Timing Constraints and Configuration
   4.1 Base SDC File
   4.2 Pre-CTS and New Tcl Configuration
5. Clock Tree Synthesis
   5.1 CTS Objectives and Clock-Tree Structure
   5.2 CTS Configuration (cts.tcl)
6. Clock Buffers and Clock Distribution
   6.1 Clock Buffer Insertion
   6.2 Clock Net Shielding
7. Setup and Hold Timing Analysis
   7.1 Setup Time
   7.2 Hold Time
   7.3 Setup Analysis with an Ideal Clock
   7.4 Hold Time Analysis
8. Clock Skew and Crosstalk
   8.1 Clock Skew
   8.2 Crosstalk and Delta Delay
9. Timing Reports and Slack Analysis
   9.1 Timing Slack
   9.2 Maximum-Delay (Setup) Analysis
   9.3 Minimum-Delay (Hold) Analysis
   9.4 Delay Tables
10. Physical Design Configuration and Routing Information
    10.1 Converting Labels to Ports
    10.2 Grid-to-Track Conversion and `tracks.info`
- Author

---

## 1️⃣ Technology Libraries and Physical Files

Technology files tell the physical-design tools how standard cells, routing layers, and timing behave. The SKY130 standard-cell library ships fast, slow, and typical characterization corners, plus a LEF description of each cell's physical footprint.

### 1.1 Standard-Cell Timing Corners (Fast / Slow / Typical)

**Fast corner** — cells characterized under conditions where they switch relatively quickly; used to study minimum-delay behavior and hold timing.

<img width="921" height="531" alt="Screenshot 2026-09-26 181946" src="https://github.com/user-attachments/assets/1381335c-9327-4096-9f7f-f1af97b4d5ad" />

**Slow corner** — cells characterized under conditions where they switch relatively slowly; relevant to maximum-delay and setup timing.
<img width="922" height="547" alt="Screenshot 2026-09-26 182044" src="https://github.com/user-attachments/assets/3df5afb7-fa32-4c46-9703-0a069b0d1f63" />


**Typical corner** — nominal characterized conditions, used for baseline/typical timing evaluation.

<img width="758" height="490" alt="Screenshot 2026-09-26 182225" src="https://github.com/user-attachments/assets/9eecd30d-d5c0-41ec-a600-d307fcf12d7e" />

### 1.2 LEF File

The LEF format captures the physical characteristics of a standard cell — dimensions, pin locations, and routing-related geometry needed for placement and routing.

<img width="652" height="486" alt="Screenshot 2026-09-26 182251" src="https://github.com/user-attachments/assets/63c4de48-afce-4b23-913f-cd85d991d859" />

---

## 2️⃣ Physical Design Flow Overview

The physical-design flow turns a synthesized netlist into an implementation by placing cells, building the clock network, routing connections, and checking timing:

```text
RTL Design
    |
    v
Synthesis
    |
    v
Floorplanning
    |
    v
Standard-Cell Placement
    |
    v
Placement Optimization
    |
    v
Clock Tree Synthesis
    |
    v
Clock Distribution
    |
    v
Timing Analysis
    |
    v
Routing and Optimization
    |
    v
Physical Verification
```

**Synthesis** maps RTL onto standard cells from the target library. **Floorplanning** sets the core area and design boundaries. **Placement** assigns physical locations to cells. **CTS** builds the clock distribution network with buffers and clock cells. **Routing** wires cells together on the metal layers. **Timing analysis** checks whether the implementation meets its constraints.

---

## 3️⃣ Synthesis and Placement

### 3.1 Running Synthesis

Synthesis converts the RTL into a gate-level netlist mapped onto the chosen standard-cell library; this netlist is what feeds the rest of the physical flow.
<img width="913" height="537" alt="Screenshot 2026-09-26 183403" src="https://github.com/user-attachments/assets/96c0ab6d-b4a1-4c4e-b62d-3d3b4156f1cc" />

### 3.2 Standard-Cell Placement

Placement locates standard cells inside the core area, weighing cell density, wirelength, connectivity, and timing — a good placement keeps interconnect delay down and eases later timing optimization.

<img width="922" height="547" alt="Screenshot 2026-09-26 183427" src="https://github.com/user-attachments/assets/8ffea235-abb8-4486-a77a-529346306cb9" />

### 3.3 Expanded Placement View

A zoomed-in view of the placement makes it easier to see how cells are physically distributed and spatially related within the core.

<img width="925" height="533" alt="Screenshot 2026-09-26 183505" src="https://github.com/user-attachments/assets/050dac76-f644-4815-8de1-b5562dbf824a" />

---

## 4️⃣ Timing Constraints and Configuration

Timing constraints capture the design's operating requirements and feed Static Timing Analysis. SDC (Synopsys Design Constraints) is the format used here to define the clock, I/O delays, and timing uncertainty.

### 4.1 Base SDC File

`my_base.sdc` holds the design's core timing requirements, typically including:

- Clock period and clock definition
- Input and output delays
- Clock uncertainty
- Input transition times
- Output loads

<img width="903" height="525" alt="Screenshot 2026-09-26 185052" src="https://github.com/user-attachments/assets/0fbdb50f-4aa7-405e-88ee-b4b46faa424b" />

### 4.2 Pre-CTS and New Tcl Configuration

The pre-CTS configuration holds the settings used for implementation just before Clock Tree Synthesis runs. A separate Tcl configuration file carries additional design/flow settings for physical implementation — these must stay consistent with the chosen technology, design, and timing requirements.

<img width="913" height="741" alt="Screenshot 2026-09-26 185544" src="https://github.com/user-attachments/assets/9ba229d3-8f3b-411e-8ff5-2dee4087f13c" />

---

## 5️⃣ Clock Tree Synthesis

### 5.1 CTS Objectives and Clock-Tree Structure

**CTS** builds the distribution network that carries the clock from its source to every sequential element. As the number of clock sinks grows, distribution gets harder because of capacitive loading, long interconnects, insertion delay, arrival-time mismatches, clock skew, and transition requirements — CTS inserts buffers and organizes the network to manage all of this.

Its main goals are to:

1. Reach every required clock sink.
2. Keep skew between sequential elements under control.
3. Keep clock transition times acceptable.
4. Manage insertion delay and network loading.
5. Support setup and hold requirements.
6. Improve overall clock distribution quality.

A clock tree is generally made of a clock source, intermediate buffers, branch points, and sinks; its structure shapes latency, skew, power, and overall timing behavior.

<img width="922" height="483" alt="Screenshot 2026-09-26 185720" src="https://github.com/user-attachments/assets/0c3eb074-b387-4aa5-b8bf-c1a19c80d8f2" />

<img width="920" height="485" alt="Screenshot 2026-09-26 185757" src="https://github.com/user-attachments/assets/3f6264c7-9c1e-49b5-8c32-ec861924dfc7" />

### 5.2 CTS Configuration (cts.tcl)

`cts.tcl` holds the settings that govern the CTS run and how the clock distribution network gets built.

<img width="870" height="427" alt="Screenshot 2026-09-26 185821" src="https://github.com/user-attachments/assets/d2173ec8-19fd-448f-8266-897a0eb066e2" />

---

## 6️⃣ Clock Buffers and Clock Distribution

### 6.1 Clock Buffer Insertion

Clock buffers drive the capacitive load of clock sinks and help distribute the clock across the design while controlling loading and transition times. Their count, sizing, and placement directly affect insertion delay, skew, and power. Inserting buffers splits one large clock load into smaller loads each buffer can drive, which keeps the clock signal intact and lets it reach physically separated flip-flops.

<img width="1168" height="743" alt="Screenshot 2026-09-26 185858" src="https://github.com/user-attachments/assets/cb28094d-94bb-435e-8723-e87243052660" />

### 6.2 Clock Net Shielding

Shielding routes a fixed-potential wire alongside a clock net to cut down unwanted coupling from neighboring signal wires — it improves signal integrity at the cost of extra routing resources.
<img width="922" height="510" alt="Screenshot 2026-09-26 190134" src="https://github.com/user-attachments/assets/5f8fc621-c7c6-4cbd-b1b6-bca1405d64af" />


---

## 7️⃣ Setup and Hold Timing Analysis

Setup and hold define the window during which data at a flip-flop's input must stay stable around the active clock edge.

### 7.1 Setup Time

Setup time is the minimum interval data must be stable **before** the active clock edge. A setup violation happens when data shows up too late. For a simplified register-to-register path:

$$T_{cq} + T_{comb} + T_{setup} \leq T_{period}$$

where $T_{cq}$ is the launching flip-flop's clock-to-Q delay, $T_{comb}$ is the combinational/interconnect delay, $T_{setup}$ is the capturing flip-flop's setup time, and $T_{period}$ is the clock period. The real requirement also folds in clock arrival times and timing uncertainty.

### 7.2 Hold Time

Hold time is the minimum interval data must stay stable **after** the active clock edge. A hold violation happens when data changes too soon. For a simplified path:

$$T_{cq,min} + T_{comb,min} \geq T_{hold}$$

The actual requirement depends on the launch/capture clock arrival times and the design's timing constraints — hold is especially sensitive to minimum data-path delay.

### 7.3 Setup Analysis with an Ideal Clock

An ideal clock ignores clock-distribution delay and skew, which makes it useful for looking at data-path timing on its own before the implemented clock tree is factored in.

<img width="887" height="522" alt="Screenshot 2026-09-26 190216" src="https://github.com/user-attachments/assets/722153d2-4adb-4ef5-b7d0-6fe81ea5ccc0" />
<img width="886" height="335" alt="Screenshot 2026-09-26 190248" src="https://github.com/user-attachments/assets/389f6575-acf4-4689-a05d-da6164262bb1" />

### 7.4 Hold Time Analysis

Hold analysis checks the minimum data-path delay against the hold requirement after the active edge; negative hold slack means the minimum-delay requirement isn't met.


Setup and hold checks together evaluate the timing relationship between launching and capturing elements — a path can pass setup and still fail hold, so both checks are necessary.

<img width="807" height="440" alt="Screenshot 2026-09-26 190341" src="https://github.com/user-attachments/assets/13e2fb2a-972f-48c7-902b-00541b812684" />
<img width="908" height="441" alt="Screenshot 2026-09-26 190410" src="https://github.com/user-attachments/assets/88e71119-11e1-44b9-aa31-a3a26ea9cd69" />

---

## 8️⃣ Clock Skew and Crosstalk

### 8.1 Clock Skew

Clock skew is the difference in clock arrival time between two sequential elements, caused by the clock passing through different buffer/interconnect combinations on its way to each one.

| Skew type | Meaning |
|---|---|
| Positive skew | Capture clock arrives later than the launch clock |
| Negative skew | Capture clock arrives earlier than the launch clock |
| Zero skew | Clock reaches both registers at the same time |

Skew affects setup and hold differently depending on its direction and size.

### 8.2 Crosstalk and Delta Delay

Crosstalk is unwanted electrical coupling between neighboring interconnects — switching on one net can shift the voltage or delay of a nearby net through capacitive or inductive coupling, affecting transition times, propagation delay, timing margin, and signal integrity.

**Crosstalk-induced delta delay** is the change in propagation delay caused by a neighboring net's switching; depending on the relative switching direction and timing, coupling can speed up or slow down a signal.

<img width="917" height="486" alt="Screenshot 2026-09-26 190527" src="https://github.com/user-attachments/assets/cb251e9f-cd1c-4962-9aee-8fc7915c0c49" />

---

## 9️⃣ Timing Reports and Slack Analysis

Static Timing Analysis checks whether the design's timing requirements hold, by comparing signal arrival times against required arrival times — without needing input-vector simulation.

### 9.1 Timing Slack

For setup analysis:

$$\text{Setup Slack} = \text{Required Arrival Time} - \text{Data Arrival Time}$$

| Slack | Meaning |
|---|---|
| Positive slack | Timing requirement satisfied |
| Zero slack | Requirement met right at the boundary |
| Negative slack | Timing violation |

Setup and hold slack each use their own maximum-delay / minimum-delay checks.

### 9.2 Maximum-Delay (Setup) Analysis

Maximum-delay analysis checks whether data beats the setup deadline by evaluating the longest relevant paths; negative slack here flags a setup violation.

<img width="792" height="417" alt="Screenshot 2026-09-26 190625" src="https://github.com/user-attachments/assets/2f68b9cd-8c2e-4506-8b4b-e6ade88dbd98" />

### 9.3 Minimum-Delay (Hold) Analysis

Minimum-delay analysis checks whether data arrives late enough to satisfy hold, by evaluating the shortest relevant paths; negative slack here flags a hold violation.

<img width="575" height="375" alt="Screenshot 2026-09-26 190650" src="https://github.com/user-attachments/assets/2d3049f8-4bd6-4c0b-9a41-868ae3f6dc69" />

### 9.4 Delay Tables

Timing reports and delay tables list cell delay, interconnect delay, arrival/required times, and slack — helpful for spotting critical paths and understanding how much cells vs. interconnect contribute to overall timing.

<img width="910" height="547" alt="Screenshot 2026-09-26 190816" src="https://github.com/user-attachments/assets/6ec18e70-1682-4e93-a1eb-4fedb35a30e9" />

---

## 🔟 Physical Design Configuration and Routing Information

### 10.1 Converting Labels to Ports

Converting labels to ports is a preparation step tied to the design's connectivity and interface representation ahead of physical implementation.

<img width="897" height="526" alt="Screenshot 2026-09-26 190854" src="https://github.com/user-attachments/assets/4ef14ab4-915b-4d48-8de4-5afe3e7a8293" />

### 10.2 Grid-to-Track Conversion and `tracks.info`

Routing-track information marks where wires can be placed on each routing layer; grid information is converted into track information to set this up.

<img width="871" height="520" alt="Screenshot 2026-09-26 190919" src="https://github.com/user-attachments/assets/088b9296-158b-45d9-906b-90b1f08778ab" />

`tracks.info` holds the routing-track data used through the rest of the physical implementation flow.
<img width="143" height="193" alt="Screenshot 2026-09-26 191021" src="https://github.com/user-attachments/assets/dfd8bcda-48a0-45be-8005-3344a0925979" />


---

## ✅ Takeaways

- ✅ Placement directly affects timing — cell locations shape interconnect length and propagation delay.
- ✅ Clock buffers are what make clock distribution practical, spreading a single large clock load across manageable branches.
- ✅ CTS-introduced insertion delay and skew feed straight into setup and hold timing behavior.
- ✅ Setup and hold checks are complementary — a path can clear one and still fail the other, so both maximum- and minimum-delay analysis are needed.
- ✅ Crosstalk-induced delta delay can shift propagation delay and skew, and shielding is one way to control it.
- ✅ Liberty, LEF, and SDC files are the backbone that timing analysis and physical implementation both depend on.
- ✅ Slack values and path reports are the practical tools for finding and fixing timing violations during optimization.

## 👤 Author

**Arpitha**
B.Tech, Electronics and Communication Engineering
Anurag University, Hyderabad

---
