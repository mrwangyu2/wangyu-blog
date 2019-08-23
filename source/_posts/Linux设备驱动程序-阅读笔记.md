---
title: Linux设备驱动程序-阅读笔记
date: 2012-06-20 09:28:05
tags:
- linux
---

LINUX设备驱动程序(LINUX DEVICE DRIVERS )--阅读笔记


由  王宇 原创并发布 ：


设备驱动程序简介 

设备驱动程序的作用 

内核功能划分 

进程管理

内存管理

文件系统

设备控制

网络功能

可装在模块

insmod将模块链接到内核

rmmode将模块移除内核

设备和模块的分类 

字符模块

块模块

网络模块

安全问题

版本编号

许可证条款 

加入内核开发社区 


构造和运行模块 

设置测试系统

HelloWorld模块 

insmod模块安装工具

rmmod模块移去工具

核心模块与应用程序的对比 

模块预先注册自己，用于服务某种请求

模块退出时需要释放资源

模块错误会影响到整个系统

用户空间和内核空间

不同地址空间

编译和装载

内核符号表 

预备知识

初始化和关闭

模块参数 

在用户空间编写驱动程序

快速参考 

用来装载模块到正运行的内核和移除模块的用户空间工具： 

insmod

modprobe

modprobe功能就是，对系统里的模块进行增、减、安装、删除等等操作。类似于insmod

rmmod

用于指定模块的初始化和清除函数的宏(可选的)



  __init

  __initdata

  __exit

  __exitdata



  最重要的头文件之一包含驱动程序使用的大部分内核API的定义 

#include<sched.h>

  当前进程：structtask_struct*current;

  当前进程的进程ID和命令:current->pidcurent->comm

  必要的头文件，它必须包含在模块代码中： 

#include<linux/module.h>

  构造内核版本信息的头文件 

#include<linux/version.h>

  整数宏，在处理版本依赖的预处理条件语句中非常有用 

  LINUX_VERSION_CODE

  用于导出单个符号到内核的宏 

  EXPORT_SYMBOL（symbol）

EXPORT_SYBMOL_GPL(symbol)

  用来声明模块初始化和清除函数的宏 

  module_init(int_function);

  module_exit(exit_function);

  在目标文件中添加关于模块的文档信息 

  MODULE_AUTHER(author);

  MODULE_DESCRIPTION();

  MODULE_VERSION();

MODULE_DEVICE_TABLE()

MODULE_ALIAS()

  用来创建块参数的宏，用户可在装载模块时调整这些参数的值

#include<linux/moduleparam.h>

  module_param(varible,type,perm);

  函数printk的内核代码 

#include<linux/kernel.h>

  intprintk(constchar*fmt,...); 

  字符设备驱动程序 

  scull的设计 

  全局

  持久

  FIFO（先入先出）

  主设备号和次设备号 

  在dev目录下ls-l 

  字符设备用“c”识别

  块设备用“b”识别

  主设备号标识设备对应的驱动程序，第一列

  次版本号由内核使用，第二列

  设备编号的内部表达 

  linux/types.h中定义

  分配和释放设备编号

  动态分配主设备号

  /proc/devices中表示主版本号

  增加文件节点

#mknod/dev/sc_devc2500

  一些重要的数据结构（非常重要） 

  file_operations 

  open、write、read、release、close等函数关联到file_operations数据结构中 

  初始化设备时，初始化数据结构file_operations

init_modules()

  cdev_init();等

  书中55页的file_operations方法清单

  file 

  代表一个打开的文件

  inode 

  内核在内部表示文件

执行流程：获取设备号->注册设备->关联Fileoperations结构->open(打开设备)->write->read->release资源->close(关闭设备) 

  字符设备的注册 

  open和release 

  Open方法提供初始化的能力

  检查设备特定的错误

  设备首次打开则初始化

  有必要，则更新f_op指针

  分配并填写置于filp->private_data里的数据结构

  Release方法 

  释放由Open分配的、保存在filp->private_data中的所有内容

  在最后一次关闭操作时关闭设备

  scull的内存使用

  read和write

  试试新设备

  快速参考 

  dev_t是内核中用来表示设备编号的数据类型

