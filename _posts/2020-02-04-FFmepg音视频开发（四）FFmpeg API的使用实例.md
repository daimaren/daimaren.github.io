---
layout:     post
title:      FFmepg音视频开发（四）FFmpeg API的使用实例
subtitle:   
date:       2020-02-04
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - FFmepg
    - 音视频
---

**1. 引用头文件**

```
extern "C" {
#include "libavutil/imgutils.h"
#include "libswscale/swscale.h"
#include "libavutil/dict.h"
#include "libswresample/swresample.h"
#include "libavutil/samplefmt.h"
#include "libavutil/timestamp.h"
#include "libavformat/avformat.h"
}
```

**2. 注册协议和编解码器**

```
av_register_all();
```

**3. 打开媒体流，设置超时回调**

```
AVFormatContext *pFormatCtx = avformat_alloc_context();
AVIOInterruptCB int_cb = {interrupt_cb, this};
pFormatCtx->interrupt_callback = int_cb;
avformat_open_input(pFormatCtx, videoSourceURI, NULL, NULL);
avformat_find_stream_info(pFormatCtx, NULL);
```

**4. 查找流并打开对应解码器**

```
//寻找音视频流
for (int i = 0; i < pFormatCtx->nb_streams; i++) {
	AVStream* stream = pFormatCtx->streams[i];
    if (AVMEDIA_TYPE_AUDIO == stream->codec->codec_type) {
        videoStreamIndex = i;
    } else {
        audioStreamIndex = i;
    }
}
//打开音频解码器
AVCodecContext *audioCodecCtx = audioStream->codec;
AVCodec *audioCodec = avcodec_find_decoder(audioCodecCtx->codec_id);
if (audioCodec == NULL) {
    LOGI("can not find the audioStream's Codec ...");
    return -1;
}
avcodec_open2(audioCodecCtx, audioCodec, NULL);
//打开视频解码器
AVCodecContext *videoCodecCtx = videoStream->codec;
AVCodec *videoCodec = avcodec_find_decoder(videoCodecCtx->codec_id);
if (videoCodec == NULL) {
    LOGI("can not find the videoStream's Codec ...");
    return -1;
}
avcodec_open2(videoCodecCtx, videoCodec, NULL);
```

**5. 创建音频转换对象和解码后数据存放对象**

```
SwrContext* swrContext = swr_alloc_set_opts(NULL, av_get_default_channel_layout(audioCodecCtx->channels), AV_SAMPLE_FMT_S16, audioCodecCtx->sample_rate,
        av_get_default_channel_layout(audioCodecCtx->channels), audioCodecCtx->sample_fmt, audioCodecCtx->sample_rate, 0, NULL);
if (!swrContext || swr_init(swrContext)) {
    if (swrContext)
        swr_free(&swrContext);
    LOGI("init resampler failed...");
    return -1;
}
AVFrame* audioFrame = avcodec_alloc_frame();
```

**6. 创建视频转换对象和解码后数据存放对象**

```
AVFrame* videoFrame = avcodec_alloc_frame();
```

**7. 读取流并解码**

```

```

**8. 解码后数据处理**

```

```

**9. 释放所有资源**

```

```





