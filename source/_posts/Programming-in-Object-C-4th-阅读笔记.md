---
title: Programming in Object C 4th-阅读笔记
date: 2016-04-14 09:14:35
tags:
- cpp
---
 
![](/assets/blog-image/objective-c-p-t.jpg)
由 王宇 原创并发布 ：

Programming in Object C 4th
## 1. Introduction

## 2. Programming in Objective-C

Compiling and Running Programs

Table 2.1 Common Filename Extensions

<!--more-->

Extension Meaning

c C language source file

.cc, .cpp C++ language source file

.h Header file

.m Objective-C source file

.mm Objective-C++ source file

.pl Perl source file

.o Object (compiled) file

```cpp
// First program example
#import <Foundation/Foundation.h>
int main (int argc, const char * argv[])
{
  @autoreleasepool {
    NSLog (@"Programming is fun!");
  }
  return 0;
}
Comment : // /**/
```

#import <Foundation/Foundation.h> // import or include the information from the file

main is a special name that indicates precisely where the program is to begin execution.

auto release pool is a mechanism that allows the system to efficiently manage the memory your application uses as it creates new objects. More detail in Chapter 17

@"Programming is fun" this is known as a constant NSString object

You must terminate all program statements in Objective-C with a semicolon(;)

The backslash() NSLog (@"Testing...\n..1\n...2\n....3");


Displaying the Values of Variables
```cpp
int main (int argc, const char *argv[])
{
  @autoreleasepool {
    int value1, value2, sum;
    value1 = 50;
    value2 = 25;
    sum = value1 + value2;
    NSLog (@"The sum of %i and %i is %i", value1, value2, sum);
  }
  return 0;
}
```

## 3. Classes, Objects, and Methods

What is an Object class instancemethod(behavior) member data

Instances and Methods
yourCar = [Car new];      // get a new car
[yourCar drive];          // drive your car
[yourCar setSpeed: 55];   // set the speed to 55 mph


call message


An Objective-C Class for Working with Fractions

Define an actual class in Objective-C

```cpp
#import <Foundation/Foundation.h>
//---- @interface section ----
@interface Fraction: NSObject
-(void) print;
-(void) setNumerator: (int) n;
-(void) setDenominator: (int) d;
@end
//---- @implementation section ----
@implementation Fraction
{
  int numerator;
  int denominator;
}
-(void) print
{
  NSLog (@"%i/%i", numerator, denominator);
}
-(void) setNumerator: (int) n
{
  numerator = n;
}
-(void) setDenominator: (int) d
{
  denominator = d;
}
@end
//---- program section ----
int main (int argc, char * argv[])
{
  @autoreleasepool {
    Fraction *myFraction;
    // Create an instance of a Fraction
    myFraction = [Fraction alloc];
    myFraction = [myFraction init];
    // Set fraction to 1/3
    [myFraction setNumerator: 1];
    [myFraction setDenominator: 3];
    // Display the fraction using the print method
    NSLog (@"The value of myFraction is:");
    [myFraction print];
  }
  return 0;
}
```

@interface section describes the class and its methods

@implementation section describes the data(memeber data) and contains the actual code that implements the methods declared in th interface section

program section contains the program code to carry out the intended purpose of the program


The @interface Section
Choosing Names 小写字母开头等等

Class and Instance Methods

-(void)print
+(void)print

The leading minus sign (-) tells the Objective-C compiler that the method is an instance method.The only other option is a plus sign (+), which indicates a class method. A class method is one that performs some operation on the class itself, such as creating a new instance of the class.


(+)可以直接访问，有点类似于，类的static 成员

Return Values

–(int) currentAge;
Method Arguments




The @implementation Section
```cpp
@implementation NewClassName
{
  memberDeclarations;
}
methodDefinitions;
@end
The program Section

int main (int argc, char * argv[])
{
  @autoreleasepool {
    Fraction *myFraction;

    // Create an instance of a Fraction and initialize it
    myFraction = [Fraction alloc];
    myFraction = [myFraction init];

    // Set fraction to 1/3
    [myFraction setNumerator: 1];
    [myFraction setDenominator: 3];
    // Display the fraction using the print method
    NSLog (@"The value of myFraction is:");
    [myFraction print];
  }
  return 0;
}
```

- Fraction *myFraction; is an object of type Fraction(存储实例化对象的内存地址)

- myFraction = [Fraction alloc] alloc is short for allocate. You want to allocate memory storage space for a new fraction.

- myFraction = [myFraction init] initializes the instance of a class

- the two messages are typically conbined:
myFraction = [[Fraction alloc] init];

Accessing Instance Variables and Data Encapsulation

You’ve seen how the methods that deal with fractions can access the two instance variables numerator and denominator directly by name. In fact, an instance method can always directly access its instance variables.A class method can’t(不可以直接访问类的成员数据，要通过method去实现，比如get set), however, because it’s dealing only with the class itself, not with any instances of the class (think about that for a
    second).


Summary

Now you know how to define your own class, create objects or instances of that class, and send messages to those objects.We return to the Fraction class in later chapters.You’ll learn how to pass multiple arguments to your methods, how to divide your class definitions into separate files, and also how to use key concepts such as inheritance and dynamic binding. However, now it’s time to learn more about data types and writing expressions in Objective-C. First, try the exercises that follow to test your understanding of the important points covered in this chapter.


## 4. Data Types and Expressions

Data Types and Constants

There are four basic data types:

int

float

dbouble

char

Qualifiers: long, long long short, unsigned, and signed

Type id is used to store an object of any type.


Arithmetic Expressions
Operator Precedence + - * /

Integer Arithmetic and the Unary Minus Operator

```cpp
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    int a = 25;
    int b = 2;
    float c = 25.0;
    float d = 2.0;
    NSLog (@"6 + a / 5 * b = %i", 6 + a / 5 * b);
    NSLog (@"a / b * b = %i", a / b * b);
    NSLog (@"c / d * d = %f", c / d * d);
    NSLog (@"-a = %i", -a);
  }
  return 0;
}

```

Program 4.3 Output
6 + a / 5 * b = 16
a / b * b = 24
c / d * d = 25.000000
-a = -25

The Modulus Operator

Program 4.4
// The modulus operator
```cpp
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    int a = 25, b = 5, c = 10, d = 7;
    NSLog (@"a %% b = %i", a % b);
    NSLog (@"a %% c = %i", a % c);
    NSLog (@"a %% d = %i", a % d);
    NSLog (@"a / d * d + a %% d = %i", a / d * d + a % d);
  }
  return 0;
}

```

Program 4.4 Output
a % b = 0
a % c = 5
a % d = 4
a / d * d + a % d = 25
Integer and Floating-Point Conversions

The Type Cast Operator: f2=(float) i2 / 100;


Assignment Operators += -= /+

## 5. Program Looping
The for Statement
for ( init_expression; loop_condition; loop_expression )
program statement

  The While Statement
