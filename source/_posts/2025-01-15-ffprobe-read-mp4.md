---
title: 如何使用ffprobe读取mp4片段文件
date: 2025-01-15 16:22:07
tags:
  - Ffmpeg
---

最近在实现一个功能，希望能快速获取mp4文件的元数据，普通做法是把把整个视频发到后端，通过ffprobe读取，但这种方式太消耗资源，于是希望能上传一部分mp4的二进制数据给后端，快速完成解析。

在实现过程中发现ffprobe读取某些mp4的切片文件时会报错`moov atom not found` 。查阅资料得知，mp4的metadata结构长下面这样
<!--more-->
![](1.webp)
moov部分就是metadata（想要看更详细的format信息可以看[CFFMediaFormat](https://www.uvcentral.com/files/CFFMediaFormat-2_1.pdf)和[Apple文档](https://developer.apple.com/library/archive/documentation/QuickTime/QTFF/QTFFChap2/qtff2.html#//apple_ref/doc/uid/TP40000939-CH204-56313)）。这部分既可以在文件开头，也可以在文件末尾，通过[atomicparsley](https://atomicparsley.sourceforge.net/) 查看两种不同形式的mp4文件。

|![moov在开头](2.webp)|![moov在末尾](3.webp)|

上面两张图对比了一下正常文件和报错文件的结构，确实发现报错文件的moov在末尾，因此ffprobe读取不到。用ffmpeg尝试了一下手动把moov移动到文件开头，再使用ffprobe解析文件前1M，发现便可以了，所以问题就是出在mp4文件的moov文件位置。

于是突发奇想，能不能把后面moov的部分slice后拼到前面ftyp后面让ffprobe读取，尝试了一下发现还是不行，ffprobe还是会把moov部分当作mdat读取，而且结构还是坏的，似乎进入了一条死胡同。想了许久后想到，mp4文件也是个二进制文件，如果ffprobe无法识别，何不自己直接读取二进制识别，于是又搜索mp4文件moov块的详细分布，最后找到一篇文章，[链接在这](https://www.cimarronsystems.com/wp-content/uploads/2017/04/Elements-of-the-H.264-VideoAAC-Audio-MP4-Movie-v2_0.pdf)。里面详细说明了moov的二进制描述，于是本地尝试了一下，用hexdump读取拼接的mp4文件，内容如下

![](4.webp)
结合下面的mp4时长计算公式

Time scale

A time value that indicates the time scale for this movie—that
is, the number of time units that pass per second in its time coordinate
system. A time coordinate system that measures time in sixtieths of a
second, for example, has a time scale of 60.

Duration

A time value that indicates the duration of the movie in time
scale units. Note that this property is derived from the movie’s tracks.
The value of this field corresponds to the duration of the longest
track in the movie.

duration需要根据time scale换算

可以看到红圈标着的就是moov块的开始标记，根据上面文章的说明，蓝色光标处的00 26 23 e7便是duration信息，换算成十进制是2499559，单位毫秒。看来直接读取二进制的路是可以走通的，但显然自己读取很不靠谱，毕竟mp4文件格式还是挺复杂的，于是又开始搜索mp4文件解析之类的库，最后找到了[https://github.com/zdong22/mp4reader](https://github.com/zdong22/mp4reader)，支持解析拼接的mp4文件，且只有单文件，无依赖，接入友好。输出效果如下

![](5.webp)