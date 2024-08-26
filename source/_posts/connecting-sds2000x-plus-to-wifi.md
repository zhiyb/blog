---
title: Connecting SDS2000X Plus to WiFi
tags:
  - Siglent
  - WiFi
id: '1379'
categories:
  - - c-c
    - Embedded
  - - Linux
date: 2021-06-26 14:24:31
---

I've managed to connect my Siglent SDS2104X Plus oscilloscope to my WiFi network using a USB WiFi dongle.
<!-- more -->
I've managed to get code execution and telnet access to my SDS2104X+ without opening it up, and had a look around at its file systems. (Probably will be blogged later)

Anyway, the main thing here is, as expected, it uses Zynq FPGA + Linux, and does have a few WiFi driver modules already lying around. So I'd like to connect it to my WiFi network, to avoid having to route and connect the ethernet cable every time I move it around.

```
# ls -1 /usr/bin/siglent/drivers/
g_usbtmc.ko
insmod_after_app.sh
insmod_before_app.sh
libcomposite.ko
mt7601u.ko
siglent_cpld_irq.ko
siglent_dma.ko
siglent_vdma.ko
siglent_vnc.ko
siglentkb.ko
udc-xilinx.ko
uinput.ko
xilinx_axicdma.ko
xilinx_axidma.ko
xilinx_vdma.ko
```

```
# ls -R -1 /lib/firmware/
/lib/firmware/:
mt7601u.bin
rt2870.bin
rtlwifi

/lib/firmware/rtlwifi:
rtl8188eufw.bin
```

That mt7601u is clearly a WiFi driver. I can also find r8188eu in `/sys/module`, and these firmware binary blobs.

So I got a cheap little mt7601u WiFi dongle from Amazon.  
Plugging it in, and sure it's detected alright, as `wlan0`:

```
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 74:5b:c5:20:3b:a1 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq qlen 1000
    link/ether 1c:bf:ce:4e:bd:e3 brd ff:ff:ff:ff:ff:ff
```

The `iw` wireless management tool isn't here, so that's a bit annoying.  
But at least `wpa_supplicant` does exist, that's good enough to connect to some WiFi networks.

First, write a `wpa_supplicant` configuration file, e.g. `wpa.conf`:

ctrl\_interface=/tmp/wpa\_supplicant
update\_config=1

network={
ssid="MyWiFi"
psk="Passw0rd"
}

Power on, start WiFi, and DHCP client:

```
echo on > /sys/class/net/wlan0/power/control
wpa_supplicant -B -i wlan0 -c wpa.conf
udhcpc -i wlan0
```

...and, Ethernet connection is dropped.  
Apparently, the oscilloscope main application (`/usr/bin/siglent/main.app`) checks network interfaces periodically. It is probably not expecting to see a second `wlan0` interface up, ends up resetting assigned IP addresses on all network interfaces.

So, either kill `main.app`, or disconnect ethernet once WiFi is connected.

* * *

To make this process automatic after reboot, prepare a USB drive, and in its top level directory, create a shell script named `siglent_device_startup.sh`:

#!/bin/sh
root=\`dirname $0\`
if ip link show wlan0; then
echo on > /sys/class/net/wlan0/power/control
wpa\_supplicant -B -i wlan0 -c $root/wpa.conf
udhcpc -i wlan0
fi

And, of course put the `wpa.conf` file at the same place too.

Plug in the USB drive and WiFi dongle on its two front USB ports, power it on, it should automatically run the code and connect to the WiFi network.