# 🔧 Chip Design Program — Physical Design (PD)

<p>
  <img src="https://img.shields.io/badge/Tool-OpenLANE-blue" alt="OpenLANE">
  <img src="https://img.shields.io/badge/Tool-OpenROAD-purple" alt="OpenROAD">
  <img src="https://img.shields.io/badge/Tool-Magic-orange" alt="Magic">
  <img src="https://img.shields.io/badge/Tool-ngspice-teal" alt="ngspice">
  <img src="https://img.shields.io/badge/STA-OpenSTA-darkred" alt="OpenSTA">
  <img src="https://img.shields.io/badge/Routing-TritonRoute-blueviolet" alt="TritonRoute">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
  <img src="https://img.shields.io/badge/Flow-RTL--to--GDS-green" alt="RTL to GDS">
</p>

This repository documents my learning journey and hands-on experiments completed during the Physical Design track of the Chip Design Program. It contains module-wise documentation, practical exercises, floorplan/placement/routing results, and screenshots from the labs.

---

## 📂 Repository Contents

### 🟦 Module-1 — Inception of Open-Source EDA, OpenLANE, and SKY130 PDK

**Topics Covered:**

- Introduction to open-source EDA tools and why they matter for chip design
- The RTL-to-GDS flow at a high level
- Introduction to OpenLANE as an automated ASIC implementation flow
- Introduction to the SKY130 PDK (Process Design Kit)
- Setting up and running a basic OpenLANE flow

➡️ **Documentation:** [Module-1 README](./Module-1-pd/README.md)

---

### 🟩 Module-2 — Floorplanning and Library Cells

**Topics Covered:**

- What floorplanning is and why it's one of the most consequential steps in physical design
- Good floorplanning practices vs. bad floorplanning, and the downstream problems bad floorplanning causes
- Utilization factor and aspect ratio
- Introduction to library cells and standard-cell characterization
- How library cells connect back to floorplanning and placement decisions

➡️ **Documentation:** [Module-2 README](./Module-2-pd/README.md)

---

### 🟨 Module-3 — CMOS Inverter Design, Fabrication, Layout & Characterization

**Topics Covered:**

- Transistor-level CMOS inverter design and ngspice simulation
- Voltage Transfer Characteristic (VTC) generation and switching threshold (Vm)
- Effect of transistor sizing (W/L ratios) on inverter performance and timing
- The 16-mask CMOS fabrication process — active region (LOCOS), well formation, gate formation, LDD, source/drain, contacts, and metal interconnect
- Standard-cell layout design in Magic using SKY130A technology files, including cell boundary and power/ground connectivity
- Design Rule Checking (DRC), including debugging the `poly.9` error
- SPICE extraction from the layout and post-layout characterization using ngspice

➡️ **Documentation:** [Module-3 README](./Module-3-pd/README.md)

---

### 🟧 Module-4 — Pre-Layout Timing Analysis, Clock Tree Synthesis & Physical Design

**Topics Covered:**

- Standard-cell timing modeling — delay tables, Liberty libraries, and LEF generation
- Static Timing Analysis (STA) with ideal clocks using OpenSTA
- Setup time, clock jitter, clock uncertainty, synthesis optimization, and timing ECO
- Clock Tree Synthesis (CTS) using TritonCTS — H-Tree clock distribution, buffering, crosstalk and clock shielding
- Setup and hold timing analysis with real (propagated) clocks, and the impact of CTS buffer sizing
- Full OpenLane/OpenROAD physical-design run — synthesis, placement, and CTS — on the PicoRV32A RISC-V core

➡️ **Documentation:** [Module-4 README](./Module-4-pd/README.md)

---

### 🟥 Module-5 — Routing, DRC, Power Distribution & Parasitic Extraction

**Topics Covered:**

- Global and detailed routing fundamentals, including maze routing using Lee's Algorithm
- Power Distribution Network (PDN) construction — rings, straps, rails, and macro power connections
- Design Rule Checking (DRC) — wire-width and via-spacing verification, DRC-clean layouts
- TritonRoute detailed routing engine — preprocessed route guides, intra-layer/inter-layer routing, and connectivity handling (Access Points and Clusters)
- Routing topology optimization and OpenLane routing/run-directory outputs
- Parasitic extraction, SPEF generation, and post-route timing verification using OpenSTA

➡️ **Documentation:** [Module-5 README](./Module-5-pd/README.md)

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **OpenLANE** | Automated RTL-to-GDS ASIC flow |
| **OpenROAD** | Floorplanning, placement, CTS, routing |
| **Magic** | Layout viewing and DRC |
| **ngspice** | CMOS circuit simulation and standard-cell characterization |
| **OpenSTA** | Static Timing Analysis (ideal and real clocks) |
| **TritonCTS** | Clock Tree Synthesis |
| **TritonRoute** | Design-rule-aware detailed routing |
| **SKY130 PDK** | Process design kit / standard-cell library |
| **Git / GitHub** | Version control & documentation |

---

## 🗂️ Repository Structure

```
Chip_Design_Program_PD/
│── README2.md
│
│── Module-1-pd/
│   └── README.md
│
│── Module-2-pd/
│   └── README.md
│
│── Module-3-pd/
│   └── README.md
│
│── Module-4-pd/
│   └── README.md
│
│── Module-5-pd/
│   └── README.md
```

---

## 👤 Author

**Name:** Arpitha Garrepalli
**Department:** Electronics and Communication Engineering (ECE)
**College:** Anurag University
