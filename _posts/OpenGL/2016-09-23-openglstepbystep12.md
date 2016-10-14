---
layout: post
title:  "一步步学OpenGL(12) -《透视投影》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-23
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程12***

# **透视投影**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial12/tutorial12.html](http://ogldev.atspace.co.uk/www/tutorial12/tutorial12.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***透视投影原理其他文章：*** [http://blog.csdn.net/goncely/article/details/5397729](http://blog.csdn.net/goncely/article/details/5397729)

***

### ***背景***

总算到了如何实现最优化显示3d图形的阶段了：**在保留物体深度立体感的前提下将3d世界的物体投影到2d平面上**。一个很典型的例子就是3d世界中往远方延伸的公路，2d屏幕上看上去会越来越窄最后在很远的地平线上交汇成了一个点。

我们现在要创建一种满足上面要求的一种投影变换方式，同时我们还想对其进行简化，将投影后的坐标系展示在一个-1到1的单位化的盒子空间内，使裁剪工作更加简单，这样裁剪器不需要知道屏幕的维度以及远近透视平面的位置，就可以直接进行裁剪。

对图形的透视变换需要提供四个参数：
1.屏幕宽高比：举行屏幕的宽高比例是投影的目标；
2.垂直视野：相机窗口看向3d世界的垂直方向上的角度；
3.Z轴近平面的位置：近平面用于将离相机太近的物体裁剪掉；
4.Z轴远平面的位置：远平面用于将离相机太远的物体裁剪掉；

**屏幕宽高比**是一个必要的参数，因为我们要在一个宽高相等的单位化的盒子内展示所有的坐标系，而通常屏幕的宽度是大于屏幕的高度的，所以需要在水平方向上的轴线上布置更加密集的坐标点，竖直方向上相对稀疏。这样经过变换，我们就可以在保证看到更宽阔屏幕图像的需求下，根据X轴在单位盒子空间内的比例，在X方向上添加更多的X坐标。

**垂直视野**允许我们通过调整来放大或者缩小3d世界中的视野。思考下面图示中的例子：左边的图片中相机的视角较大而在屏幕上物体看上去应该会较小；右边的视角较小而物体看上去应该会更大。注意这是由于相机的位置导致的效果，有点违背直觉，（同样的物体咋就看上去一个大一个小了咧）。左边的相机靠近投影屏幕使视角变大而右边的相机远离屏幕视角变小。然而，要记住**在程序中这个现象不会有实际的效果，因为投影的坐标系和屏幕是映射匹配的，相机的位置不会产生任何影响**。