while ( expression )
  program statement

  The do Statement
  do
  program statement
  while ( expression );

  The break Statement

  The continue Statment


## 6 Making Decisions
  The if Statement
if ( expression )
  program statement 1
  else
  program statement 2

  The switch statement
switch ( expression )
{
  case value1:
    program statement
      program statement
      ...
      break;
  case value2:
    program statement
      program statement
      ...
      break;
    ...
  case valuen:
      program statement
        program statement
        ...
        break;
  default:
      program statement
        program statement
        ...
        break;
}

The conditional operator
condition ? expression1 : expression2


## 7. More on Classes

Separate Interface and Implementation files

Fraction.h // for interface
Fraction.m // for implementation


Synthesized Accessor Methods

The first step is to use the @property directive in your interface section to identify your properties


Objective-C compiler automatically generate or synthesize getter and setter for us by @synthesize directive in the implementation section


Accessing Properties Using the Dot Operator

[myFraction numerator]
myFraction.numerator


Multiple Arguments to Methods

[myFraction setTo: 1 over:3] // setTo is method name 'over' is arguments name.


Again, choosing good method names is important for program readability

Methods Without Argumen Names
-(int) set: (int) n: (int) d;
[aFraction set: 1 :3];

Operations on Fractions
-(int) add: (Fraction *) f;


Local Variables
- (void) reduce
{
  int u= numerator;    // local variables
  int v= denominator;  // local variables
  inttemp;             // local variables
  while(v != 0) {
    temp= u % v;
    u= v;
    v= temp;
  }
  numerator/= u;
  denominator/= u;
}
Method Arguments

-(void) calculate: (double) x
{
  x *= 2;
  ...
}

[myData calculate: ptVal];


Whatever value was contained in the variable ptVal would be copied into the local variable x when the calculate method was executed.

The static Keyword

-(int) showPage
{
  static int pageCount = 0;
  ...

    ++pageCount;
  ...
    return pageCount;
}

The local static variable would be set to 0 only once when the program started and would retain its value through successive invocations of the showPage method.


The self Keyword

You can use the keyword self to refer to the object that is the receiver of the current message


[self reduce]


Allocating and Returning Objects from Methods
-(Fraction *) add: (Fraction *) f
{
  // To add two fractions:
  // a/b + c/d = ((a*d) + (b*c)) / (b * d)
  // result will store the result of the addition
  Fraction *result = [[Fraction alloc]init];
  result.numerator = numerator * f.denominator + denominator *f.numerator;
  result.denominator = denominator * f.denominator;
  [result reduce];
  return result;
}

resultFraction = [aFraction add: bFraction];

## 8. Inheritance

It All Begins at the Root
#import <Foundation/Foundation.h>
// ClassA declaration and definition
@interface ClassA: NSObject
{
  int x;
}
-(void) initVar;
@end
@implementation ClassA
-(void) initVar
{
  x = 100;
}
@end
// Class B declaration and definition
@interface ClassB : ClassA
-(void) printVar;
@end
@implementation ClassB
-(void) printVar
{
  NSLog (@"x = %i", x);
}
  @end
int main (int argc, char * argv[])
{
  @autoreleasepool {
    ClassB *b = [[ClassB alloc] init];
    [b initVar]; // will use inherited method
    [b printVar]; // reveal value of x;
  }
  return 0;
}
Finding the Right Method
First, the class to which the object belongs is checked to see whether a method is explicitly defined in that class with the specific name. If it is, that’s the method that is used. If it’s not defined there, the parent class is checked. If the method is defined there, that’s what is used. If not, the search
continues.


Extension Through Inheritance: Adding New Methods
A Point Class and Object Allocation

The @class Directive
Using the @class directive is more efficient because the compiler doesn’t need to import and therefore process the entire XYPoint.h file (even though it is quite small); it just needs to know that XYPoint is the name of a class. If you needed to reference one of the XYPoint class’ methods (say in the implementation section), the @class directive would not suffice because the compiler would need more information; it would need to know how many arguments the method takes, what their types are, and what the method’s return type is.

Classes Owning Their Objects
```cpp
#import “Rectangle.h”
#import "XYPoint.h"
int main (int argc, char * argv[])
{
  @autoreleasepool{
    Rectangle *myRect = [[Rectanglealloc]init];
    XYPoint *myPoint = [[XYPointalloc]init];

    [myPoint setX: 100andY:200];
    [myRect setWidth: 5andHeight:8];

    myRect.origin=myPoint; // 将对象赋值给另外一个对象的成员

    NSLog (@"Origin at (%i, %i)",myRect.origin.x,myRect.origin.y);

    [myPoint setX: 50andY:50];

    NSLog (@"Origin at (%i, %i)",myRect.origin.x,myRect.origin.y);
  }
  return0;
}
```

