---
layout: post
title:  "一步步学OpenGL(4) -《着色器》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-14
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程4：***

# **着色器**

**原文：**  [http://ogldev.atspace.co.uk/www/tutorial04/tutorial04.html](http://ogldev.atspace.co.uk/www/tutorial04/tutorial04.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景：***

从这篇教程开始，我们将使用shader着色器来实现每一个效果和技术点。着色器是目前做3D图形最流行的方式。在某种程度上我们可以说这是一个“退步”吧，或者说技术实现上的一个回退，因为本来多数固定功能管线提供的那些3D功能（开发者只需要定义配置参数即可实现的功能函数）现在开发者必须自己通过shader着色器来实现，然而同时，这个可编程性也使我们的开发更加灵活和具有创新性。

**PS：理解这篇文章的介绍，对OpenGL的渲染管线整个流程就很清晰了！**

OpenGL的可编程管线可以通过下面的图示来表示：


**顶点处理器—>几何处理器—>裁剪器—>光栅器（片段处理器）**

![可编程管线](http://img.blog.csdn.net/20160910140257317)

**Vertex Processor顶点处理器阶段**负责执行处理经过管线的每一个顶点的vertex shader顶点着色代码（顶点的数量取决于draw call的参数）。顶点着色器还并不知道渲染的图元的topology拓扑结构是怎样的。另外你不可以在顶点着色处理器阶段删除丢弃顶点。每个顶点有且只有一次经过顶点处理器，经过变换后继续进入管线的下一步。

下一个阶段是**Geometry Processor几何处理器阶段**。在这个阶段，图元的完整数据（比如：所有的顶点数据）和相邻顶点的数据全部都提供给**shader着色器**，这样可以使其必须考虑除了顶点本身的其他的一些额外的更全面完整的信息。几何处理器还可以将输出的图形拓扑结构转换成在draw call中选择的另一种结构。比如：你可以提供一系列的点来产生由两个三角形所构成的图形（像四边形），也就是顶点链接成三角形图元，两个三角形图元可以构成四边形（一种叫做**billboarding公告板技术**的技术）。另外，你也可以直接给几何着色器提供多个顶点然后根据你选择的输出拓扑结构产生多种图元。

在管线的下一个阶段就要开始**clip裁剪**工作了。这是一个固定功能单元的很明确的一个任务：像前面教程中一样它将所有图元裁剪到那个单位化的盒子模型内，它另外还会将图元裁剪在Z轴的远近平面范围内（也就是说太远或太近都不显示）。同时也提供用户自定义的裁剪平面进行自定义的裁剪。经过裁剪后保留下来的顶点现在会被映射到屏幕空间坐标系上，光栅器将会根据他们的拓扑结构把他们渲染到屏幕上。举个例子：对于三角形的裁剪就是发现三角形内部所有可见的点，对每个点**rasterizer光栅器**都会触发**fragment processor片段处理器**，现在你可以对每个片段像素定义颜色，颜色可以从一张材质上取或者使用其他取色技术方式。

上面这三个可编程阶段**（vertex processor顶点处理、geometry processor几何处理和fragment processor片段处理）**都是可选的而不是必须的。如果你不在这几个处理器上绑定你自己的shaer着色器，那么就会执行一些默认的函数功能，也就是备胎着色器。

Shader着色器的使用跟C/C++程序的创建过程类似。首先你要写一个shader着色器文本并使其在你的程序中有效可用，这个过程可以通过依次简单的引用这些源码脚本或者从外部文件中加载，注意都是以字符串数组的形式。然后一个个的编译这些shader文本成shader对象。然后你就可以将这些shader着色器连接到单个程序中并加载到GPU中。链接这些shader可以使驱动器能够有机会精减这些shader并根据他们的关系优化他们。例如：可能一个顶点着色器发出的法向量在相应的片段着色器阶段中被忽视，这样驱动中的**GLSL编译器**就会移除着色器中与这个法向量相关的函数功能从而更快的执行这个顶点着色器。如果之后那个着色器又匹配了需要用到那个法向量的片段着色器，然后连接到其他程序后会产生一个不同的顶点着色器。


### ***源代码详解***
(1)`GLuint ShaderProgram = glCreateProgram();`

我们通过创建程序对象来**建立shader着色器程序**。我们将把所有的着色器连接到这个对象上。

(2)`GLuint ShaderObj = glCreateShader(ShaderType);`

使用上面的函数**创建两个shader着色器对象**。其中一个使用的ShaderType为GL_VERTEX_SHADER，另一个的类型为GL_FRAGMENT_SHADER。这两个着色器对象的shader脚本源定义和他们的编译方式是一样的。

(3)
`const GLchar* p[1];`

`p[0] = pShaderText;`

`GLint Lengths[1];`

`Lengths[0]= strlen(pShaderText);`

`glShaderSource(ShaderObj, 1, p, Lengths);`

**在编译shader对象之前我们必须先定义它的代码源。**函数glShaderSource以shader对象为参数，使你可以灵活的定义代码来源。shader源代码（也就是我们所常说的shader脚本）可以由多个字符串数组排布组合而成，你需要提供一个指针数组来对应指向这些字符窜数组，同时要提供一个整型数组来对应表示每个数组的长度。为了简单，我们这里只使用一个字符串数组来保存所有的shader源代码，并且分别用数组的一个元素来分别指向这个字符串数组和表示数组的长度。
第二个参数表示的是这两个数组的元素个数（我们的例子中则只有1个）。

(4)`glCompileShader(ShaderObj);`

编译shader对象是非常简单的，只要一句话即可。。。

(5)`GLint success;`

`glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);`

`if (!success) {`

`    GLchar InfoLog[1024];`

`    glGetShaderInfoLog(ShaderObj, sizeof(InfoLog), NULL, InfoLog);`

`    fprintf(stderr, "Error compiling shader type %d: '%s'\n", ShaderType, InfoLog);
}`

。。。然而按照期望，你通常只能遇到很少的编译错误。使用上面的代码块可以获得编译状态，并且可以打印编译器碰到的所有编译错误。

(6)`glAttachShader(ShaderProgram, ShaderObj);`

最后，我们**将编译好的shader对象绑定在program object程序对象上**。这和定义一系列对象然后在makefile中连接类似。由于这里没有makefile所以要通过编程来实现连接绑定。只有绑定的对象才会加入到连接过程中。

(7)`glLinkProgram(ShaderProgram);`

**编译好所有的shader对象并将他们绑定到程序中后我就可以连接他们了。**注意在完成程序的连接后你可以通过调用函数glDetachShader和glDeleteShader来清除每个中介shader对象。OpenGL保存着由它产生的多数对象的引用计数，如果一个shader对象被创建后又被删除的话驱动程序也会同时清除掉它，但是如果他被绑定在程序上，只调用glDeleteShader函数只是会标记它等待删除，只有等你调用glDetachShader后它的引用计数才会被置零然后被移除掉。

(8)
`glGetProgramiv(ShaderProgram, GL_LINK_STATUS, &Success);`

`if (Success == 0) {`

`    glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);`

`    fprintf(stderr, "Error linking shader program: '%s'\n", ErrorLog);
}`

注意我们检查和程序相关的错误（比如link链接错误）和shader着色器相关的错误是不太一样的。我们**使用glGetProgramiv而不是glGetShaderiv，glGetProgramInfoLog而不是glGetShaderInfoLog**。

(9)`glValidateProgram(ShaderProgram);`

你也许会问为什么我们在程序成功link后还要再验证它。不同的地方是link错误的检查是和shader结合在一起的，而上面这句代码检查的是在**当前的管线状态程序是否可以被执行**。在一个有大量shader和状态变化的应用中，最好在每次draw call之前都进行验证。在我们这个简单的应用中我们只检查一次。你也想只在开发的时候做这个检查，避免最终这个项目产品的开销。

(10)`glUseProgram(ShaderProgram);`

最后，**要使用连接好的shader程序你需要用上面的回调函数将它设置到管线声明中**。这个程序将在所有的draw call中一直生效直到你用另一个替换掉它或者使用glUseProgram指令将其置NULL明确地禁用它。如果你创建的shader程序只包含一种类型的shader（只是为某一个阶段添加的自定义shader），那么在其他阶段的该操作将会使用它们默认的固定功能操作。

到现在我们已经完成了OpenGL中和shader相关操作的准备工作的学习。在这篇教程之后的内容主要是关于顶点着色器和片段着色器（包含在pVS和pFS脚本中）。

(11)`#version 330`

这个告诉编译器我们的**目标GLSL编译器版本是3.3**.如果编译器不支持这个版本会抛出一个错误。

(12)`layout (location = 0) in vec3 Position;`

这段语法代码会出现在**顶点着色器**中，它表示声明一个含有3个浮点数的向量的具体顶点属性将在shader中被认为是一个 **'Position' 位置**。具体的顶点是说对于在GPU中每一个调用的shader，从缓冲区获得的每一个新顶点的值都会被提供。
代码的第一部分： layout (location = 0)，将缓冲区中的属性名和属性绑定在一起，当我们的顶点有多个属性（位置，法向量，纹理坐标等等）的时候这个是必须要指明的。我们必须要让编译器知道缓冲区中顶点的哪个属性要和shader中声明的哪个属性进行映射匹配，有两种办法可以做到：
（1）我们可以像这里我们所做的一样明确设置它（为0），那样我们就可以在我们的应用中使用一个写死的编码值（使用第一个参数调用glVertexAttributePointer）；
（2）或者我们可以不管它（简单的在shader中声明'in vec3 Position'）然后使用glGetAttribLocation检索应用中这个实时的位置。在那种情况下，我们还需要为glVertexAttributePointer提供返回值，而不是使用写死的编码值。这里我们选择简单的方式，但是在复杂的应用中最好让编译器定义属性的索引标志然后实时检索他们。这样可以更容易的从多个来源整合shader着色器而不用使他们适应你的缓冲器布局。

(13)`void main()`

你可以通过连接多个shader对象来创建你的着色器。然而，每一个着色器阶段（VS顶点着色,GS几何着色,FS片段着色）有且只有一个main函数作为**shader着色器的入口**。例如：你可以创建一个拥有多功能的光照库并和你的shader连接，但是所有的函数都不可以命名为‘main’。

(14)`gl_Position = vec4(0.5 * Position.x, 0.5 * Position.y, Position.z, 1.0);`

这里我们对引入的**顶点位置做固定编码转换**，我们让X和Y的值减半并保持Z的值不变。
'gl_Position' 是一个内置的变量，用来保存同类的（包含X,Y,Z和W元素）顶点位置。光栅器将会查找那个变量并用它作为屏幕空间（跟随一些更多的变换）的位置。将X和Y的值减半意味着让三角形的尺寸变成之前教程中三角形的四分之一。注意我们要设置W的值为1.0，这个对三角形的正确显示极其重要。将物体从3d投影到2d事实上是经过两个独立的阶段完成的。首先，你要使所有的顶点乘以投影变换矩阵（这个会在一些教程中建立），然后，在顶点到达光栅器之前GPU会自动对位置属性进行所谓的‘透视分割’，这意味着它将使用W分量分割gl_Position的所有其他分量元素。在这个教程中我们还没有在顶点着色器中进行任何的投影变换，但是这个‘透视分割’阶段是我们无法禁止的。无论我们从顶点着色器输出什么样的gl_Position值都将被HW使用它的W分量所分割，我们需要注意这个否则我们无法得到我们期望的结果。为了规避‘透视分割’的影响我们设置W为1.0。被1.0分割不会影响顶点位置中其他分量的值从而顶点可以仍然在我们的单位化的盒子模型内部。

如果一切工作运行正确，顶点 (-0.5, -0.5), (0.5, -0.5) 和 (0.0, 0.5)将会到达光栅器，裁剪器无须作任何事情，因为所有的顶点都在单位化盒子模型内部。这些值将会被映射到屏幕空间坐标系，并且光栅器开始处理三角形内部的所有点，对于每个点片段着色器都会被执行。
下面的代码开始来自于片段着色器。

(15)`out vec4 FragColor;`

通常片段着色器的工作就是定义每个片段（像素）的颜色。另外，片段着色器可以抛弃所有的像素或者改变它的Z值（这会影响后面Z test的结果）。**输出的颜色**是通过上面的代码中声明的变量完成的。4个分量代表**R,G,B和A（透明度）**。你在这些变量上设置的值将会被光栅器接收最后写入帧缓冲中。

(16)`FragColor = vec4(1.0, 0.0, 0.0, 1.0);`

In the previous couple of tutorials there wasn't a fragment shader so the everything was drawn in the default color of white. Here we set FragColor to red.
在前面的教程中都没有**片段着色器**所以所有的绘制都默认使用白色。这里我们可以设置FragColor为红色。

### ***示例Demo***

```

#include <stdio.h>
#include <string.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h" //这里要添加作者的工具类用于读取文本文件
#include "ogldev_math_3d.h"

GLuint VBO;

// 定义要读取的顶点着色器脚本和片断着色器脚本的文件名，作为文件读取路径（这样的话shader.vs和shader.fs文件要放到工程的根目录下，保证下面定义的是这两个文件的文件路径）
const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";

static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

	// 依然还是绘制一个三角形
    glDrawArrays(GL_TRIANGLES, 0, 3);

    glDisableVertexAttribArray(0);

    glutSwapBuffers();
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
}

static void CreateVertexBuffer()
{
    Vector3f Vertices[3];
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
    Vertices[1] = Vector3f(1.0f, -1.0f, 0.0f);
    Vertices[2] = Vector3f(0.0f, 1.0f, 0.0f);

 	glGenBuffers(1, &VBO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
}

// 使用shader文本编译shader对象，并绑定shader都想到着色器程序中
static void AddShader(GLuint ShaderProgram, const char* pShaderText, GLenum ShaderType)
{
	// 根据shader类型参数定义两个shader对象
    GLuint ShaderObj = glCreateShader(ShaderType);
	// 检查是否定义成功
    if (ShaderObj == 0) {
        fprintf(stderr, "Error creating shader type %d\n", ShaderType);
        exit(0);
    }

	// 定义shader的代码源
    const GLchar* p[1];
    p[0] = pShaderText;
    GLint Lengths[1];
    Lengths[0]= strlen(pShaderText);
    glShaderSource(ShaderObj, 1, p, Lengths);
    glCompileShader(ShaderObj);// 编译shader对象

	// 检查和shader相关的错误
    GLint success;
    glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);
    if (!success) {
        GLchar InfoLog[1024];
        glGetShaderInfoLog(ShaderObj, 1024, NULL, InfoLog);
        fprintf(stderr, "Error compiling shader type %d: '%s'\n", ShaderType, InfoLog);
        exit(1);
    }

	// 将编译好的shader对象绑定到program object程序对象上
    glAttachShader(ShaderProgram, ShaderObj);
}

// 编译着色器函数
static void CompileShaders()
{
	// 创建着色器程序
    GLuint ShaderProgram = glCreateProgram();
	// 检查是否创建成功
    if (ShaderProgram == 0) {
        fprintf(stderr, "Error creating shader program\n");
        exit(1);
    }
    
    // 存储着色器文本的字符串缓冲
    string vs, fs;
    // 分别读取着色器文件中的文本到字符串缓冲区
    if (!ReadFile(pVSFileName, vs)) {
        exit(1);
    };
    if (!ReadFile(pFSFileName, fs)) {
        exit(1);
    };

	// 添加顶点着色器和片段着色器
    AddShader(ShaderProgram, vs.c_str(), GL_VERTEX_SHADER);
    AddShader(ShaderProgram, fs.c_str(), GL_FRAGMENT_SHADER);

	// 链接shader着色器程序，并检查程序相关错误
    GLint Success = 0;
    GLchar ErrorLog[1024] = { 0 };
    glLinkProgram(ShaderProgram);
    glGetProgramiv(ShaderProgram, GL_LINK_STATUS, &Success);
	if (Success == 0) {
		glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
		fprintf(stderr, "Error linking shader program: '%s'\n", ErrorLog);
        exit(1);
	}

	// 检查验证在当前的管线状态程序是否可以被执行
    glValidateProgram(ShaderProgram);
    glGetProgramiv(ShaderProgram, GL_VALIDATE_STATUS, &Success);
    if (!Success) {
        glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
        fprintf(stderr, "Invalid shader program: '%s'\n", ErrorLog);
        exit(1);
    }

	// 设置到管线声明中来使用上面成功建立的shader程序
    glUseProgram(ShaderProgram);
}

// 主函数
int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGB);
    glutInitWindowSize(1024, 768);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 04");

    InitializeGlutCallbacks();

    // 必须在glut初始化后！
    GLenum res = glewInit();
    if (res != GLEW_OK) {
      fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
      return 1;
    }
    
    printf("GL version: %s\n", glGetString(GL_VERSION));

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

    CreateVertexBuffer();

	// 编译着色器
    CompileShaders();

    glutMainLoop();

    return 0;
}

```

**顶点着色器shader.vs脚本代码：**

```

#version 330  //告诉编译器我们的目标GLSL编译器版本是3.3

layout (location = 0) in vec3 Position; // 绑定定点属性名和属性，方式二缓冲属性和shader属性对应映射

void main()
{
    gl_Position = vec4(0.5 * Position.x, 0.5 * Position.y, Position.z, 1.0); // 为glVertexAttributePointer提供返回值
}

```

**片段着色器shader.fs脚本代码：**

```

#version 330  //告诉编译器我们的目标GLSL编译器版本是3.3

out vec4 FragColor;  // 片段着色器的输出颜色变量

// 着色器的唯一入口函数
void main()
{
    // 定义输出颜色值
    FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}

```


### **运行效果图**

![这里写图片描述](http://img.blog.csdn.net/20160913140438134)