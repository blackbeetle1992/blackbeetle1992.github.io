---
layout: post
title:  "一步步学OpenGL(13) -《相机空间》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-30
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程13***

# **相机空间**
***原文:*** [http://ogldev.atspace.co.uk/www/tutorial13/tutorial13.html](http://ogldev.atspace.co.uk/www/tutorial13/tutorial13.html)

*** CSDN完整版专栏:*** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

在之前的几个教程我们已经学习了两种变换。第一种是基本变换，用来改变物体的位置（平移变换）、方向（旋转变换）、尺寸（缩放变换），可以将物体至于3d世界任意位置；第二种变换是透视投影变换用于将3d世界的一个顶点位置投射到2d世界（比如：一个平面），一旦坐标置于2d那么就很容易将他们映射到屏幕空间坐标系了。这些坐标实际上是用来将组成物体的图元（点、线、三角形等）光栅化。但还有一个疑惑的问题是相机的位置，在之前所有的教程中我们都默认相机在3d世界的原点位置。实际应用中，我们想自由地将相机放在任何位置并将3d世界的顶点投射到相机前面的平面上，这才能反映出屏幕上相机和物体的正确关系。

在下面的图片中我们可以看到相机处于一个背向我们的位置，在它的前面有一个虚拟的平面并且一个球投射到平面上，相机和平面都有文字标注。由于相机的视野受其视角的限制，2d平面上可见的部分是一个矩形，矩形外面的都会被裁剪掉。我们的目标就是将矩形区域映射到屏幕上。

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial13/camera_space.png)

理论上，我们是可以创建一种变换，来将3d世界的物体投影到一个位于3d世界任意位置相机前面的2d平面上。但是，那种变换在数学上会比之前见到的变换复杂得多。相对来说，将相机固定在3d世界的原点然后往下看向Z轴会简单得多。比如，一个物体在(0,0,5)，相机在(0,0,1)并往下朝向Z轴（直接朝向物体）。如果我们将相机和物体同时向原点移动一个单位，那么他们的相对距离和方向（相机的方向）会保持不变，只是相机的位置位于了原点。同样，将所有的物体都按照这种方法移动，那么在保证相机仍然在原点的情况下，我们就可以用之前的知识来正确的渲染场景了。

上面的例子非常简单因为相机已经往下朝向Z轴并和坐标系保持一致。但是，如果相机朝向其他的方向的时候怎么办?看下面的图片，为了简单，这里用一个2d坐标系并且我们从上面往下看相机。

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial13/camera_axes2.png)

相机开始往下朝向Z轴，但是顺时针旋转45度之后，可以看到相机的坐标系可能和世界坐标系一致（上面的图），也可能不一样了（下面的图）。所以事实上同时会有两个坐标系。有一个定义物体位置的世界坐标系，还有一个和相机的三个轴线（相机的目标朝向，上，右）一致的相机坐标系。这两个坐标系叫做‘世界空间’和‘相机/视角空间’。绿球位于世界空间的(0,y,z)位置，而在相机空间中它位于相机坐标系的左上方的象限内（X为负，Z为正），我们需要找到球在相机空间的位置（世界坐标系转到相机坐标系）。然后我们就可以不管世界空间只需要考虑在相机空间中了。在相机空间中相机位于原点并朝向Z轴的方向。可以相对于相机定义物体的位置，并使用我们学过的方法来渲染它。
事实上，相机顺时针旋转45度等同于绿球绕相机逆时针旋转45度。物体的运动总是和相机的运动相反。所以一般情况下，我们需要添加两个新的变换矩阵并把它们加入到我们的管线类中。对于相机的旋转变换，我们需要在保持物体和相机距离不变的情况下移动物体（也就是绕相机中心在一个球面范围上移动了，相机中心位于原点），同时物体的实际移动方向和相机旋转的方向是相反的，对于相机的平移变换也一样的道理。

