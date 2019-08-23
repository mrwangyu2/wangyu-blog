---
title: 深度探索C++对象模型-关于对象
date: 2013-09-26 09:21:05
tags:
- cpp
---

由 王宇 原创并发布 ：


让我肃然起敬并崇拜的牛人：

       Bjarne Stroustrup    -  C++之父    (www.stroustrup.com)

       Stanley B. Lippman  -  本书作者、C++代言人、C++编译器的设计者之一

       侯捷                         -  本书译者   (jjhou.boolan.com)

第一章 关于对象

        C++在布局及存取时间上主要的额外负担是由virtual引起的。

         两个概念：

                  1) virtual function机制： 用以一种有效的“执行期绑定”（runtime binding）

                  2) virtual base class:    用以实现“多次出现在继承体系中的 base class, 有一个单一而被共享的实体”

1.1 C++对象模型(The C++ Object Model)

            类数据成员(class data memeber): 静态(static)和非静态的(nonstatic)

            类成员函数(class member function):静态(static)函数、非静态函数(nonstatic)和虚(virtual)函数  


 

           在C++对象模型中，Nonstatic data memeber 被配置与每一个class object之内，static data members则被存放在所有的class object之外，static和nonstatic function members也被放在所有的class object之外，virtual functions则以两个步骤支持之：

                      1、每个class产生出一堆指向virtual function的指针，放在表格之中。这个表格被成为virtual table(vtbl).

                      2、每个class object被添加了一个指针，指向相关的vitrual table.通常这个指针被称为vptr. vptr的设置和重置都由每一个class的constructor、destructor和copy assignment运算符自动完成。每一个class所关联的type_info object(用于支持runtime type identification,RTTI)也经由vitual talbe被指出来，通常是放在表格的第一个slot处。


             优点：它的空间和存取时间的效率

             缺点：如果应用程序代码本身未曾改变，但所用到的class object的nonstatic data memeber有所改变，那么那些应用程序代码同样得重新编译。

            对象模型如何影响程序：

                       class X 定义了一个copy constructor, 一个virtual destructor,和一个virtual function foo:
 

X foobar()
{
    X xx;
    X *px = new X;
    //foo() 是一个virtual function
    xx.foo();
    px->foo();

    delete px;
    return xxx;
}
 

                     这个函数可能在内部转换为：


void foobar(X &_result)
{
    ...
    //扩展xx.foo()但不使用virtual机制
    //以 _result 取代xx
    foo(&_result);

    //使用virtual机制扩展px->foo()
    ( *px->vtbl[ 2 ] )( px );
    ...
}
 

                    结论是：

                                 以下方式不能使用virtual机制：

                                    X xx;
                                    xx.foo()

                                 以下方式可以使用virtual机制：

                                    X *px = new X;
                                    px->foo();

1.2 关键词所带来的差异

                 关键词的困扰：

                         在C++观念上，struct 是可以替代class的，但仍然声明public、protected、private等等存取区段。 

                策略性正确的struct:

                        C程序员的巧计有时候却成为C++程序员的陷阱。例如把单一元素的数组放在一个struct的尾端，于是每个struct objects可以拥有可变大小的数组：

                  struct mumberl{
                                 /* stuff */
                                 char pc [1];
                   };

                       如果改用class来声明，在C++中凡处于同一个access section的数据，必定保证以其声明次序出现在内存布局当中，然而被放置在多个access section中的各笔数据，排列次序就不一定了，所以最好的忠告就是：不要那么做。

        
1.3 对象的差异：
    
                 C++ 程序设计模型直接支持三种程序设计典范(programming paradigms)

                        1)程序模型(procedural model)，就像C一样

                        2)抽象数据类型模型(abstract data type model,ADT),该模型所谓的“抽象”是和一组表达式(public接
口)一起提供，而其运算定义任然隐而未明。

                        3)面向对象模型

                需要多少内存才能够表现一个class:

                      1)其nonstatic data members的总和大小

                      2)加上任何由于alignment的需求而填补上去的空间。(alignment就是将数值调整到某数的倍数。在32位计算机上，通常alignment为4bype,以使bus的“运输量”达到最高效率)(我在此处理解为对齐内存中的对象。)

                      3)加上为了支持virtual而由内部产生的任何额外负担

              指针的类型：

                     寻址出来的object类型不同。也就是说，“指针类型”会教导编译器如何解释某个特定地址中的内存内容及其大小
    
                    例：

                           int    *p        地址：1000 - 1003
                           string *pstr  地址：1000 - 1015
                           viod   *pV    地址：指向1000，而地址空间我们不知道！这就是为什么一个类型为void*的指针只能够含有一个地址，而不能通过它操作所指之object的缘故。必须进行转型。

                     转型(cast)其实是一种编译器指令。大部分情况下它并不改变一个指针所含的真正地址，它只影响"被指出之内存的大小和其内容"的解释方式

                加上多态之后的布局：


    class Bear : public ZooAnimal{
        public:
            Bear();
            ~Bear();
            void rotate();
            virtual void dance();
        protected:
            enum Dances{ .. };
           
            Dances dances_known;
            int cell_block;  
    };
 

                             Bear b("Yogi");
                             Bear *pb = &b;
                             Bear &rb = *pb;

    

 

                             假设一：
                                        Bear b;
                                        ZooAnimal *pz = &b;
                                        Bear *pb = &b;

                                        差别是pb所涵盖的地址包含整个Bear object,而pz所涵盖的地址只包含Bear object中的ZooAnimal suboject

                                       除了virtual机制，pz不能指向Bear 中的函数
        
                                       pz的类型将在编译时期决定一下两点：
                                               (1)固定的可用接口，也就是说pz只能够调用ZooAnimal的public接口
                                               (2)该接口的access level
        
                              假设二：


                                        Bear b;
                                        ZooAnimal za = b;//这会引起切割(sliced)

                                         za.rotate();    //调用ZooAnimal::rotate();    
        
                                       为什么za的vptr不指向Bear的virtual table？

                                                za并不是(而且也绝不会是)一个Bear,它是(并且只能是)一个ZooAnimal。多态所造成的“一个以上的类型”的潜在力量，并不能够实际发挥在"直接存取 object"这件事情上。有一个似是而非的观念：OO程序设计并不支持对object的直接处理

                                      ZooAnimal za;
                                      ZooAniall *pza;
    
                                      Bear b;
                                      Panda *pp = new Panda;

                                      pza = &b;

    

 

    
                                        一个pointer或一个reference之所以支持多态，是因为它们并不引发内存中任何“与类型有关的内存委托操作(type-dependent commitment)”;会受到改变的只是他们所指向的内存的“大小和内容解释方式（参考上面的‘指针的类型’）”而已






 




