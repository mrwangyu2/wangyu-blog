---
title: OpenGL学习记录-基础篇
date: 2018-06-18 09:08:30
tags:
- 图形图像
---


![](/assets/blog-image/gljc-t.jpg)
由 王宇 原创并发布 ：


## 教程地址：
https://learnopengl-cn.github.io/
     
## OpenGL
* Khronos组织制定并维护的规范
* 立即渲染模式: OpenGL3.2 开始废弃
* 核心模式(Core-profile)
* 扩展
* 状态机(State Machine)
* 对象

<!--more-->

## 创建窗口
* 创建OpenGL上下文(Context),目前流行的库：GLUT、SDL、SFML、GLFW
* 构建GLFW
  * 下载地址： http://www.glfw.org/download.html
* 我们的第一个工程
  * 链接： 
  * Windows上OpenGL： opengl32.lib已经包含在Microsoft SDK里了
  * Linux上的OpenGL： 在Linux下你需要链接libGL.so库文件，这需要添加-lGL到你的链接器设置中

```c
-lGLEW -lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi
```

<!--more-->

* GLAD 配置
打开GLAD的在线服务，将语言(Language)设置为C/C++，在API选项中，选择3.3以上的OpenGL(gl)版本（我们的教程中将使用3.3版本，但更新的版本也能正常工作）。之后将模式(Profile)设置为Core，并且保证生成加载器(Generate a loader)的选项是选中的。现在可以先（暂时）忽略拓展(Extensions)中的内容。都选择完之后，点击生成(Generate)按钮来生成库文件。
GLAD现在应该提供给你了一个zip压缩文件，包含两个头文件目录，和一个glad.c文件。将两个头文件目录（glad和KHR）复制到你的Include文件夹中（或者增加一个额外的项目指向这些目录），并添加glad.c文件到你的工程中。
地址： http://glad.dav1d.de/

## 你好，窗口
## 你好，三角形
* 对象
  * 顶点数组对象：Vertex Array Object，VAO
  * 顶点缓冲对象：Vertex Buffer Object，VBO
  * 索引缓冲对象：Element Buffer Object，EBO或Index Buffer Object，IBO
* OpenGL的图形渲染管线-Graphics Pipeline 
大多译为管线，实际上指的是一堆原始图形数据途经一个输送管道，期间经过各种变化处理最终出现在屏幕的过程
  * 第一部分把你的3D坐标转换为2D坐标
  * 第二部分是把2D坐标转变为实际的有颜色的像素。
* 着色器
图形渲染管线接受一组3D坐标，然后把它们转变为你屏幕上的有色2D像素输出。图形渲染管线可以被划分为几个阶段，每个阶段将会把前一个阶段的输出作为输入。所有这些阶段都是高度专门化的（它们都有一个特定的函数），并且很容易并行执行。正是由于它们具有并行执行的特性，当今大多数显卡都有成千上万的小处理核心，它们在GPU上为每一个（渲染管线）阶段运行各自的小程序，从而在图形渲染管线中快速处理你的数据。这些小程序叫做着色器(Shader)。

* 图形渲染管线的每个阶段的抽象展示。要注意蓝色部分代表的是我们可以注入自定义的着色器的部分。图形渲染管线的每个阶段的抽象展示。要注意蓝色部分代表的是我们可以注入自定义的着色器的部分。
![](/assets/blog-image/gljc-zsq.jpg)
  * 顶点着色器 - Vertex Shader
  把一个单独的顶点作为输入。顶点着色器主要的目的是把3D坐标转为另一种3D坐标（后面会解释），同时顶点着色器允许我们对顶点属性进行一些基本处理。
  * 图元装配 - Primitive Assembly
  将顶点着色器输出的所有顶点作为输入，并所有的点装配成指定图元的形状
  * 几何着色器 - Geometry Shader
  把图元形式的一系列顶点的集合作为输入，它可以通过产生新顶点构造出新的（或是其它的）图元来生成其他形状。例子中，它生成了另一个三角形。
  * 光栅化阶段 - Rasterization Stage
  它会把图元映射为最终屏幕上相应的像素，生成供片段着色器(Fragment Shader)使用的片段(Fragment)。在片段着色器运行之前会执行裁切(Clipping)。裁切会丢弃超出你的视图以外的所有像素，用来提升执行效率。
  * 片段着色器
  主要目的是计算一个像素的最终颜色，这也是所有OpenGL高级效果产生的地方。通常，片段着色器包含3D场景的数据（比如光照、阴影、光的颜色等等），这些数据可以被用来计算最终像素的颜色。
  * Alpha测试和混合(Blending)阶段
  这个阶段检测片段的对应的深度（和模板(Stencil)）值（后面会讲），用它们来判断这个像素是其它物体的前面还是后面，决定是否应该丢弃。这个阶段也会检查alpha值（alpha值定义了一个物体的透明度）并对物体进行混合(Blend)。所以，即使在片段着色器中计算出来了一个像素输出的颜色，在渲染多个三角形的时候最后的像素颜色也可能完全不同。