(1). 移动相机很简单，如果相机从原点移动到(x,y,z)，那么相应物体的变换矩阵就应该是(-x,-y,-z)。原因很简单：相机从原点按照向量(x,y,z)平移，所以然后再将相机移回原点同时又保证和物体相对位置不变的话，那么物体就要按照相反的方向向量(-x-y-z)进行平移。对应的变换矩阵如下：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial13/camera_space_translation.png)
(2).然后是旋转相机朝向世界坐标系中定义的一些物体。我们想找到旋转后顶点在相机定义的坐标系中的位置，所以实际的问题是：我们如何将坐标从一个坐标系转换到另一个坐标系？再看上图，我们可以说世界坐标系是由三个线性独立的单位向量：**(1,0,0),(0,1,0)和(0,0,1)**定义的。线性独立意味着我们找不到三个不是全为0的x,y,z值使得x*(1,0,0) + y*(0,1,0) + z*(0,0,1) = (0,0,0)，从集合的角度，就是说任意两个单位向量所在的平面和第三个向量垂直（平面XY和Z轴垂直等等）。可以很容易看出，图中的相机坐标系是由**(1,0,-1),(0,1,0),(1,0,1)**三个向量定义的。单位化这些向量之后我们得到:**(0.7071,0,-0.7071),(0,1,0)和(0.7071,0,0.7071)**。
下面的图片展示了向量的位置在两个不同的坐标系中分别是如何定义的。

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial13/camera_axes.png)

