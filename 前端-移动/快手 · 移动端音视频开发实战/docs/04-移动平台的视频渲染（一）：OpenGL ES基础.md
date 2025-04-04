你好，我是展晓凯。今天我们来学习移动平台的视频渲染。

虽然我们的主题是视频渲染，但严格来讲其实是视频画面（不包含声音）的渲染。在音视频领域的大部分经典场景是离不开画面的渲染的，比如：视频播放器、视频编辑器、短视频录制、贴纸以及礼物的渲染等等，所以掌握视频画面渲染技术的重要性是不言而喻的。那视频渲染都有哪些技术可用呢？最常用的是哪一种呢？带着这些问题，让我们一起开始本节课的学习吧！

注意，这里我们所说的渲染并不单单指的是直接绘制到系统屏幕上，一些高级处理操作，比如滤镜、磨皮、瘦脸、贴纸等，也是渲染的一部分。

## 视频渲染技术选型

在开始学习移动平台的视频渲染技术之前，我们先探讨一下，渲染的源头是什么？前两节课我们一起学习了音频的渲染源是PCM数据，而视频的渲染源是YUV或者RGBA格式的数据，这种数据是描述画面最基础的格式，其中YUV常用在视频的原始格式中，RGBA常用在一些图像的原始格式上。

目前各个平台最终渲染到屏幕上的都是RGBA格式的，因为硬件对屏幕上的设计就是按照每个像素点分为四个子像素来实现的，所以YUV和RGBA之间是可以互相转换的。我们最终的实战案例是视频播放器，所以这节课我们重点学习的是如何将YUV数据渲染出来，在移动平台上都有哪些落地方案。

### 平台上层的API

Android平台在SDK层提供了Canvas和Paint的API接口，便于开发者将位图（Bitmap）绘制到系统的屏幕上，常用于一些复杂的自定义View的实现，也可以再结合SurfaceView或者TexureView，将绘制线程从主线程中独立出来进行渲染，满足基本的绘图实现。

iOS平台的SDK层也提供了Quartz 2D把位图（Bitmap）绘制到屏幕上，在一些自定义View以及获取snapshot的场景下使用。这种方式的优点是各自平台的开发者使用起来非常简单，无需了解更深层次的绘图原理就能做一些简单的图形图像处理，也可以利用系统提供的一些API进行各种变换、阴影以及渐变的处理。缺点也比较明显，需要各个平台独立书写自己平台代的码进行渲染绘制，并且当需要一些高级处理和更高性能的时候，就会捉襟见肘，比如视频的美颜、跟脸贴纸以及一些视频编辑的处理等。

### 跨平台的OpenGL ES

在移动设备上，各个平台提供了更加底层的图形绘制接口，就是众所周知的OpenGL ES。它天生就是为了跨平台而设计的，在跨平台方面具有独特的优势，由于是最底层的图形绘制接口，所以在性能以及图形的高级处理方面也都没有任何问题。缺点其实也很明显，就是掌握这项技术的学习成本是比较高的，不单单要了解OpenGL ES的各种概念，并且还要为不同平台创建自己的上下文环境，同时还得掌握OpenGL自己的编程语言GLSL的语法和运行机制。

### 平台提供的最新技术

Android在10.0系统之后开始支持Vulkan，Vulkan是用于高性能 3D 渲染的渲染方法，是Khronos Group组织开发的新一代API，用于替换掉OpenGL，所以天生也是跨平台的，支持Windows、Linux、Android，还有iOS和Mac平台，只不过在iOS平台上底层是MoltenVK来实现的。

由于这项渲染技术是OpenGL的下一代，所以渲染性能方面是非常好的，并且在模块化角度也更好一些。但是有两个缺点，一是对现有的渲染系统做大规模的迁移，没有特别友好的方案；另外一个就是对于普通的开发人员来讲，上手难度比较高。但总的来说，未来它将成为用于专业图形图像渲染引擎的底层技术。

同样iOS平台提供了Metal来作为OpenGL的替代者，在iOS12系统之后弃用 OpenGL ES，系统的一些框架全面改成默认 Metal 支持。由于Metal的设计和OpenGL比较类似，开发者上手难度并不大，绘制性能也有比较明显的提升，主要缺点是跨平台性比较差。

