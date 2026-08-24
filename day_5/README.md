
# DAY 5 – RTL Coding Styles and Synthesis Optimization

## 1. Introduction

Day 5 focuses on how the way we write Verilog RTL can affect the hardware produced after synthesis. Small coding differences, such as leaving out an `else` statement or missing a `case` condition, can change a combinational circuit into a design containing storage elements.

In this session, I studied incomplete conditional statements, case structures, loop constructs, generated hardware, and compilation of multi-module designs.

---

## 2. Learning Objectives

The main topics covered during Day 5 are:

* Incomplete `if` statements
* Incomplete `case` statements
* Latch inference
* Complete combinational assignments
* Overlapping or wildcard case conditions
* Procedural `for` loops
* `for generate`
* Hierarchical Verilog designs
* Compilation of dependent modules
* RTL coding practices for better synthesis

---

## 3. Incomplete `if` Statement

In combinational logic, an output should normally be defined for every possible input situation.

Consider the following example.

### File: `incomp_if.v`

```verilog id="uy7ljr"
module incomp_if (
    input i0,
    input i1,
    input i2,
    output reg y
);

always @(*)
begin
    if(i0)
        y <= i1;
end

endmodule
```

When `i0` is high, `y` receives the value of `i1`.

However, when `i0` is low, the code does not specify a new value for `y`. Therefore, the synthesized circuit may need to remember the previous output value.

---

## 4. Understanding Latch Inference

A latch can be created unintentionally when a combinational `always` block does not assign an output in all possible execution paths.

The basic idea is:

```text
Condition not satisfied
        ↓
No new output assignment
        ↓
Previous output must be preserved
        ↓
Possible latch inference
```

This is why complete assignments are an important part of RTL design.

---

## 5. How to Avoid an Unwanted Latch

One solution is to use an `else` branch.

```verilog id="wzkxf6"
always @(*)
begin
    if(i0)
        y = i1;
    else
        y = i2;
end
```

Another useful technique is to assign a default value before applying conditions.

```verilog id="apn2jt"
always @(*)
begin
    y = i2;

    if(i0)
        y = i1;
end
```

With these approaches, `y` receives a value in every situation.

---

## 6. Incomplete `case` Statement

A similar issue can occur with a `case` statement.

### File: `incomp_case.v`

```verilog id="zoaey1"
module incomp_case (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y
);

always @(*)
begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
    endcase
end

endmodule
```

The select signal has four possible binary combinations:

```text
00
01
10
11
```

Only two of them are handled. For the remaining values, `y` does not receive a new assignment, which can result in latch inference.

---

## 7. Using `default` in a Case Statement

A `default` branch can handle all combinations that are not explicitly written.

### Example

```verilog id="qlyxen"
always @(*)
begin
    case(sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        default : y = i2;
    endcase
end
```

This makes the design more complete because every possible value of `sel` produces an output assignment.

---

## 8. Complete Case and Multiplexer Logic

The `partial_case` design assigns a different input to the output depending on the select value.

### File: `partial_case_assign.v`

```verilog id="rcgn2c"
module partial_case (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

always @(*)
begin
    case (sel)
        2'b00 : y = i0;
        2'b01 : y = i1;
        2'b10 : y = i2;
        2'b11 : y = i3;
    endcase
end

endmodule
```

This behavior is equivalent to a **4-to-1 multiplexer**, where `sel` chooses one of the four input signals.

---

## 9. Overlapping Case Conditions

Care should be taken when wildcard patterns are used in case-related constructs.

For example, a pattern such as:

```text
1?
```

can match more than one binary combination:

```text
10
11
```

If another condition also specifically handles `10`, the conditions overlap.

Overlapping conditions can make the intended behavior less clear and may introduce priority effects or unexpected synthesis results. RTL conditions should therefore be written so that the selected behavior is clear and predictable.

---

## 10. Procedural `for` Loop

A normal `for` loop is used inside procedural blocks such as `always` or `initial`.

It helps perform repetitive operations without manually writing the same statements many times.

### Example: 1-to-8 Demultiplexer Style Logic

