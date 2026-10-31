# VSDBabySoC — RTL to GDSII

**End-to-end ASIC Design | SKY130HD | RTL Design, Synthesis, Physical Design and Timing Analysis**

VSDBabySoC is a System-on-Chip (SoC) design integrating an RVMYTH RISC-V processor, PLL and DAC. This project documents my work from RTL simulation and synthesis through physical implementation, post-route timing analysis and the final GDSII layout stage.

The objective is to understand how a Verilog design is transformed into a technology-mapped netlist and then implemented as a physical chip layout using the SKY130HD technology platform.

![VSDBabySoC RTL to GDSII Flow](openlane.png)

*Complete RTL-to-GDSII flow showing the major design, implementation and verification stages.*

## Project at a Glance

| Item | Details |
|---|---|
| Design | VSDBabySoC |
| Processor | RVMYTH RISC-V core |
| Supporting IP | PLL and 10-bit DAC |
| Technology | SKY130HD |
| RTL synthesis | Yosys |
| Simulation | Icarus Verilog and GTKWave |
| Physical design | OpenROAD-based flow |
| Static timing analysis | OpenSTA / OpenROAD |
| Timing assessment | Post-route multi-corner STA across 16 PVT corners |
| Physical verification | DRC and LVS |
| Final layout format | GDSII |

## 1. SoC Architecture

The VSDBabySoC top-level module integrates the major functional blocks into a single system.

- **RVMYTH:** RISC-V processor core responsible for processing instructions.
- **PLL:** Phase-Locked Loop for clock generation.
- **DAC:** Digital-to-Analog Converter for analog output.
- **Top-level integration:** Connects the processor and supporting blocks through their interfaces.

The project covers both logical implementation and the physical-design concepts needed to transform the synthesized design into a layout.

## 2. Complete RTL-to-GDSII Flow

The project is organized into three major phases.

### Phase A — RTL to Gate-Level Simulation (GLS)

1. RTL design exploration and hierarchy understanding.
2. Pre-synthesis functional simulation.
3. Logic synthesis using Yosys.
4. SKY130HD standard-cell technology mapping.
5. Gate-level netlist generation.
6. Post-synthesis functional simulation using standard-cell models.

### Phase B — Physical Design

1. Floorplanning.
2. Power planning and I/O placement.
3. Standard-cell placement and optimization.
4. Clock Tree Synthesis (CTS).
5. Global routing and detailed routing.

### Phase C — Timing Analysis and Physical Verification

1. Post-route multi-corner Static Timing Analysis.
2. Setup and hold timing assessment.
3. Design Rule Checking (DRC).
4. Layout Versus Schematic (LVS).
5. Final GDSII layout generation and verification.

The exact completion status of each stage is documented in its corresponding README and supported by the available implementation reports and screenshots.

## 3. Physical Design Documentation

Explore each stage of the project below.

| Stage | Documentation |
|---|---|
| 01 | [Floorplanning](01_Floorplanning/README.md) |
| 02 | [Power Planning and I/O Placement](02_Power_Planning/README.md) |
| 03 | [Standard-Cell Placement](03_Placement/README.md) |
| 04 | [Clock Tree Synthesis](04_CTS/README.md) |
| 05 | [Global and Detailed Routing](05_Routing/README.md) |
| 06 | [Post-Route Multi-Corner STA](06_Post_Route_STA/README.md) |
| 07 | [Design Rule Checking (DRC)](07_DRC/README.md) |
| 08 | [Layout Versus Schematic (LVS)](08_LVS/README.md) |
| 09 | [GDSII Generation](09_GDSII/README.md) |

Each stage documents its underlying theory, purpose, implementation methodology, relevant commands where applicable, results and screenshots.

## 4. Post-Route Multi-Corner STA

Post-route Static Timing Analysis evaluates timing using the implemented design, timing constraints, cell libraries and interconnect parasitics.

