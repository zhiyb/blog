---
title: Nosumor
tags:
  - C/C++
  - Embedded
  - MicroSD
  - MSC
  - Nosumor
  - SDIO
  - USB
id: '1251'
categories:
  - - uncategorized
date: 2018-01-10 00:25:02
---

\* Implemented USB Mass Storage Class (MSC) for Micro SD card reader
<!-- more -->
Bulk-only transport was used for USB MSC. The protocol consists of 3 stages: Command Block Wrapper (CBW) `OUT`, Data Transport `OUT/IN`, Command Status Wrapper (CSW) `IN`. Multiple `LUNs` can be used for different disks: ![47677f5 Bootloader USB interfaces](47677f5-bootloader-usb-interfaces.png) [Mass Storage Bulk Only 1.0](http://www.usb.org/developers/docs/devclass_docs/usbmassbulk_10.pdf) [723a87a8ea8f63b83a0cf610811ad4661ba7b3a5](https://github.com/zhiyb/Nosumor/commit/723a87a8ea8f63b83a0cf610811ad4661ba7b3a5)

The USB MSC is basically an encapsulation layer of other storage protocols, such as the wildly used Small Computer System Interface (SCSI). The [SCSI documentations](http://www.t10.org/drafts.htm) are only available to t10 members or through purchasing. Fortunately, [Seagate has a documentation](https://www.seagate.com/staticfiles/support/disc/manuals/scsi/100293068a.pdf) with all necessary SCSI commands for a removable storage device. Together with [SCSI codes on t10.org](http://www.t10.org/lists/1spc-lst.htm), I am able to implement a working layer of SCSI for the Micro SD card slot as a card reader. [afa9107afffeaa8fed1b1146fca1f6f4fb789a7a](https://github.com/zhiyb/Nosumor/commit/afa9107afffeaa8fed1b1146fca1f6f4fb789a7a)

The essential SCSI commands are: `TEST UNIT READY`, `INQUIRY`, `REQUEST SENSE`, `READ CAPACITY (10)`, `READ (10)` and `WRITE (10)`

To support removable devices, the operating system will continuously pull its status through the TEST UNIT READY command: ![131ff4d Hot plug during write](131ff4d-hot-plug-during-write.png) [30ecaf8ece2857c9469894619d860baed19539f0](https://github.com/zhiyb/Nosumor/commit/30ecaf8ece2857c9469894619d860baed19539f0)

For some unknown reason, Microsoft Windows (10) seems only supports small disks (less than 1MiB capacity) only if it is marked as removable. Linux has better support for these disks.