---
author: HKL
categories:
- 默认分类
date: "2023-08-01T12:44:00+08:00"
slug: convert-m4a-to-8bit-wav
status: publish
tags:
- ffmpeg
- Music
- ChatGPT
title: Convert m4a/mp3 to 8bit WAV using ffmpeg
---

I Convert a m4a/mp3 to 8bit WAV using ffmpeg recently!

Steps:

1. Download and Install "ffmpeg" windows build,

2. run `D:\soft\ffmpeg> .\bin\ffmpeg.exe -i .\luocha2.mp3 -ar 8000 -ac 1 -acodec pcm_u8 output3.wav`

<!--more-->

```bash
PS D:\soft\ffmpeg> .\bin\ffmpeg.exe -i .\luocha2.mp3 -ar 8000 -ac 1 -acodec pcm_u8 output3.wav
ffmpeg version N-111652-gbf9f6a5e55-20230730 Copyright (c) 2000-2023 the FFmpeg developers
  built with gcc 13.1.0 (crosstool-NG 1.25.0.196_227d99d)
  configuration: --prefix=/ffbuild/prefix --pkg-config-flags=--static --pkg-config=pkg-config --cross-prefix=x86_64-w64-mingw32- --arch=x86_64 --target-os=mingw32 --enable-gpl --enable-version3 --disable-debug --enable-shared --disable-static --disable-w32threads --enable-pthreads --enable-iconv --enable-libxml2 --enable-zlib --enable-libfreetype --enable-libfribidi --enable-gmp --enable-lzma --enable-fontconfig --enable-libvorbis --enable-opencl --disable-libpulse --enable-libvmaf --disable-libxcb --disable-xlib --enable-amf --enable-libaom --enable-libaribb24 --enable-avisynth --enable-chromaprint --enable-libdav1d --enable-libdavs2 --disable-libfdk-aac --enable-ffnvcodec --enable-cuda-llvm --enable-frei0r --enable-libgme --enable-libkvazaar --enable-libass --enable-libbluray --enable-libjxl --enable-libmp3lame --enable-libopus --enable-librist --enable-libssh --enable-libtheora --enable-libvpx --enable-libwebp --enable-lv2 --enable-libvpl --enable-openal --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenh264 --enable-libopenjpeg --enable-libopenmpt --enable-librav1e --enable-librubberband --enable-schannel --enable-sdl2 --enable-libsoxr --enable-libsrt --enable-libsvtav1 --enable-libtwolame --enable-libuavs3d --disable-libdrm --disable-vaapi --enable-libvidstab --enable-vulkan --enable-libshaderc --enable-libplacebo --enable-libx264 --enable-libx265 --enable-libxavs2 --enable-libxvid --enable-libzimg --enable-libzvbi --extra-cflags=-DLIBTWOLAME_STATIC --extra-cxxflags= --extra-ldflags=-pthread --extra-ldexeflags= --extra-libs=-lgomp --extra-version=20230730
  libavutil      58. 14.100 / 58. 14.100
  libavcodec     60. 22.100 / 60. 22.100
  libavformat    60. 10.100 / 60. 10.100
  libavdevice    60.  2.101 / 60.  2.101
  libavfilter     9. 10.100 /  9. 10.100
  libswscale      7.  3.100 /  7.  3.100
  libswresample   4. 11.100 /  4. 11.100
  libpostproc    57.  2.100 / 57.  2.100
Input #0, mp3, from '.\luocha2.mp3':
  Metadata:
    encoder         : Lavf60.3.100
  Duration: 00:05:32.83, start: 0.023021, bitrate: 140 kb/s
  Stream #0:0: Audio: mp3, 48000 Hz, stereo, fltp, 140 kb/s
    Metadata:
      encoder         : Lavc60.3.
File 'output3.wav' already exists. Overwrite? [y/N] y
Stream mapping:
  Stream #0:0 -> #0:0 (mp3 (mp3float) -> pcm_u8 (native))
Press [q] to stop, [?] for help
Output #0, wav, to 'output3.wav':
  Metadata:
    ISFT            : Lavf60.10.100
  Stream #0:0: Audio: pcm_u8 ([1][0][0][0] / 0x0001), 8000 Hz, mono, u8, 64 kb/s
    Metadata:
      encoder         : Lavc60.22.100 pcm_u8
[out#0/wav @ 000001a4ecd49cc0] video:0kB audio:2600kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.002930%
size=    2600kB time=00:05:32.79 bitrate=  64.0kbits/s speed= 741x
PS D:\soft\ffmpeg>
```

A example review Luoshahaishi:

https://important-elbow-bba.notion.site/luoshahaishi-ab9c18989f3f40e79892d5c1877cc84a?pvs=4

For more information about ffmpeg:

- Website: http://www.ffmpeg.org/

- Wikipedia: http://en.wikipedia.org/wiki/FFmpeg

- Documentation: http://www.ffmpeg.org/ffmpeg-doc.html


