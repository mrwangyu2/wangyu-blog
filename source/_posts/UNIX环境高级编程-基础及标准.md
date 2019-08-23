---
title: UNIX环境高级编程-基础及标准
date: 2012-11-07 09:23:13
tags:
- linux
---


由 王宇 原创并发布 ：



第1章UNIX基础知识 

1.1引言 

所有操作系统都需要向它们运行的程序提供各种服务。通常这些服务包含执行新程序、打开文件、读文件、分配存储区、以及获得当前时间等 。本章为不熟悉UNIX的程序设计人员简要介绍UNIX提供的各种服务。

1.2UNIX体系结构 

在严格意义上，可将操作系统定义为一种软件，它控制计算机硬件资源，提供程序运行环境，我们称此软件为内核（kernel），它相对较小 ，位于环境的中心。图1-1**



内核的接口被称为系统调用 

shell是一种特殊的应用程序，它为运行其他应用程序提供了一个接口。 

在广义上，操作系统包括了内核和一些其他软件，这些软件使得计算机能够发挥作用，并给予计算机以独有的特性。这些软件包括系统实用程序（system utilities）应用软件、shell以及公用函数库等  

1.3登录

(1)登录名 

/etc/passwd文件中查看登录名 

格式：登录名：加密口令:数值用户ID(205):数值组ID(105):注释字段:起始目录（/home/sar）:shell程序（/bin/ksh） 

                                                     sar:x:205:105:StephenRago:/home/sar:/bin/ksh

                                                     (2)shell 

                                                     shell是一个命令解释器，它读取用户输入，然后执行命令 。表1-1

                                                     1.4文件和目录 

                                                     1、文件系统 

                                                     UNIX文件系统是目录和文件组成的一种层次结构，目录的起点为根(root),其名字是一个字符/。目录(directory)是一个包含许多目录项的文件，在逻辑上，可以认为每个目录项都包含一个文件名，同时还包含说明该文件属性的信息。文件属性是指文件类型、文件大小、文件所有者、文件权限以及文件最后的修改时间等。stat和fstat函数返回包含所有文件属性的一个信息结构

                                                     2、文件名 

目录中的各个名字为文件名(file name)

  创建新目录时会自动创建两个文件名：.(称为点)和..(称为点一点)。点指当前目录，点一点则指父目录。在最高层次的根目录中，点一点与点相同 

  3、路径名 

  一个或多个以斜线分隔的文件名序列构成路径名，以斜线开头的路径名称为绝对路径名，否则称为相对路径名 

  程序清单1-1

  4、工作目录

  每个进程都有一个工作目录，有时称其为当前工作目录。进程可以用chdir函数更改其工作目录。 

  5、起始目录

  登录时，工作目录设置为起始目录，该起始目录从口令文件中相应用户的登录项中取得。

  1.5输入和输出 

  1、文件描述符 

  文件描述符通常是一个小的非负整数，内核用它标识一个特定进程正在访问的文件。当内核打开一个已有文件或创建一个新文件时，它返回一个文件描述符，在读、写文件时，就可使用它 

  2、标准输入、标准输出和标准出错 

按惯例，每当运行一个新程序时，所有shell都为其打开三个文件描述符：标准输入(standard input)、标准输出(standard output)、标准错误(standard error) 

  3、不用缓冲的I\O 

  函数open、read、write、lseek以及close提供了不用缓冲的I/O.这些函数都使用文件描述符。 

  4、标准I/O

  标准I/O函数提供一种对不用缓冲I/O函数的带缓冲的接口。使用标准I/O函数可以无需担心如何选择最佳的缓冲区大小。使用标准I/O函数的另一个优点是简化了对输入行的处理 

  1.6程序和进程 

  1、程序 

  程序是存放在磁盘上、处于某个目录中的一个可执行文件 

  2、进程和进程ID 

  程序的执行实例被称为进程 

  UNIX系统确保每个进程都有一个唯一的数字标识符，称为进程ID（processID ）

  3、进程控制 

  有三个用于进程控制的主要函数:fork、exec、waitpid 

  程序清单1-5

  4、线程和线程ID 

  在一个进程内的所有线程共享同一地址空间、文件描述符、栈及与进程相关的属性。

  线程也用ID标识，但是，线程ID只在它所属进程内起作用 

  1.7出错处理 

  当UNIX函数出错时，常常返回一个负值 ，而且整形变量errno通常被设置为含有附加信息的一个值。

  文件<errno.h>中定义了符号errno以及可以赋予它的各种常量，这些常量都以字符E开头

  在linux中，出错常量在errno(3)手册页中列出

  对于errno应当知道两条规则 。

  第一条规则是：如果没有出错，则其值不会被一个例程清除。因此，仅当函数的返回值指明出错时，才检验其值。

  第二条规则是：任一函数都不会将error值设置为0，在<errno.h>中定义的所有常量都不为0 

  C标准定义了两个函数，它们帮助打印错误信息

