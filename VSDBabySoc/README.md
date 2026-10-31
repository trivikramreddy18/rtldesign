# VSDBabySoC

## 1. Introduction to VSDBabySoC

VSDBabySoC is a small System-on-Chip design that combines different hardware blocks into one design. Working on this project helped me understand how a bigger RTL design is simulated and synthesized instead of working only with small Verilog modules.

In this project, I explored the RTL files, performed pre-synthesis simulation, synthesized the design using Yosys, generated the gate-level netlist, and performed post-synthesis functional simulation.

---

## 2. Understanding the SoC Architecture

The VSDBabySoC design contains different blocks connected together through a top-level module.

A simple view of the design is:

```text
                    VSDBabySoC
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       RVMYTH           PLL            DAC
          │              │              │
      RISC-V Core     Clocking       Output
```

The top-level module connects these blocks and allows them to work together as a single system.

---

## 3. Main Building Blocks

### RVMYTH

RVMYTH is the RISC-V processor core used in the design. It performs the main processing operations.

### PLL

PLL stands for Phase Locked Loop. It is used for clock generation in the design.

### DAC

DAC stands for Digital-to-Analog Converter. It converts digital values into analog output.

### Top Module

The `vsdbabysoc` module acts as the top-level module and connects the different blocks together.

---

## 4. VSDBabySoC Design Flow

While designing a chip, the design goes through different stages before reaching the final physical implementation.

A general ASIC flow looks like this:

```text
Specification
      ↓
RTL Design
      ↓
Functional Verification
      ↓
Logic Synthesis
      ↓
Static Timing Analysis
      ↓
Floorplanning
      ↓
Placement
      ↓
Clock Tree Synthesis
      ↓
Routing
      ↓
Physical Verification
      ↓
GDSII
```

For the VSDBabySoC work completed so far, the flow is:

```text
RTL Design
    ↓
Pre-Synthesis Simulation
    ↓
Logic Synthesis
    ↓
Technology Mapping
    ↓
Gate-Level Netlist
    ↓
Post-Synthesis Simulation
```

---

## 5. Pre-Synthesis Simulation

Before synthesizing the design, the RTL is simulated to check whether the design is working correctly.

In this stage, the original Verilog RTL files are used along with the testbench.

The flow is:

```text
RTL Files
   +
Testbench
   ↓
Compilation
   ↓
Simulation
   ↓
Output / Waveform
```

For pre-synthesis simulation, the following command was used:

```bash
iverilog -o ./pre_synth_sim.out -DPRE_SYNTH_SIM src/module/testbench.v -I src/include -I src/module/
```

### What this command does

| Command                  | Purpose                                           |
| ------------------------ | ------------------------------------------------- |
| `iverilog`               | Compiles the Verilog files                        |
| `-o ./pre_synth_sim.out` | Creates the simulation output file with this name |
| `-DPRE_SYNTH_SIM`        | Defines the macro for pre-synthesis simulation    |
| `src/module/testbench.v` | Testbench used to simulate the design             |
| `-I src/include`         | Adds the include folder                           |
| `-I src/module/`         | Adds the module folder                            |

This simulation checks the RTL design before converting it into a gate-level netlist.

---

## 6. RTL Simulation Tools

### Icarus Verilog

Icarus Verilog is used to compile and simulate the Verilog design.

For example:

```bash
iverilog <verilog_files>
```

After compilation, the generated simulation file can be executed using:

```bash
vvp <simulation_output>
```

### GTKWave

GTKWave is used to view the generated waveform.

It helps us observe signals such as:

* Clock
* Reset
* Inputs
* Outputs
* Signal changes

This makes it easier to understand whether the design is behaving correctly.

---

## 7. Include Paths and Compilation

The VSDBabySoC design contains files in different directories.

Instead of putting every file path manually in the command, include paths are provided using the `-I` option.

For example:

```bash
-I src/include
```

This tells the compiler to search inside the `src/include` directory when required files are included.

Similarly:

```bash
-I src/module/
```

allows the compiler to access files inside the module directory.

During post-synthesis simulation, the Sky130 standard cell models are also included.

```bash
-I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/
```

These paths are important because the complete design contains multiple modules and dependencies.

---

## 8. Logic Synthesis

Logic synthesis converts the RTL design into a gate-level implementation.

