---
layout: post
title:  "一步步学OpenGL(15)-《相机控制2-鼠标事件》"
desc: "OpenGL Step by Step系列教程中文翻译版本"
keywords: "opengl,ogldev,jiangxh"
date: 2016-10-09
categories: [OpenGL]
tags: [opengl]
icon: 

---

### ***教程15***

# **相机控制2（鼠标事件）**

***原文：*** [http://ogldev.atspace.co.uk/www/tutorial15/tutorial15.html](http://ogldev.atspace.co.uk/www/tutorial15/tutorial15.html)

*** CSDN完整版专栏： *** [http://blog.csdn.net/column/details/13062.html](http://blog.csdn.net/column/details/13062.html)

***

### ***背景***

在这个教程我们将实现鼠标控制相机的方向，从而完成所有有关相机的部分。对于相机的设计有很多不同程度的自由度设置，我们要完成的是能够实现第一人称视角游戏（设计游戏等等），也就是说要能够使相机围绕Y轴正向360度旋转，对应的效果就是角色能够左右转动身体一整圈。另外，还要能够让相机镜头上下倾斜看到上下更广阔的视角。但是在相机完成一圈的自身旋转之前我们不能让相机抬头，也不能在相机以某个角度转头时倾斜它，这些自由度的限制现实应用中主要在飞机飞行模拟中，这个不属于教程的范围了。但在后面的教程中我们要大体实现能够自由方便的在3d世界中畅游探索。

下图中的这个二战中的防空枪结构就形象展示了我们要创建的相机移动旋转的自由度。

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/aa_gun.jpg)

这种枪有两个控制轴线：它可以围绕向量(0,1,0)360度旋转，这个旋转的角度叫做‘水平倾角’，这个向量叫做‘垂直轴线’。它也可以绕一个和地面平行的向量上下倾斜，但这个倾斜有一定程度的范围限制，不可能360度倾斜。这个倾斜角叫做‘垂直倾角’，水平向量叫做‘水平轴线’。注意在整个旋转过程中垂直轴线是固定不变的（不会随着枪上下摆动而变化），而水平轴线会随着枪左右摆动而同步旋转保持和枪的target朝向垂直。理解这个对正确的理解其中的数学模型很关键。我们目的是要实现通过鼠标左右滑动改变水平倾角枪使枪左右旋转，鼠标上下滑动改变垂直倾角使枪抬头低头。有了这两个倾角，我们想以此计算出旋转后最终的target向量和up向量。

根据水平倾角旋转target向量很简单直接，使用基本的三角几何我们可以知道，其target向量的z分量为水平倾角的sin值，而x分量为水平倾角的cos值（这里只考虑水平旋转，从枪的头顶往下看，所以视y分量为0）。这个可以再看一下教程7中旋转变换的图示，同样的原理。

沿着垂直倾角来旋转target向量就更加复杂了，因为水平轴线会随着相机的左右摆动而同步摆动。水平轴线所在的向量可以根据水平旋转之后的垂直轴线和target向量的差积运算得到，可是上下摆动任意角度可能会出现和预想的结果不一样的情况，这种想象叫做“万向锁”现象。幸好这个问题可以使用“四元数”来进行解决。四元数是一个爱尔兰的数学家Willilam Rowan Hamilton先生在1843年发现的，四元数是基于复数系统建立的。四元数Q定义如下：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/quaternion.png)

式子中i,j,k都是复数并且满下面的式子：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/quaternion1.png)

实践中，我们定义一个四元数为一个4维向量(x,y,z,w)。与Q的共轭量定义为：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/conjugate.png)

单位化一个四元数和单位化一个向量一样。这里我将使用一个四元数来描述如何绕任意向量旋转一个向量的过程，下面的过程步骤的详细数学证明可以在网上查到。

计算表示一个向量旋转a角度的‘W’值的通用方法如下：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/rotation.png)

其中Q是旋转四元数，定义如下：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/rotationq.png)

