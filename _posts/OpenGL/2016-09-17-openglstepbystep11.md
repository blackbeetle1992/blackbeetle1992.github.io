---
layout: post
title:  "一步步学OpenGL(11) -《复合变换》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-17
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程11***

# **复合变换**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial11/tutorial11.html](http://ogldev.atspace.co.uk/www/tutorial11/tutorial11.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

**注：**这个教程要添加更多的头文件了，基本上全了，其中用到了***AntTweakBar***这个库，安装很简单，和之前其他插件一样，去官网下载资源包，里面lib文件夹内已经有了编译好的dll和lib库，也可以运行vs项目自己重新编译。
将lib文件夹里面的dll文件复制到**Windows/System32**目录下，lib文件复制到比如我的**C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\lib**目录下，另外将include文件夹里面的AntTweakBar.h头文件复制到比如我的**C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\include**目录下就可以了。

### ***背景***

在前面的几个教程中我们建立了几种基本图形变换是我们能灵活的将物体移动到3d世界的任意地方。我们还有很多要学（相机控制和透视投影等等），但你可能也已经想到，复合的图形变换也是必的。多数情况下，你会想缩放物体来和3d世界相适应，旋转物体到合适的方向，移动物体到某个地方等等。到现在我们已经实践了每一次的单一图形变换。为了实现上面一系列的变换，我们需要对顶点位置左乘第一个变换矩阵，然后得到的结果再左乘下一个变换矩阵等等，将所有变换矩阵都左乘顶点位置之后实现多个变换。一种笨办法就是在shader中应用每一个变换矩阵实现所有的变换，但这样很低效，因为对于所有顶点这些矩阵都是一样的，只有顶点的位置发生变化，这样要不断重复对每个顶点位置进行这一系列的矩阵相乘操作。幸运的是线性代数中有一些规则可以说让我们的生活更加简易，他告我们对于给定的一组矩阵M0...Mn和一个向量V有下面的等式换算：

`Mn * Mn-1 * ... * M0 * V = (Mn* Mn-1 * ... * M0) * V`

所以如果你令：

`N = Mn * Mn-1 * ... * M0`

那么：

`Mn * Mn-1 * ... * M0 * V = (Mn * Mn-1 * ... * M0) * V = N * V`

这意味着我们可以一次性计算N，然后把它作为一致变量传给shader和每一个顶点位置相乘完成所有的变换，这个只需要GPU对每个顶点进行一次矩阵/向量相乘操作。

当计算N的时候怎样安排每个变换矩阵的先后顺序呢？你首先要记住向量最开始先是被最右边的矩阵左乘的（上面的例子就是M0）。然后向量被从右边到左边所有的变换矩阵所变换。在3d图形中你通常希望先缩放物体，然后旋转它，然后平移，之后再进行camera转换最后投影到2d屏幕。让我们先看先旋转后平移会怎样：

