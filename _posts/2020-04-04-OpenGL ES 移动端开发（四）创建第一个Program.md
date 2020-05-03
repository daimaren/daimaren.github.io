---
layout:     post
title:      OpenGL ES 移动端开发（四）创建第一个Program
subtitle:   
date:       2020-04-04
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - OpenGL ES
    - Android
---

​	前面已经学习了GLSL的语法以及内嵌函数，下面就来学习 一下如何创建第一个Program。下图描述了如何创建一个显卡的可执行程序，后文中将其统称为Program。

![1588476547482](E:\all_code_in_here\daimaren.github.io\img\opengl_es_diagram.png)

​	第一步是调用glCreateShader方法创建一个对象，作为shader的容器，该函数会返回一个容器的句柄，函数的原型如下：

```
GLuint glCreateShader(GLenum shaderType);
```

​	函数原型中的参数shaderType有两种类型，当要创建VertexShader时，开发者应该传人类型GL_VERTEX_SHADER ；当要创建FragmentShader时，开发者应该传入GL_FRAGMENT_SHADER类型。下一步就是为创建的这个shader添加源代码，即框图中最右边的两个Shader Content。它们就是前面讲解过的根据GLSL语法和内嵌函数编写的两个着色器程序（Shader），其为字符串类型。函数原型如下：

```
void glShaderSource(GLuint shader, int numOfStrings, const char** strings, int stringLen);
```

​	上述函数的作用就是把开发者编写的着色器程序加载到着色器句柄所关联的内存中，最后一步就是编译该Shader。编译Shader的函数原型如下：

```
void glCompileShader(GLuint shader);
```

​	待编译完成之后，还需要验证该Shader是否编译成功了。使用下面的函数即可进行验证：

```
void glGetShaderiv(GLuint shader, GLenum pname, GLint* params);
```

​	其中第一个参数就是需要验证的Shader句柄：第二个参数是需要验证的Shader的状态值，这里一般是验证编译是否成功，该状态值一般是选取GL_COMPILE_STATUS ；第三个参数是返回值。当返回值为1时，则说明该Shader是编译成功的：如果为0则说明该Shader没有被编译成功，如果编译没有成功，肯定需要知道到底是着色器代码中的哪一行出了问题，所以还需要调用上面的函数，只不过此时获取的是该Shader的另外一个状态该状态值应该选取GL_INFO_LOG_LENGTH，返回值返回的则是错误原因字符串的长度，我们可以利用这个长度分配出一个buffer，然后调用获取Shader的InfoLog函数，函数原型如下：

```
void glGetShaderInfoLog(GLuint object, int maxLen, int *len, char* log);
```

​	之后可以把InfoLog打印出来，以帮助我们调试实际Shader中的错误。按照上面的步骤可以创建出Vertex Shader和Fragment Shader，接下来再来看框图的左半部分，即如何通过这两个Shader来创建Program。

​	首先创建一个对象，作为程序的容器，此函数将返回容器的句柄。函数原型如下：

```
GLuint glCreateProgran(void);
```

​	下面将把前文编译的Shader附加到刚刚创建的程序中，调用的函数名称如下：

```
void glAttachshader(GLuint program, GLuint shader);
```

​	第一个参数就是传入上一步返回的程序容器的句柄，第二个参数就是编译的Shade容器的句柄，当然要为每一个Shader都调用一次这个方法才能把两个Shader都关联到Program中去。最后一步就是链接程序，链接函数原型如下：

```
void glLinkProgram(GLuint program);
```

传入参数就是程序容器的句柄，那么这个程序到底有没有链接成功呢？ OpenGL提供了一个函数来检查该程序的状态，函数原型如下：

```
giGetProgramiv(GLuint program, GLenum pname, GLint* params);
```

第一个参数就是传入程序容器的句柄，第二个参数代表需要检查该程序的哪一个状态，这里传入的是GL_LINK_STATUS，最后一个参数就是返回值。返回值为1则代表链接成功，如果返回值为0则代表链接失败，类似于编译Shader的操作，如果链接失败了，也可以获取错误信息，以便修改程序。如果想获取具体的错误信息，应该调用下属函数，但是第二个参数传递的是GL_INFO_LOG_LENGTH，代表获取该程序的InfoLog的长度，获取到长度之后我们分配出一个char*的内存空间以获取InfoLog，函数原型如下：

```
void glGetProgramlnfoLog(GLuint object, int maxLen, int *len, char *log);
```

调用该函数返回InfoLog之后可以将其打印出来，以便于后续修改程序。

使用Program：

```
void glUseProgram();
```

创建Program介绍完毕，但是要想完全运行到手机上，还需要为OpenGLES的运行提供一个上下文环境，下一篇我们会介绍Android&IOS两个平台上如何为OpenGL ES提供上下文环境。