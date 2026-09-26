# 🧵 Module 5 — Routing, DRC, Power Distribution & Parasitic Extraction

<p>
  <img src="https://img.shields.io/badge/Flow-OpenLane%20%2F%20OpenROAD-green" alt="OpenLane / OpenROAD">
  <img src="https://img.shields.io/badge/Routing-TritonRoute-blueviolet" alt="TritonRoute">
  <img src="https://img.shields.io/badge/STA-OpenSTA-orange" alt="OpenSTA">
  <img src="https://img.shields.io/badge/PDK-SKY130-blue" alt="SKY130">
  <img src="https://img.shields.io/badge/Verification-DRC-yellow" alt="DRC">
  <img src="https://img.shields.io/badge/Format-LEF%20%2F%20DEF%20%2F%20SPEF-red" alt="LEF / DEF / SPEF">
  <img src="https://img.shields.io/badge/Layout-Magic-lightgrey" alt="Magic">
</p>

> Part of the SKY130 VLSI Physical Design module series.

## 📖 Overview

This document covers Module 5 of the SKY130 physical-design flow: global and detailed routing, maze routing via Lee's Algorithm, Power Distribution Network (PDN) construction, Design Rule Checking (DRC), TritonRoute's detailed-routing engine, routing connectivity and topology, parasitic extraction/SPEF generation, and the resulting OpenLane output used for post-route timing verification.

|  |  |
|---|---|
| 🛠️ **Tools used** | OpenLane, OpenROAD, TritonRoute, OpenSTA, Yosys, SKY130 PDK, Magic, LEF, DEF, SPEF, Tcl, Docker |
| 🧩 **Example design(s)** | OpenLane-driven SKY130 physical design run (routing, PDN and DRC labs) |
| 📋 **Prerequisites** | OpenLane/OpenROAD environment running in Docker, SKY130 PDK installed, a placed and clock-tree-synthesized design, LEF/DEF files available |

## 📑 Table of Contents

1. Routing Fundamentals — Global, Fast and Detailed Routing
   1.1 Routing Overview and Objectives
   1.2 Global Routing vs Detailed Routing
   1.3 Fast Route and Route Guide Generation
2. Maze Routing — Lee's Algorithm
   2.1 Algorithm Steps
   2.2 Advantages and Limitations
3. Power Distribution Network (PDN)
   3.1 Purpose and Components of the PDN
   3.2 PDN Construction Flow and Configuration
   3.3 Power Straps and Standard-Cell / Macro Power Connections
4. Design Rule Checking (DRC)
   4.1 Common Design Rules
   4.2 DRC — Wire Width
   4.3 DRC — Via Spacing
   4.4 DRC-Clean Layout
5. TritonRoute Detailed Routing Engine
   5.1 Role of TritonRoute and Routing Configuration
   5.2 Preprocessed Route Guides
   5.3 Intra-Layer and Inter-Layer Routing
6. Routing Connectivity and Topology
   6.1 Access Points and Access Point Clusters
   6.2 Routing Obstacles and Optimization
   6.3 Routing Topology Algorithm
7. Parasitic Extraction and SPEF Generation
8. OpenLane Physical Design Execution and Results
   8.1 Floorplanning and Power Planning
   8.2 OpenLane Configuration Parameters
   8.3 Routing Output and Run Directories
9. Post-Route Timing Verification and Final Checklist
10. Overall Physical Design Flow (RTL to GDSII)
11. Tools and Technologies Used
- Author

---

## 1️⃣ Routing Fundamentals — Global, Fast and Detailed Routing

### 1.1 Routing Overview and Objectives

Routing connects placed standard cells, pins, and other components through the metal layers while satisfying design rules. It proceeds through global routing, fast routing, detailed routing, design-rule-aware routing, and connectivity verification.

<img width="1666" height="790" alt="Screenshot 2026-09-26 192628" src="https://github.com/user-attachments/assets/630ae945-356e-44a6-9bda-23ee3480dc1c" />

### 1.2 Global Routing vs Detailed Routing

