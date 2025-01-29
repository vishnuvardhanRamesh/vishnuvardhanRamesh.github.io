---
title: "CNN Design on FPGA - Verilog"
description: "A description on the efficient implementation of a CNN design for FPGA's"
date: "2024-04-15"
author: "Vishnu Ramesh"
math: true
---

FPGA's (Field Programmable Gate Arrays) are utilized as the middle ground between ASIC's and processors. Microprocessors are cheap and flexible, though not very fast. ASIC's are very expensive and are not flexilbe (Hence the name, Application Specfic Integrated Circuits), though they will have the highest performance. FPGA's provide a higher level of flexibility compared to ASIC's while at the same time providing higher perfomance than general purpose processors.

FPGA's are implemented as an array of CLB's packed together. They typically include the following elements:
* CLB: Configurable Logic Block. These blocks generally contain flip-flops, Lookup Tables, and multiplexors. Verilog RTL code is typically broken down into boolean functions that are implemented into these CLB's.
* Connection Box: Connects CLB to routing interconnect
* Switch Box: Connects routing interconnect throughout the FPGA

This configuration allows for FPGA's to be extremely efficient when doing a large amount of math operations. Thus, it is interesting to see an implementation of a Convolutional Neural Network design, which relies on using a large amount of Multiply/Accumulate operations, implemented on an FPGA. 

Typical implementation on a FPGA involve for-loop unrolling and memory partioning to transmit data to PE's. This can cause a few issues.
* Large scale data-reuse with DSP blocks can create large fan-out
* DSP blocks are spread throughout the FPGA, creating large interconnect
* Large multiplexors may be required for output data collection

This implementation is based on this [IEEE paper](https://ieeexplore.ieee.org/document/8060313), which utilizes a Systolic Array structure to circumvent some of the typical problems with implementing this design on an FPGA.

## Systolic Array Structure

A systolic array is a specific data structure where Processing Elements are placed in an array-like format, and are only connected to adjacent PE's. This results in local interconnections and easy to implement pipelining between PE's. Data is pumped in and out of the array and the output is stored either in each PE itself or shifted out to an IO buffer.

Here is a typical implementation showcasing matrix multiplication.

{{< math.inline >}}
$$
\begin{align*}
A = \big[\begin{matrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{matrix}\big]
\quad \quad \quad \quad
B = \big[\begin{matrix} b_{11} & b_{12} \\ b_{21} & b_{22} \end{matrix}\big]
\end{align*}
$$
{{</ math.inline >}}

{{< math.inline >}}
$$
\begin{align*}
AB = \big[\begin{matrix} a_{11} \cdot b_{11} + a_{12} \cdot b_{21} & a_{11} \cdot b_{12} + a_{12} \cdot b_{22} \\ a_{21} \cdot b_{11} + a_{22} \cdot b_{21} & a_{21} \cdot b_{12} + a_{22} \cdot b_{22} \end{matrix}\big]
\end{align*}
$$
{{</ math.inline >}}

![Systolic Array Multiplication Illustration](/images/sysyolic_mult.png)

Similarily, this data structure can be used for the multitude of multiply/accumulate operations done in a Convolutional Neural Network. The figure below shows the basic implementation. An array of Processing Elements are clumped together, with inputs cascading down the array and weights cascading to the right of the array. The output is then propogated back up and into the output buffers, where the results can be stored in memory. 

This process allows for easy pipelining between each processing element, as well as flexibility in the size of the array. This helps solve the challenges of PE placement and data reuse when building a CNN design for FPGA's.

The code for this can be found on [Github](https://github.com/vishnuvardhanRamesh/CNN_FPGA). it includes the implementation of the Processing Element as well as connecting the PE's together in a systolic aray.