刚刚我们也提到了，渲染并不单单是要把画面绘制到屏幕上，更多的是需要构建一个跨平台、可扩展、高性能的渲染引擎。这对于后续的一些处理是非常关键的，最终我们选择的技术依然是OpenGL ES，虽然它上手难度会大一些，但是通过接下来的学习，我会带你彻底掌握这项技术，让我们一起来进入OpenGL ES的新世界吧！

## OpenGL ES

OpenGL的全称是Open Graphics Library，它用于二维或三维图像的处理与渲染，是一个功能强大、调用方便的底层图形库。它定义了一套跨编程语言、跨平台编程的专业图形程序接口。而OpenGL ES（OpenGL for Embedded Systems）是它在嵌入式设备上的版本，这个版本是针对智能手机等嵌入式设备设计的，可以理解为是Open GL的一个子集。

目前，OpenGL的最新版本已经达到了3.0以上，由于兼容性以及现存渲染系统的架构，目前OpenGL ES 2.0还是使用最广泛的版本。所以我们就在2.0版本的基础上进行编程，来实现图像的处理与渲染。由于在视频应用这一场景下，绝大部分都是使用二维图像的处理与渲染，所以这里我们只讨论OpenGL ES的二维部分的内容，不涉及三维部分的介绍。其实除了专业的引擎开发者，大部分应用开发者只需要有二维部分的知识就可以完成我们的绝大部分工作了。

### 上下文环境

由于OpenGL是基于跨平台设计的，所以每个平台需要自己提供渲染的基础实现，用来抹平各个系统本身的差异。就像我们常说Java语言是跨平台的，实际上是JVM抹平了各个平台的差异一样。**每个平台提供渲染的基础实现，我们称之为OpenGL的上下文环境。**另外在OpenGL的设计中，OpenGL是不负责管理窗口的，窗口的管理也交由每个平台自己来完成。与上下文环境一样，窗口管理在每个平台也都有自己的实现。

具体来讲，作为OpenGL ES的上下文环境，iOS平台为开发者提供了EAGL，而Android平台（Linux平台）为开发者提供了EGL。所以如果想要让OpenGL程序运行在多个平台上，也要学会利用各个平台提供的API接口，为OpenGL ES创建出上下文环境。

除了了解OpenGL ES的各种上下文环境，这里我们还需要知道一个开源的跨平台多媒体开发库——SDL，它给开发人员提供了面向SDL的API编程，能够通过交叉编译这个库解决多平台下需要手动构建OpenGL上下文环境以及窗口管理的问题。但在移动端，这样的实现会让我们失去一部分更加灵活的控制，导致一些场景下的功能不能实现。因此我们这里不使用SDL，而是通过裸用各个平台的API来构建OpenGL的上下文环境。

上面介绍了OpenGL（ES）是什么以及它的上下文环境，下面我们一起来看一下用OpenGL（ES）能做什么。

### OpenGL（ES）的用途

刚刚我们也提到了OpenGL（ES）是做图形图像处理的库，尤其运行在移动设备上，它有更好的性能与绘制效果。

要想了解OpenGL ES能做什么，就不得不提到一个开源项目——GPUImage。它的实现非常优雅、完备，尤其是在iOS平台，提供了视频录制、视频编辑、离线保存这些场景。

其中很重要的一部分就是内部滤镜的实现，在里面我们可以找到大部分图形图像处理Shader的实现，包括：

- 基础的图像处理算法：饱和度、对比度、亮度、色调曲线、灰度、白平衡等；
- 图像像素处理的实现：锐化、高斯模糊等；
- 视觉效果的实现：素描、卡通、浮雕效果等；
- 各种混合模式的实现。

除此之外，开发者也可以自己去实现美颜滤镜效果、瘦脸效果以及粒子效果等等。

GPUImage框架是使用OpenGL ES中的GLSL书写出了上述的各种Shader。GLSL（OpenGL Shading Language）是OpenGL的着色器语言，也是2.0版本中最出色的功能。开发人员可以使用这种语言编写程序运行在GPU上来进行图像处理或渲染。使用GLSL写的代码最终可以编译、链接成为一个GLProgram。

> 注：Graphic Processor Unit是图形图像处理单元，可以把它理解成一种高并发的运算器。

一个GLProgram分为两部分，一是顶点着色器（Vertex Shader），二是片元着色器（Fragment Shader），这两部分分别完成各自在OpenGL 渲染管线中的功能。那它们是怎么工作的呢？我们一起来看一下。

### OpenGL渲染管线

