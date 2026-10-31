# Day 2 – Sequential Logic, Simulation and Synthesis

## Objective

Day 2 focused on understanding flip-flops, reset and set operations, PVT conditions, standard-cell libraries, RTL simulation, and synthesis using Yosys.

---

## 1. Flip-Flops

A flip-flop is a sequential logic element used to store one bit of data.

Flip-flops are used to:

- Store data
- Store the state of a digital circuit
- Build registers and counters
- Synchronize operations with a clock

---

## 2. Synchronous and Asynchronous Operation

### Synchronous

A synchronous control signal affects the flip-flop only at the active clock edge.

```text
Input → Flip-Flop → Output
             ↑
           Clock
```

### Asynchronous

An asynchronous control signal can affect the flip-flop without waiting for the clock.

```text
Input → Flip-Flop → Output
             ↑
        Clock / Reset / Set
```

---

## 3. Synchronous Reset

A synchronous reset resets the output only at the active clock edge.

```verilog
always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

**Key point:** Reset is checked only when the clock edge occurs.

---

## 4. Asynchronous Reset

An asynchronous reset can reset the output immediately when the reset signal becomes active. It does not wait for the clock.

```verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

**Key point:** Reset is included in the sensitivity list, so it can act independently of the clock.

---

## 5. Asynchronous Set

An asynchronous set forces the output to logic `1` immediately when the set signal becomes active.

```verilog
always @(posedge clk or posedge set)
begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end
```

**Key point:** Asynchronous set has priority over normal data capture.

---

## 6. Reset and Set Comparison

| Feature | Synchronous Reset | Asynchronous Reset | Asynchronous Set |
|---|---|---|---|
| Output | `0` | `0` | `1` |
| Clock required | Yes | No | No |
| Response | At clock edge | Immediate | Immediate |
| Purpose | Reset stored state | Immediate reset | Immediate set |

---

# 7. PVT – Process, Voltage and Temperature

PVT stands for:

- **Process** – Manufacturing variations in semiconductor devices.
- **Voltage** – Variation in the supply voltage.
- **Temperature** – Variation in operating temperature.

PVT conditions can affect the delay, timing, power and performance of digital circuits.

### Example SKY130 Library

```text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

The PVT information is represented by:

```text
tt_025C_1v80
```

- `tt` → Typical process
- `025C` → 25°C temperature
- `1v80` → 1.8V supply voltage

---

# 8. Liberty (.lib) File

A Liberty file contains characterization information about standard cells.

It provides information such as:

- Cell names
- Input and output pins
- Logic functionality
- Timing information
- Delay
- Power
- Area

The `.lib` file is used during synthesis and technology mapping.

---

# 9. RTL Simulation

RTL simulation is used to verify whether the Verilog design behaves as expected before synthesis.

### Simulation Flow

```text
RTL Design
    +
Testbench
    ↓
Icarus Verilog
    ↓
Simulation
    ↓
VCD File
    ↓
GTKWave
    ↓
Waveform Analysis
```

---

# 10. Testbench

A testbench is used to provide inputs to the design and observe its outputs.

It helps verify the functionality of the RTL design before synthesis.

```text
Stimulus
   ↓
Design
   ↓
Output
```

---

# 11. Icarus Verilog

Icarus Verilog is an open-source Verilog simulator used to compile and simulate RTL designs.

### Compile

```bash
iverilog design.v testbench.v
```

**Purpose:** Compiles the Verilog design and testbench.

### Run Simulation

```bash
./a.out
```

**Purpose:** Runs the compiled simulation.

---

# 12. VCD – Value Change Dump

VCD stands for **Value Change Dump**.

A VCD file stores changes in signal values during simulation.

It can be viewed using GTKWave.

---

# 13. GTKWave

GTKWave is a waveform viewer used to analyze simulation results.

It helps observe:

- Clock
- Reset
- Set
- Input signals
- Output signals
- Signal transitions
- Timing relationships

### Open Waveform

```bash
gtkwave waveform.vcd
```

**Purpose:** Opens the VCD waveform for visual analysis.

---

# 14. RTL Synthesis

Synthesis converts RTL code into a hardware representation using available standard cells.

```text
RTL Design
    ↓
Synthesis
    ↓
Logic / Standard Cells
    ↓
