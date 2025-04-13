---
title: Vixen project
tags: []
id: '358'
categories:
  - - C#
  - - c
    - Vixen
date: 2017-06-12 22:49:55
---

This week(?) is all about performance optimisations, not much to demonstrate, therefore the project meeting was cancelled.

Now, the application can run on Raspberry Pi 1 with the TCP controller at full speed (50 fps) most of the time, with a CPU usage of 10% to 100% (mostly ~40%). That performance now is actually also depends on the efficiency of controller implementations quite a lot, relative to the optimised playback engine.
<!-- more -->
[`mono` sampling profiler](http://www.mono-project.com/docs/debug+profile/profile/profiler/) was used to analysis the execution performance, but it was not very helpful. It states `__Read` and `pthread_cond_timedwait` are the most time consuming functions captured during module loading, that is probably due to async reading task waiting functions. During playback, it stats `pthread` waiting functions are the most time consuming functions, which is probably not true?

The most performance critical part turns out to be type derivation of `ICommand` from/to `_8BitCommand`. Tested by commenting out part of the code then compare the performance difference. This was improved by using compile time static type castings that is more efficient, but may cause strange runtime errors if types are not matched. Ref: [Static cast in C#](https://stackoverflow.com/questions/3894378/how-to-do-a-static-cast-in-c)

The usage of `BinaryReader` was also improved to have as less system calls as possible, by reading larger chunks of data each time. However, this is probably not very effective, since the performance of `VixenLinky` already shows that this is not a problem. Ref: [Faster (unsafe) BinaryReader](https://stackoverflow.com/questions/1238388/faster-unsafe-binaryreader-in-net)

`Dictionary` lookup of controllers was changed from `string` type controller name lookup to `Guid` based lookup, probably can give some minor performance improvements.

Ref: [6ce41cc0723c10f41f1de9372511b836464cf31f](https://github.com/zhiyb/vixen/commit/6ce41cc0723c10f41f1de9372511b836464cf31f)

The TCPLinky controller was also improved by using pre-allocated byte array instead of `Dictionary` lookups for the records of last channel value, and simplified command checking. Ref: [c39e3a62bbb9cafb7737b3e7dcb37c2f157e622d](https://github.com/zhiyb/vixen/commit/c39e3a62bbb9cafb7737b3e7dcb37c2f157e622d)

I will be implementing performance metrics and profiling this week, perhaps also audio playback that is currently missing. Hopefully I should have some data figures to talk about next week.