Before synthesis, we write the design using Verilog RTL.

After synthesis, the same logic is represented using cells from the technology library.

The basic idea is:

```text
RTL Verilog
    ↓
Synthesis
    ↓
Logic Optimization
    ↓
Technology Mapping
    ↓
Gate-Level Netlist
```

For VSDBabySoC, Yosys was used for the synthesis process.

---

## 9. Yosys Synthesis Flow

The following commands were used during synthesis.

### Starting Yosys

```tcl
yosys
```

This starts the Yosys synthesis tool.

---

### Reading the Top-Level Design

```tcl
read_verilog src/module/vsdbabysoc.v
```

This command reads the main top-level Verilog file into Yosys.

The `vsdbabysoc` module connects the major blocks of the design.

---

### Reading the RVMYTH Module

```tcl
read_verilog -I src/include src/module/rvmyth.v
```

This reads the RVMYTH processor module.

The `-I src/include` option allows Yosys to find the required include files.

---

### Reading the Clock Gate Module

```tcl
read_verilog -I src/include src/module/clk_gate.v
```

This reads the clock gate module and also provides the required include directory.

After reading all the required Verilog files, Yosys understands the design hierarchy.

---

### Selecting and Synthesizing the Top Module

```tcl
synth -top vsdbabysoc
```

This command tells Yosys that `vsdbabysoc` is the top module of the complete design.

Yosys then starts synthesizing the design from this top module.

During synthesis, Yosys processes the RTL and converts it into an internal logic representation.

---

### Mapping Flip-Flops

```tcl
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This command maps the flip-flops and other sequential elements in the design to cells available in the Sky130 standard cell library.

In simple terms:

```text
RTL Flip-Flop
      ↓
dfflibmap
      ↓
Sky130 Flip-Flop Cell
```

---

### Technology Mapping using ABC

```tcl
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The `abc` command maps the combinational logic into standard cells available in the Sky130 library.

For example, logic operations such as:

* AND
* OR
* NAND
* NOR
* Inverters

are mapped to actual cells from the technology library.

This is an important step because it changes generic logic into technology-specific logic.

---

### Replacing Undefined Values

```tcl
setundef -zero
```

Sometimes the design may contain undefined values.

This command replaces undefined values with logic `0`.

This helps clean the synthesized design and avoids unknown values in the generated logic.

---

### Cleaning the Design

```tcl
clean -purge
```

This command removes unused cells and unnecessary connections from the design.

The command was used twice:

```tcl
clean -purge
clean -purge
```

The purpose is to make sure unnecessary objects are removed after synthesis and mapping operations.

---

### Renaming Generated Objects

```tcl
rename -enumerate
```

During synthesis, Yosys may generate internal names that are difficult to read.

This command renames objects in an organized enumerated format.

It makes the generated design and netlist easier to handle.

---

## 10. Sky130 Standard Cell Library

A standard cell library is needed to map the RTL design into actual technology cells.

For this project, Sky130 libraries were used.

The following libraries were read:

### PLL Library

```tcl
read_liberty -lib src/lib/avsdpll.lib
```

This reads the Liberty file related to the PLL block.

---

### DAC Library

```tcl
read_liberty -lib src/lib/avsddac.lib
```

This reads the Liberty file related to the DAC block.

---

### Sky130 Standard Cell Library

```tcl
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This reads the main Sky130 standard cell library.

The library contains cells required for technology mapping such as:

* Logic gates
* Buffers
* Inverters
* Flip-Flops

The general flow is:

```text
RTL Design
    ↓
Yosys
    ↓
Sky130 Libraries
    ↓
Technology Mapping
    ↓
Gate-Level Design
```

---

## 11. Gate-Level Netlist Generation

After the synthesis and technology mapping steps, the RTL design is converted into a gate-level representation.

The generated design contains standard cells instead of high-level RTL statements.

The transformation can be understood as:

```text
RTL Design
│
├── always blocks
├── assign statements
└── Verilog logic

        ↓

      Synthesis

        ↓

