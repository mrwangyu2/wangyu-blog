---
title: VTK与图形学相关的基本概念
date: 2017-10-26 09:09:24
tags:
- 图形图像
---

![](/assets/blog-image/vtk-t.jpg)
由  王宇 原创并发布 ：


## <font color=red>总结VTK学习路径</font>

* 下载编译VTK代码
* 阅读编译后Example目录中的Tutorial代码，理解pipeline
* 阅读VTK User‘s Guide。比较系统，但是不详细，详细内容还需进一步查阅wifi和代码
* 阅读博客中的各种VTK效果
* 图形学的基本概念
* 顶点
* 坐标
* 摄像机
* 颜色
* 光线
* 纹理
* 其他特效

## 参考资料
http://blog.csdn.net/www_doling_net/article/details/8763686

《VTK User’s Guide (11th Edition)》
《The Visualization Toolkit – AnObject-Oriented Approach To 3D Graphics (4th Edition)》


## 基本概念
<!--more-->
<font color=red> 管线、光线、色彩、摄像机、坐标</font>

* VTK 管线(pipeline)
![](/assets/blog-image/vtk-1.jpg)
* VTK包中Examples->Tutorial->Cone
```c
int main()
{
  //
  // Next we create an instance of vtkConeSource and set some of its
  // properties. The instance of vtkConeSource "cone" is part of a
  // visualization pipeline (it is a source process object); it produces data
  // (output type is vtkPolyData) which other filters may process.
  //
  vtkConeSource *cone = vtkConeSource::New();
  cone->SetHeight( 3.0 );
  cone->SetRadius( 1.0 );
  cone->SetResolution( 10 );

  //
  // In this example we terminate the pipeline with a mapper process object.
  // (Intermediate filters such as vtkShrinkPolyData could be inserted in
  // between the source and the mapper.)  We create an instance of
  // vtkPolyDataMapper to map the polygonal data into graphics primitives. We
  // connect the output of the cone souece to the input of this mapper.
  //
  vtkPolyDataMapper *coneMapper = vtkPolyDataMapper::New();
  coneMapper->SetInputConnection( cone->GetOutputPort() );

  //
  // Create an actor to represent the cone. The actor orchestrates rendering
  // of the mapper's graphics primitives. An actor also refers to properties
  // via a vtkProperty instance, and includes an internal transformation
  // matrix. We set this actor's mapper to be coneMapper which we created
  // above.
  //
  vtkActor *coneActor = vtkActor::New();
  coneActor->SetMapper( coneMapper );

  //
  // Create the Renderer and assign actors to it. A renderer is like a
  // viewport. It is part or all of a window on the screen and it is
  // responsible for drawing the actors it has.  We also set the background
  // color here.
  //
  vtkRenderer *ren1= vtkRenderer::New();
  ren1->AddActor( coneActor );
  ren1->SetBackground( 0.1, 0.2, 0.4 );

  //
  // Finally we create the render window which will show up on the screen.
  // We put our renderer into the render window using AddRenderer. We also
  // set the size to be 300 pixels by 300.
  //
  vtkRenderWindow *renWin = vtkRenderWindow::New();
  renWin->AddRenderer( ren1 );
  renWin->SetSize( 300, 300 );

  //
  // Now we loop over 360 degrees and render the cone each time.
  //
  int i;
  for (i = 0; i < 360; ++i)
  {
    // render the image
    renWin->Render();
    // rotate the active camera by one degree
    ren1->GetActiveCamera()->Azimuth( 1 );
  }

  //
  // Free up any objects we created. All instances in VTK are deleted by
  // using the Delete() method.
  //
  cone->Delete();
  coneMapper->Delete();
  coneActor->Delete();
  ren1->Delete();
  renWin->Delete();

  return 0;
}

```

* 光线
VTK里用类vtkLight来表示渲染场景中的光照。与现实中的灯光类似，VTK中的vtkLight实例也可以打开、关闭，设置光照的颜色，照射位置(即焦点)，光照所在的位置，强度等等。

  * 位置光照(Positional Light，即聚光灯)
  * 方向光照(Direction Light)

*  相机
观众的眼睛就好比三维渲染场景中的相机，VTK则是用vtkCamera类来表示三维渲染场景中的相机。vtkCamera负责把三维场景投影到二维平面，如屏幕、图像等。

  * 因素主要有：
  * 相机位置：即相机所在的位置，用方法vtkCamera::SetPosition()设置。
  * 相机焦点：用方法vtkCamera::SetFocusPoint()设置，默认的焦点位置在世界坐标系的原点。
  * 朝上方向：即哪个方向为相机朝上的方向。
  * 投影方向：相机位置到相机焦点的向量方向即为投影方向。
  * 投影方法：确定Actor是如何映射到像平面的。vtkCamera定义了两种投影方法:
  * 一种是正交投影(OrthographicProjection)，也叫平行投影(Parallel Projection)，即进入相机的光线与投影方向是平行的。
  * 另一种是透视投影(PerspectiveProjection)，即所有的光线相交于一点。

  * 视角：透视投影时需要指定相机的视角(View Angle)，默认的视角大小为30º，可以用方法vtkCamera::SetViewAngle()设置。
  * 前后裁剪平面：裁剪平面与投影方向相交，一般与投影方向也是垂直的。裁剪平面主要用于评估Actor与相机距离的远近，只有在前后裁剪平面之间的Actor才是可见的。
