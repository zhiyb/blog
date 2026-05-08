---
title: Reverse engineering NOAHOS-V2 on NP7000
date: 2026-05-06 22:14:01
tags:
    - Noah
    - Noahedu
    - NOAHOS
    - NP7000
    - qemu
    - Ingenic
    - JZ4755
    - MIPS
---

By group chat requests, I started looking at emulating various e-learning devices manufactured by Noahedu.

NP7000 is a particularly interesting device, as it runs a fully custom NOAHOS operating system that I was always interested in reverse engineering, and it uses Ingenic SoC with available documentation and I have prior experiences working with and emulating them.

Emulation is done using qemu, see:
https://github.com/OpenNoah/OpenNoah.github.io/wiki/NP7000-Emulation

<!-- more -->

# Identifying SoC chip model

This is rather easy with NP7000.

Simply download one of its full (TF-card) firmware upgrade package, still available here:
https://downloads.youxuepai.com/source/list.shtml#107-2361-0-0-0-0-0-0-

Extract it, the V3.2 firmware package consists of 4 files:

```shell
$ ls
'NP7000(INAND V3.2).idx'  'NP7000(INAND V3.2).mop'
'NP7000(INAND V3.2).mo1'  'NP7000(INAND V3.2).mo2'
```

Open the `.idx` file in a hex editor, then it is immediately visible:

```shell
$ hexedit NP7000\(INAND\ V3.2\).idx
00000000   4E 4F 41 48  20 54 46 2D  43 41 52 44  2F 69 4E 41  4E 44 2D 46  4C 41 53 48  20 49 4D 41  47 45 20 46  NOAH TF-CARD/iNAND-FLASH IMAGE F
00000020   49 4C 45 20  56 32 2E 30  30 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ILE V2.00.......................
00000040   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................................
00000060   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................................
00000080   4A 5A 34 37  35 35 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  JZ4755..........................
000000A0   4E 50 37 30  30 30 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  NP7000..........................
```

Extracting ASCII strings from `.idx` and `.mop` also shows JZ4755:

```shell
$ strings NP7000\(INAND\ V3.2\).idx | grep JZ4
JZ4755
$ strings NP7000\(INAND\ V3.2\).mop | grep JZ4
../../Peripheral/Clock/./JZ4750/DrvClock.c
../../Peripheral/iNand/./JZ4755/InandCtrl.c
../../Peripheral/Power/./JZ4750/DrvPower.c
../../Peripheral/RTC/./JZ4755/DrvRtc.c
../../Peripheral/SD/./JZ4755/SDControl.c
../../Peripheral/Touch/./JZ4755/DrvTouch_np7000.c
../../Peripheral/UART/./JZ4750/DrvUart.c
JZ4755
../../Peripheral/Clock/./JZ4750/DrvClock.c
../../Peripheral/iNand/./JZ4755/InandCtrl.c
../../Peripheral/Power/./JZ4750/DrvPower.c
../../Peripheral/RTC/./JZ4755/DrvRtc.c
../../Peripheral/SD/./JZ4755/SDControl.c
../../Peripheral/Touch/./JZ4755/DrvTouch_np7000.c
../../Peripheral/UART/./JZ4750/DrvUart.c
```

So, most definitely the SoC chip is Ingenic JZ4755.

Here are some datasheets relevant to the JZ4755 SoC:
https://opennoah.github.io/datasheet/JZ4755_DS.PDF
https://opennoah.github.io/datasheet/JZ4755_pm.pdf
https://opennoah.github.io/datasheet/XBurst1_Core_PM.pdf
https://opennoah.github.io/datasheet/X1000_M200_XBurst_ISA_MXU_PM.pdf

# MIPS basic

A couple of basic MIPS knowledge you should know.

## `kseg` memory mapping

https://training.mips.com/basic_mips/PDF/Memory_Map.pdf

![](images/noah/Screenshot_2026-05-06_234852.png)

