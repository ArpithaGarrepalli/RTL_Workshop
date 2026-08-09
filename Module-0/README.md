# 🔧 Module 0 — Workshop Introduction

<p>
  <img src="https://img.shields.io/badge/Tool-Icarus%20Verilog-blue" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Tool-GTKWave-orange" alt="GTKWave">
  <img src="https://img.shields.io/badge/Tool-Yosys-green" alt="Yosys">
  <img src="https://img.shields.io/badge/Language-Verilog-9cf" alt="Verilog">
</p>

> Part of the [RTL Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop) series.

## 📖 Overview

This document covers Module 0 of the RTL Design Workshop, focused on getting the environment ready before any RTL work begins — an introduction to how the workshop is run, the option of using a cloud-based lab, and the steps for installing the toolchain locally.

| | |
|---|---|
| 🛠️ **Tools installed** | Icarus Verilog, GTKWave, Yosys |
| 💻 **Environments covered** | Cloud lab, local Ubuntu/Linux setup |
| 📋 **Prerequisites** | A Linux-based system or VM (Ubuntu recommended) |

## 📑 Table of Contents

- 1. Introduction and Cloud Lab Instructions
  - 1.1 Workshop Structure
  - 1.2 Using the Cloud Lab
- 2. Local Lab Installation
  - 2.1 System Requirements
  - 2.2 Installing Icarus Verilog and GTKWave
  - 2.3 Installing Yosys
  - 2.4 Verifying the Installation
- 3. Takeaways
- Author

---

## 1️⃣ Introduction and Cloud Lab Instructions

### 1.1 Workshop Structure

The workshop is organized module by module, each covering a specific stage of the RTL-to-synthesis flow,starting with simulation fundamentals and progressing through synthesis, timing libraries, and flip-flop coding styles. Each module pairs short conceptual explanations with hands-on labs, so tool setup only needs to be done once, here in Module 0.

### 1.2 Using the Cloud Lab

For anyone who doesn't want to set up a local Linux environment, the workshop provides a browser-based cloud lab with all required tools pre-installed. This is the fastest way to get started, since it skips the installation steps in Section 2 entirely.

To use it:

1. Open the cloud lab platform provided by the workshop.
2. Sign in with the enrollment credentials issued for the course.
3. Launch the pre-configured virtual machine from the dashboard.

<img width="947" height="985" alt="Screenshot 2026-08-09 174304" src="https://github.com/user-attachments/assets/0206817f-01ee-4ac7-a832-d6db85a5c70b" />


---

## 2️⃣ Local Lab Installation

### 2.1 System Requirements

Local installation assumes an Ubuntu (or other Debian-based) Linux system, either natively or inside a virtual machine such as Oracle VM VirtualBox. Update the package index before installing anything:

```bash
sudo apt update
sudo apt upgrade -y
```

### 2.2 Installing Icarus Verilog and GTKWave

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

### 2.3 Installing Yosys

```bash
sudo apt install yosys
```

If a newer version is needed, Yosys can also be built from source:

```bash
git clone https://github.com/YosysHQ/yosys.git
cd yosys
make config-gcc
make
sudo make install
```

### 2.4 Verifying the Installation

Confirm each tool installed correctly and check its version:

```bash
iverilog -V
gtkwave --version
yosys -V
```

<img width="674" height="485" alt="1-vlsi" src="https://github.com/user-attachments/assets/9a6a27f4-8f92-4560-b98f-4f2d0ac9992b" />


---

## 3️⃣ Takeaways

- ✅ Understood how the workshop modules are structured.
- ✅ Learned the option of using a pre-configured cloud lab.
- ✅ Installed Icarus Verilog, GTKWave, and Yosys locally.
- ✅ Verified the toolchain is ready before starting Module 1.

---

## 👤 Author

**Arpitha Garrepalli**
[github.com/ArpithaGarrepalli/RTL_Workshop](https://github.com/ArpithaGarrepalli/RTL_Workshop/Module-0/README.md)
