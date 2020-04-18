---
layout:     post
title:      FFmepg音视频开发（二）如何使用FFmepg命令行
subtitle:   
date:       2020-04-18
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - FFmepg
    - 音视频
    - 命令行
---

## 1. ffprobe

​	fprobe是用于探测媒体文件的格式以及详细信息。

```
ffprobe 1.mp3
```

```
ffprobe 1.mp4
```

```
ffprobe -show_frames 1.mp4
```

```
ffprobe -show_packets 1.mp4
```

## 2. ffplay

ffplay是一个播放媒体文件的工具。

```
ffplay 1.mp3
```

```
ffplay dump.pcm -f s16le -channels 2 -ar 44100
```

```
ffplay 1.mp4 -sync auido or video or ext
```

## 3. ffmpeg

ffmpeg是一个强大的媒体文件转换工具。它可以转换任何格式的媒体文件，并且还可以用自己的AudioFilter以及VideoFilter进行处理和编辑，总之一句话，有了它，进行离线处理视频时可以做任何你想做的事情了。下面先介绍总体的参数，然后再列出经典场景下的使用案例。

- 通用参数

  - -f fmt：指定格式（音频或者视频格式）。

  - -i filename：指定输人文件名，在Linux下也能指定:0.0 （屏幕录制）或摄像头。

  - -y 覆盖已有文件。

  - -t duration：指定时长。

  - -fs limit size：设置文件大小的上限。

  - -ss time-off：从指定的时间（秒）开始，也支持[-]hh:mm:ss[.xxx]的格式。

  - -re  ：代表按照帧率发送，推流时一定要加入该参数，否则ffmpeg会按照最高速率向流媒体服务器不停地发送数据。

  - -map：指定输出文件的流映射关系。例如：“-map 1:0-map 1:1”要求将第二个输入文件的第一个流

    和第二个流写入输出文件。如果没有-map选项，则fmpeg采用默认的映射关系。

- 视频参数
  - -b：指定比特率（bitls），ffmpeg是自动使用VBR的，若指定了该参数则使用平均比特率。
  - -bitexact：使用标准比特率。
  - -vb：指定视频比特率（bits/s）。
  - -r rate：帧速率（fps）。
  - -s size：指定分辨率（320 x 240 ）。
  - aspect aspect：设置视频长宽比（4:3, 16:9或1.3333，1.7777）。
  - -croptop  size：设置顶部切除尺寸（in pixels）。
  - -cropbottom size：设置底部切除尺寸（in pixels）。
  - -cropleft  size：设置左切除尺寸（in pixels）。
  - -cropright size：设置右切除尺寸（in pixels）。
  - -padtop size：设置顶部补齐尺寸（in pixels）。
  - -padbottom size：底补齐（in  pixels）。
  - -padleft size：左补齐（in  pixels）。
  - -padright size：右补齐（in  pixels）。
  - -padcolor color：补齐带颜色。
  - -vn：取消视频的输出。
  - -vcodec codec：强制使用codec编解码方式
- 音频参数
  - -ab：设置比特率（单位为bit/s）
  - -aq quality：设置音频质量（指定编码）。
  - -ar rate：设置音频采样率（单位为Hz）
  - -ac channels：设置声道数， 1就是单声道， 2就是立体声。
  - -an：取消音频轨。
  - -acodec codec：指定音频编码
  - -vol volume：设置录制音量大小（默认为256） <百分比>。

下面结合日常开发中遇到的场景逐个给出具体的实例来实践一下。

1）列出fimpeg支持的所有格式

```
ffmpeg -formats
```

2）剪切一段媒体文件，可以是音频或者视频文件：

```
ffmpeg-i input.mp4 -ss 00:00:50.0 -codec copy -t 20  output.mp4
```

表示将文件input.mp4从第50s开始剪切20s的时间，输出到文件output.mp4中，其中-ss指定偏移

时间（time Offset），-t指定的时长（duration）

3）如果录制了一个时间比较长的视频无法分享到微信，可以将该视频文件切割为多个文件：

```
ffmpeq-i input.mp4 -t 00:00:50 -c copy small-1.mp4 -ss 00:00:50 -codec copy small-2.mp4
```

4）提取一个视频文件中的音频文件：

```
ffmpeg-i input.mp4 -vn -acodec copy output.m4a
```

5）使一个视频中的音频静音，即只保留视频：

```
ffmpeg-i input.mp4 -an -vcodec copy output.mp4
```

6）从MP4文件中抽取视频流导出为裸H264数据：

```
ffmpeg -i output.mp4 -an -vcodec copy -bsf:v h264-mp4toannexb  output.h264
```

视频数据使用mp4toannexb这个bitstreamfilter转换为原始的H264数据

7）使用AAC音额数据和H264的视频生成MP4文件：

```
ffmpeg -i test.aac -i test.h264 -acodec copy -bsf:a aac_adtstoasc -vcodec copy -f mp4 output.mp4
```

上述代码中使用了一个名为aac_adtstoasc的bistream filter，因为AAC格式也有两种封装格式。

8）对音额文件的编码格式做转换

```
ffmpeg-i input.wav -acodec libfak_aac output.aac
```

9）从WAV音频文件中导出PCM课数据：

```
ffmpeg-i input.wav -acodec pcm_sl6le -f s16le output.pcm
```

导出用16个bit来表示一个sample的PCM数据，并且每个sample的字节排列顺序都是小端。

