---
layout: post
title:  "一步步学OpenGL(10) -《索引绘制》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-17
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程10***

# **索引绘制**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial10/tutorial10.html](http://ogldev.atspace.co.uk/www/tutorial10/tutorial10.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

OpenGL提供了几个绘制函数，其中我们之前使用的glDrawArrays()属于**顺序绘制**的一个函数。顺序绘制是从指定的偏移量依次扫描顶点缓冲区所有图元的每一个顶点，这样很简单易用，但是缺点是如果一个顶点是多个图元的共同顶点，那么这个顶点将会在顶点缓冲区出现多次，也就是没有顶点共享的概念。顶点共享是通过**索引绘制类的函数**来实现的。这里除了顶点缓冲器额外需要有一个**索引缓冲**，索引缓冲中存储着顶点缓冲中的顶点的索引值。扫描索引缓冲和扫描顶点缓冲类似：每一个X索引按照图元的顶点依次排放。顶点共享就是对于多个图元出现的共同的一个顶点，只要在索引缓冲中重复顶点在顶点缓冲中的索引即可，不需要将这个顶点在顶点缓冲存储多次。共享对于内存的高效利用非常重要，因为多数物体都是通过一些由三角形图元组成的封闭的网格表示的，而多数顶点都会在多个三角形图元中出现作为三角形的其中一个顶点。

这里是一个顺序绘制的例子：