* 顶点输入
  * 标准化设备坐标(Normalized Device Coordinates, NDC) - 为-1.0到1.0的范围内
  
  * 三角形的顶点：
  ```c
  float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
  };
  ```
  ![](/assets/blog-image/gljc-2.jpg)
  
  * 顶点缓冲对象 (Vertex Buffer Object)
  定义这样的顶点数据以后，我们会把它作为输入发送给图形渲染管线的第一个处理阶段：顶点着色器。它会在GPU上创建内存用于储存我们的顶点数据，还要配置OpenGL如何解释这些内存，并且指定其如何发送给显卡。顶点着色器接着会处理我们在内存中指定数量的顶点。
我们通过顶点缓冲对象(Vertex Buffer Objects, VBO)管理这个内存，它会在GPU内存（通常被称为显存）中储存大量顶点。使用这些缓冲对象的好处是我们可以一次性的发送一大批数据到显卡上，而不是每个顶点发送一次。从CPU把数据发送到显卡相对较慢，所以只要可能我们都要尝试尽量一次性发送尽可能多的数据。当数据发送至显卡的内存中后，顶点着色器几乎能立即访问顶点，这是个非常快的过程。
我们可以使用glGenBuffers函数和一个缓冲ID生成一个VBO对象

```c
unsigned int VBO;
glGenBuffers(1, &VBO);
```
  * glBindBuffer函数把新创建的缓冲绑定到GL_ARRAY_BUFFER目标上
  ```c
  glBindBuffer(GL_ARRAY_BUFFER, VBO);  
  ```
  * glBufferData函数，它会把之前定义的顶点数据复制到缓冲的内存中
  ```c
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  ```
* 顶点着色器

