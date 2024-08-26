---
title: Link-time array with libraries
tags: []
id: '1303'
categories:
  - - C/C++
  - - c-c
    - Embedded
  - - Linux
  - - embedded
    - Nosumor
date: 2019-11-10 13:01:17
---

Link time array is an interesting technique. Basically, by placing symbols in special sections, static array can be created by the linker.
<!-- more -->
First, assume we have some sections with KEEP and SORT applied, in gcc linker script:

\[code\]rolist : { . = ALIGN(4); KEEP(\*(SORT(.list.\*))) /\* Link time list support \*/ } >TEXT AT >LOAD\[/code\]

Things I tried but don't work: (Jump to next section if you don't case)

Having an array of size 0 placed in section `.list.name.0.start`, and items placed in section `.list.name.1.item`, finally an array of size 0 placed in section `.list.name.2.end`.

Assuming things work perfectly (which they never will), linker should sort these symbols in the order intended. Then we can iterate the array using address of the empty arrays at `start` and `end`.

However, gcc seems to assume symbols placed in different sections will never have the same address, and it will optimise away pointer comparisons, so iteration does not work. Maybe this makes sense for Harvard architecture?

Instead of an independent start and end sections, we can use implicit symbols GNU linker placed instead, e.g. `__start_rolist` and `__stop_rolist`.

However, this means we need a separate section for each list, and need to modify the linker script multiple times. This implicit linker symbol also seems poorly documented.

I also tried to use a termination item placed in the end section. Instead of address comparison, check the value instead. This wastes some spaces.

Ref: [list.h](https://github.com/zhiyb/Nosumor/blob/033d0863dbc5d20cc79de0f30b6b8a54f9a10b6d/device/common/list.h)

Another issue I encountered while trying to determine if an array is empty:

Test case: \[code language="cpp"\]const int \_\_start\_test\[0\] SECTION(".list.test.0.start"); const int \_\_item\_test\[1\] SECTION(".list.test.1.item") = {123}; const int \_\_end\_test\[1\] SECTION(".list.test.2.end") = {0};

static void debug() { const int \*p = &\_\_start\_test\[0\]; if (\*p == 0) dbgbkpt(); if (\_\_start\_test\[0\] == 0) dbgbkpt(); }\[/code\]

Compiles (with -O0) to: \[code language="cpp"\] 32 \[1\]const int \*p = &\_\_start\_test\[0\]; 0x802139e 17 4b ldrr3, \[pc, #92\]; (0x80213fc ) 0x80213a0 fb 60 strr3, \[r7, #12\] 33 \[1\]if (\*p == 0) 0x80213a2 fb 68 ldrr3, \[r7, #12\] 0x80213a4 1b 68 ldrr3, \[r3, #0\] 0x80213a6 00 2b cmpr3, #0 0x80213a8 02 d1 bne.n0x80213b0 34 \[1\]dbgbkpt(); 0x80213aa 00 f0 61 fe bl0x8022070 0x80213ae 00 be bkpt0x0000 35 \[1\]if (\_\_start\_test\[0\] == 0) 0x80213b0 00 23 movsr3, #0 0x80213b2 00 2b cmpr3, #0 0x80213b4 02 d1 bne.n0x80213bc 36 \[1\]dbgbkpt(); 0x80213b6 00 f0 5b fe bl0x8022070 0x80213ba 00 be bkpt0x0000 \[/code\]

That `movsr3, #0` certainly won't help...

* * *

Just remove the explicit input section specification in ld script seems to work best. By default, ld creates output sections same as input section, therefore creates the \_\_start\_SECTION and \_\_stop\_SECTION symbols.

Regarding placement, if all elements inside the array have `const` specified, the entire section will be marked as `READONLY`. By default, [ld should place them in the first available memory region with matching attributes](https://sourceware.org/binutils/docs/ld/Output-Section-Address.html). Hopefully(!), that's just after `.rodata`. For bare metal, that should be in the read-only flash region.

Ref: [list.h](https://github.com/zhiyb/Nosumor/blob/a929758957174b0c27c822432a4af08ad1017f9c/device/common/list.h)

* * *

Another thing to note is static libraries. The GNU linker by default only searches for required symbols in static library archives. If a .o file is never referenced anywhere else, they will be ignored. Which means, static global allocations and initialisation (the link-time array) will not be included in the final executable.

This behaviour can be turned off by surrounding the archive files using `--whole-archive` and `--no-whole-archive` flags. Then linker will include every .o files inside the archive, just like the .o files specified in the command line.

Ref: [GNU linker command line options](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_node/ld_3.html)

* * *

For Qt's new qbs build system (version 1.9+), this flag can be specified as:

In `Depends`:

\[code\]Depends {name: "core"; cpp.linkWholeArchive: true}\[/code\]

Ref: [qbs cpp](https://doc.qt.io/qbs/qml-qbsmodules-cpp.html)

As exported default dependency parameter in `StaticLibrary`:

\[code\] Export { Parameters {cpp.linkWholeArchive: true} } \[/code\]

Ref: [qbs Parameters](https://doc.qt.io/qbs/qml-qbslanguageitems-parameters.html)