Regions of size `0x20000000` starting from offset `kseg0 0x80000000` and `kseg1 0xa0000000` both maps to hardware physical address offset from `0x00000000`.

In the JZ4755 datasheet, the register addresses are shown in the physical address space like `0x1002101c`.
To access the register from kernel driver code, we use the `kseg1` address space, as that region is uncached.
So, for example, hardware register at `0x1002101c` would be accessed using address `0xa0000000 + 0x1002101c = 0xb002101c`.

# First stage boot loader

According to JZ4755 programming manual, section 29 XBurst Boot ROM Specification, JZ4755 supports booting from either NAND flash or SD card on `MSC0`.

In either cases, it loads 8 KiB data into internal SRAM offset `0x80000000`.

Section 29.2 Boot Sequence:
NOTE: The JZ4755 internal SRAM is 16KB, its address is from `0x80000000` to `0x80004000`.
(Actually I suspect the SRAM may be implemented simply by re-using the CPU data cache?)

Take a look at the `.mop` file:

```shell
$ hd NP7000\(INAND\ V3.2\).mop | less
00000000  00 80 1f 3c 00 00 ff 27  00 90 80 40 00 98 80 40  |...<...'...@...@|
00000010  00 60 08 40 00 10 09 3c  04 04 29 35 24 40 09 01  |.`.@...<..)5$@..|
...
00001bd0  00 1b b7 00 00 36 6e 01  00 03 00 b2 00 00 00 00  |.....6n.........|
00001be0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00001bf0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00002000  00 80 1f 3c 00 00 ff 27  00 90 80 40 00 98 80 40  |...<...'...@...@|
00002010  00 60 08 40 00 10 09 3c  04 04 29 35 24 40 09 01  |.`.@...<..)5$@..|
```

Data starting from offset `0x00001bf0` up to 8 KiB offset `0x00002000` are all `0xff`.
So, this data is very likely the first stage boot loader.

Also notice that the first 12 bytes of this data section does not have repeating `0x55` bytes, so this cannot be a NAND boot sequence, content does not match:

![](images/noah/Screenshot_2026-05-06_230604.png)

So, we know this must be loaded from `MSC0` into offset `0x80000000`, and CPU entry is also at offset `0x80000000`.

We have seen similar implementations from Noahedu's previous devices, e.g. the NP6800 is also using a flash controller chip to use a NAND chip as a SD card as its internal storage:
https://github.com/OpenNoah/OpenNoah.github.io/wiki/NP6800+-Gallery

Extract this data section to a separate file using `dd`:

```shell
dd if="NP7000(INAND V3.2).mop" of=boot_1st.bin bs=1024 count=8
```

The next 8 KiB section has exactly the same data as the first one.
The NAND boot sequence does support having a backup section like this, in case data from the first section was corrupted. However, as the device is booting from SD card, this is unused.

