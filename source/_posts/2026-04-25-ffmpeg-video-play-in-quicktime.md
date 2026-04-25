---
title: ffmpeg输出的视频文件无法在quicktime播放
date: 2026-04-25 13:42:01
tags:
  - Ffmpeg
---

原因是视频的色彩空间问题，QuickTime 默认只支持YUV420的视频，如果ffmpeg输入的原视频色彩空间超过了YUV420，如果没有任何filter，那么输出的视频色彩空间也是保持一致。

所以需要在ffmpeg里指定色彩空间 `ffmpeg … -pix_fmt yuv420p output.mp4`


