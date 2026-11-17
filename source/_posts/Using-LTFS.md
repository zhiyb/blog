---
title: Using LTFS
date: 2026-11-17 23:08:38
tags:
    - Linux
    - TrueNAS
    - LTO
    - LTFS
---

My notes on working with LTO-6 tapes using LTFS.

<!--more-->

# IBM LTFS vs. Aurora LTFS

According to the [Aurora project](https://github.com/AuroraTape):

_In January 2026, the LinearTapeFileSystem project removed all Owners and Members except two IBM employees. There was no prior notice, no warning, and no explanation._

_The project was originally established by IBM and had been operated as a genuine open-source project with reasonably transparent decision-making. That ended overnight._

The good open source replacement appears to be this AuroraTape project.

# Using LTFS in TrueNAS

TrueNAS is quite restrictive on what can be installed directly on the host system, and it does not include the software necessary for using LTFS.

Instead, we can mount LTFS using the AuroraTape project docker container.

## Find tape drive device

To access the tape drive inside the container, we need to find the `/dev/sg*` device file pointing to the tape drive.

I found it by trying to read device attributes:

```bash
$ sudo sg_read_attr /dev/sg1
Read attribute command not supported
sg_read_attr failed: Illegal request, Invalid opcode

$ sudo sg_read_attr /dev/sg2
Attribute values:
  Remaining capacity in partition [MiB]: 36750
  Maximum capacity in partition [MiB]: 36750
  ...
```

## Start a shell inside the container

As guided by the AuroraTape project README:

Pass the device file to the container via `--device`.

Pass the fuse device file and relevant security options so we can mount LTFS.

```bash
sudo docker run --rm -it --device /dev/fuse --device /dev/sg2 --cap-add SYS_ADMIN --security-opt apparmor=unconfined ghcr.io/auroratape/aurora-ltfs:1.5 bash
```

Optionally bind mount a directory from the host so we can transfer files between the host and the container, e.g.:

```bash
--mount type=bind,src=/mnt/tank/nas/tmp,dst=/mnt/tmp
```

### Check device exists

List all available tape drives:

```bash
altfs -o device_list
```

### Mount LTFS inside container

```bash
mkdir /mnt/altfs
altfs -o devname=HUJ7369758 /mnt/altfs
```
