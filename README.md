# FPGA_workshop
## Day 1

## Day 2

## VPR starting froma a blif file

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

    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif --route_chan_width 100 --sdc_file 

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



