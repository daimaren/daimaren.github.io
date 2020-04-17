---
layout:     post
title:      OpenGL ES 移动端开发（一）OpenGL ES介绍
subtitle:   
date:       2020-04-17
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - OpenGL ES
    - Android
---

## OpenGL ES介绍

​	OpenGL（Open Graphics Library）定义了一个跨编程语言、跨平台编程的专业图形程序接口。可用于二维或三维图像的处理与渲染，它是一个功能强大、调用方便的底层图形库。对于嵌入式的设备，其提供了OpenGL ES（OpenGL for Embedded Systems）版本，该版本是针对手机、Pad等嵌入式设备而设计的，是OpenGL的一个子集。到目前为止，OpenGL已经经历过很多版本的迭代与更新，最新版本为3.0，而使用最广泛的还是OpenGL ES 2.0版本。因为在视频应用这场景下，绝大部分都是使用2D的处理与渲染，所以只需要具备2D部分的知识就可以完成绝大部分的工作了。

​	由于OpenGL是基于跨平台的设计，所以在每个平台上都要有它的具体实现，即要提供OpenGL ES的上下文环境以及窗口的管理。在OpenGL的设计中，OpenGL是不负责管理窗口的，窗口的管理将交由各个设备自己来完成，上下文环境也是一样的，其在各个平台上都有自己的实现。具体来讲，在ios平台上使用EAGL提供本地平台对OpenGL ES的实现，在Android平台上使用EGL提供本地平台对OpenGL ES的实现。所以如果想要OpenGL程序运行在多个平台上，那么也要为每个平台编写自己的上下文环境的实现。

​	这里需要介绍一下另外一个库-libSDL，它可以为开发者提供面向libSDL的AP1编程，libSDL内部会解决多个平台的OpenGL上下文环境以及窗口管理问题，开发者只需要交叉编译这个库到各自的平台上就可以做到只写一份代码即可运行到多个平台。其中FFmpeg中的fplay这一工具就是基于libSDL进行开发的。但是对于移动开发者来讲，这样就会失去一些更加灵活的控制，甚至某些场景下的功能不能实现，所以跨平台的短视频SDK基本都是基于每个平台调用平台的API来提供OpenGLES的本地实现。

​	上面介绍了OpenGL（ES）是什么，下面再来介绍一下OpenGL（ES）能做什么。其实从名字上就可以看出来，OpenGL主要是做图形图像处理的库，尤其是在移动设备上进行图形图像处理，它的性能优势更能体现出来。若要使用OpenGL ES 2.0就不得不提到GLSL，GLSL是OpenGL的着色器语言，开发人员利用这种语言编写程序运行在GPU（GraphiProcessor Unit，图形图像处理单元，可以理解为是一种高并发的运算器）上以进行图像的处理或渲染.GLSL着色器代码分为两个部分，即VertexShader（顶点着色器）与FragmentShader（片元着色器）两部分，分别完成各自在OpenGL渲染管线中的功能，具体的语法与流程会在下一篇文章中介绍。对于OpenGL ES，业界有一个著名的开源库GPUImage，它的实现非常优雅，尤其是在ios平台上实现得非常完备，不仅有摄像头采集实时渲染、视频播放器、离线保存等功能，更有强大的滤镜实现。在GPUImage的滤镜实现中，可以找到大部分图形图像处理Shader的实现，包括：亮度、对比度、饱和度、色调曲线、白平衡、灰度等调整颜色的处理，以及锐化、高斯模糊等图像像素处理的实现等，还有素描、卡通效果、浮雕效果等视觉效果的实现，最后还有各种混合模式的实现等。当然除了GPUlmage提供的这些图像处理的Shader之外，开发者也可以自己实现一些有意思的Shader，比如美颜滤镜效果、瘦脸效果以及粒子效果等那么如何利用OpenGL来完成上面所述的工作呢？后续我们会在Android和ios平台上分别实践如何使用OpenGL ES。