Origin at (100, 200
Origin at (50, 50)

-(void) setOrigin: (XYPoint *) pt
{
  origin = pt;
}

Overriding Methods

Which Method is Selected?

#import <Foundation/Foundation.h>
// insert definitions for ClassA and ClassB here

int main (int argc, char * argv[])
{
  @autoreleasepool{
    ClassA *a = [[ClassAalloc]init];
    ClassB *b = [[ClassBalloc]init];
    [a initVar]; // usesClassAmethod
    [a printVar]; // reveal valueofx;
    [b initVar]; // use overridingClassBmethod
    [b printVar]; // reveal valueofx;
  }
  return0;
}

warning: 'ClassA' may not respond to '-printVar' // ClassA 中没有定义printVar, 此时选择ClassB中的printVar


Abstract Classes

## 9. Polymorphism, Dynamic Typing, and Dynamic Binding

Polymorphism Same Name, Different Class

```cpp
// Interface file for Complex class
#import <Foundation/Foundation.h>
@interface Complex: NSObject
@property double real, imaginary;
-(void) print;
-(void) setReal: (double) a andImaginary: (double) b;
-(Complex *) add: (Complex *) f;
@end

// Implementation file for Complex class
#import "Complex.h"
@implementation Complex
@synthesize real, imaginary;
-(void) print
{
  NSLog (@" %g + %gi ", real, imaginary);
}
-(void) setReal: (double) a andImaginary: (double) b
{
  real = a;
  imaginary = b;
}
-(Complex *) add: (Complex *) f
{
  Complex *result = [[Complex alloc] init];
  result.real = real + f.real;
  result.imaginary = imaginary + f.imaginary;
  return result;
}
@end

// Shared Method Names: Polymorphism
#import "Fraction.h"
#import "Complex.h"
int main (int argc, char * argv[])
{
  @autoreleasepool {
    Fraction *f1 = [[Fraction alloc] init];
    Fraction *f2 = [[Fraction alloc] init];
    Fraction *fracResult;

    Complex *c1 = [[Complex alloc] init];
    Complex *c2 = [[Complex alloc] init];
    Complex *compResult;

    [f1 setTo: 1 over: 10];
    [f2 setTo: 2 over: 15];
    [c1 setReal: 18.0 andImaginary: 2.5];
    [c2 setReal: -5.0 andImaginary: 3.2];

    // add and print 2 complex numbers
    [c1 print]; NSLog (@" +"); [c2 print];
    NSLog (@"---------");
    compResult = [c1 add: c2]; // 多态
    [compResult print];
    NSLog (@"\n");

    // add and print 2 fractions
    [f1 print]; NSLog (@" +"); [f2 print];
    NSLog (@"----");
    fracResult = [f1 add: f2]; // 多态
    [fracResult print];
  }
  return 0;
}



Dynamic Binding and the id Type
#import "Fraction.h"
#import "Complex.h"
int main (int argc, char * argv[])
{
  @autoreleasepool {
    id dataValue;
    Fraction *f1 = [[Fraction alloc] init];
    Complex *c1 = [[Complex alloc] init];

    [f1 setTo: 2 over: 5];
    [c1 setReal: 10.0 andImaginary: 2.5];

    // first dataValue gets a fraction
    dataValue = f1;  // id Type
    [dataValue print];

    // now dataValue gets a complex number
    dataValue = c1;
    [dataValue print];
  }
  return 0;
}

```

Compile Time Versus Runtime Checking

id变量中的对象类型在编译时无法确定，所以一些测试推迟到运行时运行


The id Data Type and Static Typing

不可以把所有的对象都声明为id类型：

变量定义为特定类的对象时，使用的是static形式。“static”这个词指的是这个变量总是用于存储特定类的对象，这样，存储在这种形态中的对象的类是预定(predeterminate)的

it makes your progreams more readable

Argument and Return Types with Dynamic Typing


Asking Questions About Classes

One enables you to ask whether an object conforms to a protocol (see Chapter 11,“Categories and Protocols”). Others enable you to
ask about dynamically resolving methods (not covered in this text).


[Square clase] // 返回类的名字


@selector


Exception Handing Using @try
@try {
  statement
    statement
    ...
}
@catch (NSException *exception) {
  statement
    statement
    ...
}

## 10. More on Variables and Data Types

Initialzing Objects
We didn’t write our own init method here; we use the one we inherited from the parent class, which is NSObject.

Fraction *myFract = [[Fraction alloc] init];
A class that contains many methods and instance variables in it commonly has several initialization methods as well. For example, the Foundation framework’s NSArray class contains the following six initialization methods:

  There’s a standard “template” that’s used for overriding init,
```cpp
  - (id)init  // 注意这个返回类型
{
  self = [super init];  // 执行父类的初始化

  if (self) {
    // Initialization code here.
  }
  return self;
}
```

Scope Revisited
Directives for Controlling Instance Variable Scope

@protected— Methods defined in the class and any subclasses can directly access
the instance variables that follow.This is the default for instance variables defined in
the interface section.

@private— Methods defined in the class can directly access the instance variables
that follow, but subclasses cannot.This is the default for instance variables defined in
the implementation section.

@public— Methods defined in the class and any other classes or modules can
directly access the instance variables that follow.

@package— For 64-bit images, the instance variable can be accessed anywhere
within the image that implements the class.

@interface Printer
{
  @private
    int pageCount;
  int tonerLevel;
  @protected
    // other instance variables
}
...
@end
More on Properties, Synthesized Accessors, and Instance Variables


@synthesize window=_window;


it says to synthesize the getter and setter for the property named window and to associate that property with an instance variable called _window (which does not have to be explicitly declared).This helps to distinguish the use of the instance variable from the property and to encourage you to set and retrieve the value of the instance variable through the setter and getter methods.That is, writing something like this


[window makeKeyAndVisible]; // This won't work


will fail, as there is no instance variable named window. Instead, you have to either name the instance variable directly by its name, such as


[_window makeKeyAndVisible];


or, preferably, use the accessor method:
[self.window makeKeyAndVisible]

Global Variables
```cpp
#import "Foo.h"

int gGlobalVar = 5;  // this is global variables

int main (int argc, char *argc[])
{
  @autoreleasepool {
    Foo *myFoo = [[Foo alloc] init];
    NSLog (@"%i ", gGlobalVar);
    [myFoo setgGlobalVar: 100];
    NSLog (@"%i", gGlobalVar);
  }
  return 0;
}

The definition of the global variable gGlobalVar in the previous program makes its value accessible by any method (or function) that uses an appropriate extern declaration. Suppose your Foo method setgGlobalVar: looks like this:

-(void) setgGlobalVar: (int) val
{
  extern int gGlobalVar;  // 引用全局变量
  gGlobalVar = val;

}  

```
Static Variables

static 定义全局的，但不是外部的


static in gGlobalVar = 0;


以上声明在任何方法函数之外，那么在该文件中，所有位于这条语句之后的方法或函数都可以访问gGlobalVar的值， 而其他文件中的方法和函数不行


Enumerate Data Types

enum flag { false, true };

#import <Foundation/Foundation.h>

// print the number of days in a month
int main (int argc, char * argv[])
{
  @autoreleasepool {
    enum month { january = 1, february, march, april, may, june, july, august, september, october, november, december };

    enum month amonth; // 枚举

    int days;

    NSLog (@"Enter month number: ");
    scanf ("%i", &amonth);

    switch (amonth) {
      case january:
      case march:
      case may:
      case july:
      case august:
      case october:
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
        NSLog (@"bad month number");
        days = 0;
        break;
    }
    if ( days != 0 )
      NSLog (@"Number of days is %i", days);
    if ( amonth == february )
      NSLog (@"...or 29 if it's a leap year");
  }
  return 0;
}


The typedef Statement
// 1
typedef int Counter;
Counter j, n;

// 2
typedef NSNumber *NumberObject;
NSNumber *myValue1, *myValue2, *myResult;

// 3
typedef enum { east, west, south, north } Direction;
Direction step1, step2;

Data Type Conversions

Conversion Rules

If either operand is of type long double, the other is converted to long double, and that is the type of the result.

If either operand is of type double, the other is converted to double, and that is the type of the result.

If either operand is of type float, the other is converted to float, and that is the type of the result.

If either operand is of type _Bool, char, short int, or bit field,1 or of an enumerated data type, it is converted to int.

If either operand is of type long long int, the other is converted to long long int, and that is the type of the result.

If either operand is of type long int, the other is converted to long int, and that is the type of the result.

Ifthis step is reached, both operands are oftype int, and that is the type ofthe result.


Bit Operators

Symbol Operation
& Bitwise AND
| Bitwise inclusive-OR
^ Bitwise OR
~ Ones complement
<< Left shift
>> Right shift

The Bitwise AND Operator

The Bitwise Inclusive-OR Operator

The Bitwise Exclusive-OR Operator

The Ones Complement Operator

The Left Shift Operator

The Right Shift Operator


## 11. Categories and Protocols

Categories

A category provides an easy way for you to modularize the definition of a class into groups or categories of related methods.It also gives you an easy way to extend an existing class definition without even having access to the original source code for the class and without having to create a subclass.This is a powerful yet easy concept for you to learn.
```cpp
#import "Fraction.h"  // 必须包含原始接口部分
@interface Fraction (MathOps) // This tells the compiler that you 
  // are defining a new category for the 
  // Fraction class
  // and that its name is MathOps.
  -(Fraction *) add: (Fraction *) f;
  -(Fraction *) mul: (Fraction *) f;
  -(Fraction *) sub: (Fraction *) f;
  -(Fraction *) div: (Fraction *) f;
  @end

@implementation Fraction (MathOps) // 
  -(Fraction *) add: (Fraction *) f
{
  // To add two fractions:
  // a/b + c/d = ((a*d) + (b*c)) / (b * d)
  Fraction *result = [[Fraction alloc] init];
  result.numerator = (self.numerator * f.denominator) +
    (self.denominator * f.numerator);
  result.denominator = self.denominator * f.denominator;
  [result reduce];
  return result;
}
-(Fraction *) sub: (Fraction *) f
{
  // To sub two fractions:
  // a/b - c/d = ((a*d) - (b*c)) / (b * d)
  Fraction *result = [[Fraction alloc] init];
  result.numerator = (self.numerator * f.denominator) -
    (self.denominator * f.numerator);
  result.denominator = self.denominator * f.denominator;
  [result reduce];
  return result;
}
-(Fraction *) mul: (Fraction *) f
{
  Fraction *result = [[Fraction alloc] init];
  result.numerator = self.numerator * f.numerator;
  result.denominator = self.denominator * f.denominator;
  [result reduce];
  return result;
}
-(Fraction *) div: (Fraction *) f
{
  Fraction *result = [[Fraction alloc] init];
  result.numerator = self.numerator * f.denominator
    result.denominator = self.denominator * f.numerator;
  [result reduce];
  return result;
}
  @end
int main (int argc, char * argv[])
{
  @autoreleasepool {
    Fraction *a = [[Fraction alloc] init];
    Fraction *b = [[Fraction alloc] init];

    Fraction *result;

    [a setTo: 1 over: 3];
    [b setTo: 2 over: 5];
    [a print]; NSLog (@" +"); [b print]; NSLog (@"-----");
    result = [a add: b];
    [result print];
    NSLog (@"\n");

    [a print]; NSLog (@" -"); [b print]; NSLog (@"-----");
    result = [a sub: b];
    [result print];
    NSLog (@"\n");

    [a print]; NSLog (@" *"); [b print]; NSLog (@"-----");
    result = [a mul: b];
    [result print];
    NSLog (@"\n");

    [a print]; NSLog (@" /"); [b print]; NSLog (@"-----");
    result = [a div: b];
    [result print];
    NSLog (@"\n");
  }
  return 0;
}
```

Note
By convention, the base name of the .h and .m files for a category is the class name followed by the category name. In our example, we would put the interface section for the category in a file named FractionMathOps.h and the implementation section in a file called
FractionMathOps.m. Some programmers use a ‘+’ sign to separate the name of the category from the class, as in Fraction+MathOps.h.


Class Extensions

There is a special case of creating a category without a name, that is no name is specified
between the ( and ) .This special syntax defines what is known as a class extension.
```cpp
#import "GraphicObject.h"
// Class extension
@interface GraphicObject () // 注意这里的(), 为class extension 或理解为 Categories without name
  @property int uniqueID;
  -(void) doStuffWithUniqueID: (int) theID;
  @end

  //--------------------------------------
  @implementation GraphicObject
  @synthesize uniqueID;

  -(void) doStuffWithUniqueID: (int) myID
{
  self.uniqueID = myID;
  ...
}
```
...
// Other GraphicObject methods
...
@end
Some Notes About Categories

Protocols and Delegation

Protocols
A protocol is a list of methods that is shared among classes.Defining a protocol is easy:You simply use the @protocol directive followed by the name of the protocol, which is up to you.

@protocol NSCopying
- (id)copyWithZone: (NSZone *)zone;
@end
adopt
If you adopt the NSCopying protocol in your class, you must implement a method called copyWithZone: .You tell the compiler that you are adopting a protocol by listing the protocol name inside a pair of angular brackets (<...>) on the @interface line.The protocol name comes after the name of the class and its parent class, as in the following

@interface AddressBook: NSObject <NSCopying>

This says that AddressBook is an object whose parent is NSObject and states that it conforms to the NSCopying protocol. 
check protocols

if ([currentObject conformsToProtocol: @protocol (Drawing)] == YES)
{
  // Send currentObject paint, erase and/or outline msgs
  ...
}

id <Drawing> currentObject;

extend the protocols

@protocol Drawing3D <Drawing>

@optional
@required

Delegation


Delegate本身应该称为一种设计模式，是把一个类自己需要做的一部分事情，让另一个类（也可以就是自己本身）来完成，而实际做事的类为delegate。而protocol是一种语法，它的主要目标是提供接口给遵守协议的类使用，而这种方式提供了一个很方便的、实现delegate模式的机会。


参考：http://www.cnblogs.com/qingyuan/p/3616870.html
1,先定义一个协议(Protocol) ProtocolName, 协议ProtocolName中定义了两个方法: first ,second
2,现在这里定义两个类StudentA和StudentB来实现协议ProtocolName中的方法.
3,然后新建一个Object C类Student,但是这个类并没有去遵循协议ProtocolName 去实现其他的方法，而是通过委托来代理实现协议方法的调用。

#import <Foundation/Foundation.h>
#import "ProtocolName.h"

@interface Student : NSObject{
  id<ProtocolName> delegate; // 委托
}

@property (retain) id<ProtocolName> delegate; // 将委托设置为一个属性

-(void) setMethod;

@end

// 使用委托
Student *stu=[[Student alloc] init];
StudentA *stua=[[StudentA alloc] init];
StudentB *stub=[[StudentB alloc] init];
stu.delegate=stua;
[stu.delegate first];
[stu setMethod];

stu.delegate=stub;
[stu setMethod];

Informal Protocols

Composite Objects

@interface Square: NSObject
{
  Rectangle *rect; // 合成对象，而不是继承对象
}
-(int) setSide: (int) s;
-(int) side;
-(int) area;
-(int) perimeter;
@end

## 12. The Preprocessor

The #define Statement

The #import Statement

Conditional Compilation
The #ifdef, #endif, #else, and #ifndef Statements

The #if and #elif Preprocessor Statements

The #undef Statement


## 13. Underlying C Language Feartures

Arrays
Initializing Array Elements

int integers[5] = { 0, 1, 2, 3, 4 } ;
char letters[5] = { 'a', 'b', 'c', 'd', 'e' } ;

Character Arrays

char word[] = { 'H', 'e', 'l', 'l', 'o', '!' } ;
Multidimensional Arrays

int M[4][7] = {
  { 10, 5, -3, 17, 82 } ,
  { 9, 0, 0, 8, -7 } ,
  { 32, 20, 1, 0, 14 } ,
  { 0, 0, 8, 7, 6 }
} ;

Functions
Arguments and Local Variables

#import <Foundation/Foundation.h>

// Function to calculate the nth triangular number
void calculateTriangularNumber (int n)
{
  int i, triangularNumber = 0;
  for ( i = 1; i <= n; ++i )
    triangularNumber += i;
  NSLog (@"Triangular number %i is %i", n, triangularNumber);
}

int main (int argc, char * argv[])
{
  @autoreleasepool {
    calculateTriangularNumber (10);
    calculateTriangularNumber (20);
    calculateTriangularNumber (50);
  }
  return 0;
}
Returning Function Results

int gcd (int u, int v)
{
  return u * v;
}
result = gcd (150, 35);
Functions, Methods, and Arrays


Blocks

块从本质上来说是一个闭包，即其拥有代码逻辑和运行该段代码逻辑需要的变量。这一切在定义代码时就已经确定，因此，一个块在定义时访问了某个上文变量，即使之后该上下文变量发生了变化，块中仍然使用是定义时的值，可以认为块只是在定义的时候拷贝了一个变量值到自己的作用域。定义完之后，和原来的那个上下文变量就没有关系了。

void printMessage (void)
{
  NSLog (@"Programming is fun.");
}

^(void)
{
  NSLog (@"Programming is fun.");
}

void (^printMessage)(void) =
^(void){
  NSLog (@"Programming is fun.");
} ;

Structures
struct date
{
  int month;
  int day;
  int year;
} ;


struct date today;

Initilizing Structurs

struct date today = { 7, 2, 2011 } ;

struct date today = { .month = 7, .day = 2, .year = 2011 } ;

Structures Within Structures

/* Points. */
struct CGPoint {
  CGFloat x;
  CGFloat y;
} ;

typedef struct CGPoint CGPoint;

/* Sizes. */
struct CGSize {
  CGFloat width;
  CGFloat height;
} ;

typedef struct CGSize CGSize;

additional Details About Stgructures

struct date
{
  int month;
  int day;
  int year;
} todaysDate, purchaseDate;

struct date
{
  int month;
  int day;
  int year;
} todaysDate = { 9, 25, 2011 } ;

struct
{
  int month;
  int day;
  int year;
} dates[100];

Pointers

To understand the way pointers operate, you first must understand the concept of indirection


the address operator (&) and the indirection operator (*).

Pointers and Structures

struct date
{
  int month;
  int day;
  int year;
} ;

struct date todaysDate;
struct date *datePtr;

datePtr = &todaysDate;
Pinters, Methods, and Functions
You can pass a pointer as an argument to a method or function in the normal fashion, and you can have a function or method return a pointer as its result.

Pointers and Arrays

#import <Foundation/Foundation.h>
int arraySum (int array[], int n)
{
  int sum = 0, *ptr;
  int *arrayEnd = array + n;
  for ( ptr = array; ptr < arrayEnd; ++ptr )
    sum += *ptr;
  return (sum);
}
int main (int argc, char * argv[])
{
  @autoreleasepool {
    int arraySum (int array[], int n);
    int values[10] = { 3, 7, -9, 3, 6, -1, 7, 9, 1, -5 } ;
    NSLog (@"The sum is %i", arraySum (values, 10));
  }
  return 0;
}
Is It an Array, or Is It a Pointer?

Pointers to Character Strings

Constant Character Strings and Pointers
Constant string object should be used '@'

The Increment and Decrement Operators Revisited

for ( ; *from != '\ 0'; )  // 指针定义以后，使用时为什么还要用*号
*to++ = *from++;
Operations on Pointers

Pointers to Functions

int (*fnPtr) (void);
Pointers and Memory Addresses


They're Not Objects!

Miscellaneous Language Features
Compund Literals

The goto Statement

The null Statement

The Comma Operator

The sizeof Operator

Command-Line Arguments


How Things Work
Fact #1: Instance Variables Are Stored in Structures

Fact #2: An Object Variable Is Really a Pointer

Fact #3: Methods Are Functions, and Message Expressions Are Function Calls

Fact #4: The id Type Is a Generic Pointer Type


## 14 Introduction to the Foundation Framework
Foundation Documentation


Quick reference for NSString


15. Numbers, Strings, and Collections
#import <Foundation/Foundation.h>

Number Objects
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSNumber *myNumber, *floatNumber, *intNumber;
    NSInteger myInt;

    // integer value
    intNumber = [NSNumber numberWithInteger: 100];
    myInt = [intNumber integerValue];
    NSLog (@"%li", (long) myInt);

    // long value
    myNumber = [NSNumber numberWithLong: 0xabcdef];
    NSLog (@"%lx", [myNumber longValue]);

    // char value
    myNumber = [NSNumber numberWithChar: 'X'];
    NSLog (@"%c", [myNumber charValue]);

    // float value
    floatNumber = [NSNumber numberWithFloat: 100.00];
    NSLog (@"%g", [floatNumber floatValue]);

    // double
    myNumber = [NSNumber numberWithDouble: 12345e+15];
    NSLog (@"%lg", [myNumber doubleValue]);

    // Wrong access here
    NSLog (@"%li", (long) [myNumber integerValue]);

    // Test two Numbers for equality
    if ([intNumber isEqualToNumber: floatNumber] == YES)
      NSLog (@"Numbers are equal");
    else
      NSLog (@"Numbers are not equal");

    // Test if one Number is <, ==, or > second Number
    if ([intNumber compare: myNumber] == NSOrderedAscending)
      NSLog (@"First number is less than second");
  }
  return 0;
}

