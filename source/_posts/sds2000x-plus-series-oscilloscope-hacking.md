---
title: SDS2000X Plus series oscilloscope hacking
tags:
  - Oscilloscope
  - Siglent
  - Zynq
id: '1390'
categories:
  - - c-c
    - Embedded
  - - hdl
    - FPGA
  - - Linux
date: 2021-06-26 15:42:07
---

I got a SDS2104X+ oscilloscope now, crossing off one item that has been on my wish list for so long...
<!-- more -->
As with other things, I'd like to see how it works internally.  
Especially because it's using Zynq platform, running Linux, that should be pretty fun to mess around with.

However I also don't want to disassemble it, as it is a very expensive toy, does have a warranty void sticker, and most importantly the calibration might be invalidated.  
Dave from EEVblog already did a teardown on one of this series oscilloscopes here:

https://youtu.be/HxzQS-Bn2R0

Searching around the internet, again from EEVblog forum, I found this post:  
See reply #826, [Siglent SDS2000X Plus - Page 34 (eevblog.com)](https://www.eevblog.com/forum/testgear/siglent-sds2000x-plus-coming/825/)

The most important thing is, that shell script `siglent_device_startup.sh` at top-level directory of USB drive does get executed on boot 😏.  
Instead of guessing root password, I used `-l` parameter on `telnetd` to skip that:

\# cat siglent\_device\_startup.sh
telnetd -l /bin/sh

Connect the oscilloscope to ethernet, power on, and I can telnet to it on the default telnet port (23), drops me directly to a root shell. Note it is terribly unsafe to use a telnet connection, especially without any authentication and with default port number. Make sure this is on a trusted local network.

Anyway, now I can take a look freely around its Linux system 😆.

* * *

Some interesting things I noted while playing around:

```
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
111/tcp  open  rpcbind
5900/tcp open  vnc
(And port 5024 for SCPI commands)
```

```
/ # cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00780000 00020000 "fsbl"
mtd1: 00400000 00020000 "kerneldata"
mtd2: 00080000 00020000 "device-tree"
mtd3: 00500000 00020000 "Manufacturedata"
mtd4: 00500000 00020000 "reserved1"
mtd5: 01400000 00020000 "rootfs"
mtd6: 00a00000 00020000 "firmdata0"
mtd7: 06e00000 00020000 "siglent"
mtd8: 05a00000 00020000 "datafs"
mtd9: 00400000 00020000 "reserved2"

/ # df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                30.6M     30.6M         0 100% /
devtmpfs                234.3M         0    234.3M   0% /dev
none                    242.4M      8.0K    242.4M   0% /tmp
ubi1_0                   91.7M     78.4M     13.4M  85% /usr/bin/siglent
ubi2_0                    5.7M     68.0K      5.6M   1% /usr/bin/siglent/firmdata0
ubi0_0                   73.8M    240.0K     73.6M   0% /usr/bin/siglent/usr
/dev/sda1                14.9G    147.8M     14.7G   1% /usr/bin/siglent/usr/mass_storage/U-disk0
```

```
/ # cat /proc/cpuinfo
processor       : 0
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 1528.62
Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpd32
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc09
CPU revision    : 0

processor       : 1
...(identical)

Hardware        : Xilinx Zynq Platform
Revision        : 0003
Serial          : 0000000000000000
```

```
# cat /proc/cmdline
console=ttyPS0,115200 root=/dev/mtdblock5 rootfstype=cramfs init=/linuxrc earlyprintk uboot_version=05
```

Oscilloscope application `/usr/bin/siglent/main.app` uses Qt5:  
`/usr/bin/siglent/lib/qt/lib/libQt5Core.so.5.6.2`

* * *

Web server is `lighttpd`, with `php-cgi` enabled.  
Web root is at `/usr/bin/siglent/config/www`.

More interestingly, inside that directory, there is a symlink:  
`web_img -> /usr/bin/siglent/usr`

And, as you may have noticed earlier, USB drive is mounted below there.  
Which means, I can put php webpages on my USB drive, and access it directly though a web browser.  
Even `shell()` is enabled in php:

![](https://zhiyb.wordpress.com/wp-content/uploads/2021/06/image.png?w=563)

* * *

Writing to `/dev/fb0` does write to the screen, bit depth is interestingly 32-bit.  
I wrote some simple C code to mmap and draw a test pattern to the framebuffer, and got this:

![](https://zhiyb.wordpress.com/wp-content/uploads/2021/06/image-1.png?w=1024)

Well I certainly did not draw the channel 1 waveform, and it still continues to update. Looks like the waveforms are composed with the framebuffer inside FPGA before going to the screen, very interesting..  
The measurements region at the bottom of the screen keeps getting updated by the oscilloscope UI application, even inside the help browser where it's not visible.  
Killing `main.app` does stop the bottom measurements refreshing, but waveform still updates.

Since the waveform can be obscured by UI elements, the composition must be controllable. And that's exactly why there is an alpha channel in the 32-bit depth framebuffer. Setting the alpha channel to `0xff` makes the framebuffer overwrites the waveform.  
There is alpha blending happening too, e.g. the masked wave regions in zoom mode. The colour blending seems a bit strange though.

I suspect `/dev/siglent_vnc` might be the output of the screen composition, as the VNC server does show the waveforms.