---
title: MCCS
tags:
  - CSV
  - JSON
  - Server
  - socket
id: '492'
categories:
  - - C/C++
  - - games
    - Minecraft
date: 2017-06-15 13:24:07
---

Minecraft custom server implemented in C++, work in progress.

Currently implemented status checking, basic login with encryption and trivial chunk data upload. Socket listening on both IPv4 and IPv6.
<!-- more -->
Protocol versioning was done by loading ID mapping CSV files during runtime. The protocol version updates every time when a new Minecraft snapshot/release was available, even if the actual protocol is the same and compatible. Therefore, fuzzy version lookup was implemented to enable backward compatibility. It will search for lower available version definitions when the exact protocol version was not found. Protocol information from: [http://wiki.vg/Protocol](http://wiki.vg/Protocol)

An IPv6 socket can also listen on IPv4-mapped IPv6 address, by disabling `IPV6_V6ONLY` (default: false). Currently, every established connection will have a separate handling thread for performance.

Keep alive watchdogs were not implemented at this point. But the event library [libev](http://software.schmorp.de/pkg/libev.html) was already deployed to give IO/timer/event based notifications to individual threads.

JSON parser: [RapidJSON](https://github.com/miloyip/rapidjson) CSV parser: [Fast C++ CSV Parser](https://github.com/ben-strasser/fast-cpp-csv-parser) Logging: [spdlog](https://github.com/gabime/spdlog)

Ref: [85ec5d2c31c58903e338cb406430f2e587c1d7c3](https://github.com/zhiyb/MCCS/commit/85ec5d2c31c58903e338cb406430f2e587c1d7c3)