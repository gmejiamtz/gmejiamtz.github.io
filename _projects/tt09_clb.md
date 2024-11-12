---
layout: page
title: Simple Configurable Logic Block on Tiny Tapeout
description: A 3 bit CLB on Sky130
img: assets/img/tt/clb.png
importance: 3
category: fun
---

## Introduction

Field programmable gate arrays are integrated circuits designed to deploy digital circuits outside of the fab.
The base of any FPGA architecture is the configurable logic block or CLB. At the base level a CLB contains a look-up
table or LUT, a full adder, and flip flop.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/tt/clb.png" title="CLB with 4-bit LUT" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
An example CLB complete with 4 bit LUT, full adder and FF
</div>

### Look Up Tables

To allow universal boolean function support, a function is loaded into the LUT as a binary string. LUTs are memories connected to
n to 1 muxes. The selector signals are used to "evaluate" the function and produce an output. LUT naming convention is LUTk,
where k is the number of bits in the selector signal of the mux. Common LUT sizes are LUT4 and LUT6. A LUTk can represent of up to
2^(2^k) boolean functions. Below is an example of boolean function being loaded into a LUT and evaluated.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/tt/lut_seeding.png" title="Example of LUT Loading" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
Part a: Truth table of f(a,b,c) = !a!bc | !ab!c | !abc | a!b!c | ab!c.
Part b: Mux with selector signals {a,b,c} and loaded values of 0b0101_1110
Part c: Black box of mux with inputs a,b,c and output f representing a loaded LUT module
</div>

### Flip Flops

For synchronous circuits, a flip flop is attached to the end of the CLB. A mux is provided and controlled by the synthesis tool to either send
a the output of the LUT synchronous to the clock or by pass it and send it as a combinational output.

### Full Adder

Since addition is a common arithmetic operation and a combinational function, a full adder is sometimes included inside a CLB. The carry logic
is used to improve the efficiency of common arithmetic functions like addition, subtraction, counting, comparision, etc.

### Tiny Tapeout Design

Due to the IO constraints of Tiny Tapeout, several design choices had to be made. For one, no full add logic is included in this design. While
there is space on the IO pads for a full adder enable and output, there is no space for another CLB to showcase the carry out for arithmetic
efficiency. Additionally a LUT3 is used since there are 8 bits dedicated to input and bi-directional ports. Meaning the entire bi-directional
bus is used for inputting a boolean function and the dedicated input ports are entirely for control. This means there is only 1 bit dedicated to
output and supports all 256 ternary boolean functions. Below is the logical schematic synthesis by Yosys and rendered by [DigitalJS](https://digitaljs.tilk.eu/)

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/tt/digital_js.png" title="CLB with 4-bit LUT" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
