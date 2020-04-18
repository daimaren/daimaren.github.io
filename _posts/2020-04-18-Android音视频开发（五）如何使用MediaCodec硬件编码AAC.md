---
layout:     post
title:      Android音视频开发（五）如何使用MediaCodec硬件编码AAC
subtitle:   
date:       2020-04-17
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android
    - 音视频
    - MediaCodec
    - AAC
---

**1. 实现流程**

​	使用MediaCodec编码AAC对Android系统是有要求的，必须是4.1系统以上，即要求Android的版本代号在Jelly Bean以上。MediaCodec是Android系统提供的硬件编码器，它可以利用设备的硬件来完成编码，从而大大提高编码的效率，还可以降低电量的使用，但是其在兼容性方面不如软件编码好，因为Android设备的碎片化太严

重，所以读者可以自己衡量在应用中是否使用Android平台的硬件编码特性。

AAC编码格式，其中有一种是ADTS封装格式，另外一种就是裸的AAC的封装格式。不论

这里所说的是MediaCodec编码出来的AAC，还是在6.3.3节将要讲解的AudioToolbox编码出来的AAC。

都是裸的AAC，即AAC的原始數据块，一个AAC原始数据块的长度是可变的，对原始帧加上ADTS头进行

封装，就形成了ADTS帧.ADTS的全称是Audio Data Transport  Stream，是AAC音频的传输流格式，通常

我们将得到的AAC原始帧加上ADTS头进行封装后写入文件，该文件使用常用的播放器即可播放，这是个

验证AAC数据是否正确的方法。进行封装之前，需要了解相关的参数，如采样率、声道数、原始数据块

的长度等。下面将AAC原始数据帧加工为ADTS格式的帧，根据相关参数填写组成7字节的ADTS头。

下面就来分配一下这7个字节int adt sLength 7

char packet =malloc （sizeof （char） adtsLength）；

其中，前两个字节是ADTS同步字：

第6章音视領的采集与鳞码159

packet 【0】 = （char） OxFF；

packet 【1】= （char） OxF9：

紧接着第三个字节是编码的Profle、采样率下标（注意是下标，而不是采样率）、声道配置（注意是声道配

置，而不是声道数）、数据长度的组合（注意packetLen是原始数据长度加上ADTS头的长度）：

int profile = 2： /AC LCint fregldx= 4； 1144.1kHzint chancfg  =2；

packet 【2】 = （byte） （（（profile-1） << 6） + （fregldx  << 2） + （chancfg >> 2））；packet 【3】 = （byte）

（（chancfg & 3） <<6） + （packetLen >> 11）；packet  【4】 = （byte） （（packetLen & Ox7FF） >>

3）；packet 【5】 = （byte） （（（packetLen & 7） <<5） *  Ox1F）；

其中具体的编码Profil、采样率的下标以及声道数都可以从下方这个链接中查到相关的

所有表示

https://wiki.multimedia.cx/index.php？title-MPEG-4Audio#Chann  el-configurations最后一个字

节也是固定的：

packet 【6】= （byte） OxFC

至此， ADTS头就拼接上了，对于编码出一个可以播放的AAC文件来讲，这是非常重要的，而对于

MediaCodec以及6.3.3节将要讲到的AudioToolbox编码出来的AAC来说，都需要拼接上该ADTS头，最

终文件就可以正确地播放出来了。

下面就来看看MediaCodec如何使用。类似于软件编码提供的三个接口方法，这里也提供三个接

口方法分别用于完成初始化、编码数据和销毁编码器的操作。首先来看一下初始化方法，先构造一个

MediaCodec实例，通过该类的静态函数来实现。构造一个AAC的Codec，其代码如下：

Mediacodec mediacodec = Mediacodec.createEncoderByType  （“audio/mp4a-latm”）

其实该方法有点类似于前面所讲的FFmpeg中根据Codec的name来找出编码器。构造出该实例

之后，就需要配置该编码器了，配置编码器最重要的是需要传递一个MediaFormat类型的对象，该对象中

配置的是比特率、采样率、声道数以及编码AAC的Profile，此外，还需要配置输入Buffer的最大值，代码

如下

MediaFormat encodeFormat = MediaFormat.createAudioFormat  （MINE-TYPE， sampleRate

channels）

encodeFormat。 setinteger （MediaFormat.KEY BIT RATE。  bitRate）：encodeFormat。 setinteger

MediaFormat.KEY AAC PROFI E。 Media

codecinfo.codecProfileLevel.AAcobjectLC）；