Load it into [Ghidra](https://github.com/nationalsecurityagency/ghidra).
Set language as MIPS, default variant, 32-bit, little endian, eabi compiler.
In options, set base address to `0x80000000`.

Immediately, disassembled code looks reasonable, there are a few prints, we can guess some function names:

![](images/noah/Screenshot_2026-05-06_232510.png)

(All the custom names are added by me manually.)

Looking at the print functions, we can find that it eventually writes to register address `0xb0031000`, that means it prints logs to `UART1`.

Usually the first stage boot loader configures SDRAM controller, so that it can load much more data from non-volatile storage and start executing the next stage boot code.

Look inside the `mmc0_boot(0x20,0x3e0,0xa0600000)` function:

![](images/noah/Screenshot_2026-05-06_233117.png)

Pretty standard SD card initialisation code.

SD card specification can be downloaded from:
https://www.sdcard.org/downloads/pls/
Part 1 Simplified Physical Layer Simplified Specification

A few notable lines from the disassembled code:

```c
// Inside mmc0_boot(0x20,0x3e0,0xa0600000)
// CMD16 SET_BLOCKLEN = 512 bytes
mmc0_cmd(0x10,0x200,0x401,1);
// Register 0x1002101C MSC0.NOB = param_2 = 0x3e0: load 992 blocks
_DAT_b002101c = param_2;
// CMD18 READ_MULTIPLE_BLOCK address = param_1: starting offset 0x20 blocks
mmc0_cmd(0x12,param_1,0x409,1);
```

```c
// Back to the entry() code
// The third parameter must be the read data destination address
mmc0_boot(0x20,0x3e0,0xa0600000);
// Then jump to address 0x80600000
(*(code *)&SUB_80600000)();
```

So that means, it loads `0x3e0 * 512 = 496 KiB` data from `MSC0` offset `0x20 * 512 = 16 KiB` into `0x80600000`.

# Second stage boot loader

Extract the section stage data from the `.mop` file.

```shell
dd if="NP7000(INAND V3.2).mop" of=boot_2nd.bin bs=1024 skip=16 count=496
```

```shell
$ hd NP7000\(INAND\ V3.2\).mop | less
00002000  00 80 1f 3c 00 00 ff 27  00 90 80 40 00 98 80 40  |...<...'...@...@|
00002010  00 60 08 40 00 10 09 3c  04 04 29 35 24 40 09 01  |.`.@...<..)5$@..|
...
000154a0  00 00 00 b2 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000154b0  00 00 00 00 00 00 00 00  ff ff ff ff ff ff ff ff  |................|
000154c0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
00080000  84 80 1a 3c 90 f5 5a 27  00 60 1b 40 00 00 5b af  |...<..Z'.`.@..[.|
00080010  00 40 1a 40 21 d8 40 03  82 de 1b 00 1b 00 60 17  |.@.@!.@.......`.|
```

All `0xff` from `0x000154c0` up to `0x00080000` (512 KiB offset).
So, this section also looks plausible.
Again, load into Ghidra, mips 32-bit little-endian but at offset `0x80600000` instead.

Interesting code starting around offset `0x80605038`:

![](images/noah/Screenshot_2026-05-07_002849.png)

Function at `0x80604e5c`:

![](images/noah/Screenshot_2026-05-07_003530.png)

Function at `0x80603fbc`:

![](images/noah/Screenshot_2026-05-07_003633.png)

Reading the code, again we can find the `mmc0_read()` code to read data from `MSC0`, then calculate the parameters of the `load_image()` are using 16 KiB block sizes.

Register address `0xb0010400` is the `PIN` input register of GPIO group `PE`.

We can read from the code that it supports a few different booting modes:
1. Normal boot, starting block offset `0x180 = 6 MiB`, max number of blocks `0x680 = 26 MiB`
2. Update utilities, starting block offset `0x20 = 512 KiB`, max number of blocks `0x180 = 6 MiB`, enabled by `PE4 == 0`
3. Test utilities, same starting block offset `0x20 = 512 KiB`, max number of blocks `0x180 = 6 MiB`, enabled by `PE5 == 0`
4. `MSC1` SD card boot from `noahos.img` file, enabled by `PE30 == 0`

And we can read that load address is `0x80000000`, entry address is `0x80000400`.
Number of blocks is recorded as 4 bytes located at offset `0x01f0` in the first block.

However, looking at `.mop` file offset `0x00600000 = 6 MiB`, the data is not on a clear boundary:

```shell
005FFFC0   07 00 65 88  40 02 A2 27  38 02 A7 A3  04 00 65 98  03 00 46 A8  0B 00 67 88  00 00 46 B8  44 02 A2 27  ..e.@..'8.....e...F...g...F.D..'
005FFFE0   0F 00 68 88  03 00 45 A8  08 00 67 98  00 00 45 B8  0C 00 68 98  48 02 A2 27  03 00 47 A8  4C 02 A6 27  ..h...E...g...E...h.H..'..G.L..'
00600000   00 00 47 B8  03 00 C8 A8  21 28 43 02  03 00 A7 88  10 00 69 90  00 00 A7 98  07 00 A3 88  58 02 A2 27  ..G.....!(C.......i.........X..'
00600020   00 00 C8 B8  50 02 A9 A3  04 00 A3 98  03 00 47 A8  0B 00 A8 88  00 00 47 B8  5C 02 A2 27  0F 00 A9 88  ....P.........G.......G.\..'....
```

