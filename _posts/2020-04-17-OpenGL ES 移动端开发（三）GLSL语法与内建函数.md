---
layout:     post
title:      OpenGL ES 移动端开发（三）GLSL语法与内建函数
subtitle:   
date:       2020-04-17
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - OpenGL ES
    - Android
---

## GLSL语法与内建函数

​	OpenGL学习入门技术文章连载：

- **OpenGL ES 移动端开发（一）OpenGL ES介绍** 
- [**OpenGL ES 移动端开发（二）OpenGL ES渲染管线**](http://mp.weixin.qq.com/s?__biz=MzI3MDk2ODI5MA==&mid=2247483659&idx=1&sn=9e3c30f5ebc326076ccbce115e83a402&chksm=eac9b586ddbe3c90498384e3a0fb238a0e9679f736f038efc323cb9d8852a40c927bdbe0fece&scene=21#wechat_redirect) 

​    GLSL全称为OpenGL Shading Language，是为了实现着色器的功能而向开发人员提供的一种开发语言，对其只要能理解到这个层次就可以了。

### **1. GLSL的修饰符与基本数据类型**

​    具体来说，GLSL的语法与C语言非常类似，学习一门语言，首先要看它的数据类型表示，然后再学习具体的运行流程。对于GLSL，其数据类型表示具体如下。

​    首先是修饰符，具体如下：

- const：用于声明非可写的编译时常量变量。
- atribute：用于经常更改的信息，只能在顶点着色器中使用。
- uniform：用于不经常更改的信息，可用于顶点着色器和片元着色器。
- varying：用于修饰从顶点着色器向片元着色器传递的变量。

​    然后是基本数据类型，int，float，bool，这些与C语言都是一致的，需要强调的一点就是，这里面的float是有一个修饰符的，即可以指定精度。三种修饰符的范围（范围一般视显卡而定）和应用情况具体如下。

- highp：32bit，一般用于顶点坐标（vertex Coordinate）
- medium：16bit，一般用于纹理坐标（texture Coordinate）
- lowp：8bit，一般用于颜色表示（color）

​    接下来是向量类型，向量类型是Shader中非常重要的一个数据类型，因为在做数据传递的时候需要经常传递多个参数，相较于写多个基本数据类型，使用向量类型是非常好的选择。列举一个最经典的例子，要将物体坐标和纹理坐标传递到Vertex Shader中，用的就是向量类型，每一个顶点都是一个四维向量，在Vertex Shader中利用这两个四维向量即可完成自己的纹理坐标映射操作。声明方式如下（GLSL代码）：

```
attribute vec4 position;
```

​    之后是矩阵类型，矩阵类型在Shader的语法中也是一个非常重要的类型，有一些效果器需要开发者传入矩阵类型的数据，比如后面会接触到的怀旧效果器，就需要传入一个矩阵来改变原始的像素数据。声明方式如下（GLSL代码）：

```
uniform lowp mat4 colorMatrix；
```

​    上面的代码表示了一个4x4的浮点矩阵，如果是mat2就是2 x2的浮点矩阵，如果是mat3就是3x3的浮点矩阵。若要传递一个矩阵到实际的Shader中，则可以直接调用如下函数（客户端代码）：

```
glUniformMatrix4fv(mcolorMatrixLocation， 1， false， mcolorMatrix);
```

​    紧接着是纹理类型，后面将会单独介绍应该如何加载以及渲染纹理，这里要讲解的是如何声明这个类型，一般仅在Fragment Shader中使用这个类型，二维纹理的声明方式如下（GLSL代码）：

```
uniform sampler2D texSampler；
```

​    当客户端接收到这个句柄时，就可以为它绑定一个纹理，代码如下（客户端代码）：

```
glActiveTexture(GL_TEXTUREO);
glBindTexture(GL_TEXTURE_2D，texId);
glUniformli(mGLUniformTexture, 0);
```

​    注意上述代码中第一行激活的是哪一个纹理句柄，第三行代码中的第二个参数需要传递对应的Index，就像代码中激活的纹理句柄是GL_TEXTURE0，对应的Index就是0，如果激活的纹理句柄是GL_TEXTURE1，那么对应的Index就是1，在不同的平台上句柄的个数也不一样，但是一般都会在32个以上。

​    最后来看一下比较特殊的传递类型，在GLSL中有一个特殊的修饰符就是varying，这个修饰符修饰的变量均用于在Vertex Shader和Fragment Shader之间传递参数。首先在顶点着色器中声明这个类型的变量代表纹理的坐标点，并且对这个变量进行赋值，代码如下：

```
attribute vec2 texcoord；
varying vec2 v_texcoord；
void main(void)
{
	v_texcoord = texcoord;
}
```

​    紧接着在Fragment Shader中也声明同名的变量，然后使用texture2D方法取出二维纹理中该纹理坐标点上的纹理像素值，代码如下（GLSL代码）：

```
varying vec2 v_texcoord；
vec4 pixel = texture2D(texSampler，v_texcoord)；
```

​    取出了该坐标点上的像素值之后，就可以进行像素变化操作了，比如说提高对比度，最终将改变的像素值赋值给glFragColor。

### **2. GLSL的内置函数与内置变量**

​    首先来看内置变量，最常见的是两个Shader的输出变量。

​    先来看Vertex Shader的内置变量（GLSL代码）：

```
vec4 gl_position；
```

​    上述代码用来设置顶点转换到屏幕坐标的位置，Vertex Shader-定要去更新这个数值。另外还有一个内置变量，代码如下（GLSL代码）：

```
float gl_pointSize;
```

​    在粒子效果的场景下，需要为粒子设置大小，改变该内置变量的值就是为了设置每个粒子矩形的大小。

​    其次是Fragment Shader的内置变量，代码如下（GLSL代码）：

```
vec4 glFragcolor；
```

​    上述代码用于指定当前纹理坐标所代表的像素点的最终颜色值。

​    然后是内置函数，具体的函数可以去官方文档中查询，这里仅介绍几个常用的函数。

- abs(genType x)：绝对值函数。
- floor(geType x)：向下取整函数。
- ceil(genType x)：向上取整函数。
- mod(genType x，genType y)：取模函数。
- min(genType x，genType y)：取得最小值函数。
- max(genType x，genType y)：取得最大值函数。
- clamp(genType x，genType y，genType z)：取得中间值函数。
- step(genType edge，genType x)：如果x < edge，则返回0.0，否则返回1.0。
- smoothstep(genType edge0， genType edge1， genType x）：如果x <= edge0，则返回0.0，如果x >= edge1则返回1.0；如果edge0 < x < edge1，则执行0~1之间的平滑差值。
- mix(genType x, genType y, genType a）：返回线性混合的x和y，用公式表示为x*(1-a)+y*a，这个函数在mix两个纹理图像的时候非常有用。

​    其他的角度函数、指数函数、几何函数在这里就不再赘述了，大家可以去官方文档查询。对于一个语言的语法来讲，剩下的就是控制流部分了，而GLSL的控制流与C言非常类似，既可以使用for、while以及do while实现循环，也可以使用if和if-else进条件分支的操作，在后面的实践过程中及GLSL代码中都会用到这些控制流，在这里将不讲解这些枯燥的语法。

​    GLSL的语法部分已经讲解得差不多了，下一节我们再介绍如何在应用程序中使用Shader。

![img](https://mmbiz.qpic.cn/mmbiz/A49JfDqSH0BricIm3BuCx8cYe13YdshMSib4kVicC4ianrIeUVyqY0hYLmpCAVTibhNiavCIIxiaUoPGTzckkPsK3x9aw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

​						关注公众号【OpenGL ES】，一起学习音视频开发~~~