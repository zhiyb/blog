---
title: Vixen project
tags: []
id: '739'
categories:
  - - C#
  - - c
    - Vixen
date: 2017-06-29 19:08:03
---

Meeting @19/Jun/2017 was cancelled.

Developed performances profiling on Linux by reading varies statistic files in the `/proc/[PID]` folder. This can be done on Windows as well, by using the `Process` class as in `CpuUsage`.

Options to unlimit playback and/or controller update speeds were implemented, as a baseline for IO/processing performance comparisons across different platforms and architectures.

Ref: [76b683a1aac9c4068ca7052627ad2901f2c87c51](https://github.com/zhiyb/vixen/commit/76b683a1aac9c4068ca7052627ad2901f2c87c51)

Will try to read & modify & write video files, as the input/export for playback engine, to enables the use of well established video processing techniques. `ffmpeg` may be used for media file handling.