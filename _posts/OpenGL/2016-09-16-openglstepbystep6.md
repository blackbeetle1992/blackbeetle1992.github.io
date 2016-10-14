---
layout: post
title:  "一步步学OpenGL(6) -《平移变换》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-16
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程6：***

# **平移变换**

***原文：***[http://ogldev.atspace.co.uk/www/tutorial06/tutorial06.html](http://ogldev.atspace.co.uk/www/tutorial06/tutorial06.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

从这个教程开始我们开始研究各种各样的图形变换，图形变换就可以让一个3d物体在屏幕中变换的的时候看上去保持有深度的错觉，也就是立体的投影效果。实现立体效果的方法是使用一个经过多次相乘的变换矩阵得到的最终变换矩阵来和顶点的位置再相乘，这样得到3d物体的一个多次变换后的最终复合变换效果。后面每个教程将专门研究一种变换。

这里我们先看一下平移变换，使一个物体沿着一个任意长度任意方向的向量平移，比如说让一个三角形从左边移动到右边：

![这里写图片描述](http://img.blog.csdn.net/20160914223413884)

在之前顶点着色器知识的基础上我们可以想到实现平移的一种办法是设置一个偏移向量（这里就是- 1，1 了），并把这个便宜向量定义成**一致变量**然后传递给shader让每一个顶点按照那个偏移向量移动即可。但这样就打破通过乘以一个经过多个变换矩阵相乘得到的复合变换矩阵来进行复合变换的统一性了。另外，如果平移变换变换不是一系列复合变换的第一个的话，你要先乘以平移变换之前的几个变换的复合变换矩阵，然后按照上面的方法平移图形，然后再乘以平移变换后面的复合变换矩阵（如果不止一个平移变换那么就要继续分解更繁琐了orz...）。
一个更好的办法是找到一个表示平移变换的矩阵，并加入到复合变换中作为复合变换中的其中一种变换。可是就上面的三角形平移例子，我们能找到一个矩阵和三角形左下角的点(0，0)相乘之后得到结果(1，1)吗？事实上是不可能找到这样一个2维矩阵的（同样对于点(0，0，0)也不可能找到这样一个符合要求的3维矩阵）。统一的说，我们是想找到这样一个矩阵，对于给定的点P(x,y,z)和平移向量V(v1,v2,v3)，能够**使 M * P = P1(x+v1, y+v2, z+v3) **，简单地说就是矩阵M将P转换成了P+V。在结果向量P1中我们可以看到每个分量是P和V每个分量对应相加的和，结果向量P1的+号左侧来自P本身，对于得到P本身的向量应该这样：
I * P = P(x,y,z) 。所以我们应该从得到本身开始来调整变换矩阵得到结果矩阵右侧相加结果（...+V1, ...+V2, ...+V3）的最终变换矩阵。首先自身变换矩阵的样子如下：

![这里写图片描述](http://img.blog.csdn.net/20160914230533680)

我们想修改这个自身变换矩阵使结果变成这样子：

![这里写图片描述](http://img.blog.csdn.net/20160914230632291)

如果我们坚持用3x3矩阵好像不可能得到想要的结果，但如果改成4x4矩阵我可以这样得到想要的结果：

![这里写图片描述](http://img.blog.csdn.net/20160914230750855)

这样使用一个4维向量表示一个3维向量叫做**齐次坐标**，这在3d图形学中很常用也很有用，第四个分量称作“w”。事实上，我们之前教程中看到的内部shader符号变量**gl_Position**就是一个4维向量，第四个分量“w”在从3d到2d的投影变换中起着关键作用。通常对于表示点的矩阵会让w=1，而对于表示向量的矩阵会让w=0，因为点可以被做变换而向量不可以，你可以改变一个向量的长度和方向，但是长度和方向一样的所有向量都是相等的，不管他们的起点在哪里，所以我们可以把所有的向量起点放到原点来看。对于向量设置w=0然后乘以变换矩阵会得到和自身一样的向量。

### ***源代码详解***

(1)`struct Matrix4f {
    float m[4][4];
};`

我们在math_3d这个头文件中添加4维矩阵的定义，方便以后的矩阵变换用。

(2)`GLuint gWorldLocation;`

我们用这个句柄来获取shader中的“世界”**平移变换矩阵一致变量**的引用。用“世界”来形容的原因是我们要做的是把物体移动到我们虚拟“世界”坐标系中的某个位置。

(3)
`Matrix4f World;`
`World.m[0][0] = 1.0f; World.m[0][1] = 0.0f; World.m[0][2] = 0.0f; World.m[0][3] = sinf(Scale);`
`World.m[1][0] = 0.0f; World.m[1][1] = 1.0f; World.m[1][2] = 0.0f; World.m[1][3] = 0.0f;`
`World.m[2][0] = 0.0f; World.m[2][1] = 0.0f; World.m[2][2] = 1.0f; World.m[2][3] = 0.0f;`
`World.m[3][0] = 0.0f; World.m[3][1] = 0.0f; World.m[3][2] = 0.0f; World.m[3][3] = 1.0f;`

在渲染函数中我们按照上面介绍的来设置这个矩阵，设置v2和v3为0保持在Y和Z轴上不变化，设置v1为上个教程中sin函数得到的值，这样可以让物体在X轴上在-1到1之间来回平滑的移动。
现在我们要将这个矩阵加载到shader当中。

(4)`glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, &World.m[0][0]);`

这是另外一个使用glUniform*函数将数据加载到shader一致变量中的例子。这个函数定义的可以加载4x4矩阵数据，还可以加载像2x2,3x3,3x2,2x4,4x2,3x4和4x3这样的矩阵。
第一个参数是一致变量的位置（在shader编译后使用glGetUniformLocation()获取）；第二个参数指的是我们要更新的矩阵的个数，我们使用参数1来更新一个矩阵，但我们也可以使用这个函数在一次回调中更新多个矩阵；第三个参数通常会使新手误解，第三个参数指的是矩阵是**行优先**还是**列优先**的。行优先指的是矩阵是从顶部开始一行一行给出的，而列优先是从左边一列一列给出的。C/C++中默认是行优先的。也就是说当年你构建一个二维数组时，在内存中是一行一行存储的，顶部的行在更低的地址区。比如下面的矩阵：

`int a[2][3];`
`a[0][0] = 1;`
`a[0][1] = 2;`
`a[0][2] = 3;`
`a[1][0] = 4;`
`a[1][1] = 5;`
`a[1][2] = 6;`

这个矩阵看上去和下面一样：

1 2 3

4 5 6

在内存中的排布就像：1 2 3 4 5 6（1在低地址）。
所以我们glUniformMatrix4fv()函数的第三个参数是GL_TRUE，因为我们提供的矩阵是行优先的。我们也可以设置第三个参数为GL_FALSE，但同时我们要将我们的矩阵转置（在C/C++内存布局中会保持不变，但OpenGL中会认为前四个值是一个矩阵的列）；第四个参数是内存中矩阵的起始地址。

剩下的代码是shader脚本了：

(5)`uniform mat4 gWorld;`

这是一个4x4的矩阵的一致变量，也有mat2和mat3。

(6)`gl_Position = gWorld * vec4(Position, 1.0);`

在顶点缓冲区中三角形的顶点位置是一些3维的向量，我们讨论后这里需要一个值为1的第四个分量。有两种方式：要么直接在顶点缓冲区将顶点用4维向量表示，要么在顶点着色器中再添加第4个分量，但第一个方式没啥优势，浪费内存罢了，每个顶点要额外浪费4bytes空间。后者在顶点着色器阶段再添加第4个分量会更高效，在GLSL中使用'vec4(Position, 1.0)'来实现。我们将变换矩阵和那个向量相乘然后将结果赋给gl_Position。
总之，在每一帧中我们都产生一个变换矩阵将X轴的值在-1和1之间来回变换。着色器使用那个变换矩阵和每个顶点的位置相乘从而使图形整体左右摆动。多数情况下在顶点着色器变换后三角形的一边会超出**单位盒子**边界，然后裁剪器会把超出的一侧裁剪掉，我们只能看到在单位盒子里面的图形。

### ***示例Demo***

```

#include <stdio.h>
#include <string.h>

#include <math.h>
#include <GL/glew.h>
#include <GL/freeglut.h>

#include "ogldev_util.h"
#include "ogldev_math_3d.h" // 从这个教程开始要用到Metrix4x4矩阵的数据结构了

GLuint VBO;
// 平移变换一致变量的句柄引用
GLuint gWorldLocation;

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";


static void RenderSceneCB()
{
    glClear(GL_COLOR_BUFFER_BIT);

    static float Scale = 0.0f;

    Scale += 0.001f;

    // 4x4的平移变换矩阵
    Matrix4f World;

    World.m[0][0] = 1.0f; World.m[0][1] = 0.0f; World.m[0][2] = 0.0f; World.m[0][3] = sinf(Scale);
    World.m[1][0] = 0.0f; World.m[1][1] = 1.0f; World.m[1][2] = 0.0f; World.m[1][3] = 0.0f;
    World.m[2][0] = 0.0f; World.m[2][1] = 0.0f; World.m[2][2] = 1.0f; World.m[2][3] = 0.0f;
    World.m[3][0] = 0.0f; World.m[3][1] = 0.0f; World.m[3][2] = 0.0f; World.m[3][3] = 1.0f;

    // 将矩阵数据加载到shader中
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
    glutCreateWindow("Tutorial 06");

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

***片断着色器shader脚本代码不变：***

```

#version 330

out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}


```

***顶点着色器代码修改如下：***

```

#version 330

layout (location = 0) in vec3 Position;

// 平移变换聚矩阵一致变量
uniform mat4 gWorld;

void main()
{
	// 用平移变换矩阵乘以图形顶点位置对应的4X4矩阵相乘，完成平移变换
    gl_Position = gWorld * vec4(Position, 1.0);
}


```

### ***运行效果：***

红色三角形在X轴上左右来回移动。

![这里写图片描述](http://img.blog.csdn.net/20160917220029896)
![这里写图片描述](http://img.blog.csdn.net/20160917220042459)