And if I try to boot the `.mop` file in `MSC0` as-is, it simply gets stuck after these prints:

```log
=======================================
     NOAHOS-V2 BOOT LOADER .....
=======================================
RTC_HRCR: 0x0
eSD storage device found!
LOADER: 0x0
eSD storage device found!
IMG SIZE: 0xa3a802c8
```

We can confirm that the bytes at offset `0x006001f0` is indeed `0xa3a802c8`.

# Special kernel

So, we need to look into the firmware update utility to understand how to correctly parse the firmware upgrade package.

Fortunately booting the special function kernel looks much more promising:

```log
=======================================
     NOAHOS-V2 BOOT LOADER .....
=======================================
RTC_HRCR: 0x0
eSD storage device found!
LOADER: 0x0
eSD storage device found!
IMG SIZE: 0x200000
NOAHOS TEST UTILITIES LOADED OK!


=================================================
             NOAHOS-V2 BOOTING...
               BOOT FROM NAND
=================================================
OSC = 24MHz    PLL = 384MHz
CCLK = 384MHz      HCLK = 128MHz
PCLK = 128MHz    MCLK = 128MHz
SDRAM size: 64MB
NOAHOS IMG SIZE = 0x83c000
Time slice is 100 msec
Device init: CLOCK
IRQ22 attached! priority=10
Device init: UART
IRQ8 attached! priority=10
Device init: LCDC
LCDC: 34MHz
key : 0xd
Device init: NANDC
DRV> nand share mode
DRV> NAND ID: 0xff-0xff-0xff-0xff

!!! KERNEL PANIC:
DRV> unidentify nand flash
```

Ok, for some reason it seems to have a NAND driver and is trying to read from non-existent NAND, so that failed.

So, extract the kernel binary from offset `0x00080000` (512 KiB) and size `0x00200000` (2 MiB), then load it into Ghidra at offset `0x80000000`. Create a function at offset `0x80000400` as that is the entry address.

Again reading the function at `0x8000237c`:

![](images/noah/Screenshot_2026-05-07_012418.png)

Satisfying condition `(0xb0010200 & 0x80000000) == 0` i.e. GPIO `PC31 == 0` makes it boot from `MSC0` SD card instead of NAND. So that's what Noahedu means by "INAND".

Fixing that in the emulator, now we get:

```log
=======================================
     NOAHOS-V2 BOOT LOADER .....
=======================================
RTC_HRCR: 0x0
eSD storage device found!
LOADER: 0x0
eSD storage device found!
IMG SIZE: 0x200000
NOAHOS UPDATE UTILITIES LOADED OK!


=================================================
             NOAHOS-V2 BOOTING...
               BOOT FROM INAND
=================================================
OSC = 24MHz    PLL = 384MHz
CCLK = 384MHz      HCLK = 128MHz
PCLK = 128MHz    MCLK = 128MHz
SDRAM size: 64MB
NOAHOS IMG SIZE = 0x83c000
Time slice is 100 msec
Device init: CLOCK
IRQ22 attached! priority=10
Device init: UART
IRQ8 attached! priority=10
Device init: LCDC
LCDC: 34MHz
key : 0x1b
Device init: NANDC
Device init: INAND
Device init: SDC
Device init: RTC
---BootType: 0x0
REG_RTC_RGR=0x81007fff, RSR=0x69fbdd64
IRQ6 attached! priority=8
RTC reset time
----RTC: 2010-06-01 12:00:00----
Device init: KEYBOARD
Device init: TOUCH
IRQ18 attached! priority=5
Device init: WATCHDOG
Device init: INDICATOR
Device init: POWER-MAN
IRQ206 attached! priority=0
IRQ182 attached! priority=0
Device init: MEMORY
Device init: USB-SLAVE
IRQ27 attached! priority=7
Device init: CODEC
Kernel schedule start
GUI inited ok
key : 0x1b
DRV> eSD iNAND
DRV> Use 4-bit bus width
DRV> INAND: 8192M
Logo package size:1777640
KERNEL: Create app: kernel
Bios Utility Menu
60323, 231715, 119590
BIOS GUI: 2, User Mode
<devcheck><4>DevCheck.c(399) : minute:0, AutoOff disable
<devcheck><4>DevCheck.c(536) : BackLight fade disable
default_print_level:3
key : 0x1b
key : 0x1b
key : 0x1b
key : 0x1b
desk:WM_CHARGE_DO
desk:WM_CHARGE_DONE
key : 0x1b
key : 0x1b
```

