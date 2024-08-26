---
title: ASMedia 1061 PCIe SATA controller card
tags:
  - ASM1061
  - ASMedia
  - PCIe
  - SATA
id: '1260'
categories:
  - - uncategorized
date: 2018-01-17 15:57:34
---

Brought a PCIe x1 to SATA & mSATA controller card for Jetson TX2, but it was not working reliably. [Amazon: qobobo® SuperSpeed PCI-E to mSATA3.0 1-Port and SATA 3.0 1-Port Express Card](https://www.amazon.co.uk/gp/product/B01GLGINFK)
<!-- more -->
The card would randomly stop working during intensive disk operations, or after being online for about one day: \[code\]ata3.00: exception Emask 0x10 SAct 0xffc00 SErr 0x400000 action 0x6 frozen ata3.00: irq\_stat 0x08000000, interface fatal error ata3: SError: { Handshk } ata3.00: failed command: WRITE FPDMA QUEUED ata3.00: cmd 61/38:50:c8:3c:96/13:00:05:00:00/40 tag 10 ncq 2519040 out res 40/00:98:38:ef:7a/00:00:00:00:00/40 Emask 0x10 (ATA bus error) ata3.00: status: { DRDY } ata3.00: failed command: WRITE FPDMA QUEUED ata3.00: cmd 61/00:58:00:50:96/18:00:05:00:00/40 tag 11 ncq 3145728 out res 40/00:98:38:ef:7a/00:00:00:00:00/40 Emask 0x10 (ATA bus error)\[/code\]

From the confidential(!) datasheets, the core voltage of the `ASM1061` controller should be `1.25V` (Min. `1.20V`; Max. `1.30V`). However, the resistor values on the PCB (`R4 = 40.2kΩ`, `R15 = 30kΩ`) suggests it was configurated as `1.05V` for some unknown reason: `0.6 / 40.2k * (30k + 40.2k) = 1.05V` [Resistor 59C](http://kiloohm.info/eia96-resistor/59C)

After change both of the resistors to `20kΩ` (core voltage `1.20V`), the card works a lot more reliably, no more crashes happened yet.

Datasheets: [ASM1061\_DataSheet\_R1\_8](http://ftp.mqmaker.com/WiTi/Docs/Hardware/ASM1061_Data%20Sheet_R1_8.pdf) [ASM1061 reference schematic](https://wenku.baidu.com/view/0bcccc12e53a580217fcfe60.html) [APE1501 bulk regulator](http://www.a-power.com.tw/files/ap_productdata/pd_file/ape1501y5.pdf)