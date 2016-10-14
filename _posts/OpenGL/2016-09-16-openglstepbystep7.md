---
layout: post
title:  "一步步学OpenGL(7) -《旋转变换》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-16
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程7***

# **旋转变换**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial07/tutorial07.html](http://ogldev.atspace.co.uk/www/tutorial07/tutorial07.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

继上个教程的平移变换之后，这里开始学习旋转变换，也就是能够实现让一个点沿着一个坐标轴旋转一定的角度。旋转变换将总是改变位置的其中两个坐标，第三个坐标保持不变，这意味着旋转的路径会保持在其中一个平面上：XY平面（绕Z轴旋转），YZ平面（绕X轴旋转）和XZ平面（绕Y轴旋转）。也有一些复杂的旋转变换允许图形绕着任意向量旋转，但在我们这个阶段还不需要。

让我们从普遍统一的角度来定义这个问题。看下面这个图：

![这里写图片描述](http://img.blog.csdn.net/20160916220440959)

我们想从(x1,y1)沿着圆移动到(x2,y2)，换句话说就是将点(x1,y1)旋转a2角度。假设圆的半径是1，那有下面的式子：

![这里写图片描述](http://img.blog.csdn.net/20160916220913949)

我们用下面的三角函数变换公式来推导x2和y2的表达式：

![这里写图片描述](http://img.blog.csdn.net/20160916221134233)

从上面的公式我们可以得到：

![这里写图片描述](http://img.blog.csdn.net/20160916221242657)

再上面的图中我们看的是XY平面，Z轴指向纸面。X和Y放在4维矩阵里面的的话那么上面的公式可以写成下面的矩阵形式（不影响Z和W分量）：

![这里写图片描述](http://img.blog.csdn.net/20160916223241618)

如果我们想实现绕X或Y轴旋转，那么公式基本上是类似的但是变换矩阵的排布略有不同。
下面是绕Y轴旋转的旋转变换矩阵（y不变）：

![这里写图片描述](http://img.blog.csdn.net/20160916224001355)

绕X轴旋转的旋转变换矩阵（x不变）：

![这里写图片描述](http://img.blog.csdn.net/20160916224059151)

### ***源代码详解***

在上个教程的基础上这里代码的变换非常少。我们只是改变代码中变换矩阵的内容，将平移变换矩阵换成了旋转变换矩阵而已：

`World.m[0][0]=cosf(Scale);World.m[0][1]=-sinf(Scale);World.m[0][2]=0.0f; World.m[0][3]=0.0f;`

`World.m[1][0]=sinf(Scale);World.m[1][1]=cosf(Scale);World.m[1][2]=0.0f; World.m[1][3]=0.0f;`

`World.m[2][0]=0.0f;World.m[2][1]=0.0f;World.m[2][2]=1.0f;World.m[2][3]=0.0f;`

`World.m[3][0]=0.0f;World.m[3][1]=0.0f;World.m[3][2]=0.0f;World.m[3][3]=1.0f;`

As you can see we rotate around the Z axis. You can try the other rotations as well but I think that at this point without true projection from 3D to 2D the other rotations look a bit odd. We will complete them in a full transformation pipeline class in the coming tutorials.
这样会看到图形绕着Z轴旋转了。我们也可以尝试绕其他轴的旋转，但是没有从3d到2d的投影另外两种旋转看上起去会很奇怪，我们会在后面教程中在一个完整的图形变换管线类中来完善它。

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

    // 旋转变换矩阵
    Matrix4f World;

    World.m[0][0] = cosf(Scale); World.m[0][1] = -sinf(Scale); World.m[0][2] = 0.0f; World.m[0][3] = 0.0f;
    World.m[1][0] = sinf(Scale); World.m[1][1] = cosf(Scale);  World.m[1][2] = 0.0f; World.m[1][3] = 0.0f;
    World.m[2][0] = 0.0f;        World.m[2][1] = 0.0f;         World.m[2][2] = 1.0f; World.m[2][3] = 0.0f;
    World.m[3][0] = 0.0f;        World.m[3][1] = 0.0f;         World.m[3][2] = 0.0f; World.m[3][3] = 1.0f;

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
    glutCreateWindow("Tutorial 07");

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

***着色器脚本的代码都不变***

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

### ***运行效果：***
 
红色三角形绕圆心逆时针旋转。

![这里写图片描述](http://img.blog.csdn.net/20160917220953224)
![这里写图片描述](http://img.blog.csdn.net/20160917221009134)