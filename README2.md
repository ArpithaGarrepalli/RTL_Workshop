# 🔧 Chip Design Program — Physical Design (PD)

<p>
  <img src="https://img.shields.io/badge/Tool-OpenLANE-blue" alt="OpenLANE">
  <img src="https://img.shields.io/badge/Tool-OpenROAD-purple" alt="OpenROAD">
  <img src="https://img.shields.io/badge/Tool-Magic-orange" alt="Magic">
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

➡️ **Documentation:** [Module-1 README](./Module-1/README.md)

---

### 🟩 Module-2 — Floorplanning and Library Cells

**Topics Covered:**

- What floorplanning is and why it's one of the most consequential steps in physical design
- Good floorplanning practices vs. bad floorplanning, and the downstream problems bad floorplanning causes
- Utilization factor and aspect ratio
- Introduction to library cells and standard-cell characterization
- How library cells connect back to floorplanning and placement decisions

➡️ **Documentation:** [Module-2 README](./Module-2/README.md)

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **OpenLANE** | Automated RTL-to-GDS ASIC flow |
| **OpenROAD** | Floorplanning, placement, CTS, routing |
| **Magic** | Layout viewing and DRC |
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
```

---

## 👤 Author

**Name:** Arpitha Garrepalli
**Department:** Electronics and Communication Engineering (ECE)
**College:** Anurag University
