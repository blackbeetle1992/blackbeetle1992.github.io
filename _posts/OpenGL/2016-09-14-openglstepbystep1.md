---
layout: post
title:  "一步步学OpenGL(1) -《打开一个窗口》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-14
categories: [OpenGL]
tags: [opengl]
icon: 

---

## @专栏介绍：

这里开一个专栏，翻译OGLdev的系列教程《OpenGL Step by Step》，由于本人是个程序员，所以对教程不会完全的简单直译，会根据自己的理解进行一个汉语的解释以及补充，尽量将原文的意思介绍清楚。原文核心内容是对函数的详细解释，这里为了更容易理解，将作者的实例代码加注释后贴在后面代码片里，使程序员看上去更容易理解和动手测试。由于本人也在学习阶段，难免有理解不精确的地方，欢迎大家指正。另外，这个教程对于初学OpenGL的人来说是很棒的，方便快速入门。

**PS:心急吃不了热豆腐，仔细阅读理解下面的源代码详解之后再看demo程序会事半功倍。个人领悟：走马观花不得其解。拿10秒钟看下面的详解可能只得到百分之十的收益，拿十分钟看下面的详解却可能得到百分之两百的效果！**

**PPS:看系列教程之前先去网上走马观花看看相关基础知识，对基本的概念心中有数效果会更好！**

**PPPS：注意本教程中需要使用的是`freeGLUT`（GLUT已经死了，太老了，会有潜在危险）窗口库和`GLEW`扩展库.**