**Global routing** divides the routing region into a resource grid and finds approximate paths through it, producing route guides for the detailed-routing stage. Its goals: estimate routing paths, manage congestion, allocate routing resources, spot bottlenecks, and generate guides.

**Detailed routing** turns those approximate paths into actual wire segments and vias, working at the level of exact wire locations, layer selection, via placement, design rules, connectivity, and local congestion.

| Feature | Global Routing | Detailed Routing |
|---|---|---|
| Purpose | Approximate routing paths | Actual physical connections |
| Representation | Routing resource grid | Physical wires and vias |
| Output | Routing guides | Detailed routed geometry |
| Main concern | Congestion and resource allocation | Connectivity and design-rule compliance |
| Level of detail | Coarse | Fine-grained |

### 1.3 Fast Route and Route Guide Generation

| Stage | Description |
|---|---|
| **Fast Route** | Produces an initial routing solution and generates route guides |
| **Detailed Route** | Converts that initial routing information into physical routing that satisfies detailed design rules |

<img width="1548" height="828" alt="Screenshot 2026-09-26 192715" src="https://github.com/user-attachments/assets/3c6cf241-2146-4ce6-933b-1b1607a1f793" />

---

## 2️⃣ Maze Routing — Lee's Algorithm

Maze routing is a grid-based pathfinding technique for connecting two points while avoiding obstacles. **Lee's Algorithm** does this through wavefront expansion: it explores neighboring grid cells in successive steps, assigning distance values, until it reaches the destination, then backtracks to reconstruct the shortest path.

### 2.1 Algorithm Steps

1. Mark the source point as the starting location.
2. Expand the wavefront — explore neighboring grid cells and assign distance values.
3. Exclude blocked or unavailable locations (obstacle avoidance).
4. Continue expanding until the destination is reached.
5. Backtrack along the minimum-distance path.
6. Record the final reconstructed route.

<img width="1653" height="822" alt="Screenshot 2026-09-26 192754" src="https://github.com/user-attachments/assets/5bcb9ea8-8663-4d5c-8293-59b965e2ceb2" />

### 2.2 Advantages and Limitations

**Advantages:** systematic path exploration; guarantees the shortest valid path in an unweighted grid when one exists; naturally handles obstacles by excluding blocked cells; a simple foundation for understanding maze routing.

**Limitations:** memory-intensive for large grids; can explore many unnecessary locations; doesn't inherently account for congestion, wire delay, or layer preference — real VLSI routing needs those factored in on top.

---

## 3️⃣ Power Distribution Network (PDN)

### 3.1 Purpose and Components of the PDN

A PDN distributes power and ground throughout the design, supplying standard cells and macros while keeping voltage levels acceptable. It aims to:

- Distribute power and ground across the chip
- Connect standard cells and macros electrically
- Reduce voltage drop along power paths
- Support current delivery to every region of the design
- Structure the connection between the power source and circuit components

Typical components: power/ground pins, power rings, power straps, standard-cell power rails, macro power connections, and vias linking power conductors across metal layers.

### 3.2 PDN Construction Flow and Configuration

```text
Power and Ground Definition
          |
          v
Power Ring Generation
          |
          v
Power Strap Generation
          |
          v
Power Rail Connection
          |
          v
Standard-Cell and Macro Connections
          |
          v
Power Connectivity Verification
```

In OpenROAD-based flows, PDN generation is scripted in Tcl — e.g. a `pdngen` invocation triggers the configured power-network generation process, with the actual power nets, layers, and geometry defined for the specific design and technology.

<img width="1262" height="847" alt="Screenshot 2026-09-26 193018" src="https://github.com/user-attachments/assets/4ea6ea50-6fcc-41bb-bca7-e125f4270eee" />

### 3.3 Power Straps and Standard-Cell / Macro Power Connections

**Power straps** are wider metal conductors that distribute power/ground across the core, connecting into the PDN and carrying current to cells and macros via designated layers and vias.

**Standard cells** draw power/ground through their power pins and the rails running along their placement rows, which the PDN connects into the broader power network.