![这里写图片描述](http://img.blog.csdn.net/20160917174921782)

对于上面的顶点缓冲，如果我们渲染三角形图元的话，GPU会产生下面的三角形顶点集：V0/1/2, V3/4/5, V6/7/8等。

然后下面是一个索引绘制的例子：

![这里写图片描述](http://img.blog.csdn.net/20160917175137550)

在这种情况中，GPU将产生下面的三角形：V2/0/1, V5/2/4, V6/5/7等等。

在OpenGL中使用索引绘制需要创建并维护一个索引缓冲，这个索引缓冲必须要在draw call之前绑定到顶点缓冲中，并且需要使用不同的API接口函数。

### ***源代码详解***

(1)`GLuint IBO;`

为索引缓冲添加另一个缓冲对象引用句柄。

(2)

`Vertices[1] = Vector3f(0.0f, -1.0f, 1.0f);`

`Vertices[2] = Vector3f(1.0f, -1.0f, 0.0f);`

`Vertices[3] = Vector3f(0.0f, 1.0f, 0.0f);`

为了展示顶点共享我们需要一个更加复杂一点的网格（多个三角形图元拼成的）。很多教程使用著名的旋转立方体来实现，那个需要8个顶点和12个三角形图元。由于我比较懒所以我使用旋转的金字塔锥形体来代替，这个只需要四个顶点和四个三角形图元，而且更加容易手动设置... 

当从顶部（沿着Y轴）俯视看这些顶点，我们可以看到下面的顶点布局：

![这里写图片描述](http://img.blog.csdn.net/20160917180407508)

(3)`unsigned int Indices[] = `

`{ 0, 3, 1,`

  ` 1, 3, 2,`

  `0, 1, 2 };`
  
  索引缓冲使用一个索引数组来布置，这些索引和顶点缓冲中顶点的位置相匹配。通过看这个数组和上面的顶点布局图可以知道数组最后一个三角形作为金字塔的底，其他三个三角形构成金字塔的表面。这个金字塔虽然不对称，但是很容易定义构造。

(4)`glGenBuffers(1, &IBO);`

`glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);`

`glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(Indices), Indices, GL_STATIC_DRAW);`

我们使用索引数组创建并设置好索引缓冲后，可以发现和创建顶点缓冲唯一的不同是，**顶点缓冲使用的GL_ARRAY_BUFFER参数表示缓冲的类型，而索引缓冲类型使用的是GL_ELEMENT_ARRAY_BUFFER**。

(5)`glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);`

除了要绑定顶点缓冲我们必须还要在绘制之前绑定所索引缓冲，再次重复，使用的是GL_ELEMENT_ARRAY_BUFFER作为缓冲器类型。

(6)`glDrawElements(GL_TRIANGLES, 12, GL_UNSIGNED_INT, 0);`

**这里的索引绘制我们使用的函数是glDrawElements而不是glDrawArrays。**第一个参数是要渲染的图元的类型(和glDrawArrays一样)；第二个参数是索引缓冲中用于产生图元的索引个数；后面的参数是每一个索引值的数据类型。必须要告诉GPU单个索引值的大小，否则GPU无法知道如何解析索引缓冲区。索引值可用的类型主要有：GL_UNSIGNED_BYTE, GL_UNSIGNED_SHORT, GL_UNSIGNED_INT。如果索引的范围很小应该选择小的数据类型来节省空间，如果索引范围很大自然就要根据需要选择更大的数据类型；最后一个参数告诉GPU从索引缓冲区起始位置到到第一个需要扫描的索引值得偏移byte数，这个在使用同一个索引缓冲来保存多个物体的索引时很有用，通过定义偏移量和索引个数可以告诉GPU去渲染哪一个物体，在我们的例子中我们从一开始扫描所以定义偏移量为0。注意最后一个参数的类型是GLvoid*，所以如果不是0的话必须要转换成那个参数类型。

### ***示例Demo***

```

#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_math_3d.h"

GLuint VBO;
// 索引缓冲对象的句柄
GLuint IBO;
GLuint gWorldLocation;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.01f;

    Matrix4f World;
    
    World.m[0][0] = cosf(Scale); World.m[0][1] = 0.0f; World.m[0][2] = -sinf(Scale); World.m[0][3] = 0.0f;
    World.m[1][0] = 0.0;         World.m[1][1] = 1.0f; World.m[1][2] = 0.0f        ; World.m[1][3] = 0.0f;
    World.m[2][0] = sinf(Scale); World.m[2][1] = 0.0f; World.m[2][2] = cosf(Scale) ; World.m[2][3] = 0.0f;
    World.m[3][0] = 0.0f;        World.m[3][1] = 0.0f; World.m[3][2] = 0.0f        ; World.m[3][3] = 1.0f;

    glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, &World.m[0][0]);

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

    // 每次在绘制之前绑定索引缓冲
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);
    // 索引绘制图形
    glDrawElements(GL_TRIANGLES, 12, GL_UNSIGNED_INT, 0);

    glDisableVertexAttribArray(0);

    glutSwapBuffers();
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
    glutIdleFunc(RenderSceneCB);
}

static void CreateVertexBuffer()
{
    // 金字塔的四个顶点
    Vector3f Vertices[4];
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
    Vertices[1] = Vector3f(0.0f, -1.0f, 1.0f);
    Vertices[2] = Vector3f(1.0f, -1.0f, 0.0f);
    Vertices[3] = Vector3f(0.0f, 1.0f, 0.0f);

 	glGenBuffers(1, &VBO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
}

// 创建索引缓冲器
static void CreateIndexBuffer()
{
    // 四个三角形面的顶点索引集
    unsigned int Indices[] = { 0, 3, 1,
                               1, 3, 2,
                               2, 3, 0,
                               0, 1, 2 };
    // 创建缓冲区
    glGenBuffers(1, &IBO);
    // 绑定缓冲区
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);
    // 添加缓冲区数据
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(Indices), Indices, GL_STATIC_DRAW);
}

static void AddShader(GLuint ShaderProgram, const char* pShaderText, GLenum ShaderType)
{
    GLuint ShaderObj = glCreateShader(ShaderType);

    if (ShaderObj == 0) {
        fprintf(stderr, "Error creating shader type %d\n", ShaderType);
        exit(1);
    }

    const GLchar* p[1];
    p[0] = pShaderText;
    GLint Lengths[1];
    Lengths[0]= strlen(pShaderText);
    glShaderSource(ShaderObj, 1, p, Lengths);
    glCompileShader(ShaderObj);
    GLint success;
    glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);
    if (!success) {
        GLchar InfoLog[1024];
        glGetShaderInfoLog(ShaderObj, 1024, NULL, InfoLog);
        fprintf(stderr, "Error compiling shader type %d: '%s'\n", ShaderType, InfoLog);
        exit(1);
    }

    glAttachShader(ShaderProgram, ShaderObj);
}

static void CompileShaders()
{
    GLuint ShaderProgram = glCreateProgram();

    if (ShaderProgram == 0) {
        fprintf(stderr, "Error creating shader program\n");
        exit(1);
    }
	
    string vs, fs;

    if (!ReadFile(pVSFileName, vs)) {
        exit(1);
    };

    if (!ReadFile(pFSFileName, fs)) {
        exit(1);
    };

    AddShader(ShaderProgram, vs.c_str(), GL_VERTEX_SHADER);
    AddShader(ShaderProgram, fs.c_str(), GL_FRAGMENT_SHADER);

    GLint Success = 0;
    GLchar ErrorLog[1024] = { 0 };

    glLinkProgram(ShaderProgram);
    glGetProgramiv(ShaderProgram, GL_LINK_STATUS, &Success);
	if (Success == 0) {
		glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
		fprintf(stderr, "Error linking shader program: '%s'\n", ErrorLog);
        exit(1);
	}

    glValidateProgram(ShaderProgram);
    glGetProgramiv(ShaderProgram, GL_VALIDATE_STATUS, &Success);
    if (!Success) {
        glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
        fprintf(stderr, "Invalid shader program: '%s'\n", ErrorLog);
        exit(1);
    }

    glUseProgram(ShaderProgram);

    gWorldLocation = glGetUniformLocation(ShaderProgram, "gWorld");
    assert(gWorldLocation != 0xFFFFFFFF);
}

int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGBA);
    glutInitWindowSize(1024, 768);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 10");

    InitializeGlutCallbacks();

    // Must be done after glut is initialized!
    GLenum res = glewInit();
    if (res != GLEW_OK) {
      fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
      return 1;
    }
    
    printf("GL version: %s\n", glGetString(GL_VERSION));

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

    CreateVertexBuffer();
    CreateIndexBuffer();
    
    CompileShaders();

    glutMainLoop();

    return 0;
}

```

***着色器脚本的代码相比上个教程没有变化***

***shader.vs***

```
#version 330

layout (location = 0) in vec3 Position;

uniform mat4 gWorld;

out vec4 Color;

void main()
{
    gl_Position = gWorld * vec4(Position, 1.0);
    Color = vec4(clamp(Position, 0.0, 1.0), 1.0);
}
```

***shader.fs***

```

#version 330

in vec4 Color;

out vec4 FragColor;

void main()
{
    FragColor = Color;
}

```

### ***运行效果***

绕Y轴旋转的彩色金字塔。

![这里写图片描述](http://img.blog.csdn.net/20160917222450532)