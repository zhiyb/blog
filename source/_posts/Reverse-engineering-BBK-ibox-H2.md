---
title: Reverse engineering BBK @ibox H2
date: 2026-05-23 14:23:50
tags:
    - BBK
    - EEBBK
    - ibox-H2
    - qemu
    - Ingenic
    - JZ4750L
    - MIPS
    - OpenCode
---

步步高~~打火机~~学习机，哪里不会点哪里！

The BBK @ibox H2 is another device that I had previously played with, one of my classmate had it, so I'm interested in seeing it again. It also uses the JZ4740-series Ingenic SoC. With the knowledge and skills that I now have, I should be able to reverse engineer its software, then emulate it using QEMU without having access to the hardware.

Its software is surprising difficult to reverse engineer. Here I recorded the steps I took to understand its software architecture, and I've learned quite a few new things in the process.

<!-- more -->

# Finding software packages

The JZ4740-series Ingenic SoCs have easy to access USB boot method without any security checks, using that we should be able to dump out system's flash storage, if we had the hardware.

However, since I do not have the device, let's check what kind of update or recovery packages are available.

[步步高学习机H2 资料下载 - 步步高下载中心](http://app.eebbk.com/content/list?prodId=254b48b85e3eaa9c8ec182a036cb50fe&keyword=&module=121&subject=&grade=&classId=&classId1=&classId2=&classId3=&classId4=&classId5=&classId6=&gradeTitle=&subjectTitle=&order=&orderType=&anchor=line1&isNewEdition=1)

The "视频学习机_@iboxH2_V2.2L_超级系统恢复" is an interesting target. It is a PC-based recovery software, looks like the recovery is done using the aforementioned USB boot method. Let's install it somewhere and find its contents:

```bash
# Main executable
super_recover.exe

# Data files
alldir1.dat
alldir2.dat
data0.dat
data0_L.dat
data1.dat
data1_L.dat
data2.dat
data2_L.dat
data3.dat
data3_L.dat
data4.dat
data4_L.dat
packet1.dat
packet2.dat
UserInfo.cfg

# USB boot firmware
BurnSys_H2L_V1.0.bin
BurnSys_@IboxH2_V1.0.bin
BurnSys_@IboxH2_V2.0.bin
usb_init_16M.bin
usb_init_16M_Ex.bin

# Miscellaneous, Windows drivers and installer
help.chm license.abc
difxapi.dll wd1010.cat wdreg.exe windrvr6.inf windrvr6.sys usb_recover.inf
Reg.bat UnReg.bat
unins000.exe unins000.dat
```

# Finding SoC model

I know that it uses the Ingenic JZ4740 era SoC, but don't know which specific model is used.

Fortunately it is not difficult to find:

```bash
$ strings BurnSys_* | grep 47
4750L H2+ SystemKernel entry...........
===========4750 usb1.1 ============
===========4750 usb2.0 ============
JZ4750L
```

I have documentation for [JZ4740](https://opennoah.github.io/datasheet/JZ4740_pm.pdf) and [JZ4750](https://opennoah.github.io/datasheet/JZ4750_pm.pdf), but unfortunately not JZ4750L. Later while emulating it, I found out that it's a mixture of the two, mainly JZ4750.

We do have JZ4750L mentioned in one of Ingenic's Linux kernel source code patches (`linux-2.6.24.3-jz-20100304.patch.gz`), the header files for JZ4750L is going to be very useful for finding out IRQ index and available peripherals.

# System images are encrypted

Whether the SoC loads from NAND or MMC, it must load the first stage loader of 8KiB into internal SRAM, then starts executing there.

The file sizes of `data0.dat` and `data0_L.dat` look like it could be used for this purpose, however it seems they are encrypted:

```bash
$ ent data0.dat
Entropy = 7.974006 bits per byte.

Optimum compression would reduce the size
of this 7112 byte file by 0 percent.
```

A simple entropy check shows file's data is almost random, likely either encrypted or compressed.

Usually for an executable binary, entropy should be much less, e.g. Noahedu NP1300 u-boot binary:

```bash
$ ent np1380/segment01.bin
Entropy = 3.802258 bits per byte.

Optimum compression would reduce the size
of this 960316 byte file by 52 percent.
```

`binwalk` could not find any recognisable compression headers, so it is likely encrypted.

Except for `data1*.dat`, all other `data*.dat` files are also encrypted.
`data1*.dat` looks like some sort of index file, so not super useful.

Next we must check the USB recovery process to understand how to decrypted these files.

# USB recovery process

## Windows executable

Loading the Windows executable `super_recover.exe` into Ghidra, the disassembled `entry` function looks super complicated:

```c
            *(undefined4 *)(puVar17 + -4) = 0;
            *(undefined1 **)(puVar17 + -8) = puVar17 + -4;
            *(undefined4 *)(puVar17 + -0xc) = 4;
            *(undefined4 *)(puVar17 + -0x10) = 0x1000;
            *(undefined4 *)(puVar17 + -0x14) = 0xfffff000;
            puVar20 = (undefined4 *)(puVar17 + -0x18);
            *(undefined4 *)(puVar17 + -0x18) = 0x5b4a16;
            (*pcRam002adeb0)();
            bRamfffff21f = bRamfffff21f & 0x7f;
            bRamfffff247 = bRamfffff247 & 0x7f;
            uVar10 = *puVar20;
            *puVar20 = uVar10;
            puVar20[-1] = puVar20;
            puVar20[-2] = uVar10;
            puVar20[-3] = 0x1000;
            puVar20[-4] = 0xfffff000;
            puVar21 = puVar20 + -5;
            puVar20[-5] = 0x5b4a2b;
            (*pcVar4)();
            puVar22 = (undefined1 *)((int)puVar21 + 0x24);
            do {
              puVar23 = puVar22 + -4;
              *(undefined4 *)(puVar22 + -4) = 0;
              puVar22 = puVar22 + -4;
            } while (puVar23 != (undefined1 *)((int)puVar21 + -0x5c));
                    /* WARNING: Bad instruction - Truncating control flow here */
            halt_baddata();
          }
          piVar14 = (int *)piVar25[1];
          *(int *)(puVar17 + -4) = *piVar25 + 0x2ade08;
          piVar26 = piVar25 + 2;
          puVar18 = (undefined4 *)(puVar17 + -8);
          puVar17 = puVar17 + -8;
          *puVar18 = 0x5b49a6;
          uVar10 = (*pcRam002adea8)();
          while( true ) {
            iVar11 = *piVar26;
            piVar25 = (int *)((int)piVar26 + 1);
            if ((char)iVar11 == '\0') break;
            *(int **)(puVar17 + -4) = piVar25;
            piVar27 = piVar25;
            do {
              piVar26 = piVar27;
              if (piVar25 == (int *)0x0) break;
              piVar25 = (int *)((int)piVar25 + -1);
              piVar26 = (int *)((int)piVar27 + (uint)bVar29 * -2 + 1);
              iVar9 = *piVar27;
              piVar27 = piVar26;
            } while ((char)((char)iVar11 + -1) != (char)iVar9);
            *(undefined4 *)(puVar17 + -8) = uVar10;
            puVar19 = (undefined4 *)(puVar17 + -0xc);
            puVar17 = puVar17 + -0xc;
            *puVar19 = 0x5b49bb;
            iVar11 = (*pcRam002adeac)();
```

And referenced strings have nothing to be with USB recovery.

It is likely encapsulated by another loader layer that obfuscated the actual executable.
Fortunately removing that layer is not difficult.

Looking at the file section headers at the beginning, there are sections like `UPX0` and `UPX1`.

![UPX0, UPX1 sections](<Screenshot 2026-05-22 193629.png>)

A quite Google search says these are created by [UPX](https://upx.github.io/) packer.
Install the upx packer then unpack the original executable:

```bash
upx -d -o super_recover_d.exe super_recover.exe
```

The unpacked executable looks much better in Ghidra:

```c
  hMutex = (HANDLE)FUN_00406e20();
  DVar2 = GetLastError();
  uVar5 = DVar2 < 0xb7;
  if (DVar2 == 0xb7) {
    FUN_00409614(&DAT_004a57c8,"TForm4740SysTool");
    hWnd = FindWindowA(&DAT_004a57c8,(LPCSTR)0x0);
    iVar7 = 0;
    FUN_0045786c((int)*p_p_struct_8,&DAT_0047f50c,&DAT_0047f504,0);
    SetForegroundWindow(hWnd);
    SendMessageA(hWnd,0x465,0,0);
  }
  else {
    FUN_00457250(*p_p_struct_8,"超级系统恢复");
    FUN_0045765c(*p_p_struct_8,&PTR_LAB_0046c7f8,(fw_info_t *)PTR_p_fw_info_004826a0);
    iVar7 = 0x47f44f;
    FUN_004576dc((int)*p_p_struct_8);
  }
  ReleaseMutex(hMutex);
```

And we can find strings referring to USB recovery process, and data files:

```
0047069c	@ibox H1
004706b0	BurnSys_iboxh1.bin
004706cc	@ibox H2
004706e0	BurnSys_@IboxH2_V1.0.bin
00470704	BurnSys_@IboxH2_V2.0.bin
00470728	80B04000
0047073c	4740
...
0047538c	发送失败,ErrorCode:
004753a8	严重警告：收取CSW数据失败，错误码=
004753d4	Burn NandFlash: 失败,坏块过多:
004753fc	实际数据量字节数 Size=
0047541c	StartBlock=
00475430	  TotalBlocks=
00475448	          坏块极限数=
00475468	          已经坏块数=
00475488	 前4个坏块号:
004754c0	Burn NandFlash: 失败-->USB收到的数据Sum不对!
004754f8	实际数据量字节数=
00475514	实际SUM=
00475528	从USB返回的信息:USB计算得到的SUM=
00475554	从USB返回的信息:   PC传过去的SUM=
00475580	从USB返回的信息:  PC传过去的长度=
004755ac	Burn NandFlash: 成功,Size=
```

## Hardware revisions

Following references to data file strings, I came cross this section at function offset `0x004700dc`:

```c
else if (param_2 == 6) {
  set_string?(param_1->hdl_3,"3");
  set_string?(param_1->hdl_26,"26");
  set_string?(param_1->hdl_device_name,"@ibox H2");
  set_string?(param_1->hdl_v1_0_bin,"BurnSys_@IboxH2_V1.0.bin");
  set_int?(param_1->field897_0x39c,1);
  set_string?(param_1->hdl_v2_0_bin,"BurnSys_@IboxH2_V2.0.bin");
  set_string?(param_1->hdl_load_addr,"80B04000");
  set_string?(param_1->hdl_soc,"4740");
  set_string?(param_1->hdl_data0_dat,"data0.dat");
  set_string?(param_1->hdl_data1_dat,"data1.dat");
  set_string?(param_1->hdl_data2_dat,"data2.dat");
  set_string?(param_1->hdl_data3_dat,"data3.dat");
  set_string?(param_1->hdl_data4_dat,"data4.dat");
  FUN_00437db0((string_t *)param_1->field993_0x414,CONCAT31((int3)((uint)extraout_EDX_00 >> 8),1),
               extraout_ECX_05);
}
else if (param_2 == 7) {
  set_string?(param_1->hdl_3,"3");
  set_string?(param_1->hdl_26,"26");
  set_string?(param_1->hdl_device_name,"@ibox H2");
  set_string?(param_1->hdl_v1_0_bin,"BurnSys_H2L_V1.0.bin");
  set_int?(param_1->field897_0x39c,3);
  set_string?(param_1->hdl_v2_0_bin,"BurnSys_@IboxH2_V2.0.bin");
  set_string?(param_1->hdl_load_addr,"80B04000");
  set_string?(param_1->hdl_soc,"4750");
  set_string?(param_1->hdl_data0_dat,"data0_L.dat");
  set_string?(param_1->hdl_data1_dat,"data1_L.dat");
  set_string?(param_1->hdl_data2_dat,"data2_L.dat");
  set_string?(param_1->hdl_data3_dat,"data3_L.dat");
  set_string?(param_1->hdl_data4_dat,"data4_L.dat");
  FUN_00437db0((string_t *)param_1->field993_0x414,CONCAT31((int3)((uint)extraout_EDX_01 >> 8),1),
               extraout_ECX_06);
}
```

Looks like there are actually two different hardware revisions.

One set of files used for the version with JZ4740 SoC:

```bash
BurnSys_@IboxH2_V1.0.bin
BurnSys_@IboxH2_V2.0.bin
data0.dat
data1.dat
data2.dat
data3.dat
data4.dat
```

Another set of files used for the version with JZ4750L SoC:

```bash
BurnSys_H2L_V1.0.bin
BurnSys_@IboxH2_V2.0.bin
data0_L.dat
data1_L.dat
data2_L.dat
data3_L.dat
data4_L.dat
```

## First stage loader

Reading about the USB boot process "33.5 USB Boot Specification", the first step is to load a small binary blob into SRAM, less than 16 KiB, to initialise the on-board SDRAM and storage.

As their name suggests, `usb_init_16M.bin` and `usb_init_16M_Ex.bin` seems to be used for this purpose.

We can also find references to them in the Windows executable at function offset `0x00470854`:

```c
if (param_2 == 0) {
  set_string?(param_1->hdl_usb_init_bin,"usb_init.bin");
  return;
}
if (param_2 == 1) {
  set_string?(param_1->hdl_usb_init_bin,"usb_init_16M.bin");
}
else if (param_2 == 2) {
  set_string?(param_1->hdl_usb_init_bin,"usb_init_64M.bin");
}
else if (param_2 == 3) {
  set_string?(param_1->hdl_usb_init_bin,"usb_init_16M_Ex.bin");
}
```

JZ4750 SRAM is located at address offset `0x80000000`, we can load these binaries at these address and confirm using Ghidra:

![USB first stage binary](<Screenshot 2026-05-22 121531.png>)

The function names are assign by me, we can figure out what these functions does by checking which hardware peripheral registers it accesses.

Again we see a string confirming that `usb_init_16M_Ex.bin` should be used on the JZ4750L SoC. It initialises `MSC0`, so system storage is probably emulated MMC connected to `MSC0`.

`usb_init_16M.bin` maybe used on a different hardware revision, we'll find out later.

## Second stage program

After SDRAM is initialised, we can load a larger binary that can do a lot more things, like actually writing to system storage.

These binaries are very likely used for that based on their names:

```bash
BurnSys_H2L_V1.0.bin
BurnSys_@IboxH2_V1.0.bin
BurnSys_@IboxH2_V2.0.bin
```

Finding their load address offset is rather easy, load them into Ghidra, start disassembly at the beginning, then look at the first two instructions:

```asm
00 80 1f 3c     lui        ra,0x8000
00 40 ff 27     addiu      ra,ra,0x4000
```

This means setting $ra register to `0x80004000`.

Later we can also find function calls to similar address offsets:

```c
(*(code *)&LAB_80004af0)();
```

So in this case, `0x80004000` seems like the load address offset to use.

Checking all three files, they actually load at different addresses:

```bash
BurnSys_H2L_V1.0.bin     -> 0x80004000
BurnSys_@IboxH2_V1.0.bin -> 0x80004000
BurnSys_@IboxH2_V2.0.bin -> 0x80b04000
```

`BurnSys_@IboxH2_V2.0.bin` seems to be the final program used for writing data files to system storage, let's take a look at it. Load address `0x80b04000` is also mentioned in the Windows executable.

## Decrypt executable binary data files

In `BurnSys_@IboxH2_V2.0.bin` there are some GBK-encoded strings referring to buffer decryption:

```asm
                     s_HAN#Buffer_...._80b33bfc                      XREF[2]:     FUN_80b1029c:80b103dc(*),
                                                                                  FUN_80b11e20:80b11e30(*)
80b33bfc 0d 0a b5        ds         "\r\n底层Buffer解码中...."
         d7 b2 e3
         42 75 66
80b33c13 00              ??         00h
                     s_HAN#Buffer_...._80b33c14                      XREF[2]:     FUN_80b1029c:80b10430(*),
                                                                                  FUN_80b11e20:80b11e90(*)
80b33c14 0d 0a b5        ds         "\r\n底层Buffer解码完成...."
         d7 b2 e3
         42 75 66
```

Following the references we can find this function:

```c
void FUN_80b11e20(uint param_1,undefined4 param_2,byte *param_3,undefined4 param_4)
{
  byte *pbVar1;
  int iVar2;
  uint uVar3;

  dbg_printf(s_HAN#Buffer_...._80b33bfc,param_2,param_3,param_4);
  iVar2 = 0;
  uVar3 = 0;
  if (param_1 != 0) {
    param_3 = &DAT_80c67894;
    do {
      pbVar1 = &DAT_80b50c30 + iVar2;
      uVar3 = uVar3 + 1;
      iVar2 = iVar2 + 1;
      *param_3 = *param_3 ^ *pbVar1;
      if (0xfff < iVar2) {
        iVar2 = 0;
      }
      param_3 = param_3 + 1;
    } while (uVar3 < param_1);
  }
  dbg_printf(s_HAN#Buffer_...._80b33c14,iVar2,param_3,uVar3);
  return;
}
```

So this is just a simple XOR operation with pattern located at `0x80b50c30` of size `0x1000`.
Extract the XOR pattern at that address:

```bash
$ hd xor.bin
00000000  a8 7c 10 8d 30 c5 ef 5e  85 66 34 e6 0b c3 e1 40  |.|..0..^.f4....@|
00000010  39 b8 b6 40 6b e8 b2 b1  83 fb 3b d7 d7 89 9c 8b  |9..@k.....;.....|
00000020  43 a9 c4 9e 71 63 69 48  88 7b 09 d6 f3 63 6c e0  |C...qciH.{...cl.|
00000030  ed 35 94 b3 ef 06 4a ba  46 0c b4 17 62 41 97 ad  |.5....J.F...bA..|
...
```

However, applying that XOR pattern directly from the beginning of `data0_L.dat` doesn't work, the resulting file still has very high entropy.

Check `data0_L.dat` file data again, the first `0x10` bytes look to be some short of a header with a fairly clear boundary:

```bash
$ hd data0_L.dat | head
00000000  26 04 04 20 3f cb 08 00  00 00 00 00 00 00 00 00  |&.. ?...........|
00000010  e4 2c 43 c0 31 c5 fe 5a  85 66 34 e6 2a 23 01 43  |.,C.1..Z.f4.*#.C|
00000020  39 b8 5f cf 4a 08 92 b0  83 7b 26 eb d7 c9 21 bc  |9._.J....{&...!.|
00000030  43 29 dd a2 a9 6e 50 6f  80 7b 29 d5 f3 63 6c e0  |C)...nPo.{)..cl.|
```

Same as other `data*.dat` files, the header bytes look fairly consistent, random data only starts after offset `0x10`:

```bash
# data0_L.dat
00000000   26 04 04 20  3F CB 08 00  00 00 00 00  00 00 00 00  E4 2C 43 C0  31 C5 FE 5A  85 66 34 E6  2A 23 01 43
# data2_L.dat
00000000   26 04 04 20  60 C4 C5 0C  00 00 00 00  00 00 00 00  A8 FC 0F B1  30 85 10 79  85 F6 B4 A6  0B 5B 61 00
# data3_L.dat
00000000   26 04 04 20  86 FF D0 1A  00 00 00 00  00 00 00 00  40 83 AD AA  B1 45 ED 62  0D 8D 76 C2  8C 43 E2 7C
# data4_L.dat
00000000   26 04 04 20  D1 53 74 08  00 00 00 00  00 00 00 00  30 FC 13 B1  E0 7C 8C 7A  1A E6 31 DA  EB 5D 44 64
```

Extract data after offset `0x10` then apply the XOR pattern, and there it is, decrypted executable binary:

```bash
$ ent output/data0_L.dat
Entropy = 5.381683 bits per byte.

Optimum compression would reduce the size
of this 4520 byte file by 32 percent.
```

Loading decrypted `data0_L.dat` into Ghidra at offset `0x80000000`, disassembly makes sense too.

The other `data*.dat` can be decrypted following the same steps.

# First stage loader

Now we have `data0_L.dat` decrypted, let's take a look at its content.

```c
void entry(void)
{
  bool bVar1;
  int iVar2;
  uint uVar3;

  gpio_init();
  _DAT_b0001008 = 0xffffffff;
  _DAT_b0000000 = 0x222220;
  _DAT_b0000010 = 0x1b000120;
  sdram_init();
  _DAT_b3010050 = 3;
  iVar2 = msc0_init();
  if (iVar2 != 0) {
    do {
                    /* WARNING: Do nothing block with infinite loop */
    } while( true );
  }
  uVar3 = 5;
  if ((_HSPR & 1) == 0) {
    do {
      dbg_print(s_first_0_80001124);
      iVar2 = msc0_read_2(DAT_80001180,0xd48,&SUB_80004000);
      bVar1 = uVar3 < 2;
      if (iVar2 == 0) break;
      dbg_print(s_second_0_80001130);
      iVar2 = msc0_read_2(DAT_80001184,0xd48,&SUB_80004000);
      bVar1 = uVar3 < 2;
      if (iVar2 == 0) break;
      uVar3 = uVar3 - 1;
      bVar1 = uVar3 < 2;
    } while (uVar3 != 0xffffffff);
    if ((bVar1) && (iVar2 = FUN_80000d10(), iVar2 != 0)) {
      dbg_print(s_HAN#os2_8000113c);
      do {
                    /* WARNING: Do nothing block with infinite loop */
      } while( true );
    }
  }
  else {
    do {
      dbg_print(s_second_1_800010e0);
      iVar2 = msc0_read_2(DAT_80001184,0xd48,&SUB_80004000);
      bVar1 = uVar3 < 2;
      if (iVar2 == 0) break;
      dbg_print(s_first_1_800010ec);
      iVar2 = msc0_read_2(DAT_80001180,0xd48,&SUB_80004000);
      bVar1 = uVar3 < 2;
      if (iVar2 == 0) break;
      uVar3 = uVar3 - 1;
      bVar1 = uVar3 < 2;
    } while (uVar3 != 0xffffffff);
    if ((bVar1) && (iVar2 = FUN_80000d10(), iVar2 != 0)) {
      dbg_print(s_HAN#os1_80001110);
      do {
                    /* WARNING: Do nothing block with infinite loop */
      } while( true );
    }
  }
  dbg_print(s_Enter_tiny->20110102_800010f8);
  _DAT_b0001008 = 0xffffffff;
  (*(code *)&SUB_80004000)();
  return;
}
```

```c
                             DAT_80001180                                    XREF[4]:     FUN_80000d10:80000d1c(R),
                                                                                          FUN_80000d10:80000d7c(R),
                                                                                          entry:80000eb8(R),
                                                                                          entry:80000f58(R)
        80001180 00 0c 00 00     undefined4 00000C00h
                             DAT_80001184                                    XREF[3]:     FUN_80000d10:80000db8(R),
                                                                                          entry:80000e88(R),
                                                                                          entry:80000f88(R)
        80001184 00 84 00 00     undefined4 00008400h
```

It initialises `SDRAM`, `MSC0` and some other peripherals, then starts loading next stage data from SD or MMC card connected to `MSC0`. All as expected for a first stage loader.

So, for emulation, we will need to prepare an SD/MMC image, with `data0_L.dat` placed at the very beginning.

Interestingly it seems to implement an AB partition switching mechanism, using `RTC` Hibernate Scratch Pattern Register (`HSPR`) to switch between them.
One partition is located at `0x0c00` block offset (`0x00180000`: 1.5 MiB), another one at `0x8400` block offset (`0x01080000`: 16.5 MiB), size `0x0d48` block (`0x001a9000` / 1700 KiB).
Load and entry address is `0x80004000`.

# Kernel

Loading decrypted `data2_L.dat` into Ghidra at address `0x80004000`, we also get good looking disassembly results, e.g.:

```c
void FUN_80005c70(void)
{
  undefined4 *puVar1;

  FUN_80006428();
  puVar1 = &DAT_8019f320;
  do {
    *(undefined1 *)puVar1 = 0;
    puVar1 = (undefined4 *)((int)puVar1 + 1);
  } while (puVar1 != (undefined4 *)&DAT_80488728);
  setCopReg(0,Status,0x10000400);
  memcpy(0x80000000,irq_entry,0x20);
  memcpy(0x80000180,irq_entry,0x20);
  memcpy(0x80000200,irq_entry,0x20);
  sync();
  thunk_EXT_FUN_a000823c();
  FUN_800256c0();
  FUN_8000804c();
  FUN_80006140();
  _TMR.TFR = 0;
  _TMR.TMSR = 0xffffffff;
  FUN_80004c60(0);
  FUN_80005c5c();
  FUN_8000f9a4();
  FUN_80026538(0);
  FUN_8001975c();
  FUN_8001987c();
  FUN_80012f84();
  FUN_80016058();
  FUN_800917e8();
  FUN_80042b80();
  dbg_printf(s_===jump_to_ucmain===_800aa3f4);
  start_tasks();
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

It must be the data loaded by the first stage loader.
For emulation, we will need to copy `data2_L.dat` to both partitions, offsets `0x00180000` and `0x01080000`.

How it loads other data is not clear from a quick look at the code.
Let's try to run the code in QEMU.

# JZ4750L `MSC0` boot

Following JZ4750 programming manual "33.6 MMC/SD Boot Specification", I've made a [bootrom](https://github.com/OpenNoah/bootrom) that loads 8 KiB of data from `MSC0` into `0x80000000`, and starts executing there.

However, this didn't work, SoC seems to keep rebooting from bootrom code.

Attach gdb to QEMU, set a breakpoint at `0x80000000`, we can see that code could not be disassembled, but later instructions are fine:

```bash
$ mipsel-linux-gdb \
    -ex "target remote localhost:8842" \
    -ex "set architecture mips:isa32" \
    -ex "layout asm"
```

```bash
(gdb) b *0x80000000
Breakpoint 1 at 0x80000000
(gdb) c
Continuing.

Breakpoint 1, 0x80000000 in ?? ()
(gdb)
```

```bash
B+>0x80000000      .word   0x4d53504c
   0x80000004      bal     0x8000000c
   0x80000008      nop
   0x8000000c      move    gp,ra
   0x80000010      lw      t1,0(ra)
```

Attempting to single step through an instruction does not work. Instead of continuing execution, CPU seems to jump to an interrupt somewhere else, probably because of illegal instruction exception.

Enabling QEMU `in_asm` log, we can see that QEMU also does not understand the code at `0x80000000`, and jumps to somewhere else:

```bash
./qemu-system-mipsel
    -d guest_errors,unimp,in_asm \
    -accel tcg,one-insn-per-tb=on \
    ...
```

```c
----------------
IN:
0xbfc00190:  jalr	t9

----------------
IN:
0xbfc00194:  nop

----------------
IN:
0x80000000:  0x4d53504c

----------------
IN:
0xb0030000:  nop
```

The data at `0x80000000` in ASCII says `LPSM`, no idea what it means.
JZ4750 programming manual says CPU should start execution at address `0x80000000`.
I also have another device (Noahedu NP6800) that uses JZ4750 with `MSC0` boot mode, that device's first stage loader does have a valid instruction at `0x80000000`.
This is probably a quirk with JZ4750L that it starts execution from `0x80000004` instead.

Easy fix in the bootrom, the first stage loader then runs correctly.

# gdb breakpoint commands

I was hoping that these loader code would print debug messages to one of the UART peripherals.
However, looking inside the debug print functions, they are just.. empty 😞.

For example, the debug print function at offset `0x80000270` in `data0_L.dat`:

```c
void dbg_print(char *p_str)
{
  return;
}
```

It's lucky that the debug print calls were not optimised out, otherwise losing all the debug strings would make reverse engineering much more difficult. Guess Link-Time Optimization (LTO) was not available or not enabled at that time (LTO sometimes still does not work correctly nowadays).

But now, we need to find a way to get these strings printed.

One possible solution is to somehow patch the binary to print them to a UART, but that is rather difficult.

Alternatively, the method I went with is to attach gdb to QEMU, set up breakpoint at these functions, then print the function argument register inside the gdb session.

Some more googling later, I came up with this gdb script:

```bash
b *0x80000270
commands
    silent
    printf "%s", $a0
    c
end
```

This can be simplified to a simple [`dprintf`](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Dynamic-Printf.html) command:

```bash
dprintf *0x80000270, "%s", $a0
```

It is also possible to print out where the function was called by printing out the `$ra` register, like this:

```bash
dprintf *0x8003ab60, "ra:%#010x -> %#010x: fs_read_sector(%d, %#010x, %#010x, %#010x)\n", $ra, $pc, $a0, $a1, $a2, $a3
```

Being able to insert prints and other commands after a breakpoint provided to be super useful for reverse engineering.

# gdb character encoding

Like any modern Linux system, my terminal is set to UTF-8 encoding, but these older Chinese devices like ibox-H2 generally uses GB2312/GBK as their main character set. So we need to do character encoding conversion.

I read that [gdb supports setting target character set, and automatically converting them to host character set](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Character-Sets.html).

This does work for commands like `x/s`:

```bash
b *0x80024bd4
commands
    silent
    x/s $a0
    c
end
```

```c
0x802ba110:     "\n", '=' <repeats 67 times>, "\n  启动音频任务\n  查看当前正在使用的内存情况：\n"
0x802ba110:     "  有0处内存正在使用\n  共申请0Bytes，分配了0Bytes\n", '=' <repeats 67 times>, "\n\n"
0x802ba110:     '=' <repeats 27 times>, "\n"
```

But does not seem to work for `printf` and `dprintf` using `%s`:

```bash
dprintf *0x80024bd4, "%s", $a0
```

```bash
===================================================================
  �����Ƶ����
  �鿴��ǰ����ʹ�õ��ڴ������
  ��0���ڴ�����ʹ��
  ������0Bytes��������0Bytes
===================================================================
```

So I need a different solution, that being [`luit`](https://linux.die.net/man/1/luit).

_Luit is a filter that can be run between an arbitrary application and a UTF-8 terminal emulator. It will convert application output from the locale's encoding into UTF-8, and convert terminal input from UTF-8 into the locale's encoding._

```bash
luit -encoding GBK mipsel-linux-gdb -x gdb_cmds.txt
```

Then, with both host and target encoding set as GBK in gdb, I can finally read these Chinese characters:

```bash
===================================================================
  启动音频任务
  查看当前正在使用的内存情况：
  有0处内存正在使用
  共申请0Bytes，分配了0Bytes
===================================================================
```

However, this garbles the nice CUI borders around gdb assembly code listing.
There is probably a solution but I can turn off CUI for now, a small price to pay.
For disassembly I usually look at Ghidra anyway.

![Garbled CUI borders](<Screenshot 2026-05-22 192842.png>)

# JZ4750L interrupts

## Interrupt index

After booting the `MSC0` image, it soon got stuck initialising `MSC0`.

```
===========================
Mmc_controller.c CpmSelectMscHSClk:clock = %d,div = %d,A_CPM_MSCCDR[0x%x]
set clock to %u Hz is_sd=%d clkrt=%d A_MSC_CLKRT[0x%x]
```

With `ingenic_msc_*` trace enabled in QEMU, I can see that it has enabled `MSC0` interrupts, and an interrupt has happened, but nothing afterwards:

```
ingenic_msc_write *0x28 = 0xffff
ingenic_msc_read *0x24 = 0xffff
ingenic_msc_write *0x24 = 0xfff8
ingenic_msc_write *0x2c = 0x0
ingenic_msc_write *0x30 = 0x0
ingenic_msc_write *0x18 = 0x0
ingenic_msc_write *0x1c = 0x0
ingenic_msc_write *0xc = 0x80
ingenic_msc_write *0x0 = 0x6
ingenic_msc_cmd CMD0 R0 arg 0x00000000
sdbus_command @sd-bus-msc0 CMD00 arg 0x00000000
sdcard_normal_command SD        GO_IDLE_STATE/ CMD00 arg 0x00000000 (mode transfer, state transfer)
sdcard_response RESP#0 (no response) (sz:0)
ingenic_msc_irq pending=0x6
```

I had interrupts connected as in JZ4750 documentation, but apparently JZ4750L interrupts are connected differently.
Since we don't have JZ4750L documentation, the only information we have is the Linux kernel source code patch.
Fortunately, it does tell us how the interrupts are connected:

```c
// include/asm-mips/mach-jz4750l/regs.h
// 1st-level interrupts
#define IRQ_ETH         0
#define IRQ_SFT         4
#define IRQ_I2C         5
#define IRQ_RTC         6
#define IRQ_UART2       7
#define IRQ_UART1       8
#define IRQ_UART0       9
#define IRQ_AIC         10
#define IRQ_GPIO5       11
#define IRQ_GPIO4       12
#define IRQ_GPIO3       13
#define IRQ_GPIO2       14
#define IRQ_GPIO1       15
#define IRQ_GPIO0       16
#define IRQ_BCH         17
#define IRQ_SADC        18
#define IRQ_CIM         19
#define IRQ_TSSI        20
#define IRQ_TCU2        21
#define IRQ_TCU1        22
#define IRQ_TCU0        23
#define IRQ_MSC1        24
#define IRQ_MSC0        25
#define IRQ_SSI         26
#define IRQ_UDC         27
#define IRQ_DMAC1       28
#define IRQ_DMAC0       29
#define IRQ_IPU         30
#define IRQ_LCD         31
```

Interestingly, there are 2x `DMAC` interrupts.
JZ4750 does have 2x DMA controllers, but only one interrupt connected.
So this part in the JZ4750L is different to JZ4750.

Easy fix, or so I thought.

# Filesystem

At least with interrupt index fixed, `MSC0` initialisation completes successfully.
Kernel tries to load some resource files, but failed:

```
注意：目前使用的是SD协议、小容量规格的内置卡...

mmcinfo.protect_unit         = 0x%2x
mmcinfo.erase_unit           = 0x%2x
mmcinfo.block_num           = 0x%2x
vnand capacity = %d = 0x%x!
====fs_init failed====
----------加载资源--------------
DLX打开失败
DLX打开失败
----------关键点--------------
DLX打开 %s 失败
DLX打开 %s 失败
DLX打开 %s 失败
```

After fixing some issues with LCD descriptor, I have LCD output as well:

![Touchscreen calibration?](<Screenshot 2026-05-22 195946.png>)

It looks like a touchscreen calibration process, but there is no way the display would look that primitive.
It's probably because it failed to load the resource files, let's check how it finds resource files.

## File packets

Two of the largest files from the recovery program are `packet1.dat` and `packet2.dat`.
Let's see what's inside `packet1.dat`:

```
00000000   62 62 6B 2E  28 08 82 19  45 01 00 00  4D DD 68 66  bbk.(...E...M.hf
00000010   18 FC 03 00  00 F4 01 00  32 30 20 33  41 44 33 39  ........20 3AD39
00000020   44 39 38 20  CF B5 CD B3  5C C5 E4 D6  C3 5C 62 67  D98 ....\....\bg
00000030   70 69 63 30  2E 62 69 6E  00 00 00 00  00 00 00 00  pic0.bin........
00000040   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
00000110   18 FC 03 00  18 F0 05 00  32 30 20 33  41 44 33 39  ........20 3AD39
00000120   44 39 38 20  CF B5 CD B3  5C C5 E4 D6  C3 5C 62 67  D98 ....\....\bg
00000130   70 69 63 31  2E 62 69 6E  00 00 00 00  00 00 00 00  pic1.bin........
00000140   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
00000210   18 FC 03 00  30 EC 09 00  32 30 20 33  41 42 41 39  ....0...20 3ABA9
00000220   34 44 32 20  CF B5 CD B3  5C C5 E4 D6  C3 5C 62 67  4D2 ....\....\bg
00000230   70 69 63 32  2E 62 69 6E  00 00 00 00  00 00 00 00  pic2.bin........
00000240   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
```

Ok, it is actually quite easy to guess what whose numbers mean.

First, a global header of 16 bytes.

```
00000000   62 62 6B 2E  28 08 82 19  45 01 00 00  4D DD 68 66  bbk.(...E...M.hf
```

Then, an array of file headers, each 256 bytes.

```
00000010   18 FC 03 00  00 F4 01 00  32 30 20 33  41 44 33 39  ........20 3AD39
00000020   44 39 38 20  CF B5 CD B3  5C C5 E4 D6  C3 5C 62 67  D98 ....\....\bg
00000030   70 69 63 30  2E 62 69 6E  00 00 00 00  00 00 00 00  pic0.bin........
00000040   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
00000110   18 FC 03 00  18 F0 05 00  32 30 20 33  41 44 33 39  ........20 3AD39
00000120   44 39 38 20  CF B5 CD B3  5C C5 E4 D6  C3 5C 62 67  D98 ....\....\bg
00000130   70 69 63 31  2E 62 69 6E  00 00 00 00  00 00 00 00  pic1.bin........
00000140   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
```

Going to address `0x0001f400`, that appears to be near where data starts.

```
0001F300   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
...
0001F400   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
0001F410   56 58 CC CC  CC CC E0 01  00 00 10 01  00 00 CC CC  VX..............
0001F420   CC CC CC 10  FF FF FF FF  45 00 45 00  88 00 03 00  ........E.E.....
```

So that seems to be the offset to file data after global header.

The difference between two file offsets is `0x0005f018 - 0x0001f400 = 0x0003fc18`.
Exactly same as the first 4 bytes of the first file header.
So that seems to be file size.

After file header offset 0x08, there appears to be a string.
Decode it using GBK encoding, we get strings like:

```
20 3AD39D98 系统\配置\bgpic0.bin
20 3AD39D98 系统\配置\bgpic1.bin
20 3ABA94D2 系统\配置\bgpic2.bin
```

Not sure what 20 means, probably describes some sort of file type.
Next 8 character hex string, probably some sort of checksum.
Then we have the file path.

The last file header is located at:

```
00014410   90 50 03 00  52 FF BC 15  32 30 20 33  41 42 39 39  .P..R...20 3AB99
00014420   43 31 37 20  D3 A6 D3 C3  5C D7 F7 CE  C4 5C B8 DF  C17 ....\....\..
00014430   BF BC B7 B6  CE C4 2E 62  69 6E 00 00  00 00 00 00  .......bin......
00014440   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  ................
```

So we have `(0x00014410 - 0x00000010) / 256 + 1 = 325 = 0x00000145` number of files.
The 4 bytes starting from offset 0x08 in the global header shows exactly the same number.
That should be where number of files is stored.

With these information, we can extract all the files from `packet1.dat` and `packet2.dat`.
Pictures and audio files all open without issue, text file also looking good, no extra unrecognised data at the end of file.
The two `packet*.dat` files are probably for two different partitions.

## Filesystem partitions

Going back to the `data2_L.dat` kernel binary, let's check how it initialises the file system.
There is this print we can try to follow:

```
====fs_init failed====
```

It is referenced by the function at `0x80020500`:

```c
  iVar1 = FTL_Init(0);
  if (iVar1 == 0) {
    uVar2 = fs_init(0);
    if (uVar2 == 0xfffffffe) {
      dbg_printf(s_====fs_init_failed====_800aef4c);
    }
```

Following the `fs_init(0)` function, eventually I found the function at `0x8003ab60`:

```c
int fs_read_sector(int partition,uint block_offset,int num_blocks,void *p_buf)
{
  int iVar1;
  int iVar2;
  int iVar3;
  undefined1 auStack_28 [8];

  iVar3 = -1;
  if (num_blocks == 0) {
    return 0;
  }
  mutex_lock(DAT_801a6fe0,0,auStack_28);
  if (partition == 1) {
    iVar3 = msc0_card_type();
    if ((iVar3 == 0) && (fat_offset?[1] <= block_offset)) {
      iVar3 = msc0_mmcf_read_sector(block_offset + 0xdec00,num_blocks,p_buf);
    }
    else {
      iVar3 = msc0_sd_read_sector(block_offset + 0xdec00,num_blocks,p_buf);
    }
    iVar1 = block_offset + 0xdec00;
    if ((iVar3 != 0x13) || (iVar2 = FUN_8003d438(4), iVar2 != 0)) goto LAB_8003abf0;
  }
  else {
    if (1 < partition) {
      if ((partition == 2) &&
         (iVar3 = msc1_read_sector(block_offset,num_blocks,p_buf), iVar3 == 0x13)) {
        iVar1 = tf_init(4);
        if (iVar1 == 0) {
          sd_state? = 1;
          iVar3 = msc1_read_sector(block_offset,num_blocks,p_buf);
        }
        else {
          sd_state? = 0;
        }
      }
      goto LAB_8003abf0;
    }
    if (partition != 0) goto LAB_8003abf0;
    iVar3 = msc0_card_type();
    if ((iVar3 == 0) && (fat_offset?[0] <= block_offset)) {
      iVar3 = msc0_mmcf_read_sector(block_offset + 0xf400,num_blocks,p_buf);
      goto LAB_8003abf0;
    }
    iVar1 = block_offset + 0xf400;
  }
  iVar3 = msc0_sd_read_sector(iVar1,num_blocks,p_buf);
LAB_8003abf0:
  mutex_unlock(DAT_801a6fe0);
  return iVar3;
}
```

So we can see:
For `partition == 0`, it reads from `MSC0` at block offset `0x0000f400` (`0x01e80000`: 30.5 MiB).
For `partition == 1`, it reads from `MSC0` at block offset `0x000dec00` (`0x1bd80000`: 445.5 MiB).
For `partition == 2`, it reads from `MSC1`.

Later in `fs_init()`, there is:

```c
    if (((((msc0_buf[0] == 0xe9) || (msc0_buf[0] == 0xeb)) && (msc0_buf[0x1fe] == 0x55)) &&
        (msc0_buf[0x1ff] == 0xaa)) &&
       (((iVar2 = strncmp(msc0_buf + 0x36,s_FAT12_800c8bd0,5), iVar2 == 0 ||
         (iVar2 = strncmp(msc0_buf + 0x36,s_FAT16_800c8bd8,5), iVar2 == 0)) &&
        (((uint)CONCAT11(msc0_buf[0x14],msc0_buf[0x13]) * msc0_buf._32_4_ == 0 &&
         ((byte)(msc0_buf[0x10] - 1) < 2)))))) break;
```

So it only supports FAT12 and FAT16 file system types.
That's fine, we can create two FAT16 partitions at the block offset we found earlier.
Make sure the size is suitable so it doesn't overwrite later data.

Files from `packet1.dat` look like system files, so put them in partition 0.
And files from `packet2.dat` look like data files for various programs, so that goes into partition 1.

Starting up QEMU again, now the touchscreen calibration screen looks proper:

![Touchscreen calibration!](<Screenshot 2026-05-13 025414.png>)

After calibration, it then complains that clock needs to be reconfigured:

![Please reconfigure clock](<Screenshot 2026-05-13 025527.png>)

However, this doesn't work, the code just complains endlessly about some sort of exception issue and gets stuck:

```
TaskMain>>
启动的部分 = %d
处理没有消息的情况
GUI加载完毕1
===%s
文件读写有问题
tmpdeskinfo.DT_InSideRes = %8.8X        tmpdeskinfo.DT_DIYRes = %8.8X
===jump to GuiOsPtr===

CAUSE=%8.8X EPC=%8.8X
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
=============_MM_InterMedium_CloseAll===========
```

## Filesystem issues

Earlier in the kernel prints, it complains about some sort of file access issue:

```
GUI加载完毕1
===%s
文件读写有问题
```

Let's see what that's about, the function at `0x80045e50`:

```c
    file = (file_t *)fopen(path,s_rb_800cb578);
    if (file == (file_t *)0x0) {
      dbg_printf("===%s\n",path);
      dbg_printf("文件读写有问题\n");
      return 0xffffffff;
    }
```

We can add another gdb `dprintf` to see what it's trying to print.
Similarly, we can add prints around `fopen()` functions etc..

```bash
# File access error
dprintf *0x80045f50, "=== %s\n", $s2
# fopen()
dprintf *0x8002a8e0, "%#010x: fopen(\"%s\", \"%s\")\n", $ra, $a0, $a1
dprintf *0x8002a960, "%#010x: fopen() = %#010x\n", $ra, $v0
# fclose()
dprintf *0x8003591c, "%#010x: fclose(%#010x)\n", $ra, $a0
```

```
GUI加载完毕1
0x80045e90: fopen("A:\系统\数据\Shell\imedat.dlx", "rb")
0x80045e90: fopen() = 0000000000
=== A:\系统\数据\Shell\imedat.dlx
===%s
文件读写有问题
```

Adding more and more debug prints, it seems for the file path `A:\系统\数据\Shell\imedat.dlx`, it is able to find the drive and the `系统` directory, but unable to find the `数据` directory.

Because FAT16 uses UCS2 encoding for long file support, like UTF-16, each character takes 2 bytes.
So we need to enable a temporary UCS2 to GBK charset conversion and use `x/sh` to print them out.

```bash
b *0x8002ed6c
set $long_name_last = 'a'
commands
    silent
    set target-charset UCS2
    printf "%#010x: long name = ", $pc
    x/sh $s4
    set target-charset GBK
    if *(unsigned long *)$s4 != 0 || $long_name_last != 0
        set $long_name_last = *(unsigned long *)$s4
        c
    end
end
```

```c
GUI加载完毕1
0x80045e90: fopen("A:\系统\数据\Shell\imedat.dlx", "rb")
strlen(0xcf5c3a41 = "A:\系统\数据\Shell\imedat.dlx")
strlen(0xcf5c3a41 = "A:\系统\数据\Shell\imedat.dlx")
0x8002ed6c: long name = 0x809f1720:     u"系统"
strlen(0xddbefdca = "数据\Shell\imedat.dlx")
ra:0x80037624 -> 0x8003ab60: fs_read_sector(0, 0x00000010, 0x00000001, 0x809ece00)
ra:0x80037028 -> 0x8003ab60: fs_read_sector(0, 0x00000210, 0x00000040, 0x80cd0080)
0x8002ed6c: long name = 0x809f1720:     u""
0x8002ed6c: long name = 0x809f1720:     u""
```

This is strange, because earlier, it was able to find files like `A:\系统\数据\shell\touchpanel.dlx` for showing the touchscreen calibration, which sits in the same directory:

```c
0x80044784: fopen("A:\系统\数据\shell\touchpanel.dlx", "rb")
strlen(0xcf5c3a41 = "A:\系统\数据\shell\touchpanel.dlx")
strlen(0xcf5c3a41 = "A:\系统\数据\shell\touchpanel.dlx")
0x8002ed6c: long name = 0x809f0ee0:     u"系统"
strlen(0xddbefdca = "数据\shell\touchpanel.dlx")
ra:0x80037028 -> 0x8003ab60: fs_read_sector(0, 0x000001e0, 0x00000010, 0x809eeee0)
0x8002ed6c: long name = 0x809f0ee0:     u"配置"
0x8002ed6c: long name = 0x809f0ee0:     u"数据"
strlen(0x6c656873 = "shell\touchpanel.dlx")
ra:0x80037028 -> 0x8003ab60: fs_read_sector(0, 0x00000df0, 0x00000010, 0x809eeee0)
0x8002ed6c: long name = 0x809f0ee0:     u"26字母.bin"
0x8002ed6c: long name = 0x809f0ee0:     u"alarm.mp3"
...
```

After a lot more complicated debugging, eventually, I found out that in function `0x800303d8`:

```c
uint32_t find_offset?(ushort partition,uint32_t param_2)
{
  if (partition < 2) {
    return (param_2 - 2) * (uint)sec_per_clus + fat_offset?[(short)partition];
  }
  return (param_2 - 2) * (uint)mmc_sec_per_clus + DAT_8019f440;
}
```

The same global variable `sec_per_clus` is used for both partition 0 and 1.

It is written by `fs_init()`:

```c
  num_fats = msc0_buf[0x10];
  sec_per_clus = msc0_buf[0xd];
```

Take a look at [FAT16 specs](https://teslabs.com/openplayer/docs/docs/specs/fat16_specs.pdf), the field at offset `0x0d` means sectors per cluster:

![FAT16 Boot Record](<Screenshot 2026-05-23 003357.png>)

The buggy filesystem driver uses the same global variable for both partitions, we must format both of them with exactly the same parameters 🙄.
If someone formats the user data partition on a PC with the wrong parameters, it can breaks the whole system.

I've tried to only match the sectors per cluster parameter, however somehow that doesn't work when partition 1 is larger than 1 GiB.
We need to look into how the BBK software formats the partitions, and match it exactly.

The partition formatting function can be found at `0x80032ba0`:

```c
undefined4 format_partition(int i_part,uint8_t *part_label,code *param_3)
{
  ...
          memcpy(p_buf,&fat_part_header,0x200);
          memcpy(p_buf + 0x2b,s_H2_V2.2L_800c8b8c,0xb);
          uVar4 = num_sectors_in_part[i_part];
          uVar2 = num_sectors_in_part[i_part];
                    /* num reserved sectors */
          p_buf[0xe] = 0x40;
                    /* sectors per FAT */
          p_buf[0x17] = (uint8_t)((uint)iVar10 >> 8);
                    /* num of sectors */
          p_buf[0x20] = (uint8_t)uVar2;
          p_buf[0x21] = (uint8_t)(uVar4 >> 8);
          p_buf[0x22] = (uint8_t)(uVar4 >> 0x10);
          p_buf[0x23] = (uint8_t)(uVar4 >> 0x18);
                    /* sectors per cluster */
          p_buf[0xd] = 0x40;
          p_buf[0xf] = 0x0;
                    /* sectors per FAT */
          p_buf[0x16] = (uint8_t)iVar10;
          iVar7 = fs_write_sector(i_part,0,1,p_buf);
```

It uses `fat_part_header` as the base, then updates a few parameters on top.

![fat_part_header, u8[512]](<Screenshot 2026-05-23 004228.png>)

So, the FAT16 partitions must be formatted exactly as:

Parameter                      | Value
-------------------------------|------
Format                         | FAT16
Bytes Per Sector               | 512
Sectors Per Cluster            | 64
Reserved Sectors               | 64
Maximum Root Directory Entries | 512
Sectors Per Track              | 32
Number of Heads                | 64
Number of Hidden Sectors       | 1

This can be done using `mkfs.vfat`:

```python
    if subprocess.run(debug_args(["mkfs.vfat", "-F", "16", "-a",
                                  "-S", "512", "-s", "64", "-R", "64",
                                  "-g", "64/32", "-h", "1", "-r", "512",
                                  "-n", label, lo])).returncode != 0:
        raise RuntimeError("Failed to format FAT partition")
```

Now, it is able to open these files:

```c
GUI加载完毕1
0x80045e90: fopen("A:\系统\数据\Shell\imedat.dlx", "rb")
0x80045e90: fopen() = 0x809f12a0
0x80045e90: fopen("A:\系统\数据\shell\dict_last.dlx", "rb")
0x80045e90: fopen() = 0x809f1300
0x80045e90: fopen("A:\系统\数据\shell\ZYBRes.dlx", "rb")
0x80045e90: fopen() = 0x809f1360
tmpdeskinfo.DT_InSideRes = %8.8X        tmpdeskinfo.DT_DIYRes = %8.8X
===jump to GuiOsPtr===
```

However, it is still looping inside the exception handler.

# Main OS program

Go the where the last string `"===jump to GuiOsPtr==="` was printed, we can find a function at `0x80020b9c`:

```c
void task_os(void)
{
  int iVar1;
  undefined4 uVar2;
  code *pcVar3;

  dbg_printf(s_TaskMain>>_800af0d8);
  iVar1 = fill_os_params();
  if (iVar1 != 0) {
    do {
      dbg_printf(s_=====TaskMain_os_load_failed====_800af0e8);
    } while( true );
  }
  FUN_800284fc();
  FUN_80048918();
  FUN_80045a00();
  uVar2 = FUN_800495c0();
  pcVar3 = (code *)0x80000000;
  do {
    cacheOp(1,pcVar3);
    pcVar3 = pcVar3 + 0x20;
  } while (pcVar3 < _entry);
  pcVar3 = (code *)0x80000000;
  do {
    cacheOp(0,pcVar3);
    pcVar3 = pcVar3 + 0x20;
  } while (pcVar3 < _entry);
  FUN_80043588();
  dbg_printf(s_===jump_to_GuiOsPtr===_800af10c);
  FUN_804af000(uVar2);
  return;
}
```

Looks like it attempts to load something, then runs the function at address `0x804af000`.

After staring at those functions for some more time, eventually I figured out enough of the function at `0x800451f4`, which I named `fill_os_params()`:

```c
  boot_part = _RTC.HSPR & 1;
  dbg_printf(s_HAN#=_%d_800cb474,boot_part);
  uVar6 = 0;
  puVar4 = app_block_offset_array + boot_part * 2;
  iVar5 = 0;
  psVar3 = boot_os;
  do {
    app_entry = *(uint32_t *)((int)app_entry_array + iVar5);
    uVar1 = *puVar4;
    app_size = *(uint32_t *)((int)app_size_array + iVar5);
    uVar2 = *(uint32_t *)((int)PTR_ARRAY_80190c30 + iVar5);
    psVar3->field0_0x0 = uVar6;
    uVar6 = uVar6 + 1;
    psVar3->entry_addr = app_entry;
    psVar3->block_offset = uVar1;
    psVar3->size = app_size;
    psVar3->field11_0x2c = uVar2;
```

It defines two OS programs, with entry addresses at `0x804af000` and `0x8086b000` respectively, loaded from `MSC0`.
Similar to the kernel, it also supports AB switching.
One set of block offsets are at `0x2000` (`0x00400000`: 4 MiB) and `0x6400` (`0x00c80000`: 12.5 MiB).
Another set of block offsets are at `0x9800` (`0x01300000`: 19 MiB) and `0xdc00` (`0x01b80000`: 27.5 MiB).

Using Ghidra, we can check that `data3_L.dat` does load correctly at `0x804af000`, and `data4_L.dat` loads correctly at `0x8086b000`.
Now we have the data to copy to those block offsets, we can update the `MSC0` system image again.

# Timer interrupts

Booting again, it progressed further, but still gets stuck in a different way.
This time it seems it got stuck in an interrupt handler.

```bash
===jump to GuiOsPtr===
tpIAL_Input = 0x%8.8X   0x%8.8X



GUIRegistToTinyOs


通知GUI可以使用了
hello
我是GUI————二号
T_GAL_pGC Open
xRes = %d ,%d
LineLen = %d
0x805f49fc: fopen("A:\系统\数据\HZK_LIB.BIN", "rbf")
0x805f49fc: fopen() = 0x809f1f80
A       B       C       D       55      66      InitGUI OK
fnGUI_IsRTCTimeValid = %d
%d, %d, %d, %d, %d, %d

tem_day = %d
设置时间为:y=%d,m=%d,w=%d,d=%d,h=%d,m=%d,s=%dfnGUI_SetNextAlarm();
start>>>>>>>>
alarm file create!
0x800131cc: fopen("a:\应用\数据\alarm.db", "rb")
0x800131cc: fopen() = 0000000000
alarm file not exist or file length error!!
0x80013268: fopen("a:\应用\数据\alarm.db", "wb")
0x80013268: fopen() = 0x809f34e0
0x8001338c: fclose(0x809f34e0)
get systime!
0x80013f18: fopen("a:\应用\数据\alarm.db", "rb")
0x80013f18: fopen() = 0x809f34e0
alarm.newestid :%d
alarm file  FILELEN : %d
memcpy end
alarm.newestid :%d
setsuc_flag(%d)
0x80013ff4: fclose(0x809f34e0)
Get the irq%d of DMA without irq handler, STOP here for ever
ICMR= 0x%08x
ICPR= 0x%08x
```

By placing a breakpoint in gdb at `0x800064e4`, the function that falls into with unhandled interrupts, we can check what interrupt was triggered.

```
(gdb) x/xw 0xb0001010
0xb0001010:     0x00200000
```

`ICPR = 0x00200000` means interrupt 21 i.e. `TCU2` is currently pending.

Following the exception handler routine, we can find the function at `0x80006598`:

```c
uint FUN_80006598(void)
{
  int iVar1;
  uint uVar2;
  uint uVar3;

  uVar2 = 0x1e;
  if (((_INT.ICPR & 0x40000000) == 0) &&
     ((_INT.ICPR != 0 || (uVar2 = 0xffffffff, _DAT_b3016000 == 0)))) {
    for (uVar2 = 0x1f; (_INT.ICPR & 1 << (uVar2 & 0x1f)) == 0; uVar2 = uVar2 - 1) {
    }
    if ((_INT.ICPR & 0x30e1f800) != 0) {
      if ((_INT.ICPR & 0x30000000) == 0) {
        if ((_INT.ICPR & 0x200000) == 0) {
          if (uVar2 < 0x11) {
            iVar1 = (0x10 - uVar2) * 0x100;
            uVar3 = 0x1f;
            do {
              if (((int)(*(uint *)(iVar1 + -0x4ffeff80) & ~*(uint *)(iVar1 + -0x4ffeffe0)) >>
                   (uVar3 & 0x1f) & 1U) != 0) break;
              uVar3 = uVar3 - 1;
            } while (-1 < (int)uVar3);
            uVar2 = uVar3 + (0x10 - uVar2) * 0x20 + 0x31;
          }
        }
        else {
          uVar2 = 1;
          do {
            if ((_DAT_b0002020 & ~_DAT_b0002030 & (1 << (uVar2 + 0x10 & 0x1f) | 1 << (uVar2 & 0x1f))
                ) != 0) break;
            uVar2 = uVar2 + 1;
          } while (uVar2 < 6);
          uVar2 = uVar2 + 0x20;
        }
      }
      else {
        uVar3 = (uint)((_INT.ICPR & 0x20000000) == 0);
        uVar2 = 0;
        do {
          if ((*(uint *)(&DAT_b3020304 + uVar3 * 0x100) & 1 << (uVar2 & 0x1f)) != 0) break;
          uVar2 = uVar2 + 1;
        } while (uVar2 < 6);
        uVar2 = uVar2 + uVar3 * 6 + 0x25;
      }
    }
  }
  return uVar2;
}
```

When `ICPR = 0x00200000`, it checks the timer flag register `0xb0002020` for timer events 1 to 5, that are not masked by register `0xb0002030`.

```c
(gdb) x/xw 0xb0002020
0xb0002020:     0x00210021
(gdb) x/xw 0xb0002030
0xb0002030:     0x003d803c
```

So, timer 0 full comparison match interrupt is pending, but somehow the interrupt handler does not process it.
We can check that the software does intend to set up the timer 0 with interrupts enabled, it must be the emulator that's wrong.

From the code, we can deduct that:
`TCU0` triggers interrupt handler 21, does not seem used by the BBK software. Guessing it may be used by the `OST`.
`TCU1` triggers interrupt handler 22, registered by timer 0 init code.
`TCU2` is the combined interrupt for timer 1 to timer 5.

It seems JZ4750L requires a new `TCU` timer model in the emulator, that I can fix.
This interrupt connection is different to both JZ4740 and JZ4750.

# API calls

The emulator progresses further after timer interrupts fixed, however it still gets stuck at an exception handler:

```
设置时间为:y=%d,m=%d,w=%d,d=%d,h=%d,m=%d,s=%dfnGUI_SetNextAlarm();
start>>>>>>>>
alarm file create!
alarm file not exist or file length error!!
get systime!
alarm.newestid :%d
alarm file  FILELEN : %d
memcpy end
alarm.newestid :%d
setsuc_flag(%d)

===================================================================
  准备进入音频任务主入口
  查看当前正在使用的内存情况：
  1:
  内存地址：0x809f32c0
  分配大小：申请了30828Bytes实际分配了30836Bytes
  分配位置：XMediaPlayerRegister.cpp文件中的第19行

  有1处内存正在使用
  共申请30828Bytes，分配了30836Bytes
===================================================================

bda不存在

mmc_get_clu_cache: %d is free!
进行初始化操作
DownPtr = %x
通知APP可以使用了
辞典模块注册成功


mmc_get_clu_cache: %d is free!

mmc_get_clu_cache: %d is free!

mmc_get_clu_cache: %d is free!
手写库已经被打开
fnGUI_SetCurIme(int type) = 0x%x
INSIDE_RES = %8.8x, DIY_RES = %8.8x
%d, %d, %d, %d, %d, %d

tem_day = %d
设置时间为:y=%d,m=%d,w=%d,d=%d,h=%d,m=%d,s=%d%s

Breakpoint 4, 0x80008644 in ?? ()
(gdb) c
Continuing.

CAUSE=%8.8X EPC=%8.8X
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x
%4s %08x %4s %08x %4s %08x %4s %08x =============_MM_InterMedium_CloseAll===========
```

By enabling logging for `fopen()`, we can see:

```
设置时间为:y=%d,m=%d,w=%d,d=%d,h=%d,m=%d,s=%d%s
0x804c4c00: fopen("A:\应用\程序\中学时间.bda", "rb")
0x804c4c00: fopen() = 0x809fbf40
0x804c4de0: fclose(0x809fbf40)

Breakpoint 7, 0x80008644 in ?? ()
```

It loads in the application file `A:\应用\程序\中学时间.bda` to reconfigure the clock.
That file we can find from `packet1.dat`, but we need to understand how to load it.

The return address `0x804c4c00` sits inside `data3_L.dat`, not the kernel.
So we need to load `data3_L.dat` into Ghidra to take a look.

```c
  iVar6 = 0;
  (*gops->dbg_printf)(s_%s_80652b28,param_1);
  iVar1 = (*fops->fopen)(param_1,s_rb_80652b2c);
  if (iVar1 == 0) {
    FUN_8058ce18(0,s_HAN#_80652d74,s_HAN#_80652b5c,0);
  }
  else {
    (*fops->fread)(buf,1,0x88,iVar1);
    p_buf = (uint *)buf;
```

The code here looks like it's trying to call kernel functions by using pointers to arrays of functions.
We can find that these pointers are initialised by the function at `0x804afa20`:

```c
  fops = _DAT_81c30008;
  op3 = _DAT_81c30010;
  gops = _DAT_81c30018;
  DAT_8080eba0 = _DAT_81c3000c;
  DAT_8080eb98 = _DAT_81c30028;
  DAT_8080eb9c = _DAT_81c3002c;
```

Addresses like `0x81c30018` takes us back into the kernel code.
Function at `0x80042b80` copies 16 bytes of data from `0x800cae04` to `0x81c30000`:

```c
void api_fp_init(void)
{
  void *pvVar1;
  void **ppvVar2;
  undefined4 *puVar3;
  int iVar4;

  iVar4 = 0x10;
  puVar3 = &DAT_800cae04;
  ppvVar2 = PTR_ARRAY_81c30000;
  do {
    pvVar1 = (void *)*puVar3;
    iVar4 = iVar4 + -1;
    puVar3 = puVar3 + 1;
    *ppvVar2 = pvVar1;
    ppvVar2 = ppvVar2 + 1;
  } while (iVar4 != 0);
  return;
}
```

Indeed, the data at `0x800cae04` contains pointers to array of function pointers:

```c
                             DAT_800cae04                                    XREF[1]:     api_fp_init:80042b94(R)
        800cae04 00 00 00 00     undefined4 00000000h
                             DAT_800cae08                                    XREF[1]:     api_fp_init:80042b94(R)
        800cae08 00 00 00 00     undefined4 00000000h
        800cae0c 50 a9 0c 80     addr       fops                                             = 8002a8e0
        800cae10 30 a8 0c 80     addr       op3                                              = 80054090
        800cae14 44 ae 0c 80     addr       gops

                             fops                                            XREF[1]:     800cae0c(*)
        800ca950 e0 a8 02 80     addr       fopen
        800ca954 1c 59 03 80     addr       fclose
        800ca958 6c 59 03 80     addr       fread
        800ca95c 40 5b 03 80     addr       fwrite
        800ca960 8c b2 02 80     addr       fseek
        800ca964 38 5c 03 80     addr       ftell

                             gops                                            XREF[1]:     800cae14(*)
        800cae44 58 2c 04 80     addr      FUN_80042c58
        800cae48 60 2c 04 80     addr      FUN_80042c60
        800cae4c b8 5a 02 80     addr      malloc
        800cae50 54 56 02 80     addr      free
```

Now we understood how API calls are connected, we can read the code from `data3_L.dat`.

## `.bda` file format

Back to the `.bda` file loading function at `0x804c4bac`:

```c
    (*fops->fread)(&buf,1,0x88,iVar1);
    p_buf = (uint8_t *)&buf;
    iVar2 = 10;
    pbVar2 = (uint32_t *)p_buf;
    do {
      iVar2 = iVar2 + -1;
      *pbVar2 = *pbVar2 ^ DAT_80652b14;
      pbVar2 = pbVar2 + 1;
    } while (-1 < iVar2);
    uVar3 = 0;
    buf.checksum = buf.checksum ^ DAT_80652b1c;
    do {
      uVar3 = uVar3 + 1;
      uVar5 = uVar5 + (byte)((bda_t *)p_buf)->magic1[0];
      p_buf = (uint8_t *)(((bda_t *)p_buf)->magic1 + 1);
    } while (uVar3 < 0x84);
    if (((buf.magic2 == 0x5d245562) && (iVar2 = strcmp(buf.magic1,s_BBK_80652b24), iVar2 == 0)) &&
       (buf.checksum == uVar5)) {
      (*fops->fseek)(iVar1,0,2);
      file_size = (*fops->ftell)(iVar1);
      if (file_size < 0x400001) {
        (*fops->fseek)(iVar1,buf.data_offset,0);
        iVar2 = strcmp(s_HAN#C:\_\_.bda_80652b84,param_1);
```

So the first `0x88` number of bytes in the `.bda` application files are the file header, encrypted by XOR'ing with `*0x80652b14 = 0x44525744`.
The checksum XOR is `*0x80652b1c = 0x322d464b`.
A couple of magic signature checks at the beginning.
And data starting offset can be found at offset `0x14`.

Generally, the entry address seems to be `0x81c30040`.
But there are also a couple of special cases depending on the file path.
This is good enough for extracting and loading the executable binary data from the `中学时间.bda` application into Ghidra.

## SDRAM wrapping and hardware revisions

Now we can check why exception handler was triggered by this application.

The application has a similar structure to the OS program.

First, it also fills in arrays of function pointers:

```c
  ops = DAT_83c30004;
  DAT_81c4c698 = DAT_83c30008;
  DAT_81c4c6a0 = DAT_83c30010;
  DAT_81c4c694 = DAT_83c3000c;
  DAT_81c4c680 = DAT_83c30014;
  DAT_81c4c69c = DAT_83c30018;
  DAT_81c4c68c = DAT_83c30028;
  DAT_81c4c690 = DAT_83c3002c;
  DAT_81c4c684 = DAT_83c30030;
```

Later, it tries to call the function, that's where exception handler was triggered.

```c
  (*(code *)*DAT_81c4c69c)("----DL_Main--------\n");
```

Looking at the memory data around offset `0x83c30018` in the emulator, we can see it's all zeros:

```c
(gdb) x/8xw 0x83c30000
0x83c30000:     0x00000000      0x00000000      0x00000000      0x00000000
0x83c30010:     0x00000000      0x00000000      0x00000000      0x00000000
```

I could not find any references to this address range in any of the previous programs: first stage loader, kernel and OS program.
Then I suddenly realised, if SDRAM is only 32 MiB, accessing this `0x83c30018` address would wrap back to `0x81c30018`.
`0x81c30018` is also used by the OS program to call kernel API functions, so this makes sense.

Easy fix, reduce SDRAM size to 32 MiB, and:

![The clock!](<Screenshot 2026-05-16 230035.png>)

YAY! The application runs!

Quit out of the application, the desktop appears:

![The desktop appears!](<Screenshot 2026-05-23 035440.png>)

So, the JZ4750L hardware revision has smaller 32 MiB SDRAM size, that's interesting.
The kernel and OS program code for the JZ4740 hardware revision does make use of RAM addresses like `0x83c30018`, so the JZ4740 hardware revision must have a larger 64 MiB SDRAM size.

## Hardware revision specific applications

The system settings application actually fails to load:

![Failed to load application](<Screenshot 2026-05-23 035651.png>)

We can see that the application file path is:

```c
0x804c4c00: fopen("A:\应用\程序\中学系统设置.bda", "rb")
```

Inside `应用/程序` directory from `packet1.dat`, there isn't a file named `中学系统设置.bda`, but two files instead:
`中学系统设置.bda_4720` and `中学系统设置.bda_4750l`.

Looks like some `.bda` files are actually hardware revision specific.
The USB recovery tool must have some special handling to rename the files.
That's easy to fix, I can rename the files when writing system image partitions too.

Now it runs fine:

![The settings application](<Screenshot 2026-05-23 042505.png>)

# Is RTC time valid?

I'd like to find out how it checks whether current time is valid, so I don't need to reconfigure clock every time.

This is easy, find the `fnGUI_IsRTCTimeValid` function at `0x8001d5c4`:

```c
bool fnGUI_IsRTCTimeValid(void)
{
  return (_RTC.HSPR & 0xfffffffe) == 0x12345678;
}
```

The `RTC.HSPR` register should preserve its content across reboots.
Just need to initialise its content as `0x12345678`.
Note as we discovered earlier, bit 0 is also used for AB partition switching.

# JZ4750L CPU identification

The recovery package contains software version V2.20.
There is also software version V3.20 available from "视频学习机_H2_V3.20_一卡系统恢复", I'd like to check it out.
Since it is loaded from SD card on `MSC1`, just need to make a `MSC1` image with the update program extracted there.

It does prompt for system update straight after booting, however, update fails:

![Update failed](<Screenshot 2026-05-17 004911.png>)

Looking at the log, it appears to be trying to access a non-existent NAND flash?
I do know that the JZ4740 SoC must be booting from NAND, so it seems it is confused about the hardware revision somehow.
Great, now I need to debug the system update program too.

Extract executable binary from the `系统恢复.bda` file, load into Ghidra, following the bad block prints, I came cross this code segment:

```c
          // 0x81c306bc
          iVar1 = get_gPrid();
          if (iVar1 == 0xad0024f) {
            nand_upd();
          }
          (*(code *)DAT_81c40378[0x17])(0,0x81c3e9a8,0);
          (*(code *)DAT_81c40378[0x17])(1,&DAT_81c3e9ac,0);
          iVar1 = get_gPrid();
          if ((iVar1 == 0x1ed0024f) && (iVar1 = (**(code **)&ops->field_0x230)(), iVar1 == 0x221)) {
            uVar2 = (*(code *)DAT_81c40378[0x16])(0);
            (*ops->dbg_printf)(s_wuzhx:)_%s_%d__81c3e7f8,s_fileOperate.c_81c3e7c0,1099);
            (*ops->dbg_printf)(s_fs_init(0)_=_%d_81c3e9b0,uVar2);
            uVar2 = (*(code *)DAT_81c40378[0x16])(1);
            (*ops->dbg_printf)(s_wuzhx:)_%s_%d__81c3e7f8,s_fileOperate.c_81c3e7c0,0x44d);
            (*ops->dbg_printf)(s_fs_init(1)_=_%d_81c3e9c4,uVar2);
            (*unaff_s6)();
          }
```

```c
// 0x81c39c3c
void update_gPrid(void)
{
  gPrid = PRId;
  if (PRId == 0xad0024f) {
    FUN_81c35030();
  }
  else if (PRId == 0x1ed0024f) {
    FUN_81c32838(4);
  }
  return;
}
```

So far, I've been emulating the `CPU.PRId` register with value `0x00ad0024f`, the value from Ingenic JZ4750 PM 2.2 Extra Features of CPU core. JZ4740 PM also has the same value.
Either the document is wrong, or the register value on the JZ4750L SoC is different.
I think it is more likely the document is wrong, JZ4750 CPU probably all has `0x01ed0024f`.

# SD vs. eMMC

After corrected the `CPU.PRId` register value, system update still fails, but for a different reason:

![Update failed again](<Screenshot 2026-05-17 013720.png>)

Add some debugging around `MSC0` commands:

```c
-------------------退出消息框------------------
ra:0x81c34ac4 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=4294967295, arg=0000000000, nb=0000000000)
Mmc_controller.c CpmSelectMscHSClk:clock = %d,div = %d,A_CPM_MSCCDR[0x%x]
set clock to %u Hz is_sd=%d clkrt=%d A_MSC_CLKRT[0x%x]
ra:0x81c34ae8 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=0, arg=0000000000, nb=0000000000)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=1, arg=0x00ff8000, nb=0000000000)
MMC/SD timeout, MMC_STAT 0x%x CMD %d
ra:0x81c34d14 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=55, arg=0000000000, nb=0000000000)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=41, arg=0x40300000, nb=0000000000)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=2, arg=0000000000, nb=0000000000)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=3, arg=0x00000010, nb=0000000000)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=9, arg=0x878d0000, nb=0000000000)

csd->wp_grp_size = 0x%x, csd->wp_grp_enable = 0x%x
ra:0x81c34764 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f940, cmd=7, arg=0x878d0000, nb=0000000000)
ra:0x81c3482c -> 0x81c3469c: upg_msc0_cmd(p=0x8039f940, cmd=55, arg=0x878d0000, nb=0000000000)
ra:0x81c3486c -> 0x81c3469c: upg_msc0_cmd(p=0x8039f940, cmd=6, arg=0x00000002, nb=0000000000)
Use 4-bit bus width
Mmc_controller.c CpmSelectMscHSClk:clock = %d,div = %d,A_CPM_MSCCDR[0x%x]
set clock to %u Hz is_sd=%d clkrt=%d A_MSC_CLKRT[0x%x]
ra:0x8003ce34 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd0, cmd=13, arg=0xc0260000, nb=0000000000)
MMC/SD timeout, MMC_STAT 0x%x CMD %d
ra:0x8003f2b4 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=4294967295, arg=0000000000, nb=0000000000)
Mmc_controller.c CpmSelectMscHSClk:clock = %d,div = %d,A_CPM_MSCCDR[0x%x]
set clock to %u Hz is_sd=%d clkrt=%d A_MSC_CLKRT[0x%x]
ra:0x8003f2d8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=0, arg=0000000000, nb=0000000000)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=1, arg=0x00ff8000, nb=0000000000)
MMC/SD timeout, MMC_STAT 0x%x CMD %d
ra:0x8003f4d8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=55, arg=0000000000, nb=0000000000)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=41, arg=0x40300000, nb=0000000000)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=2, arg=0000000000, nb=0000000000)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=3, arg=0x00000010, nb=0000000000)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=9, arg=0x7b540000, nb=0000000000)

