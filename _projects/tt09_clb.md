---
layout: page
title: Simple Configurable Logic Block on Tiny Tapeout
description: A 3 bit CLB on Sky130
img: assets/img/tt/clb.png
importance: 3
category: fun
pretty_table: true
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

## Tiny Tapeout Design

Due to the IO constraints of Tiny Tapeout, several design choices had to be made. For one, no full add logic is included in this design. While
there is space on the IO pads for a full adder enable and output, there is no space for another CLB to showcase the carry out for arithmetic
efficiency. Additionally a LUT3 is used since there are 8 bits dedicated to input and bi-directional ports. Meaning the entire bi-directional
bus is used for inputting a boolean function and the dedicated input ports are entirely for control. This means there is only 1 bit dedicated to
output and supports all 256 ternary boolean functions. Below is the logical schematic synthesized by Yosys and rendered by [DigitalJS](https://digitaljs.tilk.eu/).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/tt/digital_js.png" title="Top Mod Schematic" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

### LUT3 Design

As mentioned before, due to IO constraints a LUT3 is implemented. There are two main choices in implementing the memory for the LUT: SRAM or flip flops.
Flip flops are simple to implement in RTL as it is just the synthesizers job but have 8 extra nets added to the clock net. SRAMs only have a single connection
to the clock net for write enable and are smaller, but require an external memory compiler, changes to the config file, and there are not enough bits to remember
to justify them. Additionally, given that the design already is small enough, the extra routing and clock tree synthesis for 8 flip flops is not that big of a deal.

The IO for this module is simple and works as follows:

| Port       | Direction | Width | Description                                         |
| ---------- | --------- | ----- | --------------------------------------------------- |
| clk_i      | Input     | 1     | Clock for FFs                                       |
| resetn     | Input     | 1     | Active low reset for FFs                            |
| new_seed_i | Input     | 1     | Clock enable for FFs to take in a new seed/function |
| args_i     | Input     | 3     | Arguments in the function to evaluate it            |
| seed_i     | Input     | 8     | Represents boolean function to be held in FFs       |
| out_o      | Output    | 1     | Evaluation of the function given arg_i              |

Below is a waveform of loading in NAND3 functionality into the LUT.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/tt/nand3_vcd.png" title="NAND3 VCD" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Once the new_seed_i signal is high is the new function stored in the LUT. Since we have
programmed a NAND3, we only see out_o go low when all 3 arguments are high.

### Top Module

Given that the full adder and carry logic are omitted from this IC due to IO constraints,
the top module is just describing the multiplexing of the LUT3's direct output and the
output of a flip flop driven by the LUT3. This is done by saving the ui_in[4] input in a
separate flip flop when ui_in[3] is high to send in a new seed in the LUT3. When ui_in[4]
is high, the final output is driven by the flip flop rather than directly by the LUT3.

Below is a description of the IO for the top module.

| Port    | Direction | Width | Description                                                                |
| ------- | --------- | ----- | -------------------------------------------------------------------------- |
| clk     | Input     | 1     | Clock for the entire design, at 50Mhz                                      |
| rst_n   | Input     | 1     | Active low reset for entire design                                         |
| ui_in   | Input     | 8     | Dedicated Input                                                            |
| uio_in  | Input     | 8     | Input path of the bi-direction IO, dedicated entirely to the seed/function |
| ena     | Input     | 1     | Used by Tiny Tapeout, ignored                                              |
| uo_out  | Output    | 8     | Dedicated Output                                                           |
| uio_out | Output    | 8     | Output path of the bi-directional IO, unused                               |
| uio_oe  | Output    | 8     | Enable path of the bi-directional IO, set low to only use input path       |

Below is a description of the ui_in bus separated by bits.

| Port       | Direction | Width | Description                                                                                     |
| ---------- | --------- | ----- | ----------------------------------------------------------------------------------------------- |
| ui_in[0]   | Input     | 1     | Input C of the boolean function, also the LSB of the function                                   |
| ui_in[1]   | Input     | 1     | Input B of the boolean function                                                                 |
| ui_in[2]   | Input     | 1     | Input A of the boolean function, also the MSB of the function                                   |
| ui_in[3]   | Input     | 1     | When high writes in what uio_in holds in the LUT3 FFs and saves synchronous setting             |
| ui_in[4]   | Input     | 1     | Selects the between outputting the direct LUT3 output or from a FF, saved when ui_in[3] is high |
| ui_in[7:5] | Input     | 3     | Unused                                                                                          |

Below is a description of the uo_out bus separated by bits.

| Port        | Direction | Width | Description                                                                                       |
| ----------- | --------- | ----- | ------------------------------------------------------------------------------------------------- |
| uo_out[0]   | Output    | 1     | Evaluation of the boolean function given the args, synchronous depending on the state of ui_in[4] |
| uo_out[7:1] | Output    | 7     | Unused                                                                                            |

### Common Functions

Here is a table for common 3 bit boolean functions and the corresponding seed to program the CLB.

| Function    | Seed/uio_in[7:0] |
| ----------- | ---------------- |
| NAND3       | 01111111         |
| NOR3        | 00000001         |
| NOT         | 01010101         |
| XOR2        | 01100110         |
| Majority    | 11101000         |
| Even Parity | 01101001         |
| One Hot     | 00010110         |

## Verification

Given the RTL itself is simple and did not take a lot of time, most of the effort went into verification. Since Tiny Tapeout is aimed
at those with little to no hardware design experience, it uses coco-tb, a Python framework, for the test flow. A vcd (value change dump)
file is produced to create a waveform that is displayed in a waveform viewer of choice. During this project, a Github Codespaces was used
to facilitate development over the web without any local installation of any tool and used the default waveform viewer provided: Surfer.

### Test Design

Extensive effort was spent optimizing tests. There is a main test that simply calls all other test functions. This main test thread holds
a dictionary mapping strings to boolean integer literals as shown below.

```python
seeds = {"not": 0b01010101, "and2": 0b10001000, "or2": 0b11101110, "xor2": 0b01100110,
            "nand2": 0b01110111, "nor2": 0b00010001, "nand3": 0b01111111,
            "nor3": 0b00000001,"majority": 0b11101000, "even_parity": 0b01101001,
            "one_hot":0b00010110}
```

Each test function is an async Python or co-routine that takes in 3 arguments: a "dut" argument which is the main top module as a Python class,
"seed" which is an integer representing the function to be programmed and tested in the CLB, "sync" a boolean defaulted to false
representing if the flip flop attached to the end of the CLB is used or bypassed. Below is the co-routine to test the NAND3 function.

```python
async def test_a_b_c_nand(dut,seed,sync=False):
    dut._log.info(f"Start Test: A nand B nand C | Is Sync:{sync}")
    dut.uio_in.value = seed
    sync_mask = int(sync) << 4
    sync_cycles = 2 if sync else 1
    dut.ui_in.value = 0b00001000 | sync_mask
    await ClockCycles(dut.clk,1)
    dut.ui_in.value = 0b00000000 | sync_mask
    await ClockCycles(dut.clk,1)
    input_value = 0
    while(input_value != 8):
        dut.ui_in.value = input_value | sync_mask
        a_value = input_value & 0b00000001
        b_value = (input_value & 0b00000010) >> 1
        c_value = (input_value & 0b00000100) >> 2
        await ClockCycles(dut.clk,sync_cycles)
        dut._log.info(
            f"
            A:{bin(a_value)} B:{bin(b_value)} C:{bin(c_value)} Out:{dut.uut.uo_out.value}
            ")
        assert dut.uo_out.value == (~(a_value & b_value & c_value) & 0b00000001)
        input_value += 1
    await ClockCycles(dut.clk,1)
```

Co-routines begin with logging the function to be tested. To handle synchronous selection a sync mask is created and
bitwise OR'ed with inputs into ui_in assignments. This works since synchronous selection is an active high signal. When
the CLB flip flop is used a 2 cycle wait is used and 1 if it bypassed since the testbench moves the clock by cycles rather than
edges. First the seed is loaded and CLB bypassed is selected. After this sequence a while loop is ran. An assert is ran checking
the value of the CLB compared to a Python evaluation of the function after an appropriate amount of cycles is passed. The loop
terminates after the number 8 is reached since this means we have checked all possible arguments into the function.
