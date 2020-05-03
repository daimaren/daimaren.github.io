---
layout:     post
title:      OpenGL ES 移动端开发（五）Android平台OpenGL ES上下文环境搭建
subtitle:   
date:       2020-04-05
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - OpenGL ES
    - Android
---

OpenGL不负责窗口管理及上下文环境管理，该职责将由各个平台或者设备自行完成。为了在OpenGL的输出与设备的屏幕之间架接起一个桥梁，Khronos创建了EGL的API,EGL是双缓冲的工作模式，即有一个Back Frame Buffer和一个Front Frame Buffer，正常绘制操作的目标都是Back Frame Buffer，操作完毕之后，调用eglSwapBuffer这个API，将绘制完毕的FrameBuffer交换到Front Frame Buffer并显示出来。而在Android平台上，使用的是EGL这一套机制，EGL承担了为OpenGL提供上下文环境以及窗口管理的职责。对于Android平台的实现下面将分别给出详细的代码以及解释。

要在Android平台上使用OpenGL ES，第一种方式是直接使用GLSurfaceView，通过这种方式使用OpenGL ES比较简单，因为不需要开发者搭建OpenGL ES的上下文环境，以及创建OpenGL ES的显示设备。但是凡事都有两面，有好处也就有坏处，使用GLSurfaceView不够灵活，很多真正的OpenGL ES的核心用法（比如共享上下文来达到多线程共同操作一份纹理）都不能直接使用。所以本书的OpenGL ES上下文环境，都是直接使用EGL的API来搭建的，并且是基于C++的环境搭建的。因为如果仅仅在Java层编写，那么对于普通的应用也许可行，但是对于要进行解码或使用第三方库的场景（比如人脸识别），则需要到C++层来实施。出于效率和性能的考虑，这里的架构将直接使用Native层的EGL搭建一个OpenGL ES的开发环境。要想在Native层使用EGL，那么就必须要在Makef ile文件（Android.mk）中加入EGL库，并在使用该库的C++文件中引入对应的头文件。

需要包含的头文件：

```
#include <EGL/egl.h>
#include <EGL/eglext.h>
```

需要引入的so库：

```
LOCAL_LDLIBS += -lEGL
```

这样就可以在Android的C++开发中使用EGL了，不过要想使用OpenGL ES，还需要引入OpenGL ES对应的头文件与库。

```
#include <GLES2/gl2.h>
#include <GLES2/gl2ext.h>
```

需要引入的so库，注意这里使用的是OpenGL ES的2.0版本：

```
LOCAL_LDLIBS += -lGLESv2
```

至此，对于OpenGL的开发需要用到的头文件以及库文件就引入完毕了，下面再来看一下如何使用EGL搭建出OpenGL的上下文环境以及渲染的目标屏幕。首先EGL需要知道绘制内容的目标在哪里，EGLDisplay是一个封装系统物理屏幕的数据类型（可以理解为绘制目标的一个抽象），通常会调用eglGetDisplay方法返回EGLDisplay来作为OpenGL ES渲染的目标。在调用该方法的时候，常量EGL_DEFAULT_DISPLAY会被传进该方法中，每个厂商通常都会返回默认的显示设备，代码如下：

```
if ((display = eglGetDisplay(EGL_DEFAULT_DISPLAY)) == EGL_NO_DISPLAY) {
    LOGE("eglGetDisplay() returned error %d", eglGetError());
    return false;
}
```

然后调用eglInitialize来初始化这个显示设备，该方法会返回一个布尔型变量来代表执行状态，后面两个参数则代表Major和Minor的版本，比如EGL的版本号是1.0，那么Major将返回1,Minor则返回0。如果不关心版本号，则可都传入0或者NULL，代码如下：

```
if (!eglInitialize(display, 0, 0)) {
    LOGE("eglInitialize() returned error %d", eglGetError());
    return false;
}
```

接下来就需要准备配置选项了，一旦EGL有了Display之后，它就可以将OpenGL ES的输出和设备的屏幕桥接起来，但是需要指定一些配置项，类似于色彩格式、像素格式、RGBA的表示以及SurfaceType等，不同的系统以及平台使用的EGL标准是不同的，Android平台下的配置代码如下所示：

```
const EGLint attribs[] = { EGL_BUFFER_SIZE, 32, EGL_ALPHA_SIZE, 8, EGL_BLUE_SIZE, 8, EGL_GREEN_SIZE, 8, EGL_RED_SIZE, 8, EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,
EGL_SURFACE_TYPE, EGL_WINDOW_BIT, EGL_NONE };
if (!eglChooseConfig(display, attribs, &config, 1, &numConfigs)) {
    LOGE("eglChooseConfig() returned error %d", eglGetError());
    release();
    return false;
}
```

最终可通过调用eglChooseConfig方法得到配置选项信息，接下来就需要创建OpenGL的上下文环境EGLContext了，这里需要用到之前介绍过的EGLDisplay和EGLConf ig，因为任何一条OpenGL指令都必须在自己的OpenGL上下文环境中运行，所以可以按照如下代码构建出OpenGL的上下文环境：

