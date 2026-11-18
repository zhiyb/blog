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

Use `sg_map` to find the tape drive `/dev/nst0`, e.g.:

```bash
$ sudo sg_map
/dev/sg0
/dev/sg1  /dev/sda
/dev/sg2  /dev/nst0
/dev/sg3  /dev/sdf
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
