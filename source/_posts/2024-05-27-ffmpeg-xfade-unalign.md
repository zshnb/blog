---
title: ffmpeg使用xfade滤镜滑入滑出时间对不齐的问题
date: 2024-05-27 13:57:41
tags:
  - Ffmpeg
---

最近在做图片拼接视频的项目时，需要实现一个功能：在每张图片中间加滑入滑出的过渡效果。ffmpeg有一个[xfade](https://ffmpeg.org/ffmpeg-filters.html#xfade)的filter，支持slideleft/slideright等过渡效果（[在这](https://trac.ffmpeg.org/wiki/Xfade)可以看到所有过渡效果的动图）。
<!--more-->
最开始我尝试使用fade的filter格式应用xfade，类似`-filter_complex “zoompan=xxxx,xfade=xxxx”` 这个命令直接就报错了，搜了一下xfade的语法才知道xfade是需要输入两个视频流，然后输出一个视频流。于是改成了

ffmpeg -t 8.34 -i ./tmp/R70NOq/script_1_image_20240326_211935.png -t 10.219999999999999 -i ./tmp/R70NOq/script_2_image_20240326_211958.png -t 10.46 -i ./tmp/R70NOq/script_3_image_20240326_212013.png -t 8.360000000000003 -i ./tmp/R70NOq/script_4_image_20240326_212028.png -t 8.799999999999997 -i ./tmp/R70NOq/script_5_image_20240326_212045.png -t 8.380000000000003 -i ./tmp/R70NOq/script_6_image_20240326_212059.png -i ./tmp/R70NOq/input.mp3 -filter_complex "[0:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x-10':y='y':d=209[v0];[1:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x+10':y='y':d=255[v1];[2:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':y='if(gte(ih,y),ih,y)-10':x='x':d=264[v2];[3:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':y='y+10':x='x':d=209[v3];[4:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x-10':y='y':d=220[v4];[5:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x+10':y='y':d=210[v5];[v0][v1]xfade=transition=slideleft:duration=0.5:offset=7.84[x0];[v1][v2]xfade=transition=slideleft:duration=0.5:offset=17.55[x1];[v2][v3]xfade=transition=slideleft:duration=0.5:offset=27.51[x2];[v3][v4]xfade=transition=slideleft:duration=0.5:offset=34.38[x3];[v4][v5]xfade=transition=slideleft:duration=0.5:offset=42.68,format=yuv420p[v]" -map "[v]" -map 6 -s 1024x1792 ./tmp/R70NOq/concat2.mp4

这个命令会报错“invalid stream label v1”，搜索了一下找到一个[回答](https://stackoverflow.com/questions/65072355/ffmpeg-invalid-stream-specifier-si)解释说在ffmpeg的filter里，中间阶段的流是不能复用的，一种解决方案是用split把一个流拆成两个同样的流供后面使用，我选择了另一种解决方案，把前一个xfade的输出流和原始输入流作为后一个xfade的输入流，命令如下

ffmpeg -t 8.34 -i ./tmp/R70NOq/script_1_image_20240326_211935.png -t 10.219999999999999 -i ./tmp/R70NOq/script_2_image_20240326_211958.png -t 10.46 -i ./tmp/R70NOq/script_3_image_20240326_212013.png -t 8.360000000000003 -i ./tmp/R70NOq/script_4_image_20240326_212028.png -t 8.799999999999997 -i ./tmp/R70NOq/script_5_image_20240326_212045.png -t 8.380000000000003 -i ./tmp/R70NOq/script_6_image_20240326_212059.png -i ./tmp/R70NOq/input.mp3 -filter_complex "[0:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x-10':y='y':d=209[v0];[1:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x+10':y='y':d=255[v1];[2:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':y='if(gte(ih,y),ih,y)-10':x='x':d=264[v2];[3:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':y='y+10':x='x':d=209[v3];[4:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x-10':y='y':d=220[v4];[5:v]scale=8000x4000,zoompan=z='min(zoom+0.0015,1.5)':x='x+10':y='y':d=210[v5];[v0][v1]xfade=transition=slideleft:duration=0.5:offset=7.84[x0];[x0][v2]xfade=transition=slideleft:duration=0.5:offset=17.55[x1];[x1][v3]xfade=transition=slideleft:duration=0.5:offset=27.51[x2];[x2][v4]xfade=transition=slideleft:duration=0.5:offset=34.38[x3];[x3][v5]xfade=transition=slideleft:duration=0.5:offset=42.68,format=yuv420p[v]" -map "[v]" -map 6 -s 1024x1792 ./tmp/R70NOq/concat2.mp4

可以看到v0和v1作为第一个xfade的输入产生了x0，接着x0和v2输入产生了x1，以此类推。

### 问题
上面的命令生成的视频是可以播放的，但是有一个问题，图片切换的时间点和预期的不一样，期望图片是在指定字幕结束的时间切换，但结果是越后面图片切换的时间点越提前了。这里先给出xfade的offset计算公式：

|**input**|**input duration**|**+**|**previous xfade **|**offset**|**-**|**xfade **|**duration**|**offset**|** =**|
|----|----|----|----|----|----|----|----|----|----|
|`v0.mp4`|4|+|0|-|1|3|
|`v1.mp4`|8|+|3|-|1|10|
|`v2.mp4`|12|+|10|-|1|21|
|`v3.mp4`|5|+|21|-|1|25|

可以看出xfade的计算方式，会导致越后面的片段，offset越提前，如果强行延后offset，会导致剪辑出来的视频出现黑屏，因为xfade要求输入的两个视频流，offset加上duration不能大于第一个视频流的长度。

### 解决
在没有应用xfade时，输入图片的时长都是根据音频字幕的开始结束时间计算出来的。由于xfade的offset会隐性改变输入源的时长，为了让结果视频图片能对的上字幕时间，我们可以手动延长输入源图片的时长，比如第一张图片原本时长在8s，希望在8s结束时出现slideleft，如果直接设置offset=8s会黑屏，所以我们增加图片时长到8.5s，然后设置offset=5:duration=0.5，后面的图片以此类推，这样就能完美让xfade匹配上图片的切换时间。


