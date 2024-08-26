---
title: Nintendo Switch 破解入门
tags: []
id: '1325'
categories:
  - - games
    - Nintendo Switch
date: 2019-11-30 17:52:10
---

This post is in Chinese. I may also write an English version soon™.

以下是我破解 Nintendo Switch，尝试 CFW，emuMMC，Homebrew 的步骤。
<!-- more -->
* * *

# 注意：

本篇文章不涉及盗版游戏。抵制盗版，支持正版。  
Switch 破解方法很多，本篇文章只是我个人的记录，使用 Windows 10 主机。  
仅支持早期版本未修复 RCM bug 的 Switch。已经修复的硬件版本会在进入 RCM 模式步骤失败。可以在这里检查你的 Switch 序列号：  
https://ismyswitchpatched.com  
使用破解系统联网会导致 Nintendo 账号以及 Switch 主机被封禁。  
某些简单步骤和细节在文章中可能会被省略，不建议电脑操作萌新尝试。  
作者不对任何因为本篇文章导致的损失负责。

* * *

# 需要的软件：

TegraRcmGUI  
https://github.com/eliboa/TegraRcmGUI/releases

hekate  
https://github.com/CTCaer/hekate/releases

Lockpick\_RCM  
https://github.com/shchmue/Lockpick\_RCM/releases

Atmosphère  
https://github.com/Atmosphere-NX/Atmosphere/releases

* * *

# 硬件准备：

至少 64GB 的 MicroSD 卡。注意备份数据。  
RCM Jig（推荐），或者任何其它能帮助进入 RCM 模式的硬件。  
一根双公头的杜邦线，一边连接 pin 10，另一边连接散热片都可以。

参考：  
https://nh-server.github.io/switch-guide/user\_guide/emummc/entering\_rcm  
https://gbatemp.net/threads/need-help-getting-into-rcm-mode.503623  
https://gbatemp.net/threads/show-us-your-jig.502364/page-3

* * *

# SD 卡准备：

## 使用 Windows 磁盘管理重新分区 SD 卡

1\. 删除所有 SD 卡上已有的分区  
2\. 创建新分区，大小是总可用大小减去 29850 MB (\*)。格式化为 FAT32 或 exFAT，备用  
3\. 创建新分区，使用剩下的全部空闲空间。不要格式化

\* 数值不准确。有的帖子建议 29828 MB，但是之后创建 emuMMC 的时候会出错。这里建议剩余空间稍微大一些。  
https://gbatemp.net/threads/setting-up-partitions-for-emummc-with-hekate5-0-nyx.542321/  
\*\* 因为 MBR 分区表限制，最多支持3个 SD 卡分区模式的 emuMMC 配置。这里仅使用了一个。文件挂载模式也支持，不过性能会差一些。

## 准备 RCM Payload + CFW

\* 以下操作针对 SD 卡第一个 FAT32/exFAT 分区。  
1\. 将 hekate，Atmosphère 解压至 SD 卡根目录  
2\. 复制 `Lockpick_RCM.bin` 到 `SD 卡/bootloader/payloads`  
3\. 在 `SD 卡/bootloader` 创建文件 `hekate_ipl.ini`（设定启动项），复制粘贴以下内容：

\[config\]
autoboot=0
autoboot\_list=0
bootwait=3
verification=1
backlight=100
autohosoff=0
autonogc=1

\[Stock\]
fss0=atmosphere/fusee-secondary.bin
stock=1
emummc\_force\_disable=1
# Both above disable kernel patching

\[Atmo emuMMC\]
fss0=atmosphere/fusee-secondary.bin
# hekate does not support stock emuMMC yet
#stock=1

参考：  
https://github.com/CTCaer/hekate/blob/master/res/hekate\_ipl\_template.ini

* * *

# 准备主系统

emuMMC 会以 Switch 主系统为参照。所以需要先在主系统里做一些准备工作，以避免启动 emuMMC 联网时被封禁。  
1\. 正常启动 Switch  
2\. 禁用家长控制  
3\. 删除所有 WiFi 网络记录，避免自动连接网络  
或者可以在网络上设定 90DNS 之类的工具来避免连接 Nintendo 服务：  
https://gbatemp.net/threads/90dns-dns-server-for-blocking-all-nintendo-servers.516234  
3\. （可选）断开 Nintendo 账号关联，或者删除用户数据  
4\. 启动飞行模式  
5\. 正常关机