Gate-Level Netlist
│
├── Standard Cells
├── Flip-Flops
├── Logic Gates
└── Interconnections
```

After synthesis, the design is flattened into a gate-level representation where the logic hierarchy is converted into technology-mapped cells.

This netlist is then used for post-synthesis simulation.

---

## 12. Post-Synthesis Simulation

After synthesis, the gate-level netlist is simulated again.

This is called post-synthesis simulation.

Unlike pre-synthesis simulation, this stage does not use only the original RTL logic. The synthesized netlist and standard cell models are used.

The flow is:

```text
Testbench
    +
Gate-Level Netlist
    +
Sky130 Standard Cell Models
    ↓
Icarus Verilog
    ↓
Post-Synthesis Simulation
```

The command used was:

```bash
sudo iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL \
-I src/include/ \
-I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ \
-I src/module/ \
src/module/testbench.v
```

This checks whether the synthesized design works correctly after technology mapping.

---

## 13. Understanding POST_SYNTH_SIM

The following option is used:

```bash
-DPOST_SYNTH_SIM
```

The `-D` option defines a macro during compilation.

So:

```bash
-DPOST_SYNTH_SIM
```

allows the Verilog code to identify that post-synthesis simulation is being performed.

The code can use conditional compilation like this:

```verilog
`ifdef POST_SYNTH_SIM

    // Post-synthesis configuration

`else

    // Pre-synthesis configuration

`endif
```

This helps the same testbench or design setup work differently depending on whether RTL simulation or post-synthesis simulation is being performed.

---

## 14. Understanding FUNCTIONAL

The following option is used during post-synthesis simulation:

```bash
-DFUNCTIONAL
```

The synthesized design contains Sky130 standard cells.

These cells have Verilog models used during simulation.

The `FUNCTIONAL` macro enables the functional behavior of these standard cell models.

The main purpose is to check the logical functionality of the synthesized design.

In simple terms:

```text
RTL Design
    ↓
Synthesis
    ↓
Standard Cells
    ↓
Functional Cell Models
    ↓
Simulation
```

---

## 15. Complete Simulation and Synthesis Flow

The complete flow followed in this project is:

```text
                    VSDBabySoC
                         │
                         ▼
                 RTL Design Files
                         │
                         ▼
              Pre-Synthesis Simulation
                         │
                         ▼
              RTL Functional Check
                         │
                         ▼
                    Yosys
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
          Read RTL   Read Libs   Top Module
                         │
                         ▼
                     Synthesis
                         │
                         ▼
                    dfflibmap
                         │
                         ▼
                       abc
                         │
                         ▼
                 Technology Mapping
                         │
                         ▼
                  Gate-Level Netlist
                         │
                         ▼
              Post-Synthesis Simulation
                         │
              ┌──────────┼──────────┐
              │          │          │
     POST_SYNTH_SIM  FUNCTIONAL  Sky130 Models
                         │
                         ▼
                Functional Verification
```

### Commands Used During the Flow

```tcl
yosys

read_verilog src/module/vsdbabysoc.v

read_verilog -I src/include src/module/rvmyth.v

read_verilog -I src/include src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib

read_liberty -lib src/lib/avsddac.lib

read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

setundef -zero

clean -purge

clean -purge

rename -enumerate

stat
```

### What `stat` Does

```tcl
stat
```

The `stat` command displays statistics about the synthesized design.

It helps us see information such as:

* Number of cells
* Types of cells
* Number of wires
* Design hierarchy information

This gives a quick summary of the synthesized design after the synthesis flow.

---

## 16. Learning Outcomes

Through the VSDBabySoC project, I understood how a larger RTL design moves through the initial stages of the ASIC design flow.

The main things I learned are:

* Understanding the architecture of a multi-module SoC
* Identifying the top module and different IP blocks
* Performing pre-synthesis simulation
* Using Icarus Verilog for RTL compilation and simulation
* Using GTKWave to observe simulation results
* Working with include paths
* Reading multiple Verilog files in Yosys
* Reading Liberty files
* Selecting the top module for synthesis
* Understanding the `synth` command
* Mapping sequential logic using `dfflibmap`
* Mapping combinational logic using `abc`
* Using Sky130 standard cell libraries
* Cleaning and organizing the synthesized design
* Understanding gate-level representation
* Generating and analyzing the synthesized design
* Using `stat` to view synthesis statistics
* Performing post-synthesis simulation
* Understanding `POST_SYNTH_SIM`
* Understanding `FUNCTIONAL`
* Comparing RTL-level and synthesized-level functionality
