---
layout:     post
title:      Android音视频开发（六）如何使用ffmpeg软件编码AAC
subtitle:   libfdk_aac库编译进FFmpeg
date:       2020-04-20
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android
    - 音视频
    - 软编
    - AAC
---

**1. 实现流程**

要使用第三方库libfdk_aac编码AAC文件，必须在做交叉编译的时候将libfdk_aac库编译进FFmpeg。

可编写一个C++的类，命名为audio_encoder，然后对外提供三个接口，分别是初始化、编码以及销毁方法。

首先来看一下初始化接口：

```
int init(int bitRate，int channels, int sampleRate，int bitsPerSample,const char* aacFilePath, const char* codec_name)；
```

​	对于传入的参数，首先是比特率，接着是声道数、采样率，然后是最终编码的文件路径，最后是编码器的名字。其实现步骤具体如下：

​	首先调用方法av_register_all()将所有的封装格式以及编解码器注册到FFmpeg框架中；

然后调用avformat_alloc ouput _context方法传入输出文件格式，分配出上下文，即分配出封装格式；

之后调用avio_open2方法传入AAC的编码路径，相当于打开文件连接通道。

接下来要为AVFormatContext上下文填充AVStream：

```
audiostream = avformat_new_stream(avFormatContext, NULL);
```

现在要为audioStream的Codec属性填充内容了。Codec属性是一个AVCodecContext的结构体类型，需要为该结构体填充如下几个属性，首先是codec type，赋值为AVMEDIA_TYPE_AUDIO，代表其是音频类型：其次是bit rate，samplerate， channels等基本属性，然后是channel_layout ，最后也是最重要的sample fmt，代表了如何数字化表示采样，使用的是AV_SAMPLE_FMT_S16，即用一个short来表示一个采样点，这样就把AVCodecContext结构体构造完成了。

​	下面要准备最重要的编码器了，通过调用avcodec_find_encoder_by_name函数来找出对应的编码器，然后调用方法avcodec_open2打开这个编码器，接下来为编码器指定frame_size的大小，一般指定1024作为一帧的大小，至此我们就把编码器部分给分配好了。

​	在此需要注意的是，某些编码器只允许特定格式的PCM作为输入源，比如声道数、采样率、表示格式（比如LAME编码器就不允许SInt16的表示格式）的要求，所以需要构造个重采样器来将PCM数据转换为编码器输人的PCM数据，即前面讲解过的需要将输入的声道、采样率、格式和输出的声道、采样率、表示格式传递给初始化方法，然后分配出重采样上下文SwrContext。此外，还要分配一个AVFrame类型的inputFrame，作为输入PCM数据存放的地方，这里需要知道inputFrame分配的buffer的大小，如上一步所述，默认一帧的大小是1024，所以对应的buffer（按照uint8_t类型作为一个元素来分配）大小就应该是：

buffersize = frame_size * sizeof(SInt 16) * channels；

​	当然也可以调用FFmpeg提供的方法av_samples_get_buffer_size来帮助开发者计算。其实这个方法内部的计算公式就是上面所列的公式。如果需要进行重采样处理，那就需要额外分配一个重采样之后的AVFrame类型的swrFrame，作为最终得到结果的AVFrame。在初始化方法的最后，需要调用FFmpeg提供的方法avformat_write_header将该音频文件的Header部分写进去，然后记录一个标志isWitHeaderSuccess，使其为true，因为后续在销段资源的阶段需要根据该标志来判断是否调用write_trailer方法，即写入文件尾部的方法，否则会造成Crash。

​	接下来看一下提供的第二个接口方法：

void encode(byte* buffer, int size)；

这里传入的参数是uint8 t类型的指针，以及这块内存所表示的数据长度，具体的实现是将该buffer填充入inputFrame，由于前面已经知道了每一帧buffer需要填充的大小是多少，所以这里可以利用一个while循环来做数据的缓冲，一次性填充到AVFrame中去，然后调用编码方法avcodec_encode_audio2，该方法会将编码好的数据放入AVPacket的结构体中，接着调用av_interleaved_write _frame方法，则可以将该packet输出到最终的文件中去。

​	最后，来看一下第三个接口方法：

```
void destroy();
```

​	上述代码所述方法需要销毁前面所分配的资源以及打开的连接通道。

​	如果初始化了重采样器，那么就销毁重采样的数据缓冲区以及重采样上下文；然后销毁为输入PCM数据分配的AVFrame类型的inputFrame，再判断标志isWriteHeaderSuccess变量决定是否需要填充duration以及调用方法av_write_trailer，最终关闭编码器以及连接通道。

**2. 实例Demo**

本文的Demo代码下载地址如下：

[下载地址]()

下载项目，运行工程之前，需要将PCM文件放入SDCard根目录下。运行工程进入编码界面，点击start开始编码，点击stop停止编码，从SDCard根目录下拷贝出AAC文件进行播放。