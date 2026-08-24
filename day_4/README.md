# Day 4 — Gate-Level Simulation and Verilog Coding Concepts

## 1. Introduction

Day 4 focuses on understanding what happens to a Verilog design after synthesis. The main topics covered are Gate-Level Simulation, RTL and netlist comparison, sensitivity-list issues, blocking assignments, non-blocking assignments, and common reasons for simulation mismatches.

The objective is to understand how coding style can affect simulation results and synthesized hardware.

---

## 2. Gate-Level Simulation

Gate-Level Simulation, commonly called **GLS**, is performed using the synthesized netlist rather than the original RTL source code.

The synthesized design may contain standard cells such as:

* AND gates
* OR gates
* Inverters
* Multiplexers
* Flip-flops

GLS helps verify whether the synthesized circuit behaves as expected.

Basic flow:

```text
RTL Code
   ↓
RTL Simulation
   ↓
Synthesis
   ↓
Gate-Level Netlist
   ↓
Gate-Level Simulation
```

---

## 3. RTL Simulation and GLS

### RTL Simulation

RTL simulation checks the functionality of the original Verilog description before synthesis.

For example, RTL commonly uses:

* `always` blocks
* `if-else`
* `case`
* continuous assignments
* blocking assignments
* non-blocking assignments

### Gate-Level Simulation

GLS simulates the hardware generated after synthesis. Instead of high-level RTL statements, it uses the resulting gate-level netlist and cell models.

Therefore:

```text
RTL Simulation → Checks intended Verilog behavior
GLS            → Checks synthesized hardware behavior
```

---

## 4. Why RTL and GLS Can Differ

In some cases, RTL simulation and the synthesized netlist may not produce identical results.

Common causes include:

* Incorrect sensitivity lists
* Improper assignment ordering
* Latch inference
* Incorrect use of blocking assignments
* Incorrect use of non-blocking assignments
* Initialization differences

These issues can cause a design to look correct during RTL simulation but behave differently after synthesis.

---

## 5. Sensitivity Lists in Combinational Logic

A combinational `always` block must respond whenever any input used inside the block changes.

Consider:

```verilog
always @(sel) begin
    if (sel)
        y = b;
    else
        y = a;
end
```

Here, only `sel` is present in the sensitivity list.

If `a` or `b` changes while `sel` remains unchanged, the block may not execute during simulation.

A better approach is:

```verilog
always @(*) begin
    if (sel)
        y = b;
    else
        y = a;
end
```

`always @(*)` automatically includes the signals required for combinational evaluation.

---

## 6. Demonstrating the Sensitivity-List Problem

Example design:

```verilog
module mux_example(
    input a,
    input b,
    input sel,
    output reg y
);

always @(sel) begin
    if (sel)
        y = b;
    else
        y = a;
end

endmodule
```

If:

```text
sel = 0
a changes from 0 to 1
```

the output may not update because the `always` block is only sensitive to `sel`.

This demonstrates why incomplete sensitivity lists can create incorrect RTL simulation behavior.

---

## 7. Synthesizing the Design with Yosys

Yosys converts RTL code into a synthesized hardware representation.

Example commands:

```text
yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_example.v
synth -top mux_example
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr mux_example_netlist.v
exit
```

The generated netlist represents the actual logic structure created from the RTL design.

---

## 8. Performing Gate-Level Simulation

After synthesis, the netlist can be compiled along with the standard-cell models and testbench.

Example:

```text
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
mux_example_netlist.v \
tb_mux_example.v \
-o gls.out
```

Run the simulation:

```text
./gls.out
```

The generated waveform can then be opened using GTKWave:

```text
gtkwave tb_mux_example.vcd
```

---

## 9. Blocking Assignments

A blocking assignment uses the `=` operator.

Example:

```verilog
always @(*) begin
    x = a | b;
    y = x & c;
end
```

Statements execute sequentially during simulation.

Therefore:

```text
First:  x = a | b
Then:   y = x & c
```

Blocking assignments are generally preferred for combinational logic.

---

## 10. Assignment Order Matters

Consider this code:

```verilog
always @(*) begin
    y = x & c;
    x = a | b;
end
```

Here, `y` is calculated before `x` receives its new value.

This can produce unexpected simulation behavior.

A better implementation is:

```verilog
always @(*) begin
    x = a | b;
    y = x & c;
end
```

The order now follows the intended data flow.

---

## 11. Non-Blocking Assignments

Non-blocking assignments use the `<=` operator.

Example:

```verilog
always @(posedge clk) begin
    q <= d;
end
```

The update is scheduled rather than occurring immediately in the same procedural step.

Non-blocking assignments are commonly used to model sequential circuits such as:

* Flip-flops
* Registers
* Counters
* Shift registers

---

## 12. Blocking vs Non-Blocking

| Feature         | Blocking `=`                | Non-Blocking `<=`                       |
| --------------- | --------------------------- | --------------------------------------- |
| Update behavior | Immediate procedural update | Scheduled update                        |
| Common usage    | Combinational logic         | Sequential logic                        |
| Typical block   | `always @(*)`               | `always @(posedge clk)`                 |
| Execution       | Sequential                  | Updates occur together after evaluation |

Quick guideline:

```text
Combinational Logic → always @(*) → =
Sequential Logic    → always @(posedge clk) → <=
```

---

## 13. Example of Sequential Behavior

Consider two registers connected in series:

```verilog
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;
end
```

At each clock edge:

* `q1` receives the current sampled value of `d`
* `q2` receives the previous value stored in `q1`

This correctly represents two sequentially connected flip-flops.

```text
d → [FF1] → q1 → [FF2] → q2
```

---

## 14. Verification and Timing Flow

A complete verification flow can be represented as:

```text
Write RTL
    ↓
RTL Simulation
    ↓
Verify Functionality
    ↓
Synthesis
    ↓
Generate Netlist
    ↓
Gate-Level Simulation
    ↓
Compare Results
    ↓
Timing Analysis
```

For more detailed timing verification, delay information can be included using **SDF (Standard Delay Format)**.

SDF-based simulation can help analyze:

* Cell delays
* Interconnect delays
* Propagation delays
* Setup timing
* Hold timing

---

## 15. Learning Outcome and Conclusion

After completing Day 4, I understood:

* The purpose of Gate-Level Simulation
* The difference between RTL simulation and GLS
* How synthesis creates a gate-level netlist
* Why RTL and synthesized behavior can sometimes differ
* Problems caused by incomplete sensitivity lists
* Why `always @(*)` is useful for combinational logic
* The difference between `=` and `<=`
* Why blocking assignments are generally used for combinational circuits
* Why non-blocking assignments are preferred for clocked circuits
* How assignment ordering can affect simulation
* How Yosys is used for synthesis
* How Icarus Verilog and GTKWave are used for verification
* The basic role of SDF in timing simulation