计算出‘W’以后旋转向量就是(W.x,W.y,W.z)了。需要注意的一点是我们要让Q乘以V，这是个四元数和向量的乘法，得到的是一个四元数。然后让得到的四元数乘以右边的四元数，也就是Q*V的结果再乘以Q的轭。上面这两种类型的相乘运算是不一样的，其中各自的乘法实现已经在math_3d.cpp中定义了。

在用户移动鼠标时，我们需要不断的更新水平倾角和垂直倾角，并决定如何来初始化它们。合理的选择是根据camera构造函数提供的target向量来初始化倾角。首先我们从水平倾角开始，如下图从上朝下看向XZ平面：

![这里写图片描述](http://ogldev.atspace.co.uk/www/tutorial15/h_angle.png)

target向量是(x,z),我们想要找出那个用alpha表示的水平倾角（Y分量只和垂直倾角有关这里不关心）。由于这个圆的半径是单位1所以很容易可以得到z的值就是alpha角的sin值，所以计算z的反正弦asin值即可得到alpha角。这样就ok了么？不是的，由于正弦函数值z的范围为[-1,1],反正弦asin得到的角度范围为-90度到+90度，但是水平倾角的范围是360度。另外，我们的四元数是控制顺时针旋转的。就是说当我们使用四元数旋转90度之后我们停留在Z轴的-1处，和真正的90度的正弦值1相反。个人认为，纠正这个的最简单的方法是始终计算Z的正数值的反正弦asin值，同时结合当前是处于哪个四分之一圆区域内来确定向量的具体位置。例如：当我们的target向量是(0,1)的时候，我们计算1的反正弦值为90度，用它减去360度结果为270度，0到1的反正弦范围为0到90度，结合具体处于哪个四分之一圆我么可以得到最终的水平倾角。

计算垂直倾角就比较容易了。我们限制垂直旋转的角度范围为-90（等同于用四元数旋转270度，target垂直朝上）度到90度（target垂直朝下）。因此我们只需要target向量中Y分量的负aisn值即可。当Y等于1（垂直朝上）时asine值为90，我们只需要取反符号即可。当Y等于-1（垂直朝下）是asine值为-90取反符号后得到90.如果还有疑惑的话可以再看一下示意图，然后用Y替换掉Z以及用Z替换掉X。

### ***源代码详解***

***(camera.cpp:38)***

```
Camera::Camera(int WindowWidth, int WindowHeight, const Vector3f& Pos, const Vector3f& Target, const Vector3f& Up)
{
    m_windowWidth = WindowWidth;
    m_windowHeight = WindowHeight;
    m_pos = Pos;

    m_target = Target;
    m_target.Normalize();

    m_up = Up;
    m_up.Normalize();

    Init();
}
```
camera的构造函数现在获得了窗口的尺寸信息，这个需要用来将鼠标移动到屏幕正中央。另外注意，初始化函数Init()的调用可以设置camera的内部属性。

***(camera.cpp:54)***
```
void Camera::Init()
{
    Vector3f HTarget(m_target.x, 0.0, m_target.z);
    HTarget.Normalize();

    if (HTarget.z >= 0.0f)
    {
        if (HTarget.x >= 0.0f)
        {
            m_AngleH = 360.0f - ToDegree(asin(HTarget.z));
        }
        else
        {
            m_AngleH = 180.0f + ToDegree(asin(HTarget.z));
        }
    }
    else
    {
        if (HTarget.x >= 0.0f)
        {
            m_AngleH = ToDegree(asin(-HTarget.z));
        }
        else
        {
            m_AngleH = 90.0f + ToDegree(asin(-HTarget.z));
        }
    }

    m_AngleV = -ToDegree(asin(m_target.y));

    m_OnUpperEdge = false;
    m_OnLowerEdge = false;
    m_OnLeftEdge = false;
    m_OnRightEdge = false;
    m_mousePos.x = m_windowWidth / 2;
    m_mousePos.y = m_windowHeight / 2;

    glutWarpPointer(m_mousePos.x, m_mousePos.y);
}
```

在Init()初始化函数中我们开始先计算水平倾角。我们创建了一个叫做HTarget(水平target)的新target向量，这个target向量是原target向量在XZ平面上的投影。我们将它单位化（在XZ平面上使用单位向量）。然后我们检查这个target向量在哪个四分之一圆区域内，并根据Z分量的正值计算最终的倾斜角度。之后再计算垂直倾角就容易多了。camera还有四个标志变量用来标记鼠标是否指向了屏幕的其中一个边缘，当指向边缘后我们会在相应的方向上进行自动的转向旋转，这样就可以实现360轻松旋转了。开始的时候由于鼠标在屏幕中央，我们将四个标志变量初始化设置为FALSE。下面两行代码用于计算窗口中屏幕中心的位置。后面那个新的函数glutWarpPointer（）实际就是移动鼠标到屏幕中央的了。

***(camera.cpp:140)***

```
void Camera::OnMouse(int x, int y)
{
    const int DeltaX = x - m_mousePos.x;
    const int DeltaY = y - m_mousePos.y;

    m_mousePos.x = x;
    m_mousePos.y = y;

    m_AngleH += (float)DeltaX / 20.0f;
    m_AngleV += (float)DeltaY / 20.0f;

    if (DeltaX == 0) {
        if (x <= MARGIN) {
            m_OnLeftEdge = true;
        }
        else if (x >= (m_windowWidth - MARGIN)) {
            m_OnRightEdge = true;
        }
    }
    else {
        m_OnLeftEdge = false;
        m_OnRightEdge = false;
    }

    if (DeltaY == 0) {
        if (y <= MARGIN) {
            m_OnUpperEdge = true;
        }
        else if (y >= (m_windowHeight - MARGIN)) {
            m_OnLowerEdge = true;
        }
    }
    else {
        m_OnUpperEdge = false;
        m_OnLowerEdge = false;
    }

    Update();
}
```
这个函数用于通知camera鼠标的移动事件。参数是鼠标在屏幕中的新的位置坐标。我先计算从之前的点到当前点在X和Y轴上的变化。然后将鼠标的位置设置为当前的点坐标作为下次调用的上个点坐标。按比例缩小后更新改变当前水平方向和竖直方向上的倾角。这里使用了一个效果比较好的缩放比例值20.0，但是在不同的电脑可能要调整不同的值是旋转的速度看上去合适（改变鼠标灵敏度相当于）。之后的教程我们会将帧率这个参数加入计算来更好的改善这个旋转灵敏度。

之后的设置就是根据鼠标的位置来更新那四个'm_On*Edge'标志变量了。默认当鼠标离屏幕边缘在10像素以内时就出发鼠标到达屏幕边缘事件。最后调用Update()函数来根据新的水平倾角和竖直倾角重新计算targe向量和up向量。

***(camera.cpp:183)***

```
void Camera::OnRender()
{
    bool ShouldUpdate = false;

    if (m_OnLeftEdge) {
        m_AngleH -= 0.1f;
        ShouldUpdate = true;
    }
    else if (m_OnRightEdge) {
        m_AngleH += 0.1f;
        ShouldUpdate = true;
    }

    if (m_OnUpperEdge) {
        if (m_AngleV > -90.0f) {
            m_AngleV -= 0.1f;
            ShouldUpdate = true;
        }
    }
    else if (m_OnLowerEdge) {
        if (m_AngleV < 90.0f) {
            m_AngleV += 0.1f;
            ShouldUpdate = true;
        }
    }

    if (ShouldUpdate) {
        Update();
    }
}
```
这个函数是在主渲染循环回调中调用的。当鼠标处于屏幕边缘且不移动时执行这个函数，这种情况下鼠标在边缘不移动，但我们希望相机在该方向上继续移动直到鼠标离开屏幕边缘。我们检测四个标志量是否有为TRUE的并更新相对应的camera角度。角度发生变换后就调用Update()函数更新target向量和up向量。当鼠标离开屏幕边缘后我们通过鼠标事件句柄可以检测到然后可以将对应的标志变量设置为FALSE。要注意的是竖直倾角是限制在-90到90度之间的，防止抬头低头出现旋转一圈的现象。

***(camera.cpp:214)***

```
void Camera::Update()
{
    const Vector3f Vaxis(0.0f, 1.0f, 0.0f);

    // Rotate the view vector by the horizontal angle around the vertical axis
    Vector3f View(1.0f, 0.0f, 0.0f);
    View.Rotate(m_AngleH, Vaxis);
    View.Normalize();

    // Rotate the view vector by the vertical angle around the horizontal axis
    Vector3f Haxis = Vaxis.Cross(View);
    Haxis.Normalize();
    View.Rotate(m_AngleV, Haxis);
    View.Normalize();

    m_target = View;
    m_target.Normalize();

    m_up = m_target.Cross(Haxis);
    m_up.Normalize();
}
```
这个函数会根据水平倾角和竖直倾角来跟新target向量和up向量。开始我们重置窗口视角让target向量和地面平行（竖直倾角为0）并且直接看向右方（按照图中水平倾角也为0）。我们设置竖直轴线指向正上方然后按照水平倾角绕其旋转窗口向量。结果是一个偏于指向target方向的向量在高度上可能没有变化（在XZ平面上），将这个向量和和垂直轴线向量做差积可以得到XZ平面上另一个向量，这个向量和做差积的两个向量所在的明面垂直。这个新向量就是我们的新水平轴线了，然后就可以将窗口向量围绕水平轴线上下旋转竖直倾角，然后可以得到最终的target向量并且可以将它设置到相应的成员属性中。现在我们必须修复up向量。比如如果相机朝上，up向量必须要向后倾斜（他要和target向量保持90度）。这个和你抬头看天空时你的头后部要倾斜类似。新的up向量可以通过将最终的target向量和水平轴线做差积计算得到。如果竖直倾角仍然是0度，那么target向量将还在XZ平面上，up向量依然是(0,1,0)。如果target向量朝上或朝下，那么同时up向量将分别朝前或朝后。

***(tutorial15.cpp:209)***
`glutGameModeString("1920x1200@32");`
`glutEnterGameMode();`

在所谓的高性能‘游戏模式’下，这些GLUT函数允许我们进行全屏显示。它使camera的360度旋转更加简单，因为你要做的只是拖动鼠标到屏幕的一个边缘（屏幕和游戏窗口重合了）。要注意通过游戏模式字符串配置的分辨率和每英寸的比特数，32bits每像素就提供了渲染的最大数量的颜色了。

***(tutorial15.cpp:214)***
`pGameCamera = new Camera(WINDOW_WIDTH, WINDOW_HEIGHT);`

在这里camera被动态放置，因为它同时作为一个GLUT回调（glutWarpPointer）。这个回调在GLUT还没有初始化时会失败。

***(tutorial15.cpp:99)***
`glutPassiveMotionFunc(PassiveMouseCB);`
`glutKeyboardFunc(KeyboardCB);`

这里注册两个GLUT回调。一个用于监听鼠标事件，另一个监听常规键盘按下的事件（特殊键盘事件监听方向键和功能键的按下）。Passive motion意思是鼠标在没有按键按下情况下的移动。

***(tutorial15.cpp:81)***

```
static void KeyboardCB(unsigned char Key, int x, int y)
{
    switch (Key) {
        case 'q':
            exit(0);
    }
}

static void PassiveMouseCB(int x, int y)
{
    pGameCamera->OnMouse(x, y);
}
```
现在我们在全屏模式下，退出应用会不太方便，使用键盘事件回调监听Q键按下可以使应用退出。鼠标事件回调可以简单的将鼠标位置传给camera。

***(tutorial15.cpp:44)***

```
static void RenderSceneCB()
{
    pGameCamera->OnRender();
}
```
不管什么时候，只要开始调用主渲染循环就一定要通知camera，这样在鼠标位于屏幕边缘同时又不移动没有鼠标事件的时候相机仍可以继续旋转。
