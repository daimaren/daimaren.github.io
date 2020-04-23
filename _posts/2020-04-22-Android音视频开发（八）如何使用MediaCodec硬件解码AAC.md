---
layout:     post
title:      Android音视频开发（八）如何使用MediaCodec硬件解码AAC
subtitle:   MediaCodec硬件解码AAC
date:       2020-04-22
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android
    - 音视频
    - MediaCodec
---

**1. 实现流程**

​	使用MediaCodec编码AAC对Android系统是有要求的，必须是4.1系统以上，即要求Android的版本代号在Jelly Bean以上。MediaCodec是Android系统提供的硬件编码器，它可以利用设备的硬件来完成编码，从而大大提高编码的效率，还可以降低电量的使用，但是其在兼容性方面不如软件编码好，因为Android设备的碎片化太严重，所以读者可以自己衡量在应用中是否使用Android平台的硬件编码特性。

​	AAC编码格式分为两种，其中一种是ADTS封装格式，另外一种是裸的AAC的封装格式。MediaCodec编码出来的AAC是裸的AAC，即AAC的原始数据块，一个AAC原始数据块的长度是可变的，对原始帧加上ADTS头进行封装，就形成了ADTS帧。ADTS的全称是Audio Data Transport  Stream，是AAC音频的传输流格式，通常我们将得到的AAC原始帧加上ADTS头进行封装后写入文件，该文件使用常用的播放器即可播放，这是个验证AAC数据是否正确的方法。进行封装之前，需要了解相关的参数，如采样率、声道数、原始数据块的长度等。下面将AAC原始数据帧加工为ADTS格式的帧，根据相关参数填写组成7字节的ADTS头。

下面就来分配一下这7个字节：

```
int adtsLength = 7;
char* packet = malloc(sizeof(char) * adtsLength);
```

其中，前两个字节是ADTS同步字：

```
packet[0] = (char) 0xFF;
packet[1] = (char) 0xF9;
```

紧接着第三个字节是编码的Profle、采样率下标（注意是下标，不是采样率）、声道配置（注意是声道配

置，而不是声道数）、数据长度的组合（注意packetLen是原始数据长度加上ADTS头的长度）：

```
int profile = 2; //AAC LC
int freqIdx = 4; //44.1kHz
int chanCfg = 2;
packet[2] = (byte) (((profile - 1) << 6) + (freqIdx << 2) + (chanCfg >> 2));
packet[3] = (byte) (((chanCfg & 3) << 6)  + (packetLen >> 11));
packet[5] = (byte) (((packetLen & 7） <<5) + 0x1F);
```

最后一个字节也是固定的：

```
packet[6] = (byte) 0xFC;
```

​	至此， ADTS头就拼接上了，对于编码出一个可以播放的AAC文件来讲，这是非常重要的，对于MediaCodec编码出来的AAC来说，需要拼接上该ADTS头，最终文件才可以正确地播放出来了。

​	下面就来看看MediaCodec如何使用。类似于软件编码提供的三个接口方法，这里也提供三个接口方法分别用于完成初始化、编码数据和销毁编码器的操作。首先来看一下初始化方法，先构造一个MediaCodec实例，通过该类的静态函数来实现。构造一个AAC的Codec，其代码如下：

```
MediaCodec mediaCodec = MediaCodec.createEncoderByType("audio/mp4a-latm");
```

其实该方法有点类似于前面所讲的FFmpeg中根据Codec的name来找出编码器。构造出该实例之后，就需要配置该编码器了，配置编码器最重要的是需要传递一个MediaFormat类型的对象，该对象中配置的是比特率、采样率、声道数以及编码AAC的Profile，此外，还需要配置输入Buffer的最大值，代码如下：

```
MediaFormat encodeFormat = MediaFormat.createAudioFormat(MINE_TYPE, sampleRate, channels);
encodeFormat.setInteger(MediaFormat.KEY_BIT_RATE, bitRate);
encodeFormat.setInteger(MediaFormat.KEY_AAC_PROFILE,MediaCodecInfo.CodecProfileLevel.AACObjectLC);
encodeFormat.setInteger(MediaFormat.KEY_MAX_INPUT_SIZE, 10 * 1024);
```

与FFmpeg中的配置编码器类似，上述代码也配置了编码器要求的输人，现在就将该对象配置到编码器内部：

```
mediaCodec.configure(encodeFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
```

最后一个参数代表需要配置一个编码器，然后调用start方法开启该编码器。

​	至此，编码器已经完全配置好了，现在可以从MediaCodec实例取出两个buffer，一个是inputBuffer，用于存放输入的PCM数据；另外一个是outputBuffer，用于存放编码之后的原始AAC的数据。代码如下：

```
ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();
ByteBuffer[] outputBuffers =  mediaCodec.getOutputBuffers();
```

到此，初始化方法已实现完毕，下面来看一下编码方法。

首先，从mediaCodec中读取出一个可以用来输入的buffer的Index，然后填充数据，并且把填充好的buffer发送给Codec：

```
int inputBufferIndex = mediaCodec.dequeueInputBuffer(-1);
if (inputBufferIndex >= 0) {
    ByteBuffer inputBuffer = inputBuffers[inputBufferIndex];
    inputBuffer.clear();
    inputBuffer.put(data);
    mediaCodec.queueInputBuffer(inputBufferIndex, 0, len, System.nanoTime(), 0);
}
```

​	然后从Codec中读取出一个编码好的buffer的Index，通过Index读取出对应的outputBuffer，然后将数据读取出来，添加上ADTS头部，写文件，之后再把该outputBuffer放回到待编码填充队列中去：

```
MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
while (outputBufferIndex >= 0) {
    ByteBuffer outputBuffer = outputBuffers[outputBufferIndex];
    if (outputAACDelegate != null) {
        int outPacketSize = bufferInfo.size + 7;// 7为ADTS头部的大小
        outputBuffer.position(bufferInfo.offset);
        outputBuffer.limit(bufferInfo.offset + bufferInfo.size);
        byte[] outData = new byte[outPacketSize];
        addADTStoPacket(outData, outPacketSize);//添加ADTS头
        outputBuffer.get(outData, 7, bufferInfo.size);//将AAC数据取出到byte[]中, 偏移量offset=7
        outputBuffer.position(bufferInfo.offset);
        outputAACDelegate.outputAACPacket(outData);
    }
    mediaCodec.releaseOutputBuffer(outputBufferIndex, false);
    outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);
}
```

​	需要注意的是，上述代码中的第一行是构造出一个Bufferlnfo类型的对象，当开发者从Codec的输出缓冲区中读取一个buffer的时候， Codec会将该buffer的描述信息放入该对象中，后续会按照该对象中的信息来处理实际的内容。

​	最后来看一下销毁方法，使用完了Mediacodec编码器之后，就需要停止运行并释放编码器，代码如下：

```
if (null != mediaCodec) {
    mediaCodec.stop();
    mediaCodec.release();
}
```

​	外界调用端需要做的事情是先初始化该类，然后读取PCM文件并调用该类的编码方法，实现该类的Delegate接口，在重写的方法中将输出带有ADTS头的AAC码流直接写文件，最终编码结束之后，调用该类的停止编码方法。

**2. 实例Demo**

本文的Demo代码下载地址如下：

[下载地址]()

下载项目，运行工程之前，需要将PCM文件放入SDCard根目录下。运行工程进入编码界面，点击start开始编码，点击stop停止编码，从SDCard根目录下拷贝出AAC文件进行播放。