**Macros** (e.g. RAM blocks) may need dedicated power pins; the PDN must connect these to the chip-level power network.

Key PDN-design factors: metal-layer selection, strap width/spacing, via connectivity, current demand, voltage drop, macro placement/pin locations, and rail-to-network connectivity.

---

## 4️⃣ Design Rule Checking (DRC)

DRC verifies that a physical layout satisfies the fabrication rules of the chosen technology — geometry restrictions that keep the design manufacturable.

### 4.1 Common Design Rules

- Minimum wire width
- Minimum spacing between wires and between vias
- Minimum enclosure and area requirements
- Layer-specific restrictions
- Connectivity constraints, and restrictions on overlapping/improperly connected shapes

A DRC violation surfaces when a route breaks a spacing, width, enclosure, or other physical constraint — routing and verification stages exist to find and resolve these.

### 4.2 DRC — Wire Width

Wires thinner than the minimum allowed width can violate manufacturing rules and hurt interconnect reliability.

<img width="865" height="406" alt="Screenshot 2026-09-26 193307" src="https://github.com/user-attachments/assets/221df795-50eb-4b06-aa01-7efde6b247eb" />

### 4.3 DRC — Via Spacing

Correct spacing between vias and neighboring structures prevents manufacturing violations and unintended shorts.

<img width="847" height="422" alt="Screenshot 2026-09-26 193325" src="https://github.com/user-attachments/assets/768aeb48-1c2b-4366-a976-0e66f08f7e76" />

### 4.4 DRC-Clean Layout

A **DRC-clean** result means the layout satisfies every rule checked under the verification conditions used — though that alone doesn't confirm timing, electrical connectivity, or antenna requirements are also met.

<img width="1243" height="587" alt="Screenshot 2026-09-26 193340" src="https://github.com/user-attachments/assets/3b3f5884-415b-4b44-8c65-abd10da8ad32" />

---

## 5️⃣ TritonRoute Detailed Routing Engine

### 5.1 Role of TritonRoute and Routing Configuration

TritonRoute is the detailed-routing engine in the OpenROAD flow. It turns route guides into actual physical connections while respecting routing constraints and design rules — processing guides, connecting pins, routing across metal layers, handling obstacles, resolving conflicts, and enforcing design-rule compliance.

```text
Global Routing → Routing Guides → TritonRoute → Detailed Wire/Via Generation → Routing Verification → Post-Route Design
```

An illustrative OpenROAD command for this stage is `detailed_route`, invoked with the design and routing configuration already loaded; exact options depend on the installed OpenROAD version and flow configuration.

<img width="1471" height="543" alt="Screenshot 2026-09-26 193559" src="https://github.com/user-attachments/assets/88e06beb-0b6b-4b02-92e2-bab8309a78b1" />
<img width="1600" height="953" alt="WhatsApp Image 2026-09-25 at 7 53 47 PM (1)" src="https://github.com/user-attachments/assets/a3eaaa84-156d-4c64-bec6-7c5f3ce6116c" />


### 5.2 Preprocessed Route Guides

Route guides mark the preferred regions/directions for routing a net, generated during global routing to steer detailed routing toward the paths chosen earlier. Requirements: guides should have unit width and follow the preferred routing direction. Guides help by providing direction, making use of allocated routing resources, coordinating global and detailed routing, and cutting down unnecessary exploration.

<img width="1296" height="837" alt="Screenshot 2026-09-26 193617" src="https://github.com/user-attachments/assets/5c6913f1-a94e-43e4-9787-e8bd15956e91" />
<img width="1600" height="948" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (3)" src="https://github.com/user-attachments/assets/dc1e8be6-618b-466f-b258-f9fbcf776703" />
<img width="1600" height="951" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (4)" src="https://github.com/user-attachments/assets/5d1ade17-624e-4332-8f14-6bd8731ac023" />


### 5.3 Intra-Layer and Inter-Layer Routing

| Strategy | Description |
|---|---|
| **Intra-Layer (Parallel)** | Connections handled within the same metal layer; multiple routing tasks processed in parallel |
| **Inter-Layer (Sequential)** | Routing proceeds sequentially across layers, using vias to connect conductors, for full connectivity |

