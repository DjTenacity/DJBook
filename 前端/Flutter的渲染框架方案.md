## [Flutter的渲染框架方案](https://mp.weixin.qq.com/s/WM46H-mR3rab521bb0wS_A)

目前正在研究的实现渲染的方案主要有2种形式PlatformView和Texture Widget。先大概讲述一下Flutter的渲染框架原理和实现，然后会对这两种方案进行对比分析。

### Flutter渲染框架和原理

Flutter的框架主要包括Framework和Engine两层，应用是基于Framework层开发的，Framework负责渲染中的Build，Layout，Paint，生成Layer等。Engine层是C++实现的渲染引擎，负责把Framework生成的Layer组合，生成纹理，然后通过OpenGL接口向GPU提交渲染数据。

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOM0hjUAqEnicyicBGnWiaHkmzibR0mjt6gWCDicIDCJIpAvmmRicB5kwibPEhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Flutter：Framework的最底层，提供工具类和方法 Painting ：封装了Flutter Engine提供的绘制接口，主要是为了在绘制控件等固定样式的图形时提供更直观、更方便的接口 Animation ：动画相关的类 Gesture ：提供了手势识别相关的功能，包括触摸事件类定义和多种内置的手势识别器 Rendering：渲染库，Flutter的控件树在实际显示时会转换成对应的渲染对象（RenderObject）树来实现布局和绘制操作

#### 渲染原理

当GPU发出Vsync信号时，会执行Dart代码绘制新UI，Dart会被执行为Layer Tree，然后经过Compositor合成后交由Skia引擎渲染处理为GPU数据，最后通过GL/Vulkan发给GPU，具体流程如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOibHoqrshnF17uPHILkuoVMjfzryQjdz4J4jfAtg0QAwnj8NqOc7gC0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当需要更新UI的时候，Framework通知Engine，Engine会等到下个Vsync信号到达的时候通知Framework，Framework进行animations，build，layout，compositing，paint，最后生成layer提交给Engine。Engine再把layer进行组合，生成纹理，最后通过OpenGL接口提交数据给GPU，具体流程如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOib82RrGuoLvLeZVcBc9asCrUZM0bPWhy0cDoojd5uhX3UHFGpHyoMaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



接下来分别分析一下两个方案的各自的特点以及使用的方式。

### PlatformView

PlatformView是Flutter官方在1.0版本推出的组件，以解决开发者想在Flutter中嵌入Android和iOS平台原生View的Widget。例如想嵌入地图、视频播放器等原生组件，对于想尝试Flutter，但是又想低成本的迁移复杂组件的团队，可以尝试PlatformView，在 Dart 中的类对应到 iOS 和 Android 平台分别是UIKitView和AndroidView。

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOjXTKx6Q2jRlVKsKKq9iaMzYwMC7RSYiaZrPva5XqfFuGWZcRgobfpXZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

那么PlatformView在点播功能中应该怎么实现，如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOLgKJ4GuQ4mFV2cKmicTHaEJyUe7SibvtunRZ7HiakQRibvEhHmc5wgTRqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中的ARMPlatformView代表业务View。

 

### Texture Widget

基于纹理实现视频渲染，Flutter官方提供的video_player则是通过这种方式实现的，以iOS为例，Native需要提供一个CVPixelBufferRef给Texture Widget，具体实现流程如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjgZmcw1EX0UE8lpWWib2cXSOcHibtBsYv1TKPF7pg9jwpnPZXy4VklibxrJICCfW8up7d4gxm9HnhfXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中的ARMTexture是业务提供CVPixelBufferRef，具体实现步骤主要是 1.继承FlutterTexture 2.管理已注册textures集合

### 性能对比

播放同一段MP4视频PlatformView和Texture Widget性能对比，Texture Widget性能相对差一些，分析主要原因是因为CVPixelBufferRef提供给Flutter的Texture，Native到Flutter会经过GPU->CPU->GPU的拷贝过程，1.0版本数据对比如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjj72KaStLuFYeRk1KAoMCcKrGyMzM7jxLMDxGcbkKVOHQRjFQLjGNv4hXjI6mNRibn64xvVYuia1ibCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/tM7aEHyOicjj72KaStLuFYeRk1KAoMCcK1qkEz2EpaJnaDxarHCgcyLoPiaMcnAH9SxKs9X9lpQL60w99qQ4szWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 遇到的问题 

Flutter播放器接入到课堂iPad中采用的是Texure的方案，在实现PlatformView和Texture Widget两个方案的时候，主要遇到了以下几个问题。

#### PlatformView内存增长问题

课堂在连续播放视频之后，出现内存暴增问题，主要原因是OpenGL操作都需要设置[EAGLContext setCurrentContext:context_]，在IOSGLRenderTarget析构的时候，没有设置context上下文。

[为什么我推荐你用 Ubuntu 开发？](http://mp.weixin.qq.com/s?__biz=MzIyMjQ0MTU0NA==&mid=2247490525&idx=1&sn=30d9939d5fc003ecc09be9bf0c79eb57&chksm=e82c22fadf5babecd47c286edf64b9cb6b31066854806c44a0c4de0f5c0836522af34644a4f4&scene=21#wechat_redirect)

[支付宝 App 启动性能优化](http://mp.weixin.qq.com/s?__biz=MzIyMjQ0MTU0NA==&mid=2247489284&idx=2&sn=f8e94b34ac4fe36bc4c7be362605ccd0&chksm=e82c2e23df5ba735ca9cd69ad6aac920f3664d06106164d508e8e511ecf6b60913a113c744f1&scene=21#wechat_redirect)

[美团基于跨平台 Flutter 的动态化平台建设](http://mp.weixin.qq.com/s?__biz=MzIyMjQ0MTU0NA==&mid=2247489796&idx=2&sn=2decea8cb8ce2145cb98253b5198c1ce&chksm=e82c2023df5ba935bf067a824479725c4d0f2ccbf07283c479a6cd6d55471dea1ddd5ed986bf&scene=21#wechat_redirect)