csd->wp_grp_size = 0x%x, csd->wp_grp_enable = 0x%x
ra:0x8003ef54 -> 0x8003ee88: krn_msc0_cmd(p=0x8039ef30, cmd=7, arg=0x7b540000, nb=0000000000)
ra:0x8003f01c -> 0x8003ee88: krn_msc0_cmd(p=0x8039ef30, cmd=55, arg=0x7b540000, nb=0000000000)
ra:0x8003f05c -> 0x8003ee88: krn_msc0_cmd(p=0x8039ef30, cmd=6, arg=0x00000002, nb=0000000000)
Use 4-bit bus width
Mmc_controller.c CpmSelectMscHSClk:clock = %d,div = %d,A_CPM_MSCCDR[0x%x]
set clock to %u Hz is_sd=%d clkrt=%d A_MSC_CLKRT[0x%x]
ra:0x8003ce34 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd0, cmd=13, arg=0x7b540000, nb=0000000000)
...
ra:0x8003cf5c -> 0x8003ee88: krn_msc0_cmd(p=0x8039ef30, cmd=13, arg=0x7b540000, nb=0000000000)
wuzhx:)_%s_%d_  lStartBlock = %d, lEndBlock = %d
ra:0x81c32308 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9c8, cmd=13, arg=0x878d0000, nb=0000000000)
MMC/SD timeout, MMC_STAT 0x%x CMD %d
```

These are the important bits:

```c
// Update program asks the card to publish a new relative address (RCA)
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=3, arg=0x00000010, nb=0000000000)
// The new RCA is 0x878d0000
ra:0x81c34c04 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9e8, cmd=9, arg=0x878d0000, nb=0000000000)
// Kernel tries to access the card using RCA = 0xc0260000
ra:0x8003ce34 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd0, cmd=13, arg=0xc0260000, nb=0000000000)
// That failed, since RCA has been changed by the update program
MMC/SD timeout, MMC_STAT 0x%x CMD %d
// Kernel asks the card to publish a new relative address (RCA)
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=3, arg=0x00000010, nb=0000000000)
// The new RCA is 0x7b540000
ra:0x8003f3e8 -> 0x8003ee88: krn_msc0_cmd(p=0x8039efd8, cmd=9, arg=0x7b540000, nb=0000000000)
// Update program is still using RCA = 0x878d0000
ra:0x81c32308 -> 0x81c3469c: upg_msc0_cmd(p=0x8039f9c8, cmd=13, arg=0x878d0000, nb=0000000000)
// That failed, since RCA has been changed again by the kernel
MMC/SD timeout, MMC_STAT 0x%x CMD %d
```

So far, I've been using SD card mode for the system image attached to `MSC0`.
Looks like the update program has a separate set of `MSC0` access functions, that are conflicting with kernel, by requesting new random `RCA` as part of SD card initialisation process.

I know that during SD card initialisation process, one of the steps is to check whether the connected card is SD card or MMC.
Let's see how MMC works.

Specification documents can be found at:
[MMC System Specification](https://ptacts.uspto.gov/ptacts/public-informations/petitions/1489594/download-documents?artifactId=4hRzPSl_JCJ6nsyxaBA0ICb5znnBRnsl4HQ763Kn-S8mS23jKLq0nGc)
[SD Simplified Specifications](https://www.sdcard.org/downloads/pls/)

Different to SD card, MMC initialisation `CMD3` is `SET_RELATIVE_ADDR`, host assigns the address, instead of the card generating a random `RCA`.
Perhaps, if kernel and update program always assign the same `RCA`, MMC won't complain about timeouts.
This feels like a terrible hack, but BBK's software is probably as horrible as this 🙄.

And indeed, switching to eMMC mode works, it's updating now:

![System updating...](<Screenshot 2026-05-21 000152.png>)

And I get to see the V3.20 system:

![V3.20 desktop](<Screenshot 2026-05-21 012051.png>)

In system settings, there is also a cartoon theme mode (using OS program `data4_L.dat`), that doesn't seem to have changed much between V2.20 and V3.20:

![Cartoon themed desktop](<Screenshot 2026-05-21 010644.png>)

# Serial number encryption

Opening system version information application, it complains about serial number not found:

![Fetching serial number failed](<Screenshot 2026-05-16 231515.png>)

Let's go back to V2.20 to debug this, since all the breakpoint addresses I have in gdb scripts are only valid for V2.20.

Finding the serial number read function is not difficult, the function is `fnGUI_GetSNData()` at `0x8004fd60`.

```c
undefined4 fnGUI_GetSNData(int param_1)
{
  uint8_t *buf;
  undefined4 uVar1;
  int iVar2;
  int iVar3;
  uint8_t *puVar4;
  uint8_t *piVar5;
  int unaff_s5;
  undefined4 local_270 [10];
  int local_248;
  int local_244;
  int local_240;
  undefined1 stack_buf_sn [16];
  uint8_t stack_buf [512];

  dbg_printf(s_Real_fnGUI_GetSNData()_is_called_800cc220);
  buf = malloc(0x600);
  uVar1 = 0xffffffff;
  if (buf != (uint8_t *)0x0) {
    memset(buf,0xaa,0x600);
    iVar2 = FUN_8003f330(buf);
    if (iVar2 == 0) {
      rtc_get_time(local_270);
      srand(local_270[0]);
      iVar3 = rand();
      iVar2 = iVar3 + 7;
      if (-1 < iVar3) {
        iVar2 = iVar3;
      }
      stack_buf._468_4_ = iVar3 + (iVar2 >> 3) * -8;
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._472_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while (stack_buf._472_4_ == stack_buf._468_4_);
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._476_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while ((stack_buf._476_4_ == stack_buf._472_4_) || (stack_buf._476_4_ == stack_buf._468_4_))
      ;
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._480_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while (((stack_buf._480_4_ == stack_buf._476_4_) || (stack_buf._480_4_ == stack_buf._472_4_)
               ) || (stack_buf._480_4_ == stack_buf._468_4_));
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._484_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while (((stack_buf._484_4_ == stack_buf._480_4_) || (stack_buf._484_4_ == stack_buf._476_4_)
               ) || ((stack_buf._484_4_ == stack_buf._472_4_ ||
                     (stack_buf._484_4_ == stack_buf._468_4_))));
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._488_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while (((stack_buf._488_4_ == stack_buf._484_4_) || (stack_buf._488_4_ == stack_buf._480_4_)
               ) || ((stack_buf._488_4_ == stack_buf._476_4_ ||
                     ((stack_buf._488_4_ == stack_buf._472_4_ ||
                      (stack_buf._488_4_ == stack_buf._468_4_))))));
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._492_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while (((((stack_buf._492_4_ == stack_buf._488_4_) ||
                 (stack_buf._492_4_ == stack_buf._484_4_)) ||
                (stack_buf._492_4_ == stack_buf._480_4_)) ||
               ((stack_buf._492_4_ == stack_buf._476_4_ || (stack_buf._492_4_ == stack_buf._472_4_))
               )) || (stack_buf._492_4_ == stack_buf._468_4_));
      do {
        iVar3 = rand();
        iVar2 = iVar3 + 7;
        if (-1 < iVar3) {
          iVar2 = iVar3;
        }
        stack_buf._496_4_ = iVar3 + (iVar2 >> 3) * -8;
      } while ((((stack_buf._496_4_ == stack_buf._492_4_) ||
                (stack_buf._496_4_ == stack_buf._488_4_)) ||
               ((stack_buf._496_4_ == stack_buf._484_4_ ||
                (((stack_buf._496_4_ == stack_buf._480_4_ ||
                  (stack_buf._496_4_ == stack_buf._476_4_)) ||
                 (stack_buf._496_4_ == stack_buf._472_4_)))))) ||
              (stack_buf._496_4_ == stack_buf._468_4_));
      iVar3 = 0;
      dbg_printf(s_record_=_%d,%d,%d,%d,%d,%d,%d,%d_800cc244);
      iVar2 = 0;
      do {
        switch(*(undefined4 *)(stack_buf + iVar2 + 0x1d4)) {
        case 0:
          uVar1 = 0x7c00;
          break;
        case 1:
          uVar1 = 0x7e00;
          break;
        case 2:
          uVar1 = 0x8000;
          break;
        case 3:
          uVar1 = 0x8200;
          break;
        case 4:
          uVar1 = 0x7d00;
          break;
        case 5:
          uVar1 = 0x7f00;
          break;
        case 6:
          uVar1 = 0x8100;
          break;
        case 7:
          uVar1 = 0x8300;
          break;
        default:
          goto switchD_80050044_default;
        }
        unaff_s5 = msc0_read_sn(uVar1,buf + 0x200);
switchD_80050044_default:
        if (unaff_s5 == 0) {
          FUN_80050354(buf,0x100);
          puVar4 = buf + 0x200;
          iVar2 = 0x1f;
          piVar5 = (uint8_t *)&local_248;
          do {
            iVar2 = iVar2 + -1;
            FUN_80050898(puVar4,piVar5);
            puVar4 = puVar4 + 0x10;
            piVar5 = piVar5 + 0x10;
          } while (-1 < iVar2);
          if (((local_248 == 0x20101228) && (local_244 == 0x44313030)) &&
             ((local_240 == 0x5d245588 &&
              (iVar2 = strncmp(stack_buf,s_JZ4750L_800cc294,0x10), iVar2 == 0)))) {
            free(buf);
            if (param_1 != 0) {
              memcpy(param_1,&local_248,0x200);
            }
            dbg_printf(s_HAN#:_%s_800cc29c,stack_buf_sn);
            return 0;
          }
        }
        iVar3 = iVar3 + 1;
        iVar2 = iVar3 * 4;
      } while (iVar3 < 8);
      dbg_printf(s_read_sn_failed......no_sn_or_rea_800cc268);
      free(buf);
      uVar1 = 0xfffffffd;
    }
    else {
      free(buf);
      uVar1 = 0xfffffffe;
    }
  }
  return uVar1;
}
```

However, the decryption logic inside looks super complicated:

![What is this complicated logic?](<Screenshot 2026-05-16 234217.png>)

Later, in group chat, my friends are playing with AI tools.
As an exercise, we gave it the following prompts, let's see if AI can figure out how the serial number is encoded:

_`data2_L.dat`_
_这是运行在 JZ4750L soc 上的一段内核代码，加载地址是 0x80004000%_
_偏移量 0x8004fd60 的位置有读取序列号使用的函数，分析一下如何写入自定义的序列号，应该写入在 MMC 的什么位置_

![AI at work](OpenCode_serial_number.png)

It is surprisingly good at recognising the general structure of the code:

```c
## 3. 调用链
get_and_set_sn (0x800502D8)
  │
  ├─ read_sn (0x8004FD60)          ← 读取序列号
  │   ├─ kmalloc(0x600)            ← 分配1536字节缓冲区
  │   ├─ memset(buf, 0xAA, 0x600)  ← 填充0xAA
  │   ├─ mmc_init(buf) (0x8003F330)
  │   │   ├─ 检查 0x801A7084 == 0x3F
  │   │   ├─ memcpy(buf, 0x801A708C, 8) ← 复制AES密钥前8字节
  │   │   └─ return 0 (成功) / 1 (已初始化)
  │   ├─ srand(jiffies)
  │   ├─ 生成8个伪随机值 (对8取模, 互不重复)
  │   ├─ switch(rand_val) → mmc_read(sector, buf+0x200)  ← 读加密数据
  │   ├─ aes_key_expand(buf, 256) (0x80050354)  ← 密钥扩展
  │   ├─ for(i=0; i<32; i++)                      ← 32块解密
  │   │   aes_decrypt_block(buf+0x200+i*16, sp+80+i*16)
  │   ├─ 验证 Magic (0x20101228 / 0x44313030 / 0x5D245588)
  │   ├─ 提取序列号 (偏移0x1C, 16字节)
  │   └─ return
  │
  └─ prepare_sn_data (0x8005021C)  ← 构造明文数据块(写入时使用)
```

It is able to recognise functions like `malloc`, `memset`, able to recognise MMC access functions, and RTC functions.
The most useful thing is that it was able to recognise the encryption method looks like `AES-128-ECB`.

I can confirm using QEMU the code indeed generated a S-box used by AES.

![Rijndael S-box](<Screenshot 2026-05-21 132846.png>)

AI could not find the AES encryption key at `0x801a708c` from the binary, but this I can also set a breakpoint and dump it from QEMU:

![Dumping AES encryption key](<Screenshot 2026-05-21 122427.png>)

Unfortunately the Python script it created to modify the serial number simply didn't work ☹️, some more manual debugging is needed.

Looking at the AES key expansion function, it has made a mistake about the specific AES encryption method.
32 bytes of the AES key buffer was read, so this is `AES-256-ECB`, not `AES-128`.

Using `openssl`, I was also able to confirm `AES-256-ECB` is indeed correct.
The decrypted data from `openssl` is same as the data in decryption buffer:

```bash
openssl aes-256-ecb -d -K 212101deadbeef29aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa -in sn_seg.bin -out - | hd
```

![`openssl` decryption data matches](<Screenshot 2026-05-21 142558.png>)

After further prompting, AI also figured out that it should be `AES-256-ECB`, and produced a new version of the SN tool script.
Well, unfortunately that still didn't work.
It passed the first 3 magic checks, but failed the last `strncmp()` check:

![Still no serial number](<Screenshot 2026-05-23 140550.png>)

It has mistaken the `"JZ4750L"` string as some sort of `sscanf()` prefix, but no, it is part of the sanity checks.

After manually fixing up all the sanity checks, finally, I can see strings printed in the serial number field:

![Serial number shows something](<Screenshot 2026-05-21 160640.png>)

The string is random so that I can find out what's the actual offsets for them.

And now I can fill any string I like as the serial number:

![Custom serial number](<Screenshot 2026-05-23 141940.png>)

## Where is the AES key from?

The data at `0x801a708c` is:
```c
(gdb) x/8bx 0x801a708c
0x801a708c:     0x21    0x21    0x01    0xde    0xad    0xbe    0xef    0x29
```

But where is it actually coming from?

AI was not able to find it, but using QEMU + gdb breakpoints, we can find the code here:

```c
  // 8003f378
  switch(msc->cmd) {
  ...
  case 2:
    if (DAT_804877a4 == 0) {
      iVar3 = FUN_8003ec00(msc,&DAT_804877b8);
    }
    else {
      iVar3 = FUN_8003ecd4(msc,&DAT_804877b8);
    }
    if (iVar3 == 0) {
                    /* AES key */
      memcpy(&DAT_801a7084,&msc->aes_key,0x10);
    }
    else {
      memset(&DAT_801a7084,0,0x10);
      if (iVar3 != 0x14) {
        return 2;
      }
    }
```

It looks related to MMC `CMD2`, which is `ALL_SEND_CID`.
Let's check what values QEMU fills the MMC `CID` register with:

```c
// hw/sd/sd.c

/* Card IDentification register */

#define MID     0xaa
#define OID     "XY"
#define PNM     "QEMU!"
#define PRV     0x01
#define MDT_YR  2006
#define MDT_MON 2

static void emmc_set_cid(SDState *sd)
{
    sd->cid[0] = MID;       /* Fake card manufacturer ID (MID) */
    sd->cid[1] = 0b01;      /* CBX: soldered BGA */
    sd->cid[2] = OID[0];    /* OEM/Application ID (OID) */
    sd->cid[3] = PNM[0];    /* Fake product name (PNM) */
    sd->cid[4] = PNM[1];
    sd->cid[5] = PNM[2];
    sd->cid[6] = PNM[3];
    sd->cid[7] = PNM[4];
    sd->cid[8] = PNM[4];
    sd->cid[9] = PRV;       /* Fake product revision (PRV) */
    stl_be_p(&sd->cid[10], 0xdeadbeef); /* Fake serial number (PSN) */
    sd->cid[14] = (MDT_MON << 4) | (MDT_YR - 1997); /* Manufacture date (MDT) */
    sd->cid[15] = (sd_crc7(sd->cid, 15) << 1) | 1;
}
```

Mystery solved, it is just `CID` register bytes 7 to 14.
