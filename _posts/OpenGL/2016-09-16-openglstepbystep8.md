---
layout: post
title:  "一步步学OpenGL(8) -《缩放变换》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-16
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程8***

# **缩放变换**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial08/tutorial08.html](http://ogldev.atspace.co.uk/www/tutorial08/tutorial08.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

缩放变换非常简单，它的目的是增大或者缩小物体的尺寸。比如你想使用同一个模型来制作很多不同的物体（大小不一的树组成的树林，用的同一个模型），或者你想按照比例让物体和现实世界尺寸一致。在上面的情形中你就需要在三个坐标轴上同等缩放顶点的位置。当然，有时也希望物体只在一个轴上或者两个轴上缩放使模型更薄、更瘦或者更高等等。

进行缩放变换其实很简单。我们从最开始的原变换矩阵来看，回忆平移变换矩阵的样子，我们保持结果矩阵中V1,V2和V3保持原样的办法是让变换矩阵主对角线上的值都为'1'，这样原向量一次都**和1相乘**之后依然保持不变，各分量之间互不影响。所以，这里的缩放变换，只要把那些‘1’换成我们想缩放的值，原向量各分量分别乘以这些值之后就会在相应坐标轴上进行相应的缩放了，值大于1则放大，值小于1则缩小。

![这里写图片描述](http://img.blog.csdn.net/20160916233755083)

![这里写图片描述](http://img.blog.csdn.net/20160916234847756)

### ***源代码详解***

`World.m[0][0]=sinf(Scale); World.m[0][1]=0.0f;        World.m[0][2]=0.0f;        World.m[0][3]=0.0f;`

`World.m[1][0]=0.0f;        World.m[1][1]=sinf(Scale); World.m[1][2]=0.0f;        World.m[1][3]=0.0f;`

`World.m[2][0]=0.0f;        World.m[2][1]=0.0f;        World.m[2][2]=sinf(Scale); World.m[2][3]=0.0f;`

`World.m[3][0]=0.0f;        World.m[3][1]=0.0f;        World.m[3][2]=0.0f;        World.m[3][3]=1.0f;`

和上个教程相比代码还是只是根据上面的描述改变了变换矩阵的内容。可以看到，这里我们使用一个-1到1之间的一个缩放值来使物体在各坐标轴上进行缩放，(0,1]范围内三角形会在原尺寸和极小尺寸间变化，当变换矩阵对角线上的值为0时三角形彻底消失。在[-1,0)范围内效果看上去一样但三角形翻转了，因为对角线上的值改变了顶点坐标的正负符号。

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
GLuint gWorldLocation;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.001f;

    // 缩放变换矩阵
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
    glutCreateWindow("Tutorial 08");

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

***顶点着色器shader.vs和片断着色器shader.fs的代码保持不变：***

***shader.vs***

```
#version 330

layout (location = 0) in vec3 Position;

uniform mat4 gWorld;

void main()
{
    gl_Position = gWorld * vec4(Position, 1.0);
}
```

***shader.fs***

```

#version 330

out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}

```

### ***运行效果***

位于屏幕中心的红色三角形动态从0放大到原尺寸又缩小到消失，然后翻转放大后又缩小如此循环。

和教程5一样的效果：

![这里写图片描述](http://img.blog.csdn.net/20160917213551791)
![这里写图片描述](http://img.blog.csdn.net/20160917213607041)
![这里写图片描述](http://img.blog.csdn.net/20160917213618854)
![这里写图片描述](http://img.blog.csdn.net/20160917213629760)