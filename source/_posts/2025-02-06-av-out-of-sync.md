---
title: Ffmpeg提取的音视频不同步的问题
date: 2025-02-06 22:57:13
tags:
  - Ffmpeg
---

# 背景
最近有用户反馈，上传的视频经过处理后，音频和字幕跟视频对不上的问题，研究了几个视频后发现，有以下几种情况
<!--more-->
1. 提取的音频时长短于视频
   ![视频时长](img1.webp)
   ![音频时长](img2.webp)

2. 提取的音频时长没有问题，但是中间几秒会突然出现音频提早1s的情况

# 思路
一开始是直接搜索ffmpeg extract audio not same as video，看了几个stackoverflow的回答，以及其他论坛的回答，都是说加-async 1，但尝试过发现-async 1无法结果问题，实际上可以解决case2，但无法解决case1。

看了好多解决方法后，有个人说可以用apad这个audio filter，于是接着搜索apad是啥，发现apad是给音频填充数据包用的，于是有了一个新思路，可能是因为视频文件里的音轨存在空数据包，而提取的音轨会丢掉这些数据包，所以导致提取音频时长不一致，于是搜索ffmpeg extract audio with fill gap，找到了

- [https://superuser.com/questions/1552916/audio-is-not-in-sync-after-re-encoding-with-ffmpeg-of-video-and-changing-audio-f](https://superuser.com/questions/1552916/audio-is-not-in-sync-after-re-encoding-with-ffmpeg-of-video-and-changing-audio-f)

- [https://forum.videohelp.com/threads/377547-FFMPEG-how-to-fill-the-audio-gap-with-silence](https://forum.videohelp.com/threads/377547-FFMPEG-how-to-fill-the-audio-gap-with-silence)

根据帖子里的思路，尝试了`ffmpeg -i video-raw.video -i input.flac -map 0:v -map 1:a -shortest -af apad -c:a libmp3lame -c:v copy marco3.mp4` 然后再提取音频，结果出来的音频时长一致了，正当我以为解决了问题的时候，扔进播放器一放，音频跟视频还是对不上。之后还想过使用ffmpeg的detectsilence，检测视频中音轨空白部分，然后手动补上，但是检测出来的结果不对，所以也行不通

# 绝处逢生
正当思路陷入死胡同时，我请教了一位音视频专家朋友，他给了我几个建议是

![](img3.webp)
关于pts，dts等概念，可以参考[http://dranger.com/ffmpeg/tutorial05.html](http://dranger.com/ffmpeg/tutorial05.html)

根据上面的建议，可以通过`ffprobe -v error -print_format json -show_format -show_streams video-raw-mul-3.mp4` 拿到轨道的pts信息，问题视频的ffprobe信息如下

|![视频](img4.webp)|![音频](img5.webp)|

抽取的音频的ffprobe信息如下

![](img6.webp)
可以看出，抽取的音频start_pts跟视频音轨的start_pts对不上，看数字不太明显，通过剪映可以更明显地看出延迟

![](img7.webp)
上面视频的音轨柱子明显比下面的后面，于是问题找到了，就该解决了。

# 解决
经过上面的分析，可以知道case1的情况是原视频中的音视频并没有不同步，只是音频的start_pts和视频的不一样，而ffmpeg的-vn参数提取的音频会默认设置为0，这样就导致单独播放音频时，声音与视频画面出现延迟现象，于是我们需要重新设置提取音频的start_pts参数。

经过一阵搜索，从[https://trac.ffmpeg.org/ticket/2938](https://trac.ffmpeg.org/ticket/2938)找到了解决方法，通过audio filter里的`aresample=first_pts=0` 可以重新对音频进行设置first_pts，目标值就是原视频中音轨的start_pts，经过测试后发现视频同步。

上面还有个case2是怎么解决的呢？我去了一个视频论坛提问，帖子在这[https://forum.videohelp.com/threads/410351-audio-is-out-of-sync-after-extract-from-video#post2697182](https://forum.videohelp.com/threads/410351-audio-is-out-of-sync-after-extract-from-video#post2697182)，第一位老哥回复依然是-async 1，但我之前用的case1的视频测试过这个命令，发现不行，所以再没有试过，因为上面已经解决了case1，于是抱着试一试的想法，用-async 1提取了case2的音频，经过测试发现也没有问题。于是

1. 对于音频时长不一致，使用`ffmpeg -i input.mp4 -async 1 -vn -ar aresample=first_pts=$pts audio.flac`解决

2. 对于音频时长一致，播放中间出现不同步，使用`ffmpeg -i input.mp4 -async 1 -vn audio.flac` 解决

为什么async 1对case1不起作用呢，是因为async解决的是原视频的声音不同步，而case1原视频声音并没有不同步，自然不起作用

