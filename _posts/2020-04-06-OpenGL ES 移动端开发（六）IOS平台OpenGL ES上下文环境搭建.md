---
layout:     post
title:      OpenGL ES 移动端开发（六）IOS平台OpenGL ES上下文环境搭建
subtitle:   
date:       2020-04-06
author:     Glen
header-img: img/post-bg-none.jpg
catalog: true
tags:
    - OpenGL ES
    - IOS
---

在iOS平台上不允许开发者使用OpenGL ES直接渲染屏幕，必须使用FrameBuffer与RenderBuffer来进行渲染。若要使用EAGL，则必须先创建一个RenderBuffer，然后让OpenGLES渲染到该RenderBuffer上去。而该RenderBuffer则需要绑定到一个CAEAGLLayer上面去，这样开发者最后调用EAGLContext的presentRenderBuffer方法，就可以将渲染结果输出到屏幕上去了。实际上，在调用这个方法时，EAGL也会执行类似于前面的swapBuffer过程，将OpenGL ES渲染的结果绘制到物理屏幕上去（View的Layer），具体使用步骤如下。首先编写一个View类，继承自UIView，然后重写父类UIView的一个方法layerClass，并且返回CAEAGLLayer类型：

```
+ (Class) layerClass
{
    return [CAEAGLLayer class];
}
```

然后在该View的initWithFrame方法中，获得layer并且强制类型转换为CAEAGLLayer类型的变量，同时为layer设置参数，其中包括色彩模式等属性：

```
- (id) initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        CAEAGLLayer *eaglLayer = (CAEAGLLayer*) self.layer;
        eaglLayer.opaque = YES;
        eaglLayer.drawableProperties = [NSDictionary dictionaryWithObjectsAndKeys:
                                        [NSNumber numberWithBool:FALSE], kEAGLDrawablePropertyRetainedBacking,
                                        kEAGLColorFormatRGBA8, kEAGLDrawablePropertyColorFormat,
                                        nil];
    return self;
}
```

接下来构造EAGLContext与RenderBuffer并且绑定到Layer上，之前也提到过，必须为每一个线程绑定OpenGLES上下文。所以首先必须开辟一个线程，开发者在iOS中开辟一个新线程有多种方式，可以使用dispatch_queue，也可以使用NSOperationQueue，甚至使用pthread也可以，反正必须在一个线程中执行以下操作，首先创建OpenGL ES的上下文：

```
EAGLContext *context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
```

然后实施绑定操作，代码如下：

```
[EAGLContext setCurrentContext:_context];
```

此时就已经为该线程绑定了刚刚创建好的上下文环境了，也就是说已经建立好了EAGL与OpenGL ES的连接，接下来再建立另一端的连接。创建帧缓冲区：

```
glGenFramebuffers(1, &_framebuffer);
```

创建绘制缓冲区：

```
glGenRenderbuffers(1, &_renderbuffer);
```

绑定帧缓冲区到渲染管线：

```
glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer);
```

绑定绘制缓存区到渲染管线：

```
glBindRenderbuffer(GL_RENDERBUFFER, _renderbuffer);
```

为绘制缓冲区分配存储区，此处将CAEAGLLayer的绘制存储区作为绘制缓冲区的存储区：

```
[_context renderbufferStorage:GL_RENDERBUFFER fromDrawable:(CAEAGLLayer*)self.layer];
```

获取绘制缓冲区的像素宽度：

```
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH, &_backingWidth);
```

获取绘制缓冲区的像素高度：

```
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_backingHeight);
```

将绘制缓冲区绑定到帧缓冲区：

```
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, _renderbuffer);
```

检查FrameBuffer的status：

```
GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
if (status != GL_FRAMEBUFFER_COMPLETE) {
	NSLog(@"failed to make complete framebuffer object %x", status);
     return FALSE;
}
```

至此我们就将EAGL与Layer（设备的屏幕）连接起来了，绘制完一帧之后（当然绘制过程也必须在这个线程之中），调用以下代码：

```
[_context presentRenderbuffer:GL_RENDERBUFFER]；
```

这样就可以将绘制的结果显示到屏幕上了。至此我们就搭建好了iOS平台的OpenGL ES的上下文环境。