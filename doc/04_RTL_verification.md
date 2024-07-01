There are several RTL simulators, open-source or proprietary.

We advise you to use QuestaSim as it can produce average switching activity files with inter-gate delays.

An open-source alternative you might be interested in is [Verilator](https://www.veripool.org/verilator/), you can install and play with it on the server.
For the installation, we suggest to either use a Docker container (podman), or lmod/lua or a conda environment. If you are interested we will be happy to introduce and explain what all these things are.

### QuestaSim

To use proprietary tools you have to always activate the license in your current terminal.

To do so:

```bash
user@lab_machine$ source /esat/micas-data/data/design/scripts/questasim_VERSION.rc
```

We suggest to try to use always one of the latest versions.

There are two ways of using EDA tools: with GUI or through command line.

- GUI: better if you are doing waveform debugging
- CMD: to do validation on a daily basis

Even if you are using the GUI is always good to have some basic scripts to start your simulation.

Example on how to write a .tcl file to start a simulation (works with 2021 version):

```bash
#it will be used for transcript file + work library
quietly set SIMNAME "rtl"

#if can trigger a silent simulation by calling your simulator with NO_GUI=1 in front
#default the simulation starts with the GUI
if { [info exist ::env(NO_GUI)] } {
  quietly set NO_GUI $::env(NO_GUI)
} else {
  quietly set NO_GUI 0
}

#set library directory
quietly set WLIB "./work/work_${SIMNAME}"
vlib ${WLIB}
vmap work_lib ${WLIB}

#set hardware directory
quietly set HDL_PATH "path_to_your_hdl"

#compiling HDL
vlog -sv -work ${WLIB} ${HDL_PATH}/your_hdl_files ...'
#if you want to define global variables you can do it here as:
# vlog -sv -work ${WLIB} +define+YOUR_VARIABLE=VALUE1+YOUR_OTHER_VARIABLE=VALUE2

if {${NO_GUI} == 0} {
  # add signals to wave window
  vopt -work ${WLIB} +acc your_tb_file -o dbg 
  quietly set OBJ "dbg"
} else {
  vopt -work ${WLIB} your_tb_file -o nodbg 
  quietly set OBJ "nodbg"
}

vsim \
  -wlf work/${SIMNAME}.wlf \
#  -voptargs="+acc" \ #should not be needed
  -msgmode both -displaymsgmode both \
  -L work_lib  \
  -work ${WLIB}\
  ${OBJ}
```

### CocoTb

CocoTb is an interface between testing structures in python and your simulation program (QuestaSim, Verilator, etc.)

This is a very powerful tool as it allows, in the same python script/environment, to:

- Trigger test routines with pytest
- Generate test inputs
- Run your HDL simulation
- Check the correctness of the design

We are still working at an example to play with with cocotb…