Looking at strings found by Ghidra, we can find strings that got printed to UART like:
`Bios Utility Menu`

![](images/noah/Screenshot_2026-05-07_013913.png)

To decode Chinese strings, set default charset to `GBK` or `GB2312`, then set the relevant data section data type as `TerminatedCString`:

![](images/noah/Screenshot_2026-05-07_014658.png)

It is strange that Ghidra cannot find any references to these strings, there must be further logic to be understood.
There is likely another binary executable embedded in there and loaded by the kernel, especially these log prints look suspicious:

```log
Logo package size:1777640
KERNEL: Create app: kernel
```

Looking back at the `.mop` file data, I found that earlier offset `0x00200000` appears to be a starting location of a new data section, as it's a new set of data just after a bunch of `0xff` bytes:

```log
001FFFC0   FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  ................................
001FFFE0   FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  FF FF FF FF  ................................
00200000   E0 FF BD 27  14 00 B1 AF  10 00 B0 AF  21 88 E0 00  18 00 BF AF  20 3E 10 0C  21 80 C0 00  21 20 00 02  ...'........!....... >..!...! ..
00200020   C4 00 10 0C  21 28 20 02  CB 41 10 0C  00 00 00 00  D0 41 10 0C  21 20 40 00  0A 00 10 08  00 00 00 00  ....!( ..A.......A..! @.........
```

Load it into Ghidra with address offset set to 0 as I did not know the correct value yet, I can see some failed references:

![](images/noah/Screenshot_2026-05-07_021312.png)

Based on the function addresses it references, it seems very likely that the load address should be `0x00400000` instead. And indeed, after fixing the address, Ghidra disassembly looks much better:

![](images/noah/Screenshot_2026-05-07_021634.png)

And there are now references to these strings we found earlier:

![](images/noah/Screenshot_2026-05-07_021810.png)

After some more painful reverse engineering of that executable, I finally located a few functions used for firmware upgrade, at `0x00406990`, `0x00407cf4` and `0x00407448`.

![](images/noah/Screenshot_2026-05-07_023108.png)
![](images/noah/Screenshot_2026-05-07_023334.png)

From these functions I understood how it parses the `.idx` index file, and `.mop`, `.mo1` etc. data files.
It can take an arbitrary number of data files, it simply overwrites the file suffix with a number.

Here are the most important header fields, all integers are unsigned 32-bit little-endian:

![](images/noah/np7000_idx_en.svg)

From these information, finally I can re-construct a full system image.

Main kernel now boots, except for a segmentation fault in QEMU because of a bug in emulating the Ingenic Q16ADD MXU1 instruction, which was easy to fix.