<img width="1342" height="711" alt="Screenshot 2026-09-26 193628" src="https://github.com/user-attachments/assets/d3a2fc47-b364-40ec-9596-4332b3eb87d0" />

A typical multi-layer connection looks like:

```text
Source Pin → Metal Layer 1 → Via → Metal Layer 2 → Via → Destination Pin
```

---

## 6️⃣ Routing Connectivity and Topology

### 6.1 Access Points and Access Point Clusters

- **Access Point (AP):** an on-grid point on a route guide's metal layer, used to connect lower-layer segments, upper-layer segments, pins, or I/O ports.
- **Access Point Cluster (APC):** the union of access points that all derive from the same lower-layer segment, upper-layer guide, pin, or I/O port.

<img width="1466" height="770" alt="Screenshot 2026-09-26 193707" src="https://github.com/user-attachments/assets/b6ca48bf-67f9-4a84-9bf3-2c5e6a0888cc" />

### 6.2 Routing Obstacles and Optimization

**Obstacles** restrict where wires/vias can go — macro blockages, existing routed wires, restricted regions, spacing constraints, and pin-access limits all count. The detailed router must route around these while keeping every net's terminals fully connected — an open connection or incomplete route is a connectivity failure.

**Routing optimization** then improves the physical connections without breaking connectivity, weighing wirelength, congestion, design-rule compliance, via count, signal delay, and routing-resource use.
<img width="1600" height="951" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (7)" src="https://github.com/user-attachments/assets/56df0f41-ff0e-4e71-b900-99b97423cff1" />
<img width="1600" height="950" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (6)" src="https://github.com/user-attachments/assets/35ca128c-19d5-4f28-bffd-0e34ad27be24" />


### 6.3 Routing Topology Algorithm

The routing-topology algorithm decides how a multi-terminal net's connection points link together efficiently — minimizing routing cost while keeping every terminal connected. The physical router then implements that topology as actual wire segments and vias, respecting all routing constraints; topology choice affects wirelength, resource use, and parasitic characteristics.

<img width="836" height="415" alt="Screenshot 2026-09-26 193717" src="https://github.com/user-attachments/assets/28a669c5-7aaa-431c-85b3-a31d0020e67a" />


---

## 7️⃣ Parasitic Extraction and SPEF Generation

After routing, the resistance, capacitance, interconnect delay, and coupling effects introduced by the physical wires must be extracted for accurate post-layout timing analysis.
<img width="1790" height="806" alt="Screenshot 2026-09-26 193730" src="https://github.com/user-attachments/assets/9f4e1314-915a-4091-9054-4a13111b6d9c" />

**SPEF** (Standard Parasitic Exchange Format) carries this extracted parasitic information — resistance, capacitance, nets, interconnect parasitics, and connectivity — generated by an extraction script that processes the routed DEF.
<img width="951" height="595" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (2)" src="https://github.com/user-attachments/assets/55e26362-e2cc-45a8-82f3-882e027d695f" />
<img width="1600" height="952" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (5)" src="https://github.com/user-attachments/assets/5c0d0e40-24c0-4c7f-a990-be4a7c9bae14" />

---

## 8️⃣ OpenLane Physical Design Execution and Results

### 8.1 Floorplanning and Power Planning

Key floorplanning elements: core/die area, standard-cell rows, I/O placement, power rings and straps, and macro placement — all set up to ensure reliable VPWR/VGND distribution.

<img width="1247" height="720" alt="Screenshot 2026-09-26 193755" src="https://github.com/user-attachments/assets/421a8ef1-0340-460c-b2b3-47a15ca4600d" />

### 8.2 OpenLane Configuration Parameters

Configuration parameters span placement, CTS, routing, Magic, density, timing, and routing optimization — controlling cell density, clock-tree generation, routing layers/optimization, and layout generation.

<img width="1900" height="840" alt="Screenshot 2026-09-26 193811" src="https://github.com/user-attachments/assets/73e6d019-9a93-4fca-b285-e9462edb16e9" />

