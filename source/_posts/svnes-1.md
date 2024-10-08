---
title: SVNES
tags:
  - Embedded
  - NES
  - SDRAM
  - SystemVerilog
  - TFT
id: '563'
categories:
  - - hdl
    - FPGA
  - - HDL
  - - fpga
    - SVNES
date: 2017-06-23 03:10:15
---

SystemVerilog implementation of the NES hardware.

Using Altera DE0 Nano FPGA board, features Cyclone IV EP4CE22F17C6N FPGA (22,320 LEs, 594 kb) On-board 32MB SDRAM AT070TN92 800x480 24-bit TFT display

A complete reimplementation of previously unfinished version: Ref: [86ce2d5c7c7bdbaeb6bb298edfd711bb95508241](https://github.com/zhiyb/SVNES/tree/86ce2d5c7c7bdbaeb6bb298edfd711bb95508241)
<!-- more -->
Had finished CPU, APU, and some basics of GPU design in previous version, but that was discontinued due to SDRAM controller unreliability, code and implementation style.

This time, with SDRAM controller re-implemented with a different structure: `Command scheduler -> FIFO -> SDRAM IO` And multiple levels of pipelining, the SDRAM controller works quite efficiently and reliably. The reliability was tested by having a memory test client, writing then read back verifying frames of pseudo random sequences generated by a 33-bit LFSR.

A "proper" prioritised arbiter structure was also implemented to share SDRAM access between different components, including TFT and memory test client, potentially the ROM and GPU framebuffers (banked nametables).

However, due to the fundamental limitations of dynamic RAM (refresh, precharge and active phases), worsen by sharing access, the worst case latency of accessing the SDRAM from CPU exceeds the latency allowed by CPU clock frequency. Therefore, a [dynamic frequency scaling (?)](https://en.wikipedia.org/wiki/Dynamic_frequency_scaling) clock generator was designed, to delay the clock edge during SDRAM fetches and catches up later.

The original structure of the 6502 CPU uses latches as registers and a [two-phase clock](https://en.wikipedia.org/wiki/Clock_signal#Two-phase_clock) design. The microcode ROM was also more like a PLA than ROM. To save FPGA LE resources (also reduces compilation time), 32-bit microcode ROM approach was used instead, with minor processor architectural changes. The memory access also updates between the 2 phases to compensate for not using latches, by having 4-phase clock generation.

Currently, load, store, register transfer, status control and break instructions was implemented with the same clock cycle counts. More instructions can be supported by expanding the microcode ROM.

Ref: [614f9c6640a084045b191d8d79bcd746c1823639](https://github.com/zhiyb/SVNES/tree/614f9c6640a084045b191d8d79bcd746c1823639) Ref: [NES reference guide](http://wiki.nesdev.com/w/index.php/NES_reference_guide)