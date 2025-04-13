---
title: Using ffmpeg qsv on Intel gen12
date: 2025-04-13 01:50:20
tags:
  - ffmpeg
  - qsv
  - Intel
  - i7-12650H
  - minisforum
---

I had some troubles enabling ffmpeg qsv hardware acceleration on my i7-12650H CPU, managed to solve it.

<!--more-->

## Problem

ffmpeg fails with:

```
[AVHWDeviceContext @ 0x562068479140] Error creating a MFX session: -9.
Device creation failed: -1313558101.
[vist#0:0/hevc @ 0x5620682fef40] [dec:hevc_qsv @ 0x5620683c2700] No device available for decoder: device type qsv needed for codec hevc_qsv.
[vist#0:0/hevc @ 0x5620682fef40] [dec:hevc_qsv @ 0x5620683c2700] Hardware device setup failed for decoder: Unknown error occurred
```

## oneVPL

12650H is Alder Lake (gen12), oneVPL is required according to: https://trac.ffmpeg.org/wiki/Hardware/QuickSync

Install `onevpl-tools` and run `vpl-inspect` gives:

```
$ vpl-inspect
Warning - no implementations found by MFXEnumImplementations()
```

Looking at the [debian source package](https://packages.debian.org/source/bookworm/onevpl-intel-gpu), there is a shared library that must also be installed: `libmfx-gen1.2`

Now `vpl-inspect` output seems good:

{% collapsecard Expand Collapse %}

```
$ vpl-inspect
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_17
libva info: va_openDriver() returns 0
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_17
libva info: va_openDriver() returns 0

Implementation #0: mfx-gen
  Library path: /usr/lib/x86_64-linux-gnu/libmfx-gen.so.1.2.8
  AccelerationMode: MFX_ACCEL_MODE_VIA_VAAPI
  ApiVersion: 2.8
  Impl: MFX_IMPL_TYPE_HARDWARE
  VendorImplID: 0x0000
  ImplName: mfx-gen
  License: MIT License
  Version: 1.2
  Keywords:
  VendorID: 0x8086
  mfxAccelerationModeDescription:
    Version: 1.0
    Mode: MFX_ACCEL_MODE_VIA_VAAPI
  mfxPoolPolicyDescription:
    Version: 1.0
    Policy: MFX_ALLOCATION_OPTIMAL
    Policy: MFX_ALLOCATION_UNLIMITED
    Policy: MFX_ALLOCATION_LIMITED
  mfxDeviceDescription:
    DeviceID: 46a3/0
    Version: 0.0
  mfxDecoderDescription:
    Version: 1.0
    CodecID: VC1
    MaxcodecLevel: 5
    CodecID: AV1
    MaxcodecLevel: 63
      Profile: MFX_PROFILE_AV1_MAIN
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
      Profile: MFX_PROFILE_AV1_HIGH
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
      Profile: MFX_PROFILE_AV1_PRO
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
    CodecID: VP8
    MaxcodecLevel: 0
      Profile: MFX_PROFILE_VP8_0
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_VP8_1
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_VP8_2
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_VP8_3
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
    CodecID: VP9
    MaxcodecLevel: 0
      Profile: MFX_PROFILE_VP9_0
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
      Profile: MFX_PROFILE_VP9_1
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
      Profile: MFX_PROFILE_VP9_2
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
      Profile: MFX_PROFILE_VP9_3
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, AYUV, Y410, Y416
    CodecID: AVC
    MaxcodecLevel: 62
      Profile: MFX_PROFILE_AVC_BASELINE
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_AVC_MAIN
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_AVC_HIGH
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_AVC_EXTENDED
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 4096
          Width Step: 16
          Height Min: 16
          Height Max: 4096
          Height Step: 16
          ColorFormats: NV12
    CodecID: MPG2
    MaxcodecLevel: 6
      Profile: MFX_PROFILE_MPEG2_SIMPLE
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_MPEG2_MAIN
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
      Profile: <unknown MFX_CODEC_MPEG2 profile>
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 2048
          Width Step: 16
          Height Min: 16
          Height Max: 2048
          Height Step: 16
          ColorFormats: NV12
    CodecID: HEVC
    MaxcodecLevel: 62
      Profile: MFX_PROFILE_HEVC_MAIN
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_HEVC_MAIN10
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010
      Profile: MFX_PROFILE_HEVC_MAINSP
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12
      Profile: MFX_PROFILE_HEVC_REXT
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, YUY2, Y210, Y216, AYUV, Y410, Y416
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, P016, YUY2, Y210, Y216, AYUV, Y410, Y416
      Profile: MFX_PROFILE_HEVC_SCC
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
    CodecID: JPEG
    MaxcodecLevel: 0
      Profile: MFX_PROFILE_JPEG_BASELINE
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, RGB4, YUY2
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 16384
          Width Step: 16
          Height Min: 16
          Height Max: 16384
          Height Step: 16
          ColorFormats: NV12, RGB4, YUY2
  mfxEncoderDescription:
    Version: 1.0
    CodecID: VP9
    MaxcodecLevel: 0
    BiDirectionalPrediction: 0
    ReportedStats: 0
      Profile: MFX_PROFILE_VP9_0
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
      Profile: MFX_PROFILE_VP9_1
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
      Profile: MFX_PROFILE_VP9_2
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
      Profile: MFX_PROFILE_VP9_3
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 16
          Width Max: 8192
          Width Step: 16
          Height Min: 16
          Height Max: 8192
          Height Step: 16
          ColorFormats: NV12, P010, AYUV, Y410
    CodecID: HEVC
    MaxcodecLevel: 318
    BiDirectionalPrediction: 1
    ReportedStats: 0
      Profile: MFX_PROFILE_HEVC_MAIN
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
      Profile: MFX_PROFILE_HEVC_MAIN10
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
      Profile: MFX_PROFILE_HEVC_MAINSP
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
      Profile: MFX_PROFILE_HEVC_REXT
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
      Profile: MFX_PROFILE_HEVC_SCC
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 8
          Width Max: 16384
          Width Step: 8
          Height Min: 8
          Height Max: 12288
          Height Step: 8
          ColorFormats: P010, P210, Y210, Y410, RG10, NV12, YUY2, RGB4, BGR4, P016, Y216, Y416, AYUV
    CodecID: JPEG
    MaxcodecLevel: 0
    BiDirectionalPrediction: 0
    ReportedStats: 0
      Profile: MFX_PROFILE_JPEG_BASELINE
        MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
          Width Min: 1
          Width Max: 16384
          Width Step: 1
          Height Min: 1
          Height Max: 16384
          Height Step: 1
          ColorFormats: NV12, YV12, YUY2, RGB4
        MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
          Width Min: 1
          Width Max: 16384
          Width Step: 1
          Height Min: 1
          Height Max: 16384
          Height Step: 1
          ColorFormats: NV12, YV12, YUY2, RGB4
  mfxVPPDescription:
    Version: 1.0
    FilterFourCC: PAMP
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
    FilterFourCC: MCTF
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12
    FilterFourCC: DNIS
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
    FilterFourCC: DET
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
    FilterFourCC: FRC
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410
    FilterFourCC: ROT
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: MIRR
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: VSCL
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: TDLT
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: VCSC
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: VPDI
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, P016
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, P016
        InFormat: P010
          OutFormats: NV12, YUY2, P010, P016
        InFormat: P016
          OutFormats: NV12, YUY2, P010, P016
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, P016
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, P016
        InFormat: P010
          OutFormats: NV12, YUY2, P010, P016
        InFormat: P016
          OutFormats: NV12, YUY2, P010, P016
    FilterFourCC: FPRO
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12
    FilterFourCC: VCLF
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
    FilterFourCC: FIWF
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
    FilterFourCC: FISF
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: YUY2
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P010
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: AYUV
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y210
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y410
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: P016
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y216
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
        InFormat: Y416
          OutFormats: NV12, YUY2, P010, AYUV, Y210, Y410, P016, Y216, Y416
    FilterFourCC: VCMP
    MaxDelayInFrames: 0
      MemHandleType: MFX_RESOURCE_SYSTEM_SURFACE
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
      MemHandleType: MFX_RESOURCE_VA_SURFACE_PTR
      Width Min: 16
      Width Max: 16384
      Width Step: 1
      Height Min: 16
      Height Max: 16384
      Height Step: 1
        InFormat: NV12
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: YUY2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: BGR4
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: RGB2
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P010
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: AYUV
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y210
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y410
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: P016
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y216
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
        InFormat: Y416
          OutFormats: NV12, YUY2, RGB4, BGR4, RGB2, RGBP, P010, RG10, AYUV, Y210, Y410, P016, Y216, Y416, BGRP
  NumExtParam: 0

Total number of implementations found = 1
```

{% endcollapsecard %}

ffmpeg now also able to use qsv.

Although I now have some other issues with qsv encoding parameters, but that's some other unrelated issue:

```
[hevc_qsv @ 0x5da8c1b01800] Low power mode is unsupported
[hevc_qsv @ 0x5da8c1b01800] some encoding parameters are not supported by the QSV runtime. Please double check the input parameters.
[vost#0:2/hevc_qsv @ 0x5da8c1e2a440] [enc:hevc_qsv @ 0x5da8c1c0f500] Error while opening encoder - maybe incorrect parameters such as bit_rate, rate, width or height.
```
