---
title: Nosumor remake
tags:
  - Nosumor
  - PCB
id: '879'
categories:
  - - C/C++
  - - c-c
    - Embedded
date: 2017-07-20 02:20:04
---

PCB version 0.3 arrived at 10/Jul/2017.

All modules are working.
<!-- more -->
A special USB PHY delay configuration bit that enables high speed emulation was documented in the datasheet, but not defined in `CMSIS`. High speed USB communication was achieved after set that bit. Ref: [d070bdc07e3bdfec3a8a624d0a44e92771caf28e](https://github.com/zhiyb/Nosumor/commit/d070bdc07e3bdfec3a8a624d0a44e92771caf28e)

DMA mode USB data transfer was also implemented to improve efficiency and speed. Ref: [05829952d575437ad2a27e92f92a793220bf7cf9](https://github.com/zhiyb/Nosumor/commit/05829952d575437ad2a27e92f92a793220bf7cf9)

Implemented basic `SDXC` card reading support for `FatFs` with `exFAT`. Currently able to list the root directory. Ref: [a533bfca300d274cbab92f0cf839af180bcc39e1](https://github.com/zhiyb/Nosumor/commit/a533bfca300d274cbab92f0cf839af180bcc39e1)

The `STM32F722` datasheet specifies a maximum clock of `50MHz` for the `SDMMC` module, but apparently overclocking to `72MHz` seems working correctly. Ref: [f921fa0003cd33b52d53c10216d65118ff924933](https://github.com/zhiyb/Nosumor/commit/f921fa0003cd33b52d53c10216d65118ff924933)

The `19.2MHz` clock output do work for both USB PHY and audio codec, exact `192kHz` audio sample rate was achieved. Ref: [c284e13682f06af5ae020a14341e6799a0cb27d1](https://github.com/zhiyb/Nosumor/commit/c284e13682f06af5ae020a14341e6799a0cb27d1) Ref: [a9983ac51ba2dee561845c982c949c1cd8b875a2](https://github.com/zhiyb/Nosumor/commit/a9983ac51ba2dee561845c982c949c1cd8b875a2)

Implemented `DMA` transfer for audio codec, sine wave output seems working. 3.5mm jack was also working as expected. Ref: [acdb68e6fa551f508fab2dfc265e23cf27ff71f5](https://github.com/zhiyb/Nosumor/commit/acdb68e6fa551f508fab2dfc265e23cf27ff71f5)

TODO list: USB audio interface USB configuration interface and GUI application USB mass storage interface for `SDMMC` RGB LEDs rendering and configuration In-Application Programming for firmware update Bootloader for recovery Propose a new name?

Ref: [c284e13682f06af5ae020a14341e6799a0cb27d1](https://github.com/zhiyb/Nosumor/tree/c284e13682f06af5ae020a14341e6799a0cb27d1)

Gallery: \[gallery ids="941,939,938,940" type="rectangular" link="file"\]