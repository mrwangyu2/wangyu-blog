---
title: OpenGL学习记录-纹理
date: 2018-06-19 09:08:30
tags:
- 图形图像
---


![](/assets/blog-image/wl-t.jpg)
由 王宇 原创并发布 ：

## 纹理

* 纹理概述
为了能够把纹理映射(Map)到三角形上，我们需要指定三角形的每个顶点各自对应纹理的哪个部分。这样每个顶点就会关联着一个<font color=red>纹理坐标(Texture Coordinate)</font>，用来标明该从纹理图像的哪个部分<font color=red>采样</font>（译注：采集片段颜色）。之后在图形的其它片段上进行片段<font color=red>插值</font>(Fragment Interpolation)。

纹理坐标在x和y轴上，范围为0到1之间（注意我们使用的是2D纹理图像）。使用纹理坐标获取纹理颜色叫做<font color=red>采样(Sampling)</font>。纹理坐标起始于(0, 0)，也就是纹理图片的左下角，终始于(1, 1)，即纹理图片的右上角。

<!--more-->

* 纹理环绕方式
纹理坐标的范围通常是从(0, 0)到(1, 1)，那如果我们把纹理坐标设置在范围之外会发生什么？

  * 环绕方式
GL_REPEAT    对纹理的默认行为。重复纹理图像。
GL_MIRRORED_REPEAT    和GL_REPEAT一样，但每次重复图片是镜像放置的。
GL_CLAMP_TO_EDGE    纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。
GL_CLAMP_TO_BORDER    超出的坐标为用户指定的边缘颜色。

* 纹理过滤
纹理坐标不依赖于分辨率(Resolution)，它可以是任意浮点值，所以OpenGL需要知道怎样将纹理像素(Texture Pixel，也叫Texel，译注1)映射到纹理坐标。当你有一个很大的物体但是纹理的分辨率很低的时候这就变得很重要了。你可能已经猜到了，OpenGL也有对于纹理过滤(Texture Filtering)的选项。纹理过滤有很多个选项，但是现在我们只讨论最重要的两种：GL_NEAREST和GL_LINEAR。
  
  * GL_NEAREST
  GL_NEAREST（也叫邻近过滤，Nearest Neighbor Filtering）是OpenGL默认的纹理过滤方式。当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。下图中你可以看到四个像素，加号代表纹理坐标。左上角那个纹理像素的中心距离纹理坐标最近，所以它会被选择为样本颜色：

   ![](/assets/blog-image/wl-gl-1.jpg)

  * GL_LINEAR
  GL_LINEAR（也叫线性过滤，(Bi)linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。下图中你可以看到返回的颜色是邻近像素的混合色：
  
   ![](/assets/blog-image/wl-gl-2.jpg)

    GL_NEAREST产生了颗粒状的图案，我们能够清晰看到组成纹理的像素，而GL_LINEAR能够产生更平滑的图案，很难看出单个的纹理像素
  
  * 多级渐远纹理
  OpenGL使用一种叫做<font color=red>多级渐远纹理(Mipmap)</font>的概念来解决这个问题，它简单来说就是一系列的纹理图像，后一个纹理图像是前一个的二分之一。多级渐远纹理背后的理念很简单：距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。由于距离远，解析度不高也不会被用户注意到。同时，多级渐远纹理另一加分之处是它的性能非常好。让我们看一下多级渐远纹理是什么样子的：
  
   ![](/assets/blog-image/wl-ncfp.jpg)
  
    * 过滤方式
    GL_NEAREST_MIPMAP_NEAREST    使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样
    GL_LINEAR_MIPMAP_NEAREST    使用最邻近的多级渐远纹理级别，并使用线性插值进行采样
    GL_NEAREST_MIPMAP_LINEAR    在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样
    GL_LINEAR_MIPMAP_LINEAR    在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行

    ```c
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    ```

  * 加载与创建纹理
  选一个需要的文件格式，比如.PNG，然后自己写一个图像加载器，把图像转化为字节序列。写自己的图像加载器虽然不难，但仍然挺麻烦的，而且如果要支持更多文件格式呢？你就不得不为每种你希望支持的格式写加载器了。
另一个解决方案也许是一种更好的选择，使用一个支持多种流行格式的图像加载库来为我们解决这个问题。比如说我们要用的<font color=red>stb_image.h</font>库。

  * 生成纹理
    * 生成： glGenTextures
    * 绑定： glBindTextrures
    * 设置环绕、过滤方式： glTextParameteri
    * 加载纹理: stb_image
    
```c
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// 为当前绑定的纹理对象设置环绕、过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);   
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// 加载并生成纹理
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```
    
  * 应用纹理
    * 跟新坐标
    需要告知OpenGL如何采样纹理，所以我们必须使用纹理坐标更新顶点数据：
    
    ```c
    float vertices[] = {
    //     ---- 位置 ----       ---- 颜色 ----     - 纹理坐标 -
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
    };
    ```
    * 内存布局
    ![](index_files/gl_texture_vertex.png)

    ```
    glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
    glEnableVertexAttribArray(2);
    ```

    * 调整顶点着色器
    需要调整顶点着色器使其能够接受顶点坐标为一个顶点属性，并把坐标传给片段着色器

    ```c
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;
    layout (location = 2) in vec2 aTexCoord;

    out vec3 ourColor;
    out vec2 TexCoord;

    void main()
    {
      gl_Position = vec4(aPos, 1.0);
      ourColor = aColor;
      TexCoord = aTexCoord;
    }
    ```

* 设置片段着色器采样

```c
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}
```

使用GLSL内建的texture函数来采样纹理的颜色


* 绑定纹理

```c
glBindTexture(GL_TEXTURE_2D, texture);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```
    
* 纹理单元
纹理单元的主要目的是让我们在着色器中可以使用多于一个的纹理。通过把纹理单元赋值给采样器，我们可以一次绑定多个纹理，只要我们首先激活对应的纹理单元。

```c
glActiveTexture(GL_TEXTURE0); // 在绑定纹理之前先激活纹理单元
glBindTexture(GL_TEXTURE_2D, texture);
```

