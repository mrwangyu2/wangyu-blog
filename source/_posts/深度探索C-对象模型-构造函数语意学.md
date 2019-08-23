---
title: 深度探索C++对象模型-构造函数语意学
date: 2013-10-10 09:20:44
tags:
- cpp
---

由 王宇 原创并发布 ：


让我肃然起敬并崇拜的牛人：

       Bjarne Stroustrup    -  C++之父    (www.stroustrup.com)

       Stanley B. Lippman  -  本书作者、C++代言人、C++编译器的设计者之一

       侯捷                         -  本书译者   (jjhou.boolan.com)




第二章 构造函数语言学

    英文术语：

        implicit  :暗中、隐含的（通常意指并非在程序源代码中出现的）

        explicit  :明确（通常意指程序源代码中所出现的）
        
        trival    :没有用的

        nontrival :有用的
        
        memberwise:对每一个member施以...

        bitwise   :对每一个bit施以...

        semantics :语意


    2.1 Default Constructor 的建构操作

        “default constructors... 在需要的时候被编译器产生出来”。关键字是“在需要的时候”

        “带有 Default Constructor”的Member Class Object:
            
            如果一个class 没有任何constructor,但它内含一个member object, 而后者有default constructor, 那么这个class的implicit default constructor就是nontrivial，编译器需要为此class合成出一个default constructor。不过这个合成操作只有在constructor真正需要被调用时才会发生

            例：

              class Foo
                {
                    public:
                        Foo(); //构造函数
                    ...
                };

                class Bar
                {
                    public:
                        Foo foo; //是一个member object, 而其class Foo 拥有default constructor.
                        char *str;
                };
   
                void foo_bar()
                {
                    Bar bar //合成constructor
                }
                
                注意：被合成的default constructor只满足编译器的需要，而不是程序的需要。    

        “带有Default Constructor”的Base Class
            
            如果一个没有任何constructor的class派生自一个“带有default constructor”的base class，那么这个derived class的default constructor会被视为nontrivial，并因此需要被合成出来。它将调用上一层base classes的default constructor（根据它们的声明次序）。对一个后派生的class而言，这个合成的constructor和一个“被明确提供的default constructor”没有什么差异。

        “带有一个Virtual Function”的Class
        

        “带有一个Virtual Base Function”的Class

            另有两种情况，也需要合成出default constructor
                (1)class声明(或继承)一个 virtual function
                (2)class派生自一个继承串链，其中有一个或更多的virtual base classes

        总结：

            C++ Stardand 把那些合成物称为：implicit nontrivial default constructor。被合成出来的constructor只能满足编译器的需要，而非程序的需要。它之所以能够完成任务，是借着“调用member object或base class的default constructor”或是“为每一个object初始化其virtual function机制或virtual base class机制”而完成。至于没有存在那四种情况而又没有声明任何constructor的class，我们说它们拥有的是implicit trivial default constructor，它们实际上并不会被合成出来。
    
            在合成default constructor中，只有base class subobject和member class objects会被初始化。所有其它的nonstatic data member，如整数、整数指针、整数数组等等都不会被初始化。这些初始化操作对程序而言或许有需要，但对编译器则并非必要。

            C++新手一般有两个常见的误解：

                (1)任何class如果没有定义default constructor，就会被合成出一个来。
                (2)编译器合成出来的default constructor会明确设定“class 内每一个data member的默认值”

        结论：无论是自定义或合成出来的构造函数，均需要自己初始化数据成员(除了member class objects)。

     2.2 Copy Constructor建构操作
        
        有三种情况，会以一个object的内容作为另一个class object的初值：
           

            class X { ... };
            X x;

            X xx = x;               // 情况1,赋值对象
           
            extern void foo( X x);

            void bar()
            {
               
                X xx;

                foo( xx );         // 情况2，作为参数
       
            }

            X foo_bar()
            {

                X xx;
       
                return xx;         // 情况3，作为返回值
            }
        
     
                
        Default Memberwise Initalization

            如果class 没有提供一个explicit copy constructor又当如何？当class object 以 “相同class的另一个object”作为初值时，其内不是以所谓的default memberwise initalization手法完成的，也就是把每一个内建的或派生的data member的值，从某个object拷贝一份到另一个object身上。不过它不会拷贝其中的member class object, 而是以递归的方式实行memberwise initalization.
            
            例子：


                class String
                {

                    public:
                   
                        //..没有explicit copy constructor

                    private:

                        char *str;
                        int   len;
                   
                };           
           
                class Word
                {
                    public:
                   
                        //..没有explicit copy constructor
                    private:

                        int _occurs;
                        String _word;  //String object成为class word的一个member. 此处以递归的方式实行memberwise initalization.
                                  // Word 是否合成 copy constructor 取决于 bitwise copy semantics.
                                //此例子不合成copy constructor 编译器会自动复制每一个数据成员
                };
 
                指出一个错误概念：“如果一个class未定义copy constructor，编译器就自动为它产生出一个”这句话不对
                正确的概念：Default constructor 和 copy constructor在必要的时候才由编译器产生出来。“必要”意指当class不展现bitwise copy semantics时。

        Bitwise Copy Semantics（位逐次拷贝）

            上例展示了Bitwise copy Semantics.

            有一点很值得注意：在被合成出来的copy constructor中，如整数、指针、数组等等的nonclass memebers也都会被复制，正如我们所期待的一样。    

        不要Bitwise Copy Semantics
        
            有四种情况不展示Bitwise Copy Semantics, 不展示的时候需要编译器合成copy constructor:

                (1)当class内含一个member object而后者的class声明有一个copy constructor时
                (2)当class继承自一个base class 而后者存在有一个copy constructor时
                (3)当class声明了一个或多个virtual functions时
                (4)当class派生自一个继承串链，其中有一个或多个virtual base classes时

        结论：如果是自定义复制构造函数时，需要自己把每一个数据成员复制；如果是没有自定义复制构造函数，无论是合成或非合成，编译器都会自动复制每一个数据成员。复制构造函数的用途是：如果构造函数中存在动态内存分配，则必须定义复制构造函数，否则会出现“指针悬挂问题”。


        class A
        {
            private:
                int *p;

            public:
                A()
                {
                    p = new int(3);
                }
        };    
 
        
        在这种情况下，复制对象，会造成两个对象的成员指向同一地址。

        重新设定Virtual Table的指针
    
            例子：


                class ZooAninal
                {
                    public:
                        ZooAnimal();
                        virtual ~ZooAnimal();

                        virtual void animate();
                        virtual void draw();       
                };

                class Bear : public ZooAnimal()
                {
                   
                    public:
                        Bear();
                        void animate();
                        void draw();   
                        virtual void dance();

                };

 
                Bear yogi;
                Bear winnie = yogi;

                把yogi 的vptr值拷贝给winnie的vptr是安全的

                

 

                ZooAnimal franny = yogi; // 这会发生切割行为

                

 

                合成出来的ZooAinmal copy constructor会明确设定object的vptr指向ZooAnimal class的virtual table,而不是直接从右手边的class object中将其vptr现值拷贝过来。
        
        处理Virtual Base Class Subobject

    2.3 程序转化语意学(Program Transformation Semantics)
        
        明确的初始化操作(Explicit initialization)    
   

            X x0;

            void foo_bar()
            {

                X x1(x0);       //定义了x1
                X x2 = x0;      //定义了x2
                X x3 = x(x0);   //定义了x3
            }
             
            //可能的程序转换
            // C++ 伪码   
       
            void foo_bar()
            {

                X x1;  //定义被重写
                X x2;  //定义被重写
                X x3;  //定义被重写
               
                //编译器安插X copy construction的调用操作
                x1.X::X( x0 );
                x2.X::X( x0 );
                x3.X::X( x0 );
            }
 
        

            总结：程序转化有两个阶段：1、重写每一个定义 2、class的copy constructor调用操作会被安插进去

        参数的初始化(Argument Initialization)
        
            把一个class object当做参数传递给一个函数，相当于：X xx = arg;


            void foo( X x0 );

            X xx;

            foo(xx);

            //C++ 伪码
           
            X __temp0;

            //编译器对copy constructor的调用
            __temp0.X::X(xx);

            //重新改写函数调用操作，以便使用上述的暂时对象
            foo(__temp0);
 
            
            总结：编译器采用暂时性object策略,并调用copy constructor将它初始化，然后将该暂时性object交给函数。

        返回值的初始化(Return Value Initialization)
   

            X bar()
            {

                X xx;
                //处理xx ...
                return xx;
            }   

            //函数转换
            //C++伪码

            void bar( X& __result) // 加上了一个额外参数   
            {

                X xx;
               
                //编译器所产生的default constructor调用操作
                xx.X::X();

                //处理xx ...

               
                //编译器所产生的copy constructor调用操作
                __result.X::XX(xx);
            }   
           

            ///////////////////

            X xx = bar(); 

            //被转换为：

            X xx;
            bar( xx );

            ///////////////////

            bar().memfunc();

            //被转换为：

            X __temp0;
           
            (bar( __temp0 ), __temp0).memfunc(); 

            ////////////////////

            x( *pf )();
            pf = bar;

            //被转换为：

            void ( *pf )( X& );
            pf = bar;
 
            
        Copy Constructor:要还是不要？

            正常情况下不需要，没有任何理由要你提供一个copy constructor函数实体，因为编译器自动为你实施了最好的行为。除非需要解决类似“指针悬挂”等问题时，需要Copy Constructor

    2.4 成员们的初始化队伍
        
        何时使用initialization list才有意义：
        
            1、当初始化一个reference member时
            2、当初始化一个const member时

                例子：


                class Shape
                {
                     const int m_size; //const 常量
                     int & m_ref;      // reference member

                     float m_width;
                     float m_height;

                    public:
                     Shape(int s,float w,float h):m_size(s),m_ref(s) //只能在这初始化
                     {
                      //m_size =s; //在初始化将出错
                      m_width = w;
                      m_height = h;
                     }
                     
                };
 
                
            3、当调用一个base class的constructor，而它拥有一组参数时
                
                例子：   

                class A
                {
                    A(int x);          // A 的构造函数
                };

                class B : public A
                {
                    B(int x, int y);   // B 的构造函数
                };

                B::B(int x, int y): A(x)   // 必须在初始化表里调用基类 A 的构造函数
                {
                    …
                }
 
                
            4、当调用一个member class的constructor，而它拥有一组参数时    
                
                例子：
                    
 

               class A
                {
                    A(int x);          // A 的构造函数
                };

                class B
                {
                    A m_a;
                    B(int x, int y);   // B 的构造函数
                };

                B::B(int x, int y): m_a(x) // 在初始化表里调用基类 A 的构造函数
                {
                    …
                    m_a = A(x);  //赋值，并非初始化m_a
                }
 
        initialization list的真正操作是什么：
        
            例子：

                class Word
                {
                    String _name;
                    int    _cnt;
                    public:
                        Word()  
                        {   
                            _name = 0;  // 可以编译功过并运行，但是效率低下
                            _cnt  = 0;  // 是否使用initialization list 都是相同的
                        }   
                };

                // C++ 伪码

                Word::Word()
                {
                    //调用String的 default constructor
                    _name.String::String();

                    //产生暂时性对象
                    String temp = String(0);

                    // "memberwise"地拷贝_name
                    _name.String::operator=( temp );   
                   
                    //摧毁暂时性对象
                    temp.String::~String();

                    _cnt = 0;

                }

 


            在这里，Word constructor会产生一个暂时性的String object，然后将它初始化，再以一个assignment运算符将暂时性object指定给_name,然后再摧毁那个暂时性object


                //较佳的方式
                Word::Word(): _name( 0 )
                {
                    _cnt = 0;
                }
       
                // C++ 伪码
                Word::Word()
                {
                    //调用String(int) constructor
                    _name.String::String( 0 );
                    _cnt = 0;                     //这里是：是否使用initialization list 都是相同的原因

                }


 
        initialization list的顺序：

            (1) list中的项目次序是由class中的members声明次序决定，不是由initialization list中的排序次序决定

            例子：
               

                class X
                {
                    int i;
                    int j;

                    public:
                        X(int val) : j( val ), i( j ) //注意这里有一个陷阱，i值为一个不可预知未初始化的值。原因是按照定义的顺序，i先被初始化，这个时候j还没有被初始化。
                };    
 

            (2) initialization list的项目被放在explicit user code之前

            例子：

                class X
                {
                    int i;
                    int j;

                    public:
                        X(int val) : j( val )
                        {
                            i = j;  //是正确的，原因是initialization list的项目被放在explicit user code之前，及这里j已经在构造函数之前初始化了。

                        }
                };   
 


            (3) 调用一个member function 以设定一个member的初值    
                
            例子：
  

                class X
                {
                    int i;
                    int xfoo( int val)
                    {
                        return val;
                    }

                    public:
                        X(int val) : i ( xfoo(val) ) //这里是正确的
                        {

                        }
                };   


                // C++ 伪码
                X::X()
                {
                    i = this->xfoo( val );

                }

           
                















