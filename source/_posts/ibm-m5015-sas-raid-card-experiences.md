---
title: IBM M5015 SAS RAID card experiences
tags:
  - NAS
  - RAID
  - SAS
id: '1445'
categories:
  - - Linux
date: 2022-12-10 16:09:00
---

Got a M5015 RAID card from Amazon. A.k.a. 9260-8i, LSI 46M0851, LSI9261.

It doesn't support JBOD mode, can only be used as RAID card.
Cannot be used as SAS HBA for software raid like zfs.

So returned it.
<!-- more -->
Anyway, I updated the firmware for this card.
Also have some notes on how to manage it as a RAID card.

For tools, on Broadcom support downloads, search for
**Group:** Legacy Products, **Family:** Legacy RAID Controllers, **Product:** MegaRAID SAS 9260-8i
https://www.broadcom.com/support/download-search

Need the latest firmware (e.g. mr2108fw.rom), and MegaCLI.
And perhaps the Windows drivers. Linux (debian bookworm) seems to support it natively.

## Firmware update:

Some useful commands:

```
MegaCli64.exe -adpcount
MegaCli64.exe -adpallinfo -aall
MegaCli64.exe -adpfwflash -f mr2108fw.rom -aALL
```

![](image.png)

![](image-1.png)

![](image-2.png)

## Workaround, set all disks are separate RAID0 pool

Commands:

```
megacli -adpsetprop CoercionMode -0 -a0
MegaCli64.exe -CfgEachDskRaid0 WT RA Cached NoCachedBadBBU -a0
```

However, this will not work with existing software RAID installations.
Even with CoercionMode set to disabled, the controller still takes up quite some space at the end of disk, presumably for RAID stripping. (around 1GB in my case)

As a result, the backup GPT will be corrupted, data at the end of disk likely corrupted too.

If using completely empty new hard drives, this might be fine.
We'll just have a bit less space available on each drive.

It is definitely not ideal. Also zfs likes to take whole raw disks.
The RAID controller also doesn't expose S.M.A.R.T. to the host system, making monitoring the hard drives much more difficult.

![](https://zhiyb.wordpress.com/wp-content/uploads/2022/12/image-3.png?w=960)

It is a proper hardware RAID controller.
Perhaps it could be useful for Windows partitions, the RAID configuration will be transparent.
(Although my Windows install is on NVMe drives, so won't work for me anyway.)