Note: Table 15.1 NSNumber Creation and Retrieval Methods


String Objects
More on the NSLog Function

int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSString *str = @"Programming is fun";
    NSLog (@"%@", str);
  }
  return 0;
}
The description Method
In fact, they can be used to display objects from your own classes as well, as long as you override the description method inherited by your class.

-(NSString *) description
{
  return [NSString stringWithFormat: @"%i/%i", numerator, denominator];
}
Mutable Versus Immutable Objects
you might want to delete some characters from a string or perform a search-and-replace operation on a string. These types of strings are handled through the NSMutableString class.

Mutable Strings

```cpp
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSString *str1 = @"This is string A";
    NSString *search, *replace;
    NSMutableString *mstr;
    NSRange substr;

    // Create mutable string from nonmutable
    mstr = [NSMutableString stringWithString: str1];
    NSLog (@"%@", mstr);

    // Insert characters
    [mstr insertString: @" mutable" atIndex: 7];
    NSLog (@"%@", mstr);

    // Effective concatentation if insert at end
    [mstr insertString: @" and string B" atIndex: [mstr length]];
    NSLog (@"%@", mstr);

    // Or can use appendString directly
    [mstr appendString: @" and string C"];
    NSLog (@"%@", mstr);

    // Delete substring based on range
    [mstr deleteCharactersInRange: NSMakeRange (16, 13)];
    NSLog (@"%@", mstr);

    // Find range first and then use it for deletion
    substr = [mstr rangeOfString: @"string B and “];
    if (substr.location != NSNotFound) {
      [mstr deleteCharactersInRange: substr];
      NSLog (@"%@", mstr);
    }

    // Set the mutable string directly
    [mstr setString: @"This is string A"];
    NSLog (@"%@", mstr);

    // Now let’s replace a range of chars with another
    [mstr replaceCharactersInRange: NSMakeRange(8, 8)
      withString: @"a mutable string"];
    NSLog (@"%@", mstr);

    // Search and replace
    search = @"This is";
    replace = @"An example of";
    substr = [mstr rangeOfString: search];
    if (substr.location != NSNotFound) {
      [mstr replaceCharactersInRange: substr
        withString: replace];
      NSLog (@"%@", mstr);
    }

    // Search and replace all occurrences
    search = @"a";
    replace = @"X";
    substr = [mstr rangeOfString: search];
    while (substr.location != NSNotFound) {
      [mstr replaceCharactersInRange: substr
        withString: replace];
      substr = [mstr rangeOfString: search];
    }
    NSLog (@"%@", mstr);
  }
  return 0;
}
```

