---
title: Vixen project
tags: []
id: '160'
categories:
  - - c
    - Vixen
date: 2017-02-27 14:00:03
---

Project meeting @20/Feb/2017 cancelled due to no significant discoveries. Supervisor was not in the college @27/Feb/2017.

I added a spearate playback path to Vixen, and successfully playback the exported sequence to the controllers. The application uses only ~6% CPU time with 50 FPS playback. All active controllers are running at their specified frame rates too.
<!-- more -->
When the playback is stopped, it switches back to the original execution engine. However, the execution engine takes ~45% CPU time even when no sequence is playing.

It takes about 18 minutes to export the 06:49 sequence with 50 FPS. The file size is 0.98 GB, but I should be able to reduce the file size at least to a third, by improving the data storage structure.

The media file associated with the sequence was not recorded, and there is no audio playback currently.

The playback interface is very basic, with only start and stop buttons. A more complete interface may be developed, and perhaps integrated with schedulers.

The playback reads the data file sequentially on-the-fly, only caching 1 byte for each channel output. So not much memory was used in the playback path. Only 8-bit commands can be exported currently, but this can be extended.

Because the Vixen application supports audio playback in the original sequence engine, so maybe I need to add the audio information as well. Probably some lighting show projects can involve audio effects.

Vixen also has schedulers to play sequences at specific date and time. It seems I can quite easily fit the data dump playback control in it.

I haven't thought about the preview yet.. It may be possible to feed the data dump to the preview, if the element to controller mapping was saved during export, or computed before preview.

Probably the data dump can be saved as a video stream, so some video processing algorithms can be used for compressing or something else.

Ref: [830a17bb445aab1c98ddd44540cc77a0b701177b](https://github.com/zhiyb/vixen/commit/830a17bb445aab1c98ddd44540cc77a0b701177b)

![Vixen playback screenshot](vixen-playback-screenshot.jpg)![Vixen playback performance](vixen-playback-performance.jpg)
