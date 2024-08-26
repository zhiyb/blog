---
title: Change rootfs from ext4 to zfs
tags:
  - zfs
id: '1411'
categories:
  - - Linux
date: 2022-11-28 21:31:01
---

My notes on changing my debian Linux installation rootfs to a zfs pool.
<!-- more -->
zfs can only be applied to the rootfs, so EFI should be on a separate partition.  
Also, swap does not need to be managed by zfs. There might be bugs for swap on zfs too.

It is best to move /boot to a separate basic partition, e.g. ext4.  
grub can boot from a zfs pool, but it only supports a very limited number of zfs features, basically single-disk vdev.  
Does not support mirroring, definitely not raidz.  
So it does not make sense to use zfs for this.

So, partition the disk with separate EFI, boot and swap, e.g.:

```
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1953791   953.0 MiB   EF00  EFI system partition
   2         1953792        18731007   8.0 GiB     8200  Linux swap
   3        18731008      1951427982   921.6 GiB   8300  Linux filesystem
   4      1951428608      1952477183   512.0 MiB   8300  Linux filesystem
```

Create zfs pool on rootfs partition 3:

zpool create os ca7c5962-52e0-4dd3-9514-4cdf3589d688
zfs set compression=on os
zfs create os/debian
zfs inherit compression os/debian

Now copy rootfs to os/debian, e.g.:

rsync -auhxAXH --info=progress2 /mnt/rootfs/ /os/debian/

Populate the boot partition:

cd /os/debian
mv boot boot\_bak
mkfs.ext4 /dev/nvme0n1p4
mount /dev/nvme0n1p4 boot
cp -a boot\_bak/\* boot/

Update grub boot configurations:

cd /os/debian
mount --bind /dev dev
mount --bind /proc proc
mount --bind /sys sys
mount --bind /run run
mount /dev/nvme0n1p1 boot/efi
chroot .
update-grub
grub-install /dev/nvme0n1

Update rootfs and swap:

\# No rootfs entry needed in fstab, remove it
# Also add the new /boot partition
$ vim /etc/fstab
# Update reference to swap partition
# https://unix.stackexchange.com/a/557628
$ vim /etc/initramfs-tools/conf.d/resume
$ update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.0.0-4-amd64
cryptsetup: ERROR: Couldn't resolve device os/debian
cryptsetup: WARNING: Couldn't determine root device
$
# (How to solve the cryptsetup warnings?)

Preparation of rootfs done, clean up manual mounts:

umount boot/eft
umount dev proc sys run

Set zfs boot flags:

zfs set mountpoint=none os
zfs set canmount=off os
zfs set mountpoint=none os/debian
zpool set bootfs=os/debian os

Now boot!

$ neofetch
       \_,met$$$$$gg.          zhiyb@zhiyb-Linux
    ,g$$$$$$$$$$$$$$$P.       -----------------
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux bookworm/sid x86\_64
 ,$$P'              \`$$$.     Kernel: 6.0.0-4-amd64
',$$P       ,ggs.     \`$$b:   Uptime: 1 min
\`d$$'     ,$P"'   .    $$$    Packages: 3383 (dpkg)
 $$P      d$'     ,    $$P    Shell: bash 5.2.2
 $$:      $$.   -    ,d$$'    Resolution: 1920x1080
 $$;      Y$b.\_   \_,d$P'      Terminal: /dev/pts/3
 Y$$.    \`.\`"Y$$$$P"'         CPU: Intel i9-9900K (16) @ 5.000GHz
 \`$$b      "-.\_\_              GPU: Intel CoffeeLake-S GT2 \[UHD Graphics 630\]
  \`Y$$                        Memory: 4083MiB / 31640MiB
     \`$$b.
       \`Y$$b.
          \`"Y$b.\_
              \`"""

$ sudo zpool status
  pool: os
 state: ONLINE
config:

        NAME                                    STATE     READ WRITE CKSUM
        os                                      ONLINE       0     0     0
          ca7c5962-52e0-4dd3-9514-4cdf3589d688  ONLINE       0     0     0

errors: No known data errors

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             16G     0   16G   0% /dev
tmpfs           3.1G  3.1M  3.1G   1% /run
os/debian       892G  195G  698G  22% /
tmpfs            16G  4.0K   16G   1% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
tmpfs           3.1G   56K  3.1G   1% /run/user/1000

## Miscellaneous issues

### Docker on ZFS

overlayfs on zfs does not work:

```
[  379.923528] overlayfs: upper fs does not support RENAME_WHITEOUT.
[  379.923622] overlayfs: upper fs missing required features.
```

Change docker storage driver to zfs, see:  
https://docs.docker.com/storage/storagedriver/zfs-driver/

## References

https://blog.heckel.io/2016/12/31/move-existing-linux-install-zfs-root/  
https://www.funtoo.org/ZFS\_as\_Root\_Filesystem\\