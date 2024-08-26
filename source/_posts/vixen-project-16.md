---
title: Vixen project
tags:
  - codec
  - ffmpeg
  - Video
id: '839'
categories:
  - - C/C++
  - - c
    - Vixen
date: 2017-07-07 19:43:23
---

Project meeting this week was cancelled.

I have implemented video decoding and encoding in C/C++ using `ffmpeg`. I can now encode the sequence (~337MB) to a video file (~26.6MB), and then decode it back to the exactly same sequence, losslessly, using `h264` with `yuv444p` pixel format.
<!-- more -->
The decoding program can accept any video formats `ffmpeg` supports. Therefore, I can post-process/transcode the video using any existing programs, for example, the `ffmpeg` command line tool. The decoding program will still be able to read the video.

I have tested a few other encoding codecs and quality parameters by transcoding, and tested pixel accuracy using `ffmpeg`’s `PSNR` filter, the results are shown below: (`PSNR` in `dB`)

![data](data.png)

Input (original) formats: `yuv444p_yuv`: Channels ordered as packed `YUV (yuvyuv...)`, encoded as `yuv444p` `yuv444p_y-u-v`: Channels ordered as planar `YUV (yy...uu...vv...)`, encoded as `yuv444p` `yuv444p_rgb`: Channels ordered as packed `RGB (rgbrgb...)`, encoded as `yuv444p` `rgb24_rgb`: Channels ordered as packed `RGB`, encoded as `rgb24`

Transcoding output formats: \[encoding\]-\[quality factor (lower the better)\]

Uploaded some sample videos, with 8x upscaling: [Vixen Project playlist](https://www.youtube.com/playlist?list=PLxL3DVSgdZueJIEqVfMt0vup68Ke8MndG) However, the video player uses interpolation, so the video will be distorted. The original video has clear boundaries between pixels.

I will be working on porting encoding & decoding to C#, recording controller information to media file metadata, and mux audio & video channels.

Ref: [e66b0392249f971f8368a517dfd1e71e1d4c9a6e](https://github.com/zhiyb/EE9-APRJ/tree/e66b0392249f971f8368a517dfd1e71e1d4c9a6e)