10）重新编码视频文件，复制音频流，同时封装到MP4格式的文件中

```
ffmpeg-i input.flv -vcodec libx264 -acodec copy output.mp4
```

11）将一个MP4格式的视频转换成为gif格式的动图

```
ffmpeg-i input.mp4 -vf scale=100:-1 -t 5 -r 10 image.gif
```

上述代码按照分辨比例不动宽度改为100（使用VideoFilter的scaleFilter），帧率改为10（-r），只处理前5秒钟（-t）的视频，生成gif

12）将一个视频的画面部分生成图片，比如要分析一个视频里面的每一帧都是什么内容的时候：

```
ffmpeg-i output.mp4 -r 0.25 frames_%04d.png
```

上述命令每4秒钟截取一帧视频画面生成一张图片，生成的图片从frames-0001.png开始递增。

13）使用一组图片可以生成一个gif：

```
ffmpeg -i frames_%04d.png -r 5 output.gif
```

14）使用音量效果器，可以改变一个音频媒体文件中的音量：

```
ffmpeg -i input.wav -af 'volume=0.5'  output.wav
```

上述命令是将input.wav中的声音减小一半，输出到output.wav文件中。

15）淡入效果器的使用：

```
ffmpeg -i input.wav -filter-complex afade=t=in:ss=0：d=5 output.wav
```

上述命令可以将input.wav文件中的前5s做一个淡入效果，输出到output.wav中，可以在Audacity中对比波形图。

16）淡出效果器的使用：

```
ffmpeg-i input.wav -filter-complex afade=t=out:st=200:d=5 output.wav
```

上述命令可以将input.wav文件从200s开始，做5s的淡出效果，并放到output.wav文中。

17）将两路声音进行合并，比如要给一段声音加上背景音乐：

```
ffmpeg -i vocal.wav -i accompany.wav -filter_complex amix=inputs=2:duration=shortest output.wav
```

上述命令是将vocal.wav和accompany.wav两个文件进行mix，按照时间长度较短的时间长度作为最终输出的output.wav的时间长度.

18）声音变速不变调处理

```
ffmpeg-i vocal.wav -fltercomplex atempo=0.5 output.wav
```

上述命令是将vocal.wav按照0.5倍的速度进行处理生成output.wav，时间长度将会变为输入的2倍。但是音高是不变的，这就是大家常说的变速不变调。

19）为视频增加水印效果

```
ffmpeg -i input.mp4 -i icon.png -filter_complex '[0：v][1:v]overlay=main_w-overlay_w-10:10:1[out]' -map'[out]' output.mp4
```

上述命令包含了几个内置参数， main_w代表主视频宽度， overlay_w代表水印宽度，main_h代表主视频高度，

overlay_h代表水印高度

20）视频提亮效果器的使用：

```
ffmpeg -i input.flv -c:v 1ibx264 -b:v 800k -c:a libfdkaac -vf eg=brightness=0.25 -f mp4 output.mp4
```

提亮参数是brightness，取值范围是从-1.0到1.0，默认值是0

21）为视频增加对比度效果：

```
ffmpeg-i input.flv -c:v libx264 -b:v 800k -c:a libfdk_aac -vf eq=contrast=1.5 -f mp4 output.mp4
```

对比度参数是contrast，取值范围是从-2.0到2.0，默认值是1.0

22）视频旋转效果器的使用：

```
ffmpeg -i input.mp4 -vf "transpose=1" -b:v 600k output.mp4
```

23）视频裁剪效果器的使用：

```
ffmpeg-i input.mp4 -an -vf “crop=240:480:120:0” -vcodec libx264 -b:v 600k output.mp4
```

24）将一张RGBA格式表示的数据转换为JPEG格式的图片：

```
ffmpeg -f rawvideo -pix_fmt raba -s 480*480 -i texture.rab -f image2 -vcodec mjpeg output.jpg
```

25）将一个YUV格式表示的数据转换为JPEG格式的图片

```
ffimpeg -f rawvideo -pix_fmt yuv420p -s 480*480 -i texture.yuv -f image2 -vcodec mjpeg output.jpg
```

26）将一段视频推送到流媒体服务器上：

```
ffmpeg -re -i input.mp4 -acodec copy -vcodec copy -f flv  rtmp://xxx
```

上述代码中， rtmp://xxx代表流媒体服务器的地址，加上-re参数代表将实际媒体文件的播放速度作为推

流速度进行推送。

27）将流媒体服务器上的流dump到本地：

```
ffmpeg -i http://xxx/xxx.flv -acodec copy -vcodec copy -f flv output.flv
```

上述代码中http://xxxx代表一个可以访问的视频网络地址，将文件下载到本地文件中

28）将两个音频文件以两路流的形式封装到一个文件中，比如在K歌的应用场景中，原伴唱实时切换的场

景下，可以使用一个文件包含两路流，一路是伴奏流，另外一路是原唱流：

```
ffmpeg -i 131.mp3 -i 134.mp3 -map o:a -c:a:0 libfdk_aac -b:a:0 96k -map 1:a -c:a:1

libfdk_aac -b:a:1 64k -vn -f mp4 output.m4a
```

## 4. 总结

只要是FFmpeg框架能够实现的功能，那么FFmpeg命令行工具就能将其提供出来了。下面将会介绍如何从API层面调用FFmpeg来完成日常的开发工作。