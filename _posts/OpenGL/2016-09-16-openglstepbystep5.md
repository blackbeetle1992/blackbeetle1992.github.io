---
layout: post
title:  "一步步学OpenGL(5) -《一致变量》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-16
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程 5：***
# 一致变量
***原文：*** [http://ogldev.atspace.co.uk/www/tutorial05/tutorial05.html](http://ogldev.atspace.co.uk/www/tutorial05/tutorial05.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

在这个教程中我们将遇到一个新的shader变量：**一致变量（Uniform Variables）**。一致变量和普通属性的区别：普通变量所包含的数据是顶点具体化的，所以在每个着色器引入的时候它们将从顶点缓冲区加载一个新的值；但是一致变量的值在整个draw call中保持不变。这意味着你在draw call之前加载一直变量的值之后，你可以在每一个顶点着色器引入的时候总可以取得相同的值。一致变量主要的作用是保存像光照参数（光的位置和方向等）、变换矩阵、材质对象的handle等数据。

在这个教程中我们最后可以让图形在屏幕上产生移动，我们使用一个每一帧的值都会改变的一致变量和GLUT提供的一个闲置的回调来实现移动的效果。问题是如果不是必须的话GLUT不会重复调用我们的渲染回调。GLUT只有在像最大最小化窗口或者从另外一个窗口后面重新出现等这样的事件下才会必须重新调用我们的渲染回调。如果在应用启动后我们没有在窗口布局中做任何变化那么渲染回调只会被执行一次，这个可以通过在渲染函数中添加一个prinf打印语句可以验证，你会看到只会输出一次，但如果你最小化窗口然后最大化窗口的话会再次看到输出。
在之前的教程中只在GLUT中注册渲染回调是可以的，但是现在我们想多次重复的改变一个变量的值，需要通过再注册一个闲置的函数回调。这个闲置的函数会被GLUT在没有收到任何窗口事件的时候被调用。你可以使用一个专门的函数用于这个回调，在函数内做一些像时间更新的工作或者只是简单地注册这么一个渲染回调作为待用的闲置回调函数。在我们这个教程中我们将在这个渲染回调函数中更新变量的值。

### ***源代码详解***
(1)`glutIdleFunc(RenderSceneCB);`

这里我们将渲染回调注册为全局闲置回调。注意如果你决定使用一个专用的闲置回调那需要在它后面添加一个glutPostRedisplay()以标记重新绘制窗口，不然那个闲置回调会被不停的调用但渲染函数却不会被调用，看不到效果。glutPostRedisplay()标记当前的窗口需要被重新绘制，当下一个GLUT的主循环开始的时候这个渲染回调就会被调用了。

(2)
`gScaleLocation = glGetUniformLocation(ShaderProgram, "gScale");`

`assert(gScaleLocation != 0xFFFFFFFF);`

在链接好程序之后，我们就可以查询这个**一致变量uniform variable**的位置了。这也是另一个要将C/C++应用中的执行环境映射到shader着色器执行环境中的例子，你无法直接获取shader着色器的内容跟不能改变它的变量的值。当你编译shader对象的时候，GLSL编译器会对每一个**uniform variable一致变量**赋予一个**index索引**。在shader的内部表示中它是通过变量的索引值来获取编译器内部的变量值的，那个索引可以通过glGetUniformLocation函数设置**程序对象的handle句柄**参数和**变量名**参数来获取，如果有错误会返回-1。检查错误是非常重要的（就像上面语句通过assert检查）否则在以后的update中这个变量无法传给shader，这个函数出错的原因主要有两个：要么是你的变量名输错了，要么是被编译器优化掉了，因为如果GLSL编译器发现变量没有在shader中被实际使用就会被丢弃掉，那样调用glGetUniformLocation获取index肯定就失败了。

(3)
`static float Scale = 0.0f;`

`Scale += 0.001f;`

`glUniform1f(gScaleLocation, sinf(Scale));`

我们维护一个静态的浮点数变量，这个变量在每个渲染回调中都变化一点点（如果上面0.001这个值在你的电脑上运行时图形变化的太快或者太慢的话可以调整一下）。上面参数中加了sin函数，这样实际传给shader的值实际上是Scale变量的正弦值，实现缩放参数值在 -1.0到1.0之间平滑循环变换。注意sinf()函数的参数是弧度值而不是角度值，但这里我们不关心，我们只是需要一个正贤平滑变换。sinf()函数的结果值会通过glUniform1f函数传递给shader。
OpenGL提供了多个和glUniform1f类似的实例函数，命名形式为glUniform{1234}{if}。在这些函数中的第二个参数，你可以将浮点数（后缀是i）或者整型数（后缀是f）赋给不同维度（1D,2D,3D,4D）的vector向量中作为参数，当然也有别的版本函数采用其他的参数形式:vector向量的地址或者特殊的采用矩阵；第一个参数是我们通过glGetUniformLocation()函数获取的索引位置。


*下面我们看一下我们在VS顶点着色器脚本中作的变化（FS脚本不变）：*

(4)`uniform float gScale;`

这里在shader中定义一个一致变量。

(5)`gl_Position = vec4(gScale * Position.x, gScale * Position.y, Position.z, 1.0);`

我们将位置向量X和Y的值乘以在应用每一帧的循环中不断变化的scale值。
*你能解释三角形为什么会在每个循环的中途上下翻转吗？*
We multiply the X and Y values of the position vector with the value that is changed from the application every frame. Can you explain why the triangle is upside down half of the loop?

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
GLuint gScaleLocation; // 位置中间变量

const char* pVSFileName = "shader.vs";
const char* pFSFileName = "shader.fs";

static void RenderSceneCB()
{
	glClear(GL_COLOR_BUFFER_BIT);

	// 维护一个不断慢慢增大的静态浮点数
	static float Scale = 0.0f;
	Scale += 0.01f;
	// 将值传递给shader
	glUniform1f(gScaleLocation, sinf(Scale));

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
	// 将渲染回调注册为全局闲置回调
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
	Lengths[0] = strlen(pShaderText);
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
	
	// 查询获取一致变量的位置
	gScaleLocation = glGetUniformLocation(ShaderProgram, "gScale");
	// 检查错误
	assert(gScaleLocation != 0xFFFFFFFF);
}

int main(int argc, char** argv)
{
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);
	glutInitWindowSize(1024, 768);
	glutInitWindowPosition(100, 100);
	glutCreateWindow("Tutorial 05");

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

***相比上个教程做了修改后的shader.vs脚本代码：***

```

#version 330  //告诉编译器我们的目标GLSL编译器版本是3.3

layout (location = 0) in vec3 Position; // 绑定定点属性名和属性，方式二缓冲属性和shader属性对应映射

uniform float gScale;

void main()
{
    gl_Position = vec4(gScale * Position.x, gScale * Position.y, Position.z, 1.0); // 为glVertexAttributePointer提供返回值
}

```

***shader.fs脚本代码保持不变：***

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

### ***运行效果***

位于屏幕中心的红色三角形动态从0放大到原尺寸又缩小到消失，然后翻转放大后又缩小如此循环，也就是后面教程8中的缩放变换效果。

![这里写图片描述](http://img.blog.csdn.net/20160917213551791)
![这里写图片描述](http://img.blog.csdn.net/20160917213607041)
![这里写图片描述](http://img.blog.csdn.net/20160917213618854)
![这里写图片描述](http://img.blog.csdn.net/20160917213629760)