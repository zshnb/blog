---
title: ffmpeg给图片加zoom效果导致图片抖动
date: 2024-04-25 18:21:23
tags:
  - Ffmpeg
---

在使用ffmpeg给图片拼接的视频加放大动画时，出现一个很奇怪的问题，如果指定放大的位置点在中心，画面中的图片会在放大的同时来回抖动，好像网络卡了的感觉，如下面的动图：
<!--more-->
![动图](img1.gif)
最后从[这里](https://superuser.com/questions/1112617/ffmpeg-smooth-zoompan-with-no-jiggle/1112680#1112680)找到了解决办法，回答解释是因为在zoom的过程中，x，y在变化的时候会四舍五入，导致不均匀，想象一下就是细微的突然左一下右一下。可以通过在zoompan之前提高图片分辨率来消除抖动

```shell
ffmpeg -y -t 3 -i input.png -filter_complex "[0:v]scale=8000*4000,zoompan=z='min(zoom+0.0015,1.5)':d=75:x='if(gte(zoom,1.5),x,x+1)':y='y'[v0];[v0]concat=n=1:v=1:a=0,format=yuv420p[v]" -map "[v]" -s "1024x1792" ./tmp/AFTJ3W/concat.mp4
```