My STA assessment covers **16 SKY130HD PVT corners**, including FF, SS and TT process groups.

The report evaluates:

- Setup Worst Slack (WS), WNS and TNS.
- Hold Worst Slack (WS), WNS and TNS.
- Timing behavior across different process, voltage and temperature conditions.
- Critical setup corners and the need to review unconstrained endpoints.

The supplied report identifies `ss_n40C_1v28` as the worst setup corner, with a reported setup WS of **−28.66 ns**. The supplied hold WS values are positive across all 16 corners.

These results indicate that setup timing requires further investigation. Complete timing sign-off also requires verification of constraint coverage and the associated timing models.

**Detailed report:** [Post-Route Multi-Corner STA Report](VSDBabySoC_Post_Route_Multi_Corner_STA_Report_final.pdf)

## 5. Physical Verification and GDSII

### Design Rule Checking (DRC)

DRC checks whether the layout complies with the manufacturing rules of the selected technology. The stage-specific documentation will contain the actual verification report, violation count and status.

### Layout Versus Schematic (LVS)

LVS compares the connectivity extracted from the layout against the reference netlist. The documentation will record the actual comparison results and any reported mismatches.

### GDSII Generation

GDSII represents the physical layout geometry and layer information for downstream semiconductor manufacturing workflows. The final section documents the generated layout, output files and available verification evidence.

DRC-clean, LVS-correct and timing-closed status must be established from the actual reports rather than inferred from the presence of a layout file.

## 6. VSD RTL Design Workshop — Day 1 to Day 10

This VSDBabySoC project builds upon the concepts and practical exercises completed during the VSD RTL Design Workshop. The workshop documentation covers my learning progress from Day 1 to Day 10, connecting RTL design and synthesis concepts with the physical-design implementation of VSDBabySoC.

| Workshop | Documentation |
|---|---|
| Day 1 | [Day 1 — RTL Design Workshop](../Day1/) |
| Day 2 | [Day 2 — RTL Design Workshop](../Day2/) |
| Day 3 | [Day 3 — RTL Design Workshop](../Day3/) |
| Day 4 | [Day 4 — RTL Design Workshop](../Day4/) |
| Day 5 | [Day 5 — RTL Design Workshop](../Day5/) |
| Day 6 | [Day 6 — RTL Design Workshop](../Day6/) |
| Day 7 | [Day 7 — RTL Design Workshop](../Day7/) |
| Day 8 | [Day 8 — RTL Design Workshop](../Day8/) |
| Day 9 | [Day 9 — RTL Design Workshop](../Day9/) |
| Day 10 | [Day 10 — RTL Design Workshop](../Day10/) |

These workshop folders document the progression of my learning, while the individual VSDBabySoC physical-design READMEs document the application of relevant concepts to the project.

**Repository:** [VSD-RTL-Design-workshop](https://github.com/murali2755/VSD-RTL-Design-workshop)


## 7. Key Learning Outcomes

- Understanding the architecture of a multi-block SoC.
- Performing RTL simulation and logic synthesis.
- Understanding technology mapping and gate-level netlists.
- Learning the physical-design stages from floorplanning through routing.
- Understanding clock-tree construction and timing optimization.
- Analyzing setup and hold timing across multiple PVT corners.
- Understanding DRC, LVS and GDSII in the ASIC implementation flow.
- Connecting theoretical VLSI concepts with practical implementation evidence.

## References

- [Reference VSDBabySoC Physical Design Flow](https://github.com/cpatilabhijith/RTL_Workspace/blob/main/VSDBabySoC/PD_flow.md)
- [My VSD RTL Design Workshop Repository](https://github.com/murali2755/VSD-RTL-Design-workshop)
- [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD)
- [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)

---

**Project:** VSDBabySoC — RTL to GDSII  
**Technology:** SKY130HD  
**Focus:** RTL Design, Gate-Level Simulation, Physical Design, Multi-Corner STA and Physical Verification
