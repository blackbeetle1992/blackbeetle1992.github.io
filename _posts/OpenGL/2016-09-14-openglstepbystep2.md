---
layout: post
title:  "一步步学OpenGL(2) -《你好，顶点》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-14
categories: [OpenGL]
tags: [opengl]
icon: 

---

# **你好顶点**

*** 原文: ***  [http://ogldev.atspace.co.uk/www/tutorial02/tutorial02.html](http://ogldev.atspace.co.uk/www/tutorial02/tutorial02.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***


### ***背景***

这里要第一次开始使用**GLEW（the OpenGL Extension Wrangler Library）**库。GLEW可以帮助我们解决一些伴随OpenGL扩展库管理出现的一些头疼的问题，初始化之后，它会检索你平台中所有可用的扩展库，动态的加载并且可以通过简单引用一个头文件来使用。

在这个教程中，我们将第一次使用**定点缓冲器对象（VBOs）**。顾名思义，VBO是用来存储顶点的。我们试图在屏幕上显示的存在于3d世界中的物体，像一个怪物、城堡或者一个简单的旋转的立方体，都是通过连接一组顶点来实现的。VBOs是在GPU中加载顶点最有效的方式，他们是可以存储在视频内存的缓冲并且可迅速到达GPU处理，所以强烈推荐这种顶点加载方式。

这篇教程和下一篇教程是在这一系列中唯一依赖于固定功能管线的而不是可编程管线，事实上在这所有教程中也没有出现这两种管线的转换，我们只是依靠数据流经管线的方式。在接下来的教程中将会有关于管线的透彻学习，而现在已经足够可以理解在到光栅化（在屏幕上使用屏幕坐标画点、线、三角形等图元）之前，这些可见的顶点都有他们的XYZ坐标（[-1.0，1.0]），光栅化程序将这些坐标映射到屏幕空间（例如：如果屏幕宽度是1024，那么X=-1.0就映射到0，X=1.0映射到1023）。最后，光栅化程序根据在draw call（下面代码中会讲到）中定义的拓扑结构来绘制这些图元。由于我们没有将任何shader着色器绑定到渲染管线上，我们的顶点也就没有经过任何变换。也就是说，我们只是给这些顶点一个给定范围的值来使他们可见。将X和Y坐标都设置为0可将顶点精确地至于两个坐标轴中间，也就是屏幕的中央。

### ***安装GLEW：***

GLEW可在官网下载：[http://glew.sourceforge.net](http://glew.sourceforge.net) 
大多数Linux发行版提供预先构建的包。在Ubuntu上可以通过下面的指令安装：

`apt-get install libglew1.6 libglew1.6-dev`

**Mac上安装GLEW的教程(需要安装MacPort):** [http://blog.csdn.net/huyisu/article/details/42742379](http://blog.csdn.net/huyisu/article/details/42742379)

**VisualStudio安装GLEW：** [http://blog.csdn.net/xuguangsoft/article/details/8002375](http://blog.csdn.net/xuguangsoft/article/details/8002375)


### ***源代码详解***


(1)`#include <GL/glew.h>`

GLEW库引入(一定要在GLUT引入之前引入，否则会编译错误),如果要引入其他OpenGL头文件，必须要注意将这个头文件放在前面。为了将项目与GLEW进行连接需要在makefile中添加‘-lGLEW’(Mac上这个在GLEW的安装教程上有说明，需要在Building Setting中设置).

(2)`#include "math3d.h"`

这个用于OpenGL的3d数学库可到网上自行下载，版本不一样可能变量名和变量初始化函数会不一样, 但使用方法都一样。也可以下载原作者的源码，里面也有这个头文件。
在这个教程中我们开始使用像向量这种辅助数据结构，并且慢慢我们会扩展这个头文件.

(3)
`GLenum res = glewInit();`

`    if (res != GLEW_OK)`

`
    {
        fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
    }`
    
这里初始化GLEW并检查是否有错误，这个必须在GLUT初始化之后完成.
    
(4)
`Vector3f Vertices[1];`
    
`Vertices[0] = Vector3f(0.0f, 0.0f, 0.0f);`
    
这里创建一个Vector3f结构的数组，并初始化XYZ坐标为0。这样使该点显示在屏幕中央。
    
(5)`GLuint VBO;`

这里在项目中定义一个全局的GLuint引用变量，来操作顶点缓冲器对象。后面会看到绝大多数OpenGL对象都是通过GLuint类型的变量来引用的.
    
(6)`glGenBuffers(1, &VBO);`

OpenGL定义了几个glGen*前缀的函数来产生不同类型的对象。它们通常有两个参数：第一个参数用来定义你想创建的对象的数量，第二个参数是一个GLuint变量的数组的地址，来存储分配给你的引用变量handles（要确保这个数组足够大来处理你的请求！）。以后对这个函数的调用将不会重复产生相同的handle对象，除非你先使用glDeleteBuffers删除他们。注意你不需要在buffer中定义你具体想要做的事情，将其一般化、通用化，具体的工作由下一步来完成。
    
(7)`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

OpenGL使用handle的方式很独特，在很多API中handle可以提供给任何相关的函数并且具体的操作就是通过那个handle来操作完成，在OpenGL中我们需要将handle与一个目标的名称进行绑定，然后在该目标上执行命令。这些指令只会在与handle绑定的目标上生效直到另外有其他的对象跟这个handle绑定或者这个handle被置空。目标名GL_ARRAY_BUFFER意思是这个buffer将存储一个顶点的数组。另外一个有用的目标是GL_ELEMENT_ARRAY_BUFFER,这个的意思是这个buffer存储的是另一个buffer中顶点的标记。还有很多其他的目标，后面的教程中会看到。
    
(8)`glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);`

绑定了我们的对象之后，我们要往里面添加数据。这个回调函数取得我们之前绑定的目标名参数GL_ARRAY_BUFFER，还有数据的比特数参数，顶点数组的地址，还有一个表示这个数据模式的标志变量。因为我们不会去改变这个buffer的内容所以这里用了GL_STATIC_DRAW标志，相反的标志是GL_DYNAMIIC_DRAW, 这个只是给OpenGL的一个提示来给一些觉得合理的标志量使用，驱动程序可以通过它来进行启发式的优化（比如：内存中哪个位置最合适存储这个buffer缓冲）。
    
(9)`glEnableVertexAttribArray(0);`

在shader着色器教程中,可看到顶点着色器中使用的属性(位置、法线等)有索引来对它们进行映射,使你能够绑定C/C++程序中的数据和着色器中的属性名称，而且必须要为每一个顶点属性添加索引。在这个教程暂时不会使用任何着色器，但是我们加载到buffer中的顶点位置在固定功能管线中是被认为是索引为0的顶点属性（当没有着色器绑定时被启用）。你必须开启每一个顶点的属性，否则渲染管线无法获取这些数据。
    
(10)`glBindBuffer(GL_ARRAY_BUFFER, VBO);`

这里我们再次绑定我们的buffer准备开始draw call回调。在这个小程序中我们只有一个顶点的缓冲因此每一帧都调用这个回调是很冗余的，在更加复杂的程序中，将会有很多的buffer缓冲来存储不同的模型，你必须用将要调用的buffer来不断更新管线的状态。
    
(11)`glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);`

这个回调告诉管线怎样解析bufer中的数据。
第1个参定义了属性的索引，再这个例子中我们知道这个索引默认是0，但是当我们开始使用shader着色器的时候，我们既要明确的设置着色器中的属性索引同时也要检索它；
第2个参数指的是属性中的元素个数（3个表示的是：X,Y,Z坐标）；
第3个参数指的是每个元素的数据类型；
第4个参数指明我们是否想让我们的属性在被管线使用之前被单位化，我们这个例子中希望我们的数据保持不变的被传送；
第5个参数（称作’stride‘）指的是缓冲中那个属性的两个实例之间的比特数距离。当只有一个属性（例如：buffer只含有一个顶点的位置数据）并且数据被紧密排布的时候将该参数值设置为0。如果我们有一个包含位置和法向量（都是有三个浮点数的vector向量，一共6个浮点数）两个属性的数据结构的数组的时候，我们将设置参数值为这个数据结构的比特大小（6*4=24）；
最后一个参数在前一个例子中非常有用。我们需要在管线发现我们的属性的地方定义数据结构中的内存偏移值。在有位置数据和法向量数据的结构中，位置的偏移量为0，而法向量的偏移量则为12。
    
 (12)`glDrawArrays(GL_POINTS, 0, 1);`
 
最后，我们调用函数毁回调来绘制几何图形。之前所有的指令都非常重要，但它们只是设置了绘制指令的每一步的准备工作。这个指令才是GPU真正开始工作的地方。这个指令将整合这个指令收到的绘制参数和之前为这一个点的图形建立的状态数据来将结果渲染在屏幕上。

OpenGL提供了集中不同类型的draw call绘制回调，每一种各自适用于不同的案例情况。一般情况下可以将他们分成两类：顺序绘制和索引绘制。顺序绘制较简单，GPU经过你的顶点缓冲区，一个一个的挨着处理每一个顶点，并根据draw call中定义的拓扑结构来解析他们。
 
 例如：如果你定义了三角形GL_TRIANGLES，那么第0-2个顶点成为第一个三角形，第3-5个顶点成为第二个等等。如果你想让多个三角形共用同一个顶点你仍然要在缓冲区多次定义存储这个顶点，很浪费空间。
    
索引绘制相比顺序绘制更加复杂而且额外有一个索引缓冲区。索引缓冲区存储着顶点缓冲区中顶点的索引标志。GPU以和上面描述的类似的模式扫描索引缓冲区，索引0-2表示第一个三角形等等以此类推。如果两个三角形共用一个顶点只需要在索引缓冲区定义两次这个顶点的索引即可，顶点缓冲区只需要存储一个顶点数据。在游戏中索引绘制更常用，因为多数游戏模型是使用三角形图元来组成模型的表面（人的皮肤，城堡的墙等等），这些相连的三角形很多要共用一个顶点。
    
在这个教程中我们使用最简单的draw call：glDrawArrays。这是一个顺序绘制所以没有索引缓冲器。第一个参数我们定义拓扑结构为每一个顶点只表示一个点;下一个参数是第一个要绘制的顶点的索引，在我们这个例子中我们想从最开始的缓冲开始绘制，所以参数设置为0，但这也使我们能够在同一个缓冲区存储多个模型，然后根据它的偏移量选择其中一个进行绘制;最后一个参数是要绘制的顶点数。
    
 (13)`glDisableVertexAttribArray(0);`
 
当顶点短时间内不会被使用的时候及时禁用他们是个很好的习惯，当着色器不用他们的时候让他们可用无非是自找麻烦。

### 示例Demo

```

#include <stdio.h>
#include <GL/glew.h>        // GLEW扩展库
#include <GLUT/freeglut.h>  // freeGLUT图形库
#include "ogldev_math_3d.h" // 用于OpenGL的3d数学库,这里主要用到了顶点这个数据结构，下载原作者的源码可以找到这个头文件，
                            // 这里运行可能会报错找不到vector3、matrix3x3、matrix4x4以及作者的ogldev_util头文件（作者源代码内有提供，后面会加入），暂时先将报错的都注释掉即可

GLuint VBO;

/**
 * 渲染回调函数
 */
static void RenderScenceCB(){
    // 清空颜色缓存
    glClear(GL_COLOR_BUFFER_BIT);
    
    // 开启顶点属性
    glEnableVertexAttribArray(0);
    // 绑定GL_ARRAY_BUFFER缓冲器
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 告诉管线怎样解析bufer中的数据
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
    
    // 开始绘制几何图形(绘制一个点)
    glDrawArrays(GL_POINTS, 0, 1);
    
    //  禁用顶点数据
    glDisableVertexAttribArray(0);
    
    // 交换前后缓存
    glutSwapBuffers();
}

/**
 * 创建顶点缓冲器
 */
static void CreateVertexBuffer()
{
    // 创建含有一个顶点的顶点数组
    Vector3f Vertices[1];
    // 将点置于屏幕中央
    Vertices[0] = Vector3f(0.0f, 0.0f, 0.0f);
    
    // 创建缓冲器
    glGenBuffers(1, &VBO);
    // 绑定GL_ARRAY_BUFFER缓冲器
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 绑定顶点数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
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
    glutCreateWindow("Tutorial 02");   // 窗口标题
    
    // 开始渲染
    glutDisplayFunc(RenderScenceCB);
    
    // 检查GLEW是否就绪，必须要在GLUT初始化之后！
    GLenum res = glewInit();
    if (res != GLEW_OK) {
        fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
        return 1;
    }
    
    // 缓存清空后的颜色值
    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
    
    // 创建顶点缓冲器
    CreateVertexBuffer();
    
    
    // 通知开始GLUT的内部循环
    glutMainLoop();
  
    return 0;
}

```
### **运行效果**

![这里写图片描述](http://img.blog.csdn.net/20160913131327469)