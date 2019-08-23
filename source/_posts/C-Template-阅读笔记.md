---
title: C++ Template 阅读笔记
date: 2012-05-26 09:28:31
tags:
- cpp
---

由  王宇 原创并发布 ：


## 第2章函数模板 

### 2.1初探函数模板 

#### 2.1.1定义模板
```cpp
template<typename T>
inline T max(T const&a,T const&b){}
```

class,typename是等价的，推荐使用typename

#### 2.1.2使用模板:函数模板的隐形使用：max(a,b);

具体类型替代模板参数的过程叫做实例化

<!--more-->

### 2.2实参的演绎 

调用参数的类型构造自模板参数，所以模板参数和调用参数通常是相关的。我们把这个概念称为：函数模板的实参演绎

max(4,7); //ok隐形

max(4,4.2); //error T不能够自动转换

解决办法：

* (1)max(static_cast<double>(4),4.2);

* (2)max<double>(4,4.2);//显式

* (3)指定两个参数可以具有不同的类型

### 2.3模板参数 

template<typenameT>//T模板参数，可以定义多个不同的参数    

```cpp
template<typename RT,typename T1,typename T2>
inline RT max(T1 const &a,T2 const&b);
max<double>(4,4.2); //RT为double,T1为int,T2为double
```

### 2.4重载函数模板 

对于非模板函数和同名的函数模板，如果其他条件都是相同的话，那么在调用的时候，重载解析过程通常会调用非模板函数，而不会从该模板产生出一个实例

模板是不允许自动类型转化的；但普通函数可以进行自动类型转换，所以最后一个调用将使用非模板函数

```cpp
inline int const &max(int const &a,int const &b){}

template<typename T>

inline T const &max(T const &a,T const &b){}

::max(7,42);//使用非模板函数

::max('a',42.7);//使用非模板函数
```



## 第3章类模板 

### 3.1类模板Stack的实现 

#### 3.1.1类模板的声明

```cpp
template<typename T> //可以使用class
class Stack{};
```

#### 3.1.2成员函数的实现

voidStack<T>::push(Tconst&elem);

### 3.2类模板Stack的使用 

```cpp
Stack<int> intStack;
Stack<std::string> strStack;
Stack<Stack<int>> intStackStack; //Error，>>不允许使用，中间需放入一个空格，> >
```

### 3.3类模板的特化 

为了特化一个类模板，你必须在起始处声明一个template<>,接下来声明用来特化类模板的类型
```cpp
template<>
class Stack<std::string>{};
```

### 3.4局部特化 

* 相同类型：

```cpp
template<typename T>

class MyClass<T,T>{};
```

* int参数：

```cpp
template<typename T>

class MyClass<T,int>{};
```

* 指针类型：

```cpp
template<typename T1, typename T2>
classMyClass<T1*,T2*>
```

* 使用：

```cpp
MyClass<int,float>mif;//MyClass<T1,T2>

MyClass<float,float>mif;//MyClass<T,T>

MyClass<float,int>mif;//MyClass<T,int>

MyClass<int*,float*>mp;//MyClass<T1*,T2*>
```

### 3.5缺省模板实参

```cpp
template<typename T,typename CONT=std::vector<T> >

classStack{

  CONT elems;

  void push(T const &elem);

}
```





第4章非类型模板参数 

4.1非类型的类模板参数

template<typenameT,intMAXSIZE>

claseStack{

  if(MAXSIZE>0){};

}



Stack<int,20>intStack;

4.2非类型的函数模板参数 

template<typenameT,intVAL>

TaddValue(Tconst&x){returnT+VAL;};

4.3非类型模板参数的限制 

不能使用浮点数(double)和类对象，以及全局指针作为模板参数，这是历史原因。



第5章技巧性基础知识 

5.1关键字typename 

引入关键字typename是为了说明：模板内部的标识符可以是一个类型

template<typenameT>

classMyClass{

  typenameT::SubType*ptr;//声明一个指针

}




5.2使用this-> 

对于具有基类的类模板，自身使用名称x并不一定等同于this->x.即使该x是从基类继承获得的

template<typenameT>

classBase{

  public:

    voidexit();

};

template<typenameT>

classDerived:Base<T>{

  voidfoo(){

    exit();//Error,应该使用this->exit

  }

};




5.3成员模板 

重载操作符=

5.4模板的模板参数 

让模板参数本身成为模板是很有用的

template<typenameT,template<typenameELEM>classCONT=std::deque>

5.5零初始化

template<typenameT>

voidfoo(){

  intx;//error

  Tx;//error

  Tx=T();//ok

}




5.6使用字符串作为函数模板的实参 

template<typenameT>

inlineTconst&max(Tconst&a,Tconst&b){};//可以修改成(Tconsta,Tconstb)//导致无用的拷贝

::max("apple","peach");//OK相同类型，长度一样

::max("apple","tomato");//Error不同类型，长度不同

std::strings;

::max("apple",s);//Error不同类型

无通用的解决办法：

1、使用非引用参数，取代引用参数，导致无用的拷贝

2、进行重载，编写接收引用参数和非引用参数的两个重载函数，这可能会导致二义性

3、对具体类型进行重载

4、强制要求应用程序程序员使用显示类型转换

第6章模板实战 

6.1包含模型 

6.1.1链接器错误

把模板放在doc-C（cpp）文件中使用，链接器报错，找不到函数，原因是没有被实例化。

6.1.2头文件中的模板

把模板的定义放在头文件中，这种组织方式为包含模式

不足：增加的头文件的开销

6.2显式实例化 

由关键字template和紧接其后的我们所需要实例化的实体（可以是类、函数、成员函数等）的声明组成，而且，该声明是一个已经用实参完全替换参数之后的声明            