#include<string.h>

  char*strerror(interrnum);

#include<stdio.h>

  voidperror(constchar*msg); 

  出错恢复：

  可将在<errno.h>中定义的各种出错分成致命性和非致命性的两类。对于致命性的错误，无法执行恢复动作，而对于非致命性的错误，有时可以较妥善地进行处理。大多数非致命性出错在本质上是暂时的    

  1.8用户标识 

  1、用户ID

  口令文件登录项中的用户ID(userID)是个数值，它向系统标识各个不同的用户。系统管理员在确定一个用户的登录名的同时，确定其用户ID。用户不能更改其用户ID。通常每个用户有一个唯一的用户ID

用户ID为0的用户为根(root)或超级用户(superuser)

  2、组ID 

  口令文件等录项也包括用户的组ID(groupID),它是一个数值。组ID也是由系统管理员在指定用户登录名时分配的

  组文件将组名映射为数字组ID，它通常是/etc/group

  3、附加组ID 

  允许一个用户属于另外的组    

  1.9信号 

  信号(signal)是通知进程已发生某种事情的一种技术 

  1.10时间值 

  (1)日历时间

  (2)进程时间，也称为CPU时间 

  1.11系统调用和库函数 

  所有的操作系统都提供多种服务的入口点，程序由此向内核请求服务。这些入口点称为:系统调用 

  第2章UNIX标准化及实现

  2.1引言

  2.2UNIX标准化 

  2.2.1ISO C 

  1986年下半年，C程序设计语言的ANSI标准X3.159-1989得到批准。此标准已被采纳为国际标准ISO/IEC9899:1990。ANSI是美国国家标准学会(American National Standards Institute) ,它在国际标准化组织(International Organization for Standardization,ISO)中是代表美国的成员。IEC是国际电子技术委员会（International Electrotechnal Commission）的缩写 

  ISO C标准现在由ISO/IEC的C程序设计语言国际标准化工作组维护和开发，该工作组被称为ISO/IECJTC1/SC22/WG14简称WG14.ISO C标准的意图是提供C程序的可移植性，使其能适合于大量不同的操作系统，而不只是UNIX系统。此标准不仅定义了C程序设计语言的语法和语义，还定义了其标准库[ISO1999第7章；Plauger1992;Kernighan及Ritchie1988中的附录B]，所以该标准是很重要的。

  在1999年，ISO C标准被更新为ISO/IEC9899:1999

  gcc对ISO C标准1999版本的当前符合程度的总结可见：http://www.gnu.org/software/gcc/c99status.html

  ISO C库分成24个区，参见表2-1*** 



  2.2.2IEEE POSIX 

