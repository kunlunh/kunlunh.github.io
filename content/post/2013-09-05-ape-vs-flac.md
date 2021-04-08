---
author: HKL
categories:
- 默认分类
cid: 16
date: "2013-09-05T09:58:00Z"
slug: ape-vs-flac
status: publish
tags:
- Opensource
- Music
- Office
title: Monkey's Audio vs. WavPack vs. FLAC
updated: 2019/01/29 16:44:46
---


Background (背景)

  For a long time I've been thinking about getting a HTPC running Windows Media Center Edition.  I'm not quite there yet for various reasons. However, one of the problems I'm trying  to solve is what to do with my music collection. I recently bought some very decent  loudspeakers, and now my CD collection is growing again.

  For the HTPC one could argue that space is not really an issue. With a 750 GB harddrive  I can probably store somewhere between 1.000 and 1.500 uncompressed CD's. The first problem is,  I also need space for more demanding content: Video recordings. Secondly, I need a  format that can be transfered to my portable music player. Finally, WAV lacks tag support, i.e.  no metadata for the music files.

  Sorry about the messy state this article is in. I never got around to finishing it, so I'll  just publish as it is.

  （原作者大概是想要给家里添一台htpc,但是750G的硬盘可能不能放完ta的wav格式的唱片「应该是正版，我们不用考虑这东西」，所以就作者开始找个较好的无损音频压缩格式了）


<!--more-->


I'm faced with the following dilemmas when choosing an audio format:  (主要对比以下几点)

**Quality requirements (音质)**

**Fair license (使用协议)**

**Tag support (标签支持)**

**Ease of use (possibility of automatic compression and tagging) (使用便捷度)**

**Software/hardware support (软/硬件兼容)**


**Quality requirements(音质)**

I'm not sure which MP3 or AAC bitrate is "transparent" to me, and of course, it depends on the  music being encoded. With MP3 I probably need at least 192 kbps for casual listening - and more  to feel sure that I won't one day suddenly start noticing artifacts in some of the songs. With  AAC some say that 150 kbps is "transparent" to most listeners. It's hard to decide exactly which  bitrate to choose. Beyond 200 kbps takes up too much space on portable devices and is probably  overkill. So it's either a compromise in quality, or separate encodings for the HTPC and portable  devices.

(mp3与aac都是有损压缩,作者觉得少于192kbp/s的mp3与少于150kbp/s的aac就不用考虑了,特别是对于htpc来说)

**Fair license (使用协议)**

You already paid for the music and the cellphone, HTPC or whatever. How much are you willing topay extra for proprietary codec support? Restrictive licenses limits the support/availability.

(我们买手机要花钱,买htpc要花錢.总不能再花钱买个音频格式的授权费用了[天朝人民这个倒不用考虑,只是基本上无损音频格式都是开源的])

**Tag support (标签支持)**

The minimal amount of expected metadata fits into the old and primitive ID3v1 tags. While this  usually may be enough for most people, more advanced content such as album art, lyrics, etc. might  also be nice for certain applications.

(至少要支持老旧的ID3v1标签)

**Ease of use (使用便捷度)**

The manual process when ripping a music album should involve a minimum amount of steps. Does the  ripper integrate with the encoder, is CDDB-like databases supported for fetching of metadata, etc.  Manually entering tag metadata tends to become boring.

(主要是开转码是否方便)

**Software/hardware support (软/硬件兼容)**

For the broadest support MP3 with ID3v1 tags is obviously the road to follow. However, many modern  codecs are now supported in various applications: Cell phones, MP3 players, DVD players, traditional  software players for various platforms and even LCD TV's with built-in card readers.


Initial impressions

  First I went to Monkey Audio's website without  knowing that my journey would not end there. I already knew it to have the best compression  ratio, so I went ahead and installed the package, including the Winamp plugin. I compressed  an album: Shania Twain - Come On Over. So far so good - everything worked, and the files were  playable in Winamp. Tags were missing, but the package included an excellent plugin for Winamp.

  In the application used to compress the files I noticed an option to spawn an external coder,  and I found the WavPack command line encoder. Out of curiousity  I went and fetched the latest version of this.

  After a while just before buying a new Denon surround receiver, I found out that it supported  FLAC files. I experimented with FLAC earlier, but never found a reason to favour this format  over WavPack. This changed immediately.

The battleEncoding results

  All files are encoded with "high" quality in both Monkey's Audio and WavPack, and compression  level 6 for FLAC.

FlexibilitySoftware support

Winamp: Works equally well with WavPack and Monkey's Audio through the supplied plugins.  Monkey's Audio comes with a tag editor (including a picture of a monkey), so one could argue that this  will bring it one star ahead of WavPack. FLAC is natively supported by Winamp and includes a nice tag  editor.

Windows Media Center Edition and Media Player: WavPack comes with a DirectSound Filter. After installing  this, both MCE and MP will play the files like they were its own babies (WMA). A DirectSound Filter is also  available for FLAC.

Nero Burning ROM: Nero plugins exist for WavPack and FLAC, enabling full support when burning Audio CD's.  These plugins works so great that you won't notice that it's there. They also support tags, which is useful  when including CD TEXT information for car CD players.

Exact Audio Copy: FLAC and WavPack both have command line options for tagging, so they can be spawned  automatically from EAC in a neat way.

Directory Opus: Native support for FLAC files, which means they are fully supported by the Folder Options  attribute system.

Hardware support

  Denon's A/V receivers AVR-3808 and AVR-4308 have FLAC support. They can play FLAC files from a UPnP server  This is very neat if you don't use a HTPC for playing the files, but have them stored on a NAS or  regular PC far away from the Hi-Fi equipment.

Tag support

  Monkey's Audio and WavPack both use APEv2 tags. FLAC uses its own type of tags. There's a plug-in for Media Player  that adds support for both APEv2 and FLAC tags. FLAC tags are natively supported by Directory Opus.

Conclusion

  Monkey's Audio compresses slightly better than WavPack, and WavPack compresses slighty better than FLAC.  But this alone doesn't win the competition - let's have a look at the scoreboard:

  The difference in compression efficiency is so small that it's hardly relevant for making the  best decision. The software and hardware support is far more important, and here FLAC ultimately  wins. I made my choice and converted all my WavPacks to FLAC. I now enjoy my music collection  directly from my Denon receiver and never looked back. It would be nice with PlayStation 3  support, but this goes for all three formats. I've made a small GUI frontend to easily convert  my FLAC files to AAC files I can use on my cellphone. I'll probably release this later on.

Tips & tricks

  Exact Audio Copy compression settings:  WavPack

    Program, including path, used for compression: wavpack.exe    

    Additional command line options: -w "Artist=%a" -w "Title=%t" -w "Album=%g" -w "Year=%y" -w "Track=%n" -w "Genre=%m" -h %s %d

FLAC

    Program, including path, used for compression: flac.exe    

    Additional command line options: --compression-level-6 -T "artist=%a" -T "title=%t" -T "album=%g" -T "date=%y" -T "tracknumber=%n" -T "genre=%m" %s

by Jacob Laursen, 2007 at http://www.vindvejr.dk/jacob/lossless.php