#include<linux/types.h>

  dev_t

  从设备编号中抽取出主、次设备号 

intMAJOR(dev_tdev)

intMINOR(dev_tdev)

  由主次设备号构造一个dev_t数据项 

  dev_tMKDEV(unsignedintmajor,unsignedintminor);

  "文件系统"头文件，它是编写设备驱动程序必须的头文件，有很多的重要的函数和数据结构 

#include<linux/fs.h>

  intregister_chrdev_region(dev_tfirst,unsignedintcount,char*name);

  提供给驱动程序用来分配和释放设备编号范围的函数

  intalloc_chrdev_region(dev_tfirst,unsignedintfirstminor,unsignedintcount,char*name);

  动态分配，使用alloc_chrdev_region

  intunregister_chrdev(unsignedintmajor,constchar*name);

  用于注销驱动程序

  三个重要的数据结构 

  struct file_operation;保存了字符驱动程序的方法

  struc tfile表示一个打开的文件

  struct inode表示一个磁盘上的文件 

用来管理cdev(用来代表字符设备)结构的函数，内核中使用该结构表示字符设备(过时不用了) 

#include<linux/cdev.h>

  struct cdev*cdev_alloc(void);

  void cdev_init(structcdev*dev,structfile_operation*fops);

  int cdev_add(structcdev*dev,dev_tnum,unsignedintcount);

  void cdev_del(structcdev*dev);

  从包含在某个结构中的指针获得结构本身的指针

