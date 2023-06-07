---
title: processing-media-for-the-vault
categories:
tags:
Description: 
---


A collection of notes I wrote up for a friend who's in the process of reviving their plex library

Caveats are that this is based on early 2023 assumptions around storage and playback support for 1080p content on local devices with hardware support for h265 (nvidia shield, modern GPUs, etc).

The author uses a combination of [makemkv](https://www.makemkv.com/), [handbrake](https://github.com/HandBrake/HandBrake), [mkvtoolnix](https://mkvtoolnix.download/), [ffmpeg CLI](https://ffmpeg.org/), and [Subtitle Edit](https://github.com/SubtitleEdit/subtitleedit).

## Video Notes
* HEVC gives best balance between compression and direct-stream compatibility.  AV1 is the new hotness, but the variability of support means that in practical terms, everything will end up requiring transcode by the server 

* NVENC (Nvidia GPU) based encodes go significantly faster, however have worse overall compression and can introduce some artifacting (this also applies to AMD VCE). Gold standard for live decode/transcode, bad for processing your "master" copy 

* CPU encode effectiveness scales with encoder preset to a point. I run at medium, and while it takes a little longer than (ultra/super/very/)fast(er), there is an appreciable difference in final encode size. diminishing returns will hit hard at slow+ 

    * Handbrake has an interesting bottleneck associated with small/low-bitrate files which can result in occupying threads as expected, but not truly loading the CPU.  This is not typically an issue with 1080p+ video, but running for 480p items will leave a lot of headroom. Running these smaller items in parallel will remove that bottleneck, but running multiple 1080p+ in parallel will result in significantly worse performance.

* base (8-bit) vs 10-bit color depth is to-taste. IIRC 10bit has a little more overhead to transcode if it's required to, but it's a lifesaver on reducing banding and other artifacts if it's for direct play (this improvement functions independently of your quality slider) 

* quality slider value is totally up to you. I used to run them at 19-20 before backing off to ~22, but unless you're pixel-peeping, I've found 24-26 to be the "sweet spot" where you get maximum space savings without it impacting the experience (caveat, 55" tv, referring to SDR content @ 24NTSC-25PAL FPS) 

* MATCH FRAMERATE TO SOURCE. "most" things will use 23.976NTSC, but britcoms use PAL 25FPS, and other things can use 30FPS, if you specifying this framerate (say, by setting it one day and forgetting about it) you will screw up any subtitle timings and introduce a small but noticeable amount of "stutter".

* Do not process HDR content. Handbrake/FFMPEG haets dealing with the associated luminance/colorspace/etc metadata, even if you specify it to just be passed through via the CLI. 

  * As a general rule, if HDR content is not direct-played it will be transcoded to SDR using whatever tonemapping the system uses.  This is a Bad Thing.

  * Dolby Video carries all of the HDR caveats, but with an added layer of proprietary compatibility issues which may cause it to fall back to HDR10

  * Another fun caveat is that TVs often have a full suite of settings for SDR vs HDR content.  If you're direct-playing HDR content and it does not behave as expected, your TV may swapped into its HDR copy of the default settings and may have added motion-smoothing

## Audio Notes

* Opus is your friend. I typically run at 320kbps mixdown to best-available (up to surround 5.1). While re-encoding from lossless formats is naturally contentious 
* Opus-specific complaints typically focusing on “trimming” referring to spec definitions: […Opus never codes audio above 20 kHz, as that is the generally accepted upper limit of human hearing.](https://www.rfc-editor.org/rfc/rfc6716#section-2)