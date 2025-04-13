---
title: Vixen project
tags: []
id: '221'
categories:
  - - c
    - Vixen
date: 2017-05-01 03:18:33
---

Supervisor was not in the college today, possibly because of bank holiday.

I have implemented a simple console application (`VixenLinky`) that reads and transmits channels of the dumped data through TCP, using C#. Ref: [172359860efc8cc093e252b73ec0ed3a408a285f](https://github.com/zhiyb/vixen/commit/172359860efc8cc093e252b73ec0ed3a408a285f)
<!-- more -->
Currently I haven't implemented the module/DLL loader, so it does not uses the configuration files and cannot select between different controllers yet. Data read from a file reader thread transfers directly to a single fixed TCP controller thread without intermediate conversions. The thread timer intervals and interested range of channels can be selected at runtime as parameters, as the basis of a modular design. It may be used as a reference of ideal performace figures of C#.

This simple application works surprisingly well. With all 8600 channels and a single controller, it uses less than 2% of CPU time on 1 out of all 4 cores on the Raspberry Pi for 50 fps playback. With an interval of 1 ms, the Raspberry Pi can output the sequence with 1000 fps, using only 50% on 2 cores. With an interval value of 0 ms, the controller updates at ~2900 fps, and the entire sequence can finish within 3 seconds, suggesting the file IO is capable of ~6500 fps.

If you remember the handheld device I brought in last time, I installed mono on it as well. It has a 336MHz MIPS CPU. Although it needs a few seconds to start the application, but it can output the sequence at a steady 50 fps, uses mostly 50% and maximum of ~80% CPU time. The performance may be further improved with better kernel drivers.

I will be implementing the controller module interface and support of configuration files. The performances may be different with different controllers, but probably not much.

Having several exams in 2 weeks, so next project meeting will be @15/May/2017.