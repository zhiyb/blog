---
title: Vixen project final
tags:
  - fmod
id: '1162'
categories:
  - - c
    - Vixen
date: 2017-09-13 22:33:31
---

The report was submitted on 2017-09-08, and the poster presentation was done on 2017-09-11. Ref: [Report submission](https://github.com/zhiyb/EE9-APRJ/releases/tag/v2.0)

Sincere thanks to my project supervisor [Dr James J. Davis](http://www.imperial.ac.uk/people/james.davis06) for supporting this project.
<!-- more -->
Audio playback was implemented using newer version of FMOD with C wrapper functions.

More work need to be done to shape the modifications to match the existing codebase, before changes can be integrated into the official Vixen repo. May also be separated into multiple steps.

`QuickTime` encoder with `MOV` container format and `rgb24` pixel format seems give the best video decoding performance.