encodeFormat。 set Integer （MediaFormat.KEY_MAX INPUT-SIZE， 10 *  1024）；与FFmpeg中的配

置编码器类似，上述代码也配置了编码器要求的输人，现在就将该对象配置到编码器内部：

mediacodec.configure （encodeFormat， null， null

Mediacodec。 CONFIGUREFLAGENCODE）：

最后一个参数代表需要配置一个编码器，而非解码器（如果是解码器则传递为0），調用start方法，代表开

启该编码器。至此，编码器已经完全配置好了，打开编码器，现在开发者可以从MediaCodec实例4取出两

个buffer，一个是inputBuffer，用于存放输入的PCM数据（类似于FFmpeg编码白AVFrame）；另外一个是

outputBuffer，用于存放编码之后的原始AAC的数据（类似于FFmpe编码的AVPacket）。代码如下：

ByteBuffer【】 inputBuffers = mediacodec.getInput Buffers（）；

ByteBuffer 【】 outputBuffers =  mediacodec.getoutputBuffers（）；

到此，初始化方法已实现完毕，下面来看一下编码方法，MediaCodec的工作原理如图6-16所示，图

6-16左边的Client元素代表要将PCM放到inputBuffer中的某个具体的buffer去；图6-16右边的Client

元素代表要将编码之后的原始AAC数据从outputBuffer中的某个具体的buffer中取出来，图6-16中，左

边的小方块代表各个inputBuffr元素，右边的小方块则代表各个outputBuffer元素。

不家家指

input

filled output buffers

empty input bufferso

Client

Codec

Clien

filled input buffers

client providesinput data

discardedoutput buffers

client consumesoutput data

图6-16

代码实现具体如下。

首先，从mediaCodec中读取出一个可以用来输入的buffer的Index，然后填充数据，并且把填充好的

buffer发送给Codec：

nt bufferlndex = codec.dequeueinputBuffer（

if （inputBufferlndex >= 0） （

ByteBuffer inputBuffer = inputBuffers （bufferindex）；input  Buffer.clear（）；

input Buffer。 put （data）；

long time = System。 nanorime （）；

11 codec。 queuelnputBuffer （bufferlndex。 o。 len， time， O）；

然后从Codec中读取出一个编码好的buffer的Index，通过Index读取出对应的output

Buffer，然后将数据读取出来，添加上ADTS头部，写文件，之后再把该outputBuffer放回到待编码填充队歹

中去：第4Bufferinfo info = new Bufferinfo（）； e数特有療得情能界int index =  codec。

dequeueoutputBuffer （info， O）；while （index>=0） { ByteBuffer  output Buffer = outputBuffers

（index） if （outputAACDelegate ！= null

int outPacketsize

= info.size + 7：

outputBuffer.position （info.offset）；

outputBuffer.limit（info.offset +info.size）：bytel】 outData = new  byte （out Packetsize】； 11添加

ADTS头信息

addADTStoPacket （outData， outPacketSize）；

1/将编码得到的AAC数据取出到目标数组中，其中7代表偏移量outputBuffer.get （outData， 7，

info.size）；

outputBuffer.position（info.offset）；下eをせoutputAACDelegate。  outputAACPacket （outData）：

codec.releaseoutputBuffer （index， false）：

index = mediacodec。 dequeueoutputBuffer （bufferinfo， O）。

需要注意的是，上述代码中的第一行是构造出一个Bufferlnfo类型的对象，当开发者从Codec的输出缓冲

区中读取一个buffer的时候， Codec会将该buffer的描述信息放入该对象中，后续会按照该对象中的信息

来处理实际的内容。

最后来看一下销毁方法，使用完了Mediacodec编码器之后，就需要停止运行并释放编码器，代码如

下：

if （null ！ ： mediacodec） （

mediacodec.release（）；外界调用端需要做的事情是先初始化该类，然后读取PCM文件并调用该类的编码方法，实现该类的Delegate接口，在重写的方法中将输出带有ADTS头的AAC码流直接写文件，最终编码结束之后，调用该类的停止编码方法。

本节的代码实例是代码仓库中的MediaCodecAudioEncoder工程，运行之前，需要将resource目录下的PCM文件放入SDCard目录下的根目录下面，编码结束之后，可以在对应的目录下获取AAC文件并播放试听。

**2. 实例Demo**

本文的Demo代码下载地址如下：

[下载地址]()

下载项目，运行工程之前，需要将PCM文件放入SDCard根目录下。运行工程进入编码界面，点击start开始编码，点击stop停止编码，从SDCard根目录下拷贝出AAC文件进行播放。