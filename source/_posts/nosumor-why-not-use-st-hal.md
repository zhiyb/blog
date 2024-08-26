---
title: Nosumor
tags:
  - C/C++
  - Embedded
  - MicroSD
  - Nosumor
  - STM32
  - USB
id: '1269'
categories:
  - - uncategorized
date: 2018-02-12 03:01:48
---

The hardware controller code of this Nosumor project was implemented from register level, using only CMSIS library. But why not use the official ST HAL driver library?
<!-- more -->
* * *

1\. **Code footprint**

Actually, the initial attempt at developing custom code for [the original hardware](http://noodlefighter.com/osu_keyboard/v3_show_en.html) (STM32F103C8T6) does use the ST32CubeMX program and the ST driver library. To preserve the original firmware, the new code under development was running in the 20kiB internal SRAM.

In fact, I am able to implement [a working USB HID mouse](https://github.com/zhiyb/Nosumor/tree/d5f4aff74fe20872e88527b30894e7ddb7e7b8a8) in under a single day. However, the code(.text) section of this implementation consumes about 11564 bytes under a release build (-Os). To fit a debug build into the 20KiB SRAM, I had to enable optimisation, which makes debugging difficult since variables might be optimised out. There was also not much space left for future development. Therefore, I decided to implement the USB stack from scratch.

\[code\]section size addr .isr\_vector 268 0x20000000 .text 11564 0x2000010c .rodata 176 0x20002e38 .init\_array 8 0x20002ee8 .fini\_array 4 0x20002ef0 .data 1344 0x20002ef8 .bss 2220 0x20003438\[/code\]

After 7 days of development, I finally made [the USB HID mouse working again](https://github.com/zhiyb/Nosumor/tree/222eb10087d3114fc2ec7f8375b2b1eb8c02d1db). This time, the code(.text) section of a release build consumes 3556 bytes, only 30.8% of the previous implementation. A debug build without optimisation consumes 6624 bytes, leaving plenty of space for future development. In addition, [DMA was used for USB data transfer](https://github.com/zhiyb/Nosumor/blob/222eb10087d3114fc2ec7f8375b2b1eb8c02d1db/src/usb/usb.c#L199), which is [not available from the ST HAL driver](https://github.com/zhiyb/embedded-drivers/blob/258f57e626d37432be921a9c5e6b4640bd5d580f/STM32F1xx_HAL_Driver/Src/stm32f1xx_ll_usb.c#L2105).

\[code\]section size addr .isr\_vector 268 0x20000000 .text 3556 0x2000010c .rodata 272 0x20000ef0 .init\_array 8 0x20001000 .fini\_array 4 0x20001008 .data 1072 0x20001010 .bss 80 0x20001440\[/code\]

The newer design (remake) uses STM32F722RET6, flash/SRAM space is no longer an issue. However, [at this time](https://github.com/zhiyb/Nosumor/tree/651f3dece87164485a3cd5caf1b496868232568a), the size of a debug build of the bootloader is approaching 64kiB, the size of the specified flash region. Using another layer of ST HAL might exceed the size.

* * *

2\. **Learning purpose**

Implement every thing from scratch does provide lots of learning opportunities. The primary reason of doing so is for me to learn the USB stack, the structure and limitations of this widely used protocol. In the process, I have also encountered all kinds of bugs. More importantly, how to effectively debug and fix those issues.

For simple modules, sometimes controlling the registers directly might be easier than using HAL functions.

* * *

3\. **Performance and latency**

Remove layers of overheads can improve the overall system performance. Implement from a lower layer also provides finer and more flexible controls over various operations. For example, skipping unnecessary checks, removing unused operation modes and optimising overall state management.

Some specific examples including never used USB host mode, I2C slave mode and timer capture modes.

For the original design utilising STM32F103, in addition to [the aforementioned DMA operation](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_ll_usb.c#L633), the program can also [allocate USB data buffer directly from USB RAM](https://github.com/zhiyb/Nosumor/blob/68f6aa0f110fd3c2bccccec3e3a57f499d70ab5e/device/keyboard.c#L30) with the help of [a linker script](https://github.com/zhiyb/Nosumor/blob/68f6aa0f110fd3c2bccccec3e3a57f499d70ab5e/device/STM32F103C8_SRAM.ld#L33). In this way, data updates can be written directly to USB RAM without copying, improves both performance and latency, which is not possible with the ST HAL driver.

The current implementation of the SD card reader with USB Mass Storage Class (MSC) also has a better performance than a ST library based one. The best sequential performance of a SD card only occurs when using continuous multi-block data transfer. (Block size is generally 512 bytes.) In this mode, the SD card can interleave block erase and programming with data transfer for write operations, also saves time on command transfers between blocks.

Although the ST HAL driver supports multi-block data transfer with DMA, it does not expose the SDIO data path state machine (DPSM), and will [automatically send a `STOP TRANSFER` command after each data transfer](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_hal_mmc.c#L1403). To read certain bytes of data from a SD card, the buffer must have a large enough size. To write certain bytes of data to a SD card, all data must be available in the buffer beforehand.

However, the USB MSC uses bulk-only transport protocol, and endpoint size is limited to 512 bytes for USB high-speed. To utilise multi-block data transfer with ST HAL driver, a large buffer is required. This will also affect the responsiveness of the progress dialog on a Windows computer, since SD card transfer delays only happen after each buffered transfer.

With [current implementation](https://zhiyb.wordpress.com/2018/01/10/nosumor-usb-mass-storage-class/), a large buffer of 64kiB was still used for read operations. But, [the DMA progress was monitored](https://github.com/zhiyb/Nosumor/blob/651f3dece87164485a3cd5caf1b496868232568a/device/peripheral/mmc.c#L395), so that the USB MSC is able to start and interleave USB transfers of 512 bytes during the process. For write operations, without the automatic transmission of `STOP TRANSFER` commands, [a buffer of only 512 bytes is needed for any sizes of multi-block operations](https://github.com/zhiyb/Nosumor/blob/651f3dece87164485a3cd5caf1b496868232568a/device/peripheral/mmc.c#L343). Double buffering was also implemented from the USB MSC side, so USB and SDIO data transfers can be interleaved as well. Double buffering instead of the large buffer for read operations is also possible without any performance decrease.

The sequential performance of current implementation with a class 10 SDHC card is: 18MiB/s read & 6MiB/s write, similar to if not faster than using my laptop's built-in commercial SD card reader. Thanks to the non-blocking interrupt-based design, all other activities that might present at the same time, including high definition audio streaming, do not affect the performance at all. Here is [a thread of people achieving 10MB/s read & 5MB/s write with ST libraries (but not with USB MSC)](https://community.st.com/thread/36673-very-slow-sdmmc-read-performance-on-stm32f746ngh6).

Latency is especially important and noticeable in audio streaming. Therefore, [the special endpoint-specific interrupts for endpoint 1](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/CMSIS/Device/ST/STM32F7xx/Source/Templates/gcc/startup_stm32f722xx.s#L235) was used for USB audio class (UAC2) with the highest interrupt priority. Looking through the low-level USB implementation in ST HAL driver, [it does not seems to have an endpoint specific interrupt handler](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_hal_pcd.c#L319), which kind of defeat the purpose of having endpoint-specific interrupts.

* * *

4\. **Cortex-M7 caching**

For better processor performance, Cortex-M7 instruction and data caching was enabled. However, since Cortex-M7 does not have a cache coherency protocol between DMAs, cache lines need to be flushed and invalidated manually. DMA transfers also require [proper data alignment](https://github.com/zhiyb/Nosumor/blob/306e0d635d14406e709eed82b03899bf486c19f4/device/usb/usb_ep0.c#L28). Specifying [data alignment](https://github.com/zhiyb/Nosumor/blob/1867cce7d57bfbf09b7257aab506ed013ccfe7ee/device/logic/scsi_buf.c#L10), [section placement](https://github.com/zhiyb/Nosumor/blob/3e2310e0ee2c246e5b1cfd5b6afe0b7838dbb4b0/device/peripheral/audio.c#L31) and [cache operations](https://github.com/zhiyb/Nosumor/blob/cd7534c90d8665ae4efa94c11d6a867ed0b4b645/device/peripheral/mmc.c#L686) directly inside low-level layer can be beneficial in this case.

* * *

5\. **Logic structure**

For the newer design, linked lists were used extensively with interrupts to simplify interfacing and improve performance, similar to RTOS queues.

For example, the I2C module uses a linked list to manage non-blocking read/write operations. I2C transfers can therefore be queued to start automatically one after another, without impacting the performance of other modules because of busy wait. The ST HAL driver does support [non-blocking I2C transfers](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_hal_i2c.c#L1152), but these functions are quite primitive, another layer is required for queue management anyway.

The USB protocol stack uses linked lists to manage registration of multiple interfaces, and automatically update descriptors and assign endpoints accordingly. In this way, the program can easily configure available USB interfaces based on [hardware availability](https://github.com/zhiyb/Nosumor/blob/651f3dece87164485a3cd5caf1b496868232568a/device/main.c#L133) or [user configuration](https://github.com/zhiyb/Nosumor/blob/651f3dece87164485a3cd5caf1b496868232568a/device/main.c#L167). This is actually also possible using ST libraries, but requires adding another layer of root USB class.

* * *

6\. **Issues with individual modules**

`USB_OTG_HS`:

`OTG_GUSBCFG` register, bit 4 selects whether to use UTMI (internal PHY) or ULPI (external PHY) interface. Although internal PHY through UTMI is not available for STM32F722, the register defaults to use UTMI. The ST HAL driver [does not correct this behaviour](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_ll_usb.c#L134), and will not select ULPI for STM32F723 even if specified.

`OTG_DCFG` register, bit 14 selects enable or disable delay in ULPI timing during device chirp. It is [necessary for the USB3370 PHY](https://github.com/zhiyb/Nosumor/blob/cd7534c90d8665ae4efa94c11d6a867ed0b4b645/device/usb/usb.c#L144) to emulate the device as high-speed instead of full-speed, but it defaults to disable. Surprisingly, this bit is not even defined in CMSIS.

There are also [some undocumented features](https://zhiyb.wordpress.com/2017/09/13/nosumor-remake-v0-3/).

`UART/USART`:

The baud rate register calculation is [very inefficient for older devices](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_usart.h#L562) (up to STM32F4 series). It was [improved for STM32F7 series](https://github.com/zhiyb/embedded-drivers/blob/324fcd443868965a960141ec218689d148467145/STM32F7xx_HAL_Driver/Src/stm32f7xx_hal_usart.c#L1929). And here is [my implementation](https://github.com/zhiyb/Nosumor/blob/cd7534c90d8665ae4efa94c11d6a867ed0b4b645/device/peripheral/uart.c#L29).

* * *

Despite the problems with ST libraries, they are very helpful as a known working reference design when debugging. They are also very helpful for adapting a design to different generations of STM32 microcontrollers. For simple designs, using ST libraries can be very effective for time saving purposes and reducing program bugs.