![这里写图片描述](http://img.blog.csdn.net/20160918100141044)

然后再看先平移后旋转会怎样：

![这里写图片描述](http://img.blog.csdn.net/20160918100235493)

可以看出，如果先平移物体的话会很难来设置物体的最终位置，因为当你将物体远离坐标原点后，再旋转物体会同时造成物体的平移效果（是绕原点旋转，而不是绕自身旋转了），这是我们希望能避免的。通过先旋转后移动可以避免这两个操作的相互依赖性，这也是尽量围绕原点对称建模的原因，那样当你缩放或者旋转物体不会产生副作用，缩放和旋转后物体依然保持和之前一样对称。

### ***源代码详解***

(1)
`#define ToRadian(x) ((x) * M_PI / 180.0f)`

`#define ToDegree(x) ((x) * 180.0f / M_PI)`

在这个教程中我们开始要使用具体的角度值了。标准C语言库中三角函数使用弧度值作为参数。上面的宏定义可以**实现角度和弧度之间的转换**。

(2)

```
inline Matrix4f operator*(const Matrix4f& Right) const
{
    Matrix4f Ret;
    for (unsigned int i = 0 ; i < 4 ; i++) {
       for (unsigned int j = 0 ; j < 4 ; j++) {
           Ret.m[i][j] = m[i][0] * Right.m[0][j] +
                         m[i][1] * Right.m[1][j] +
                         m[i][2] * Right.m[2][j] +
                         m[i][3] * Right.m[3][j];
       }
    }

    return Ret;
}
```

这里是**定义了矩阵相乘**的操作符。可以看到结果矩阵的值依次是左边矩阵每一行和右边矩阵对应每一列的点乘。这个矩阵相乘的操作对实现管线类十分重要。

(3)

```
class Pipeline
{
    public:
       Pipeline() { ... }
       void Scale(float ScaleX, float ScaleY, float ScaleZ) { ... }
       void WorldPos(float x, float y, float z) { ... }
       void Rotate(float RotateX, float RotateY, float RotateZ) { ... }
       const Matrix4f* GetTrans();
    private:
       Vector3f m_scale;
       Vector3f m_worldPos;
       Vector3f m_rotateInfo;
       Matrix4f m_transformation;
};

```

**管线类**抽象出了一个物体的所有变换的数据信息。现在有**3个私有vector成员变量分别来存储物体在世界空间中的缩放比例，位置和每个像素的旋转角度**。另外有接口函数来设置他们的值，以及可以获取表示所有变换最终的复合变换矩阵。

(4)

```
const Matrix4f* Pipeline::GetTrans()
{
    Matrix4f ScaleTrans, RotateTrans, TranslationTrans;
    InitScaleTransform(ScaleTrans);
    InitRotateTransform(RotateTrans);
    InitTranslationTransform(TranslationTrans);
    m_transformation = TranslationTrans * RotateTrans * ScaleTrans;
    return &m_transformation;
}

```

这个函数**初始化了那三个独立的变换矩阵并把它们一个一个的相乘然后返回最终的积**。注意相乘的顺序是和上面描述的一样固定死的。如果你想更灵活一点你可以设置一个定义变换顺序的位掩码。还要注意它总是将最终的复合变换矩阵作为一个陈成员变量。你可以通过检查标记来做优化，当上一次的变换函数没有任何变化时可以返回上一次存储的变换矩阵，避免重复计算。

这个函数按照之前教程的内容讲的使用私有方法来产生不同的变换。在下一个教程中这个类还将被扩展，来进行相机控制和透视变换。

(5)

```
Pipeline p;
p.Scale(sinf(Scale * 0.1f), sinf(Scale * 0.1f), sinf(Scale * 0.1f));
p.WorldPos(sinf(Scale), 0.0f, 0.0f);
p.Rotate(sinf(Scale) * 90.0f, sinf(Scale) * 90.0f, sinf(Scale) * 90.0f);
glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, (const GLfloat*)p.GetTrans());

```

上面的语句是渲染函数中做的变化。我们实例化一个pipeline管线类对象，初始化配置好之后传递给shader。通过调整参数来看最终图像的效果。

### ***示例Demo***

用到了matrix4x4头文件中的扩展工具函数；

```

#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_pipeline.h"

GLuint VBO;
GLuint IBO;
GLuint gWorldLocation;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.001f;

    // 实例化一个pipeline管线类对象，初始化配置好之后传递给shader
    Pipeline p;
    p.Scale(sinf(Scale * 0.1f), sinf(Scale * 0.1f), sinf(Scale * 0.1f));
    p.WorldPos(sinf(Scale), 0.0f, 0.0f);
    p.Rotate(sinf(Scale) * 90.0f, sinf(Scale) * 90.0f, sinf(Scale) * 90.0f);
    glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, (const GLfloat*)p.GetWorldTrans());

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);

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
    Vector3f Vertices[4];
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
    Vertices[1] = Vector3f(0.0f, -1.0f, 1.0f);
    Vertices[2] = Vector3f(1.0f, -1.0f, 0.0f);
    Vertices[3] = Vector3f(0.0f, 1.0f, 0.0f);

 	glGenBuffers(1, &VBO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
}

static void CreateIndexBuffer()
{
    unsigned int Indices[] = { 0, 3, 1,
                               1, 3, 2,
                               2, 3, 0,
                               0, 1, 2 };

    glGenBuffers(1, &IBO);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);
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
    glutCreateWindow("Tutorial 11");

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

***shader.vs和shader.fs脚本不变***

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

缩放，旋转和平移的复合变换效果，可自行设计调整，可以看到彩色四面体在空间中自由的灰来灰去打转。

![这里写图片描述](http://img.blog.csdn.net/20160924231247381)