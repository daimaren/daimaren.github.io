---
layout:     post
title:      深入了解Android NDK
subtitle:   
date:       2020-01-03
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - Android NDK
---

Android原生开发包（NDK）可用于Android平台上的C++开发，NDK不仅仅是一个单一功能的工具，还是一个包含了API、交叉编译器、链接程序、调试器、构建工具等的综合工具集。

下面大致列举了一下经常会用到的组件。

- ARM、x86的交叉编译器
- 构建系统
- Java原生接口头文件
- C库
- Math库
- 最小的C++库
- ZLib压缩库
- POSIX线程
- Android日志库
- Android原生应用API
- OpenGL ES（包括EGL）库
- OpenSL ES库

下面来看一下Android所提供的NDK根目录下的结构。

- ndk-build：该Shell脚本是Android NDK构建系统的起始点，一般在项目中仅仅执行这一个命令就可以编译出对应的动态链接库了，后面会有详细的介绍。
- ndk-gdb：该Shell脚本允许用GUN调试器调试Native代码，并且可以配置到Eclipse的IDE中，可以做到像调试Java代码一样调试Native的代码。
- ndk-stack：该Shell脚本可以帮助分析Native代码崩溃时的堆栈信息，后续会针对Native代码的崩溃进行详细的分析。[插图]build：该目录包含NDK构建系统的所有模块。
- platforms：该目录包含支持不同Android目标版本的头文件和库文件，NDK构建系统会根据具体的配置来引用指定平台下的头文件和库文件。
- toolchains：该目录包含目前NDK所支持的不同平台下的交叉编译器——ARM、x86、MIPS，其中比较常用的是ARM和x86。构建系统会根据具体的配置选择不同的交叉编译器。

接下来详细了解一下NDK的编译脚本语法——Android.mk和Application.mk。Android.mk是在Android平台上构建一个C或者C++语言编写的程序系统的Makef ile文件，不同的是，Android提供了一系列的内置变量来提供更加方便的构建语法规则。Application.mk文件实际上是对应用程序本身进行描述的文件，它描述了应用程序要针对哪些CPU架构打包动态so包、要构建的是release包还是debug包以及一些编译和链接参数等。

Android.mk分为以下几部分。

LOCAL_PATH := (call my-dir)，返回当前文件在系统中的路径，Android.mk文件开始时必须定义该变量。

include (CLEAR_VARS)，表明清除上一次构建过程的所有全局变量，因为在一个Makefile编译脚本中，会使用大量的全局变量，使用这行脚本表明需要清除掉所有的全局变量。

LOCAL_SRC_FILES，要编译的C或者Cpp的文件，注意这里不需要列举头文件，构建系统会自动帮助开发者依赖这些文件。

LOCAL_STATIC_LIBRARIES，所依赖的静态库文件。

LOCAL_LDLIBS := -L(SYSROOT)/usr/lib -llog -lOpenSLES -lGLESv2-lEGL -lz，指定编译过程所依赖的NDK提供的动态与静态库，SYSROOT变量代表的是NDK_ROOT下面的目录NDK_ROOT/platforms/android-18/arch-arm，而在这个目录的usr/lib/目录下有很多对应的so的动态库以及．a的静态库。

LOCAL_CFLAGS，编译C或者Cpp的编译标志，在实际编译的时候会发送给编译器。比如常用的实例是加上-DAUTO_TEST，然后在代码中就可以利用条件判断#ifdefAUTO_TEST来做一些与自动化测试相关的事情。

