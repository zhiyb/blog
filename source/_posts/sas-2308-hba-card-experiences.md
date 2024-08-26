---
title: SAS 2308 HBA card experiences
tags:
  - NAS
  - RAID
  - SAS
  - zfs
id: '1466'
categories:
  - - Linux
date: 2022-12-10 16:46:39
---

Got another SAS 2308 card from Amazon. A.k.a. SAS 9217-8i

Now this is a proper SAS HBA card that can be used for software RAID (e.g. zfs).
The raw hard drive is passthrough to the host system, no hidden space, S.M.A.R.T. available too.
<!-- more -->
## Firmware flashing to IT mode

My card arrived with IR mode firmware pre-installed.
Since I intend on only using it as HBA card like JBOD, for software RAID zfs, I do not need any hardware RAID functionalities.
The IT mode firmware is much more suited for this purpose, slimmer layers and better performance.
However, flashing it to IT mode proves to be a lot more complicated than I first anticipated.

### Firmware download

On Broadcom support download, search for:
**Group:** Storage Adapters, Controllers, and ICs, **Family:** Legacy Host Bus Adapters, **Product:** SAS 9217-8i Host Bus Adapter
https://www.broadcom.com/support/download-search

Need firmware package:
9217-8i\_Package\_P20\_IR\_IT\_Firmware\_BIOS\_for\_MSDOS\_Windows
Firmware installer:
Installer\_P20\_for\_UEFI
Management utility:
SAS2IRCU\_P20

### UEFI flashing

#### Preparation

This is what finally worked for me.
Since I was trying to flashing it with a pretty modern motherboard (Asus Z690 Hero), I have to flash it with a UEFI utility.

To enter UEFI shell on my ASUS motherboard, I need to prepare a UEFI shell application, as it seems the motherboard doesn't have it.
Following the links at https://wiki.archlinux.org/title/Unified\_Extensible\_Firmware\_Interface
I have tried UEFI shell v2, but it isn't supported by the UEFI installer. It simply complains and exits:
_Application not started from Shell_
UEFI shell v1 works. https://github.com/tianocore/edk2/tree/UDK2018/EdkShellBinPkg
Rename it to shell.efi as required by my motherboard.

So, prepare the following files on a FAT32 disk:

\# Firmware binary
9207-8.bin
# SAS BIOS binary
mptsas2.rom
# UEFI installer application
sas2flash.efi
# UEFI shell application
shell.efi

#### Firmware flashing

Reboot and enter UEFI shell from BIOS.

Find the prepared disk:

![](screenshot_20221210_165003.png.jpg)

Find the SAS controller card:

![](screenshot_20221210_165030.png.jpg)

Erase card's flash:

![](screenshot_20221210_165127.png.jpg)

Flash new firmware:

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_165209.png.jpg?w=1024)

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_165250.png.jpg?w=1024)

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_165321.png.jpg?w=1024)

Check new firmware is loaded:

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_165356.png.jpg?w=1024)

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_165510.png.jpg?w=1024)

### DOS flashing

A dos executable is available, however it seems incompatible with modern UEFI motherboards.
I tried to run it within FreeDOS, and it fails with:
_Failed to initialize PAL._

### Windows flashing

A windows executable is also available.

It is able to find and manage the card:

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/image-4.png?w=1024)

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/image-5.png?w=621)

However, the Windows executable is unable to flash from IR mode firmware to IT mode firmware.
It must be done at a pre-boot environment.

Maybe it can be used to update firmware.
But seeing the latest firmware is from 2016, it seems not needed...

## Thermal concerns

The SAS card is intended for use in a server chassis with constant airflow, so it only has a minimal heatsink.
But on a personal system, it may be undesirable due to noise.

So, be prepared to strap a small fan on top of the card's heatsink, as otherwise it runs very hot.

Unfortunately the IT mode firmware does not support reporting card's temperature...

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/screenshot_20221210_172423.png.jpg?w=1024)
