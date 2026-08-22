# Transistor-Level 6T SRAM Bitcell (Magic VLSI & Sky130)

![EDA](https://img.shields.io/badge/EDA-Magic%20VLSI-blue)
![PDK](https://img.shields.io/badge/PDK-SkyWater%20130nm-green)
![Simulation](https://img.shields.io/badge/Simulation-Ngspice%20%7C%20SPICE-orange)
![Design](https://img.shields.io/badge/Design-Custom%20IC%20Layout-purple)

A full-custom transistor-level silicon layout of a 6-Transistor Static Random-Access Memory (6T SRAM) bitcell designed in Magic VLSI using the open-source SkyWater 130nm CMOS process (`sky130A`).

---

## 🔬 Bitcell Circuit Topology

The 6T SRAM bitcell consists of two cross-coupled CMOS inverters forming a bi-stable latch (`M1`–`M4`) and two NMOS access pass-transistors (`M5`, `M6`) gated by the Wordline (`WL`):

```
                 VDD                     VDD
                  │                       │
                ┌─┴─┐                   ┌─┴─┐
                │M2 │                   │M4 │ (PMOS Pull-Up)
                └─┬─┘                   └─┬─┘
       BL         │        Q     QB       │         BLB
       │        ┌─┴─────────┐   ┌─────────┴─┐        │
  WL ──┼───────┤  M5 (AX)   │   │  M6 (AX)  ├───┼── WL
       │        └─┬─────────┘   └─────────┬─┘        │
                ┌─┴─┐                   ┌─┴─┐
                │M1 │                   │M3 │ (NMOS Pull-Down)
                └─┬─┘                   └─┬─┘
                  │                       │
                 GND                     GND
```

---

## 📐 Transistor Sizing & Stability Ratios

To ensure non-destructive read operations and reliable writeability, transistor sizing follows strict ratio constraints:

1. **Read Stability (Cell Ratio / $\beta$-Ratio)**:
   $$\text{Cell Ratio} = \frac{(W/L)_{M1,M3}}{(W/L)_{M5,M6}} > 1.2\text{--}1.5$$
   *Ensures the internal node voltage ($V_Q$) does not rise above the trip point of the opposing inverter during a read access.*
2. **Writeability (Pull-Up Ratio / $\gamma$-Ratio)**:
   $$\text{Pull-Up Ratio} = \frac{(W/L)_{M2,M4}}{(W/L)_{M5,M6}} < 1.0$$
   *Ensures the access pass-transistor can successfully pull down the internal node against the PMOS pull-up transistor during write operations.*

---

## 🏗️ Physical Layout Specifications (Magic VLSI)

- **Process Node**: SkyWater 130nm (`sky130A`) Open-Source PDK.
- **Layers Used**: Diffusion (`ndiff`, `pdiff`), Polysilicon (`poly`), Metal-1 (`m1` for local interconnects), Metal-2 (`m2` for power rails & Bitlines).
- **Design Rule Checking (DRC)**: 100% clean under Sky130 scalable CMOS rules.
- **Parasitic Extraction**: Netlist extracted to SPICE format using Magic's hierarchical extraction engine (`ext2spice`).

---

## ⚡ Operational Modes & Waveform Analysis

* **Hold State**: `WL = 0`. Access transistors are OFF; cross-coupled inverters maintain state indefinitely through positive feedback.
* **Read Access**: Bitlines (`BL` and `BLB`) precharged to $V_{DD}$; `WL` asserted high. The side storing '0' discharges its respective bitline through the access transistor and driver NMOS.
* **Write Access**: `BL` and `BLB` driven differentially with write data; `WL` asserted high to overpower the internal state.

---

## 🛠️ Extraction & SPICE Verification Flow

```bash
# 1. Open layout in Magic VLSI with Sky130 tech file
magic -T sky130A.tech srambc1.mag

# 2. Inside Magic command window, extract SPICE netlist:
extract all
ext2spice lvs
ext2spice

# 3. Run transient simulation in Ngspice:
ngspice sram_tb.spice
```
