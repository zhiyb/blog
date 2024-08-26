---
title: Vixen project
tags:
  - codec
  - DLL
  - ffmpeg
  - Video
id: '943'
categories:
  - - C#
  - - c
    - Vixen
date: 2017-07-31 02:20:54
---

Last week's project meeting was cancelled.

Implemented audio muxing encoding and decoding, and integrated into the Vixen application.
<!-- more -->
The audio stream was muxed by directly copying the audio packets to a new stream in output video file, so no encoding or decoding in the process. Therefore the speed of creating the audio stream is very fast. Ref: [9471f67010b5ac5a8aba6dd8236cc06da7860e34](https://github.com/zhiyb/EE9-APRJ/tree/9471f67010b5ac5a8aba6dd8236cc06da7860e34)

However, audio decoding and encoding was still implemented, as the project will need audio decoding to playback the audio samples. Need to find out how to play audio on Linux and Windows systems, separate libraries and routines may be needed.

The library was directly incorporated into the original Vixen System, by adding a C# file providing the interface and using the functions accordingly. Making this a separate runtime loadable module might be more fit into the Vixen Structure? Ref: [d0fc1215248e0d555172eeef69787e4df03aec5b](https://github.com/zhiyb/vixen/commit/d0fc1215248e0d555172eeef69787e4df03aec5b)

The Visual Studio solution file and some projects related to WPF and media were changed to be able to compile on Linux using mono. These need to be incorporated into upstream somehow.

The project report should be started writing at this time.