```verilog id="k4k33q"
module demux_for (
    input i,
    input [2:0] sel,
    output reg [7:0] y
);

integer k;

always @(*)
begin
    y = 8'b00000000;

    for(k = 0; k < 8; k = k + 1)
    begin
        if(sel == k)
            y[k] = i;
    end
end

endmodule
```

The loop checks each output position and activates the selected location.

The initial assignment:

```verilog id="ixzh47"
y = 8'b00000000;
```

ensures that all output bits receive a defined value before the conditional assignment.

---

## 11. What is `for generate`?

`for generate` is different from a procedural `for` loop.

It is used to describe repeated **hardware structures**. Instead of repeatedly executing statements inside an `always` block, the generate construct creates multiple pieces of hardware during design elaboration.

It is useful for:

* Repeated module instantiation
* Parameterized hardware
* Bus structures
* Multi-bit arithmetic circuits
* Arrays of identical logic blocks

---

## 12. Example: Ripple Carry Adder Using Generate

A multi-bit adder can be created by connecting several 1-bit full adders.

Instead of manually instantiating every full adder, a generate loop can repeat the hardware structure.

```verilog id="5ls0y0"
genvar i;

generate
    for(i = 1; i < 8; i = i + 1)
    begin : fa_gen

        fa u_fa (
            .a(num1[i]),
            .b(num2[i]),
            .cin(carry[i-1]),
            .sum(sum[i]),
            .cout(carry[i])
        );

    end
endgenerate
```

Each iteration represents another hardware instance.

The generated structure can be viewed conceptually as:

```text
FA0 → FA1 → FA2 → FA3 → FA4 → FA5 → FA6 → FA7
```

---

## 13. `for` Loop vs `for generate`

| Feature       | `for` Loop                   | `for generate`             |
| ------------- | ---------------------------- | -------------------------- |
| Style         | Procedural                   | Structural                 |
| Location      | Inside `always` or `initial` | Outside procedural blocks  |
| Main purpose  | Repeat operations            | Repeat hardware structures |
| Loop variable | Usually `integer`            | Usually `genvar`           |
| Typical use   | Array/vector operations      | Multiple module instances  |
| Example       | Demultiplexer logic          | Ripple Carry Adder         |

### Simple Way to Remember

```text
for
↓
Repeat operations

for generate
↓
Replicate hardware
```

---

## 14. Hierarchical Design and Compilation

Large Verilog designs are often built using multiple modules.

For example:

```text
fa.v
   ↓
rca.v
   ↓
tb_rca.v
```

Here, the ripple carry adder depends on the full adder module. Therefore, all required source files must be available during compilation.

An example simulation command using Icarus Verilog is:

```bash id="8b0spw"
iverilog fa.v rca.v tb_rca.v
```

After compilation:

```bash id="0cojbs"
./a.out
```

The waveform can then be viewed using:

```bash id="m0ce0q"
gtkwave tb_rca.vcd
```

If a required module file is missing during compilation, the simulator cannot resolve the instantiated module correctly.

---

## 15. Important RTL Coding Rules and Conclusion

The major learning points from Day 5 are:

1. Every output in combinational logic should receive an assignment for all possible conditions.
2. An incomplete `if` can cause a latch to be inferred.
3. Missing branches in a `case` statement can also create unwanted storage behavior.
4. A `default` branch is useful for covering remaining case conditions.
5. Default assignments before conditional statements can help maintain purely combinational logic.
6. Overlapping conditions should be avoided unless their priority is intentionally designed.
7. A procedural `for` loop is used for repeated operations inside procedural blocks.
8. `for generate` is used to build repeated hardware structures.
9. Hierarchical designs require all dependent Verilog modules to be included during compilation.
10. Proper RTL coding helps produce predictable synthesized hardware.

### Final Summary

Day 5 demonstrated that Verilog is not only about obtaining the correct simulation output. The way RTL is written directly affects the hardware created by synthesis.

By writing complete conditions, avoiding unnecessary latches, selecting the correct loop construct, and organizing dependent modules properly, a cleaner and more reliable digital design can be developed.
