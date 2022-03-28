# FPGA_workshop
## Day 1: FPGA Intro and Vivado 

### Vivado counter

The firts exercise consists of creating a Vivado project using a counter coded in verilog.

The steps followed are detailed here:

1. Create the project adding the counter source and the testbench for simulation.

![01_vivado_project_created](/images/day1/01_vivado_project_created.JPG)

2. Run the simulation:

![02_vivado_counter1_sim](/images/day1/02_vivado_counter1_sim.JPG)

3. Elaborate the design

![03_vivado_counter_elaborated](/images/day1/03_vivado_counter_elaborated.JPG)

4. Add the pinout constraints:

![04_vivado_counter_iopins_constrs](/images/day1/04_vivado_counter_iopins_constrs.JPG)

5. Synthesize the design:

![05_vivado_counter_synthesized](/images/day1/05_vivado_counter_synthesized.JPG)

6. Add the timing constraints:

![06_vivado_counter_clock_constrs](/images/day1/06_vivado_counter_clock_constrs.JPG)

7. Synthesis again, the schematic can be seen:

![07_vivado_counter_sch_timing_summary](/images/day1/07_vivado_counter_sch_timing_summary.JPG)

8. Run implementation

![08_vivado_counter_implemented](/images/day1/08_vivado_counter_implemented.JPG)

9. We can check the report utilization to see the are used:

![08_vivado_counter_report_utilization](/images/day1/08_vivado_counter_report_utilization.JPG)

10. Timing analysis report:

![09_vivado_counter_timing](/images/day1/09_vivado_counter_timing.JPG)

![10_vivado_counter_timing_summary](/images/day1/10_vivado_counter_timing_summary.JPG)

11. Power analysis:

![11_vivado_counter_power_summary](/images/day1/11_vivado_counter_power_summary.JPG)

12. Resources summary:

![12_vivado_counter_resources_summary](/images/day1/12_vivado_counter_resources_summary.JPG)

13. Add the VIO to the design:

![13_vivado_counter_vio](/images/day1/13_vivado_counter_vio.JPG)

14. Generate the bitstream:

![14_vivado_counter_vio_implemented](/images/day1/14_vivado_counter_vio_implemented.JPG)


## Day 2: OpenFPGA - VPR - VTR

## VPR starting from a blif file

### blif format (previously generated, for instance for ABC tool):
.model = model description  
.inputs  
.outputs  
.latch = Flip flops  
.names = LUTs  

### EArch xml format (provided by VTR)
Description of the architecture: Heterogeneour architecture with Carry Chains  
6 input LUTs  
Description how it is modeled  
    <architecture>  

      <models>
        <model name="model_name1">
          <input_ports>
            <port name="a" combinational_sink_ports="out"/>
            <port name="b" combinational_sink_ports="out"/>
          </input_ports>
          <output_ports>
            <port name="out"/>
          </output_ports>
         </model>
        
        ...
        
      </models>
      <tiles>
        blocks FPGA architecture is going to have: 
        io, clb, multiplication tile, memory tile
      </tiles>
      <layout>
        FPGA grid layout
      </layout>
      <device>
        How the transistors are defined
      </device>
      <switchlist>
        how the switch matrix is described
      </switchlist>
      <segmentlist>
        routing info
      </segmentlist>
      <directlist>
        adder, carry
      </directlist>
      <complexblocklist>
        group of pb_type = type of the functional blocks
        io
        clock
      </complexblocklist>
    </architecture>
        
### VTR Commands:

    mkdir vpr_tseng
    cd vpr_tseng
    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif --route_chan_width 100 --disp on

the --disp on option opens the GUI:  

![VPR tseng GUI](/images/day2/01_VPR_GUI.JPG)

From the GUI different options are available to see nets, Proceed with the P&R, etc:

![VPR tseng GUI 2](/images/day2/02_VPR_GUI.JPG)

We can see the Congestion where Blue means less congested area.

![VPR tseng GUI Congestion](/images/day2/03_VPR_GUI_Congestion.JPG)

We can also see the critical paths:

![VPR tseng GUI Critical path](/images/day2/04_VPR_GUI_critical_path.JPG)

When done, the console shows the log of the process:
![VPR tseng console](/images/day2/05_VPR_console_done.JPG)


