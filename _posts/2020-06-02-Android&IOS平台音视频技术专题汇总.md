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
    - IOS
---

工作的一些总结:
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

- [x] [Android音视频开发（一）如何采集音频](https://daimaren.github.io/2020/04/15/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E4%B8%80-%E5%A6%82%E4%BD%95%E9%87%87%E9%9B%86%E9%9F%B3%E9%A2%91/)
- [x] [Android音视频开发（二）如何播放音频](https://daimaren.github.io/2020/04/16/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E4%BA%8C-%E5%A6%82%E4%BD%95%E6%92%AD%E6%94%BE%E9%9F%B3%E9%A2%91/)
- [x] [Android音视频开发（三）使用OpenSL ES播放音频](https://daimaren.github.io/2020/04/17/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E4%B8%89-%E4%BD%BF%E7%94%A8OpenGL-ES%E6%92%AD%E6%94%BE%E9%9F%B3%E9%A2%91/)
- [ ] [Android音视频开发（四）如何使用Camera API采集预览视频 ]()
- [x] [Android音视频开发（五）如何使用MediaCodec硬件编码AAC](https://daimaren.github.io/2020/04/19/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E4%BA%94-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8MediaCodec%E7%A1%AC%E4%BB%B6%E7%BC%96%E7%A0%81AAC/)
- [x] [Android音视频开发（六）如何使用ffmpeg软件编码AAC](https://daimaren.github.io/2020/04/20/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E5%85%AD-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ffmpeg%E8%BD%AF%E4%BB%B6%E7%BC%96%E7%A0%81AAC/)
- [ ] [Android音视频开发（七）如何使用libfdk_aac软件编码AAC]()
- [x] [Android音视频开发（八）如何使用MediaCodec硬件解码AAC](https://daimaren.github.io/2020/04/22/Android%E9%9F%B3%E8%A7%86%E9%A2%91%E5%BC%80%E5%8F%91-%E5%85%AB-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8MediaCodec%E7%A1%AC%E4%BB%B6%E8%A7%A3%E7%A0%81AAC/)
- [ ] [Android音视频开发（九）如何使用ffmpeg软件解码AAC]()
- [ ] [Android音视频开发（十）如何使用libfdk_aac软件解码AAC]()

- [ ] [Android音视频开发（十一）如何使用MediaCodec硬件编码H.264]()

- [ ] [Android音视频开发（十二）如何使用libx264软件编码H.264]()

- [ ] [Android音视频开发（十三）如何使用MediaCodec硬件解码H.264]()

#### 1.2 ExoPlayer 汇总

[多媒体工程师必备技能](https://www.jianshu.com/p/c731515edb9d)<br>
[MediaPlayer 播放器全面剖析(一)](https://www.jianshu.com/p/db7104daa842)<br>
[MediaPlayer 播放器全面剖析(二)](https://www.jianshu.com/p/5513f0bd3dbf)<br>
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

- [x] [OpenGL ES 移动端开发（一）OpenGL ES介绍](https://daimaren.github.io/2020/04/01/OpenGL-ES-%E7%A7%BB%E5%8A%A8%E7%AB%AF%E5%BC%80%E5%8F%91-%E4%B8%80-OpenGL-ES%E4%BB%8B%E7%BB%8D/)
- [x] [OpenGL ES 移动端开发（二）GLSL语法与内建函数](https://daimaren.github.io/2020/04/02/OpenGL-ES-%E7%A7%BB%E5%8A%A8%E7%AB%AF%E5%BC%80%E5%8F%91-%E4%BA%8C-OpenGL-ES%E6%B8%B2%E6%9F%93%E7%AE%A1%E7%BA%BF/)
- [x] [OpenGL ES 移动端开发（三）OpenGL ES渲染管线](https://daimaren.github.io/2020/04/03/OpenGL-ES-%E7%A7%BB%E5%8A%A8%E7%AB%AF%E5%BC%80%E5%8F%91-%E4%B8%89-GLSL%E8%AF%AD%E6%B3%95%E4%B8%8E%E5%86%85%E5%BB%BA%E5%87%BD%E6%95%B0/)
- [x] [OpenGL ES 移动端开发（四）创建第一个Program](https://daimaren.github.io/2020/04/04/OpenGL-ES-%E7%A7%BB%E5%8A%A8%E7%AB%AF%E5%BC%80%E5%8F%91-%E5%9B%9B-%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AAProgram/)

#### 5.IOS音视频开发

- [x] []()
- [ ] 

### 6.多媒体封装格式剖析

- [ ] [多媒体封装格式剖析（一）MP4篇](https://daimaren.github.io/2020/03/01/%E5%A4%9A%E5%AA%92%E4%BD%93%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F%E5%89%96%E6%9E%90-%E4%B8%80-MP4%E7%AF%87/)
- [ ] [多媒体封装格式剖析（二）TS篇](https://daimaren.github.io/2020/03/02/%E5%A4%9A%E5%AA%92%E4%BD%93%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F%E5%89%96%E6%9E%90-%E4%BA%8C-TS%E7%AF%87/)
- [ ] [多媒体封装格式剖析（三）FLV篇](https://daimaren.github.io/2020/03/03/%E5%A4%9A%E5%AA%92%E4%BD%93%E5%B0%81%E8%A3%85%E6%A0%BC%E5%BC%8F%E5%89%96%E6%9E%90-%E4%B8%89-FLV%E7%AF%87/)

### 7.多媒体协议格式剖析

- [ ] [多媒体协议格式剖析（一）M3U8篇](https://daimaren.github.io/2020/03/10/%E5%A4%9A%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE%E6%A0%BC%E5%BC%8F%E5%89%96%E6%9E%90-M3U8%E7%AF%87/)
- [ ] [多媒体协议格式剖析（二）RTMP篇](https://daimaren.github.io/2020/03/11/%E5%A4%9A%E5%AA%92%E4%BD%93%E5%8D%8F%E8%AE%AE%E6%A0%BC%E5%BC%8F%E5%89%96%E6%9E%90-RTMP%E7%AF%87/)

### 8.多媒体编码格式剖析

- [ ] [多媒体编码格式剖析（一）AAC篇]()
- [ ] [多媒体编码格式剖析（二）H.264篇]()

### 9.视频播放疑难杂症排查

- [ ] [视频播放疑难杂症排查（一）如何分析播放失败](https://daimaren.github.io/2020/06/01/%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E7%96%91%E9%9A%BE%E6%9D%82%E7%97%87%E6%8E%92%E6%9F%A5-%E4%B8%80-%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90%E6%92%AD%E6%94%BE%E5%A4%B1%E8%B4%A5/)