```log
=======================================
     NOAHOS-V2 BOOT LOADER .....
=======================================
RTC_HRCR: 0x0
eSD storage device found!
LOADER: 0x0
eSD storage device found!
IMG SIZE: 0x2ec000
NOAHOS LOADED OK!


=================================================
             NOAHOS-V2 BOOTING...
               BOOT FROM INAND
=================================================
OSC = 24MHz    PLL = 384MHz
CCLK = 384MHz      HCLK = 128MHz
PCLK = 128MHz    MCLK = 128MHz
SDRAM size: 64MB
NOAHOS IMG SIZE = 0xb26000
Time slice is 100 msec
Device init: CLOCK
IRQ22 attached! priority=10
Device init: UART
IRQ8 attached! priority=10
Device init: LCDC
LCDC: 34MHz
Device init: NANDC
Device init: INAND
Device init: SDC
Device init: RTC
---BootType: 0x0
REG_RTC_RGR=0x81007fff, RSR=0x69fbf887
IRQ6 attached! priority=8
RTC reset time
----RTC: 2010-06-01 12:00:00----
Device init: KEYBOARD
Device init: TOUCH
IRQ18 attached! priority=5
Device init: WATCHDOG
Device init: INDICATOR
Device init: POWER-MAN
IRQ206 attached! priority=0
IRQ182 attached! priority=0
Device init: MEMORY
Device init: USB-SLAVE
IRQ27 attached! priority=7
Device init: CODEC
Kernel schedule start
GUI inited ok
DRV> eSD iNAND
DRV> Use 4-bit bus width
DRV> INAND: 8192M
Logo package size:104840
KERNEL: Create app: kernel
```

# Syscall, syscall and more syscalls

Whilst looking at the firmware update program, I found that it does not operate on the storage device directly.
Instead, it does a bunch of syscalls with the file `'\\DEVICE\BURNFILE'`.
Inferring from the usage of the syscall functions, they look like file access functions e.g. `fopen`, `fwrite`, `fread`, `fclose`.

![](images/noah/Screenshot_2026-05-07_220302.png)

The syscall function itself does not do any operation on the registers, so Ghidra is unable to infer things like number of function arguments and return type. It simply does a syscall then nothing else.

![](images/noah/Screenshot_2026-05-07_220554.png)

I was also looking at LCD screen operations, since I didn't have it working in the emulator yet, that's the main thing I was looking to fix.
Again, following the text strings to LCD print functions, the main functionalities are hidden behind syscalls.

![](images/noah/Screenshot_2026-05-07_225438.png)

![](images/noah/Screenshot_2026-05-07_225538.png)

![](images/noah/Screenshot_2026-05-07_225608.png)

So now, we need to go back to the kernel to understand what the syscalls are actually doing.

Syscalls should trigger an interrupt in the kernel, details about interrupts are described in the CPU programming manual, section 3 Exceptions.
The Ingenic documentation isn't very clear, I believe `Status.BEV == 0` and `Cause.IV == 0` for the kernel, so exception handler entry address should be at `0x80000180`.

![](images/noah/Screenshot_2026-05-07_231809.png)

Back to the kernel and start disassemble at that address, an exception handler appears:

![](images/noah/Screenshot_2026-05-07_232703.png)

Following the syscall handler logic, it uses an array of arrays of pointer to look up the function to process syscalls.

![](images/noah/Screenshot_2026-05-07_233106.png)

![](images/noah/Screenshot_2026-05-07_233234.png)

Finally, we can figure out what operations are actually happening.

# LCD framebuffer and Ingenic MXU

The functions to put some text on the screen are quite complicated to analyse, as it supports multiple different encodings and fonts.

![](images/noah/Screenshot_2026-05-07_234712.png)

Eventually I figured out that syscall `0x4002` is the function to update LCD framebuffer after screen updates all completed.

Being able to use QEMU to run the actual code, set breakpoints, single step through instructions and looking at memory content is super helpful on doing the analysis. I can also add debug logs on LCD controller emulation to show the physical addresses of the framebuffer:

```log
2026-05-07T23:18:15.076780Z ingenic_lcd_enable 1
2026-05-07T23:18:15.076782Z ingenic_lcd_mode 800x480 mode=4565
2026-05-07T23:18:15.077742Z ingenic_lcd_schedule 611160864
2026-05-07T23:18:15.077882Z ingenic_lcd_desc dda=0x741030 dsa=0x7fe000 fid=0xbeafbeae cmd=0x10000008 offs=0x0 pw=0x0 cnum=0x0 size=0x0
2026-05-07T23:18:15.077888Z ingenic_lcd_desc dda=0x741030 dsa=0x7fe040 fid=0xbeafbead cmd=0xbb80 offs=0x0 pw=0x0 cnum=0x0 size=0x0
2026-05-07T23:18:15.077890Z ingenic_lcd_desc dda=0x741010 dsa=0x742000 fid=0xbeafbeaf cmd=0x2ee00 offs=0x0 pw=0x0 cnum=0x0 size=0x0
2026-05-07T23:18:15.078896Z ingenic_lcd_read *0x30 = 0x2000000a
2026-05-07T23:18:15.078906Z ingenic_lcd_write *0x30 = 0x2000000a
```

Inside syscall `0x4002`, there is a `memcpy`-looking function at `0x80001a98`:

![](images/noah/Screenshot_2026-05-08_002039.png)

Using `gdb` attached to QEMU, we can check its three parameters in registers `a0`, `a1` and `a2`:

```log
(gdb) b *0x80001a98
Breakpoint 1 at 0x80001a98
(gdb) c
Continuing.

Breakpoint 1, 0x80001a98 in ?? ()
(gdb) i r
          zero       at       v0       v1       a0       a1       a2       a3
 R0   00000000 00000000 00000000 8059e000 80742000 8059e000 00000640 80742000
            t0       t1       t2       t3       t4       t5       t6       t7
 R8   00000000 00000640 00000000 ffffffff ffffffff ffffffff ffffffff ffffffff
            s0       s1       s2       s3       s4       s5       s6       s7
 R16  80742000 8059e000 00000000 00000001 00000640 805a0000 000001e0 80659800
            t8       t9       k0       k1       gp       sp       s8       ra
 R24  00000000 0040122c 8083f5b0 10000403 00000000 8012b904 00000000 8003a4a0
            sr       lo       hi      bad    cause       pc
      10000401 8009c9f8 00000030 0041890c 00800000 80001a98
           fsr      fir
      00000000 00000000
(gdb)
```

`a0 = 0x80742000` matches the LCD framebuffer address `dsa=0x742000`, so this is indeed the function that updates LCD screen content.

Interestingly, there are instructions that Ghidra could not understand, mnemonics listed as `SPECIAL2`.

`gdb` also could not understand them:

```log
B+>0x80001a98      li      v0,-32
   0x80001a9c      addiu   a0,a0,-4
   0x80001aa0      and     v0,a2,v0
   0x80001aa4      addu    a3,a0,v0
   0x80001aa8      sltu    v1,a0,a3
   0x80001aac      beqz    v1,0x80001b00
   0x80001ab0      addiu   a1,a1,-4
   0x80001ab4      .word   0x70a00454
   0x80001ab8      .word   0x70a00494
   0x80001abc      .word   0x70a004d4
   0x80001ac0      .word   0x70a00514
   0x80001ac4      .word   0x70a00554
   0x80001ac8      .word   0x70a00594
   0x80001acc      .word   0x70a005d4
   0x80001ad0      .word   0x70a00614
   0x80001ad4      .word   0x70800455
   0x80001ad8      .word   0x70800495
   0x80001adc      .word   0x708004d5
   0x80001ae0      .word   0x70800515
   0x80001ae4      .word   0x70800555
   0x80001ae8      .word   0x70800595
   0x80001aec      .word   0x708005d5
   0x80001af0      .word   0x70800615
   0x80001af4      sltu    v0,a0,a3
   0x80001af8      bnez    v0,0x80001ab4
   0x80001afc      nop
```