Array Objects

Immutable arrays are handled by the NSArray class, whereas mutable ones are handled by NSMutableArray.
```cpp
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  int i;
  @autoreleasepool {
    // Create an array to contain the month names
    NSArray *monthNames = [NSArray arrayWithObjects:
      @"January", @"February", @"March", @"April",
      @"May", @"June", @"July", @"August", @"September",
      @"October", @"November", @"December", nil ];
    // Now list all the elements in the array
    NSLog (@"Month Name");
    NSLog (@"===== ====");
    for (i = 0; i < 12; ++i)
      NSLog (@" %2i %@", i + 1, [monthNames objectAtIndex: i]);
  }
  return 0;
}

```
Making an Address Book

Sorting Arrays


Dictionary Objects

A dictionary is a collection of data consisting of key-object pairs
```cpp
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSMutableDictionary *glossary = [NSMutableDictionary dictionary];

    // Store three entries in the glossary
    [glossary setObject: @"A class defined so other classes can inherit from it"
      forKey: @"abstract class" ];
    [glossary setObject: @"To implement all the methods defined in a protocol"
      forKey: @"adopt"];
    [glossary setObject: @"Storing an object for later use"
      forKey: @"archiving"];

    // Retrieve and display them
    NSLog (@"abstract class: %@", [glossary objectForKey: @"abstract class"]);
    NSLog (@"adopt: %@", [glossary objectForKey: @"adopt"]);
    NSLog (@"archiving: %@", [glossary objectForKey: @"archiving"]);
  }
  return 0;
}
```
Enumerating a Dictionary