缺点：我们必须仔细跟踪每个需要实例化的实体。对于大项目而言，这种跟踪很快就会带来巨大负担：因此，我们不建议使用这种方式

优点：实例化可以在需要的时候才进行

6.22整合包含模型和显式实例化

6.3分离模型,（不建议使用） 

导出模板，这种机制通常也被称为C++模板的分离模型

6.3.1关键字export

6.4模板和内联 

对于许多不属于类定义一部分的短小模板函数，你应该使用关键字inline类声明它们

6.5预编译头文件



6.6调试模板



第7章模板术语 

7.1“类模板”还是“模板类” 

class和struct的唯一区别在于：缺省访问权限：class:private;struct:public.C++推荐class,C推荐struct

                                                                7.2实例化和特化 

模板实例化是一个通过使用具有值替换模板实参，从模板产生出普通类、函数或者成员函数的过程。这个过程最后获得的实体（譬如类、函数或者成员函数）就是我们通常所说的特化(specialization)

  7.3声明和定义 

  声明：引用了一个名字到某个作用域中

  定义：声明并分配内存

  7.4一处定义原则 

  ODR原则：

  和全局变量与静态数据成员一样，在整个程序中，非内联函数和成员函数只能被定义一次

  类类型和内联函数在每个翻译单元中最多只能被定义一次，如果存在多个翻译单元，则其所有的定义都必须是等同的

  7.5模板实参和模板参数 

  模板参数：位于模板声明或定义内部，关键字template后面所列举的名字

  模板实参：用来替换模板参数的各个对象

  一个基本原则：模板实参必须是一个可以在编译期确定的模板实体或者值



  第14章模板与设计 

  14.1动多态 

  继承和虚函数实现的多态

  14.2静多态 

  模板实现的多态

  14.3动多态和静多态 

  通过继承实现的多态是绑定的和动态的

  绑定：对于参与多态行为的类型，它们的接口是在公共基类的设计中就预先确定的

  动态：接口的绑定是在运行期完成的

  通过模板实现的多态是非绑定的和静态的

  非绑定：对于参与多态行为的类型，它们的接口是没有预先确定的

  静态：接口的绑定是在编译期完成的

  与动多态相比，静多态被认为具有更好的类型安全性    

  14.4新形式的设计模板

  14.5泛型程序设计 

  泛型程序设计定义为运用模板的程序设计，就像面向对象的程序设计被看成是运用虚函数的程序设计

  在一个框架中，设计模板的目的是为了能够得到多种有用的组合（类型）

  STL实际上是一个框架，它提供了许多有用的操作，我们也把这些操作称为算法；它同时也为对象集合提供了许多线性数据结构，我们把这些数据结构称为容器，而且，算法和容器都是模板。



  第15章trait与plicy类 

  15.1一个实例：累加一个序列 

  15.1.1fixedtrait

  使用模板，实现不同类型(intcharshortfloat)的累加

  定义：定义模板，并特化：

  template<typenameT>

  classAccumulationTraits;

  template<>

  classAccumulationTraits<char>{

    public:

      typedefcharAccT;

  };

classAccumulationTraits<int>{

  public:

    typedefintAccT;

};

classAccumulationTraits<double>{

  public:

    typedefdoubleAcct;

};




实现模板函数：

template<typenameT>

AccumulationTraits<T>::AccTaccum(Tconst*beg,Tconst*end){

  typedeftypenameAccumulationTrains<T>::AccTAccT;//特化演绎

  AccTtotal=Acct();

  while(beg!=end){

    total+=beg;

    ++beg;

  }

}




15.1.2valuetrait

--    缺点：C++只允许我们对整形和枚举类型初始化静态成员变量    

15.1.3参数化trait

15.1.4policy和policy类

针对accum()的所有操作，唯一需要改变的只是total+=*beg操作，于是，我们就把这个操作称为该累积过程的一个policy.因此，一个policy类就是一个提供了一个接口的类，该操作能够在算法中应用一个或多个policy

template<typenameT,typenamePlicy=SumPolicy,typenameTraits=AccumulationTraits<T>>

classAccum{

  public:

    typedeftypenameTraits::AccTAccT;

    staticAccTaccum(Tconst*beg,Tconst*end)

    {

      Accttotal=Traits::zero();

      while(beg!=end){

        Policy::accumulate(total,*beg);//this is policy

        ++beg;

      }

    }

};




15.1.5trait和policy：区别

trait字面上：用来刻划一个事物的(与众不同)特性

policy字面上：为了某种有益或有利的目的而采用的一系列动作

policy更加注重于行为，而trait则更加注重于类型

实现形式上：

trait是用特化演绎

policy是模板参数 

15.2类型函数 

15.2.1确定元素的类型

15.2.2确定class类型

15.2.3引用和限定符

15.3policytrait 



第16章模板与继承 

16.1命名模板参数

16.2空基类优化

16.3奇特的递归模板模式

16.4参数化虚拟性



第17章metaprogram 

metaprogramming含有“对一个程序进行编程”的意思，换句话说，编程系统将会执行我们所写的代码，来生成新的代码，而这些新代码才真正实现了我们所期望的功能。（递归模板）

17.1metaprogram的第一个实例

template<intN>

classPow3{

  public:

    enum{result=3*Pow3<N-1>::result};

};

template<>

classPow3<0>{

  public:

    enum{result=1};

};




Pow3<>模板，包括特化，被称为一个metagrogramming

17.2枚举值和静态常量

staticintconstFour=4;

静态成员变量只能是左值。

17.3第2个例子：计算平方根

17.4使用归纳变量

17.5计算完整性

17.6递归实例化和递归模板实参

17.7使用metaprogram来展开循环



第18章表示式模板 

18.1临时变量和分割循环

18.2在模板实参中编码表达式

18.3表达式模板的性能与约束
