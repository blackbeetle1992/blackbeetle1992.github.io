---
layout: post
title:  "一步步学OpenGL(3) -《第一个三角形》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-09-14
categories: [OpenGL]
tags: [opengl]
icon: 

---

# **第一个三角形**
***原文：*** [http://ogldev.atspace.co.uk/www/tutorial03/tutorial03.html](http://ogldev.atspace.co.uk/www/tutorial03/tutorial03.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

这篇教程非常简短，我们只是扩展前一个教程来渲染一个三角形。
这篇教程中我们依然使用那个单位化的盒子模型。可见的点必须在这个盒子内，这样他们将可以通过视窗的变换映射到窗口中可见的坐标上。当俯视Z坐标轴的负方向时这个单位化盒子看上去如下图：

![单位盒子](http://img.blog.csdn.net/20160910131812271)

点(-1.0, -1.0)映射到盒子的左下角，（-1.0，1.0）映射到左上角等等。如果将三角形的顶点往盒子外扩展移到盒子外，这个三角形将会被裁剪，只能看到三角形的一部分。

### ***源代码详解：***

(1)
`Vector3f Vertices[3];`

`Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);`

`Vertices[1] = Vector3f(1.0f, -1.0f, 0.0f);`

`Vertices[2] = Vector3f(0.0f, 1.0f, 0.0f);`

这我们扩展上个教程中的顶点数组使其包含三个顶点；

(2)`glDrawArrays(GL_TRIANGLES, 0, 3);`

Two changes were made to the drawing function: we draw triangles instead of points and we draw 3 vertices instead of 1.
在绘制函数中有两个变化：画三角形而不是点，画三个顶点而不是一个。

### 示例Demo

```

#include <stdio.h>
#include <GL/glew.h>        // GLEW扩展库
#include <GLUT/freeglut.h>  // freeGLUT图形库
#include "ogldev_math_3d.h" // 用于OpenGL的3d数学库

GLuint VBO;

/**
 * 渲染回调函数
 */
static void RenderScenceCB(){
    // 清空颜色缓存
    glClear(GL_COLOR_BUFFER_BIT);
    
    // 开启顶点属性
    glEnableVertexAttribArray(0);
    // 绑定GL_ARRAY_BUFFER缓冲器
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 告诉管线怎样解析bufer中的数据
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
    
    // 开始绘制几何图形(绘制一个三角形，三个顶点)
    glDrawArrays(GL_TRIANGLES, 0, 3);
    
    //  禁用顶点数据
    glDisableVertexAttribArray(0);
    
    // 交换前后缓存
    glutSwapBuffers();
    
    glFlush();
}

/**
 * 创建顶点缓冲器
 */
static void CreateVertexBuffer()
{
    // 创建含有3个顶点的顶点数组
    Vector3f Vertices[3];
    // 三角形的三个顶点位置
    Vertices[0] = Vector3f(-1.0f, -1.0f, 0.0f);
    Vertices[1] = Vector3f(1.0f, -1.0f, 0.0f);
    Vertices[2] = Vector3f(0.0f, 1.0f, 0.0f);
    
    // 创建缓冲器
    glGenBuffers(1, &VBO);
    // 绑定GL_ARRAY_BUFFER缓冲器
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 绑定顶点数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
}

/**
 * 主函数
 */
int main(int argc, char ** argv) {
    
    // 初始化GLUT
    glutInit(&argc, argv);
    
    // 显示模式：双缓冲、RGBA
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);
    
    // 窗口设置
    glutInitWindowSize(480, 320);      // 窗口尺寸
    glutInitWindowPosition(100, 100);  // 窗口位置
    glutCreateWindow("Tutorial 03");   // 窗口标题
    
    // 开始渲染
    glutDisplayFunc(RenderScenceCB);
    
    // 缓存清空后的颜色值
    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
    
    // 检查GLEW是否就绪，必须要在GLUT初始化之后！
    GLenum res = glewInit();
    if (res != GLEW_OK) {
        fprintf(stderr, "Error: '%s'\n", glewGetErrorString(res));
        return 1;
    }
    
    // 创建顶点缓冲器
    CreateVertexBuffer();
    
    // 通知开始GLUT的内部循环
    glutMainLoop();
  
    return 0;
}

```
### **运行效果**

![这里写图片描述](http://img.blog.csdn.net/20160913131957744)