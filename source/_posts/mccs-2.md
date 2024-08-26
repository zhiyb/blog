---
title: MCCS
tags:
  - Server
id: '540'
categories:
  - - C/C++
  - - games
    - Minecraft
date: 2017-06-16 18:53:05
---

Implemented server tick, keep alive and watchdog, player object, basic text chat object, dynamic chunk loading and unloading.
<!-- more -->
Server tick timer thread for accurate tick timing and no missing ticks, server worker thread for actual game tick. Players update sequentially after game tick event.

A `std::deque` was used for packet data communication between update thread and handler thread. `std::mutex` was used to protect the queue, however, it is unable to remove a packet from the queue and still fails sometime under heavy communication load.

Ref: [770ff988d78f2e8aa999a4b6dab7fa30b38bcb1e](https://github.com/zhiyb/MCCS/commit/770ff988d78f2e8aa999a4b6dab7fa30b38bcb1e)