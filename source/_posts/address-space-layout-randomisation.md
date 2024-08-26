---
title: Address space layout randomisation
tags: []
id: '1296'
categories:
  - - uncategorized
date: 2018-01-17 09:37:42
---

https://en.wikipedia.org/wiki/Address\_space\_layout\_randomization
<!-- more -->
## On Linux:

\[code\]sudo sysctl kernel.randomize\_va\_space=1\[/code\]

https://askubuntu.com/a/318476

## For MinGW:

\[code\]-Wl,--nxcompat,--dynamicbase -Wl,--pic-executable -Wl,-e,mainCRTStartup\[/code\]

https://patches.libav.org/patch/54505/