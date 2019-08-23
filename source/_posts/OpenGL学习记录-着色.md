---
title: OpenGL学习记录-着色
date: 2018-06-20 09:08:30
tags:
- 图形图像
---


![](/assets/blog-image/zsq-t.jpg)
由 王宇 原创并发布 ：

## 着色器 - 运行在GPU上的小程序
从基本意义上来说，着色器只是一种把输入转化为输出的程序。着色器也是一种非常独立的程序，因为它们之间不能相互通信；它们之间唯一的沟通只有通过输入和输出。

* GLSL - 包含一些针对向量和矩阵操作的有用特性。

```c
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

<!--more-->

* 数据类型
  * GLSL中包含C等其它语言大部分的默认基础数据类型：int、float、double、uint和bool
  * GLSL也有两种容器类型，分别是向量(Vector)和矩阵(Matrix)
* 向量
GLSL中的向量是一个可以包含有1、2、3或者4个分量的容器，分量的类型可以是前面默认基础类型的任意一个。它们可以是下面的形式（n代表分量的数量）：
  * 一个向量的分量可以通过vec.x这种方式获取
  * 重组：分量选择方式
  
  ```c
  vec2 someVec;
  vec4 differentVec = someVec.xyxx;
  vec3 anotherVec = differentVec.zyw;
  vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
  ```
  
|类型|含义|
|:---|:---|
|vecn|包含n个float分量的默认向量|
|bvecn|包含n个bool分量的向量|
|ivecn|包含n个int分量的向量|
|uvecn|包含n个unsigned int分量的向量|
|dvecn|包含n个double分量的向量|
  
* 输入与 输出
虽然着色器是各自独立的小程序，但是它们都是一个整体的一部分，出于这样的原因，我们希望每个着色器都有输入和输出，这样才能进行数据交流和传递。GLSL定义了in和out关键字专门来实现这个目的。每个着色器使用这两个关键字设定输入和输出，只要一个输出变量与下一个着色器阶段的输入<font color=red>匹配</font>，它就会传递下去。但在顶点和片段着色器中会有点不同。
  * 顶点着色器应该接收的是一种特殊形式的输入,它从顶点数据中直接接收输入
    * location: 这一元数据指定输入变量，这样我们才可以在CPU上配置顶点属性。这一元数据指定输入变量，这样我们才可以在CPU上配置顶点属性。
    * layout: 顶点着色器需要为它的输入提供一个额外的标识，这样我们才能把它链接到顶点数据
  * 另一个例外是片段着色器，它需要一个vec4颜色输出变量，因为片段着色器需要生成一个最终输出的颜色。
  * 顶点着色器
  
  ```c
  #version 330 core
  layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0
  
  out vec4 vertexColor; // 为片段着色器指定一个颜色输出
  
  void main()
  {
      gl_Position = vec4(aPos, 1.0); // 注意我们如何把一个vec3作为vec4的构造器的参数
      vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // 把输出变量设置为暗红色
  }
  ```
  
  * 片段着色器
  
  ```c
  #version 330 core
  out vec4 FragColor;
  
  in vec4 vertexColor; // 从顶点着色器传来的输入变量（名称相同、类型相同）
  
  void main()
  {
      FragColor = vertexColor;
   }
  ```
  ![](/assets/blog-image/zsq-1.jpg)

* Uniform
Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式

  * uniform和顶点属性有些不同:
    * uniform是全局的(Global)。全局意味着uniform变量必须在每个着色器程序对象中都是独一无二的，而且它可以被着色器程序的任意着色器在任意阶段访问。
    * 无论你把uniform值设置成什么，uniform会一直保存它们的数据，直到它们被重置或更新。
    * 例子
    
    ```c
    #version 330 core
    out vec4 FragColor;
    
    uniform vec4 ourColor; // 在OpenGL程序代码中设定这个变量

    void main()
    {
        FragColor = ourColor;
    }
    ```
    随时间变化赋值
    
    ```c
    float timeValue = glfwGetTime();
    float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
    int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
    glUseProgram(shaderProgram);
    glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
    ```
    * 使用uniform 渲染
    
    ```c
    while(!glfwWindowShouldClose(window))
    {
        // 输入
        processInput(window);

        // 渲染
        // 清除颜色缓冲
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 记得激活着色器
        glUseProgram(shaderProgram);

        // 更新uniform颜色
        float timeValue = glfwGetTime();
        float greenValue = sin(timeValue) / 2.0f + 0.5f;
        int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
        glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);

        // 绘制三角形
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

        // 交换缓冲并查询IO事件
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    ```
* 更多属性
    * 将色彩加入到顶点数据中
  
    ```c
     float vertices[] = {
        // 位置              // 颜色
         0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
        -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
         0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
      };
    ```

    * 调整一下顶点着色器, 用layout标识符来把aColor属性的位置值设置为1

    ```c
    #version 330 core
    layout (location = 0) in vec3 aPos;   // 位置变量的属性位置值为 0 
    layout (location = 1) in vec3 aColor; // 颜色变量的属性位置值为 1

    out vec3 ourColor; // 向片段着色器输出一个颜色

    void main()
    {
        gl_Position = vec4(aPos, 1.0);
        ourColor = aColor; // 将ourColor设置为我们从顶点数据那里得到的输入颜色
    }
    ```

    * 修改一下片段着色器.不再使用uniform来传递片段的颜色了，现在使用ourColor输出变量

    ```c
    #version 330 core
    out vec4 FragColor;  
    in vec3 ourColor;

    void main()
    {
        FragColor = vec4(ourColor, 1.0);
    }
    ```

    * VBO内存中的数据

    ![](/assets/blog-image/zsq-vbo.jpg)
    
    * 使用glVertexAttribPointer函数更新顶点格式

    ```c
    // 位置属性
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    // 颜色属性
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
    glEnableVertexAttribArray(1);
    ```
    ![](/assets/blog-image/zsq-2.jpg)
    

* 我们自己的着色器类
  * 从硬盘读取着色器，然后编译并链接它们，并对它们进行错误检测
  
  ```c
    #ifndef SHADER_H
    #define SHADER_H

    #include <glad/glad.h>; // 包含glad来获取所有的必须OpenGL头文件

    #include <string>
    #include <fstream>
    #include <sstream>
    #include <iostream>
    class Shader
    {
    public:
        // 程序ID
        unsigned int ID;

        // 构造器读取并构建着色器
        Shader(const GLchar* vertexPath, const GLchar* fragmentPath);
        // 使用/激活程序
        void use();
        // uniform工具函数
        void setBool(const std::string &name, bool value) const;  
        void setInt(const std::string &name, int value) const;   
        void setFloat(const std::string &name, float value) const;
    };

    #endif
  ```

  use用来激活着色器程序，所有的set…函数能够查询一个unform的位置值并设置它的值。
  
* 从文件读取
  * 使用C++文件流读取着色器内容，储存到几个string对象里使用C++文件流读取着色器内容，储存到几个`string`对象里
  
  ```c
  Shader(const char* vertexPath, const char* fragmentPath)
{
    // 1. 从文件路径中获取顶点/片段着色器
    std::string vertexCode;
    std::string fragmentCode;
    std::ifstream vShaderFile;
    std::ifstream fShaderFile;
    // 保证ifstream对象可以抛出异常：
    vShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);
    fShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);
    try 
    {
        // 打开文件
        vShaderFile.open(vertexPath);
        fShaderFile.open(fragmentPath);
        std::stringstream vShaderStream, fShaderStream;
        // 读取文件的缓冲内容到数据流中
        vShaderStream << vShaderFile.rdbuf();
        fShaderStream << fShaderFile.rdbuf();       
        // 关闭文件处理器
        vShaderFile.close();
        fShaderFile.close();
        // 转换数据流到string
        vertexCode   = vShaderStream.str();
        fragmentCode = fShaderStream.str();     
    }
    catch(std::ifstream::failure e)
    {
        std::cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << std::endl;
    }
    const char* vShaderCode = vertexCode.c_str();
    const char* fShaderCode = fragmentCode.c_str();
    [...]
  ```
  * 编译和链接着色器
  
  ```c
  // 2. 编译着色器
    unsigned int vertex, fragment;
    int success;
    char infoLog[512];

    // 顶点着色器
    vertex = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertex, 1, &vShaderCode, NULL);
    glCompileShader(vertex);
    // 打印编译错误（如果有的话）
    glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
    if(!success)
    {
        glGetShaderInfoLog(vertex, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
    };

    // 片段着色器也类似
    [...]

    // 着色器程序
    this->Program = glCreateProgram();
    glAttachShader(this->Program, vertex);
    glAttachShader(this->Program, fragment);
    glLinkProgram(this->Program);
    // 打印连接错误（如果有的话）
    glGetProgramiv(this->Program, GL_LINK_STATUS, &success);
    if(!success)
    {
        glGetProgramInfoLog(this->Program, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
    }

    // 删除着色器，它们已经链接到我们的程序中了，已经不再需要了
    glDeleteShader(vertex);
    glDeleteShader(fragment);
  ```
 * use函数非常简单
 
 ```c
 void Use()
{
    glUseProgram(this->Program);
}void Use() { glUseProgram(this->Program); }
 ```
 * uniform的setter函数也很类似
 
 ```c
 void setBool(const std::string &name, bool value) const
{         
    glUniform1i(glGetUniformLocation(ID, name.c_str()), (int)value); 
}
void setInt(const std::string &name, int value) const
{ 
    glUniform1i(glGetUniformLocation(ID, name.c_str()), value); 
}
void setFloat(const std::string &name, float value) const
{ 
    glUniform1f(glGetUniformLocation(ID, name.c_str()), value); 
}
 ```
 * 使用这个着色器类很简单；只要创建一个着色器对象，从那一点开始我们就可以开始使用了：
 
 ```c
 Shader ourShader("path/to/shaders/shader.vs", "path/to/shaders/shader.fs");
...
while(...)
{
    ourShader.use();
    ourShader.setFloat("someUniform", 1.0f);
    DrawStuff();
}
 ```