Netlist
```

---

# 15. Yosys

Yosys is an open-source RTL synthesis tool.

It is used to:

- Read Verilog RTL
- Synthesize the design
- Optimize logic
- Perform technology mapping
- Generate a synthesized netlist

---

# 16. Commands Used

### Start Yosys

```bash
yosys
```

**Purpose:** Starts the Yosys synthesis environment.

### Read Verilog

```text
read_verilog multiple_modules.v
```

**Purpose:** Reads the RTL Verilog design into Yosys.

### Read Liberty Library

```text
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd_tt_025C_1v80.lib
```

**Purpose:** Loads the SKY130 standard-cell library information.

### Synthesis

```text
synth -top multiple_modules
```

**Purpose:** Performs synthesis using `multiple_modules` as the top module.

### Technology Mapping

```text
abc -liberty ../my_lib/lib/sky130_fd_sc_hd_tt_025C_1v80.lib
```

**Purpose:** Maps the synthesized logic to cells available in the selected standard-cell library.

### Flatten

```text
flatten
```

**Purpose:** Removes the module hierarchy and creates a flattened representation.

### Show Design

```text
show
```

**Purpose:** Displays a graphical representation of the synthesized design.

### Write Verilog

```text
write_verilog multiple_modules_flat.v
```

**Purpose:** Saves the synthesized/flattened design as a Verilog netlist.

---

# 17. Hierarchical Design

In a hierarchical design, the module structure is preserved.

```text
Top Module
   ├── Module A
   ├── Module B
   └── Module C
```

### Advantages

- Module structure is maintained.
- Easier to understand.
- Easier to debug.
- Useful for modular designs.

---

# 18. Flattened Design

In a flattened design, the hierarchy between modules is removed.

```text
Before:

Top
 ├── Module A
 └── Module B

After:

Single Combined Design
```

### Advantages

- Logic from different modules can be optimized together.
- Provides a single-level representation.
- Useful during synthesis and technology mapping.

---

# 19. Hierarchical vs Flattened Design

| Hierarchical Design | Flattened Design |
|---|---|
| Module structure is preserved | Module hierarchy is removed |
| Submodules remain identifiable | Modules are combined |
| Easier to understand | More difficult to trace original modules |
| Easier debugging | Debugging can be more difficult |
| Good for modular analysis | Useful for global optimization |

---

# 20. Overall RTL Flow

```text
RTL Design
    ↓
Testbench
    ↓
Icarus Verilog
    ↓
Simulation
    ↓
VCD File
    ↓
GTKWave
    ↓
RTL Verification
    ↓
Yosys
    ↓
Read Liberty Library
    ↓
Synthesis
    ↓
Technology Mapping
    ↓
Flattening
    ↓
Synthesized Netlist
```

---

# 21. Learning Outcomes

### Sequential Logic

- Understood the purpose of flip-flops.
- Learned synchronous operation.
- Learned asynchronous operation.
- Understood synchronous reset.
- Understood asynchronous reset.
- Understood asynchronous set.
- Learned the difference between synchronous and asynchronous controls.

### PVT and Libraries

- Understood Process, Voltage and Temperature.
- Learned how PVT conditions affect circuit performance.
- Understood the SKY130 library naming convention.
- Learned the purpose of a Liberty `.lib` file.
- Understood that Liberty files contain cell timing, power, area and functional information.

### Simulation

- Understood the role of a testbench.
- Learned to compile Verilog using Icarus Verilog.
- Learned to run an RTL simulation.
- Learned about VCD files.
- Learned to analyze waveforms using GTKWave.

### Synthesis

- Understood the purpose of RTL synthesis.
- Learned how Yosys reads Verilog RTL.
- Learned how to load a standard-cell library.
- Understood technology mapping.
- Learned about hierarchical and flattened designs.
- Learned how to generate a synthesized Verilog netlist.

---

# 22. Key Takeaways

- Flip-flops are basic building blocks of sequential logic.
- Synchronous controls depend on the clock.
- Asynchronous reset and set can act independently of the clock.
- PVT conditions affect circuit timing and performance.
- Liberty files provide standard-cell characterization information.
- Icarus Verilog is used for RTL simulation.
- GTKWave is used for waveform analysis.
- Yosys is used for RTL synthesis.
- Technology mapping connects synthesized logic to standard cells.
- Flattening removes module hierarchy and provides a combined design representation.

---

## Conclusion

Day 2 helped connect the concepts of **sequential logic, RTL simulation and synthesis**.

The practical work demonstrated how a Verilog design is verified using a testbench and waveform analysis, and how the verified RTL can then be synthesized and mapped to standard cells using Yosys and the SKY130 library.

**RTL → Simulation → Waveform Analysis → Synthesis → Technology Mapping → Netlist**
