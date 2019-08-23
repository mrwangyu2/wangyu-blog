---
title: 深度探索C++对象模型-Data语意学
date: 2014-02-08 09:20:21
tags:
- cpp
---

由 王宇 原创并发布 ：


让我肃然起敬并崇拜的牛人：

Bjarne Stroustrup    -  C++之父    (www.stroustrup.com)

  Stanley B. Lippman  -  本书作者、C++代言人、C++编译器的设计者之一

侯捷                         -  本书译者   (jjhou.boolan.com)



  第三章 Data 语意学

  3.1 Data Member的绑定

  例：

  extern float x;

  class Point3d
{
  public:
    point3d();
    //问题：被传回和被设定的x是哪一个x呢？
    float X() const
    {
      return x;
    }

  private:
    float x, y, z;//Point3d::X()将返回内部的x。
};


在早期(2.0之前)C++的编译器上，将会指向global x object, 导致C++的两种防御性程序设计风格：    
1、把所有的data members放在class 声明起头处，以确保正确的绑定
2、把所有的inline functions, 不管大小都放在class声明之外

对于 member function的argument list并不为真（同以上情况相反）。

例:


typedef int length;

class Point3d
{
  public:
    //length 将被决议为global
    //_val将被决议为Point3d::_val
    void mumbel(length val)
    {    
      _val = val;
    }

    length mumble()
    {
      return _val;
    }

  private:
    //导致 _val = val; return _val;不合法    
    typedef float length;
    length _val;    
};


预防性程序风格：请始终把“nested type 声明”放在class的起始处。

3.2 Data Member的布局
Nonstatic data member 在class object中的排列顺序将和其被声明的顺序一样，任何中间介入的static data members 如 freelist 和 chunkSize都不会被放入对象布局中

传统上vptr放在所有明确声明member之后

3.3 Data Member的存取
Point3d origin, *pt = &origion;
origin.x = 0.0;
pt->x = 0.0;

这两种方式的差异？

Static Data Members:
结论是完全相同, 原因是：每一个static data member 只有一个实体， 存放在程序的data segment之中, member其实并不在class object之中。

Nonstatic Data Members:
结论是完全相同，原因是：
pt->x = 0.0; 方式：事实上是经由一个“implicit class object”(由this指针表达)完成


//member function 的内部转化
  Point3d
Point3d::translate( Point3d *const this, const Point3d &pt)
{
  this->x = pt.x;
}


origin.x = 0.0; 方式：class object的起始地址加上data member的偏移量(offset)
&origin + (&Point3d::_y - 1);

3.4 "继承"与 Data Member    
一个derived class object所表现出来的东西， 是其自己的member加上其base class(es) members的总和。图3.1b



只要继承不要多态

继承易犯的错误：
(1)重复设计一些相同的操作函数
(2)把一个class分解为两层或更多层，有可能会为了“表现class体系之抽象化”而膨胀所需空间。（因为类型对齐，会浪费空间）    图p105    

填充空间的原因是为了复制对象时，避提高效率,免产生偏差。

加上多态

空间和存取时间的额外负担：
(1)导入一个virtual table 和一个vptr
(2)加强constructor 和distructor 使其能够初始化和抹消virtual table和vptr    。

把vptr放置在class object的那里会最好？
(1)放在尾部保留C struct 的对象布局，C++最初期，这种方式被许多人采用。
(2)放在前端，对于“在多重继承之下，通过指向class members的指针调用virtual function”，会带来一些帮助。当然代价就是丧失了C语言兼容性。

多重继承

多重继承的复杂度在于derrived class 和其上一个base class乃至于上上一个base class...之间的“非自然”关系。参考下列代码：


class Point2d
{
  public:
  protected:
    float _x, _y;    

};

class Point3d : public Point2d
{
  public:
  protected:
    float _z;    

};

class Vertex
{
  public:
  protected:
    Vertex *next;    
};

class Vertex3d : public Point3d, public Vertex
{
  public:
  protected:
    float mumble;    

};


下列操作并不需要编译器去调停或修改地址，它很自然地可以发生，而且提供了最佳执行效率。
Point3d p3d;
Point2d *p = &p3d;    

多重继承的问题主要发生于derived class object 和其第二或后续的base class object之间的转换：

Vertex3d  v3d;
Vertex    *pv;
Point2d   *p2d;
Point3d   *p3d;

pv = &v3d;
// 需要这样的内部转换：
pv = (Vertex*)(((char*)&v3d) + sizeof(Point3d));

// 下列转换只需要简单地拷贝地址就行了
p2d = &v3d;    
p3d = &v3d;



参考图:3.4 p115

C++ Standard 并未要求Vertex3d中的base classes Point3d和Vetex有特定的排列次序。原始的cfont编译器是根据声明次序来排序它们。目前各编译器仍然是以此方式完成多重base classes的布局（但如果加上虚拟继承，就不一样了）

虚拟继承

多重继承的一个语意上的副作用就是，它必须支持某种形式的“shared subobject继承”。参考图p117



不论是istream或ostream都内含一个ios subobject。然而在iosstream的对象布局中，我们只需要单一一份ios subobject就好。语言层面的解决办法是导入所谓的虚拟继承

3.5 对象成员的效率
应该确实地测量效率，而不是靠着推论与常识判断。

3.6 指向Data Members的指针
class Point
{
  public:
    virtual ~Point();
    static Point origion;
    float x,y,z;
};

如果vptr放在对象的尾端，则三个坐标值在对象布局中的offset分别是0,4,8。如果vptr放在对象的起头，则三个坐标值在对象布局中的offset分别是4,8,12.然而你若去取data members的地址， 传回的值总是多1， 就是1,5,9,或5,9,13

如何区分一个“没有指向任何data member”的指针和一个指向“第一个data member”的指针？考虑下列例子：
Point3d p3d;
Point3d *p1 = 0;
Point3d *P2 = &p3d;

if( p1 == p2){};

为了区分p1 和p2 ,每一个真正的member offset值都被加上1. 因此，无论编译器或使用者都必须记住，在真正使用该值以指出一个member之前，请先减掉1。