如果想要了解着色器并理解它们的工作机制，就要对OpenGL的渲染管线有深入的了解。在学习之前，我们还是先来了解几个概念。我们平时说的点、直线、三角形都是几何图元，是在顶点着色器中指定的，用这些几何图元创建的物体叫做模型，而**根据这些模型创建并显示图像的过程就是我们所说的渲染。**在渲染过程结束之后，图像就会以像素点的形式绘制到屏幕上了。

一张RGBA格式的图像在内存中是这样描述的：每四个Byte表示一个像素点的RGBA的数据，这些像素点可以被组织成一个大的一维数组，用来在内存中表述这张图片。那在显存中如何描述呢？

在显存中，这些像素点被组织成帧缓冲区（framebuffer），帧缓冲区保存了显卡为了控制屏幕上所有像素的颜色以及强度所需要的全部信息。理解了帧缓冲区的概念，接下来我们就可以进入OpenGL渲染管线的学习了，这个部分是学会使用OpenGL非常重要的环节。

OpenGL的渲染管线具体是指什么呢？其实就是**OpenGL引擎把图像（内存中的RGBA数据）一步步渲染到屏幕上去的步骤**。渲染管线主要分为以下几个阶段：

![图片](https://static001.geekbang.org/resource/image/10/34/10490f4dec8ce4cac5572b6ff2096434.png?wh=1920x404 "图1 OpenGL的渲染管线")

**阶段一：指定几何对象**

几何对象，就是刚才我们说的几何图元，这里OpenGL引擎会根据开发者的指令去绘制几何图元。OpenGL（ES）提供给开发者的绘制方法glDrawArrays的第一个参数mode，就是指定绘制方式。绘制方式的可选值有点、线、三角形三种，分别应用于不同的场景中。

粒子效果的场景中，我们一般用点（GL\_POINTS）来绘制；直线的场景中，我们主要用线（GL\_LINES）来绘制；所有二维图形图像的渲染，都用三角形（GL\_TRIANGLE\_STRIP）来绘制。

我们根据不同的场景选择的绘制方式，就决定了OpenGL（ES）渲染管线的第一阶段怎么去绘制几何图元。

**阶段二：顶点变换**

不论阶段一指定的是哪种几何对象，所有的几何数据都要来到顶点处理阶段，顶点处理阶段的操作就是变换顶点的位置：一是根据模型视图和投影矩阵的计算来改变物体（顶点）坐标的位置；二是根据纹理坐标与纹理矩阵来改变纹理坐标的位置。这里的输出是用gl\_Position来表示具体的顶点位置的，如果是用点（GL\_POINTS）来绘制几何图元的话，还应该输出gl\_PointSize。

如果涉及三维的渲染，这里还会处理光照计算和法线变换，因为这节课不会涉及三维图像的渲染，所以这里不做过多介绍。

**阶段三：图元组装**

在阶段二的时候，我们已经确定了物体坐标和顶点坐标，然后到这个阶段进行图元组装。这里我们根据应用程序送往图元的规则（如GL\_POINTS 、GL\_TRIANGLES 等）以及具体的纹理坐标，把纹理组装成图元。

**阶段四：栅格化操作**

由上一阶段传递过来的图元数据，在这里会被分解成更小的单元并对应帧缓冲区的各个像素。这些单元叫做“片元”，一个片元可能包含窗口颜色、纹理坐标等属性。片元的属性则是图元上顶点数据经过插值而确定的，这其实就是栅格化操作，这个操作能够确定每一个片元是什么。

**阶段五：片元处理**

通过纹理坐标取得纹理中相对应的片元像素值，根据自己的需要改变这个片元，比如调节饱和度、锐化等，最终输出的是一个四维向量gl\_FragColor，我们用它来表示修改之后的片元像素。

**阶段六：帧缓冲操作**

帧缓冲操作是渲染管线的最后一个阶段，执行帧缓冲的写入等操作，OpenGL引擎负责把最终的像素值写入到帧缓冲区中。

这就是OpenGL渲染管线的六个步骤，从指定几何图元到帧缓冲区写入像素，图像就被OpenGL 引擎一步步地渲染到屏幕（FBO）上去了。

还记得我们之前提到过，OpenGL ES 2.0版本与之前的版本相比，最出色的功能就是开发者能够编写着色器来完成渲染管线的某个阶段。这里的“某个阶段”，实际上就是顶点变换阶段和片元处理阶段。开发者可以使用顶点着色器自己完成顶点变换阶段；也能使用片元（像素）着色器来完成片元处理阶段。

具体该如何书写顶点着色器和片元着色器呢？在专栏的第6讲我会详细地讲解GLSL语法以及内嵌函数，在这里先假设我们已经书写好了顶点着色器和片元着色器，现在需要通过OpenGL ES提供给开发者的接口来创建显卡可执行程序了。

### 创建显卡执行程序

假如我们已经完成了一组Shader的实例。那如何让显卡来运行这一组Shader呢？或者说如何用我们的Shader来替换掉OpenGL渲染管线中的那两个阶段呢？下面我们就来学习一下如何把Shader扔给OpenGL的渲染管线，这一部分叫做接口程序部分，就是利用OpenGL提供的接口来驱动OpenGL进行渲染。我们先来看一下图2，它描述了如何创建一个显卡的可执行程序（Program）。

![图片](https://static001.geekbang.org/resource/image/2c/6e/2cf655052d6a9e2bd5472e26a7b7be6e.png?wh=1920x1137 "图2 创建显卡可执行程序流程图")

根据图示，我们来看一看如何一步步地创建出这个Program。首先看图的右半部分，也就是创建Shader的过程。

#### 创建Shader

第一步是调用glCreateShader方法创建一个对象，作为Shader的容器，这个函数返回一个容器的句柄，函数原型如下：

```plain
GLuint glCreateShader(GLenum shaderType); 
```

函数原型中的参数shaderType有两种类型：

- 一是GL\_VERTEX\_SHADER，创建顶点着色器时开发者应传入的类型；
- 二是GL\_FRAGMENT\_SHADER，创建片元着色器时开发者应传入的类型。

第二步就是给创建的这个Shader添加源代码，对应图片最右边的两个Shader Content，它们就是根据GLSL语法和内嵌函数书写出来的两个着色器程序，都是字符串类型。这个函数的作用就是把开发者的着色器程序加载到着色器句柄所关联的内存中，函数原型如下：

```plain
void glShaderSource(GLuint shader, int numOfStrings, const char **strings, int         
        *lenOfStrings)
```

最后一步就是来编译这个Shader，编译Shader的函数原型如下：

```plain
void glCompileShader(GLuint shader);
```

编译完成之后，怎么知道这个Shader被编译成功了呢？我们可以使用下面这个函数来验证。

```plain
void glGetShaderiv (GLuint shader, GLenum pname, GLint* params);
```

其中，第一个参数GLuint shader就是我们要验证的Shader句柄；第二个参数GLenum pname是我们要验证的这个Shader的状态值，我们一般在这里验证是否编译成功，这个状态值选择GL\_COMPILE\_STATUS。第三个参数GLint* params就是返回值，当返回值是1的时候，说明这个Shader编译成功了；如果是0，就说明这个shader没有被编译成功。

这个时候，我们就需要知道着色器代码中的哪一行出了问题，所以还需要开发者再次调用这个函数，只不过是获取这个Shader的另外一种状态，这个时候状态值应该选择GL\_INFO\_LOG\_LENGTH，那么返回值返回的就是错误原因字符串的长度，我们可以利用这个长度分配出一个buffer，然后调用获取出Shader的InfoLog函数，原型如下：

```plain
void glGetShaderInfoLog(GLuint object, int maxLen, int *len, char *log);
```

之后我们可以把InfoLog打印出来，帮助我们来调试实际的Shader中的错误。按照上面的步骤我们可以创建出Vertex Shader和Fragment Shader，那么接下来我们看图片中的左半部分，也就是如何通过这两个Shader来创建Program。

#### 创建Program

首先我们创建一个程序的实例作为程序的容器，这个函数返回程序的句柄。函数原型如下：

```plain
GLuint glCreateProgram(void);
```

紧接着将把上面部分编译的shader附加（Attach）到刚刚创建的程序中，调用的函数名称如下：

```plain
void glAttachShader(GLuint program, GLuint shader);
```

其中，第一个参数GLuint program就是传入在上面一步返回的程序容器的句柄，第二个参数GLuint shader就是编译的Shader的句柄，当然要为每一个shader都调用一次这个方法才能把两个Shader都关联到Program中去。

当顶点着色器和片元着色器都被附加到程序中之后，最后一步就是链接程序，链接函数原型如下：

```plain
void glLinkProgram(GLuint program);
```

传入参数就是程序容器的句柄，那么具体这个程序到底有没有链接成功呢？OpenGL也提供了一个函数用来检查这个程序的状态，函数原型如下：

```plain
glGetProgramiv (GLuint program, GLenum pname, GLint* params);
```

第一个参数就是传入程序容器的句柄，第二个参数代表我们要检查这个程序的哪一个状态，这里面我们传入GL\_LINK\_STATUS，最后一个参数就是返回值。返回值是1则代表链接成功，如果返回值是0则代表链接失败。类似于编译Shader的操作，如果链接失败了，可以获取错误信息，以便修改程序。

获取错误信息时，也得先调用这个函数，但是第二个参数我们传递的是GL\_INFO\_LOG\_LENGTH，代表获取出这个程序的InfoLog的长度，拿到长度之后我们分配出一个char\*的内存空间以获取出InfoLog，函数原型如下：

```plain
void glGetProgramInfoLog(GLuint object, int maxLen, int *len, char *log);
```

调用这个函数返回的InfoLog，可以作为Log打印出来，以便定位问题。

到这里我们就创建出了一个Program。回顾一下整个过程和C语言的编译和链接阶段非常相似，对比构造OpenGL Program，编译阶段做了编译顶点着色器和片元着色器，并且把两个着色器Attach到Program中，链接阶段就是链接这个Program。

创建显卡执行程序之后就是如何使用这个程序，其实使用这个构建出来的程序也很简单，调用glUseProgram这个方法即可，然后将一些内存中的数据与指令传递给这个程序，让这个程序运行起来。但是要想完全运行到手机上，还需要为OpenGL ES的运行提供一个上下文环境，下节课我们就来学习在两个平台上如何为OpenGL ES 创建上下文环境，但是你一定要记住我们刚刚创建好的这个Program，因为在接下来的课程中还会使用到。

## 总结

这节课我们从视频渲染的技术选型开始，给你介绍了视频渲染的实现手段和各自的优缺点，然后对OpenGL ES概念、用途、操作步骤等有了一个整体的了解。

![图片](https://static001.geekbang.org/resource/image/60/10/600e8cb4a0397c4d9c83a88bb061d410.png?wh=1920x1528)

其中我们重点学习了OpenGL ES的渲染管线以及渲染管线中留给开发者书写的顶点着色器和片元着色器两个阶段，最后学习使用OpenGL ES提供的接口API创建了一个显卡可执行程序，但是想让这个程序运行到显卡上，还需要一个OpenGL ES的上下文环境，关于构建OpenGL ES上下文环境的内容，我们会在下一节课中进行实际操作。期待一下吧！

## 思考题

学完了OpenGL ES基础内容，想必你已经对OpenGL ES有了比较深的了解。那我来考考你。OpenGL ES渲染管线分为哪几个步骤呢？哪些步骤是我们开发者可以自己去完成的？期待在评论区看到你的回答，也欢迎你把这节课分享给需要的朋友。我们下节课再见！
<div><strong>精选留言（2）</strong></div><ul>
<li><span>大土豆</span> 👍（2） 💬（1）<p>老师，屏幕有自己的缓冲区，和FBO还不能等同😀。FBO是我们自己新建的帧缓冲对象，通常可用来做离屏渲染，还有就是要上屏之前做个统一处理的节点，OpenGL的一些管线步骤之后可以先渲染到FBO上。但是我们完全可以绕过FBO，h264解封装解码之后的YUV数据，直接走纹理贴图的流程，对接到屏幕的缓冲区，上屏。</p>2022-08-01</li><br/><li><span>peter</span> 👍（0） 💬（1）<p>请教老师几个问题：
Q1：Android10.0以后，还能使用OpenGL ES吗？
文中提到，Android10.0以后，系统开始支持Vulkan，那么，还支持OpenGL ES吗？如果应用原来用OpenGL ES开发的，在10.0版本还能使用吗？
Q2：视频播放器是否有源代码？
文中提到会开发一个视频播放器，有源码吗？ 是安卓版本还是iOS版本？

Q3：OpenGL ES本身不能完成底层绘制功能吗？
文中有一句：“由于 OpenGL 是基于跨平台设计的，所以每个平台需要自己提供渲染的基础实现”，从这句话看，似乎OpenGL ES本身不能完成绘制功能。
Q4：安卓中canvas.draw跟OpenGL ES是什么关系？
在安卓中使用自定义View时，常用的操作是canvas.draw（）。请问，这个draw函数和
OpenGL ES是什么关系？</p>2022-08-02</li><br/>
</ul>