### 8.3 Routing Output and Run Directories
<img width="1600" height="953" alt="WhatsApp Image 2026-09-25 at 7 53 46 PM (1)" src="https://github.com/user-attachments/assets/2472fec6-b465-4fbe-b881-0b2e4fa5e62a" />

Synthesis and routing results are organized into stage-specific run directories:

```text
results/
├── synthesis/
├── routing/
├── placement/
├── cts/
├── floorplan/
└── signoff/
```

---

## 9️⃣ Post-Route Timing Verification and Final Checklist
<img width="1600" height="956" alt="WhatsApp Image 2026-09-25 at 7 53 47 PM (2)" src="https://github.com/user-attachments/assets/c746c647-edfd-475e-9354-6f06d62653c4" />


**OpenSTA** evaluates post-route timing using the routed netlist, SDC constraints, and extracted parasitics, computing arrival time, required time, and setup/hold slack:

```text
Routed Netlist
      |
      v
Timing Constraints (SDC)
      |
      v
Parasitic Information
      |
      v
OpenSTA
      |
      v
Arrival Time and Required Time
      |
      v
Setup and Hold Slack
      |
      v
Timing Verification
```

Final implementation checklist:

- [ ] Power distribution network generated
- [ ] Global routing completed
- [ ] Detailed routing completed
- [ ] Routing connectivity checked
- [ ] Design rule checking performed
- [ ] Parasitic information generated
- [ ] Post-route timing analysis performed
- [ ] Final physical design files generated

---

## 🔟 Overall Physical Design Flow (RTL to GDSII)

```text
RTL Design
 ↓
Synthesis
 ↓
Floorplanning
 ↓
Power Distribution Network
 ↓
Standard-Cell Placement
 ↓
Clock Tree Synthesis
 ↓
Global Routing
 ↓
Fast Route / Route Guide Generation
 ↓
Route Guide Preprocessing
 ↓
Detailed Routing (TritonRoute)
 ↓
Design Rule Checking
 ↓
Parasitic Extraction / SPEF Generation
 ↓
Post-Route Timing Analysis (OpenSTA)
 ↓
Physical Verification
 ↓
GDSII
```

---

## 1️⃣1️⃣ Tools and Technologies Used

| Tool / Technology | Purpose |
|---|---|
| OpenLane | Automated RTL-to-GDSII physical design flow |
| OpenROAD | Physical implementation and optimization |
| TritonRoute | Design-rule-aware detailed routing engine |
| OpenSTA | Static Timing Analysis |
| Yosys | RTL synthesis |
| SKY130 | Open-source 130 nm process technology |
| Magic | Layout viewing and physical verification |
| LEF | Physical cell and technology information |
| DEF | Physical design placement/routing representation |
| SPEF | Standard Parasitic Exchange Format |
| Tcl | Flow configuration and automation |
| Docker | Execution environment |

---

## ✅ Takeaways

- ✅ Understood how global routing (resource grid, route guides) differs from detailed routing (actual wires and vias).
- ✅ Traced Lee's Algorithm's wavefront-expansion approach to maze routing, along with its memory/congestion trade-offs.
- ✅ Built and configured a Power Distribution Network — rings, straps, rails, and macro power connections — via `pdngen`.
- ✅ Applied DRC checks for wire width, via spacing, and overall layout cleanliness.
- ✅ Learned how TritonRoute consumes LEF/DEF/route guides and resolves intra-layer and inter-layer routing to produce a wirelength/via-optimized solution.
- ✅ Understood connectivity handling through Access Points and Access Point Clusters, and how routing topology balances cost against full connectivity.
- ✅ Extracted post-route parasitics into SPEF and used OpenSTA for post-route setup/hold timing verification.
- ✅ Connected the full flow: **PDN → Global Routing → Detailed Routing (TritonRoute) → DRC → Parasitic Extraction → Post-Route STA → GDSII**.

## 👤 Author

**Arpitha**
B.Tech, Electronics and Communication Engineering
Anurag University, Hyderabad

---
