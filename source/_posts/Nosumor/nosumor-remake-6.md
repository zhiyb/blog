---
title: Nosumor remake
tags:
  - HID
  - IAP
  - Keyboard
  - Nosumor
  - PCB
  - STM32
  - USB
  - USB Audio Class
id: '979'
categories:
  - - C/C++
  - - c-c
    - Embedded
date: 2017-08-03 03:25:17
---

USB configuration HID port was implemented. It is able to communicate with the host application to flash firmware.
<!-- more -->
HID code structure was updated to use only a single HID interface for keyboard, the configuration port and possibly others.

The configuration port uses a HID vendor defined report structure for communication. Therefore, no specific driver needed.

A host console application was implemented to communicate with the device and upload new firmware images, using cross-platform `hidapi` library. Ref: [HID API for Linux, Mac OS X, and Windows](www.signal11.us/oss/hidapi)

The firmware image format supported was Intel HEX format. Ref: [Intel HEX](https://en.wikipedia.org/wiki/Intel_HEX)

The host application compresses the ASCII content to binary streams, by forming a new packet for every start code (`:`). Then, every 2 ASCII bytes in the line will be read as hexadecimal and stored as a single byte. The packet will be transferred as HID report, has a constant size of `3 + 32` bytes, should be enough for `gcc objcopy` generated hex files.

The device receives and verifies the checksums of hex contents, storing them in a linked list. It also replies with a validity packet when the host asked.

The device will begin flashing after it received a start packet. It only erases affected flash sections, and do address translation from `ITCM` bus addresses to `AXIM` bus addresses for programming. The difficulties were the flashing routine must not access any resources inside the flash after it has been erased.

The flash has a separate `ITCM` bus with `ART` accelerator specifically for instruction fetching, might be faster than sharing the `AXIM` bus. It was switched to in the release build variant. However, the ITCM interface does not support writing. Therefore, when using the ITCM interface, the `OpenOCD` `gdb` server cannot write to this section. The debug build variant keeps using `AXIM` interface for `gdb` debugging support.

A stripped down version consists of only HID keyboard and configuration port was used as an optional bootloader, flashed to the last `128KiB` section of the flash. It can be activated by pressing the 3 small buttons at the same time before booting. It is still possible to corrupt the main firmware image to be not able to boot into the bootloader. But setting the flash option byte and switch the PCB boot jumper can still save the device.

The boot jumper switches between `VCC` and `GND`. If used incorrectly, may cause a short circuit between `VCC` and `GND`. It should be redesigned as a 10k pull-up resistor to VCC, and an optional jumper connecting the `BOOT` pin after the resistor to `GND`. Possibly sharing a button press for easier access.

Audio interface and descriptors were implemented according to USB Audio Class (UAC) specification version 1.00. However, UAC1 does not support USB high speed isochronous endpoint polling intervals. Therefore, it is unfortunately not working, need to upgrade to UAC version 2.00.

Ref: [d2b47402f44870d5f69e770abdb3bcf500d43c78](https://github.com/zhiyb/Nosumor/tree/d2b47402f44870d5f69e770abdb3bcf500d43c78)

Gallery: \[gallery ids="1062,1063" type="rectangular" link="file"\]