#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSDictionary *glossary =
      [NSDictionary dictionaryWithObjectsAndKeys:
      @"A class defined so other classes can inherit from it",
      @"abstract class",
      @"To implement all the methods defined in a protocol",
      @"adopt",
      @"Storing an object for later use",
      @"archiving",
      nil
      ];

    // Print all key-value pairs from the dictionary
    for ( NSString *key in glossary )
      NSLog (@"%@: %@", key, [glossary objectForKey: key]);
  }
  return 0;
}

Set Objects

A set is a collection of unique objects, and it can be mutable or immutable.
```cpp
#import <Foundation/Foundation.h>

// Create an integer object
#define INTOBJ(v) [NSNumber numberWithInteger: v]
// Add a print method to NSSet with the Printing category
@interface NSSet (Printing)
  -(void) print;
  @end

@implementation NSSet (Printing)
  -(void) print {
    printf ("{ ");
    for (NSNumber *element in self)
      printf (" %li ", (long) [element integerValue]);
    printf ("} \n");
  }
@end

int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSMutableSet *set1 = [NSMutableSet setWithObjects:
      INTOBJ(1), INTOBJ(3), INTOBJ(5), INTOBJ(10), nil];
    NSSet *set2 = [NSSet setWithObjects:
      INTOBJ(-5), INTOBJ(100), INTOBJ(3), INTOBJ(5), nil];
    NSSet *set3 = [NSSet setWithObjects:
      INTOBJ(12), INTOBJ(200), INTOBJ(3), nil];
    NSLog (@"set1: ");
    [set1 print];
    NSLog (@"set2: ");
    [set2 print];
    // Equality test
    if ([set1 isEqualToSet: set2] == YES)
      NSLog (@"set1 equals set2");
    else
      NSLog (@"set1 is not equal to set2");
    // Membership test
    if ([set1 containsObject: INTOBJ(10)] == YES)
      NSLog (@"set1 contains 10");
    else
      NSLog (@"set1 does not contain 10");
    if ([set2 containsObject: INTOBJ(10)] == YES)
      NSLog (@"set2 contains 10");
    else
      NSLog (@"set2 does not contain 10");
    // add and remove objects from mutable set set1
    [set1 addObject: INTOBJ(4)];
    [set1 removeObject: INTOBJ(10)];
    NSLog (@"set1 after adding 4 and removing 10: ");
    [set1 print];
    // get intersection of two sets
    [set1 intersectSet: set2];
    NSLog (@"set1 intersect set2: ");
    [set1 print];
    // union of two sets
    [set1 unionSet:set3];
    NSLog (@"set1 union set3: ");
    [set1 print];
  }
  return 0;
}
```

## 16. Working with Files

Managing Files and Directories: NSFileManager

