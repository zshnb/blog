---
title: 本地编译最新版本ffmpeg，支持CUDA和zscale、libplacebo
date: 2023-08-05 15:21
tags:
- Ffmpeg
---

# 背景
最近有一个需求，使用ffmpeg把非yuv420p色彩的视频转换成yuv420p bt709色彩，这个需求的具体解决过程放在下一篇文章，此文仅介绍本地ffmpeg编译内容。

因为在搜索色彩空间转换的过程中，发现了一些需要额外构建的filter，以及需要支持CUDA硬件加速，于是决定自己编译一个最新版本的ffmpeg，同时记录下此次编译的操作过程，希望可以帮到其他有需要的人。
<!--more-->

# 准备
1. 一台带Nvidia GPU的机器，同时安装好CUDA
2. [Nvidia官方教程](https://docs.nvidia.com/video-technologies/video-codec-sdk/11.1/ffmpeg-with-nvidia-gpu/index.html#compiling-for-linux)，如何编译开启CUDA的ffmpeg

此处最重要的就是ffmpeg的./configure 后面跟着的参数了，上面官方教程里仅仅给了最基础的开启CUDA的参数，还有非常多基础参数没有加上去。我的做法是复制目前生产环境在用的ffmpeg里的参数，加上CUDA需要的参数，作为最终编译的参数。

# 开始编译
一切准备好之后，开始执行`./configure` ，ffmpeg会出现各种xxx not found in pkg-config的错误，这是因为本地没有对应的lib，需要一个个安装缺失的lib。我在搜索这个错误的时候，发现了[https://forums.raspberrypi.com/viewtopic.php?t=316065](https://forums.raspberrypi.com/viewtopic.php?t=316065)，里面有最初出现的几个错误的解决办法，于是发现了一个规律，大部分的lib都是libxxxx-dev，一个个安装即可。但仍然有少数lib需要手动编译安装，比如`aom、chromaprint、libzimg、libplacebo` 。

# 完成
上面所有依赖全部安装完便可以开始编译ffmpeg了，一切顺利的话，编译完成后执行`./ffmpeg` 会看到

```bash
ffmpeg version N-111698-g9549712056 Copyright (c) 2000-2023 the FFmpeg developers
  built with gcc 9 (Ubuntu 9.4.0-1ubuntu1~20.04.1)
  configuration: --enable-nonfree --enable-cuda-nvcc --enable-libnpp 
--extra-cflags=-I/usr/local/cuda/include 
--extra-ldflags=-L/usr/local/cuda/lib64 
--incdir=/usr/include/x86_64-linux-gnu 
--libdir=/usr/lib/x86_64-linux-gnu 
--disable-static --enable-shared --enable-cuvid --enable-decoder=aac 
--enable-decoder=h264 --enable-decoder=h264_cuvid --enable-demuxer=mov 
--enable-filter=scale --enable-gnutls --enable-gpl --enable-libass 
--enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus 
--enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 
--enable-nonfree --enable-nvdec --enable-nvenc --enable-pic 
--enable-protocol=file --enable-protocol=https --enable-vaapi 
--enable-libplacebo --enable-vulkan --enable-ladspa --enable-libaom 
--enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio 
--enable-libcodec2 --enable-libflite --enable-libfontconfig --enable-libfribidi 
--enable-libzimg --enable-libgme --enable-libgsm --enable-libjack 
--enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libpulse 
--enable-librsvg --enable-librubberband --enable-libshine --enable-libsnappy 
--enable-libsoxr --enable-libspeex --enable-libssh --enable-libtwolame 
--enable-libvidstab --enable-libwebp --enable-libx265 --enable-libxml2 
--enable-libxvid --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx 
--enable-openal --enable-opencl --enable-opengl --enable-sdl2 
--enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-frei0r
```
最后附上一键安装脚本的[仓库地址](https://github.com/zshnb/ffmpeg-gpu-compile-guide)，有需要可以自取
