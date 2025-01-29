---
title: "HTAX Verification - SystemVerilog/UVM"
description: "A description on using verification techniques and UVM to validate a basic transmission/receiving module"
date: "2024-11-10"
author: "Vishnu Ramesh"
math: true
---

Hardware verification is a integral part of the chip design process. Failures to catch defects can cost millions of dollars for companies. Software bugs can often be patched with a quick update, but most hardware faults can only be fixed by recalling the chip and replacing it.

Because of this, Verification Engineers are often employed by companies for the sole purpose of debugging and catching errors in the design before final chip tape out.

A typical verification cycle goes through the following steps:

1. **Functional Specification**: Describes what functions the design performs, as well as any interfaces for module specification. This is typically implemented in HDL by design engineers.
2. **Verification Plan**: List functions to be verified, specific tests/methods, measurements to indicate completed verification, and estimate of verification cost
3. **Verification Environment**: Build a set of software code that allows engineers to easily find and debug errors in HDL code.
4. **Run Regression**: Run tests defined in the verification plan to uncover *hard to find* bugs and ensure that coverage goals have been met
5. **Coverage Analysis**: Check for functional and code coverage, as well as assertions

These steps are just an overview to the verification process, but they entail the basics of how engineers find and debug errors in chip design.

The rest of the article describes the verification of the [HyperTransport Advanced X-Bar](https://drive.google.com/file/d/1btEmv-M3NiU0EfXjCkvxFGIwOvoqm19v/view) module, a communication protocol that can be used for transmitting/receiving signals between chips. This includes the following:
* Verification plan (using Cadence vManager)
* Verification environment including: UVM Interface, sequences, monitor, agent, and scoreboard
* SystemVerilog Assertions and coverage checkers for regression testing

## Verification Plan

The verification plan specifies *what* is being verified, and *how* the verification is done. All engineers
are involved in this process, both from the design and verification team. The most important part of the plan is *following the specifications*, so that issues between the two teams can be resolved. In the case of the HTAX communication protocol, planning was split into three categories: testcases, asssertions, and functional coverage

Testcases are very specific scenarios that test the design under unique circumstances. These are typically the backbone of verification and allows for coverage goals to be met, and assertions to be verified. Each testcase typically has a brief name and description. Examples include the following:
* Transmission Port-to-Port test (from port X to port Y)
* Short data packet length test (length between 3 to 10 bytes)
* Double transmit request utilizing minimum latency (Single virtual channel request)

Assertions are statements that check the state of the design at various points during simulation. For example, if there is a register that is coded to always be zero, an assertion can be made that will raise an error if the register contains a non-zero value at any point during the simulation. Hardware assertions are generally split into two types of assertions: combinational and temporal. Combinational assertions check for functional correctness at all times during the simulation. Temporal assertions check to verify sequential behavior over time. Examples include the following:
* Transmission outport request signal is always one-hot encoded. (combinational)
* Transmission start signal only happens after virtual channel granted (temporal)
* Receiver start signal goes high immediately after end signal in back-to-back transmissions (temporal)

Functional coverage indicates how well the design has been checked. It is a measure of every possible scenario that is designed in the specification. Examples include the following:
* In ports used (ports 0,1,2,3)
* Packet data length (All possible data lengths between 3 and 60)
* Virtual channels (all 4 virtual channels, also check coverage between channels used at same time)

Encorporating all of these elements into the verification plan while taking into account the design specification leads to a *golden document* that can be used as a guide on how to proceed forward with verification.

## UVM Overview

Before talking about the verification environment, it is important to know how the Universal Verification Methodology (UVM) works. UVM is a set of classes and methods that build upon the original SystemVerilog language. It allows for engineers to easily build Verification Components (VCs) and uses Object-Oriented Programming for enhanced code reusability. A basic overview is given below.

First, there is a UVM test class, the topmost class that configures the environment, interface, and Design Under Test (DUT). The DUT is simply an instantiation of the design being verified, and the interface is a intermediary connection between the DUT and environment.

The environment is composed of one or more *agents*, each with its own components. An agent will have a *sequencer* that queues up *sequences*, or test cases to be passed on to the design. The sequence will go through a *driver* and onto the interface. The agent will also have a *monitor* that captures the response from the DUT and sends that response to a *scoreboard* located in the environment, which keeps track of DUT results and ensures that the expected output was recorded. 
![UVM Overview Picture](/images/UVM_overview.png)
Although this sounds like a complicated process with too many different elements, it is a key part of UVM. It allows for code reusability and parameterization, as well as a defined reporting system to log results. UVM utilizes Transaction Level Modeling (TLM) for communication, and often uses macros to automate the generation of UVM code.

Overall, UVM is a fundamental verification tool that has been widely adopted by industry. This is just a brief overview of UVM, and further reading can be found in the [Accellera UVM user guide](https://www.accellera.org/images/downloads/standards/uvm/uvm_users_guide_1.2.pdf).

## HTAX Verification Environment
The following sections give a brief overview of how various UVM components were implemented in the context
of verifying the HTAX communication module. Snippets will be shown throughout, and the full code can be found on [Github](https://github.com/vishnuvardhanRamesh/HTAX_verification).
### Packets

The first part of the environment involves creating HTAX packets. These packets contain information that will be sent to the DUT, and are the basis for any stimulus. These packets typically contain class member variables that can be randomized for testing, and constraints to control the randomization. An abbreviated version of a packet is showcased below.

```verilog
class htax_packet_c extends uvm_sequence_item;
    //define class members
    rand bit[6:0] delay;
	rand bit[3:0] dest_port;
	rand bit[VC-1:0] vc;
	rand bit[64:0] length;
	rand bit [WIDTH-1:0] data [];

    /*
    define UVM macros
    define class constructor
    */

    //add constraints
    //Constraint 1 : delay should be between 1 and 20
	constraint delay_cons {delay inside {[1:20]};}

	//Constraint 2 : dest_port should be between 0 and (PORTS-1)
	constraint dest_port_cons {dest_port inside {[0:PORTS-1]};}

	//Constraint 3 : VC should be valid VC request (should not be 0)
	constraint vc_cons {vc != 0;}

endclass: htax_packet_c
```

### Sequences
After creating packets, we create a *sequences* to drive these packets onto the interface, which will eventually pass the signals onto the DUT. A sample sequence is shown below.

```verilog
class fix_dest_port_seq extends htax_base_seq;

	//Factory Registration
	`uvm_object_utils(fix_dest_port_seq)

	//Constructor
	function new ( string name = "fix_dest_port_seq");
        super.new(name);
    endfunction : new

	//Body task -- 
	virtual task body();
		int i;
		i=$urandom_range(0,3);
		`uvm_info(get_type_name(),"Executing fix destination port sequence with 5 transactions", UVM_NONE)
		//Generate a sequence with 5 packets
		repeat(5) begin
			`uvm_do_with (req, {req.dest_port==i;}) //fix dest_port 
		end
	endtask
endclass : fix_dest_port_seq
```
### Driver

The next key component is the driver. The driver will take the data from the packet and actually send those signals to the DUT, taking into account variables given from the packet. For example, the HTAX packets have a *delay* variable, which specifies the delay before a signal is sent. The driver will take the packet delay as input, and wait however many cycles specified before sending the packet. An abbreviated driver is shown below.

```verilog
task htax_tx_driver_c::drive_thru_dut(htax_packet_c pkt);
	`uvm_info (get_type_name(), $sformatf("Input Data Packet to DUT : \n%s", pkt.sprint()),UVM_NONE)
	//Wait for pkt.delay clock cycles
	repeat (pkt.delay) @(posedge htax_tx_intf.clk);
	
	//Send VC and Outport Request
	@(posedge htax_tx_intf.clk)
	htax_tx_intf.tx_outport_req = 4'b0001 << (pkt.dest_port); //one hot encoding
	htax_tx_intf.tx_vc_req = pkt.vc;
	//Wait till htax_tx_intf.tx_vc_gnt is received
	@(posedge |htax_tx_intf.tx_vc_gnt);

	//Wait for VC grant, then start transmission (and send first data packet)
	@(posedge htax_tx_intf.clk)
	for (int j = 0; j < 2; j++) begin
		if (htax_tx_intf.tx_vc_gnt[j] == 1'b1) begin
			htax_tx_intf.tx_sot[j] = 1'b1;
			break;
		end
	end
	htax_tx_intf.tx_data = pkt.data[0]; //driving LSB
	htax_tx_intf.tx_outport_req = 0;
	htax_tx_intf.tx_vc_req = 0;

	//Sending command signals (start/end of transmission, releasing virtual channel)
	for(int i=1; i < pkt.length; i++) begin 
		@(posedge htax_tx_intf.clk);
		htax_tx_intf.tx_data = pkt.data[i];
		if (i == 1) begin
			htax_tx_intf.tx_sot = 0;
		end
		if (i == (pkt.length - 2)) begin
			htax_tx_intf.tx_release_gnt = 1;
		end
		if (i == (pkt.length - 1)) begin
			htax_tx_intf.tx_release_gnt = 0;
			htax_tx_intf.tx_eot = 1;
		end
	end

    //End of transmission
	@(posedge htax_tx_intf.clk);
	htax_tx_intf.tx_data = 'x; //assign all X
	htax_tx_intf.tx_eot = 0;

	`uvm_info (get_type_name(), $sformatf("Ended Driving Data Packet to DUT"), UVM_NONE)
endtask : drive_thru_dut	
```

### Monitor and Scoreboard

Just like how the driver takes information from the packet and sends it to the DUT, a monitor can be implemented to capture the result from the DUT. This result typically gets sent to a *scoreboard*, a component that tracks results and compares them with expected values. Monitors and scoreboards often communicate with each other using Transaction Level Modeling. An abbreviated scoreboard is shown below. At a high level, this scoreboard checks to make sure that all packets received in the monitor are the exact packets delivered by the transmitter. If the queue is not empty at the end of the testbench, that means there was a mistake somewhere in the design, and the simulation will flag that as an issue.

```verilog
    class htax_scoreboard_c extends uvm_scoreboard;
    
    /*
    Register scoreboard with factory
    */

    //Creating queue to store the incoming TX transactions
	htax_tx_mon_packet_c port0_queue[$], port1_queue[$], port2_queue[$], port3_queue[$], cmp_pkt[4];	

	//Write Method - TX[0] Monitor
	function void write_tx0_export(htax_tx_mon_packet_c tx_mon_packet);
		tx_mon_packet.print();
		push_to_queue(tx_mon_packet);
	endfunction : write_tx0_export

    /*
        Similar functions will be written for all TX ports
    */

    function void push_to_queue(htax_tx_mon_packet_c mon_pkt);
        case (mon_pkt.dest_port)
            0: begin
                port0_queue.push_front(mon_pkt);
            end
            1: begin
                port1_queue.push_front(mon_pkt);
            end
            2: begin
                port2_queue.push_front(mon_pkt);
            end
            3: begin
                port3_queue.push_front(mon_pkt);
            end
        endcase
	endfunction : push_to_queue

	//Write Method - RX[0] Monitor
	function void write_rx0_export(htax_rx_mon_packet_c rx_mon_packet);
		`uvm_info("SCOREBOARD",$sformatf("Received Data Packet on Port0:"), UVM_NONE)
		rx_mon_packet.print();
		cmp_pkt[0] = new ();
		cmp_pkt[0] = port0_queue.pop_back();
		if(cmp_pkt[0].data==rx_mon_packet.data)
			`uvm_info("SCOREBOARD","Data matches for received pkt on port 0", UVM_NONE)
		else
			`uvm_fatal("SCOREBOARD","Data mismatch for received pkt on port 0")
		`uvm_info("SCOREBOARD","Dropping pkt from queue 0", UVM_NONE)
	endfunction : write_rx0_export

    /*
        Similar methods will be written for each monitor port
    */

    function void check();
		`uvm_info("SCOREBOARD", "End of Simulation Checking", UVM_NONE)
		if(port0_queue.size()==0)
      `uvm_info("SCOREBOARD","Port 0 Queue is empty", UVM_NONE)
    else
      `uvm_fatal("SCOREBOARD","Port 0 Queue is non-empty at end of simulation")
    /*
        Similar statements will be made for each queue
    */
    endfunction: check
endclass : htax_scoreboard_c
```

## SystemVerilog Assertions

The verification environment described above illustrates how to send packets to the DUT, and check if the results calculated match the expected value. One other aspect of checking is to see if the design is behaving as specified in the funcitonal specification. This involves creating assertions, as described in the verification plan, to catch any critical errors when testing the design. A few example assertions used in the HTAX verification can be seen below.

```verilog
property tx_outport_req_one_hot;
      @(posedge clk) disable iff(!rst_n)
      (|tx_outport_req) |-> $onehot(tx_outport_req);
   endproperty

   assert_tx_outport_req_one_hot : assert property(tx_outport_req_one_hot)
   else
      $error("HTAX_TX_INF ERROR : tx_outport request is not one hot encoded");


   logic tx_vc_gnt_high; //checking if tx_vc_gnt is high in ANY previous clock cycle
   assign tx_vc_gnt_high = ~rst_n ? 1'b0 : (|tx_eot) ? 1'b1 : tx_sot ? 1'b0 : tx_vc_gnt_high;
   property no_tx_eot_without_previous_tx_vc_gnt;
      @(posedge clk) disable iff(!rst_n)
      (tx_eot) |-> tx_vc_gnt_high;
   endproperty

   assert_no_tx_eot_without_previous_tx_vc_gnt: assert property(no_tx_eot_without_previous_tx_vc_gnt)
   else
      $error("HTAX_TX_INF ERROR : tx_eot without prev tx_vc_gnt");
```

## Technologies

SystemVerilog was used as the primary language for coding the verification environment. Cadence XCelium was used to simulate the environment, and Cadence vManager and IMC were used to gather verification metrics. Github was used to save progress throughout the duration of the project.

## Conclusion

This verification project was done throughout a semester as part of a Hardware Verification class. Although there were many different components involved, this comprehensive environment allowed for easy regression testing. Over one thousand tests were done to find errors in the design that would be hard to find otherwise.

It is important to note that this is a small design. Designs in industry can be thousands of times bigger, and will require an even larger verification environment to accurately find any errors before tape-out. Because of this, many say that verification is one of the bottlenecks in chip design. There are many impressive research papers that attempt to make verification faster, such as this [IEEE paper](https://ieeexplore.ieee.org/document/10616069/) that uses automated assertion-based coverage to reduce the manual labor required. There are also attempts to use [machine learning based approaches](https://dvcon-proceedings.org/wp-content/uploads/machine-learning-guided-stimulus-generation-for-functional-verification.pdf) to speed up the process of verification.

Ultimately, hardware verification is an interesting problem that will most likely see more and more innovations in an attempt to speed up chip production in the semiconductor industry.













