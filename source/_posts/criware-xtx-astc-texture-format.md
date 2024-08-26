---
title: CRIWARE xtx ASTC texture format
tags:
  - galgame
  - Jetson
  - NVIDIA
  - Tegra
  - Texture
id: '1316'
categories:
  - - Games
  - - games
    - Nintendo Switch
date: 2019-11-30 09:41:14
---

After setting up CFW on my Nintendo Switch, I dumped certain game cartridge and tried to extract its content. The game was created using CRIWARE engine. (D4D63B5B034258311F243102C3D327AD)

So, here is my adventure on dumping image assets from the game.
<!-- more -->
* * *

First, extract contents in .xci game cartridge dump using [hactool](https://github.com/SciresM/hactool): \[code\]hactool.exe --securedir=secure -t xci rom.xci\[/code\]

Then, find the largest .nca, and extract again using hactool: \[code\]hactool.exe -k prod.keys -t nca rom.nca\[/code\]

For CRIWARE, there will be multiple .cpk archive files. There are several tools that can extract them. [YACpkTool](https://github.com/Brolijah/YACpkTool) works just fine for me.

Drag & drop the .cpk file onto YACpkTool.exe, it will be extracted to the same folder.

* * *

Now, to the main reverse engineering part.

A lot of dumped files have `xtx` magic signature in the header, e.g.

\[code\] # hd station.bin head 00000000 10 00 00 00 20 a4 1f 00 00 00 00 00 00 00 00 00 .... ........... 00000010 78 74 78 00 09 00 00 00 00 00 07 80 00 00 04 38 xtx............8 00000020 00 00 07 80 00 00 04 38 00 00 00 00 00 00 00 00 .......8........ 00000030 51 02 2f 36 d6 f0 98 a3 41 8b bf 2d 73 42 6a 64 Q./6....A..-sBjd 00000040 51 02 2d 34 d4 ee 96 a5 01 37 ff d8 0c f5 2d e7 Q.-4.....7....-.\[/code\]

\[code\] # hd face.bin head 00000000 78 74 78 00 09 00 00 00 00 00 02 58 00 00 02 10 xtx........X.... 00000010 00 00 02 58 00 00 02 0d 00 00 00 00 00 00 00 00 ...X............ 00000020 fc fd ff ff ff ff ff ff 00 00 00 00 00 00 00 00 ................ \* 0000c110 42 40 0f 7d 7b 01 00 36 00 00 00 00 03 00 00 00 B@.}{..6........\[/code\]

They are CRIWARE images files, however don't confuse them with Nintendo Switch XTX(NvFD) texture files - they are completely different things. I wasted a lot of time on this, but found a nice utility: [Raw Texture Cooker](https://zenhax.com/viewtopic.php?t=7099) (requires [texconv.exe](https://github.com/microsoft/DirectXTex/releases)).

As shown above, some files have an additional header before the xtx magic signature:

\[code\] 00000000 10 00 00 00 20 a4 1f 00 00 00 00 00 00 00 00 00 .... ........... \[/code\]

Not sure what exactly is this header for. `10 00 00 00` seems to be some kind of signature, followed by `20 a4 1f 00`, the size of next xtx data section. Then, padded by 8 bytes of zeros.

From [a forum post](https://zenhax.com/viewtopic.php?t=9021), I found that [arc\_unpacker has an implementation of `cri/xtx` decoder](https://github.com/vn-tools/arc_unpacker/blob/b9843a13e2b67a618020fc12918aa8d7697ddfd5/src/dec/cri/xtx_image_decoder.cc).

From the decoder implementation and sample files, the xtx file header is structured as: \[code\] 4 bytes header (xtx\\0) 4 bytes format (0,1,2,9 or other unknowns) 4 bytes aligned width 4 bytes aligned height 4 bytes width 4 bytes height 4 bytes x offset 4 bytes y offset \[/code\]

The xtx decoder in arc\_unpacker only implemented texture format 0, 1 and 2 (There is probably a segmentation fault bug with format 2, but I don't have an example image for testing). However here we have an unknown format 9.

Putting the texture data section into the Raw Texture Cooker tool mentioned earlier, and trying various texture types supported by the tool, shows that certain compression formats seem to give better results, e.g. BC7:

![Raw texture cooker 30_11_2019 02_01_49.png](raw-texture-cooker-30_11_2019-02_01_49.png)

Looking into the details of those compression formats, it's highly likely that the actual texture compression format is 4x4 block based, 16 bytes per block.

There are also repeating patterns in texture data. Together with likely usage of that texture, I can roughly guess the expected colour of that block, e.g.

\[code\] # Full transparency 00000020 fc fd ff ff ff ff ff ff 00 00 00 00 00 00 00 00 ................ # Pure black 00000030 fc fd ff ff ff ff ff ff 00 00 00 00 00 00 ff ff ................ \[/code\]

At this point, I still don't know what's the actual compression format. But I guess these data will be send directly to the Tegra X1 GPU in Nintendo Switch, so I turned to search for compression formats supported by Tegra X1.

After a quick Google search, an article from NVIDIA comes up directly, [suggesting ASTC texture compression for game assets](https://developer.nvidia.com/astc-texture-compression-for-game-assets), which also supports 4x4 8bpp mode.

Another quick search, I found [an ASTC texture conversion tool from ARM on GitHub](https://github.com/ARM-software/astc-encoder). So, let's try it.

\[code\] # Compress a PNG image to an ASTC texture astcenc -c black.png black.astc 4x4 -medium # Decompress an ASTC texture to a TGA image astcenc -d test.astc test.tga \[/code\]

Converting a pure black 1920x1080 PNG image to 4x4 ASTC texture, hexdump shows:

\[code\] # hd black.astc head 00000000 13 ab a1 5c 04 04 01 80 07 00 38 04 00 01 00 00 ...\\......8..... 00000010 fc fd ff ff ff ff ff ff 00 00 00 00 00 00 ff ff ................ \* 001fa410 \[/code\]

Well, that repeating pixel block data is exactly the same as the black block I guessed earlier. So, I separated out the ASTC file header, concatenated with texture data from another one of dumped game assets, decompressed it. And, I got a perfectly reconstructed image, with flipped y-axis. ASTC compression confirmed.

Now, I have two options. Either write some scripts to generate a valid ASTC file header, extract and concatenate texture data, then use the ARM tool to decompress. Or, implement an ASTC decoder in arc\_unpacker. I end up copying most of ARM's source files into arc\_unpacker, and my arc\_unpacker can now extract xtx format 9. Not sure if ARM's license allows me to publish the modified arc\_unpacker...

It's quite amazing that these texture compression algorithms can reduce data rate down to 8bpp, whilst still having excellent image quality.

For details: [BC1 to BC5 formats (Direct3D 10)](https://docs.microsoft.com/en-gb/windows/win32/direct3d10/d3d10-graphics-programming-guide-resources-block-compression) [BC6 and BC7 formats (Direct3D 11)](https://docs.microsoft.com/en-us/windows/win32/direct3d11/texture-block-compression-in-direct3d-11) [ASTC Compressed Texture Image Formats](https://www.khronos.org/registry/DataFormat/specs/1.2/dataformat.1.2.html#ASTC)

* * *

By the way, for HCA audio files, deretore-toolkit has a working hca to wav converter.

* * *

![ID00010.png](id00010.png)