# 进入 RCM 模式，加载 hekate

1\. 从 hekate 中复制出 `hekate_ctcaer_*.bin`  
2\. 启动 TegraRcmGUI，选择 `hekate_ctcaer_*.bin`  
可以选上 `Settings -> Auto inject` 方便使用。  
3\. 连接 RCM Jig，按下音量+，同时按下电源键启动 Switch  
没有问题的话，会黑屏而不是显示 Nintendo logo 启动系统。  
失败的话，等启动后正常关机，重新尝试进入 RCM 模式。  
4\. 将准备好的 SD 卡装入 Switch  
5\. 连接 USB 线，TegraRcmGUI 会自动发送 hekate，或者手动点击 `Inject payload`  
6\. Switch 上应该会显示 hekate 主界面

* * *

# 备份 eMMC

首先，备份 eMMC：  
1\. 进入 hekate  
2\. 选择 `Tools -> Backup eMMC -> eMMC BOOT0 & BOOT1`，备份 boot 分区  
3\. 选择 `eMMC RAW GPP`，备份数据分区

* * *

# 创建 emuMMC

1\. 返回 hekate `Home`  
2\. 选择 `emuMMC -> Create emuMMC -> SD Partition`  
3\. 确定显示的是 `Found applicable partition: [Part 1]`  
失败的话，说明 SD 卡没准备好，撤出 SD 卡重新准备。  
不需要关闭 Switch。下次装入 SD 卡的时候 hekate 会自动重新加载。  
4\. 点击 `Continue` 开始创建 emuMMC 分区，等待完成

\* 有些地方会写 emuNAND，和 emuMMC 是一样的意思。

* * *

# 破解加密密钥

1\. 返回 hekate `Home`，选择 `Payloads`  
2\. 加载 `Lockpick_RCM.bin`  
3\. 用音量键选择 `Dump from EmuNAND`，按电源键确定。  
4\. 自动重启之后会破解出主机以及游戏的密钥，存在 `SD 卡/switch` 里。  
5\. 按电源键返回 Lockpick\_RCM 主菜单，选择 `Payloads -> hekate_ctcaer_*.bin` 或者 `Reboot (RCM)` 通过 RCM 重新加载 hekate。

* * *

# 恢复 eMMC 备份

一般不需要。不过不放心的话，可以恢复 eMMC 备份来避免主系统被封禁。  
在 hekate 里选择 `Tools -> Restore MMC -> eMMC RAW GPP`。

* * *

# 启动 emuMMC + Atmosphère CFW

1\. 在 hekate 主界面选择 `Launch`  
2\. 选择 `Atmo emuMMC`  
3\. 等待启动  
4\. 保持飞行模式，可以去设置里打开蓝牙和 NFC 功能  
5\. 按住 R 键启动相册会进入有一些功能限制的 CFW Homebrew menu  
6\. 按住 R 键启动任意游戏会进入完整的 CFW Homebrew menu  
...  
8\. Profit?

\* emuMMC 的 Nintendo 文件夹（截图，数字版游戏等）会储存在：  
`SD 卡/emuMMC/RAW1/Nintendo`

* * *

# 识别版本号

设置 -> 系统，查看版本号

应该可以看到版本号里写了 `AMS` 版本，末尾应该有个 `E`，意思是已经启动进了 emuMMC 模式。  
https://nh-server.github.io/switch-guide/user\_guide/emummc/launching\_cfw/

* * *

# 备注

从 emuMMC 系统关机之后，下次正常启动会进入纯净的主系统（sysMMC/sysNAND）。主系统和 emuMMC 是完全分离的，所以主系统仍然可以正常联网，使用 Nintendo 网络服务。

理论上 Nintendo 可以通过检查 SD 卡内容来判定是否有使用过 CFW，所以不建议在主系统中使用同一张 SD 卡。

WiFi 网络记录并不在 SD 卡上储存，所以可以放心复制主系统的 Nintendo 文件夹到 CFW 的对应目录。

经常玩 CFW 的话，买一个 RCM Loader 之类的东西会很方便，比如：  
https://www.xkitcn.com/rcmloader

* * *

参考：  
https://switch.homebrew.guide  
https://switch.hacks.guide  
https://nh-server.github.io/switch-guide  
https://www.scenefolks.com/pages.php?page=4210  
https://wiki.no-intro.org/index.php?title=Nintendo\_Switch\_Dumping\_Guide