A file or directory is uniquely identified to NSFileManager using a pathname to the file.A pathname is an NSString object that can either be a relative or full pathname.A relative pathname is one that is relative to the current directory.
The special tilde character (~) is used as an abbreviation for a user’s home directory.
```cpp
#import <Foundation/Foundation.h>
int main (int argc, const char * argv[]) {
  @autoreleasepool {
    NSString *fName = @"testfile";
    NSFileManager *fm;
    NSDictionary *attr;

    // Need to create an instance of the file manager
    fm = [NSFileManager defaultManager];

    // Let's make sure our test file exists first
    if ([fm fileExistsAtPath: fName] == NO) {
      NSLog(@"File doesn't exist!");
      return 1;
    }

    //now lets make a copy

    if ([fm copyItemAtPath: fName toPath: @"newfile" error: NULL] == NO) {
      NSLog(@"File Copy failed!");
      return 2;
    }
    // Now let's test to see if the two files are equal
    if ([fm contentsEqualAtPath: fName andPath: @"newfile"] == NO) {
      NSLog(@"Files are Not Equal!");
      return 3;
    }
    // Now lets rename the copy
    if ([fm moveItemAtPath: @"newfile" toPath: @"newfile2"
        error: NULL] == NO){
      NSLog(@"File rename Failed");
      return 4;
    }
    // get the size of the newfile2
    if ((attr = [fm attributesOfItemAtPath: @"newfile2" error: NULL])
        == nil) {
      NSLog(@"Couldn't get file attributes!");
      return 5;
    }
    NSLog(@"File size is %llu bytes",
        [[attr objectForKey: NSFileSize] unsignedLongLongValue]);

    // And finally, let's delete the original file
    if ([fm removeItemAtPath: fName error: NULL] == NO) {
      NSLog(@"file removal failed");
      return 6;
    }
    NSLog(@"All operations were successful");

    // Display the contents of the newly-created file
    NSLog(@"%@", [NSString stringWithContentsOfFile:
        @"newfile2" encoding:NSUTF8StringEncoding error:NULL]);
  }
  return 0;
}
Working with the NSData Class

#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSFileManager *fm;
    NSData *fileData;
    fm = [NSFileManager defaultManager];

    // Read the file newfile2
    fileData = [fm contentsAtPath: @"newfile2"];
    if (fileData == nil) {
      NSLog (@"File read failed!");
      return 1;
    }

    // Write the data to newfile3
    if ([fm createFileAtPath: @"newfile3" contents: fileData
        attributes: nil] == NO) {
      NSLog (@"Couldn't create the copy!");
      return 2;
    }
    NSLog (@"File copy was successful!");
  }
  return 0;
}
Working with Directories

#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSString *dirName = @"testdir";
    NSString *path;
    NSFileManager *fm;

    // Need to create an instance of the file manager
    fm = [NSFileManager defaultManager];

    // Get current directory
    path = [fm currentDirectoryPath];
    NSLog (@"Current directory path is %@", path);

    // Create a new directory
    if ([fm createDirectoryAtPath: dirName withIntermediateDirectories: YES
        attributes: nil error: NULL] == NO) {
      NSLog (@"Couldn't create directory!");
      return 1;
    }

    // Rename the new directory
    if ([fm moveItemAtPath: dirName toPath: @"newdir" error: NULL] == NO) {
      NSLog (@"Directory rename failed!");
      return 2;
    }
    // Change directory into the new directory

    if ([fm changeCurrentDirectoryPath: @"newdir"] == NO) {
      NSLog (@"Change directory failed!");
      return 3;
    }

    // Now get and display current working directory
    path = [fm currentDirectoryPath];
    NSLog (@"Current directory path is %@", path);
    NSLog (@"All operations were successful!");
  }
  return 0;
}

Enumeratiing the Contents of a Directory

#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSString *path;
    NSFileManager *fm;
    NSDirectoryEnumerator *dirEnum;
    NSArray *dirArray;

    // Need to create an instance of the file manager
    fm = [NSFileManager defaultManager];
    // Get current working directory path
    path = [fm currentDirectoryPath];

    // Enumerate the directory
    dirEnum = [fm enumeratorAtPath: path];
    NSLog (@"Contents of %@", path);
    while ((path = [dirEnum nextObject]) != nil)
      NSLog (@"%@", path);

    // Another way to enumerate a directory
    dirArray = [fm contentsOfDirectoryAtPath:
      [fm currentDirectoryPath] error: NULL];
    NSLog (@"Contents using contentsOfDirectoryAtPath:error:");
    for ( path in dirArray )
      NSLog (@"%@", path);
  }
  return 0;
}
```

Working with Paths: NSPathUtilities.h

Basic File Operations: NSFileHandle

The NSURL Class

The NSBundle Class

## 17. Memory Management and Automatic Reference Counting

Automatic Garbage Collection

The iOS runtime environment doesn’t support garbage collection, so you don’t have the option to use it when developing programs for that platform.That is, you can only use it when developing Mac OS X applications.


Manual Reference Counting

Automatic Reference Counting (ARC)


  The general concept is as follows:When an object is created, its initial reference count is set to 1. Each time you need to ensure that the object be kept around, you effectively create a reference to it by incrementing its reference count by 1.This is done by sending the object a retain message, like so:


  [myFraction retain];
  When you no longer need an object, you decrement its reference count by 1 by sending it a release message, like this:


  [myFraction release];
  When the reference count of an object reaches 0, the system knows that the object is no longer being used (because, in theory, it is no longer being referenced anywhere in the application), so it frees up (deallocates) its memory.
  Object Reference and the Autorelease Pool
  You might need to write a method that first creates an object (say with alloc) and then returns that object as the result of the method call. Here’s the dilemma: Even though the method is done using the object, it can’t release it, as it needs to return its value.The NSAutoreleasePool class was created in order to help solve problems like these by keeping track of objects to be released at a later time in an object known as an autorelease pool.That later time is when the pool gets drained, which is done by sending the autorelease pool object a drain message. In order to add an object to the list of objects maintained by the autorelease pool, you send that object an autorelease message, like so:


  [result autorelease];
  When dealing with programs that use classes from the Foundation, UIKit, or AppKit frameworks, you must create an autorelease pool because classes from these frameworks can create and return autoreleased objects.You do that in your application with a statement like this:
  NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];
  Xcode generates a template file with this statement at the start of main if you create a new project without ARC enabled. As noted, after the pool is set up, framework methods automatically add new objects such as arrays, strings, dictionaries, views, buttons, and other objects to the list maintained by the autorelease pool.
  When you’re done using the pool, you send it a drain message:


  [pool drain];
  The Event Loop and Memeory Allocation

  当一次Event Loop 发生时，系统会偏离 pool 中 auto release 数组，并且释放资源


  Summary of Manual Memory Management Rules
  If you need to hold onto an object to make sure it doesn’t get destroyed by someone else, you should retain it. Make sure to release the object when you’re done with it.
  Sending a release message does not necessarily destroy an object.When an object’s reference count is decremented to 0, the object is destroyed.The system does this by sending the dealloc message to the object to free its memory.
  Release any objects that you have retained or have created using a copy, mutableCopy, alloc, or new method.This includes properties that have the retain or copy attribute.You can override dealloc to release your instance variables at the time your object is to be destroyed.
  The autorelease pool provides for the automatic release ofobjects when the pool itselfis drained.The system does this by sending a release message to each object in the pool for each time it was autoreleased. Each object in the autorelease pool whose reference count goes down to 0 is sent a dealloc message to destroy the object.
  If you no longer need an object from within a method but need to return it, send it an autorelease message to mark it for later release.The autorelease message does not affect the reference count of the object.
  When your application terminates, all the memory your objects take up is released, regardless of whether they were in the autorelease pool.
  When you develop Cocoa or iOS applications, autorelease pools will be created and drained throughout execution of the program (this will happen each time an event occurs). In such cases, if you want to ensure that an autoreleased object survives automatic deallocation when the autorelease pool is drained, you need to retain it.All objects that have a reference count greater than the number of autorelease messages they have been sent will survive the release of the pool.