#include<linux/kernel.h>

  container_of(pointer,type,field);

  在内核代码和用户空间之间移动数据的函数 

  unsigned longcopy_from_user(void*to,constvoid*from,unsignedlongcount);

  unsigned longcopy_to_user(void*to,constvoid*from,unsignedlongcount);

  调试技术 

  内核中的调试支持 

  编译内核，打开相关选项 

  make menuconfig

  选择kernel-hacking项 

  按"?"现实帮助，会显示出变量名称。CONFIG_*

  参考书中的内容

  按“/”查找信息

  symbol:查询的符号

  prompt:提示。应该是菜单名称

  definds:定义

  dependson:依赖

  打印调试 

  使用printk 

  使用dmesg来查看信息

  查询最后几条：dmesg|tail

  驱动开发用得最多的还是printk啦

  重定向控制台消息

  消息记录 

  klogd

  klogd是一个专门截获并记录Linux内核消息的守护进程

  klogd守护进程获得并记录Linux内核信息。通常，syslogd会记录klogd传来的所有信息，然而，如果调用带有-ffilename变量的klogd时，klogd就在filename中记录所有信息，而不是传给syslogd。

  /var/log/message等文件中记录

  开启及关闭消息

  速度限制

  打印设备编号 

  查询调试 

  使用/proc文件系统 ** 

  每一个文件对应一个设备

  在/proc中实现文件

  老的/proc接口

  创建自己的/proc文件

  seq_file接口

  ioctl方法

  内核向外界导出信息

  监视调试 

  strace（重点使用） 

  是一个非常强大的工具，可以显示由用户空间程序所发出的所有系统调用

  输出结果，不知道如何分析

  调试系统故障 

  内核大多数错误只会导致oops消息

  系统挂起 

  工具SysRq

  调试器和相关工具 

  使用gdb 

  不能执行典型的调试命令，例如：设置断点或者修改变量值

  kdb内核调试器 

  首先获得补丁，对内核进行patch操作。

  kgdb补丁

  用户模式的linux虚拟机

  linux跟踪工具包 

  内核调试技术（培训教材中）

  通过打印函数

  获取内核信息

  处理出错信息

  内核代码调试

  并发和竟态 

  scull的缺陷 

  竞态 

  导致对共享数据的非控制访问，结果是内存泄露、系统崩溃等问题

  并发及其管理 

  规则一：只要可能，就应该避免资源的共享 

  避免使用全局变量

  硬规则：显式地管理对该资源的访问

  访问管理常见技术：“锁定”、“互斥”

  重要规则：对象必须在还有其他组件引用自己时保持存在

  访问管理的常见技术称为锁定或互斥 

  信号量和互斥体 

  临界区：任意给定的时刻，代码只能被一个线程执行 

  进入临界区调用函数P，信号量大于零进程继续，否则进程等待。解锁调用函数V

  信号量本质上是一个整数值，有时也称为一个“互斥体”

  互斥（锁定）：避免多个进程同时在一个临界区中运行 

  信号量值应初始化为1

  LINUX信号量的实现

  在scull中使用信号量

  读取者/写入者信号量 

  信号量对所有的调用者执行互斥，但是许多任务可以划分为两种不同的工作类型：

  一、只需要读取受保护的数据结构

  二、必须做出修改

  信号量和互斥概念 

  互斥：即两个线程不能同时进入被互斥保护的代码。

  信号量的使用主要是用来保护共享资源,使的资源在一个时刻只有一个进程所拥有.为此我们可以使用一个信号灯.当信号灯的值为某个值的时候,就表明此时资源不可以使用.否则就表>示可以使用.

  Completion 

  轻量级的机制 

  LINUX信号量的实现

  它允许一个线程告诉另一线程某个工作已经完成

  自旋锁 

  自旋锁比信号量性能更高

  自旋锁API介绍

  自旋锁和原子上下文

  任何拥有自旋锁的代码都必须是原子的

  自旋锁函数

  读取者/写入者自旋锁

  锁陷阱 

  不明确的规则 

  锁模式在开始就要安排好，否则以后改进非常困难

  不允许锁拥有者第二次获得这个锁

  提供给外部调用的函数则必须显式地处理锁定

  锁的顺序规则 

  必须获得多个锁时，应该始终以相同的顺序获得

  细粒度锁和粗粒度锁的对比

  除了锁之外的方法 

  免锁算法

  原子变量

  位操作

  seqlock

  读取-复制-更新

  快速参考 

  信号量定义的包：#include<asm/semaphore.h> 

  声明和初始化用在互斥模式中的信号量得两个宏

  DECLARE_MUTEX(name);

  DECLARE_MUTEX_LOCKED(name);

  运行时初始化

  void init_MUTEX(structsemaphore*sem);

  void init_MUTEX_LOCKED(structsemaphore*sem);

  锁定和解锁信号量。如果必要，down会将调用进程至于不可中断的休眠状态；相反，down_interruptible可被信号中断.down_trylock不会休眠，并且会在信号量不可用时立即返回。锁定信号量的代码最后必须使用up解锁该信号量

  void down(structsemaphore*sem);

  int down_interruptible(structsemaphore*sem);

  in tdown_try(structsemaphore*sem);

  void up(structsemaphore*sem);

  读取者、写入者

  初始化

  struct rw_semaphore;

  init_rwsem(structrw_semaphore*sem);

  读取访问

  void down_read(structrw_semaphore*sem);

  void down_read_trylock(structrw_semaphore*sem);

