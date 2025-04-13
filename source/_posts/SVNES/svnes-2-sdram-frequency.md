---
title: SVNES
tags:
  - DE0-Nano
  - FPGA
  - SDRAM
  - SystemVerilog
id: '1242'
categories:
  - - fpga
    - SVNES
date: 2017-12-09 23:43:19
---

The SDRAM IO frequency was increased to 142.5MHz, after corrected an error in timing specification.
<!-- more -->
The problematic setting, carried on from a DE0-Nano pin assignment template: `set_instance_assignment -name TSU_REQUIREMENT "10 ns" -from * -to *` [50d5dd12527513ab245274e7f4844358b2c1b96f](https://github.com/zhiyb/SVNES/commit/50d5dd12527513ab245274e7f4844358b2c1b96f#diff-9976d22341cd2ba3b6544a3ca1650f0aL380)

After removed the inappropriate setting, the SDRAM frequency can then be increased to the maximum (143MHz from [datasheet](http://www.issi.com/WW/pdf/42-45S83200G-16160G.pdf)).

To achieve feasible PLL settings, the frequency was decreased to 142.5MHz. Main PLL settings: `10MHz`(Audio), `30MHz`(TFT), `142.5MHz`(SDRAM), `285MHz`(HS bus) [e674cdabebc9c2bed2a1e4f2d7a111b008d90f5b](https://github.com/zhiyb/SVNES/commit/e674cdabebc9c2bed2a1e4f2d7a111b008d90f5b)

To debug SDRAM stability, I've used PLL dynamic phase reconfiguration to adjust IO phase shift relative to clock signal. Some interesting random patterns appeared though: \[youtube https://youtu.be/lxEIYpWmzDs\]