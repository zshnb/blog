---
title: 使用ffmpeg转码HDR视频为SDR
date: 2025-09-13 18:21:49
tags:
  - Ffmpeg
---

# 背景
最近有用户反馈平台上产出的clip色彩表现和原视频不一致，下载了原片和clip，对比图如下
<!--more-->
|![](1.webp)|![](2.webp)|

通过ffprobe查看，发现原片的属性是
Stream #0:0[0x1](https://www.notion.so/zhengsihua/und): Video: hevc (Main 10) (hvc1 / 0x31637668), **yuv420p10le(tv, bt2020nc/bt2020/arib-std-b67)**, 1920x1080, 13043 kb/s, 60 fps, 60 tbr, 600 tbn (default)，这里稍微解释一下属性的含义，yuv420p表示yuv色彩空间，三元素比例是4:2:0，10le表示用10bit的颜色位深，bt2020nc表示bt2020色域，通过下图可以更直观地看出参数含义。

|![](3.webp)|![](4.webp)|

简单来说yuv420p10le bt2020的视频属于HDR视频，而目前大部分在线媒体，像YouTube，以及比较旧的播放器都不支持HDR视频，播放的都是SDR（yuv420p bt709）视频，因此需要把用户的HDR视频经过转码变成SDR视频，同时还要使用GPU，加速转码过程。

# 解决
首先在新服务器上尝试使用ffmpeg with gpu命令转码问题视频，报错

Provided device doesn't support required NVENC features。