void up_read(structrw_semaphore*sem)

  写人访问

  void down_write(structrw_semaphore*sem);

  void down_write_trylock(structrw_semaphore*sem);

  void up_write(structrw_semaphore*sem);

  void downgrade_write(structrw_semaphore*sem);

  completion机制的包含文件：#include<linux/completion.h> 

  DECLARE_COMPLETION(name);

  int_completion(structcompletion*c);

  INIT_COMPLETION(structcompletion*c);

  等待一个completion事件的发生

  void wait_for_completion(structcompletion*c);

  发出completion事件信号

  void completion(structcompletion*c);只唤醒一个等待的线程

  void completion_all(structcompletion*c);唤醒所有的等待者

  通过调用completion并调用当前线程的exit函数而发出completion事件信号

  定义自旋锁接口的包含文件：#include<linux/spinlock.h> 

  spinlock_tlock=SPIN_LOCK_UNLOCKED;

  spinlock_lock_init(spinlock_t*lock);

  锁定自旋锁

  void spin_lock(spinlock_t*lock);

  void spin_lock_irqsave(spinlock_t*lock,unsignedlongflags);

  void spin_lock_irq(spinlock_t*lock);

  void spin_lock_bh(spinlock_t*lock);

  未完，请参考书中的内容

  高级字符驱动程序操作 

  ioctl 

  通过设备驱动程序执行各种类型的硬件控制

  比如，用户空间经常会请求设备锁门、弹出介质、报告错误信息、改变波特率或者执行自破坏

  系统调用中，具有ioctlApi接口

  选择ioctl命令

  返回值

  预定义命令

  使用ioctl参数

  权能与受限操作

  ioctl命令的实现

  非ioctl设备控制

  阻塞型I/O 

  休眠的简单介绍

  永远不要在原子上下文中进入休眠

  当我们被唤醒时，永远无法知道休眠了多长时间，或者休眠期间都发生了些什么事情

  除非我们知道有其他人会在其他地方唤醒我们，否则进程不能休眠。

  简单休眠

  阻塞和非阻塞型操作



  一个阻塞I/O示例

  高级休眠

  进程如何休眠

  手工休眠

  独占等待

  唤醒的相关细节

  poll和select 

  与read和write的交互

  底层数据结构

  异步通知

  定位设备

  设备文件的访问控制

  快速参考

  ioctl命令的所有宏：#include<linux/ioctl.h>#include<linux/fs.h>

  以下宏不知道如何使用

  __IOC__NRBITS

  __IOC__TYPEBITS

  __IOC__SIZEBITS

  __IOC__DIRBITS

  __IOC__NONE

  __IOC__READ

  __IOC__WRITE

  用于生成ioctl命令的宏

__IOC(dir,type,nr,size)

__IO(type,nr)

__IOR(type,nr,size)

__IOW(type,nr,size)

__IOR(type,nr,size)

  用于解码ioctl命令的宏

__IOC_DIR(nr)

__IOC_TYPE(nr)

__IOC_NR(nr)

__IOC_SIZe(nr)

  用户空间的交互 

  验证指向用户空间的指针是否可用，如果允许访问，access_ok返回非零值

#include<asm/uaccess.h>

int access_ok(inttype,constvoid*addr,unsignedlongsize)

  用于向用户空间保存单个数据项的宏

#include<asm/uaccess.h>

  int put_user(datum,ptr);

  int get_user(local,prt);

  int __put_user(datum,prt);

  int __get_user(local,prt);

  定义有各种CAP_符号，用于描述用户空间进程可能拥有的权能操作

#include<linux/capability.h>

  int capable(intcapability);

  等待队列 

  预定义等待队列类型#include<linux/wait.h>

  typedefstruct(/*..*/)wait_queue_head_t;

  void init_waitqueue_head(wait_queue_head_t*queue);

  DECLARE_WAIT_QUEUE(queue);

  使进程在指定的队列上休眠，知道给定的condition值为真

  void wait_event(wait_queue_head_tq,intcondition);

  void wait_event_interruptible(wait_queue_head_tq,intcondition);

  void wait_event_timeout(wait_queue_head_tq,intconditon,inttime);

  void wait_event_interruptible_timeout(wait_queue_head_tq,intconditon,inttime);

  唤醒休眠在队列q上的进程

  void wait_up(structwait_queue**q);

  void wait_up_interruptible(structwait-queue**q);

  void wait_up_nr(structwait_queue**q);

  voidwait_up_interruptible_nr(structwait_queue*q);

  void wait_up_all(structwait_queue*q);

  void wait_up_interruptible_all(structwait_queue*q);

void wati_up_interruptibel_sync(structwait_queue*q)

  设置当前进程的状态#include<linux/schud.h>

set_current_state(intstate)

  TASK_RUNNINGTASK_INTERRUPTIBLETASK_UNINTERRUPTIBLE

  从运行队列中选择一个可运行的进程

  void schedule（void）

  用于将某个进程放置到一个等待队列中

  init_waitqueue_entry(wait_queue_t*entry,structtask_struct*task);

  可用于手工休眠代码的辅助函数

void perpare_to_wait()

void perpare_to_wait_exclusive()

  void finish_wait

  将当前进程置于某个等待队列但并不立即调度。主要用于poll方法