![这里写图片描述](http://img.blog.csdn.net/20160920201019155)

我们先从投影屏幕到相机的距离这个问题开始看。投影屏幕是一个和XY平面平行的平面。很明显，整个平面不会都可见，因为它太大了。我们只能从一个和屏幕有相同的宽高比的矩形区域（投影窗）来看物体，屏幕宽高比为：
***ar = screen width / screen height***

我们简单的定义投影窗的高度为2，也就是说屏幕的宽度刚好是屏幕宽高比的两倍：2*ar。如果我们把相机放在原点，并从相机背面看这个区域我们会看到下面的坐标系：

![这里写图片描述](http://img.blog.csdn.net/20160920201059359)

所有超出这个矩形的物体都会被裁剪掉，我们已经看到在里面的坐标系会在该范围内有他的Y坐标分量，X轴分量现在相对较大但之后会提供一个固定值。
现在我们从一侧看向YZ平面：

![这里写图片描述](http://img.blog.csdn.net/20160920201138219)

通过垂直视角我们可以看到相机到投影平面的距离（用角alpha表示）：

![这里写图片描述](http://img.blog.csdn.net/20160920201200141)

下一步是计算投影坐标系中的X和Y。看下面一幅图（还是看向YZ平面）。

![这里写图片描述](http://img.blog.csdn.net/20160920201235198)
3d世界中有一个坐标为(x,y,z)的点，我们想在投影平面上找到这个点投影到平面上的坐标(xp,yp)。这个图形中X轴分量由于垂直纸面已经不再观察范围内了，所以我们以Y轴来计算，根据相似三角形我们可以得到：

![这里写图片描述](http://img.blog.csdn.net/20160920201303752)

同理，在X轴上计算有：

![这里写图片描述](http://img.blog.csdn.net/20160920201334253)

由于我们的屏幕尺寸宽为2*ar，高为2，所以3d世界的点只要X坐标在-ar到ar之间，Y坐标在-1到1之间那么就会在屏幕上有投影点。这样在Y分量上我们实际上是单位化的了，但X分量上没有单位化。我们可以让Xp除以屏幕宽高比来将其单位化，单位化后原本X坐标为+ar的就变成+1了。比如如果投影后X坐标是+0.5，屏幕宽高比是1.333（分辨率为1024x768的屏幕），单位化后的新X坐标变为0.375。总之，除以屏幕宽高比进行单位化可以起到在X轴上浓缩更多的坐标点的效果。
对于X和Y坐标我们得到下面的等式：

![这里写图片描述](http://img.blog.csdn.net/20160920201406206)

在求解最终结果之前让我们先尝试看一下这个投影变换矩阵大致应该是什么样子的。也就是说我们要使用矩阵来表示上面的等式。那么问题来了，在两个等式中，我们分别都需要让X或Y除以Z，而Z同时也是表示位置的向量的一个分量，Z的值从一个顶点到下个顶点会改变，也就是不是常量，所以不可能把它放在一个矩阵中来对所有顶点进行变换。为了好理解我们可以先看变换矩阵顶上第一行的四个分量(a,b,c,d)。我们需要找到一组值使下面的等式成立：

![这里写图片描述](http://img.blog.csdn.net/20160920201430777)

这是第一行的这组值和顶点位置向量的点积，最后要作为变换后的顶点位置向量中X分量的值。我们可以使‘b’和‘d’都为0，但是无论‘a’和‘c’怎么取值也得不到等号右边的结果。OpenGL中的解决办法是将这个变换分解成两步：先乘以一个投影变换矩阵，然后再单独除以Z分量的值。我们的应用中会提供那个投影变换矩阵，shader中要进行顶点和投影变换矩阵相乘的这个步骤，除以Z分量的单独步骤在GPU中是固定的，而且是在光栅器中进行（在顶点着色器和片段着色器之间的某个地方）。那么GPU怎么知道顶点着色器输出的哪些顶点需要除以它们的Z值呢？简单的是，这个会由内置的gl_Position变量来负责实现，不需要我们操心。现在我们要做的是找到上面只关于X和Y两个分量的投影变换矩阵。乘以这个投影变换矩阵之后GPU之后会自动帮我们进行Z值得相除变换使我们可以得到我们想要的最终结果。
但是这里还有一个复杂的点是：如果我们将顶点位置和变换矩阵相乘，然后除以Z值我们事实上会丢失Z值，因为每个顶点中Z的值都变成1了。最初的Z值是必须要保存下来的，因为之后要用来进行深度检测(depth test)。这里的技巧是将原始的Z值保存在结果向量的W分量中，然后只将XYZ除以W分量而不是Z。W保存Z的原始值用于最后的深度检测。将gl_Position除以W分量的自动步骤称为**‘透视分割’**。现在我们就可以创建一个实现上面两个等式的中间变换矩阵，以及在W分量中保存Z的值：

![这里写图片描述](http://img.blog.csdn.net/20160920201500059)

正如我之前说的，我们想实现既将Z值单位化，还要使裁剪器更容易的对图形进行深度裁剪，而不需要知道Near Z和Far Z的值。然而上面的矩阵将Z都变成0了。我们知道向量变换之后系统会自动进行透视分离，我们需要选择变换矩阵第三行的一组值，使视窗内（比如：NearZ <= Z <= FarZ）Z分量上的分离操作樱映射到[-1,1]范围内。这个映射操作由两部分组成：首先我们将 [NearZ, FarZ]这个范围缩放到任意一个宽度为2的范围内，然后进行平移使其起点从-1开始，也就是[-1,1]的范围了。对Z进行先缩放然后平移的操作可以用下面的一个通用函数表示：

![这里写图片描述](http://img.blog.csdn.net/20160920201525067)

但是之后的透视分离会将等号右侧的函数变成：

![这里写图片描述](http://img.blog.csdn.net/20160920201546606)

现在我们找出将Z的范围映射到[-1,1]范围的A和B的值。特别的我们知道当Z等于NearZ时映射结果为-1，而当Z等于FarZ时映射结果为1，所以代入后我们可以得到：

![这里写图片描述](http://img.blog.csdn.net/20160920201615278)

Now we need to select the third row of the matrix as the vector (a b c d) that will satisfy:
现在我们需要设置变换矩阵的第三行(a,b,c,d)的值满足下面的等式：

![这里写图片描述](http://img.blog.csdn.net/20160920201640897)

首先我么立即会将‘a’和‘b’的值设置为0，因为我们不想X和Y在变换过程中对Z产生影响。然后我们可以让A作为‘c’的值而B作为‘d’的值（W已知为1）。因此我们最终的变换矩阵为：

![这里写图片描述](http://img.blog.csdn.net/20160920201704716)

将顶点位置向量和投影变换矩阵相乘之后，坐标系将会变换到裁剪空间中，并且在透视分离之后坐标系会变换到NDC 空间（Normalized Device Coordinates：单位化设备坐标系）中。
这一系列教程中的整个渲染路线现在应该十分清晰了。之前在没有投影变换的情况下，我们只能从顶点着色器VS中简单地输出XYZ各分量都在[-1,1]范围内的顶点，保证他们在屏幕当中，并且通过让W的值为1我们可以防止透视分离产生的影响，然后将坐标变换到屏幕空间就结束了。当使用了投影变换矩阵之后，透视分离就成了3d投射到2d平面的一个集成的部分了。

### 源代码详解

(1)

```
void Pipeline::InitPerspectiveProj(Matrix4f& m) const>
{
    const float ar = m_persProj.Width / m_persProj.Height;
    const float zNear = m_persProj.zNear;
    const float zFar = m_persProj.zFar;
    const float zRange = zNear - zFar;
    const float tanHalfFOV = tanf(ToRadian(m_persProj.FOV / 2.0));

    m.m[0][0] = 1.0f / (tanHalfFOV * ar); 
    m.m[0][1] = 0.0f;
    m.m[0][2] = 0.0f;
    m.m[0][3] = 0.0f;

    m.m[1][0] = 0.0f;
    m.m[1][1] = 1.0f / tanHalfFOV; 
    m.m[1][2] = 0.0f; 
    m.m[1][3] = 0.0f;

    m.m[2][0] = 0.0f; 
    m.m[2][1] = 0.0f; 
    m.m[2][2] = (-zNear - zFar) / zRange; 
    m.m[2][3] = 2.0f * zFar * zNear / zRange;

    m.m[3][0] = 0.0f;
    m.m[3][1] = 0.0f; 
    m.m[3][2] = 1.0f; 
    m.m[3][3] = 0.0f;
}

```
管线类中添加一个叫做m_persProj的数据结构，用来保存透视变换的配置信息。上面的方法可以创建我们在上面演算得到的变换矩阵。

(2)`m_transformation = PersProjTrans * TranslationTrans * RotateTrans * ScaleTrans;`

我们将投影变换矩阵放在相乘式子的第一个位置来实现完整的变换。注意由于位置向量在最右边所以透视变换实际上是最后进行的，我们先缩放，然后旋转，平移，最后投影。

(3)`p.SetPerspectiveProj(30.0f, WINDOW_WIDTH, WINDOW_HEIGHT, 1.0f, 1000.0f);`

在渲染函数中我们设置投影变换的参数，可以调节这个看不同的效果。


***PS：我发现上面的有个式子原作者好像失误写错了，大家注意一下：***

![这里写图片描述](http://img.blog.csdn.net/20160923155216410)

![这里写图片描述](http://img.blog.csdn.net/20160923155234188)


### ***示例Demo***

```

#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
// 管线类
#include "ogldev_pipeline.h"
// 屏幕宽高宏定义
#define WINDOW_WIDTH 1024
#define WINDOW_HEIGHT 768

GLuint VBO;
GLuint IBO;
GLuint gWorldLocation;
// 透视变换配置参数数据结构
PersProjInfo gPersProjInfo;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.1f;

    Pipeline p;
    p.Rotate(0.0f, Scale, 0.0f);
    p.WorldPos(0.0f, 0.0f, 5.0f);
    // 设置投影变换的参数
    p.SetPerspectiveProj(gPersProjInfo);

    glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, (const GLfloat*)p.GetWPTrans());

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
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.5773f);
    Vertices[1] = Vector3f(0.0f, -1.0f, -1.15475f);
    Vertices[2] = Vector3f(1.0f, -1.0f, 0.5773f);
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
    glutInitWindowSize(WINDOW_WIDTH, WINDOW_HEIGHT);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 12");

    InitializeGlutCallbacks();

    // Must be done after glut is initialized!
    GLenum res = glewInit();
    if (res != GLEW_OK) {
      fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
      return 1;
    }
    
    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);

    CreateVertexBuffer();
    CreateIndexBuffer();

    CompileShaders();

    // 初始化透视变换配置参数
    gPersProjInfo.FOV = 30.0f;
    gPersProjInfo.Height = WINDOW_HEIGHT;
    gPersProjInfo.Width = WINDOW_WIDTH;
    gPersProjInfo.zNear = 1.0f;
    gPersProjInfo.zFar = 100.0f;
	                
    glutMainLoop();

    return 0;
}

```

***着色器代码不变：***
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

可以看出加了透视很有立体感了：
![这里写图片描述](http://img.blog.csdn.net/20160924233033481)