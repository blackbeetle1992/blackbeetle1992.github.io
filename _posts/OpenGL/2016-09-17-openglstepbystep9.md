---
layout: post
title:  "一步步学OpenGL(9) -《插值》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-17
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程9***

# **插值**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial09/tutorial09.html](http://ogldev.atspace.co.uk/www/tutorial09/tutorial09.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

这个教程将介绍3d渲染管线中非常重要的一个部分：**光栅器**对从顶点着色器传来的变量的**插值**。通过之前的学习我们知道，为了让一些有意义东西在屏幕上真正显示，你必须将**顶点着色器vs**的输出变量设置为**'gl_Position'**，gl_Position是一个保存着顶点齐次坐标的4维向量。XYZ分量被W分量所分割（称作视角分割，这个是教程的重点话题）并且XYZ分量上超出单位化盒子([-1,1])的部分会被裁剪掉。最终的结果会被转换到屏幕坐标系然后三角形（或者其他支持的图元类型）被光栅器渲染到屏幕上。

光栅器在三角形的三个顶点之间进行插值（或者通过另外一种技术一行一行的插值）并执行片断着色器遍历三角形的每一个像素。片断着色器会返回光栅器存在颜色缓存中用于显示的像素颜色值（在其他一些额外的检测之后，比如：**深度测试depth test**等）。从顶点着色器传来的其他的变量不会经过上面的步骤。如果片断着色器没有显式地请求那个变量（你可以使用同一个顶点着色器混合并匹配多个片断着色器）那么一般的驱动优化会丢弃顶点着色器vs中只是影响该变量的操作（特定的shader程序是针对vs和fs的配对组合）。但如果片断着色器fs确实使用到了那个变量，光栅器会在光栅化阶段对其进行插值，并且每一次片断着色器fs的调用都会提供一个匹配特定位置的插值后的值，这意味着相邻的两个像素的值都略有不同（虽然随着三角形离摄像头越来越远那样会越来越不合适）。

***经常情况下依赖插值的两个变量是三角形的法向量和纹理坐标***。顶点的法向量通常是计算包含这个顶点的所有三角形法向量的平均值而得。如果物体不是平坦的话那么每个三角形的三个顶点的法向量会各不相同，那样我们可以通过插值来计算每个像素的法向量，那些向量会用于光线的计算，从而产生更逼真可信的光照效果。插值对于纹理坐标的应用也类似，这些坐标作为模型的一部分定义在每个顶点上。为了用贴图覆盖三角形你必须对每个像素进行一样的插值操作并给每个像素定义正确的纹理坐标，这些坐标都是插值的结果。

这个教程我们会对整个三角形插入不同的颜色来观察插值的效果。因为我比较懒所以我们就***在顶点着色器VS中来产生插值的颜色值***。另一个更单调的办法是在顶点缓冲中提供这些颜色值，但通情况下我们不会这样，你通常会***提供一个纹理坐标并从纹理上取样一个颜色值，那个颜色之后会被进行光照计算处理***。

### ***源代码详解***

(1)`out vec4 Color;`

在各渲染管线之间进行传递的参数必须在shader中使用‘out’保留字来全局的进行声明定义。颜色值是一个4维向量，因为XYZ分量记录RGB三个通道的颜色值，W分量表示像素的透明度值。

(2)`Color = vec4(clamp(Position, 0.0, 1.0), 1.0);`

在图形管线中颜色值通常使用一个在[0.0,1.0]范围内的浮点数来表示。这个值之后会映射匹配到每个颜色通道的中对应的0到255之间的整型数值（一共16M种颜色）。我们设置顶点的颜色值为以顶点位置为参数的一个函数，首先我们使用内置的clamp()函数保证值不会超出0.0-1.0的范围，原因是三角形的左下角位于(-1,-1)，如果我们使用原来那个值它会被光栅器插值，在X和Y的值低于0的时候我们将看不到任何东西，因为所有的值都小于等于0的时候会被渲染成黑色，这样的话图形各个方向上边缘的一半会是黑色。通过clamp()函数限制我们只是让左下角一个点为黑色并且随着远离这个颜色会迅速变亮。可以尝试去掉clamp函数或者改变它的参数来看效果。

clamp函数的输出结果不会直接作为输出变量，因为输出变量是一个4维向量，而位置position是一个3维向量（clamp不会改变分量的个数，只会改变分量的值）。在GLSL中没有一个默认的3维到4维的转换，因此这里我们要自己去明确定义它。我们**使用'vec4(vec3, W)' 来用3维向量和一个分量W值来构造一个4维向量**。这里我们使用值为1.0的W因为它会作为颜色值的透明度分量，这里我们希望每个像素都是完全不透明的。

(3)`in vec4 Color;`

在顶点着色器VS中的output输出颜色变量对应的另一边是片断着色器FS中的input输入的颜色变量。这个变量经过了光栅器的插值，所以每一个被执行的片断着色器FS可能都会是不同的颜色。

(4)`FragColor = Color;`

我们使用插值后得到的颜色作为片断着色器的颜色，没有更多其他的变化。

### ***示例Demo***

***源代码相对上个教程没有任何变化，修改的是shader.vs脚本和shader.fs脚本：***

```

#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_math_3d.h"

GLuint VBO;
GLuint gWorldLocation;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.001f;

    Matrix4f World;

    World.m[0][0] = sinf(Scale) ; World.m[0][1] = 0.0f       ; World.m[0][2] = 0.0f;        World.m[0][3] = 0.0f;
    World.m[1][0] = 0.0f        ; World.m[1][1] = sinf(Scale); World.m[1][2] = 0.0f;        World.m[1][3] = 0.0f;
    World.m[2][0] = 0.0f;       ; World.m[2][1] = 0.0f;      ; World.m[2][2] = sinf(Scale); World.m[2][3] = 0.0f;
    World.m[3][0] = 0.0f;       ; World.m[3][1] = 0.0f;      ; World.m[3][2] = 0.0f;        World.m[3][3] = 1.0f;

    glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, &World.m[0][0]);

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);

    glDrawArrays(GL_TRIANGLES, 0, 3);

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
    Vector3f Vertices[3];
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
    Vertices[1] = Vector3f(1.0f, -1.0f, 0.0f);
    Vertices[2] = Vector3f(0.0f, 1.0f, 0.0f);

 	glGenBuffers(1, &VBO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
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
    glutCreateWindow("Tutorial 09");

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

    CompileShaders();

    glutMainLoop();

    return 0;
}

```

***顶点着色器shader.vs脚本***

```

#version 330

layout (location = 0) in vec3 Position;

uniform mat4 gWorld;

// 顶点着色器输出的颜色值
out vec4 Color;

void main()
{
    gl_Position = gWorld * vec4(Position, 1.0);
    // 颜色插值
    Color = vec4(clamp(Position, 0.0, 1.0), 1.0);
}

```

***片段着色器shader.fs脚本***

```

#version 330

// 接受vs传来的插值后的颜色值
in vec4 Color;

out vec4 FragColor;

void main()
{
	// 颜色值作为片段着色器fs的输出
    FragColor = Color;
}

```

### ***运行效果***

尺寸上和上个教程的缩放效果一样，同时三角形有着从三个顶点往里的红绿黑颜色渐变效果，并随着三角形缩放动态缩放。

![这里写图片描述](http://img.blog.csdn.net/20160917222009795)
![这里写图片描述](http://img.blog.csdn.net/20160917222022723)