POSIX是一系列由IEEE（Institude of Electricaland Electronics Engineers,电气与电子工程师协会） 制定的标准。POSIX指的是可移植的操作系统接口(Portable OPerating SystemInterface) 

  由于1003.1标准定义了一个接口(interface)而不是一种实现(implementation),所以并不区分系统调用和库函数。标准中的所有例程都称为函数。

  标准是不断演变的，1003.1标准也不例外，最终的文档作为IEEEStd.1003.1-1990正式出版[IEEE1990],这也就是国际标准ISO/IEC9945-1:1990。

  表2-2、表2-3、以及表2-4总结了POSIX.1指定的必须和可选的头文件 。因为POSIX.1包括ISOC标准库函数，所以它还需要表2-1中列出的头文件。这4个表总结了本书所讨论的4种UNIX系统实现中包括的头文件

  POSIX.1标准现由称为Austion Group(http://www.opengroup.org/austin)的开发工作组维护













  2.2.3Single UNIX Specification 

Single UNIX Specification(单一UNIX规范)是POSIX.1标准的一个超集，定义了一些附加的接口，这些接口扩展了基本的POSIX.1规范所提供的功能。相应的系统接口全集被称为X/Open系统接口(XSI,X/OpenSystemInterface)

  XSI还定义了实现必须支持POSIX.1的哪些可选部分才能认为是遵循XSI的

  由OpenGroup发布

  2.2.4FIPS 

  含义是联邦信息处理标准(Federal Information Processmation Standard),它由美国政府出版。本书不进一步考虑它。 

  2.3UNIX系统实现 

  各自独立的组织所制定的三个标准：ISO C IEEE POSIX以及Single UNIX Specification,但是标准只是接口的规范。这些标准由制造商采用，然后转变成具体实现

  UNIX的各种版本和变体都起源与在PDP-11系统上运行的UNIX分时系统第6版(1976年)和第7版(1979年)(通常称为V6和V7)。这两个版本是在贝尔实验室以外首先得到广泛应用的UNIX系统，从树上演变出三个分支：

  （1）AT&T分支，从此导出了系统III和系统V（被称为UNIX的商用版本）

  （2）加州大学伯克利分校分支，从此导出4.xBSD实现

  （3）由AT&A贝尔实验室的计算科学研究中心开发的UNIX研究版本，从此导出UNIX分时系统第8、第9版以及于1990年发布的最后一版第10

  2.3.1SVR4 

  SVR4（UNIXSystem V Release4，UNIX系统V第4版）是AT&A的UNIX系统实验室(USL,其前身是AT&A的UNIXSofewareOperation)的产品。 它将下列系统的功能合并到一个一致的操作系统中：

  AT&T的UNIX系统V第3.2版(SVR3.2)、

  Sun Microsystem公司的SunOS操作系统，

  加州大学伯克利分校的4.3BSD版本

  微软的Xenix系统

  SVR4符合POSIX1003.1标准和X/Open Portability Guide第3版(XPG3)标准    

  2.3.2 4.4BSD 

  BSD(Berkeley Software Distribution)版是由加州大学伯克利分校的计算机系统研究组(CSRG)研究开发和分发的 

  2.3.3FreeBSD

  FreeBSD的基础是4.4BSD-Lite操作系统。在加州大学伯克利分校的CSRG决定终止其在UNIX操作系统的BSD版本上的研发工作后，并且386BSD项目看起来似乎被忽视了太长的时间，为了继续坚持BSD系统，设立了FreeBSD项目

  2.3.4Linux 

  Linux是一种提供丰富的UNIX编程环境的操作系统，在GNU公用许可证指导下，Linux是免费使用的。 

  Linux是由Linus Torvalds 在1991年为替代MINIX而研发的。一位当时名不见经传 人物的努力掀起了澎湃巨浪 ，吸引了遍布全世界的很多软件开发者，自愿地贡献出他们大量的时间来使用和不断地增强Linux

  2.3.5MacOSX 

  与其以前的版本相比，MacOSX使用了完全不同的技术，其核心操作系统被称为Darwin ，它基于Mach内核和FreeBSD操作系统的组合。类似于FreeBSD和Linux，Darwin是一个开放源代码项目         

  2.3.6Solaris 

  Solaris是由Sun公司开发的UNIX系统版本，它基于SVR4, 并在10余年间由Sun公司的工程师对其进行了不断的增强。它是唯一在商业上取得成功的SVR4后裔

  2.3.7其他UNIX系统

  AIX，IBM版的UNIX系统

  HP-UX，HP版的UNIX系统

  IRIX，Silicon Graphics版的UNIX系统

  UnixWare，SVR4派生的UNIX系统，现由SCO销售

  2.4标准和实现的关系 

  FreeBSD5.2.1 Linux2.4.22 MacOSX10.3和Solaris9.在这4种系统都提供UNIX编程环境。因为所有这4种系统都在不同程度上依从POSIX，所以我们也将重点关注POSIX.1标准所要求的功能，并指出这4中系统的具体实现与POSIX.1之间的差别

  2.5限制 

  UNIX系统实现定义了很多幻数 和常量 ，其中有很多已被硬编码进程序中，或用特定的技术确定。已有若干种可移植的方法用以确定这些幻数和实现定义的限制。这非常有助于软件的可移植性：

（1）编译时限制(例如，短整型的最大值是什么)

（2）运行时限制(例如，文件名可以有多少个字符) 

  编译时显示可在头文件中定义，程序在编译时可以包含这些头文件。但是，运行时限制则要求进程调用一个函数以获得这种限制值

  三种限制：

  （1）编译时限制（头文件）

  （2）不与文件或目录相关联的运行时限制（sysconf函数）

（3）与文件或目录相关联的运行时限制(pathconf和fpathconf函数) 

  2.5.1ISO C限制 

  ISOC定义的限制都是编译时限制。表2-6列出了文件<limits.h> 中定义的C标准限制 

  2.5.2POSIX限制

  POSIX.1定义了很多涉及操作系统实现限制的常量，不幸的是，这是POSIX.1中最令人迷惑不解的部分之一。我们只关心与基本POSIX.1接口有关的部分。这些限制和常量被分成下列5类：

  (1)不变的最小值：表2-8中的19个常量

  (2)不变值：SSIZE_MAX

  (3)运行时可以增加的值：CHARCLASS_NAME_MAX,COLL_WEIGHTS_MAX,LINE_MAX,NGROPUS_MAX以及RE_DUP_MAX

  (4)运行时不变的值(可能不确定)：ARG_MAX,CHILD_MAX,HOST_NAME_MAX,LOGIN_NAME_MAX,OPEN_MAX,PAGESIZE,RE_DUP_MAX,STREAM_MAXS,SYMLOOP_MAX,TTY_NAME以及TZNAME_MAX

  (5)路径名可变值（可能不确定）：FILESIZEBITS,LINK_MAX,MAX_CANON,MAX_INPUT,NAME_MAX,PATH_MAX,PIPE_BUF以及SYMLINK_MAX

  有些可定义在<limits.h>    





  表2-8



  2.5.3XSI限制 

  (1)不变最小值：表2-9中列出的10个常量

  (2)数值限制：LONG_BIT和WORD_BIT

  (3)运行时不变值，（可能不确定）：ATEXIT_MAX,IOV_MAX以及PAGE_SIZE



  2.5.4sysconf、pathconf和fpathconf函数

  后两个函数之间的差别是一个用路径名作为其参数，另一个则取文件描述符作为参数

  2.5.5不确定的运行时限制 

  观察两种特殊的情况：为一个路径名分配存储区，以及确定文件描述符的数目

  2.6选项 

  表2-5中列出了POSIX.1选项。如果我们要编写一些可移植的应用程序而这些程序与所有得到支持的选项有关，难么就需要一种可移植的方法以决定一种实现是否支持一个给定的选项

  2.7功能测试宏 

  如果在编译一个程序时，希望它只使用POSIX的定义而不使用任何实现定义的限制，那么就需定义常量_POSIX_C_SOURCE

  常量_POSIX_C_SOURCE即_XOPEN_SOURCE被称为功能测试宏(feturetestmacro)。所有功能测试宏都以下划线开始

  2.8基本系统数据类型 

  历史上，某些UNIX系统变量已与某些C数据类型联系在一起

  头文件<sys/types.h>中定义了某些与实现有关的数据类型，它们被称为基本系统数据类型(primitivesystedatatype),它们绝大多数都以_t结尾。表2-16某些常用的基本系统数据类型

  2.9标准之间的冲突 

  就整体而言，这些不同的标准之间配合得相当好。但是我们也很关注它们之间的差别，特别是IOSC标准和POSIX.1之间的差别，而它们之间也确实有些差别

  ISOC定义了函数clock，它返回进程使用的CPU时间，返回值类型是clock_t。为了将此值变换成以秒为单位，将其除以在<time.h>头文件中定义的CLOCKS_PER_SEC。POSIX.1定义了函数times,它返回其调用者以其所有终止子进程的CPU时间以及时钟时间，所有这些值都是clock_t类型值。sysconf函数用来获取每秒钟的滴答数，用于表示times函数的返回值。有一个相同的术语，即每秒钟的滴答数，但ISOC和POSIX.1的定义却不同

  另一个可能产生冲突的领域是：在ISO标准定义函数时，可能没有考虑到POSIX.1的某些要求



