---
title: Nosumor remake
tags:
  - codec
  - Nosumor
  - RGB
  - STM32
  - UAC2
  - USB
  - USB Audio Class
id: '1067'
categories:
  - - C/C++
  - - c-c
    - Embedded
date: 2017-09-13 22:10:32
---

\* Added USB audio rendering using USB Audio Class 2 Ref: [remake-v0.3](https://github.com/zhiyb/Nosumor/releases/tag/remake-v0.3)
<!-- more -->
Currently, only 192000kHz & 32bit audio format was supported.

The audio codec supports multiple layers of volume/gain configurations for different elements. However, only the feature unit with volume control just before the end of output terminal will be used as the overall volume control. [64da0dc2fa2a438d14f6d29fb2357af368ce0a3b](https://github.com/zhiyb/Nosumor/commit/64da0dc2fa2a438d14f6d29fb2357af368ce0a3b)

Probably only the DAC volume should be reported, or try to linearize the analog attenuations. Other units may be controlled using the HID vendor report and dedicated host application.

High-speed asynchronous isochronous endpoint (8kHz) with feedback endpoint (1kHz) was used for audio output, to ensure no buffer overrun/underrun and reduce latency.

Dedicated endpoint 1 interrupts was used for audio endpoints, to reduce latency in calling audio processing functions. STM32 seems forcing NAKs to isochronous endpoints if enabled too late / at inappropriate times, results in audio randomly stopped working.

To enable dedicated endpoint interrupts, registers `DEACHINT`, `DEACHMSK` and `DOUTEP1MSK`/`DINEP1MSK` need to be used. While `DEACHINT` and `DEACHMSK` were documented in the `STM32F72` reference manual (rev 1), `DOUTEP1MSK` and `DINEP1MSK` were not. They are documented in the `STM32F42` reference manual (rev 13) and are defined in the header files. Ref: [98f132bba9c94cd500419340bd4b8075443fc399](https://github.com/zhiyb/Nosumor/commit/98f132bba9c94cd500419340bd4b8075443fc399)

The mysterious interrupt flag causing endpoint 0 to stop firing interrupts after received data packets was also found. There are apparently an `OTEPSPR` bit in the `DOEPINT` register and an `OTEPSPRM` bit in the `DOEPMSK` register, defined in the header files as "Status Phase Received For Control Write" and "Status Phase Received mask". However, they were not mentioned anywhere in `STM32F72` and `STM32F42` reference manuals. Ref: [f1094f83f84872099f925fe712ae05f483ff4699](https://github.com/zhiyb/Nosumor/commit/f1094f83f84872099f925fe712ae05f483ff4699)

The sideway RGB LEDs have built-in reverse zener(?) diodes to protect green and blue LEDs (why not red?). This requires open-drain cathode controls. Otherwise, current can still flow through the diodes: ![rgb_diode](rgb_diode.png) Ref: [MSL0104RGB Series](http://www.mouser.com/ds/2/348/msl0104rgb-e-1139176.pdf) Ref: [66c5f3891aad4c7d8f02718bb6facaa1c5a482bd](https://github.com/zhiyb/Nosumor/commit/66c5f3891aad4c7d8f02718bb6facaa1c5a482bd)