```c
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

* GLSL顶点着色器的源代码：
  * 第一行版本声明
  * in 关键字，表述输入
  * layout (location = 0) : 在存储中的起始位置
  * 向量： 方向 + 大小
    最多4个分量： vec.x vec.y vec.z vec.w
  * 预定义变量gl_Position
  会成为着色器的输出
  
* 编译着色器
  * 创建着色器对象
  ```c
  unsigned int vertexShader;
  vertexShader = glCreateShader(GL_VERTEX_SHADER);
  ```
  * 将着色器源代码附加到着色器对象上
  ```c
  glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
  glCompileShader(vertexShader);
  ```
  * 检测编译是否成功
  ```c
  int  success;
  char infoLog[512];
  glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
  if(!success)
  {
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
  }
  ```
* 片段着色器
片段着色器所做的是计算像素最后的颜色输出
  * 颜色表示
  在计算机图形中颜色被表示为有4个元素的数组：红色、绿色、蓝色和alpha(透明度)分量，通常缩写为RGBA。当在OpenGL或GLSL中定义一个颜色的时候，我们把颜色每个分量的强度设置在0.0到1.0之间。比如说我们设置红为1.0f，绿为1.0f，我们会得到两个颜色的混合色，即黄色。这三种颜色分量的不同调配可以生成超过1600万种不同的颜色！
  ```c
  #version 330 core
  out vec4 FragColor;

  void main()
  {
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
  } 
  ```
  * 编译片段着色器的过程与顶点着色器类似，只不过我们使用GL_FRAGMENT_SHADER常量作为着色器类型
  
* 着色器程序
  * 着色器程序对象(Shader Program Object)
  是多个着色器合并之后并最终链接完成的版本。如果要使用刚才编译的着色器我们必须把它们链接(Link)为一个着色器程序对象，然后在渲染对象的时候激活这个着色器程序。已激活着色器程序的着色器将在我们发送渲染调用的时候被使用。
  * 创建一个程序对象
  ```c
  unsigned int shaderProgram;
  shaderProgram = glCreateProgram();
  ```
  * 链接
  ```c
  glAttachShader(shaderProgram, vertexShader);
  glAttachShader(shaderProgram, fragmentShader);
  glLinkProgram(shaderProgram);
  ```
  * 检测链接
  ```c
  glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
  if(!success) {
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    ...
  }
  ```
  * 激活
  ```c
  glUseProgram(shaderProgram);
  ```
  * 删除着色器对象
  ```c
  glDeleteShader(vertexShader);
  glDeleteShader(fragmentShader);
  ```
* 链接顶点属性
顶点着色器允许我们指定任何以顶点属性为形式的输入。这使其具有很强的灵活性的同时，它还的确意味着我们<font color=red>必须手动指定输入数据的哪一个部分对应顶点着色器的哪一个顶点属性。</font>所以，我们必须在渲染前指定OpenGL该如何解释顶点数据。
![](/assets/blog-image/gljc-3.jpg)

  * glVertexAttribPointer函数告诉OpenGL该如何解析顶点数据
  ```c
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
  glEnableVertexAttribArray(0);
  ```
    * 第一个参数指定我们要配置的顶点属性。还记得我们在顶点着色器中使用layout(location = 0)定义了position顶点属性的位置值(Location)吗？它可以把顶点属性的位置值设置为0。因为我们希望把数据传递到这一个顶点属性中，所以这里我们传入0。
    * 第二个参数指定顶点属性的大小。顶点属性是一个vec3，它由3个值组成，所以大小是3。
    * 第三个参数指定数据的类型，这里是GL_FLOAT(GLSL中vec*都是由浮点数值组成的)。
    * 下个参数定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。
    * 第五个参数叫做步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。由于下个组位置数据在3个float之后，我们把步长设置为3 * sizeof(float)。要注意的是由于我们知道这个数组是紧密排列的（在两个顶点属性之间没有空隙）我们也可以设置为0来让OpenGL决定具体步长是多少（只有当数值是紧密排列时才可用）。一旦我们有更多的顶点属性，我们就必须更小心地定义每个顶点属性之间的间隔，我们在后面会看到更多的例子（译注: 这个参数的意思简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
    * 最后一个参数的类型是void*，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。由于位置数据在数组的开头，所以这里是0。我们会在后面详细解释这个参数。

  * 启用顶点属性: glEnableVertexAttribArray
  * <font color=red>小结</font>：
  顶点属性默认是禁用的。自此，所有东西都已经设置好了：我们使用一个顶点缓冲对象将顶点数据初始化至缓冲中，建立了一个顶点和一个片段着色器，并告诉了OpenGL如何把顶点数据链接到顶点着色器的顶点属性上。在OpenGL中绘制一个物体，代码会像是这样：
  ```c
  // 0. 复制顶点数组到缓冲中供OpenGL使用
  glBindBuffer(GL_ARRAY_BUFFER, VBO);
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  // 1. 设置顶点属性指针
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
  glEnableVertexAttribArray(0);
  // 2. 当我们渲染一个物体时要使用着色器程序
  glUseProgram(shaderProgram);
  // 3. 绘制物体
  someOpenGLFunctionThatDrawsOurTriangle();
  ```
  
* 顶点数组对象
每当我们绘制一个物体的时候都必须重复这一过程(复制顶点对象到缓冲区)。这看起来可能不多，但是如果有超过5个顶点属性，上百个不同物体呢（这其实并不罕见）。绑定正确的缓冲对象，为每个物体配置所有顶点属性很快就变成一件麻烦事。有没有一些方法可以使我们把所有这些状态配置储存在一个对象中，并且可以通过绑定这个对象来恢复状态呢？
可以像顶点缓冲对象那样被绑定，任何随后的顶点属性调用都会储存在这个VAO中。这样的好处就是，当配置顶点属性指针时，你只需要将那些调用执行一次，之后再绘制物体的时候只需要绑定相应的VAO就行了。这使在不同顶点数据和属性配置之间切换变得非常简单，只需要绑定不同的VAO就行了
  * 一个顶点数组对象会储存以下这些内容
    * glEnableVertexAttribArray和glDisableVertexAttribArray的调用。
    * 通过glVertexAttribPointer设置的顶点属性配置。
    * 通过glVertexAttribPointer调用与顶点属性关联的顶点缓冲对象。
![](/assets/blog-image/gljc-4.jpg)

  * 创建一个VAO
  ```c
  unsigned int VAO;
  glGenVertexArrays(1, &VAO);
  ```
  
  * 小结
 <font color=red>VAO指定VBO的使用方式， VBO和EBO都是存储空间， EBO存索引， VBO存属性，例如坐标、颜色等 
举例说明：拿数据的视图和表为例，视图就是VAO, 表就是VBO, 索引就是EBO</font>  

  ```c
  // ..:: 初始化代码（只运行一次 (除非你的物体频繁改变)） :: ..
  // 1. 绑定VAO
  glBindVertexArray(VAO);
  // 2. 把顶点数组复制到缓冲中供OpenGL使用
  glBindBuffer(GL_ARRAY_BUFFER, VBO);
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  // 3. 设置顶点属性指针
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
  glEnableVertexAttribArray(0);

  [...]

  // ..:: 绘制代码（渲染循环中） :: ..
  // 4. 绘制物体
  glUseProgram(shaderProgram);
  glBindVertexArray(VAO);
  someOpenGLFunctionThatDrawsOurTriangle();
  ```
  前面做的一切都是等待这一刻，一个储存了我们顶点属性配置和应使用的VBO的顶点数组对象。一般当你打算绘制多个物体时，你首先要生成/配置所有的VAO（和必须的VBO及属性指针)，然后储存它们供后面使用。当我们打算绘制物体的时候就拿出相应的VAO，绑定它，绘制完物体后，再解绑VAO。
  
* 我们一直期待的三角形
```c
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, 3);
```
![](/assets/blog-image/gljc-5.jpg)

* 索引缓冲对象-Element Buffer Object，EBO，也叫Index Buffer Object，IBO
  * 举例：假设我们不再绘制一个三角形而是绘制一个矩形。我们可以绘制两个三角形来组成一个矩形（OpenGL主要处理三角形）
  ```c
  float vertices[] = {
    // 第一个三角形
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, 0.5f, 0.0f,  // 左上角
    // 第二个三角形
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
  };
  ```
  指定了右下角和左上角两次！一个矩形只有4个而不是6个顶点，这样就产生50%的额外开销。
  * 解决方案:
  解决方案是只储存不同的顶点，并设定绘制这些顶点的顺序。这样子我们只要储存4个顶点就能绘制矩形了，之后只要指定绘制的顺序就行了。
  ```c
  float vertices[] = {
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
  };

  unsigned int indices[] = { // 注意索引从0开始! 
    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
  };
  ```
  当时用索引的时候，我们只定义了4个顶点，而不是6个。
  * 创建索引缓冲对象
  ```c
  unsigned int EBO;
  glGenBuffers(1, &EBO);
  ```
  * 绑定并复制EBO
  ```c
  glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
  glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
  ```
  * 绘制EBO
  ```c
  glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
  glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
  ```
  * VAO 保存 EBO
  ![](/assets/blog-image/gljc-6.jpg)

  * 小结
  ```
  // ..:: 初始化代码 :: ..
  // 1. 绑定顶点数组对象
  glBindVertexArray(VAO);
  // 2. 把我们的顶点数组复制到一个顶点缓冲中，供OpenGL使用
  glBindBuffer(GL_ARRAY_BUFFER, VBO);
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
  // 3. 复制我们的索引数组到一个索引缓冲中，供OpenGL使用
   glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
  glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
  // 4. 设定顶点属性指针 
  glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
  glEnableVertexAttribArray(0);
  
  [...]
  
  // ..:: 绘制代码（渲染循环中） :: ..
  glUseProgram(shaderProgram);
  glBindVertexArray(VAO);
  glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)
  glBindVertexArray(0);
  ```
  ![](/assets/blog-image/gljc-7.jpg)

