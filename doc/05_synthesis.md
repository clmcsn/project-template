## Intro

Synthesis is the first process  of the *physical implementation.* (Turning your RTL into an actual working circuit in silicon)
It converts (or compiles) an RTL design into basic logic cells. These cells are taken from a library called PDK. Example of cells are:

- Inverters, Buffers, Clock cells
- NAND, NOR, AND, OR cells
- MUX, IMUX cells
- D-type Latches (set, reset) and clock gate cells
- D-type Flops and Scan D-Type Flops (set, reset, both, enable)
- Half-Adder, Full-Adder cells

**Technology**

Each PDK is mainly defined by the equivalent gate length (e.g. 4nm), the model of the transistors used in the process (e.g. finfet,GAAT,etc.), and the fabrication technology (SOI,FDSOI,etc.)

**Cells models and flavors**

For each cells, the library contains different model, each model has different flavors:

- Different model have different different geometries and different drive strengths.
Bigger cells can drive bigger loads (e.g. if a AND output goes to 5 other cells needs a different drive strenght compared to the case it goes to 30)
- Different flavors have different threshold voltage → LVT, RVT, HVT
L → LOW, R → REGULAR, H →HIGH
As a reference:
LVT: very fast, but very leaky
HVT: almost no leakage, but very slow

Cells model and flavor matters more on the P&R (place and route) stage of the physical implementation. At that stage it will be possible to take into account the load introduced by routing…

**Synthesis output**

The synthesis generates a flatten, gate-level verilog file made of cells present in the PDK.

**Human**
```verilog
out = sel ? A : B;
```

**Synthesis tool**
```verilog
NAND2I2 U4A (A, sel, A_p);
NAND2I2 U4B (B,sel_n,B_p);
NAND2I2 U4C (sel,sel,sel_n);
NAND2I2 U4D (A_p,B_p,out);
```

## Synthesis tools

There are mainly 2 big company providing synthesis tools:

- Cadence → Genus
- Synopsys → Design Compiler and Fusion Compiler

Fusion Compiler is one of the most advanced as it does something called Physical Synthesis (e.g. synthesis with some physical predictions).

There exist also open source synthesis tools:

- Yosis

Each vendor and tool use its own `.tcl` language, but the flow is mostly the same across tool/vendor. 

### Cadence Genus

License path:

```bash
source /esat/micas-data/data/design/scripts/genus_21.17.rc
```

Below an example of a synthesis script for Genus.

Note that there are tons of command and option to force the tool optimize some things compare to others. For that ones need an accurate knowledge of the tool, the PDK and how well the tool behaves with a certain PDK.

```bash
include load_etc.tcl

set DESIGN <your_top_level_entity_design>

#EFF should stand for EFFORT, higher the effort more time synthesis takes.
#If you are synthesizing the new NVIDIA H100 GPU you might want to put effort only on things really important, 
#       otherwise synthesis can take ages...
set POW_EFF high
set SYN_EFF high
set MAP_EFF high

set_db library <path_to_your_pdk/pdk45nm.lib>
#set_db script_search_path <path_with_other_scripts>
set_db find_takes_multiple_names true

set_db wireload_mode top
set_db information_level 7
set_db lp_power_unit uW 

set_db interconnect_mode ple 
set_db hdl_track_filename_row_col true 
set_db hdl_max_loop_limit 4096

set search_path <main_path_of_rtl>
set_db init_hdl_search_path <other_directories>
read_hdl -language sv \
    -define { \
			<list_of_defines> \
    } \
    [list \
			<list_of_files> \
    ]

elaborate $DESIGN
check_design -unresolved
read_sdc <your_sdc_file>

report timing -lint

set_db lp_power_analysis_effort $POW_EFF

synthesize -to_generic -eff $SYN_EFF

timestat GENERIC

synthesize -to_mapped -eff $MAP_EFF 

check_design -all

report qor

write_hdl -mapped > <output_path>/netlist.v

puts "Final Runtime & Memory."
timestat FINAL
puts "============================"
puts "Synthesis Finished ........."
puts "============================"

report timing > <report_path>/timing.log
report area   > <report_path>/area.log
report power  > <report_path>/power.log

quit
```

The synthesis tool needs constraints (sdc). Constraints tell the tool the frequency you want to operate, power and area budgets.

Below an example:

```bash
## SDC File Constraints ##
set sdc_version 1.5
set_load_unit -picofarads 1

# Define the circuit clock (clk_i must be the name of the clock in the top level in your design)
create_clock -name "CLK_IN" -add -period <set_expected_clock_period> [dc::get_port {clk_i}]

# Add some non-idealities to the clock
# These are needed to take into account jitter and skew in the worst case for both setup and hold violations

#set_clock_transition -rise 0.0000001 [get_clocks "CLK_IN"]
#set_clock_transition -fall 0.0000001 [get_clocks "CLK_IN"]
set_clock_uncertainty 0.000001 [dc::get_port {clk_i}]

# Ignore timing for reset (or any specific signal you want to unconstraint)
set_false_path -from [dc::get_port rst_n_i]

## INPUTS
# constraints apply only from FF to FF. But what about INPUTS and OUTPUT signals? You need to specify by hand...

set_input_delay -clock CLK_IN 0.000001        [all_inputs]
#set_input_transition -min -rise 0.00000001   [all_inputs]
#set_input_transition -max -rise 0.0000001    [all_inputs]
#set_input_transition -min -fall 0.00000001   [all_inputs]
#set_input_transition -max -fall 0.0000001    [all_inputs]
#set_output_delay -clock CLK_IN  0.000001     [all_outputs]
 
## OUTPUTS
set_load 0.00001 [all_outputs]
#set_load -max 0.32 [all_outputs]
```

## RTL fatal mistakes for synthesis

HDL languages are ways to describe your circuit to CAD (EDA) tools.
Technology development has reached a point where tools interpret and compile pretty well human written RTL.

Still, there are good practices that avoid human/tool misunderstandings and avoid problems.

**Note!** RTL is readable, the synthesis produced netlist not so much readable. 
What happens if you: simulate your RTL, it works, simulate the netlist and it doesn’t work? Every designer’s nightmare… 

### Asynchronous reset

You should never drive an async reset from logic: a tiny glitch, portions of your design will stop working 

### Inferred latches

RTL design means Register Transfer Level design. The reason is that the most important elements in a design are ***registers**.* You should be very very cheap in using them more than anything.

What you really don’t want is the sythesis tool adding registers (or latches) to your design. To avoid this you **must** have `else`  or default cases in all your if statements.

```verilog
//NOT GOOD
always @(*) begin
	if (sel==2b01)
		out = A;
	else if (sel=2b00)
		out = B;
end
```

```verilog
//BEST
always @(*) begin
	if (sel==2b01)
		out = A;
	else if (sel=2b00)
		out = B;
	else
		out = '0;
end
```

```verilog
//ALSO GOOD
always @(*) begin
	out = '0;
	if (sel==2b01)
		out = A;
	else if (sel=2b00)
		out = B;
end
```

### Register reset values

Always set a reset value for registers