#include<linux/poll.h>

  voidpoll_wait(structfile*filp,wait_queue_heard*q,poll_table*p);

  用来实现fasync设备方法的辅助函数

  int fasync_helper(structinode*inode,structfile*filp,intmode,structfasync_struct*fa);

  发送一个信号给注册在fa中的进程

  int kill_fasync(structfasync_struct*fa,intsig,intband);

  任何不支持定位的设备都应该在其open方法中调用nonseekable_open.这类设备还应该在其llseek方法中使用no_llseek

  int nonseekable_open();

  loff_tno_llseek();

  阻塞、非阻塞、异步通知图解 



  .

  时间、延迟及延缓操作 

  概述 

  如何度量时间差，如何比较时间

  如何获得当前时间

  如何将操作延迟指定的一段时间

  如何调度异步函数到指定的时间之后执行

  度量时间差 

  使用jiffiles计数器

  处理器特定的寄存器

  获取当前时间

  延迟执行 

  长延迟

  忙等待

  让出处理器

  超时

  短延迟

  内核定时器 

  定时器API

  内核定时器的实现

  tasklet 

  和内核定时器不同的是，我们不能要求tasklet在某个给定时间执行

  工作队列 

  共享队列

  tasklet会在很短的时间段内很快执行，并且以原子模式执行，而工作队列函数可具有更长的延迟并且不必原子化

  快速参考 

  时钟：#include<linux/param.h> 

  HZ符号指出每秒钟产生的时钟滴答数

  jiffies_64变量会在每个时钟滴答递增

  volatieunsignedlongjiffies

  u64jiffies_64

  以安全方式比较：jiffies

  int time_after(unsignedlonga,unsignedlongb);

  int time_before(unsignedlonga,unsignedlongb);

  int time_after_eq(unsignedlonga,unsignedlongb);

  int time_before_eq(unsignedlonga,unsignedlongb);

  无竞争地获得jiffies_64的值

  u64 get_jiffies_64(void);

  在jiffies表示的时间和其他表示法之间转换

#include<linux/time.h>

  unsigned long timespec_to_jiffies(structtimespec*value);

  void jiffies_to_timespec(unsignedlongjiffies,structtimespec*value);

  unsigned longtimeval_to_jiffies(structtimeval*value);

  void timeval_to_jiffies(unsignedlongjiffies,structtimeval*value);

  x86专用宏，用来读取时间瞬计数器

#include<asm/msr.h>

  rdtsc(low32,high32);

  rdtscl(low32);

  rdtscll(var32);

  以平台无关的方式返回时间瞬计算器

#include<linux/timex.h>

cycle_tget_cycles(void)

  根据6个无符号的int参数返回在Epoch以来的秒数

#include<time.h>

  unsigned long mktime(year,mon,day,h,m,s);

  以自Epoch以来的秒数和毫秒数的形式返回当前时间

  void do_gettimeofday(structtimeval*tv);

  以jiffies为分辨率返回当前时间

  struct timespeccurrent_kernel_time(void);

  延时：#include<linux/wait.h> 

  是当前进程休眠在等待队列上，并指定用jiffies表达的超时值

  longwait_event_interruptible_timeout(wait_queue_interruptible*q,intconditon,unsingedlongtimeout);

  调用调度器，确保当前进程可在给定的超时值之后被唤醒

#include<linux/sched.h>

  signed long schedule_timeout(signedlongtimeout);

  引入整数的纳秒、微秒、毫秒级延时

#include<linux/delay.h>

  void ndelay(unsignedlongnsec);

  void udelay(unsignedlongusec);

  void mdely(unsignedlongmsec);

  使进程休眠给定的毫秒数

  void msleep(unsignedlongmillisec);

  unsigned long msleep_interruptible(unsignedlongmillisec);

  void ssleep(unsignedintseconds);

  内核定时器 

  返回布尔值已告知调度代码是否在中断上下文或者在原子上下文中执行

#include<asm/hardirq.h>

  int in_interruptible(void);

  int in_atomic(void);

  初始化定时器