We can specify a point util the tool executes. For instance analysis will proceed until analysis:

    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif --route_chan_width 100 --analysis --disp on

![VPR tseng GUI analysis](/images/day2/06_VPR_analysis.JPG)

### Outputs  

The .net generated file is an xml with the description of the post-packed circuit. User netlist in terms of complex logic.
![VPR tseng net](/images/day2/07_VPR_tseng_net_out.JPG)

The .place generated file is the output of the placement.
![VPR tseng place](/images/day2/08_VPR_tseng_place_out.JPG)

The .route generated file is the output of the routing stage.
![VPR tseng route](/images/day2/09_VPR_tseng_route_out.JPG)

And the log file contains the terminal output.


### Reports

Different reports are generated with the information about timing, pins.  
![VPR tseng reports](/images/day2/10_VPR_reports.JPG)

By checking the report_timing_setup and hold we can see that the slack is negative (violated) as no clock constraints were created:  
![VPR tseng reports](/images/day2/11_VPR_report_timing_slack_violated.JPG)


To solve the negative slack issues we need to add the required constraints.  
We create the tseng.sdc constraints file: 
![VPR tseng sdc](/images/day2/12_VPR_sdc_constraints.JPG)

and append it to the execution of the VTR command:

    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif --route_chan_width 100 --sdc_file tseng.sdc

Ater completion, the report now shows the slack time is met:  
![VPR tseng slack_met](/images/day2/13_VPR_slack_met.JPG)


## VTR Flow

At this point we can start from an HDL design through all the steps.  

2 options to run the tool:
1. Manually using 
- ODIN II for Synthesis
- ABC for mapping
- VPR for implementation 
2. Automatic running the VTR flow

Running the counter example:  

    $VTR_ROOT/vtr_flow/scripts/run_vtr_flow.py counter.v $VTR_ROOT/vtr_flow/arch/timing/EArch.xml --route_chan_width 100

We can see the generated blif files  
![14_VTR_blif_files](/images/day2/14_VTR_blif_files.JPG)

and use the VPR specific analysis to take the pre-vpr.blif file and see the GUI until that point:  

    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml counter --circuit_file temp/counter.pre-vpr.blif  --route_chan_width 100 --analysis --disp on

![15_VTR_blif_gui](/images/day2/15_VTR_blif_gui.JPG)

We can generate the post synthesis netlist:

    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml counter --circuit_file temp/counter.pre-vpr.blif --gen_post_synthesis_netlist on
    
![16_up_counter_post_synth](/images/day2/16_up_counter_post_synth.JPG)

With this post synthesis file a Vivado project including: up_counter_post_synthesis.v, primitives.v and counter_tb.v is created and the simulation of the circuit can be done:  
![17_up_counter_vivado_sim](/images/day2/17_up_counter_vivado_sim.JPG)


### Timing Area VTR flow
We run the python script 
    $VTR_ROOT/vtr_flow/scripts/run_vtr_flow.py\
    counter.v $VTR_ROOT/vtr_flow/arch/timing/EArch.xml\
    -temp_dir temp/ \
    --route_chan_width 100
    
and checking again the timing reports,  setup for instance, we see the slack violation as no constriants were defined.

We create the sdc constriants file:  
![18_sdc_file](/images/day2/18_sdc_file.JPG)

and run vtr using the generated blif file and the sdc constraint file:
    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml temp/counter.pre-vpr.blif --route_chan_width 100 --sdc_file counter.sdc

It fails as the character ^ in up_counter^clk is not recognized. So we need to replace the ^ by _ in both the blif and sdc files:
![19_replacing_clock_name](/images/day2/19_replacing_clock_name.JPG)

After executing again the vpr command, now it completes without errors:
![20_VPR_ok_after_replacing_clock_name](/images/day2/20_VPR_ok_after_replacing_clock_name.JPG)

and the timing reports show that slack is met now.
![21_timing_slack_met](/images/day2/21_timing_slack_met.JPG)

In addition we can see from the vpr_stdout.log file useful information such as the area or resources required for this specific implementation:

![22_resources](/images/day2/22_resources.JPG)


### Power Analysis

VTR power stimation can be done using the python script with the option -power and also the option -cmos_tech <cmos_tech_property_file>

