# FPGA_workshop
## Day 1

## Day 2

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
        
VTR Commands:

    mkdir vpr_tseng
    cd vpr_tseng
    $VTR_ROOT/vpr/vpr $VTR_ROOT/vtr_flow/arch/timing/EArch.xml $VTR_ROOT/vtr_flow/benchmarks/blif/tseng.blif --route_chan_width 100 --disp on

the --disp on option opens the GUI:  

![VPR tseng GUI](/images/day2/01_VPR_GUI.JPG)


