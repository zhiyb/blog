---
title: Nosumor remake
tags:
  - JTAG
  - Keyboard
  - Nosumor
  - PCB
  - RGB
  - STM32
  - USB
id: '681'
categories:
  - - c-c
    - Embedded
date: 2017-06-25 20:56:31
---

PCB and most components arrived, the PCB had been built and some basic testing had been done.
<!-- more -->
Keys are working, layout matches. RGB LEDs all work, but RGB labelling errors for keycap LEDs was found on the schematic. Green LED on right keycap RGB appears to be brighter.

The rectangular holes for keycap LEDs were not cut, probably the milling layer was not working? Or the drills override the rectangular mills?

The PCB is thinner than the original Nosumor. It probably should be 1.2mm.

FT2232D JTAG and UART interfaces work, but the reset signal in default UART mode will be driven low, keeps resetting all chips. The reset signals need to be moved to input pins in UART mode.

The USB HS PHY is able to receive 19.2MHz clock from MCU, and generate the required 60MHz clock for ULPI. It is able to disconnect/reconnect USB, emulate the device (as LS currently). Further tests need to be done.

The audio codec and MMC has not been tested yet.

Gallery: \[gallery ids="733,735,737" type="rectangular" link="file"\]