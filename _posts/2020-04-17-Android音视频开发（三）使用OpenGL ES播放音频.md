---
layout:     post
title:      Android音视频开发（三）使用OpenGL ES播放音频
subtitle:   
date:       2020-04-17
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android
    - OpenGL ES
    - 音视频
---

**1. OpenSL ES 是什么？**

OpenSL ES 全称是：Open Sound Library for Embedded Systems，是一套无授权费、跨平台、针对嵌入式系统精心优化的硬件音频加速API。它为嵌入式移动多媒体设备上的本地应用程序开发者提供标准化、 高性能,、低响应时间的音频功能实现方法。

**2. 使用流程**

引入头文件：

```
#include <SLES/OpenSLES.h>
#include <SLES/OpenSLES_Android.h>
```

添加依赖库：

```
LOCAL_LDLIBS += -lOpenSLES
```

1）创建一个引擎对象接口。引擎对象是OpenSL ES提供API的唯一入口，调用siCreateEngine来获取SLObjectItf类型的引擎对象接口。

```
SLObjectItf engineObject;
slCreateEngine( &engineObject, 0, nullptr, 0, nullptr, nullptr);
```

2）实例化引擎对象

```
(*engineObject)->Realize(engineObject, SL_BOOLEAN_FALSE);
```

3）获取引擎对象的接口

```
SLEngineItf engineEngine;
(*engineObject)->GetInterface(engineObject, SL_IID_ENGINE, &(engineEngine));
```

4）创建需要的对象接口

通过调用SLEngineltf类型的对象接口的Createxxx方法返回新的对象的接口，比如调用CreateOutputMix方法来获取一个outputMixObject接口，或者调用CreateAudioPlayer方法来获取一个audioPlayerObject接口。

例如创建outputMixObject的接口代码：

```
SLobjectltf outputMixObject；
(*engineEngine)->CreateOutputMix(engineEngine, &outputMixObject, 0, 0, 0)；
```

5）实例化新的对象，任何对象接口获取出来之后，都必须要实例化：

```
realizeObject(outputMixObject);
realizeObject(audioPlayerObject);
```

6）对于某些比较复杂的对象，需要获取新的接口来访问对象的状态或者维护对象的状态，比如在播

放器AudioPlayer或录音器AudioRecorder中注册一些回调方法等，代码如下：

```
SLPlayltf audioPlayerPlay；
(*audioPlayerobject)->GetInterface(audioPlayerObject, SL_IID_PLAY, &audioPlayerPlay)；
//设置播放状态
(*audioplayerPlay)->SetPlaystate(audioPlayerPlay, SL_PLAYSTATE_PLAYING);
//设置暂件状态
(*audioPlayerPlay)->SetPlayState(audioPlayerPlay, SL_PLAYSTATE_PAUSED);
```

7）使用完之后，调用Destroy方法来销毁对象以及相关的资源

```
(*audioPlayerobject)->Destroy(audioPlayerobject);
(*outputMixObject)->Destroy(outputMixObject);
(*engineObject)->Destroy(engineObject);
```

**3. 实例Demo**

本文的Demo代码，使用OpensL  ES实现一个播放媒体文件的功能，下载地址如下：

[下载地址]()

下载运行项目，进入播放界面，点击play开始播放，点击stop停止播放。