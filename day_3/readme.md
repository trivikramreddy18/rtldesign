
Day 3 – Combinational and Sequential RTL Optimization
1. Objective
Day 3 focused on understanding how Yosys identifies and removes unnecessary hardware while preserving the required functionality.

The main areas covered were:

Combinational logic optimization
Constant propagation
Boolean simplification
Sequential logic optimization
Flip-flop optimization
Unused state removal
State optimization
Retiming
Sequential logic cloning
SKY130 technology mapping
2. RTL Optimization
RTL optimization improves a hardware description without changing its intended functionality.

The main objectives are:

Remove redundant logic
Simplify Boolean expressions
Eliminate unnecessary registers
Reduce hardware complexity
Improve area, power and timing efficiency
Optimization can be broadly divided into:

Combinational optimization – simplifies logic based on current inputs.
Sequential optimization – optimizes flip-flops, registers and state-dependent logic.
3. Yosys Optimization Setup
The RTL design and SKY130 standard-cell library are loaded into Yosys before optimization.

Start Yosys
yosys
Read the Liberty Library
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
Purpose: Loads the standard-cell characterization information required for technology mapping.

Read Verilog
read_verilog <design_name>.v
Purpose: Loads the RTL design into Yosys.

Select Top Module
synth -top <module_name>
Purpose: Performs synthesis using the specified module as the top-level design.

4. Combinational Optimization
Combinational optimization simplifies logic without changing its input-output behavior.

Constant Propagation
Constant propagation identifies fixed values and uses them to simplify surrounding logic.

Example:

assign y = a ? b : 1'b0;
can be simplified to:

assign y = a & b;
Boolean Simplification
Boolean simplification converts logically equivalent expressions into simpler hardware.

Example:

assign y = a ? 1'b1 : b;
can be simplified to:

assign y = a | b;
Redundant Logic Removal
Logic that does not contribute to the required output can be removed during optimization.

5. Conditional Logic Optimization
Nested conditional expressions can contain logic that is unnecessary after simplification.

Example:

assign y = a ? (c ? 1'b1 : b) : 1'b0;
Equivalent simplified form:

assign y = a & (c | b);
Similarly:

assign y = a ? (b ? (c ? 1'b1 : 1'b0) : 1'b0) : 1'b0;
can be simplified to:

assign y = a & b & c;
Key Learning
The synthesis tool can recognize equivalent Boolean behavior and implement it using simpler logic.

6. Sequential Logic Optimization
Sequential optimization deals with circuits containing state-holding elements such as flip-flops and registers.

The synthesis tool analyzes:

Register values
Reset and set behavior
State transitions
Output dependency
Unused state
The objective is to remove or simplify sequential hardware without changing the required behavior.

Constant-Driven Flip-Flop
If a flip-flop is always assigned the same value and its state is not required, the storage element may be removed.

However, reset and set behavior must also be considered before removing a flip-flop.

7. Sequential Constant Propagation and State Removal
Sequential constant propagation determines whether register values become constant after considering clocked behavior.

Example
If a register is always driven to:

q = 1
the synthesis tool can identify that the register may not be required.

Unused State
If a register or state bit cannot affect any observable output, it may be removed.

This is called state or register optimization.

Important Principle
Logic that cannot affect an observable output can potentially be removed during synthesis.

8. Counter Optimization
Counter with Unused Bits
Consider a 3-bit counter:

count[2:0]
If only:

count[0]
is connected to the output, the upper bits may not be required for the observable behavior.

3-bit Counter
     ↓
Only count[0] is used
     ↓
Unused state identified
     ↓
Unnecessary state can be removed
Counter Where All Bits Are Required
If the output depends on:

count[2:0]
then all three state bits are required.

For example:

assign q = (count[2:0] == 3'b100);
Every counter bit is needed to perform the comparison.

Key Learning
The synthesis tool considers output dependency and observability before removing state elements.

9. Advanced Sequential Optimization
State Optimization
State optimization reduces unnecessary states or state-holding elements while maintaining the required behavior of the sequential circuit.

It can reduce the number of flip-flops and the logic required to represent the state.

Retiming
Retiming is a technique in which flip-flops are moved across combinational logic to improve timing while preserving the functional behavior of the design.

The main purpose is to balance the amount of logic between pipeline stages.

Sequential Logic Cloning
Sequential logic cloning creates additional copies of sequential logic when required to reduce fanout and improve timing or implementation efficiency.

The duplicated logic performs equivalent functionality but allows different parts of the design to use separate copies.

10. Observability
Observability determines whether an internal signal or state element can influence an output that matters to the design.

If a register or logic block cannot affect any required output, the synthesis tool can potentially remove it.

Internal Logic
      ↓
Can it affect an output?
      ↓
   No → May be removed
   Yes → Must be retained
This concept is important when understanding why some registers disappear after synthesis.

11. Important Yosys Optimization Commands
Command	Purpose
opt	Performs general optimization passes
opt_expr	Simplifies expressions and propagates constants
opt_clean -purge	Removes unused and unreferenced logic
dfflibmap	Maps generic flip-flops to SKY130 flip-flop cells
abc	Performs logic optimization and technology mapping
show	Displays the synthesized design as a schematic
Commands Used
opt
opt_expr
opt_clean -purge
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
12. SKY130 Technology Mapping
After RTL optimization, the design can be mapped to cells available in the SKY130 standard-cell library.

Flip-Flop Mapping
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
Maps generic flip-flops to suitable SKY130 library cells.

Combinational Mapping
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
Maps combinational logic to cells available in the selected SKY130 library.

View the Design
show
Displays the resulting synthesized structure as a schematic.

13. Experiments Performed
The following optimization cases were studied:

Combinational
opt_check.v – Constant propagation
opt_check2.v – Hardwired constant optimization
opt_check3.v – Nested conditional optimization
opt_check4.v – Multi-level Boolean optimization
Sequential
dff_const1.v – Constant D-input flip-flop
dff_const2.v – Flip-flop permanently tied to a constant
dff_const3.v – Multi-flop constant propagation
Counter Optimization
counter_opt.v – Counter with unused upper state bits
counter_opt2.v – Counter where all state bits are required
14. Learning Outcomes
Understood the purpose of RTL optimization.
Learned how constant propagation simplifies combinational logic.
Understood Boolean and conditional logic simplification.
Learned how redundant logic can be removed.
Understood sequential optimization of flip-flops and registers.
Learned how constant-driven sequential logic can be optimized.
Understood why reset behavior must be considered during optimization.
Learned how unused state can be identified using output observability.
Understood why some counter bits can be removed while others must be retained.
Learned the basic concepts of state optimization, retiming and sequential logic cloning.
Understood the purpose of opt, opt_expr and opt_clean -purge.
Learned how dfflibmap maps flip-flops to SKY130 cells.
Understood the role of ABC in technology mapping.
15. Conclusion
Day 3 demonstrated how synthesis tools can simplify both combinational and sequential RTL while preserving the required functionality.

The experiments showed how constant propagation, Boolean simplification, redundant logic removal, state optimization and output observability can reduce unnecessary hardware.

The advanced concepts of retiming and sequential logic cloning introduced how sequential structures can also be optimized for better timing and implementation efficiency.

Overall, the session provided a practical understanding of how Yosys analyzes RTL and maps optimized logic to the SKY130 standard-cell library.