经过一番搜索，从[这篇帖子](http://mplayerhq.hu/pipermail/ffmpeg-user/2019-September/045318.html)里得到一些启发，大意是GPU decode 10bit的h265 frame，无法直接被GPU h264 encoder接收，中间需要有个改变format的filter，按着这个思路排查搜索，最终尝试`ffmpeg -y -hwaccel cuda -hwaccel_output_format cuda -i UPL_1vKBpdd4iX.mp4 -vf hwdownload,format=p010le,format=nv12,hwupload,scale_npp=format=yuv420p -c:v h264_nvenc output.mp4` 可以成功转码，于是转码的问题搞定了，接着要解决colorspace的问题。

一开始直接搜索ffmpeg change colorspace，从[这里](https://www.reddit.com/r/ffmpeg/comments/zzzmln/yuv420ptv_bt709_progressive/)得到了`-pix_fmt:v yuv420p -colorspace:v 'bt709' -color_primaries:v 'bt709' -color_trc:v 'bt709' -color_range:v 'tv'` 尝试运行后发现色彩还是不对。于是在video forum发了一个[帖子](https://forum.videohelp.com/threads/410646-ffmpeg-colorspace-bt2020-convert-to-bt709-the-color-is-not-correct#post2700413)，里面有人回复到使用`-vf zscale=t=linear:npl=100,format=gbrpf32le,zscale=p= 709,tonemap=hable:desat=2,zscale=t=709:m=709:r=tv, format=yuv420p` filter可以改变视频色域。zscale是一个第三方的filter，需要自己编译ffmpeg时额外加上依赖开启，于是自己在服务器上编译了一个带zscale filter的最新版ffmpeg（由于本文使用的ffmpeg需要非常多额外库，因此使用的是本地编译版本，关于编译ffmpeg的文章请看[这里](/e9a65c437e5b459d8636d4dcb1c281e2?pvs=25)），尝试效果如下，

|![原片](5.webp)|![转换后的](6.webp)|

可以看出颜色确实对了，但是有点偏暗。更严重的是编码速度只有0.x，于是尝试使用GPU加速，尝试了`./ffmpeg -y -vsync 0 -hwaccel cuda -hwaccel_output_format cuda -i UPL_1vKBpdd4iX_slice.mp4 -vf hwdownload,format=p010le,format=nv12,hwupload,scale_npp=format=yuv420p,zscale=t=linear:npl=100,format=gbrpf32le,zscale=p=709,tonemap=hable:desat=2,zscale=t=709:m=709:r=tv,format=yuv420p -c:v h264_nvenc output.mp4` 但是这条命令会报错Impossible to convert between the formats supported by the filter 'Parsed_scale_npp_4' and the filter 'auto_scale_1’，于是在帖子下面继续提问，然后就有人回复说zscale filter不支持GPU加速，这个方式pass。

上面发的帖子里，有个人回复建议搜索HDR to SDR with tonemap。于是我又加上了GPU关键字进行搜索，经过一番搜索，从[这里](https://forum.doom9.org/showthread.php?t=181409)找到了这条命令`./ffmpeg -y -vsync 0 -hwaccel cuda -init_hw_device opencl=ocl -filter_hw_device ocl -threads 16 -i UPL_1vKBpdd4iX_slice.mp4 -vf "format=p010,hwupload,tonemap_opencl=tonemap=hable:desat=0:r=tv:p=bt709:t=bt709:m=bt709:format=nv12,hwdownload,format=nv12" -c:a copy -c:v h264_nvenc output.slice.hable.mp4`

这条命令跑出来的效果如下

![](7.webp)
可以看到效果比zscale还好，而且速度在2.x左右，`tonemap_opencl`是ffmpeg自带的filter，从[文档](https://ffmpeg.org/ffmpeg-filters.html#Tone-mapping)里可以看到有许多tonemapping参数可以配置，经过测试后发现hable的色彩效果是最好的，只是亮度还是稍微有点暗。

在搜索过程中发现了一个[帖子](https://github.com/nilaoda/Blog/discussions/56)，里面提到了一个叫做libplacebo的filter，可以用来做杜比视界的转码，查看[官方链接](https://code.videolan.org/videolan/libplacebo)发现，它是一个支持GPU加速的渲染依赖，支持高质量编码分辨率， HDR映射等。于是赶紧按照官方文档教程安装，配置好[vulkan](https://vulkan.lunarg.com/sdk/home)，[设置vk.xml](https://vulkan.lunarg.com/issue/home?limit=10;q=;mine=false;org=false;khronos=false;lunarg=false;indie=false;status=new,open)，安装完毕后测试命令`./ffmpeg -y -init_hw_device vulkan=vulkan -filter_hw_device vulkan -i UPL_0UOVszENQF.mp4 -vf "hwupload,libplacebo=format=yuv420p10le:colorspace=bt709:color_primaries=bt709:color_trc=bt709:format=yuv420p" -c:v libx264 -c:a copy UPL_0UOVszENQF.libplacebo.mp4` 效果如下

![](8.webp)
可以看到效果比tonemap还要好，但是速度还是很慢，只有0.x。于是尝试使用GPU加速`./ffmpeg -y -hwaccel cuda -hwaccel_output_format cuda -extra_hw_frames 3 -i UPL_0UOVszENQF.mp4 -vf "hwupload=derive_device=vulkan,libplacebo=format=yuv420p10le:colorspace=bt709:color_primaries=bt709:color_trc=bt709,hwdownload,format=yuv420p" -c:v h264_nvenc -c:a copy UPL_0UOVszENQF.libplacebo.cuda.mp4` 会报错

Impossible to convert between the formats supported by the filter 'Parsed_hwupload_0' and the filter 'auto_scale_0’。

尝试按照关键字libplacebo CUDA，以及错误信息搜索，从这个[issue](https://github.com/jellyfin/jellyfin-ffmpeg/issues/215)里看到一条回复

![](9.webp)
于是尝试把libplacebo降级编译，但是编译过程中报错与vulkan版本不兼容，于是尝试降级vulkan，再次编译，又报错ffmpeg不支持低版本vulkan，于是继续降级ffmpeg，结果ffmpeg编译又报错找不到nvenc。没办法，只好向libplacebo作者开[issue](https://github.com/haasn/libplacebo/issues/187)提问。作者表示需要等他的GPU设备到了才能测试，同时让我尝试6.292.1版本的libplacebo，并提供执行过程日志给他。

于是又开始尝试编译master分支的ffmpeg和6.292.1的libplacebo，经过测试还是一模一样的错误。

接着下面有一位大佬回复道

> There is a problem with the descriptor buffer on NVIDIA, for ffmpeg master you must disable multiplane, also you should use nvdec instead of cuda and use hwupload at the end of the filter chain to avoid pointless copy operations.
>

同时提供了如下命令

`./ffmpeg -y -init_hw_device vulkan=vk,disable_multiplane=1 -filter_hw_device vk -hwaccel nvdec -hwaccel_output_format cuda -extra_hw_frames 3 -i UPL_0UOVszENQF.mp4 -vf "hwupload=derive_device=vulkan,libplacebo=format=yuv420p10le:colorspace=bt709:color_primaries=bt709:color_trc=bt709,hwupload=derive_device=cuda" -c:v h264_nvenc -c:a copy UPL_0UOVszENQF.libplacebo.cuda.mp4`

很可惜这条命令会报错

[h264_nvenc @ 0x55b770cbbac0] 10 bit encode not supported
[h264_nvenc @ 0x55b770cbbac0] Provided device doesn't support required NVENC features
[vost#0:0/h264_nvenc @ 0x55b770c6a300] Error while opening encoder - maybe incorrect parameters such as bit_rate, rate, width or height.
Error while filtering: Function not implemented
[out#0/mp4 @ 0x55b770cbc4c0] Nothing was written into output file, because at least one of its streams received no packets

有了前面CUDA转码的排查经验后，从报错信息里很快发现了问题所在，又是因为nvenc不支持10bit的frame，于是把libplacebo的format参数改成yuv420p，执行成功！同时速度在10x左右，非常快。

![](10.webp)
效果如下

![](11.webp)
可以看到色彩饱和度和原片非常接近，亮度基本和原片一致。

# 总结
为了解决colorspace的问题，排查摸索了大概3天左右，最终得到如下几种解决方案

1. `./ffmpeg -y -vsync 0 -hwaccel cuda -hwaccel_output_format cuda -i UPL_1vKBpdd4iX_slice.mp4 -vf hwdownload,format=p010le,format=nv12,hwupload,scale_npp=format=yuv420p,zscale=t=linear:npl=100,format=gbrpf32le,zscale=p=709,tonemap=hable:desat=2,zscale=t=709:m=709:r=tv,format=yuv420p -c:v h264_nvenc output.mp4` 速度慢，色彩还可以，亮度暗

2. `./ffmpeg -y -vsync 0 -hwaccel cuda -init_hw_device opencl=ocl -filter_hw_device ocl -threads 16 -i UPL_1vKBpdd4iX_slice.mp4 -vf "format=p010,hwupload,tonemap_opencl=tonemap=hable:desat=0:r=tv:p=bt709:t=bt709:m=bt709:format=nv12,hwdownload,format=nv12" -c:a copy -c:v h264_nvenc output.slice.hable.mp4` 速度稍快，色彩不错，亮度暗

3. `./ffmpeg -y -init_hw_device vulkan=vulkan -filter_hw_device vulkan -i UPL_0UOVszENQF.mp4 -vf "hwupload,libplacebo=format=yuv420p10le:colorspace=bt709:color_primaries=bt709:color_trc=bt709:format=yuv420p" -c:v libx264 -c:a copy UPL_0UOVszENQF.libplacebo.mp4` 速度慢，色彩好，亮度亮

4. `./ffmpeg -y -init_hw_device vulkan=vk,disable_multiplane=1 -filter_hw_device vk -hwaccel nvdec -hwaccel_output_format cuda -i UPL_0UOVszENQF.mp4 -vf "hwupload=derive_device=vulkan,libplacebo=format=yuv420p:colorspace=bt709:color_primaries=bt709:color_trc=bt709,hwupload=derive_device=cuda" -preset:v fast -tune:v hq -rc:v vbr -cq:v 22 -b:v 0 -c:v h264_nvenc -c:a copy UPL_0UOVszENQF.libplacebo.cuda.mp4` 速度快，色彩好，亮度亮，最终方案

# Quote
- [https://www.vdr-portal.de/forum/index.php?thread/135567-softhdvaapi-libplacebo-et-al/&pageNo=2](https://www.vdr-portal.de/forum/index.php?thread/135567-softhdvaapi-libplacebo-et-al/&pageNo=2)

- [https://web.archive.org/web/20180817195640/https://stevens.li/guides/video/converting-hdr-to-sdr-with-ffmpeg/](https://web.archive.org/web/20180817195640/https://stevens.li/guides/video/converting-hdr-to-sdr-with-ffmpeg/)

- [https://forum.doom9.org/showthread.php?t=175125&page=2](https://forum.doom9.org/showthread.php?t=175125&page=2)

- [https://www.reddit.com/r/ffmpeg/comments/q2ykjd/colorspace_help_please/](https://www.reddit.com/r/ffmpeg/comments/q2ykjd/colorspace_help_please/)

- [https://emby.media/community/index.php?/topic/107429-nvdec-h265-decoding-not-working-but-cuvid-does/](https://emby.media/community/index.php?/topic/107429-nvdec-h265-decoding-not-working-but-cuvid-does/)

- [https://forum.doom9.org/showthread.php?t=168814&page=183](https://forum.doom9.org/showthread.php?t=168814&page=183)

- [https://forum.doom9.org/showthread.php?p=1866613](https://forum.doom9.org/showthread.php?p=1866613)

- [https://forum.videohelp.com/threads/384494-Modify-Color-Space-Flag/page2](https://forum.videohelp.com/threads/384494-Modify-Color-Space-Flag/page2)