![](/assets/blog-image/vtk-2.jpg)
* 控制相机运动
vtkCamera除了提供设置与相机投影因素相关的方法之外，还提供了大量的控制相机运动的方法，如：vtkCamera::Dolly()，vtkCamera::Roll()，vtkCamera::Azimuth()，vtkCamera::Yaw()，vtkCamera::Elevation()，vtkCamera::Pitch()，vtkCamera::Zoom()。
![](/assets/blog-image/vtk-3.jpg)
* 颜色
前面的内容我们提到Actor的属性，颜色是Actor比较重要的属性之一。VTK采用RGB和HSV两种颜色系统来描述颜色。
RGB颜色系统就是由三个颜色分量：红色(R)、绿色(G)和蓝色(B)的组合表示，在VTK里这三个分量的取值都是从0到1，(0, 0, 0)表示黑色，(1, 1, 1)表示白色。vtkProperty::SetColor(r,g, b)采用的就是RGB颜色系统设置颜色属性值。
  * HSV
  * 色相(Hue)，是颜色的基本属性，就是我们平常所说的颜色名称，如红色、黄色等；
  * 饱和度(Saturation)，是指颜色的纯度，其值越高则越纯；
  * 值(Value，也就是强度Intensity或者亮度Bright)，值为0通常表示的是黑色，值为1表示的是最亮的颜色
![](/assets/blog-image/vtk-4.jpg)
* 坐标系统及空间变换
  * 坐标系统
  * Model坐标系统是定义模型时所采用的坐标系统，通常是局部的笛卡尔坐标系。例如，我们要定义一个表示球体的Actor，一般的做法是将该球体定义在一个柱坐标系统里。
  * World坐标系统是放置Actor的三维空间坐标系，Actor其中的一个功能就是负责将模型从Model坐标系统变换到World坐标系统。每一个模型可以定义有自己的Model坐标系统，但World坐标系只有一个，每一个Actor必须通过放缩、旋转、平移等操作将Model坐标系变换到World坐标系。World坐标系同时也是相机和光照所在的坐标系统。
  * View坐标系统表示的是相机所看见的坐标系统。X、Y、Z轴取值为[-1, 1]，X、Y值表示像平面上的位置，Z值表示到相机的距离。相机负责将World坐标系变换到View坐标系。
  * Display坐标系统跟View坐标系统类似，但是各坐标轴的取值不是[-1, 1]，而是使用屏幕像素值。屏幕上显示的不同窗口的大小会影响View坐标系的坐标值[-1, 1]到Display坐标系的映射。可以把不同的渲染场景放在同一个窗口进行显示，例如，在一个窗口里，分为左右两个渲染场景，这左右的渲染场景(vtkRenderer)就是不同的视口(Viewport)。
![](/assets/blog-image/vtk-5.jpg)
* 坐标系统使用
  在VTK里，Model坐标系统用得比较少，其他三种坐标系统经常使用。它们之间的变换则是由类vtkCoordinate进行管理的。根据坐标点单位、取值范围等不同，可以将坐标系统分为：
  * DISPLAY — X、Y轴的坐标取值为渲染窗口的像素值。坐标原点位于渲染窗口的左下角，这个对于VTK里所有的二维坐标系统都是一样的，且VTK里的坐标系统都是采用右手坐标系。
  * NORMALIZEDDISPLAY — X、Y轴坐标取值范围为[0, 1]，跟DISPLAY一样，也是定义在渲染窗口里的。
  * VIEWPORT— X、Y的坐标值定义在视口或者渲染器(Renderer)里。
  * NORMALIZEDVIEWPORT — X、Y坐标值定义在视口或渲染器里，取值范围为[0, 1]。
  * VIEW— X、Y、Z坐标值定义在相机所在的坐标系统里，取值范围为[-1, 1]，Z值表示深度信息。
  * WORLD — X、Y、Z坐标值定义在世界坐标系统，参考图3.9。
  * USERDEFINED— 用户自定义坐标系统。
* 空间变换
  我们在三维空间里定义的三维模型，最后显示的时候都是投影到二维平面，比如在屏幕上显示，生成二维图像等等。三维到二维的投影包括透视投影(PerspectiveProjection)和正交投影(Orthogonal Projection，正交投影也叫平行投影)。
  <font color=red>平移、缩放、旋转的转换矩阵计算,需参考资料</font>







