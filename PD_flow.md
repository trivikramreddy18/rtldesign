# VSDBabySoC: Physical Design Flow from RTL to GDSII

**Technology:** SKY130HD  
**Design:** VSDBabySoC  
**Tools:** Yosys, OpenROAD, OpenSTA and SKY130 PDK  
**Project focus:** Physical Design, Post-Route Multi-Corner STA and Physical Verification

---

## 1. Introduction

VSDBabySoC is a small mixed-signal System-on-Chip (SoC) integrating an RVMYTH RISC-V processor core with supporting blocks such as a PLL and a 10-bit DAC.

This project documents the implementation of VSDBabySoC using the SKY130HD technology platform, starting from the synthesized gate-level design and progressing through physical design, timing analysis and the final layout-generation stage.

The physical design flow transforms the logical netlist into a physical representation by assigning locations to cells, establishing the power distribution network, constructing the clock network, routing interconnections and checking the design against timing and manufacturing requirements.

## 2. Project Objectives

- Understand the physical implementation of VSDBabySoC using the SKY130HD platform.
- Study floorplanning, power planning, placement, Clock Tree Synthesis (CTS) and routing.
- Relate physical-design concepts to the theory studied during the VSD RTL Design Workshop.
- Perform post-route Static Timing Analysis across 16 process-voltage-temperature (PVT) corners.
- Understand Design Rule Checking (DRC), Layout Versus Schematic (LVS) and GDSII generation.
- Document implementation results, timing reports and physical-design screenshots.

## 3. Design Overview

VSDBabySoC combines digital processing with supporting mixed-signal IP blocks.

| Component | Function |
|---|---|
| VSDBabySoC top module | Integrates the SoC components |
| RVMYTH RISC-V core | Executes processor instructions |
| PLL (`avsdpll`) | Generates clock signals |
| 10-bit DAC (`avsddac`) | Converts digital data into an analog representation |
| Clock-gating logic | Controls clock delivery to supported logic blocks |
| SKY130HD standard cells | Implement the synthesized digital logic |

The physical implementation must account for both standard-cell logic and the physical and timing requirements of the PLL and DAC macros.

## 4. Overall RTL-to-GDSII Flow

The complete ASIC implementation flow can be represented as follows:

```text
RTL Design
    |
    v
RTL Functional Simulation
    |
    v
Logic Synthesis using Yosys
    |
    v
SKY130HD Technology Mapping
    |
    v
Gate-Level Netlist
    |
    v
Gate-Level Simulation (GLS)
    |
    v
Floorplanning
    |
    v
Power Planning and I/O Placement
    |
    v
Standard-Cell Placement
    |
    v
Clock Tree Synthesis (CTS)
    |
    v
Global and Detailed Routing
    |
    v
Post-Route Static Timing Analysis
    |
    v
DRC and LVS Verification
    |
    v
GDSII Layout Generation
```

Each stage depends on the correctness of the preceding stage. The final verification status must be established using the actual reports produced by the implementation flow.

## 5. Workshop Theory Applied to Physical Design

### 5.1 Day 7 – Floorplanning and Physical Design Concepts

Floorplanning establishes the physical organization of a chip before standard-cell placement.

Important concepts studied include:

- **Core area:** The region allocated for implementing the design logic.
- **Die area:** The overall physical boundary of the chip.
- **Core utilization:** The proportion of core area occupied by standard cells.
- **Aspect ratio:** The relationship between the dimensions of the core.
- **I/O pin placement:** Assigning physical locations to input and output pins.
- **Power planning:** Distributing power and ground throughout the design.
- **Placement blockages:** Restricting cell placement in designated regions.

The power distribution network uses VDD and VSS connections to supply the standard cells and other supported blocks.

These concepts are relevant to VSDBabySoC because the RVMYTH logic and the PLL/DAC macros must coexist within the same physical design.

### 5.2 Day 8 – CMOS Fabrication Fundamentals

Understanding CMOS fabrication helps explain how the physical layout represents actual semiconductor devices.

The studied concepts include:

- P-type substrate and N-well formation.
- Active regions and device isolation.
- Gate oxide and polysilicon gate formation.
- Source and drain formation.
- Contacts, vias and metal interconnections.

NMOS and PMOS devices form the basis of CMOS standard cells. The physical layout represents their geometry and electrical connections through different fabrication layers.