#include<linux/timer.h>

  void init_timer(structtimer_list*timer);

  struct timer_listTIMER_INITIALIZER(_function,_expires,_data);

  注册定时器结构，以在当前CPU上运行

  voidadd_timer(structtimer_list*timer);

  修改一个已经调度的定时器结构的到期时间

  intmod_timer(structtimer_list*timer,unsignedlongexpires);

  用来判断给定的定时器结构是否已经被注册运行

int timer_pending(structtimer_list*timer)

  从活动定时器清单中删除一个定时器

  void del_timer(structtimer_list*timer);

  void del_timer_sync(structtimer_list*timer);

  tasklet 

  声明tasklet结构和初始化

#include<linux/interrupt.h>

  DECLARE_TASKLET(name,func,data);

  DECLARE_TASKLET_DISABLED(name,func,data);

  禁用或重新启用tasklet

  void tasklet_disable(structtasklet_struct*t);

  void tasklet_disable_nosync(structtasklet_struct*t);

  void tasklet_enable(structtaslet_struct*t);

  调度运行某个tasklet

  void tasklet_schedule(structtasklet_struct*t);

  void tasklet_hi_schedule(structtaskletstruct*t);

  从活动列表中删除

  void tasklet_kill(structtasklet_struct*t);

  工作队列

  结构和入口项

#include<linux/workqueue.h.>

  struct workqueue_struct;

  struct work_struct;

  用于创建和消除工作队列

  struct workqueue_struct*create_workqueue(constchar*name);

  struct workqueue_struct*create_singlethread_workqueue(constchar*name);

  声明和初始化工作队列

  DECLARE_WORK();

  INIT_WORK();

  PREPARE_WORK;

  用于安排工作以便从工作队列中执行的函数

int queue_work(structworkqueue_struct*queue,structwork_struct*work)

  intqueue_delayed_work(structworkqueue_struct*queue,structwork_struct*work,unsignedlongdelay);

  删除一个入口项

  int cancel_delayed_work(structwork_struct*work);

  void flush_workqueue(structworkqueue_struct*queue);

  使用共享工作队列的函数

  int schedule_work(structwork_struct*work);

  int schedule_delayed_work(structwork_struct*work);

  void flush_schedule_work(void);

  分配内存 

  kmalloc函数的内幕

  后备高速缓存

  get_free_page和相关函数

  vmalloc及其辅助函数

  per-CPU变量

  获取大的缓冲区 

  与硬件通讯 

  I/O端口和I/O内存 

  I/O寄存器和常规内存

  I/O操作具有边际效应

  外部设备寄存器和内存结构的映射可以有两种方法1映射到内存地址中，但是不占用物理内存；ARM体系结构采用这种方法。2映射到IO端口号中，使用专门的CPU命令INOUT进行处理；X86体系结构采用这种方法。

  控制寄存器、状态寄存器和数据寄存器三大类

  使用I/O端口 

  I/O端口分配

  操作I/O端口

  在用户空间访问I/O端口

  串操作

  暂停时I/O

  平台相关性

  相关平台IO表

  I/O端口示例 

  并口介绍

  示例驱动程序

  使用I/O内存 

  I/O内存分配和映射

  访问I/O内存

  像I/O内存一样使用端口

  为I/O内存重用short

  1MB地址空间之下的ISA内存

  isa_readb及相关函数

  快速参考 

  内存屏蔽 

  “软件”内存屏蔽

#include<linux/kernel.h>

  void barrier(void);

  “硬件”内存屏蔽

#include<asm/system.h>

  void rmb(void);

  void wmb(void);

  void read_barrier_depends();

  void mb();

  用于读写I/O端口

