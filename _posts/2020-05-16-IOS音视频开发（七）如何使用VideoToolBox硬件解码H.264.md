---
layout:     post
title:      IOS音视频开发（七）如何使用VideoToolBox硬件解码H.264
subtitle:   
date:       2020-05-16
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - IOS
    - 音视频
    - VideoToolBox
---

​	首先，调用VTCompressionSessionCreate方法将要编码的视频的宽、高、编码器类型（kCMVideoCodecType_H264）、回调函数以及回调函数上下文传递进去，然后构造出一个编码器会话，该函数的返回值是一个OSStatus，如果构造成功则返回的是0，如果不成功则要给出提示，用于通知客户端代码初始化编码器会话失败。构造成功之后要为该会话设置参数，具体代码如下：

```
OSStatus status = VTCompressionSessionCreate(NULL, width, height, kCMVideoCodecType_H264, NULL, NULL, NULL, didCompressH264, (__bridge void *)(self),  &EncodingSession);
if (status != 0)
{
    NSLog(@"H264: Unable to create a H264 session status is %d", status);
    [_encoderStatusDelegate onEncoderInitialFailed];
    error = @"H264: Unable to create a H264 session";
    return ;
}

// Set the properties
VTSessionSetProperty(EncodingSession, kVTCompressionPropertyKey_RealTime, kCFBooleanTrue);
VTSessionSetProperty(EncodingSession, kVTCompressionPropertyKey_ProfileLevel, kVTProfileLevel_H264_High_AutoLevel);
VTSessionSetProperty(EncodingSession , kVTCompressionPropertyKey_AllowFrameReordering, kCFBooleanFalse);
[self settingMaxBitRate:maxBitRate avgBitRate: avgBitRate fps:fps];
```

​	这里面第一个参数设置的是需要实时编码，第二个参数设置的是使用的H264的Profile是High的AutoLevel规格，第三个参数是我们不产生B帧，第四个参数是设置关键帧间隔，也就是通常所指的gop size，第五个参数是设置帧率，第六个参数和第七个参数共同用于控制编码器输出的码率。

​	设置完这些参数之后，调用VTCompressionSessionPrepareToEncodeFrames方法，告诉编码器开始编码。

```
VTCompressionSessionPrepareToEncodeFrames(EncodingSession);
```

​	在最开始创建该会话的时候指定了一个回调函数，该回调函数是在编码器编码成功一帧之后，把编码成功的这一帧数据构造成一个CMSampleBuffer结构体以回调这个函数，开发者要在这个回调函数里处理数据。我们在第一步中仅仅明确了编码器的输入和输出，其实并没有说明编码器具体是如何使用的，具体来说是在使用我们封装的H264HWEncoder类的encode方法的时候，输入参数是一个CVPixelBuffer，然后构造当前编码视频帧的时间戳以及时长，最后调用编码会话对这三个参数进行编码。代码如下：

```
int64_t currentTimeMills = CFAbsoluteTimeGetCurrent() * 1000;
if(-1 == encodingTimeMills){
    encodingTimeMills = currentTimeMills;
}
int64_t encodingDuration = currentTimeMills - encodingTimeMills;
// Get the CV Image buffer
CVImageBufferRef imageBuffer = (CVImageBufferRef)CMSampleBufferGetImageBuffer(sampleBuffer);

// Create properties
CMTime pts = CMTimeMake(encodingDuration, 1000.); // timestamp is in ms.
CMTime dur = CMTimeMake(1, m_fps);
VTEncodeInfoFlags flags;

// Pass it to the encoder
OSStatus statusCode = VTCompressionSessionEncodeFrame(EncodingSession,
                                                      imageBuffer,
                                                      pts,
                                                      dur,
                                                      NULL, NULL, &flags);
```

​	待编码器编码成功之后，就会回调最开始初始化编码器会话时传入的回调函数，回调函数的原型如下：

```
void didCompressH264(void *outputCallbackRefCon, void *sourceFrameRefCon, OSStatus status, VTEncodeInfoFlags infoFlags, CMSampleBufferRef sampleBuffer)
```

​	在该回调函数中处理编码之后的数据，首先判断status，如果编码成功则返回0（实际上头文件中定义了一个枚举类型是noErr）；如果不成功则不处理。成功的话首先来判断编码成功之后的当前帧是否为关键帧，判断关键帧的方法如下：

```
bool keyframe = !CFDictionaryContainsKey( (CFDictionaryRef)(CFArrayGetValueAtIndex(CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true), 0)), (const void *)kCMSampleAttachmentKey_NotSync);
```

​	为什么要判断关键帧呢？因为VideoToolbox编码器在每一个关键帧前面都会输出SPS和PPS信息，所以如果本帧是关键帧，则取出对应的SPS和PPS信息。那么如何取出对应的SPS和PPS信息呢？CMSampleBuffer中有一个成员是CMVideoFormatDesc，而SPS和PPS信息就存在于这个对于视频格式的描述里面。取出SPS的代码如下：

```
CMFormatDescriptionRef format = CMSampleBufferGetFormatDescription(sampleBuffer);
// Get the extensions
// From the extensions get the dictionary with key "SampleDescriptionExtensionAtoms"
// From the dict, get the value for the key "avcC"
size_t sparameterSetSize, sparameterSetCount;
const uint8_t *sparameterSet;
OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 0, &sparameterSet, &sparameterSetSize, &sparameterSetCount, 0 );
```

​	取出PPS的代码如下：

```
size_t pparameterSetSize, pparameterSetCount;
const uint8_t *pparameterSet;
OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 1, &pparameterSet, &pparameterSetSize, &pparameterSetCount, 0 );
```

​	这样就可以取出SPS和PPS的信息了，接着再把这一帧（有可能是关键帧也有可能是非关键帧）的实际内容提取出来进行处理。首先，取出这一帧的时间戳，代码如下：

```
CMTime presentationTimeStamp = CMSampleBufferGetPresentationTimeStamp(sampleBuffer);
double presentationTimeMills = CMTimeGetSeconds(presentationTimeStamp)*1000;
```

​	然后再取出具体的压缩后的数据，代码如下：	

```
CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer);
```

​	取出真正的压缩后的数据CMBlockBuffer之后，然后就可以访问这块内存并取出具体的数据了。

​	最后是释放编码器，首先调用VTCompressionSessionCompleteFrames方法强制编码器完成编码行为，然后调用VTCompressionSessionInvalidate方法结束编码器会话，最终调用CFRelease方法释放编码器会话。释放编码器方法的签名为：

```
// Mark the completion
VTCompressionSessionCompleteFrames(EncodingSession, kCMTimeInvalid);
// End the session
VTCompressionSessionInvalidate(EncodingSession);
CFRelease(EncodingSession);
```

