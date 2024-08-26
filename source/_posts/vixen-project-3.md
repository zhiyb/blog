---
title: Vixen project
tags: []
id: '86'
categories:
  - - c
    - Vixen
date: 2017-01-30 14:00:47
---

Using hot path analysis from Visual Studio performance profiler to analysis the most time consuming code segment.

Bad performance at the first 50 seconds is still unclear: Trying to catch up with the controller? Waiting for responses from unconnected controllers?

Most of CPU time wasted in output device command generation, specifically C# reflection calling the correct handler. Preview rendering also uses some resources, but not too critical.