The CMOS technology property file is an XML containing information about transistor, lengths, leakage currents, etc. And this information is used to estimate the power consumption of the circuit.

The command to perform the power stimation analysis is:

    $VTR_ROOT/vtr_flow/scripts/run_vtr_flow.py counter.v $VTR_ROOT/vtr_flow/arch/timing/EArch.xml -power -cmos_tech $VTR_ROOT/vtr_flow/tech/PTM_45nm/45nm.xml -temp_dir temp/ --route_chan_width 100
    
It generates the file counter.power with the power estimation details:

![23_power](/images/day2/23_power.JPG)

Power Breakdown:

![24_power_breakdown](/images/day2/24_power_breakdown.JPG)



## Day 3: RISC-V on Vivado

Taking the reference files for the RISCV processor a Vivado project is created and the Behabioral Simulation is completed. A small change was required in the test to addapt the out port connection in the testbench:

![01_riscv_adapting_test_simulation](/images/day3/01_riscv_adapting_test_simulation.JPG)


The simulation confirms the implemented circuit (sum of 1 to 9 = 45):

![02_riscv_vivado_simulation](/images/day3/02_riscv_vivado_simulation.JPG)


### Adding an ILA

We remove the output out from the ports and declare it as a reg to be connected to the ILA.

Add the ILA from the IP catalog and define the sizes of the two probes:

![03_riscv_vivado_ila](/images/day3/03_riscv_vivado_ila.JPG)


Instantiate the ILA in the code:

![04_riscv_vivado_ila_instantiation](/images/day3/04_riscv_vivado_ila_instantiation.JPG)


After elaboration we assign the clock and reset pins:

![05_riscv_vivado_IOs](/images/day3/05_riscv_vivado_IOs.JPG)


And after synthesis, the constraints wizard is used to define the timing constraints:

![06_riscv_vivado_timing_constrs](/images/day3/06_riscv_vivado_timing_constrs.JPG)


Finally the implementation and bitstream generation is done. 

![07_riscv_vivado_implemented](/images/day3/07_riscv_vivado_implemented.JPG)

We can verify that the timing was met:

![08_riscv_vivado_timing](/images/day3/08_riscv_vivado_timing.JPG)

If we had the FPGA board connected we could see the signals added to the ILA responding to the corresponding inputs.

## Day 4: Intro to SOFA FPGA
Skywater Opensource FPGAs. Base repository link:  
https://github.com/lnis-uofu/SOFA

Open source FPGA IPs which use Skywater 130 nm PDK and OpenFPGA framework.

SOFA documentation link:  
https://skywater-openfpga.readthedocs.io/en/latest/

Open-source embedeed FPGA IP library. Architecture description to produce ready layouts.

### Steps to run SOFA using the counter.v example:

Clone the repository: https://github.com/lnis-uofu/SOFA.git  

It can be run using the Makefile with some config files that need to be specified for a given circuit. 

First we need to adapt the file: SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/config/task_simulation.conf  

This uses the yosys_vpr flow, the generate_testbench.openfpga shell scripts to run and the vpr_arch.xml file. It also uses the 12x12 device layout and a route_chan_width of 60.  

The place were we can include our design file is under [BENCHMARKS] 

We copy the files under the new folder counter_new and specify the counter.v file  
and under [SYNTHESIS_PARAM] we specify the top level entity: up_counter  

![01_task_simulation_conf](/images/day4/01_task_simulation_conf.JPG)


On the folder SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/ we find the file: generate_testbench.openfpga. This is a shell script that calls the VPR tool.  

The architecture file is under SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/arch folder. It is vpr_arch.xml. It containt LUTs, carry, adder, FF, etc.

To run it we need to go to the original folder: SOFA/FPGA1212_QLSOFA_HD_PNR and run the Makefile by executing the command:
    make runOpenFPGA  
    
![02_make_runOpenFPGA](/images/day4/02_make_runOpenFPGA.JPG)

Once completed, the results can be found in the task directory: ~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH

Here we see different blif files, reports, logs. 
![03_output_files](/images/day4/03_output_files.JPG)


The logs of interest are:

openfpgashell.log and vpr_stdout.log

In the vpr_stdout.log we can see the command used:  
![04_vpr_command](/images/day4/04_vpr_command.JPG)

And the Circuit statistics and Logic Elements with the information of resources used:  

