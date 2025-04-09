---
title: "8-bit Pipelined Adder - Physical Chip Design"
description: "A description of the process used to create an 8-bit pipelined adder in the ASIC design flow"
date: "2024-11-05"
author: "Vishnu Ramesh"
math: true
---

Creating an adder in Python takes about thirty seconds. As a matter of fact, in almost every programming language this simple addition operation a miniscule amount of time to program. Writing an adder in assembly code may take slightly longer, though it will still not be a problem. Creating an 8-bit adder as an IC design took me almost six weeks to accomplish.

At the end of the day, *physical integrated chip design* refers to strips of metal connecting to each other, sending electrical current one way or the other. Controlling the flow of electricity throughout these strips of metal can be a complicated process, and eventually ends up looking something like this.

![alt text](/images/innovus_circuit.png)

Regardless, physical design is the backbone of all chips, and allows for unprecendented increases in performance. Although there are many automated tools to automate this process (Synopsys DC, Cadence Innovus, etc.), In this article, I will go over creating an 8-bit adder almost entirely from scratch. I used Cadence Virtuoso for the majority of the design tooling.

## NAND, NOR, XOR Gate Creation

At a high level, a 1-bit adder is composed of 3 NOR gates and 2 XOR gates. These adders can be put together to create a 4-bit, and then an 8-bit adder. The first step is to create these gates.

Using Cadence Virtuoso, a *schematic* view can be created. This view is relatively high level, and involves placing transistors, input/output pins, and power/ground supplies. The schematic of a XOR gate is shown below.

![XOR gate schematic](/images/XOR_schematic.png)

After creating the schematic, a *layout* view is created. This is the lowest level of abstraction, and involves placing strips of metal to connect transistors together. There are multiple things to consider, such as the type of metal connecting used (poly, metal1, metal2, etc.), building the power rail using a well, and the different contact layers placed. A sample layout for a XOR gate is displayed below.

![XOR gate layout](/images/XOR_layout.png)

Next, after creating both the layout and schematic, a DRC and LVS check is performed. The Design Rule Check ensures that design rule requirements are met. For example, the DRC will flag an error if wires are too close to each other, or if there are unconnected wires. Afterwards, a Layout Versus Schematic check is performed to ensure that the layout and schematic have the same functionality. These two checks are important parts of the verification process when designing IC's.

For further testing, Cadence Spectre can be used to characterize the gates, finding results such as the rising/falling delay, as well as the sink capactiance. 

## 1-bit and 4-bit Adder

Now that the necessary gates were made, a 1-bit adder can now be implemented. The same process can be used to build these adders by building the schematic/layout view. After building the layout, additional testing can be done by creating a separate circuit. This circuit sends analog input signals to the adder and captures the result in a waveform. The layout for the 4-bit adder is shown below. The complexity of the IC design grows exponentially as more gates and interconnections are added.

![Four bit adder layout](/images/four_bit_layout.png)

## Flip-Flop

There are many different ways to design an adder. In this case, we are connecting two 4-bit adders together, using a flip-flop inbetween to store the intermediary result. The flip-flop implemented below utilizes six inverters and four latches. 

![Flip-Flop Schematic](/images/flip_flop_schematic.png)

Once again, the same process is used to create the flip-flop, and the final result connects all these pieces together to form a coherent 8-bit pipelined adder.

## Conclusion

The creation of a pipelined adder using IC design tools is very tedious when done manually. There are many things to consider when building the adder, and mistakes are easy to make and hard to uncover. Thankfully, there are many tools now that automate the process of physical design. Cadence Innovus in particular is a great tool to convert gate-level Verilog code to a physical layout automatically. Various parameters can be configured to optimize the layout, such as the aspect ratio and cell density. Using Innovus can still take quite some time to create an optimized solution (some designs in my own research take more than 24 hours!), however the process is done autonomously and engineers can work on other tasks.

Ultimately, IC design is an integral part of this world, and has seen countless innovations in the last few decades. There is more excitement in improving hardware designs with the increase in AI technologies. Custom AI accelerators are created for boosting the training of large models, and low power designs are being explored for optimized power efficiency. In the future I hope to see tools that allow for an even easier chip design process, with the potential for fabrication at a low cost.