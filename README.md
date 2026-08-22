# 🔧 RTL Design And Synthesis Workshop
<p>
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/PDK-SKY130-red" alt="SKY130">
</p>

This repository documents my learning journey and hands-on experiments completed during the RTL Design Workshop. It contains module-wise documentation, practical exercises, simulation results, waveform analysis, and Verilog RTL design implementations.

---

## 📂 Repository Contents

### 🟦 Module-0 — Workshop Introduction
**Topics Covered:**
- Introduction and Cloud Lab Instructions
- Local Lab Installation

➡️ **Documentation:** [Module-0 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-0/README.md)

---

### 🟩 Module-1 — Verilog RTL Design Through Simulation & Yosys Synthesis
**Topics Covered:**
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

➡️ **Documentation:** [Module-1 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-1/README.md)

---

### 🟨 Module-2 — RTL Design and Synthesis
**Topics Covered:**
- Module Overview
- Timing Libraries & Technology Files
- Hierarchical & Flattened Synthesis
- Flip-Flop Coding Styles
- RTL Simulation and Synthesis Flow
- Interesting Optimization
- Practical Exercise, Results & Conclusion

➡️ **Documentation:** [Module-2 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-2/README.md)

---

### 🟥 Module-3 — Combinational and Sequential Optimization
**Topics Covered:**
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

➡️ **Documentation:** [Module-3 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-3/README.md)

---

### 🟪 Module-4 — Gate-Level Simulation, Blocking vs. Non-Blocking, and Synthesis-Simulation Mismatch
**Topics Covered:**
- 1. Building and Verifying a Correct Multiplexer
  - 1.1 Simulating the Ternary MUX
  - 1.2 Mapping the MUX to Silicon
  - 1.3 Confirming Behavior Across Inputs
- 2. Diagnosing a Broken Multiplexer
  - 2.1 First Signs of Trouble
  - 2.2 Tracing the Latch
- 3. Understanding Blocking Assignment Execution
  - 3.1 Watching Execution Order in Simulation
  - 3.2 Confirming the Synthesized Result
  - 3.3 Same-Block Value Dependency
- 4. Results at a Glance
- 5. Takeaways

➡️ **Documentation:** [Module-4 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-4/README.md)

---

### 🟫 Module-5 — Incomplete Conditional Constructs and Combinational Design
**Topics Covered:**
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

➡️ **Documentation:** [Module-5 README](https://github.com/ArpithaGarrepalli/RTL_Workshop/blob/main/Module-5/README.md)

---

## 🛠️ Tools Used
| Tool | Purpose |
|---|---|
| **Verilog** | Hardware description language |
| **Icarus Verilog (iverilog)** | Simulation |
| **GTKWave** | Waveform viewing |
| **Yosys** | Logic synthesis |
| **SKY130 Standard-Cell Library** | Technology mapping |
| **Git / GitHub** | Version control & documentation |

---

## 🗂️ Repository Structure
```
RTL_Workshop/
│── README.md
│
│── Module-0/
│   └── README.md
│
│── Module-1/
│   └── README.md
│
│── Module-2/
│   └── README.md
│
│── Module-3/
│   └── README.md
│
│── Module-4/
│   └── README.md
│
│── Module-5/
│   └── README.md
```

---

## 👤 Author
**Name:** Arpitha Garrepalli
**College:** Anurag University
**Branch:** Electronics and Communication Engineering (ECE)