我们知道如何获取在世界坐标系中表示相机坐标轴的单位向量，并且我们知道向量在世界空间中的位置(x,y,z)。我们要找的是向量在相机坐标系中的位置(x',y',z')。现在我们利用**点积**操作的一个属性：**“标量投影”**。标量投影是任意一个向量A和一个单位向量B的点积，点积的结果就是向量A在单位向量B方向上的长度，也就是A在单位向量B上的投影了。上面的例子中，我们作向量(x,y,z)和相机X轴上的单位向量的点积得到的就是 x'，同样的可以得到y'和z'。(x',y',z')就是(x,y,z)在相机空间中的位置。
现在我们来看如何通过上面的知识来实现相机的定向。实现的方法称作‘UVN相机’，这是定向相机系统的其中一种（[UVN相机相关文章](http://blog.csdn.net/cppyin/article/details/6198669)）。这种方式通过下面的向量来定义相机：

![这里写图片描述](http://hi.csdn.net/attachment/201102/21/8458191_1298278315xeYN.png)
###### ***【图片来源于上面‘UVN相机相关文章’连接的文章。】***

1. N - 这个是相机目标朝向的方向向量，在一些3d术语中也称作‘look at’向量。这个向量和Z轴对应。
2. V - 这个方向向量指的是竖直站立时从头顶到天空的方向。如果你写一个模拟的飞机那么飞机翻转后这个向量恰好指向大地。这个向量和Y轴对应。
3. U - 这个向量指向相机的右侧，和X轴相对应。

为了将世界空间中的位置转换到相机空间中用UVN向量定义，我们需要将位置向量和UVN向量进行点积操作。用矩阵可以很好的表示：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial13/uvn.png)

在这个教程附带的示例代码中你会发现shader中的'gWorld'全局变量名字改成'gWVP'了。这个反映了很多教材中称的系列变换：WVP（World-View-Projection）标准。

### ***源代码详解***

在这个教程中我决定在设计上做略微的调整，将简单低级的矩阵操作从管线类中移除掉，并将矩阵操作迁移到Matrix4f类中。现在管线类会告诉Matrix4f类通过不同的方式去初始化自己，并联合多个矩阵来构造最终的图形变换操作。

***(pipeline.h:85)***

```
struct { 
    Vector3f Pos; 
    Vector3f Target;
    Vector3f Up;
} m_camera;
```
管线类中添加了一些新的成员变量用来存储相机参数。注意指向相机右侧的坐标轴消失了（‘U’轴），因为这个可以在运行的时候使用另外两个方向向量V和N**差积**运算得到。另外这里还有一个‘SetCamera’函数来传递这些值。

***(math3d.h:21)***

```
Vector3f Vector3f::Cross(const Vector3f& v) const 
{
    const float _x = y * v.z - z * v.y;
    const float _y = z * v.x - x * v.z;
    const float _z = x * v.y - y * v.x;

    return Vector3f(_x, _y, _z);
}
```
Vector3f类添加一个新的方法用来计算两个Vector3f的差积。两个向量的差积计算可以得到一个和这两个向量所在平面垂直的向量。这里可以敏感的想到到向量是只有方向和长度但是没有固定位置的性质的。所有方向和长度相等的向量都被看做同一个向量，不管他们的起点在哪儿。因此，我们最好将两个向量的起点都定义在原点，这样就可以创建一个有一个顶点在原点，另外两个顶点位于这两个向量另一端的三角形。三角形定义了一个平面，两个向量的差积是一个垂直于这个平面的向量。可以在维基百科详细了解一下差积。

***(math3d.h:30)***

```
Vector3f& Vector3f::Normalize()
{
    const float Length = sqrtf(x * x + y * y + z * z);

    x /= Length;
    y /= Length;
    z /= Length;

    return *this;
}
```

为了创建一个UVN矩阵我们需要将向量转换成单位长度。这个操作称为‘向量单位化’，通过让每个向量除以它们各自的向量长度完成。数学中这个有详细的介绍。

***(math3d.cpp:84)***

```
void Matrix4f::InitCameraTransform(const Vector3f& Target, const Vector3f& Up)
{
    Vector3f N = Target;
    N.Normalize();
    Vector3f U = Up;
    U.Normalize();
    U = U.Cross(Target);
    Vector3f V = N.Cross(U);

    m[0][0] = U.x; m[0][1] = U.y; m[0][2] = U.z; m[0][3] = 0.0f;
    m[1][0] = V.x; m[1][1] = V.y; m[1][2] = V.z; m[1][3] = 0.0f;
    m[2][0] = N.x; m[2][1] = N.y; m[2][2] = N.z; m[2][3] = 0.0f;
    m[3][0] = 0.0f; m[3][1] = 0.0f; m[3][2] = 0.0f; m[3][3] = 1.0f;
}
```
这个函数创建了一个之后供管线类使用的相机转换矩阵。UVN向量被计算出来并且按行设置到了转换矩阵中，按行横向放置是考虑到顶点坐标之后会在右侧作为一个列向量和这个变换矩阵相乘，这样就实现了UVN和位置向量的点积操作。这个会得到三个标量投影长度值作为屏幕空间中位置的XYZ值。
这个函数只得到target(N)和up(V)向量，right(U)向量是这两个向量的差积。注意我们不能确保回调会传送单位长度的向量所以我们自己对其进行单位化。计算得到U向量之后，我们重新计算target和right向量的差积再次得到up向量。原因在之后我们开始移动相机的时候就知道了。只更新target向量而保持up向量不变会更简单，但是这意味着要保证target向量和up向量之间的夹角不可以为90度，否则会使这个坐标系失效（万向锁）。上面通过重新计算点积重新得到up向量我们可以得到一个坐标轴两两垂直的坐标系。

***(pipeline.cpp:22)***

```
const Matrix4f* Pipeline::GetTrans()
{
    Matrix4f ScaleTrans, RotateTrans, TranslationTrans, CameraTranslationTrans, CameraRotateTrans, PersProjTrans;

    ScaleTrans.InitScaleTransform(m_scale.x, m_scale.y, m_scale.z);
    RotateTrans.InitRotateTransform(m_rotateInfo.x, m_rotateInfo.y, m_rotateInfo.z);
    TranslationTrans.InitTranslationTransform(m_worldPos.x, m_worldPos.y, m_worldPos.z);
    CameraTranslationTrans.InitTranslationTransform(-m_camera.Pos.x, -m_camera.Pos.y, -m_camera.Pos.z);
    CameraRotateTrans.InitCameraTransform(m_camera.Target, m_camera.Up);
    PersProjTrans.InitPersProjTransform(m_persProj.FOV, m_persProj.Width, m_persProj.Height, m_persProj.zNear, m_persProj.zFar);

    m_transformation = PersProjTrans * CameraRotateTrans * CameraTranslationTrans * TranslationTrans * RotateTrans * ScaleTrans;
    return &m_transformation;
}
```

我们更新这个函数来创建一个完成物体所有变换的矩阵。这已经变得比较复杂了，因为需要为相机额外提供两个新的变换矩阵。完成世界坐标系的基本变换（平移，缩放，旋转）之后，我们通过移动相机回原点来进行相机变换，这个通过一个相对于相机位置为负的向量来变换完成（就是和相机本来的变换向量相反的一个向量）。所以如果相机想移动到位置(1,2,3),那么我们实际需要将物体按照向量(-1,-2,-3)移动，从而保证相机仍然可以在原点。然后，我们可以根据相机的target向量和up向量计算得到相机的旋转变换矩阵（相机不存在缩放变换这一说）。这样我们就完成相机的变换部分了，最后可以投影坐标系了。

***(tutorial13.cpp:76)***

```
Vector3f CameraPos(1.0f, 1.0f, -3.0f);
Vector3f CameraTarget(0.45f, 0.0f, 1.0f);
Vector3f CameraUp(0.0f, 1.0f, 0.0f);
p.SetCamera(CameraPos, CameraTarget, CameraUp);
```
我们在主渲染循环中使用新的容器。为了实现自由布置相机效果，我们从原点沿着Z轴负向回退移动，然后右移，再竖直上移。相机朝向Z轴的正向，并且在原点的偏右侧。up向量即Y轴正向的向量。我们把这些设置到管线对象中，其余的由管线类完成。

### ***示例Demo***

```
#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_pipeline.h"

#define WINDOW_WIDTH 1024
#define WINDOW_HEIGHT 768

GLuint VBO;
GLuint IBO;
// 更换变量名
GLuint gWVPLocation;

// 相机对象
Camera* pGameCamera = NULL;
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
    p.WorldPos(0.0f, 0.0f, 3.0f);
    // 相机变换
    Vector3f CameraPos(0.0f, 0.0f, -3.0f);
    Vector3f CameraTarget(0.0f, 0.0f, 2.0f);
    Vector3f CameraUp(0.0f, 1.0f, 0.0f);
    p.SetCamera(*pGameCamera);

    p.SetPerspectiveProj(gPersProjInfo);

    // 传递最终位置
    glUniformMatrix4fv(gWVPLocation, 1, GL_TRUE, (const GLfloat*)p.GetWVPTrans());

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

    gWVPLocation = glGetUniformLocation(ShaderProgram, "gWVP");
    assert(gWVPLocation != 0xFFFFFFFF);
}

int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGBA);
    glutInitWindowSize(WINDOW_WIDTH, WINDOW_HEIGHT);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("Tutorial 13");

    InitializeGlutCallbacks();

    pGameCamera = new Camera(WINDOW_WIDTH, WINDOW_HEIGHT);

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

    gPersProjInfo.FOV = 60.0f;
    gPersProjInfo.Height = WINDOW_HEIGHT;
    gPersProjInfo.Width = WINDOW_WIDTH;
    gPersProjInfo.zNear = 1.0f;
    gPersProjInfo.zFar = 100.0f;
	                
    glutMainLoop();

    return 0;
}
```

***片段着色器不变，顶点着色器更换变量名***

***shader.vs***

```
#version 330

layout (location = 0) in vec3 Position;

// WVP标准
uniform mat4 gWVP;

out vec4 Color;

void main()
{
    gl_Position = gWVP * vec4(Position, 1.0);
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