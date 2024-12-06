---
layout: page
title: SDRAM Contoller Chisel Generator
description: Final project for CSE 228A - Agile Hardware Design
img: assets/img/sdram_gen_imgs/sdram_diagram.png
importance: 1
category: work
---

## Introduction

The goal of this project is to take in a JSON file describing timing characteristics of a SDRAM module and
generating a Verilog file describing the desired controller module for use on an FPGA or tapeout for an ASIC.
Below is the dataflow of the generator from JSON to Verilog.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/sdram_gen_imgs/sdram_gen_overview.png" title="Chisel Generator flow of .json file to .v file" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

A hardware generator language here is used rather than a hardware description language due to flexiblity. Verilog would
create a single instance of an SDRAM controller and would be rigid if any changes are needed for the controller or someone
else would like to reuse it for a different SDRAM module. My project aims at providing hobbyists a program that just needs
them to write down key values from their SDRAM's datasheet to generate a controller for them to use.

This is an open source Scala project and contributions are welcome! Visit the repo at: [https://github.com/gmejiamtz/sdram_controller_generator](https://github.com/gmejiamtz/sdram_controller_generator)

## Future Work

Below is a list of future work for the project to take off for my liking:

1. Get Data Transfer to work properly - Chisel Analog is not as mature as Verilog's inout and is causing me problems in data movement to and from the SDRAM via the controller

2. Target more SDRAMs - as of now I am targetting only the Micron MT48LC1M16A1 and I believe different SDRAMs have different initialization procedures and command formats

3. Create a JSON template and parser front-end - will be a future assignment as this is fairly easy to parse but the template will take time
