---
title: Discrepancy between bash v2 and v4
tags:
  - bash
  - Noah
id: '1222'
categories:
  - - Linux
date: 2017-12-31 19:19:53
---

I was porting a few programs, originally written for [Noah](http://opennoah.github.io/) (`bash` v2) a few years ago, to modern Linux (`bash` v4): [【插件】终端单词默写软件](https://github.com/OpenNoah/Words) [【插件】文件管理器（终端资源管理器）](https://github.com/OpenNoah/File-Manager) [【游戏】推箱子游戏，bash版](https://github.com/OpenNoah/Sokoban)

The main discrepancy being the assignments of array variables. In `bash` version 2, I wrote: `declare array=("'a' 'b' 'c' 'd'")` `array=($1)` Which, for `bash` version 4, need to be updated as: `declare array=('a' 'b' 'c' 'd')` `eval array=($1)`