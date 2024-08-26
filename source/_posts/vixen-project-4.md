---
title: Vixen project
tags: []
id: '97'
categories:
  - - c
    - Vixen
date: 2017-02-06 14:00:37
---

Implemented `TCPLinky` custom output controller module, based on `BlinkyLinky`. At least there is an actual controller that can be offloaded onto a different machine to simulate real world scenarios.

Display interface was implemented using `Qt` with `OpenGL`, very fast with low CPU usage (<2% @50fps). Controller communicates using a single `TCP` connection.

It can dump data from output channels to a file and directly playback from the dump later. Only minimal post processing involved in playback, so the rendering speed is very fast and possible on an embedded platform. (Pre-render the sequence from a high performance computer?)

The sequence can also be exported from the sequence editor.

Ref: [17fc24ce32e9d803016f7573b225710089a80041](https://github.com/zhiyb/vixen/commit/17fc24ce32e9d803016f7573b225710089a80041) Ref: [c82e16d0acf51043ae33f8afea209061bb6042e6](https://github.com/zhiyb/EE9-APRJ/commit/c82e16d0acf51043ae33f8afea209061bb6042e6)