Even QEMU, which does support executing those Ingenic MXU instructions, does not support their mnemonics in `in_asm` logs:

```log
# qemu -d in_asm,op

----------------
IN:
0x80001ab0:  addiu	a1,a1,-4

OP:
 ld_i32 loc0,env,$0xfffffffffffffff0
 brcond_i32 loc0,$0x0,lt,$L0
 st8_i32 $0x1,env,$0xfffffffffffffff4

 ---- 0000000080001ab0 0000000000011000 0000000080001b00
 add_i32 a1,a1,$0xfffffffc
 mov_i32 hflags,$0x10
 brcond_i32 bcond,$0x0,ne,$L1
 mov_i32 PC,$0x80001ab4
 call lookup_tb_ptr,$0x6,$1,tmp5,env
 goto_ptr tmp5
 set_label $L1
 mov_i32 PC,$0x80001b00
 call lookup_tb_ptr,$0x6,$1,tmp5,env
 goto_ptr tmp5
 set_label $L0
 exit_tb $0x7fffa42e1183

----------------
IN:
0x80001ab4:  udi4	a1,zero,zero,0x11

OP:
 ld_i32 loc0,env,$0xfffffffffffffff0
 brcond_i32 loc0,$0x0,lt,$L0
 st8_i32 $0x1,env,$0xfffffffffffffff4

 ---- 0000000080001ab4 0000000000000000 0000000000000000
 mov_i32 loc2,XCR
 mov_i32 loc2,$0x0
 brcond_i32 loc2,$0x0,ne,$L1
 mov_i32 loc3,a1
 mov_i32 loc4,$0x4
 add_i32 loc3,loc3,loc4
 qemu_ld_i32 loc4,loc3,noat+al+leul,0
 mov_i32 XR1,loc4
 mov_i32 a1,loc3
 set_label $L1
 mov_i32 PC,$0x80001ab8
 call lookup_tb_ptr,$0x6,$1,tmp7,env
 goto_ptr tmp7
 set_label $L0
 exit_tb $0x7fffa42e12c3
```

These must be the special Ingenic MXU instructions, we can try to decode it manually from the documentation.
Documentation for this is not very good, but usable enough.

```log
80001ab4 54 04 a0 70     SPECIAL2   zero,a1,zero,0x11,0x14

Instr Bytes:
01010100 00000100 10100000 01110000

Correct for endianness:
01110000 10100000 00000100 01010100

Major code is special2 011100
Ext is 010100
```

So, instruction is `S32LDI` or `S32LDIR`:

![](images/noah/Screenshot_2026-05-08_005307.png)

From the other table we can find out the bit widths of other instruction fields:

![](images/noah/Screenshot_2026-05-08_005142.png)

```log
Instruction = 01110000 10100000 00000100 01010100
Major code = special2 011100
rs = 00101 = 5
op = S32LDI
s12 = 0000000001 << 2 = 4
XRa = 0001 = 1
Ext = 010100
```

Now we know the instruction is `S32LDI`, we can check its function from the document:

![](images/noah/Screenshot_2026-05-08_010020.png)

![](images/noah/Screenshot_2026-05-08_005856.png)

So basically loading data from memory to a MXU special register and incrementing the pointer register at the same time.

Later instructions like this `S32SDI` then writes it to the destination address:

```log
80001ad4 55 04 80 70     SPECIAL2   zero,a0,zero,0x11,0x15
```

So this is just a special `memcpy` implementation that copies data using Ingenic MXU instructions.

Anyway, I figured out the issue with my LCD emulation, just a stupid error that framebuffer address wasn't calculated correctly for the special 4bpp palette + RGB565 OSD overlay mode, basically repeating the first line for the whole screen.
Finally I'm able to boot the NP7000 in QEMU and see its display:

![](images/noah/Screenshot_2026-05-08_010752.png)
