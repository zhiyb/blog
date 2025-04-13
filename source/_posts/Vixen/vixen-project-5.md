---
title: Vixen project
tags: []
id: '135'
categories:
  - - c
    - Vixen
date: 2017-02-13 14:00:33
---

Using the export function from sequence editor, all output channels from all controllers can be exported with specific time interval.

The timebase used for exportation was a manual timebase, so the exported output interval is kept constant and unrelated to export time.

The export pipeline only supports 8-bit commands, might be a potential problem?

Another export format was implemented, to match the playback format used in `TCPLinky` controller. Ref: [830a17bb445aab1c98ddd44540cc77a0b701177b](https://github.com/zhiyb/vixen/commit/830a17bb445aab1c98ddd44540cc77a0b701177b#diff-7aed62d024ea2a1ba393f50293e9967b)

The `resolutionComboBox` in the original application can only specify a time interval of 25, 50 or 100 ms. It was modified to allow custom values. Ref: [e882fd9d429b9c00a0fd549e46805058c82849b2](https://github.com/zhiyb/vixen/commit/e882fd9d429b9c00a0fd549e46805058c82849b2)