---
layout:     post
title:      Android&IOS平台音视频技术专题汇总
subtitle:   
date:       2020-06-02
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android
    - IOS分享工作的一些总结:
---

f工作的一些总结:
> * 1.Android 音视频开发
> * 2.FFmepg音视频开发
> * 3.Android Camera 学习指南
> * 4.OpenGL ES 移动端开发
> * 5.IOS音视频开发
> * 6.多媒体封装格式剖析
> * 7.多媒体协议格式剖析
> * 8.多媒体编码格式剖析
> * 9.视频播放疑难杂症排查

### 1.Android 音视频开发

#### 1.1 Android 音视频开发

- [x] [Android音视频开发（一）如何采集音频]()
- [x] [Android音视频开发（二）如何播放音频]()
- [ ] [Android音视频开发（三）使用OpenSL ES播放音频]()
- [ ] [Android音视频开发（四）如何使用Camera API采集预览视频 ]()
- [ ] [Android音视频开发（五）如何使用MediaCodec硬件编码AAC]()
- [ ] [Android音视频开发（六）如何使用ffmpeg软件编码AAC]()
- [ ] [Android音视频开发（七）如何使用libfdk_aac软件编码AAC]()
- [ ] [Android音视频开发（八）如何使用MediaCodec硬件解码AAC]()
- [ ] [Android音视频开发（九）如何使用ffmpeg软件解码AAC]()
- [ ] [Android音视频开发（十）如何使用libfdk_aac软件解码AAC]()

- [ ] [Android音视频开发（十一）如何使用MediaCodec硬件编码H.264]()

- [ ] [Android音视频开发（十二）如何使用libx264软件编码H.264]()

- [ ] [Android音视频开发（十三）如何使用MediaCodec硬件解码H.264]()

#### 1.2 ExoPlayer 汇总

[多媒体工程师必备技能](https://www.jianshu.com/p/c731515edb9d)
[MediaPlayer 播放器全面剖析(一)](https://www.jianshu.com/p/db7104daa842)
[MediaPlayer 播放器全面剖析(二)](https://www.jianshu.com/p/5513f0bd3dbf)
[MediaCodec底层原理剖析](https://www.jianshu.com/p/b7bacabcc019)<br>
[使用MediaCodec 播放视频](https://www.jianshu.com/p/d406314ab63c)<br>
[Android 平台音视频编辑功能 汇总](https://www.jianshu.com/p/cafac2b1c4fe)<br>
[HLS格式解析](https://www.jianshu.com/p/dbac4c041de8)<br>
[Android 平台视频边下边播技术](https://www.jianshu.com/p/27085da32a35)<br>
#### 1.3 ExoPlayer 汇总

[ExoPlayer 学习资料总结](https://www.jianshu.com/p/f48ea1b5708a)<br>
[ExoPlayer 架构剖析](https://www.jianshu.com/p/f506c279e4e5)<br>
[一个ExoPlayer anr引发的思考](https://www.jianshu.com/p/b5dff25409bb)<br>
#### 1.4 ijkplayer 汇总

[ijkplayer 开源项目分析(一)编译](https://www.jianshu.com/p/026d6071514f)<br>
[ijkplayer 开源项目分析(二)播放流程概要](https://www.jianshu.com/p/3fb005f9378b)<br>
[ijkplayer 开源项目分析(三)msg_queue消息机制剖析](https://www.jianshu.com/p/ce6e1ef8dda0)<br>
#### 1.5 VLC 汇总

### 2.FFmepg音视频开发

- [x] [FFmepg音视频开发（一）如何编译FFmepg]()
- [x] [FFmepg音视频开发（二）如何使用FFmepg命令行]()
- [x] [FFmepg音视频开发（三）FFmpeg API的分析]()
- [x] [FFmepg音视频开发（四）FFmpeg API的使用]()

### 3.Android Camera 学习指南
[Android Camera架构](https://www.jianshu.com/p/bac0e72351e4)<br>
[Android Camera原理之编译](https://www.jianshu.com/p/364b4f19ca07)<br>
[Android Camera模块解析之拍照](https://www.jianshu.com/p/bc9e96c7e95e)<br>
[Android Camera模块解析之视频录制](https://www.jianshu.com/p/779c3dc775e9)<br>
[Android Camera原理之openCamera模块(一)](https://www.jianshu.com/p/1332d3864f7c)<br>
[Android Camera原理之openCamera模块(二)](https://www.jianshu.com/p/82d4006e6cef)<br>
[Android Camera进程间通信类总结](https://www.jianshu.com/p/2eb683037379)<br>
[Android Camera原理之createCaptureSession模块](https://www.jianshu.com/p/3d88711a6911)<br>
[Android Camera原理之CameraDeviceCallbacks回调模块](https://www.jianshu.com/p/01c86ae29a6b)<br>
[Android Camera原理之setRepeatingRequest与capture模块](https://www.jianshu.com/p/6c3ca95ccaba)<br>
[Android Camera原理之cameraserver与cameraprovider是怎样联系的](https://www.jianshu.com/p/6dc1ef6df400)<br>
[Android Camera原理之camera service与camera provider session会话与capture request轮转](https://www.jianshu.com/p/c1f75c48ed7c)<br>
[Android Camera原理之camera HAL底层数据结构与类总结](https://www.jianshu.com/p/099cc3b0ab25)<br>
[Android Camera原理之camera service类与接口关系](https://www.jianshu.com/p/f02f2763d5fc)<br>
[Android Camera原理之camera provider启动](https://www.jianshu.com/p/5758f14f924e)<br>
[Android Camera原理之camx hal架构之cam chi](https://www.jianshu.com/p/80de4a6e478c)<br>
[Android Camera原理之camx hal架构](https://www.jianshu.com/p/cfb1da9d4217)<br>
[Android Camera原理之拍照流程zsl优化方案](https://www.jianshu.com/p/3beb7403025f)<br>

### 4.OpenGL ES 移动端开发

[OpenGL ES 移动端开发（一）OpenGL ES介绍]()

[OpenGL ES 移动端开发（二）GLSL语法与内建函数]()

[OpenGL ES 移动端开发（三）OpenGL ES渲染管线]()

[OpenGL ES 移动端开发（四）创建第一个Program]()

#### 5.IOS音视频开发

- [x] []()

### 6.多媒体封装格式剖析

### 7.多媒体协议格式剖析

### 8.多媒体编码格式剖析

### 9.视频播放疑难杂症排查