![05_vpr_circuit_statistics](/images/day4/05_vpr_circuit_statistics.JPG)

![06_vpr_logic_elements](/images/day4/06_vpr_logic_elements.JPG)


#### Timing Analysis

Create an sdc file with the clock and delay information:  

![07_sdc_file](/images/day4/07_sdc_file.JPG)

And this sdc file needs to be included in the generate_testbench.openfpga file:  

![08_sdc_file_path](/images/day4/08_sdc_file_path.JPG)


With these changes we go back to the folder containing the Makefile and run again the command:  
    make runOpenFPGA  

And in the folder ~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH we can see the timing reports were the slack is met:  

![09_timing_reports](/images/day4/09_timing_reports.JPG)

#### Post implementation netlist

In the generate_testbench.openfpga file were we added the sdc file we include the option: --gen_post_synthesis_netlist on:  

![10_gen_post_implem](/images/day4/10_gen_post_implem.JPG)

Even though the option is called post_synthesis_netlist, it will generate a post implementation netlist.

Run again the make.  

It will generate the post synthesis file:  

![11_post_synth_v](/images/day4/11_post_synth_v.JPG)

With this file and a testbench we can create a Vivado project and run the simulation:  

![12_vivado_simulation](/images/day4/12_vivado_simulation.JPG)

#### Power analysis

In the SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/config/task_simulation.conf we define the 

    vpr_device_layout=auto
    vpr_route_chan_width=150
    
And in the generate_testbench.openfpga file we need to add:  

    --power --activity_file ~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH/up_counter_ace_out.act  
    --tech_properties $VTR_ROOT/vtr_flow/tech/PTM_45nm/45nm.xml
    
![13_power_options_vtr](/images/day4/13_power_options_vtr.JPG)

After running make again, the power report is located at ~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH/up_counter.power


## Day 5: Running the RISCV core on SOFA FPGA

This part is similar to the one on Day 4 but instead of the counter we use the RISCV core.
First we copy the source files for the riscv into SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/BENCHMARC/rvmyth/

![01_riscv_files](/images/day5/01_riscv_files.JPG)

The SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/config/task_simulation.conf file now contains:

    vpr_device_layout=auto
    vpr_route_chan_width=180

The default architecture is: vpr_arch.xml

And the path where the riscv code is located is provided:

also as the top level entity:

    bench0_top = core

Similarly, the SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/generate_testbench.openfpga file contains some changes:

Remove --route_chan_width ${OPENFPGA_VPR_ROUTE_CHAN_WIDTH} option

The arch/vpr_arch.xml also contains some changes: 4 lines commented.

We run the make on folder SOFA/FPGA1212_QLSOFA_HD_PNR/:

    make runOpenFPGA
    
![02_make_completed](/images/day5/02_make_completed.JPG)


The log file ~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH/openfpgashell.log confirms that the execution was Ok:  

![03_openfpgashell_log](/images/day5/03_openfpgashell_log.JPG)


The area report can be seen in the same directory, by checking the vsp_stdout.log

Next we add the sdc constraints file and run again the make command. We check the timing reports to confir that slack is met:

setup time:  

![06_timing_setup](/images/day5/06_timing_setup.JPG)


hold time:  

![07_timing_hold](/images/day5/07_timing_hold.JPG)


At this point we generate the post synthesis netlist by adding the --gen_post_synthesis_netlist option on the generate_testbench.openfpga file:  

![08_post_synth_opt](/images/day5/08_post_synth_opt.JPG)

With the post synthesys generated file plus the testbench and the primitives.v a Vivado project is created to simulate the circuit.

![09_vivado_sim](/images/day5/09_vivado_sim.JPG)

![10_vivado_sim](/images/day5/10_vivado_sim.JPG)


Finally the power report is generated by adding the --power options to the generate_testbench.openfpga:  

![11_power_opt](/images/day5/11_power_opt.JPG)

We need to check that the vpr_arch.xml description contains the power information:

![12_power_vpr_arch_xml](/images/day5/12_power_vpr_arch_xml.JPG)


After running make, the power report is available at: 

~/SOFA/FPGA1212_QLSOFA_HD_PNR/FPGA1212_QLSOFA_HD_task/latest/vpr_arch/up_counter/MIN_ROUTE_CHAN_WIDTH/core.power.