```
EGLint eglContextAttributes[] = { EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE };
if (!(context = eglCreateContext(display, config, NULL == sharedContext ? EGL_NO_CONTEXT : sharedContext, eglContextAttributes))) {
    LOGE("eglCreateContext() returned error %d", eglGetError());
    return false;
}
```

函数eglCreateContext的第三个参数可以由开发者传入一个EGLContext类型的变量，该变量的意义是指可以与正在创建的上下文环境共享OpenGL资源，包括纹理ID、FrameBuffer以及其他的Buffer资源。这里暂时填写为NULL，代表不需要与其他的OpenGL ES上下文共享任何资源，后文介绍的项目中在一些条件下其实是需要共享上下文的，这点将在后面进行讨论。通过上面这三步创建OpenGL的上下文之后，说明EGL和OpenGL ES端的环境已经搭建完毕，即OpenGL ES的输出已经可以获取到了，那么应该如何将该输出渲染到设备的屏幕上呢？应该将EGL和设备的屏幕连接起来，只有这样EGL才是一个“桥”的功能，从而使得OpenGL ES的输出可以渲染到设备的屏幕上。那么如何将EGL和设备的屏幕连接起来呢？答案是使用EGLSurface, Surface实际上是一个FrameBuffer，通过EGL库提供的eglCreateWindowSurface可以创建一个可实际显示的Surface，通过EGL库提供的eglCreatePbufferSuface可以创建一个OffScreen的Surface，当然Surface也有很多属性，其中最基础的属性包括EGL_WIDTH、EGL_HEIGHT等，代码如下：

```
if (!eglGetConfigAttrib(display, config, EGL_NATIVE_VISUAL_ID, &format)) {
    LOGE("eglGetConfigAttrib() returned error %d", eglGetError());
    release();
    return surface;
}
ANativeWindow_setBuffersGeometry(_window, 0, 0, format);
if (!(surface = eglCreateWindowSurface(display, config, _window, 0))) {
    LOGE("eglCreateWindowSurface() returned error %d", eglGetError());
}
```

有的读者可能会问_window是什么？这里需要重点解释一下，这个_window就是通过Java层的Surface对象创建出的ANativeWindow类型的对象，即本地设备屏幕的表示，在Android里面可以通过Surface（通过SurfaceView或者TextureView来得到或者构建出的Surface对象）构建ANativeWindow。这需要我们在使用的时候引用头文件：

```
#include <android/native_window.h>
#include <android/native_window_jni.h>
```

调用ANativeWindow API的代码如下：

```
ANativeWindow* window = ANativeWindow_form_surface(env, surface);
```

env就是JNI层的JNIEnv指针类型的变量，surface就是jobject类型的变量，由Java层Surface对象传递而来。这样就可以把EGL和Java层的View（即设备的屏幕）连接起来了。如果要做离线的渲染，即在后台使用OpenGL进行一些图像的处理，就需要用到离线处理的Surface了，创建离线处理Surface的代码如下：

```
EGLSurface surface;
EGLint PbufferAttributes[] = { EGL_WIDTH, width, EGL_HEIGHT, height, EGL_NONE, EGL_NONE };
if (!(surface = eglCreatePbufferSurface(display, config, PbufferAttributes))) {
    LOGE("eglCreatePbufferSurface() returned error %d", eglGetError());
}
```

进行离线渲染的时候，可以使用这个Surface进行操作。现在，EGL的准备工作已经做好了，一方面为OpenGL ES的渲染准备好了上下文环境，可以接收到OpenGL ES渲染出来的纹理，另外一方面连接好了设备的屏幕（Java层提供的SurfaceView或者TextureView），那么接下来就来具体看一下如何使用创建好的EGL环境进行工作。首先需要明确一点，开发者需要开辟一个新的线程，来执行OpenGL ES的渲染操作，而且还必须为该线程绑定显示设备（Surface）与上下文环境（Context），因为每个线程都需要绑定一个上下文，这样才可以执行OpenGL的指令，所以首先需要调用eglMakeCurrent，来为该线程绑定Surface与Context。

```
eglMakeCurrent(display, eglSurface, eglSurface, context);
```

然后就可以执行RenderLoop循环了，每次循环都将调用OpenGL ES指令绘制图像。前文曾经提到过，EGL的工作模式是双缓冲模式，其内部有两个FrameBuffer（帧缓冲区，可以理解为是一个图像的存储区域），当EGL将一个FrameBuffer显示到屏幕上的时候，另外一个FrameBuffer就在后台等待OpenGL ES进行渲染输出了。直到调用函数eglSwapBuffers这条指令的时候，才会把前台的FrameBuffer和后台的FrameBuffer进行交换，这样用户就可以在屏幕上看到刚才OpenGL ES渲染输出的结果了。最后所有的绘制操作执行完毕之后，需要销毁资源。注意销毁资源也必须在这个线程中，首先要销毁显示设备（EGLSurface）：

```
eglDestroySurface(display, eglSurface);
```

为该线程解绑定Surface与Context

```
eglMakeCurrent(display, EGL_NO_SURFACE, EGL_NO_SURFACE, EGL_NO_CONTEXT);
```

最后销毁上下文（Context）：

```
eglDestroyContext(display, context);
```

至此在Android平台上的Native层中，就可以使用EGL搭建起来的OpenGL ES的开发环境了。