---
title: Vixen project
tags:
  - DLL
  - ffmpeg
  - Video
id: '859'
categories:
  - - C#
  - - C/C++
  - - c
    - Vixen
date: 2017-07-17 01:34:58
---

Last week's project meeting was cancelled.

Placed video encoding and decoding functions in a separate C/C++ library. By using C# `interop` and `DllImport`, the library can be used inside C#.
<!-- more -->
A command line tool of video transcoding from/to sequence was implemented using C#. The lossless correctness was checked by encoding to a video file then decode back to a sequence file, checked against the original sequence using the `diff` tool. This process was automated by using a `Makefile`.

Ref: [7e3b4ade3a0aa04f1a4c3f53a13d4020fac22534](https://github.com/zhiyb/EE9-APRJ/commit/7e3b4ade3a0aa04f1a4c3f53a13d4020fac22534)