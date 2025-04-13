---
title: Nosumor remake
tags:
  - Keyboard
  - Nosumor
  - PCB
  - STM32
  - USB
id: '307'
categories:
  - - c-c
    - Embedded
date: 2017-06-11 06:07:13
---

Nosumor keyboard remake project, started on 07/Jun/2017, now finalised first prototype PCB design and started prototype production. The PCB layout was made compatible with the original Nosumor v3 front panel. (PCB manufacturer: Seeed Studio)

Features: 50x50 (mm) board area 2 mechanical keyboard switches and 3 buttons Keycap lighting (white) and 2 other independent RGB LEDs 3.5mm audio jack with TLV320AIC3110 audio codec STM32F722RE Cortex-M7 high performance MCU USB 2.0 high-speed PHY USB3370 On-board FT2232D connected to UART & JTAG Extension UART with 3.3V output
<!-- more -->
Estimated costs: Basic keyboard: £20 Complete configuration: £30 Complete with debugging: £38

Errata: No current limiting resistors with keycap LEDs (3.3V PWM direct) No pull-up resistors on data lines (internal pull-up resistors inside STM32 may or may not work) Lack of ferrite bead filter on audio AVDD

Schematic design: ![schematic.png](schematic.png)

PCB top layer design: ![top.png](top.png)

PCB bottom layer design: ![bottom](bottom.png)