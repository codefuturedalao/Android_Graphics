# Android 14 图像系统

序言



## 基础

### Binder

### MessageQueue

## 启动

## 渲染

移动互联网发展至今，Android开发模式在不断更迭， 目前主要有三种开发模式 ：原生开发、Hybrid开发以及跨平台开发。

https://www.cnblogs.com/LO-ME/p/10771551.html

- **原生开发：** 移动终端的开发主要分为两大阵营， Android(Java、Kotlin) 研发与 IOS(Swift)研发。

- WebApp开发：

- **Hybrid开发：** 多种技术栈混合开发App， 在Android中主要指Native与前端(JavaScript)技术的混合开发方式。

  WebView属于Hybrid开发，采用H5编写页面，Android本身提供的WebView来进行渲染渲染

- **跨平台研发：** 同一个技术栈， 同一套代码可以在不同的终端上运行，极大的缩减了研发成本， 比如当下比较火的Flutter。

### 硬件渲染

### 软件渲染

我们都知道Android支持2D绘图，API使用的是[[Canvas\]](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fgraphics%2FCanvas)，也支持3D绘图，API使用的是[[OpenGL ES\]](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fgraphics%2Fopengl)

我们以2D绘图的流程来举例：

> **1、需要显示图形时，首先创建一个Surface对象**
>
> **2、调用Surface#lockCanvas()获取Canvas对象**
>
> **3、调用Canvas的draw开头的函数执行一系列的绘图操作**
>
> **4、调用Surface#unlockCanvasAndPost()将绘制完成的图层提交，等待下一步合成显示**