#include<asm/io.h>

  unsigned inb(unsignedport);

  void outb(unsignedcharbype,unsignedport);

  unsigned inw(unsignedport);

  void outw(unsignedshortword,unsignedport);

  unsigned inl(unsignedport);

  void outl(unsigneddoubleword,unsignedport);

  如果用户空间有访问端口的权限

  unsigned inb_p(unsignedport);....

  端口 

  为端口分配资源的函数

  struct resourcerequest_region(unsignedlongstart,unsignedlonglen,char*name);

  void release_region(unsignedlongstart,unsignedlonglen);

  int check_region(unsignedlongstart,unsignedlonglen);

  这类传输是通过对同一端口连续读/写count次实现

  void insb(unsignedport,void*addr,longcount);

  void outsb(unsinedport,void*addr,longcount);

  void intwb(unsignedport,void*addr,longcount);

  void outwb(unsignedport,void*addr,longcount);

  void inlb(unsignedport,void*addr,longcount);

  voidoutlb(unsignedport,void*addr,longcount);

  内存IO 

  处理对内存区域的资源分配

  struct resourcerequest_mem_region(unsignedlongstart,unsignedlonglen,char*name);

  void release_mem_region(unsignedlongstart,unsignedlonglen);

  int check_mem_reqion(unsignedlongstart,unsignedlonglen);

  把一个物理地址范围重新映射到处理器的虚拟地址空间，以提供内核使用

#include<asm/io.h>

  void*ioremap(unsignedlong*phys_add,unsignedlongsize);

  void*ioremap_nocache(unsignedlong*phys_add,unsignedlongsize);

  voidiounmap(void*virt_addr);

  用来访问I/O内存

#inlcude<as/io.h>

  unsigned int ioread8(void*addr);

  unsigned int ioread16(void*addr);

  unsigned int ioread32(void*addr);

  voidio write8(u8value,void*addr);

  voidio write16(u16value,void*addr);

  voidio write32(u32value,void*addr);

  重复版本

  void ioread8_req(void*addr,void*buf,unsignedlongcount);

  void iowrite8_req(void*addr,void*buf,unsignedlongcount);

  不安全的老IO内存函数

  unsigned readb(address);

  unsigned readw(address);

  unsigned readl(address);

  void writeb(unsignedvalue,address);。。。

  如果驱动程序作者希望将I/O端口作为I/O内存一样进行操作

  void*ioport_map(unsignedlongport,unsignedintcount);

  voidioport_unmap(void*addr);

  中断处理 

  准备并口

  安装中断处理例程

  实现中断处理例程

  顶半部和底半部

  中断共享

  中断驱动的I/O 

  快速参考 

  注册和注销中断处理例程 

#include<linux/interrupt.h>

  int request_irq(unsignedintirq,irqrequst_t(*handler)(),unsigndlongflags,constchar*dev_name,void*dev_id);

  void free_irq(unsignedintirq,void*dev_id);

  i386和x86_64体系架构上可用 

  int can_request_irq(unsignedintirq,unsignedlongflag);

  request_irq函数的标志 

#include<asm/signal.h>

  SA_INTERRUPT安装一个快速的处理例程

  SA_SHIRQ安装一个共享的处理例程

  SA_SAMPLE_RANDOM中断时间瞬可用来产生系统X

  文件系统节点用于汇报关于硬件中断和已安装处理例程的信息 

  /proc/interrupts

  /proc/stat

  当驱动程序不得不深探测设备，以确定该设备使用那根中断信号线时 

  unsigned long probe_irq_on(void);

  int probe_irq_off(unsignedlong);

  中断处理例程的可能返回值 

  IRQ_NONE

  IRQ_HANDLE

  IRQ_RETVAL

  驱动程序可以启动和禁止中断报告 

  voiddisable_irq(intirq);

  voiddisable_irq_nosync(intirq);

  voidenable_irq(intirq);

  禁止和启用本地处理器上的中断 

  void local_irq_save(unsignedlongflags);

  void local_irq_restore(unsignedlongflags);

  用于无条件禁用和启用当前处理器中断的函数 

  void local_irq_disable(void);

  void local_irq_enable(void);

  内核的数据类型 

  内核开发人员移植遇到若干问题都和不正确数据类型有关

  使用标准C语言类型

  为数据项分配确定的空间大小

  接口特定的类型

  其他有关移植性的问题

  链表 

  PCI驱动程序

  USB驱动程序

  Linux设备模型

  内存映射和DMA

  块设备驱动程序

  网络驱动程序

  TTY驱动程序
