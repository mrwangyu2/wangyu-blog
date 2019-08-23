---
title: 通过一个程序代码来总结Objective-C的知识点
date: 2016-04-28 09:14:00
tags:
- cpp
---

![](/assets/blog-image/objective-c-t.jpg)
由 王宇 原创并发布 ：


以下代码在Xcode 7.2.1 环境下编译通过

<!--more-->

 
```cpp
//
//  main.m
//  DemoObjectiveC
//
//  Decrible: 通过一个程序代码来总结Objective-C的知识点
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

// import 关键字导入当前目录中的头文件
#import "DemoBasicGrammar.h"
#import "DemoClass.h"
#import "DemoInheritance.h"
#import "DemoCategories.h"
#import "DemoProtocol.h"
#import "DemoDelegation.h"
#import "DemoFoundationFramework.h"
#import "DemoMemoryManagement.h"


// main 程序的入口点，参数同C语言一致, argc 程序参数的个数，argv参数数组
int main(int argc, const char * argv[]) {
    //内存管理机制
    @autoreleasepool {
        
        // @为字符串常量
        NSLog(@"Hello, World!");
        // 定义一个对象指针
        DemoBasicGrammar *demoGrammar;
        DemoClass *demoClass;
        DemoInheritance *demoInheritance;
        
        OtherTasks1 *otherTask1;
        OtherTasks1 *otherTask2;
        DemoDelegation *demoDelegation;
        
        DemoFoundationFramework *demoFoundationFramework;
        DemoMemoryManagement *demoMemoryManagement;
        
        int number;
        
        do {
            NSLog (@"---------------------");
            NSLog (@"[1] Demo Basic Grammar");
            NSLog (@"[2] Demo Class");
            NSLog (@"[3] Demo Inheritance ");
            NSLog (@"[4] Demo Categories");
            NSLog (@"[5] Demo Protocol and Delegation");
            NSLog (@"[6] Demo Foundation Framework");
            NSLog (@"[7] Demo Memory Management");
            NSLog (@"[0] Exit");
            NSLog (@"----------------------");
            
            NSLog (@"Enter your number.");
            number = 0;
            scanf ("%i", &number);
            
            switch (number) {
                case 1:
                    // 建立一个实例
                    demoGrammar = [DemoBasicGrammar alloc];
                    
                    // sent message 类似与调用类的成员函数, 但是有所区别。函数调用是编译期间即已经决定，而sent message为Runtime时决定成员方法的使用。
                    [demoGrammar demoDataType];
                    [demoGrammar demoExpressions];
                    [demoGrammar demoLogic];
                    [demoGrammar demoArray];
                    [demoGrammar demoEnumerate];
                    [demoGrammar demoStructures];
                    [demoGrammar demoBlocks];
                    
                    break;
                case 2:
                    // 静态方法, 这里没有实例化，直接使用静态方法
                    [DemoClass demoStaticFunc];
                    
                    // 实例对象并初始化
                    demoClass = [[DemoClass alloc] init];
                    // 多参数，demoMultipleParameter为方法名称， "1"为第一参数值(实参) parameterSecond为第二个参数
                    [demoClass demoMultipleParameter: 1 parameterSecond: 2];
                    
                    break;
                case 3:
                    // DemoInheritance 继承DemoClass
                    // 实例化
                    demoInheritance = [DemoInheritance alloc];
                    // 初始化
                    demoInheritance = [demoInheritance initWith: 1 over: 2];
                    
                    // 重写方法, OC 默认实现类似于C++ 的Virtual方法的多态
                    [demoInheritance demoOverriding];
                    
                    break;
                case 4:
                    demoClass = [[DemoClass alloc] init];
                    // 提供一种模块化的扩展方法
                    [demoClass demoCategories];
                    break;
                case 5:
                    demoDelegation = [DemoDelegation alloc];
                    
                    otherTask1 = [OtherTasks1 alloc];         // 任务一
                    
                    demoDelegation.delegate = otherTask1;     // 将任务一对象赋值给委托类的属性
                    [demoDelegation executeDelegationTasks];  // 执行委托
                    
                    otherTask2 = [OtherTasks2 alloc];         // 任务二
                    demoDelegation.delegate = otherTask2;     // 将任务二对象赋值给委托类的属性
                    [demoDelegation executeDelegationTasks];  // 执行委托
                    
                    break;
                case 6:
                    demoFoundationFramework = [DemoFoundationFramework alloc];
                    
                    [demoFoundationFramework demoNumberObjects];
                    [demoFoundationFramework demoStringObjects];
                    [demoFoundationFramework demoArray];
                    [demoFoundationFramework demoDictionary];
                    [demoFoundationFramework demoSet];
                    [demoFoundationFramework demoFileManager];
                    //Foundation’s NSData class provides an easy way to set up a buffer, read the contents of the file into it, or write the contents of a buffer out to a file.
                    [demoFoundationFramework demoNSData];
                    
                    [demoFoundationFramework demoArchivingWithXMLProperty];
                    [demoFoundationFramework demoArchivingWithNSKeyedArchiver];
                    
                    // 序列化 反序列化
                    [demoFoundationFramework demoEncodeAndDecode];
                    break;
                case 7:
                    demoMemoryManagement = [DemoMemoryManagement alloc];
                    
                    [demoMemoryManagement demoManualReferenceCounting];
                    [demoMemoryManagement demoAutoReleasePool];
                    [demoMemoryManagement demoAutoReleasepoolBlocks];
                    
                    
                    [demoMemoryManagement demoCopyObjects];
                    break;
                default:
                    break;
            }
        
        }while ( number != 0 );
        
    }
        
    return 0;
}
//
//  DemoBasicGrammar.h
//  DemoObjectiveC
//  Describle: 头文件，用于定义和声明
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
// import 关键字导入Fooundation Framework, 主要是封装了Numbers NSString Collections等内容
// 导入一共有三种方式：1. #include 兼容C语言 2.#import OC 专用 3. @class + 类名 告诉编译器这是一个类
#import <Foundation/Foundation.h>
#import "DemoGlobalDefine.h"

// 预处理
#define TRUE 1
#define FALSE 0


// OC 的程序代码结构主要有三个section, 分别是@interface section 用于定义 @implementation section 用于实现 @program section 主程序
//---- @interface section ----
@interface DemoBasicGrammar: NSObject
-(void) demoDataType;
-(void) demoExpressions;
-(void) demoLogic;
-(void) demoArray;
-(void) demoEnumerate;
-(void) demoStructures;
// funciton pointers blocks
-(void) demoBlocks;
@end
//
//  DemoBasicGrammar.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "DemoBasicGrammar.h"

//---- @implementation section ----
@implementation DemoBasicGrammar {
    int numerator;
    int denominator;
}

-(void) demoDataType {
    MyLog (@"Demonstated basci data type.");
    // OC 主要有四种基本的数据类型
    int integerVar = 100;
    float floatingVar = 331.79;
    double doubleVar = 8.44e+11;
    char charVar = 'W';
    
    NSLog (@"integerVar = %i", integerVar);
    NSLog (@"floatingVar = %f", floatingVar);
    NSLog (@"doubleVar = %e", doubleVar);
    NSLog (@"doubleVar = %g", doubleVar);
    NSLog (@"charVar = %c", charVar);
}

-(void) demoExpressions{
    MyLog (@"Demonstated expressions.");
    int a = 100;
    int b = 2;
    int c = 25;
    int d = 4;
    int result;
    
    result = a - b;
    NSLog (@"a - b = %i", result);
    result = b * c; // multiplication
    NSLog (@"b * c = %i", result);
    result = a / c; // division
    NSLog (@"a / c = %i", result);
    result = a + b * c; // precedence
    
    NSLog (@"a + b * c = %i", result);
    NSLog (@"a * b + c * d = %i", a * b + c * d);
    
    a += b;
    a -= b;
    a /= b;
    a *= b;
    
    a %= b;
    
    a++;
    a--;
    
}

-(void) demoLogic{
    MyLog (@"Demonstated logic grammar, such as for while if");
// 逻辑判断包括：for; while; do while; if else; switch; condition; break; continue;
    
/*
 for ( init_expression; loop_condition; loop_expression )
 program statement
 */
    int n, triangularNumber;
    triangularNumber = 0;
    for ( n = 1; n <= 200; n = n + 1 )
        triangularNumber += n;

/*
 while ( expression ) program statement
 */
    int count = 1;
    while ( count <= 5 ) {
        NSLog (@"%i", count); ++count;
    }
/*
    do
        program statement
        while ( expression );
*/
    int number, right_digit;
    
    NSLog (@"Enter your number.");
    scanf ("%i", &number);
    
    do {
        right_digit = number % 10; NSLog (@"%i", right_digit); number /= 10;
    }
    while ( number != 0 );
/*
 if ( expression ) program statement 1
 else
 program statement 2
 */
    if ( number == 0 )
        NSLog (@"The number is even.");
    else
        NSLog (@"The number is odd.");
/*
 switch ( expression ) {
 case value1:
 program statement program statement
 ... break;
 case value2:
 program statement program statement
 ... break;
 ...
 case valuen:
 program statement
 program statement ...
 break; default:
 program statement program statement
 ... break;
 }
 */
    
 /*
  condition ? expression1 : expression2
  */
    NSLog (@"Sign = %i", ( number < 0 ) ? -1
           : ( number == 0 ) ? 0 : 1);
}


-(void) demoArray{
    MyLog (@"Demonstated array");
    
    int Fibonacci[15], i;
    Fibonacci[0] = 0;
    
    /* by definition */
    Fibonacci[1] = 1;
    
    /* ditto */
    for ( i = 2; i < 15; ++i )
        Fibonacci[i] = Fibonacci[i-2] + Fibonacci[i-1];
    
    for ( i = 0; i < 15; ++i )
        NSLog (@"%i", Fibonacci[i]);

}

-(void) demoEnumerate{
    MyLog (@"Demonstated enumerate");
    enum month { january = 1, february, march, april, may, june,
        july, august, september, october, november,
        december }; enum month amonth;
    
    int days;
    
    NSLog (@"Enter month number: "); scanf ("%i", &amonth);
    
    switch (amonth) {
        case january:
        case march:
        case may:
        case july:
        case august:
        case december:
            days = 31;
            break;
        case april:
        case june:
        case september:
        case november:
            days = 30;
            break;
        case february:
            days = 28;
            break;
        default:
            NSLog (@"bad month number"); days = 0;
            break;
    }
    
    if ( days != 0 )
        NSLog (@"Number of days is %i", days);
    
    if ( amonth == february )
        NSLog (@"...or 29 if it's a leap year");
}


-(void) demoStructures{
    MyLog (@"Demonstated structure");
    // 延续了C 语言的用法
    struct date {
        int month; int day; int year;
    };
    
    struct date today;
    
    today.month = 9; today.day = 25; today.year = 2011;
    NSLog (@"Today's date is %i/%i/%.2i.", today.month, today.day, today.year % 100);
}


-(void) demoBlocks{
    MyLog (@"Demonstated blocks");
   // 注意符号^, 主要用于回调函数，类似于C语言的函数指针
    void (^print_message)(void) =
    ^(void) {
        NSLog (@"Programming is fun.");
    };
    
    print_message();

}

@end
//
//  DemoClass.h
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import <Foundation/Foundation.h>
#import "DemoGlobalDefine.h"
#import "DemoDelegation.h"

@interface DemoClass: NSObject{
    // 成员数据,
    int numerator;
}

// 总结OC的封装，
// 成员数据,由于OOP的封装性，默认的成员数据，不能够直接被访问，只能通过 getter seter方法去访问
// OC 引入@property @synthesize 通过‘.’来访问例如：demoClass.intProperty
@property int intProperty;

// 成员数据的访问范围
/*
- @protected— Methods defined in the class and any subclasses can directly access
the instance variables that follow.This is the default for instance variables defined in
the interface section.
- @private— Methods defined in the class can directly access the instance variables
that follow, but subclasses cannot.This is the default for instance variables defined in
the implementation section.
- @public— Methods defined in the class and any other classes or modules can
directly access the instance variables that follow.
- @package— For 64-bit images, the instance variable can be accessed anywhere
within the image that implements the class.
*/

// 修饰关键字
/*
 -原子性：
 
 - atomic（默认）：atomic意为操作是原子的，意味着只有一个线程访问实例变量。atomic是线程安全的，至少在当前的存取器上是安全的。它是一个默认的特性，但是很少使用，因为比较影响效率，这跟ARM平台和内部锁机制有关。
 - nonatomic：nonatomic跟atomic刚好相反。表示非原子的，可以被多个线程访问。它的效率比atomic快。但不能保证在多线程环境下的安全性，在单线程和明确只有一个线程访问的情况下广泛使用。
 
 -存取器控制：
 
 - readwrite（默认）：readwrite是默认值，表示该属性同时拥有setter和getter。
 - readonly： readonly表示只有getter没有setter。
 
 -内存管理：
 
 - strong：strong是在IOS引入ARC的时候引入的关键字，是retain的一个可选的替代。表示实例变量对传入的对象要有所有权关系，即强引用。strong跟retain的意思相同并产生相同的代码，但是语意上更好更能体现对象的关系。
 
 - weak：在setter方法中，需要对传入的对象不进行引用计数加1的操作。
 - 简单来说，就是对传入的对象没有所有权，当该对象引用计数为0时，即该对象被释放后，用weak声明的实例变量指向nil，即实例变量的值为0。
 
 - 注：weak关键字是IOS5引入的，IOS5之前是不能使用该关键字的。delegate 和 Outlet 一般用weak来声明。
 
 - copy：与strong类似，但区别在于实例变量是对传入对象的副本拥有所有权，而非对象本身。
 
 */

// 成员函数
-(void) demoInstanceMemberFunc: (int) n; // 实例方法, 注意"-"
+(void) demoStaticFunc;                  // 静态方法, 注意"+"

-(void) demoMultipleParameter: (int) p1 parameterSecond: (int) p2;
-(void) demoOverriding;                  // 重写
-(DemoClass *)init;                      // 初始化
-(void) demoDelegation;                  // 委托（Delegation）
@end//
//  DemoClass.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import "DemoClass.h"

@implementation DemoClass

// OC 引入@property @synthesize
@synthesize intProperty;

-(DemoClass *)init{
    MyLog (@"DemoClass's init function.");
    // self 关键字，表示实例自身
    self = [super init];
    if (self) {
        // Initialization code here.
    }
    
    return self;
}

-(void) demoInstanceMemberFunc: (int) n{
    MyLog (@"Demonstrated instance member function and local value.");
    int localVari = 0;         // 本地变量
    static int staticVari = 0; // 只被初始化一次
    
}

-(void) demoMultipleParameter: (int) p1 parameterSecond: (int) p2{
    MyLog (@"Demonstrated mutiple parameter.");

}

+(void) demoStaticFunc{
    MyLog (@"Demonstrated static function.");

}

-(void) demoOverriding{
    MyLog (@"This is demoOverriding in the DemoClass.");
}

-(void) demoDelegation{
    MyLog (@"Demonstrated delegation.");
    
    DemoDelegation *delegate = [DemoDelegation alloc];
    OtherTasks1 *task1 = [OtherTasks1 alloc];
    
    // 将任务授予代理属性
    delegate.delegate = task1;
    // 执行任务
    [delegate executeDelegationTasks];
    
}

@end

//
//  DemoInheritance.h
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import <Foundation/Foundation.h>
#import "DemoGlobalDefine.h"
#import "DemoClass.h"

@interface DemoInheritance: DemoClass // 继承DemoClas 类
// 总结OC的继承

// 重写(Overriding)父类的init, id 为
-(DemoInheritance *)init;
// 带有参数的初始化
-(DemoInheritance *)initWith: (int) n over: (int) d;
// 使用父类的成员数据
-(void)useMemberDataOfParent;
// 重写父类的实例方法
-(void)demoOverriding;

// 多态
-(void)whichMethodIsSelected;
@end//
//  DemoInheritance.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "DemoInheritance.h"

@implementation DemoInheritance

// 重载父类的init
-(DemoInheritance *)init{
    MyLog (@"DemoInheritance's init function.");
    // self 关键字，代表实例对象
    // super 代表父类
    self = [super init];
    if (self) {
        // Initialization code here.
    }
    
    return self;
}

// 带有参数的初始化
-(DemoInheritance *)initWith: (int) n over: (int) d{
    MyLog (@"DemoInheritance's initWith function. Demonstrated multiple parameter");
    self = [super init];
    
    if (self){
    
   }
    
    return self;
}

// 使用父类的成员数据
-(void)useMemberDataOfParent{
    MyLog (@"Demonstrated for using member data of parent ");
    NSLog (@"Numerator is : %d", numerator);
}

// 重写父类的实例方法
-(void)demoOverriding{
    MyLog (@"Demonstrated overriding");
    NSLog (@"This is demoOverriding in the DemoInheritance. ");
}

-(void)whichMethodIsSelected{
    MyLog (@"Which method is selected?");
    
    DemoClass *demoClass = [[DemoClass alloc] init];
    DemoClass *demoInheri = [[DemoInheritance alloc] init];
    
    // 多态，使用了各自的方法
    [demoClass demoOverriding];  // 使用DemoClass中的方法
    [demoInheri demoOverriding]; // 使用DemoInheritance中的方法
}

@end//
//  DemoCategories.h
//  DemoObjectiveC
//  Describle: 分类（Categories）
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import "DemoGlobalDefine.h"
#import "DemoClass.h"

// 采用() 扩展 DemoClass
@interface DemoClass(DemoCategories)

-(void)demoCategories;

@end//
//  DemoCategories.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import "DemoCategories.h"

@implementation DemoClass(DemoCategories)

-(void)demoCategories{
    MyLog (@"Demonstrated Categories");
    
    NSLog (@"This is extendtion by Categories.");
}

@end
//
//  DemoProtocol.h
//  DemoObjectiveC
//  Describle: 定义protocol
//  A protocol is a list of methods that is shared among classes.The methods listed in the proto- col do not have corresponding implementations; they’re meant to be implemented by someone else (like you!).A protocol provides a way to define a set of methods that are somehow related with a specified name.
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import <Foundation/Foundation.h>
#import "DemoGlobalDefine.h"

@protocol DemoProtocol

//使用protocol是，被标记为required的方法，必须被实现
@required
-(void)doRequiredProtocol;

// 可选
@optional
-(void)doOptionalProtocol;
@end
//
//  OtherTasks.h
//  DemoObjectiveC
//  Describle:被委托的任务, 实现DemoProtocol的method
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "DemoProtocol.h"
#import "DemoGlobalDefine.h"

@interface OtherTasks1: NSObject<DemoProtocol>

-(void)doRequiredProtocol;

@end//
//  OtherTask1.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "OtherTasks1.h"

@implementation OtherTasks1

-(void)doRequiredProtocol{
    MyLog (@"Demonstrated protocol and delegation");
    NSLog(@"This is other task 1.");
}

@end//
//  OtherTasks2.h
//  DemoObjectiveC
//  Describle:被委托的任务, 实现DemoProtocol的method
//  Created by Frank on 15/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import "DemoProtocol.h"
#import "DemoGlobalDefine.h"

@interface OtherTasks2: NSObject<DemoProtocol>

-(void)doRequiredProtocol;

@end//
//  OtherTasks2.m
//  DemoObjectiveC
//
//  Created by Frank on 15/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//


#import "OtherTasks2.h"

@implementation OtherTasks2

-(void)doRequiredProtocol{
    MyLog (@"Demonstrated protocol and delegation");
    NSLog(@"This is other task 2.");
}

@end//
//  DemoDelegation.h
//  DemoObjectiveC
//  Describle: 委托，
//  Delegate本身应该称为一种设计模式，是把一个类自己需要做的一部分事情，让另一个类（也可以就是自己本身）来完成，而实际做事的类为被委托类。而protocol是一种语法，它的主要目标是提供接口给遵守协议的类使用，而这种方式提供了一个很方便的、实现delegate模式的机会。
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "OtherTasks1.h"
#import "OtherTasks2.h"
#import "DemoGlobalDefine.h"

@interface DemoDelegation : NSObject 

@property (retain) id<DemoProtocol> delegate;

-(void)executeDelegationTasks;
@end
//
//  DemoDelegation.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//
#import "DemoDelegation.h"

@implementation DemoDelegation

-(void)executeDelegationTasks{
    MyLog (@"Demonstrated protocol and delegation");
    // 注意这里的"_"
    [_delegate doRequiredProtocol];
}

@end//
//  DemoFoundationFramework.h
//  DemoObjectiveC
//  Describle: Foundation Framework
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import <Foundation/Foundation.h>
#import "DemoGlobalDefine.h"

@interface DemoFoundationFramework : NSObject 
-(void) demoNumberObjects;
-(void) demoStringObjects;
-(void) demoArray;
-(void) demoDictionary;
-(void) demoSet;
-(int)  demoFileManager;
 //Foundation’s NSData class provides an easy way to set up a buffer, read the contents of the file into it, or write the contents of a buffer out to a file.
-(int)  demoNSData;
-(void) demoArchivingWithXMLProperty;
-(void) demoArchivingWithNSKeyedArchiver;
// 序列化 反序列化
-(void) demoEncodeAndDecode;

@end

// Create an integer object
#define INTOBJ(v) [NSNumber numberWithInteger: v]

// Add a print method to NSSet with the Printing category
@interface NSSet (Printing)

-(void) print;

@end//
//  DemoFoundationFramework.m
//  DemoObjectiveC
//
//  Created by Frank on 08/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#import "DemoFoundationFramework.h"

@implementation DemoFoundationFramework
-(void) demoNumberObjects{
    MyLog (@"Demonstrated number objects in the foundation framework");
    
    NSNumber *myNumber, *floatNumber, *intNumber; NSInteger myInt;
    
    // integer value
    intNumber = [NSNumber numberWithInteger: 100]; myInt = [intNumber integerValue];
    NSLog (@"%li", (long) myInt);
    
    // long value
    myNumber = [NSNumber numberWithLong: 0xabcdef]; NSLog (@"%lx", [myNumber longValue]);
    
    // char value
    myNumber = [NSNumber numberWithChar: 'X']; NSLog (@"%c", [myNumber charValue]);
    
    // float value
    floatNumber = [NSNumber numberWithFloat: 100.00]; NSLog (@"%g", [floatNumber floatValue]);
    
    // double
    myNumber = [NSNumber numberWithDouble: 12345e+15]; NSLog (@"%lg", [myNumber doubleValue]);
    
    // Wrong access here
    NSLog (@"%li", (long) [myNumber integerValue]); // Test two Numbers for equality
    if ([intNumber isEqualToNumber: floatNumber] == YES) NSLog (@"Numbers are equal");
    else
        NSLog (@"Numbers are not equal");
    
    // Test if one Number is <, ==, or > second Number
    if ([intNumber compare: myNumber] == NSOrderedAscending) NSLog (@"First number is less than second");
}
-(void) demoStringObjects{
    MyLog (@"Demonstrated string objects in the foundation framework");
    
    NSString *str1 = @"This is string A"; NSString *str2 = @"This is string B"; NSString *res;
    NSComparisonResult compareResult;
    
    // Count the number of characters
    NSLog (@"Length of str1: %lu", [str1 length]);
    
    // Copy one string to another
    res = [NSString stringWithString: str1]; NSLog (@"copy: %@", res);
    
    // Copy one string to the end of another
    str2 = [str1 stringByAppendingString: str2]; NSLog (@"Concatentation: %@", str2);
    
    // Test if 2 strings are equal
    if ([str1 isEqualToString: res] == YES) NSLog (@"str1 == res");
    else
        NSLog (@"str1 != res");
    
    // Test if one string is <, == or > than another
    compareResult = [str1 compare: str2];
    if (compareResult == NSOrderedAscending) NSLog (@"str1 < str2");
    else if (compareResult == NSOrderedSame) NSLog (@"str1 == str2");
    else // must be NSOrderedDescending NSLog (@"str1 > str2");
        // Convert a string to uppercase
        res = [str1 uppercaseString];
    NSLog (@"Uppercase conversion: %s", [res UTF8String]);
    
    // Convert a string to lowercase
    res = [str1 lowercaseString];
    
    NSLog (@"Lowercase conversion: %@", res);
    NSLog (@"Original string: %@", str1);
}

-(void) demoArray{
    MyLog (@"Demonstrated array objects in the foundation framework");
    
    int i;
    
    // Create an array to contain the month names
    NSArray *monthNames = [NSArray arrayWithObjects: @"January", @"February", @"March", @"April", @"May", @"June", @"July", @"August", @"September", @"October", @"November", @"December", nil ];
    
    // Now list all the elements in the array
    NSLog (@"Month Name"); NSLog (@"===== ====");
    for (i = 0; i < 12; ++i)
        NSLog (@" %2i %@", i + 1, [monthNames objectAtIndex: i]);
}

-(void) demoDictionary{
    MyLog (@"Demonstrated dictionary objects in the foundation framework");
    
    NSMutableDictionary *glossary = [NSMutableDictionary dictionary];
    
    // Store three entries in the glossary
    [glossary setObject: @"A class defined so other classes can inherit from it" forKey: @"abstract class" ];
    [glossary setObject: @"To implement all the methods defined in a protocol" forKey: @"adopt"];
    [glossary setObject: @"Storing an object for later use" forKey: @"archiving"];
    
    // Retrieve and display them
    NSLog (@"abstract class: %@", [glossary objectForKey: @"abstract class"]); NSLog (@"adopt: %@", [glossary objectForKey: @"adopt"]);
    NSLog (@"archiving: %@", [glossary objectForKey: @"archiving"]);
}

-(void) demoSet{
    MyLog (@"Demonstrated set objects in the foundation framework");
    
    NSMutableSet *set1 = [NSMutableSet setWithObjects: INTOBJ(1), INTOBJ(3), INTOBJ(5), INTOBJ(10), nil];
    NSSet *set2 = [NSSet setWithObjects:
                   INTOBJ(-5), INTOBJ(100), INTOBJ(3), INTOBJ(5), nil];
    
    NSSet *set3 = [NSSet setWithObjects: INTOBJ(12), INTOBJ(200), INTOBJ(3), nil];
    
    NSLog (@"set1: ");
    
    [set1 print];
    NSLog (@"set2: ");
    [set2 print];
    
    // Equality test
    if ([set1 isEqualToSet: set2] == YES)
        NSLog (@"set1 equals set2"); else
            NSLog (@"set1 is not equal to set2");
    
    // Membership test
    if ([set1 containsObject: INTOBJ(10)] == YES) NSLog (@"set1 contains 10");
    else
        NSLog (@"set1 does not contain 10");
    if ([set2 containsObject: INTOBJ(10)] == YES) NSLog (@"set2 contains 10");
    else
        NSLog (@"set2 does not contain 10");
    
    // add and remove objects from mutable set set1
    [set1 addObject: INTOBJ(4)];
    [set1 removeObject: INTOBJ(10)];
    NSLog (@"set1 after adding 4 and removing 10: ");
    [set1 print];
    
    // get intersection of two sets
    [set1 intersectSet: set2];
    NSLog (@"set1 intersect set2: "); [set1 print];
    
    // union of two sets
    [set1 unionSet:set3];
    NSLog (@"set1 union set3: "); [set1 print];
}

-(int) demoFileManager{
    MyLog (@"Demonstrated file manager in the foundation framework");
    
    NSString *fName = @"testfile";
    NSFileManager *fm;
    NSDictionary *attr;
    
    // Need to create an instance of the file manager fm = [NSFileManager defaultManager];
    // Let's make sure our test file exists first
    if ([fm fileExistsAtPath: fName] == NO) { NSLog(@"File doesn't exist!");
        return 1;
    }
    
    //now lets make a copy
    if ([fm copyItemAtPath: fName toPath: @"newfile" error: NULL] == NO) { NSLog(@"File Copy failed!");
        return 2;
    }
    
    // Now let's test to see if the two files are equal
    if ([fm contentsEqualAtPath: fName andPath: @"newfile"] == NO) { NSLog(@"Files are Not Equal!");
        return 3;
    }
    
    // Now lets rename the copy
    if ([fm moveItemAtPath: @"newfile" toPath: @"newfile2" error: NULL] == NO){
        NSLog(@"File rename Failed");
        return 4; }
    
    // get the size of the newfile2
    if ((attr = [fm attributesOfItemAtPath: @"newfile2" error: NULL]) == nil) {
        NSLog(@"Couldn't get file attributes!");
        return 5; }
    NSLog(@"File size is %llu bytes",
          [[attr objectForKey: NSFileSize] unsignedLongLongValue]);
    
    // And finally, let's delete the original file
    if ([fm removeItemAtPath: fName error: NULL] == NO) { NSLog(@"file removal failed");
        return 6;
    }
    NSLog(@"All operations were successful");
    
    // Display the contents of the newly-created file
    NSLog(@"%@", [NSString stringWithContentsOfFile: @"newfile2" encoding:NSUTF8StringEncoding error:NULL]);
    
    return 0;
}

-(int) demoNSData{
    NSFileManager *fm; NSData *fileData;
    
    fm = [NSFileManager defaultManager];
    
    // Read the file newfile2
    fileData = [fm contentsAtPath: @"newfile2"];
    if (fileData == nil) {
        NSLog (@"File read failed!");
        return 1;
    }
    
    // Write the data to newfile3
    if ([fm createFileAtPath: @"newfile3" contents: fileData attributes: nil] == NO) {
        NSLog (@"Couldn't create the copy!");
        return 2;
    }
    
    NSLog (@"File copy was successful!");
    return 0;
}


-(void) demoArchivingWithXMLProperty{
    MyLog (@"Demonstrated xml property in the foundation framework");
    
    NSDictionary *glossary = [NSDictionary dictionaryWithObjectsAndKeys:
     @"A class defined so other classes can inherit from it.",
     @"abstract class",
     @"To implement all the methods defined in a protocol", @"adopt",
     @"Storing an object for later use. ", @"archiving",
     nil
     ];
    
    if ([glossary writeToFile: @"glossary" atomically: YES] == NO) NSLog (@"Save to file failed!");
}

-(void) demoArchivingWithNSKeyedArchiver{
    MyLog (@"Demonstrated key archiver in the foundation framework");
    
    NSDictionary *glossary = [NSDictionary dictionaryWithObjectsAndKeys:
     @"A class defined so other classes can inherit from it",
     @"abstract class",
     @"To implement all the methods defined in a protocol", @"adopt",
     @"Storing an object for later use",
     @"archiving",
     nil
     ];
    
    [NSKeyedArchiver archiveRootObject: glossary toFile: @"glossary.archive"];
}

-(void) demoEncodeAndDecode{
    MyLog (@"Demonstrated encode and decode in the foundation framework");
}

@end


@implementation NSSet (Printing)

-(void) print {
    printf ("{ ");
    for (NSNumber *element in self)
        printf (" %li ", (long) [element integerValue]);
    printf ("} \n");
}

@end
//
//  DemoGlobalDefine.h
//  DemoObjectiveC
//
//  Created by Frank on 22/04/2016.
//  Copyright © 2016 RedRed. All rights reserved.
//

#ifndef DemoGlobalDefine_h
#define DemoGlobalDefine_h

// 定义宏，打印当前方法名称行号等信息，用于调试
#define MyLog(FORMAT, ...) fprintf(stderr,"\nfunction:%s line:%d content:%s\n", __FUNCTION__, __LINE__, [[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);

#endif /* DemoGlobalDefine_h */
 
```


