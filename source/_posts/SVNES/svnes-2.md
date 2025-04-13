---
title: SVNES
tags:
  - FPGA
  - NES
  - SystemVerilog
id: '1235'
categories:
  - - fpga
    - SVNES
date: 2017-12-12 23:48:43
---

To debug the SVNES system, a separate 6502 core was added.
<!-- more -->
It can directly write to part of SDRAM region, for output text to the TFT frame buffer.

The system bus and register inspections were done by using a parallel-to-serial shift-register-based data transfer interface, much like SPI/74xx595. Interface signals: `load`, `shift`, `din`, `dout`

[8cc84b0e572191a93f4fbc49ad9ab1b9a411a306](https://github.com/zhiyb/SVNES/commit/8cc84b0e572191a93f4fbc49ad9ab1b9a411a306)

\[youtube https://youtu.be/2E9pHYqWlOU\]

![IMG_9911](img_9911.jpg)