In physical design, the SKY130HD standard-cell library supplies predefined cells and their associated physical and timing models. The implementation tools use these models to place and connect the cells.

### 5.3 Day 9 – Static Timing Analysis Fundamentals

Static Timing Analysis evaluates whether data signals can propagate through the design within the required timing limits.

**Setup timing** checks that data arrives early enough before the active clock edge.

\[
\text{Setup Slack}=\text{Required Time}-\text{Arrival Time}
\]

Positive setup slack indicates that the analyzed setup requirement is met. Negative setup slack indicates a timing violation.

**Hold timing** checks that data does not arrive too early after the active clock edge.

\[
\text{Hold Slack}=\text{Arrival Time}-\text{Required Time}
\]

Positive hold slack indicates that the analyzed hold requirement is met, while negative hold slack indicates a hold violation.

Important timing concepts include:

- Cell propagation delay.
- Input transition or slew.
- Output load capacitance.
- Clock latency and skew.
- Setup and hold requirements.
- Worst Negative Slack (WNS).
- Total Negative Slack (TNS).

For VSDBabySoC, these concepts are applied in the post-route analysis of the design across multiple SKY130HD PVT corners.

### 5.4 Day 10 – Routing and Physical Verification Fundamentals

Routing establishes the physical electrical connections between placed cells and macros.

A routing algorithm searches the available routing space while avoiding obstacles and obeying technology constraints. Maze-routing techniques provide a conceptual foundation for understanding this process.

Physical verification includes:

- **DRC:** Checks layout geometry against manufacturing design rules.
- **LVS:** Checks whether the extracted layout connectivity matches the reference netlist.
- **Parasitic extraction:** Estimates or extracts interconnect resistance and capacitance for timing analysis.
- **GDSII generation:** Produces a layout database representing the physical design.

These checks are essential because a logically correct design is not necessarily a physically valid layout.

---

## 6. Floorplanning

Floorplanning establishes the initial physical arrangement of the design.

The die and core boundaries are defined, and space is allocated for standard cells, macros, routing resources and power distribution.

### Key activities

- Define the core and die dimensions.
- Establish placement rows and standard-cell sites.
- Account for PLL and DAC macro placement.
- Define I/O pin locations.
- Prepare the design for power planning and placement.

### VSDBabySoC floorplan

The floorplan must accommodate the RVMYTH processor logic and the supporting macro blocks while providing sufficient space for routing and power distribution.

**Screenshot – Floorplan**



*Replace `floorplan.png` with the filename of your uploaded floorplan screenshot.*

## 7. Power Planning and I/O Placement

Power planning creates a network that distributes power and ground to the cells throughout the design.

The Power Distribution Network (PDN) commonly uses power connections, rings and straps to deliver VDD and VSS across the physical layout.

A well-designed PDN helps control voltage drop and power-integrity problems.

I/O placement assigns physical positions to the design's external input and output pins. Pin locations must be compatible with the floorplan, macro positions and routing requirements.

### VSDBabySoC implementation

The power network must connect the standard cells and supported macro power pins according to their physical and electrical requirements.



## 8. Standard-Cell Placement

Placement assigns legal physical locations to the standard cells in the synthesized netlist.

The placement process considers cell density, estimated wirelength, timing, congestion and physical constraints.

### Placement stages

1. **Global placement:** Distributes cells across the core to optimize placement objectives.
2. **Legalization:** Moves cells into legal locations without overlap.
3. **Detailed placement:** Refines cell locations to improve placement quality and wirelength.
4. **Placement optimization:** Applies supported timing and electrical optimizations.

For VSDBabySoC, placement must respect the macro locations and provide suitable routing resources for the processor logic and other blocks.

## 9. Clock Tree Synthesis (CTS)

Clock Tree Synthesis constructs the clock distribution network required by the sequential elements of the design.

A clock signal must reach the intended clock sinks with acceptable latency, skew and transition characteristics.

### Main objectives

- Distribute the clock to the required sequential cells.
- Insert suitable clock buffers.
- Control clock skew and insertion latency.
- Check clock transition and fanout requirements.
- Re-evaluate setup and hold timing after clock-tree construction.

Clock skew is the difference in clock arrival times at different sequential elements. Excessive skew can negatively affect timing.


## 10. Routing

