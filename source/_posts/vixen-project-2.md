---
title: Vixen project
tags: []
id: '56'
categories:
  - - c
    - Vixen
date: 2017-01-23 14:00:37
---

Performance profiling using Visual Studio performance sessions (sampling profiler). Instrumentation logging added to record performance data.

Using `Makefile` and varies Linux tools to convert performance texts to CSV tables. Import CSV into Excel, plotting line diagrams.

Full single core CPU usage (50% total) during first 50 seconds with low (5~10) fps preview & controller refresh rate. Afterwards, CPU usage varies (from 20% to 100% total), achieving target refresh rate (20 fps controllers, 20 fps preview), but frame rates are not stable.

Ref: [Makefile](https://github.com/zhiyb/EE9-APRJ/blob/master/misc/Makefile_csv) Ref: [5002c640a54ab1b3fbc8736c125aba7a14b46d1a](https://github.com/zhiyb/vixen/commit/5002c640a54ab1b3fbc8736c125aba7a14b46d1a)