第二步的[[lockCanvas()\]](https://link.juejin.cn?target=http%3A%2F%2Fwww.aospxref.com%2Fandroid-7.1.2_r39%2Fxref%2Fframeworks%2Fbase%2Fcore%2Fjava%2Fandroid%2Fview%2FSurface.java%23299)方法返回了Canvas对象供开发者绘图使用，其内部就调用了BufferQueue#dequeueBuffer()申请一块图形buffer，后续所有的绘图结果都会写入这块内存中

第四步的[[unlockCanvasAndPost()\]](https://link.juejin.cn?target=http%3A%2F%2Fwww.aospxref.com%2Fandroid-7.1.2_r39%2Fxref%2Fframeworks%2Fbase%2Fcore%2Fjava%2Fandroid%2Fview%2FSurface.java%23321)方法内部调用了BufferQueue#queueBuffer()方法将绘制完成的Buffer入列，等待sf进程在下一次同步信号周期合成并完成送显



在Android应用程序窗口中, 每一个View都抽象为一个Render Node, 而且如果一个View设置有Background, 这个background 也被抽象为一个Render Node 。

这是由于在OpenGLRenderer库中, 并没有View的概念, 所有的一切可绘制的元素都抽象为一个Render Node。

每一个Render Node都关联有一个`DisplayList Renderer`, Display List是一个绘制命令缓冲区。当View的成员函数onDraw被调用时, 我们调用通过参数传递进来的Canvas的`drawXXX`成员函数绘制图形时, 我们实际上只是将对应的绘制命令以及参数保存在一个Display List中。接下来再通过DisplayList Renderer执行这个Display List的命令, 这个过程称为Display List Replay。

Android应用程序窗口的View是通过树形结构来组织的。这些View不管是通过硬件加速渲染还是软件渲染, 或者是一个特殊的TextureView,在它们的成员函数onDraw被调用期间, 它们都是将自己的UI绘制在ParentView的DisplayList中。

其中, 最顶层的Parent View是一个Root View, 它关联的RootNode称为`Root Render Node`。也就是说, 最终Root Render Node的DisplayList将会包含一个窗口的所有绘制命令。

在绘制窗口的下一帧时, RootRender Node的Display List都会通过一个OpenGL Renderer真正地通过Open GL命令绘制在一个`Graphic Buffer`中。

最后这个 Graphic Buffer 被交给 SurfaceFlinger 服务进行合成和显示。

## 合成

**DPU作为图形硬件的一部分，通常被封装在GPU模块当中，最主要的功能是将GPU渲染完成的图层输出到屏幕**

**兼任了合成功能以后，对于图层重叠的部分，DPU会自动计算出“脏区域”并更新像素颜色变化**

不过，DPU虽然可以执行合成工作，但它有合成数量的限制

如下图，Arm Mali-DP550这款DPU最多能够支持7层的合成任务

上一节聊DPU的时候我们提到了，如果想把合成流程独立出来，只需要单独配置一块2D渲染芯片就行了

厂商可以选择将合成工作放在`DPU`中，也可以选择在板子上加一块`2D渲染芯片`，将合成工作放在这块芯片中

抠门一点的厂商，可以选择不单独配置合成芯片，只使用GPU进行合成

这就需要一个规范把合成工作抽象成一个接口，由厂商自由选择合成方案

[Hardware Composer](https://link.juejin.cn?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Fgraphics%2Fhwc)就是专门用来定义合成工作的抽象接口，它是[Android Hardware Abstraction Layer（HAL）](https://link.juejin.cn?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Farchitecture%2Fhal-types%3Fhl%3Dzh-cn)硬件抽象层的成员之一

在HWC中，厂商使用的是DPU还是其他的2D渲染芯片不重要，只需要实现HWC的接口即可

## 显示











## 番外

### SurfaceView

SurfaceView从Android 1.0(API level 1)时就有 。它继承自类View，因此它本质上是一个View。但与普通View不同的是，它有自己的Surface。我们知道，一般的Activity包含的多个View会组成View hierachy的树形结构，只有最顶层的DecorView，也就是根结点视图，才是对WMS可见的。这个DecorView在WMS中有一个对应的WindowState。相应地，在SF中对应的Layer。而SurfaceView自带一个Surface，这个Surface在WMS中有自己对应的WindowState，在SF中也会有自己的Layer。

也就是说，虽然在App端它仍在View hierachy中，但在Server端（WMS和SF），它与宿主窗口是分离的。这样的好处是对这个Surface的渲染可以放到单独线程去做，渲染时可以有自己的GL context。这对于一些游戏、视频等性能相关的应用非常有益，因为它不会影响主线程对事件的响应。但它也有缺点，因为这个Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移，缩放等变换，也不能放在其它ViewGroup中，一些View中的特性也无法使用。

### RenderPipeLine

HWUI全称**Hardware Accelerated Rendering Engine for UI，**hwui是一个基于GPU加速的2D图形引擎。HWUI的目标是提供高效、稳定、高质量的2D图形渲染能力，为Android系统的UI体验提供技术支持。

<img src="android渲染博客架构.assets/898345ae318f0d8807bfb67dc087fb61.png" alt="898345ae318f0d8807bfb67dc087fb61.png" style="zoom: 33%;" />

Android中的图像生产者——**SKIA，OPenGL ES，Vulkan**，他们是Android中最重要的三支画笔。

* Skia

  https://skia.org/docs/

  Skia是HWUI的核心图形库，提供了基本的绘制功能，包括图形、文本、位图等。它支持硬件加速渲染，能够充分利用GPU进行并发计算，加快UI界面的渲染速度。Skia还提供了强大的API接口，方便开发人员对图像进行自定义绘制和处理。Skia在结构上可以分为三层

  1. 画布层Canvas

     画布层与设备无关，可以对字体、图片、图形等进行绘制

  2. 渲染设备层

     1. SKBitMapDevice

        CPU渲染模式绘图，用于没有显卡或者显卡驱动的设备。此模式下，最后会将需要绘制的图形转成位图数据（RGB）写入指定内存，故称为BitmapDevice。写内存操作通过AVX或者NEON指令集实现。

     2. SKGPUDevice

     3. SKPDFDevice

  3. 封装适配层

     Skia为了屏蔽不同依赖库的接口差异，对依赖库进行了封装和适配。例如基于图片编解码库libjpeg-turbo、libpng、libwebp 封装了类SKJpegCodec、SKPngCodec、SKWebpCodec。**基于底层图形库OpenGL、Metal、Vulkan封装了GrGLOpsRenderPass， GrMTOpsRenderPass, GrVKOpsRenderPass**三个类。基于苹果平台CoreText字体库和开源字体FreeType封装了类SkScalerContext_Mac和SkScalerContext_FreeType。

* OpenGL ES

  OpenGL ES是HWUI与GPU之间的桥梁，负责将Skia生成的绘制命令转化为GPU能够执行的指令序列。OpenGL是一套图像编程接口，对于开发者来说，其实就是一套C语言编写的API接口，通过调用这些函数，便可以调用显卡来进行计算机的图形开发。虽然OpenGL是一套API接口，但它并没有具体的实现这些接口，接口的实现是由显卡的驱动程序来完成的。

  OpenGL虽然是跨平台的，但是在各个平台上也不能直接使用，因为每个平台的窗口都是不一样的，而EGL就是适配Android本地窗口系统和OpenGL ES桥接层。OpenGL ES 定义了平台无关的 GL 绘图指令，EGL则定义了控制 displays，contexts 以及 surfaces 的统一

* Vulkan



- 

WebView

Android WebView加载了Chromium动态库之后，就可以启动Chromium渲染引擎了。Chromium渲染引擎由Browser、Render和GPU三端组成。其中，Browser端负责将网页UI合成在屏幕上，Render端负责加载网页的URL和渲染网页的UI，GPU端负责执行Browser端和Render端请求的GPU命令。本文接下来详细分析Chromium渲染引擎三端的启动过程。

Android WebView使用了单进程架构的Chromium来加载和渲染网页，因此它的Browser端、Render端和GPU端都不是以进程的形式存在的，而是以线程的形式存在。其中，Browser端实现在App的UI线程中，Render端实现在一个独立的线程中，而GPU端实现在App的Render Thread中。

<img src="android渲染博客架构.assets/image-20240708203729379.png" alt="image-20240708203729379" style="zoom:33%;" />

因为`Chromium WebView`没有多进程架构，所以图中所有线程都**工作在主进程**中。首先看渲染线程，它也不再工作在`Renderer`进程。同`Content Shell`类似，它也是包括渲染树和合成树。`GPU`线程也是类似的功能。不同之处在于同步的合成器，将**网页的合成器放在主线程中来工作**，这里主要的问题在于，需要绘制网页内容到`UI`中的时候需要在`UI`线程。

在`Android`系统中，如果用户界面设置的渲染方式硬件加速渲染方式，那么`Android`会使用内部`HwUI`机制。该机制会为每个`Activity`的内部`View`（`ViewRootImpl`类）构建一个`HardwareRenderer`，该对象能够创建`EGLSurface`和`EGLContext`，并将绘图动作转变成`OpenGLES`的操作。当绘制界面内容的时候，会充值当前`EGLContext`为自己创建的，逐次调用每个`View`的绘图操作。当绘制到`WebView`对象的时候，它会调用同步合成器将网页的结果绘制到当前的`FBO`中。下图是`Android`系统如何调用`WebView`的绘图网页内容的过程。

* WebKit
* Chromium

### Flutter

<img src="https://docs.flutter.dev/assets/images/docs/arch-overview/archdiagram.png" alt="Architectural diagram" style="zoom: 25%;" />





WebView

渲染管线：WebKit/Blink

Flutter

渲染管线：自定义渲染管线，底层使用Skia

## 参考

::: {#refs}
:::

rtd