Routing creates the physical metal connections between standard cells, macros and other design terminals.

The routing process must respect the available routing layers, track locations, metal-width requirements, spacing rules and via constraints.

### 10.1 Global routing

Global routing estimates routes through the available routing regions and creates routing guides. It helps evaluate congestion and determine the approximate paths of connections.

### 10.2 Detailed routing

Detailed routing assigns actual routing tracks and vias to implement the connections.

The resulting layout must be checked for routing violations, disconnected nets and other physical implementation problems.

### VSDBabySoC routing results

The routed design provides the physical interconnect needed for post-route timing analysis. Timing results depend on the routed database, constraints, selected timing libraries and the parasitic information used during analysis.



## 11. Post-Route Multi-Corner Static Timing Analysis

Post-route STA evaluates the timing of the implemented design using timing constraints, cell delay models and interconnect information.

The analysis in this project covers 16 SKY130HD PVT corners:

- 6 fast-fast (FF) corners.
- 8 slow-slow (SS) corners.
- 2 typical-typical (TT) corners.

PVT represents process, voltage and temperature. These operating conditions affect cell delays and the timing behavior of the circuit.

### 11.1 Timing metrics

| Metric | Meaning |
|---|---|
| Setup WS | Worst reported setup slack |
| Setup WNS | Worst negative setup slack |
| Setup TNS | Accumulated negative setup slack |
| Hold WS | Worst reported hold slack |
| Hold WNS | Worst negative hold slack |
| Hold TNS | Accumulated negative hold slack |

Negative setup slack indicates a setup violation. Negative hold slack indicates a hold violation. Zero WNS or TNS must be interpreted alongside the coverage of the constraints and the paths included in the reports.

### 11.2 Setup timing results

The supplied post-route report identifies `ss_n40C_1v28` as the worst setup corner, with a reported setup WS of **−28.66 ns**.

| PVT corner | Setup WS (ns) |
|---|---:|
| `ff_100C_1v65` | 8.20 |
| `ff_100C_1v95` | 9.06 |
| `ff_n40C_1v56` | 7.05 |
| `ff_n40C_1v65` | 7.76 |
| `ff_n40C_1v76` | 8.38 |
| `ff_n40C_1v95` | 9.07 |
| `ss_100C_1v40` | -3.31 |
| `ss_100C_1v60` | 1.75 |
| `ss_n40C_1v28` | -28.66 |
| `ss_n40C_1v35` | -15.83 |
| `ss_n40C_1v40` | -10.32 |
| `ss_n40C_1v44` | -7.16 |
| `ss_n40C_1v60` | 0.20 |
| `ss_n40C_1v76` | 3.62 |
| `tt_025C_1v80` | 7.03 |
| `tt_100C_1v80` | 7.11 |

The negative setup slack values identify corners that require further timing investigation. In particular, `ss_n40C_1v28` exhibits the largest setup violation in the supplied results.

### 11.3 Hold timing results

All 16 reported Hold WS values are positive.

The smallest reported Hold WS is **0.20 ns**, observed at `ff_100C_1v95` and `ff_n40C_1v95`. The largest is **1.84 ns** at `ss_n40C_1v28`.

The supplied Hold WNS and Hold TNS charts remain at zero across the 16 corners. These results indicate that no negative hold slack was captured by the supplied summary, subject to the completeness of the timing constraints and reports.

### 11.4 OpenROAD timing evidence

The supplied OpenROAD GUI screenshot shows one reported timing path with:

| Timing parameter | Reported value |
|---|---:|
| Data required time | 12.80 ns |
| Data arrival time | -5.97 ns |
| Slack | 6.83 ns |
| Status | MET |

This is an individual path report and should not be confused with the worst-case results from the complete multi-corner analysis.
(postroute.png)

### 11.5 STA interpretation

The supplied results indicate that setup timing is the main concern in several slow-slow corners, especially `ss_n40C_1v28`.

The next step is to inspect the worst setup paths, verify the clock and I/O constraints, check unconstrained endpoints and confirm that each timing library is associated with the intended PVT corner.

The uploaded report also identifies missing input/output delays and unconstrained endpoints. These must be resolved or explicitly justified before claiming complete timing sign-off.

The detailed report is available here:

[VSDBabySoC Post-Route Multi-Corner STA Report](VSDBabySoC_Post_Route_Multi_Corner_STA_Report_final.pdf)