**vs2013配置freeGLUT3.0：**[http://blog.csdn.net/yinglang19941010/article/details/50166343](http://blog.csdn.net/yinglang19941010/article/details/50166343)
**Windows上配置freeGLUT和GLEW:**[http://blog.csdn.net/xuguangsoft/article/details/8002375](http://blog.csdn.net/xuguangsoft/article/details/8002375)

(MAC上配置使用glew方法：[http://blog.csdn.net/huyisu/article/details/42742379](http://blog.csdn.net/huyisu/article/details/42742379))

***

***教程一：***

# **打开一个窗口**


*** 原文: *** [http://ogldev.atspace.co.uk/www/tutorial01/tutorial01.html](http://ogldev.atspace.co.uk/www/tutorial01/tutorial01.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)


***

### ***背景***

官方OpenGL的文档里并没有提供一个API来进行窗口的创建和操作。现在的Windows系统包含一个子系统将OpenGL上下文和Windows 系统绑定在一起从而对OpenGL提供支持。在X窗口系统中那个接口称作GLX。为了支持窗口Windows提供WGL(发音：Wiggle)接口，MacOS则有CGL。使用这些接口直接来创建窗口来显示图形通常非常麻烦，所以我们通常使用一个高水平的库来隐藏窗口的创建操作细节。这里使用的窗口库是GLUT（OpenGL utility library，**freeGLUT**是GLUT的开源版本，老GLUT早已停止更新），它提供了一个简化的API来操作窗口，以及支持事件处理，IO控制和其他一些功能。另外GLUT是一个跨平台的库所以移植性很好。和GLUT类似的其他替代库还有**SDL和GLFW**。




### ***源代码详解***  

(1)`glutInit(&argc, argue);`

调用这个函数来初始化GLUT.参数可以直接引用command line的，而且有一些有用的选项，比如：‘-sync’和‘-gldebug’，可以禁掉X窗口的异步特征并分别自动检查和显示GL错误。
    
(2)`glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);`

这里配置一些GLUT的选项设置。GLUT_DOUBLE在多数渲染结束后开启双缓冲机制（维护两个图像缓冲数据，屏幕显示一副图像时在后台同时绘制另一份图像缓冲数据，交替显示）和颜色缓冲。我们通常需要这两个设置，还有其他的选项设置后面会继续介绍。
    
(3)
`glutInitWindowSize(960, 640);      // 窗口尺寸`

`glutInitWindowPosition(200, 200);  // 窗口位置`

`glutCreateWindow("Tutorial 01");   // 窗口标题`

这几个函数的调用可以设置一些窗口参数并创建一个窗口。也可以定义窗口的标题。
    
 (4)`glutDisplayFunc(RenderScenceCB);`
 
 由于我们是在一个窗口系统中工作的，与运行的程序多数的交互是通过事件回调函数。GLUT针对与底层窗口系统的交互为我们提供了几个回调函数选项。这里我们先只用一个主回调来完成一帧图像的所有渲染工作。这个回调函数会不断地被GLUT内部循环调用。
    
 (5)`glClearColor(0.0f, 0.0f, 0.0f, 0.0f);`
 
这个是我们在OpenGL中遇到的第一个状态（OpenGL是一个状态机）。OpenGL使用状态方案的原因是渲染是一个非常复杂的任务，不能仅仅通过一个函数接受几个参数来完成（一个合理设计的函数是不会接受大量的参数的）。对于渲染效果的设置我们需要定义shader着色器，buffer缓冲还有各种各样的flag标志变量。另外，我们也经常想保存一些相同的配置在多个渲染操作中使用（比如：如果我们从来不需要禁掉深度检测depth test，我们没必要在每一个渲染回调中来明确定义它）。这也是为什么多数的渲染操作配置都是通过在OpenGL状态机中设置flag标志变量和值来完成，而且渲染回调本身通常也被局限于几个参数，参数解决需要绘制的定点数量和他们的偏移量。调用一个改变状态的函数后，具体的配置保持不变，直到下次再调用这个相同的函数再次改变状态和配置。上面的函数设置了当帧缓存（帧缓存后面还会介绍）清空后要使用的颜色值。颜色值有四个通道（RGBA）,使用单位化的值0.0-1.0来表示。
    
(6)`glutMainLoop();`

这个函数调用传递指令给GLUT现在开始它的内部循环。在这个循环中它监听窗口系统中的事件并通过我们配置的回调传递出去。在我们这个例子中，GLUT将只会调用我们注册的那个display回调（RenderScenceCB），在这个回调函数中（RenderScenceCB）我们可以自定义代码来渲染这一帧的图像。
    
(7)
`glClear(GL_COLOR_BUFFER_BIT);`

`glutSwapBuffers();`

在渲染函数中我们能做的就是清空帧缓存（使用上面定义的颜色，可以尝试任意改变颜色看效果）。第二个函数是告诉GLUT交换双缓冲机制中前后两个缓存的角色位置，也就是二者换班，后台的缓存放到前台显示，之前显示的缓存继续到后台开始另一帧的缓存工作。这样在下一个渲染回调循环中交换到当前的缓存将在屏幕上显示。


### ***示例Demo***

```

#include <iostream>
#include <GLUT/freeglut.h>//freeGLUT窗口库

/**
 * 渲染回调函数
 */
void RenderScenceCB(){
    // 清空颜色缓存
    glClear(GL_COLOR_BUFFER_BIT);
    // 交换前后缓存
    glutSwapBuffers();
}

/**
 * 主函数
 */
int main(int argc, char ** argv) {
    
    // 初始化GLUT
    glutInit(&argc, argv);
    
    // 显示模式：双缓冲、RGBA
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);
    
    // 窗口设置
    glutInitWindowSize(480, 320);      // 窗口尺寸
    glutInitWindowPosition(100, 100);  // 窗口位置
    glutCreateWindow("Tutorial 01");   // 窗口标题
    
    // 开始渲染
    glutDisplayFunc(RenderScenceCB);
    
    // 缓存清空后的颜色值
    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
    
    // 通知开始GLUT的内部循环
    glutMainLoop();
  
    return 0;
}

```

### **运行结果：**
![这里写图片描述](http://img.blog.csdn.net/20160913130322027)


**说明:**
**demo程序是原作者的代码，适用于多平台，由于在Mac上比较特殊，我在Mac上跑的时候发现GLEW有点问题暂时没解决，导致后面的三角形啥的在mac上没效果。orz......最后我放弃了在mac用opengl了，坑实在太多，装了双系统在windows上安装vs2013来跑原作者的源代码，就顺利多了。**

**所以强烈建议还是在visual studio上跑opengl吧！学习教程之前配置好开发环境，边看边跑代码效果最好，这里我的环境主要包括：**

**1.Visual Studio2013；**

**2.OpenGL，windows默认已内置了OpenGL可直接引入无需再安装（说是微软为了推广自己的DX只提供一个很低的版本，但我的默认就是最新版本，4.3呢！！！）；**

**3.为了和作者教程一致，安装freeGLUT窗口库以及GLEW扩展库，安装方法见文章开头链接；**
