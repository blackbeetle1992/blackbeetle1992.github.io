---
layout: post
title:  "一步步学OpenGL(14) -《相机控制1-键盘事件》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-30
categories: [OpenGL]
tags: [opengl]
icon: 

---


### ***教程14***

# **相机控制1（键盘事件）**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial14/tutorial14.html](http://ogldev.atspace.co.uk/www/tutorial14/tutorial14.html)

*** CSDN完整版专栏:*** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

在之前的教程中我们学习了如何将相机至于3d世界的任意一个位置，下一步就要实现让用户来控制它。移动应该是不受限制的：用户可以在任何方向上移动。相机的控制通过两种输入设备来实现：使用键盘控制位置的移动，使用鼠标来改变目标视角，这个和第一人称射击角色类似。这篇教程介绍键盘的控制，鼠标的控制放在下一个教程中。

我们要实现传统的上下左右四键控制。注意我们相机的变换是通过当前位置position、target向量(前方视角)和上方头顶up向量定义的，当我们使用键盘控制移动的时候我们只是改变我们的位置，我们不能倾斜相机不能将相机的视角移动到目标物体方向（不会改变target向量和up向量）。
为了使用键盘控制我们要用到另外一个GLUT接口函数：glutSpecialFunc()。这个函数注册了一个事件回调，当“专用功能按键”按下时触发。专用按键主要包括：功能键、方向键，以及PAGE-UP/PAGE-DOWN/HOME/END/INSERT这些按键。如果想获取常规按键（字母键和数字键）的触发事件，需要使用这个接口函数：glutKeyboardFunc()。

### ***源代码详解***

相机的所有功能都封装在相机类中。相机类中存储着相机所有的属性并且会根据它收到的运动事件改变属性的的值，这些属性可以被创建变换矩阵的管线类获取。

***(Camera.h)***

```
class Camera
{
public:
    Camera();
    Camera(const Vector3f& Pos, const Vector3f& Target, const Vector3f& Up);
    bool OnKeyboard(int Key);
    const Vector3f& GetPos() const
    const Vector3f& GetTarget() const
    const Vector3f& GetUp() const

private:
    Vector3f m_pos;
    Vector3f m_target;
    Vector3f m_up;
};
```
这个是相机类的定义，存储了三个定义相机的属性：位置(position),target向量和up向量。定义了两个构造函数，默认的构造函数将相机至于原点，朝向Z轴正向，up向量指向天空（0，1，0），另一个构造函数同时使用参数初始化三个成员属性。OnKeyboard()函数用于实现相机类的键盘事件，并返回一个布尔值来表示事件是否被类接收成功，如果收到的键值是和我们定义的相关的（方向键其中之一）就返回真值，否则返回false。这样你就可以构建一系列的客户端来接收键盘事件，并在当有一个客户端首先成功接收到某个相关的事件之后终止事件监测。

***(Camera.cpp:42)***

```
bool Camera::OnKeyboard(int Key)
{
    bool Ret = false;

    switch (Key) {

    case GLUT_KEY_UP:
    {
        m_pos += (m_target * StepSize);
        Ret = true;
    }
    break;

    case GLUT_KEY_DOWN:
    {
        m_pos -= (m_target * StepSize);
        Ret = true;
    }
    break;

    case GLUT_KEY_LEFT:
    {
        Vector3f Left = m_target.Cross(m_up);
        Left.Normalize();
        Left *= StepSize;
        m_pos += Left;
        Ret = true;
    }
    break;

    case GLUT_KEY_RIGHT:
    {
        Vector3f Right = m_up.Cross(m_target);
        Right.Normalize();
        Right *= StepSize;
        m_pos += Right;
        Ret = true;
    }
    break;
    }

    return Ret;
}
```

这个函数实现根据键盘按键事件移动相机。GLUT根据方向键定义了几个宏变量，这样switch语句就可以根据这几个宏变量判断是哪个按键了，这几个宏变量其实是简单的int整型变量，不是枚举。

前后移动是最简单的，因为这俩种移动方向和tartget向量在一条线上，我们只需要从起始位置加上或者减去一定数量的tartget向量即可实现前后移动，target向量本身不会变化。注意，在加减之前我们是使用一个常量‘步长’值和target向量相乘的，不管哪个方向的移动都会乘上这个步长，其实就是改变移动速度，这个步长常量之后我们会放到类里也作为一个变量。为了保证步长稳定不变，我们确保让它每次都和单位向量相乘（比如之前我们都确保target向量和up向量是单位长度的）。

左右两边的移动就稍微复杂一些了，左右的移动需要一个和tartget向量与up向量所在平面垂直的一个移动向量，这个平面将3d空间分成两部分，每个部分都有一个向量和平面垂直并且方向相反（可以叫他们left向量和right向量）。这两个向量可以用tartget向量和up向量作差积求得，两种组合：target * up 和 up * tartget（差积不满足交换律，改变左右顺序结果不是同一个向量）。得到left或者right向量之后对其进行单位化，然后乘以步长，最后加到position位置向量上实现左右方向上的移动，同时target和up向量都没有变化。

注意这个函数里面的操作用到了向量新的操作符比如‘+=’和‘-=’，这些都定义在Vector3f类中了。

***(tutorial14.cpp:73)***

```
static void SpecialKeyboardCB(int Key, int x, int y)
{
    GameCamera.OnKeyboard(Key);
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
    glutIdleFunc(RenderSceneCB);
    glutSpecialFunc(SpecialKeyboardCB);
}
```
这里我们新的事件回调来处理专用键盘事件。这个回调在按键按下时接收键盘键值或者鼠标的位置。我们这里忽略鼠标的具体位置信息，只把点击事件传给一个定义为全局的相机类的实例对象。

***(tutorial14.cpp:55)***

`p.SetCamera(GameCamera.GetPos(), GameCamera.GetTarget(), GameCamera.GetUp());`

之前我们使用固定写好的向量在管线类中初始化相机的参数，现在把那些固定的向量丢弃，相机的那几个向量属性值都直接从相机类中获取。

### ***示例Demo***

```
#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_glut_backend.h"
#include "ogldev_pipeline.h"
#include "ogldev_camera.h"

#define WINDOW_WIDTH 1024
#define WINDOW_HEIGHT 768

GLuint VBO;
GLuint IBO;
GLuint gWVPLocation;

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
    p.SetCamera(*pGameCamera);
    p.SetPerspectiveProj(gPersProjInfo);

    glUniformMatrix4fv(gWVPLocation, 1, GL_TRUE, (const GLfloat*)p.GetWVPTrans());

    glEnableVertexAttribArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);

    glDrawElements(GL_TRIANGLES, 12, GL_UNSIGNED_INT, 0);

    glDisableVertexAttribArray(0);

    glutSwapBuffers();
}

// 传递键盘事件
static void SpecialKeyboardCB(int Key, int x, int y)
{
    OGLDEV_KEY OgldevKey = GLUTKeyToOGLDEVKey(Key);
    pGameCamera->OnKeyboard(OgldevKey);
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
    glutIdleFunc(RenderSceneCB);
    // 注册键盘事件
    glutSpecialFunc(SpecialKeyboardCB);
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
    glutCreateWindow("Tutorial 14");

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

***着色器shader.vs和shader.fs不变。***








