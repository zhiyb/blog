---
title: Vixen project
tags: []
id: '239'
categories:
  - - c
    - Vixen
date: 2017-06-06 01:00:00
---

Implemented `VixenConsole` command line interface application for direct playback. Ref: [12f190bfdbe81205838e808cb86fa7f3a8f2039f](https://github.com/zhiyb/vixen/commit/12f190bfdbe81205838e808cb86fa7f3a8f2039f)

It uses slightly modified `Vixen.dll` from the original codebase for module loading and execution pipeline.
<!-- more -->
Borrowed an [original Raspberry Pi](https://www.raspberrypi.org/products/model-b-plus/) as an alternative platform.

It supports loading all controller modules, configurations and module data from original XML configration files. However, loading modules and configurations takes more than 1 minute on the original Raspberry Pi.

A tidy function was also implemented, to remove unused data (e.g. preview configurations) from configuration files. That reduced configuration file from `5.7MiB` to `1.5MiB` and reduced module data file from `12.4MiB` to `25KiB`. However, these does not improve the loading time significantly.

Configuration modifications may also need to be implemented, to suit the actual target platform configurations.

The execution pipeline is also very slow compare to `VixenLinky` on the original raspberry pi, running at only unstable 30 fps with 100% CPU usage. Running on Raspberry Pi 3 uses 50% CPU time of a single core (12.5% total), which is acceptable. It is relatively faster than the original application with playback engine, uses only <1% CPU time on the Windows laptop.

The workflow for the playback engine looks similar to video rendering, perhaps the playback data dump can be changed to a video format, for easy sharing, compression and post editing.

Fair performance comparsion between different platforms and architectures also need to be considered.