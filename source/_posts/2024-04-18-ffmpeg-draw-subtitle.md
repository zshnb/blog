---
title: ffmpeg拼接图片视频，添加文本出现的问题
date: 2024-04-18 11:30:50
tags:
  - Ffmpeg
---

最近在开发一个图片生成短视频的程序，最开始的需求就是需要把视频拼接成图片，每个图片有一定播放时长，这个命令非常简单，通过使用ffmpeg的concat demuxer就可以了。首先创建如下txt文件
<!--more-->

```
file './1.png'
duration 3
file './2.png'
duration 5

```
然后执行

```
ffmpeg -f concat -safe 0 -i filelist.txt -pix_fmt yuv420p -c:v libx264 output.mp4

```
一切非常顺利，点开播放后发现，最后一张图片居然只出现了一帧就结束了，表现为output.mp4的时长和filelist.txt里duration的总和对不上。

![](img1.webp)
![](img2.webp)
视频时长正好少了最后一张图的duration。谷歌搜索了一下，发现同样有人出现过这个问题，从[这个回答中](https://video.stackexchange.com/questions/20588/ffmpeg-flash-frames-last-still-image-in-concat-sequence)找到了解决办法，就是在filelist.txt里把最后一张图的路径重复一份放在末尾，这样出来的图片时长就是正确的了。



![](img3.webp)
## 加字幕
接着需要在视频上添加字幕，按照ffmpeg的文档使用以下命令

```
ffmpeg -i ./tmp/6ir0TW/concat.mp4 -vf "drawtext=text='hello':fontfile=/path/to/your/font.ttf:fontsize=30:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2:enable='between(t,0,3)',drawtext=text='world':fontfile=/path/to/your/font.ttf:fontsize=30:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2:enable='between(t,3,6)'" -codec:a copy ./tmp/6ir0TW/draw_text.mp4

```
添加文本参数中指定hello出现在0-3秒，结果生成的视频hello在第7秒才消失，同时这张图片设置的时长也是7秒，

![](img4.webp)
最初怀疑是drawtext filter的参数设置问题，于是下载了一个正常视频测试，发现文字正常出现和消失。于是怀疑是用concat图片生成的视频的问题。结合正常视频的情况，于是尝试了一下对拼接后的视频进行重新转码再添加文本，一切正常了。

接着我又去video forum[提问](https://forum.videohelp.com/threads/414170-ffmpeg-use-drawtext-filter-text-doesn-t-appear-correctly-in-given-time#post2731480)，有人说这个是因为用图片拼接的视频和正常视频的granularity不一样，所以导致时间没法对齐，然后他给了另一个解决方案是，在video filter里加fps=25。