## 12. Design Rule Checking (DRC)

Design Rule Checking verifies whether the physical layout follows the manufacturing rules specified by the technology.

These rules cover physical geometry and connectivity requirements, including:

- Minimum metal width.
- Metal-to-metal spacing.
- Via dimensions and placement.
- Layer-specific restrictions.
- Minimum distances between layout shapes.
- Other process-specific manufacturing constraints.

A DRC violation indicates that part of the layout does not satisfy one or more design rules. Violations must be investigated and corrected before the layout can be considered DRC-clean.

### VSDBabySoC verification

The final DRC result should be recorded from the actual verification report. A routed layout screenshot alone does not prove that all design rules have passed.


**Result:** Record the actual violation count and verification status from the completed DRC report.

## 13. Layout Versus Schematic (LVS)

Layout Versus Schematic checks whether the electrical connectivity extracted from the physical layout matches the reference circuit or netlist.

The verification process extracts devices and connections from the layout and compares them against the reference design.

LVS checks can identify:

- Missing or additional devices.
- Incorrect net connections.
- Open connections.
- Short circuits.
- Device or connectivity mismatches.
- Other differences between the extracted layout and the reference netlist.

For VSDBabySoC, the reference must represent the intended implemented design, including the appropriate standard cells and supported macro interfaces.

### VSDBabySoC verification

A successful LVS result should be supported by a report showing that the compared designs match according to the selected extraction and comparison settings.


**Result:** Record the actual LVS status and any reported mismatches after running the verification flow.

## 14. GDSII Generation

GDSII is a widely used database format for representing integrated-circuit layout geometry and layer information.

The GDSII output contains the physical layout representation required by downstream semiconductor implementation and manufacturing workflows.

### Main activities

- Prepare the final physical-design database.
- Include the required standard-cell and macro layout data.
- Ensure that the required routing and layout layers are represented.
- Export the layout to GDSII using the supported flow.
- Verify that the output file exists and that the required layout content is included.
- Review the applicable DRC, LVS and timing reports.

### VSDBabySoC final layout

The GDSII output should represent the intended VSDBabySoC implementation, including its standard-cell logic, interconnect and required macro layout views.

The existence of a GDSII file alone does not prove that the design is DRC-clean, LVS-correct or timing-closed.



**Output file:** Record the actual generated GDSII filename and location from your implementation flow.

**Verification status:** Update this section with the actual GDSII generation, DRC and LVS results.

## 15. Conclusion

This project documents the physical-design methodology for VSDBabySoC using the SKY130HD technology platform.

The flow covers floorplanning, power planning, standard-cell placement, Clock Tree Synthesis, routing and post-route multi-corner Static Timing Analysis. The workshop theory provides the foundation for understanding physical layout organization, CMOS fabrication, timing constraints, routing algorithms and physical verification.

The supplied 16-corner STA report shows positive hold slack across all analyzed corners but negative setup slack at several slow-slow corners. The most significant reported setup violation occurs at `ss_n40C_1v28`, with a setup WS of −28.66 ns.

Further investigation of critical paths, timing constraints and unconstrained endpoints is required before claiming timing sign-off. DRC, LVS and final GDSII status should likewise be documented using the actual verification outputs.

## 16. References

- [Reference Physical Design Flow – PD_flow.md](https://github.com/cpatilabhijith/RTL_Workspace/blob/main/VSDBabySoC/PD_flow.md)
- [VSD RTL Design Workshop – Day 7](https://github.com/murali2755/VSD-RTL-Design-workshop/tree/main/Day7)
- [VSD RTL Design Workshop – Day 8](https://github.com/murali2755/VSD-RTL-Design-workshop/tree/main/Day8)
- [VSD RTL Design Workshop – Day 9](https://github.com/murali2755/VSD-RTL-Design-workshop/tree/main/Day9)
- [VSD RTL Design Workshop – Day 10](https://github.com/murali2755/VSD-RTL-Design-workshop/tree/main/Day10)
- [VSDBabySoC Post-Route Multi-Corner STA Report](VSDBabySoC_Post_Route_Multi_Corner_STA_Report_final.pdf)
- [OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenROAD)
- [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)

---

*Documentation prepared for the VSDBabySoC physical-design project.*