Automactic Reference Counting(ARC)

  Stong Variables

  By default, all object pointer variables are strong variables.That means that assigning an object reference to such a variable causes that object to be automatically retained


  Week Variables

  To declare a weak variable you use the _ _weak keyword:


  _ _weak UIView *parentView;


  or you use the weak attribute for a property:


  @property (weak, nonatomic) UIView *parentView;


  @autoreleasepool Blocks
  @autoreleasepool {}
  替换
  NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];

  [pool drain];

  Method Names and Non-ARC Complied Code

## 18. Copying Objects

  The copy and mutableCopy Methods

  The Foundation classes implement methods known as copy and mutableCopy that you can use to create a copy of an object.This is done by implementing a method in conformance with the protocol for making copies. If your class needs to distinguish between making mutable and immutable copies of an object, you must implement a method according to the protocol as well.You learn how to do that later in this section.


  dataArray2 = [dataArray mutableCopy];


  这个例子中，数组中放的是字符串


  Shallow Versus Deep Copying
  // Shallow copy, 仅仅复制了数组的地址，没有复制数组中的元素
  dataArray2 = [dataArray mutableCopy];

  这个例子中，数组中放的是指针


  浅复制，仅仅复制地址, 例如　obj1 = obj2;
  深复制，复制对象, 例如 obj2 = [obj1 mutableCopy];


  Implementing the Protocl
  *** -[AddressBook copyWithZone:]: selector not recognized
  *** Uncaught exception:
  *** -[AddressBook copyWithZone:]: selector not recognized



  @interface Fraction: NSObject <NSCopying>

  Fraction is a subclass of NSObject and conforms to the NSCopying protocol. In the implementation file Fraction.m, add the following definition for your new method:

```cpp
  -(id) copyWithZone: (NSZone *) zone
{
  Fraction *newFract = [[Fraction allocWithZone: zone] init];
  [newFract setTo: numerator over: denominator];

  return newFract;
}


Copying Objects in Setter and Getter Methods
-(void) setName: (NSString *) theName
{
  name = [theName copy];
}
```

## 19. Archiving

Archiving with XML Property Lists

Mac OS X applications use XML propertylists (or plists) for storing things such as your default preferences, application settings, and configuration information, so it’s useful to know how to create them and read them back in.


If your objects are of type NSString, NSDictionary, NSArray, NSDate, NSData, or NSNumber, you can use the writeToFile:atomically: method implemented in these classes to write your data to a file
```cpp
#import <Foundation/Foundation.h>

int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSDictionary *glossary=
      [NSDictionarydictionaryWithObjectsAndKeys:
      @"A class defined so other classes can inheritfromit.",
      @"abstractclass",
      @"To implement all the methods defined inaprotocol",
      @"adopt",
      @"Storing an object for lateruse.",
      @"archiving",
      nil
      ];

    if ([glossary writeToFile: @"glossary" atomically: YES] ==NO)
      NSLog (@"Save to filefailed!");
  }
  return 0;
}

Archiving with NSKeyedArchiver
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSDictionary *glossary =
      [NSDictionary dictionaryWithObjectsAndKeys:
      @"A class defined so other classes can inherit from it",
      @"abstract class",
      @"To implement all the methods defined in a protocol",
      @"adopt",
      @"Storing an object for later use",
      @"archiving",
      nil
      ];

    [NSKeyedArchiver archiveRootObject: glossary toFile: @"glossary.archive"];
  }
  return 0;
}

#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{

  @autoreleasepool {
    NSDictionary *glossary;

    glossary = [NSKeyedUnarchiver unarchiveObjectWithFile: @"glossary.archive"];

    for ( NSString *key in glossary )
      NSLog (@"%@: %@", key, [glossary objectForKey: key]);
  }       
  return 0;
}
```

Writing Encoding and Decoding Methods

Using NSData to Create Custom Archives
```cpp
#import “AddressBook.h”
#import "Foo.h"
int main (int argc, char * argv[])
{
  @autoreleasepool {
    Foo *myFoo1 = [[Foo alloc] init];

    NSMutableData *dataArea;
    NSKeyedArchiver *archiver;
    AddressBook *myBook;

    // Insert code from Program 19.7 to create an Address Book
    // in myBook containing four address cards
    [myFoo1 setStrVal: @"This is the string"];
    [myFoo1 setIntVal: 12345];
    [myFoo1 setFloatVal: 98.6];

    // Set up a data area and connect it to an NSKeyedArchiver object
    dataArea = [NSMutableData data];
    archiver = [[NSKeyedArchiver alloc]
      initForWritingWithMutableData: dataArea];

    // Now we can begin to archive objects
    [archiver encodeObject: myBook forKey: @"myaddrbook"];
    [archiver encodeObject: myFoo1 forKey: @"myfoo1"];
    [archiver finishEncoding];

    // Write the archived data area to a file
    if ([dataArea writeToFile: @"myArchive" atomically: YES] == NO)
      NSLog (@"Archiving failed!");
  }
  return 0;
}
```

Using the Archiver to Copy Objects

You can use the Foundation’s archiving capabilities to create a deep copy of an object.
```cpp
#import <Foundation/Foundation.h>
int main (int argc, char * argv[])
{
  @autoreleasepool {
    NSData *data;

    NSMutableArray *dataArray = [NSMutableArray arrayWithObjects:
      [NSMutableString stringWithString: @"one"],
      [NSMutableString stringWithString: @"two"],
      [NSMutableString stringWithString: @"three"],
      nil
    ];

      NSMutableArray *dataArray2;
      NSMutableString *mStr;

      // Make a deep copy using the archiver
      data = [NSKeyedArchiver archivedDataWithRootObject: dataArray];

      dataArray2 = [NSKeyedUnarchiver unarchiveObjectWithData: data];

      mStr = [dataArray2 objectAtIndex: 0];
      [mStr appendString: @"ONE"];
      NSLog (@"dataArray: ");

      for ( NSString *elem in dataArray )
        NSLog (@"%@", elem);
      NSLog (@"\ndataArray2: ");

      for ( NSString *elem in dataArray2 )
        NSLog (@"%@", elem);
  }
  return 0;
}
```

## 20 Introduction to CoCoa and Cocoa Touch
Cocoa Touch
Whereas the Cocoa frameworks are designed for application development for Mac OS X desktop and notebook computers, the Cocoa Touch frameworks are for applications targeted for iOS devices.
Both Cocoa and Cocoa Touch have the Foundation and Core Data frameworks in common. However, the UIKit replaces the AppKit framework under Cocoa Touch, providing support for many of the same types of objects, such as windows, views, buttons, text fields, and so on. In addition, Cocoa Touch provides classes for working with the accelerometer, gyroscope, triangulating your location with GPS and Wi-Fi signals, and the touch-driven interface, and also eliminates classes that aren’t needed.


## 21. Writing iOS Application
This iOS SDK

Your First IPhone Application

An iPhone Fraction Calculator


