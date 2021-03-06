---
title: 深入理解Linux内核-定是测量
date: 2012-11-17 09:23:39
tags:
- linux
---


由 王宇 原创并发布 ：



第六章定时测量 

很多计算机化的活动都是由定时测量(timing measurement)来驱动的，这常常对用户是不可见的。

Linux内核必须完成两种主要的定时测量 ，我们可以对此加以区分：

保存当前的时间和日期 ，以便能通过time()、ftime()和gettimeofday()系统调用把它们返回给用户程序，也可以由内核本身把当前时间作为文件和网络包的时间戳。

维持定时器 ，这种机制能够告诉内核或用户程序某一时间间隔已经过去

定时测量是由基于固定频率振荡器和计数器的几个硬件电路完成的。    

1、时钟和定时器电路 

[1]实时时钟(RTC) 

  所有的PC都包含一个叫实时时钟 (Renl Time Clock RTC)的时钟，它是独立于CPU和所有其他芯片的

  即使当PC被切断电源，RTC还继续工作，因为它靠一个小电池或蓄电池供电。COMS RAM和RTC被集成在一个芯片上。 

  RTC能在IRQ8上发出周期性的中断，频率在2-8192Hz之间。也可以对RTC进行编程以使当RTC到达某个特定的值时激活IRQ8线，也就是作为一个闹钟来工作。

  Linux只用RTC来获取时间和日期， 不过通过对/dev/rtc设备文件进行操作，也允许进程对RTC编程。内核通过0x70和0x71I/O端口访问RTC。系统管理员通过执行Unix系统时钟程序可以设置时钟。

[2]时间戳计数器(TSC)

  所有的80x86微处理器都包含一条CLK输入引线，它接收外部振荡器的时钟信号。 

  与可编程间隔定时器传递来的时间测量相比，Linux利用这个寄存器可以获得更精确的时间测量

[3]可编程间隔定时器(PIT) 

  可编程间隔定时器(Programmable Interval Timer PIT).PIT的作用类似于微波炉的闹钟，即让用户意识到烹调的时间间隔已经过了。所不同的是，这个设备不是通过振铃，而是发出一个特殊的中断，叫做时钟中断（timerinterupt）来通知内核又一个时间间隔过去了。与闹钟的另一个区别是，PIT永远以内核确定的固定频率不停第发出中断 

  Linux给PC的第一个PIT进行编程，是它以（大约）1000Hz的频率向IRQ0发出时钟中断，即每1ms产生一次时钟中断。这个时间间隔叫做一个节拍(tick),它的长度以纳秒为单位存放在tick_nsec变量中。

  短的节拍产生较高分辨率的定时器，当这种定时器执行同步I/O多路复用(poll()和select()系统调用)时，有助于多媒体的平滑播放和较快的响应时间

  时钟中断的频率取决于硬件体系结构。

  在Linux的代码中，有几个宏产生决定时钟中断频率的常量：

  HZ产生每秒时钟中断的近似个数，也就是时钟中断的频率。在IBM PC上，这个值设置为1000

  CLOCK_TICK_RATE 产生的值为1193182，这个值是8254芯片的内部振荡器频率

  LATCH产生CLOCK_TICK_RATE和HZ的比值再四舍五入后的整数值。这个值用来对PIT编程

  PIT由setup_pit_timer()进行初始化    

  [4]CPU本地定时器 

  CPU本地定时器是一种能够产生单步中断或周期性中断的设备，它类似于可编程间隔定时器，区别： 

  APIC计数器是32位，而PIC计数器是16位； 因此，可以对本地定时器编程来产生很低频率的中断

  本地APIC定时器把中断只发送给自己的处理器，而PIT产生一个全局性中断，系统中的任一CPU都可以对其处理。

  APIC定时器是基于总线时钟信号的。每隔1,2,4,8,16,32,64或128总线时钟信号到来时对该定时器进行递减可以实现对其编程的目的。相反，PIT有其自己的内部时钟振荡器，可以更灵活地编程。

[5]高精度事件定时器(HPET) 

  高精度事件定时器是由Intel和Microsoft联合开发的一种新型定时器芯片。尽管这种定时器在终端用户机器上还并不普遍，但Linux2.6已经能够支持它们

  [6]ACPI电源管理定时器 

  ACPI电源管理定时器（或称ACPI PMT）是另一种时钟设备，包含在几乎所有基于ACPI的主板上。它的时钟信号拥有大约为3.58MHz的固定频率。该设备实际上是一个简单的计数器， 它在每个时钟节拍到来时增加一次。为了读取计数器的当前值，内核需要访问某个I/O端口，该I/O端口的地址由BIOS在初始化阶段确定

  如果操作系统或者BIOS可以通过动态降低CPU的工作频率或者工作电压来节省电池的电能，那么ACPI电源管理定时器就比TSC更优越。当发生ACPI PMT的频率不会改变。而另一方面，TSC计数器的高频率非常便于测量特别小的时间间隔。

  ACPI 控制CPU的频率，从而控制系统的功耗。

  2、Linux计时体系结构 

  Linux必定执行与定时相关的操作。例如，

  内核周期性地：

  更新自系统启动以来所经过的时间

  更新时间和日期

  确定当前进程在每个CPU上已运行了多长时间，如果已经超过了分配给它的时间，则抢占它。时间片（也叫时限）

  更新资源使用统计数

  检查每个软定时器的时间间隔是否已到。

  Linux的计时体系结构(time keeping architecture)是一组与时间流相关的内核数据结构和函数 

  在单处理器系统上，所有的计时活动都是由全局定时器（可以是可编程间隔定时器也可以是高精度事件定时器）产生的中断触发的

  在多处理器系统上，所有普通的活动（像软定时器的处理）都是由全局定时器产生的中断触发的，而具体CPU的活动是由本地APIC定时器产生的中断触发的。

  [1]计时体系机构的数据结构 

  (1)定时器对象

  为了使用一种统一的方法来处理可能存在的定时器资源，内核使用了"定时器对象"，它是timer_opts类型的一个描述符，该类型由定时器名称和四个标准的方法组成，如表6-1所示：**



  定时器对象中最重要的方法是mark_offset和get_offset.mark_offset方法由时间中断处理程序调用，并以适当的数据结构记录每个节拍到来时的准确时间。get_offset方法使用已记录的值来计算自上一次时钟中断(节拍)以来经过的时间(us为单位).由于这两种方法，使得Linux计时体系结构能够达到子节拍的分辨度，也就是说，内核能够以比节拍周期更高的精度来测定当前的时间，这种操作被称作“定时插补(timerinterpolation)”

  表6-2以优先级顺序列出了80x86体系结构中最常用的定时器对象



  (2)jiffies变量 

  jiffies变量是一个计数器，用来记录自系统启动以来产生的节拍总数。每次时钟中断发生时（每个节拍）它便加1.在80x86体系结构中，jiffies是一个32位的变量，因此每隔大约50天它的值会回绕(wraparound)到0，这对Linux服务器来说是一个相对较短的时间间隔。不过，由于使用了time_after、time_afer_eq、time_before和time_before_eq四个宏，内核干净利索地处理了jiffies变量的益出。

  在80x86系统中，jiffies变量通过连接器被换算成一个64位计数器的低32位，这个64位的计数器被称作jiffies64.在1ms为一个节拍的情况下，jiffies_64变量将会在数十亿年后才发生回绕，所以我们可以放心地假定它不会溢出。

  (3)xtime变量 

  xtime变量存放当前时间和日期；它是一个timespec类型的数据结构，该结构有两个字段：

  tv_sec：存放自1970年1月1日(UTC)午夜以来经过的秒数

  tv_nsec:存放自上一秒开始经过的纳秒数

  xtime变量通常是每个节拍更新一次，也就是说，大约每秒更新1000次。用户程序从xtime变量获得当前时间和日期。内核也经常引用它                

  [2]单处理器系统上的计时体系结构 

  在单处理器系统上，所有与定时有关的活动都是由IRQ线0上的可编程间隔定时器产生的中断触发的。

  （1）初始化阶段：time_init(),操作参见p236-237

  （2）时钟中断处理程序：timer_interrupt(),步骤参见p237-238

  [3]多处理器系统上的计时体系结构    

  多处理器系统可以依赖两种不同的时钟中断源：可编程间隔定时器或高精度事件定时器产生的中断，以及CPU本地定时器产生的中断。

  在linux2.6中，PIT或HPET产生的全局时钟中断触发不涉及具体CPU的活动，比如处理器软定时器和保持系统时间的更新。相反，一个CPU本地时间中断触发涉及本地CPU的计时活动，例如监视当前进程的运行时间和更新资源使用统计数

  （1）初始化阶段

  （2）全局时钟中断处理程序

  （3）本地时钟中断处理程序

  3、更新时间和日期 

  用户程序从xtime变量中获得当前时间和日期。 内核必须周期性地更新该变量，才能使它的值保持相当的精确

  全局时钟中断处理程序调用update_times() 函数更新xtime 变量的值

  4、更新系统统计数 

  内核在与定时相关的其他任务中必须周期性地收集若干数据用于：

  检查运行进程的CPU资源限制

  更新与本地CPU工作负载有关的统计数

  计算平均系统负载

  监管内核代码

  [1]更新本地CPU统计数 

  update_process_times()步骤：参考p241-242

  [2]记录系统负载

  任何Unix内核都要记录系统进行了多少CPU活动。这些统计数据由各种管理实用程序来使用（如top）。用户输入uptime命令后可以看到一些统计数据：如相对于最后1分钟、5分钟、15分钟的“平均负载”。在单处理器系统上，值0意味着没有活跃的进程（除了swapper进程0）在运行，而值1意味着一个单独的进程100%占有100，值大于1说明几个运行着的进程共享CPU

  update_times()在每个节拍都要调用calc_load()函数来计算处于TASK_RUNNING或TASK_UNINTERRUPTIBALE状态的进程数，并用这个数据更新平均系统负载。

  [3]监管内核代码

  Linux包含一个被称为read profiler的最低要求的代码监管器，Linux开发者用其发现内核在内核态的什么地方花费时间。监管器确定内核的“热点”(hotspot)--执行最频繁的内核代码片段。确定内核“热点”是非常重要的，因为这可以指出应当进一步优化的内核函数。

  监管器基于非常简单的蒙特卡洛算法：在每次时钟中断发生时，内核确定该中断是否发生在内核态：如果是，内核从堆栈取回中断发生前的eip寄存器的值，并用这个值揭示中断发生前内核正在做什么。最后，采样数据积聚在“热点”上。

  profile_tick()函数为代码监管器采集数据。这个函数在单处理器系统上是由do_timer_interrupt()调用的（即全局时钟中断处理程序调用的），在多处理器系统上是由smp_local_timer_interrupt()函数调用的（即本地时钟中断处理程序调用的）

  为了激活代码监管器，在Linux内核启动时必须传递字符串参数"profile=N" ,这里2的N次方，表示要监管的代码段的大小。采集的数据可以从/proc/profile文件中读取。可以通过修改这个文件来重置计数器；在多处理器系统上，修改这个文件还可以改变抽样频率。不过，内核开发者并不直接访问/proc/profile文件，而是用readprofile系统命令

  Linux2.6内核还包含了另一个监管器，叫做oprofile .比起readprofile,oprofile除了更灵活、更可定制外，还能用于发现内核代码、用户态应用程序以及系统库中的热点。当使用oprofile时，profile_tick()调用timer_notify()函数来收集这个新监管器所使用的数据。

  [4]检查非屏蔽中断(NMI)监视器 

  在多处理器系统上，Linux为内核开发者还提供了另外一种功能：看门狗系统 （watchdogsystem），这对于探测引起系统冻结的内核bug可能相当有用。为了激活这样的看门狗，必须在内核启动时传递nmi_watchdog参数 

  看门狗基于本地和I/O APIC一个巧妙的硬件特性：它们能在每个CPU上产生周期性的NMI中断。因为NMI中断是不能用汇编语言指令cli屏蔽的，所以，即使禁止中断，看门狗也能检测到死锁

  因而，一旦每个时钟节拍到来，所有的CPU，不管其正在做什么，都开始执行NMI中断处理程序；该中断处理程序又调用do_nmi()。这个函数获得CPU的逻辑号n，然后检查irq_stat数组第n项的apic_timer_irqs字段。如果该CPU字段工作正常，那么，第n项的值必定不同于在前一个NMI中断中读出的值。当CPU正常运行时，第n项的apic_timer_irq字段就会被本地时钟中断处理程序增加，如果计数器没有被增加，说明本地时钟中断处理程序在整个时钟节拍期间根本就没有被执行。

  当NMI中断处理程序检测到一个CPU冻结时，就会敲响所有的钟，它把引起恐慌的信息记录在系统日志文件中，转储该CPU寄存器的内容和内核栈的内容，最后杀死当前进程。这就为内核开发者提供了发现错误的机会 

  5、软定时器和延迟函数 

  定时器是一种软件功能，即允许在将来的某个时刻，函数在给定的时间间隔用完时被调用 。

  超时（time-out）表示与定时器相关的时间间隔已经用完的那个时刻

  相对来说，实现一个定时器并不难。每个定时器都包含一个字段，表示定时器将需要多长时间才能到期。这个字段的初值就是jiffies的当前值加上合适的节拍数。这个字段的值不再改变。每当内核检查定时器时，就把这个到期字段值和当前这一刻jiffies的值相比较，当jiffies大于或等于这个字段存放的值时，定时器到期。

  Linux考虑两种类型的定时器， 即动态定时器 (dynamic timer)和间隔定时器 (internal timer).第一种类型由内核使用，而间隔定时器可以由进程的用户态创建

  这里是有关Linux定时器的警告：因为对定时器函数的检查总是由可延迟函数进行，而可延迟函数被激活以后很长时间才能被执行，因此，内核不能确保定时器函数正好在定时期间开始执行，而只能保证在适当的时间执行它们，或者假定延迟到几百毫秒之后执行它们。因此，对于必须严格遵守定时时间的那些实时应用而言，定时器并不适合。

  除了软定时器外，内核还使用了延迟函数，它执行一个紧凑的指令循环直到指令的时间间隔用完 

  [1]动态定时器

  (1)动态定时器与竞争条件

  (2)动态定时器的数据结构

  (3)动态定时器处理

  (4)动态定时器应用之一:nanosleep()系统调用

  [2]延迟函数

  当内核需要等待一个较短的时间间隔--比方说，不超过几毫秒时，就无需使用软定时器。

  在这种情况下，内核使用udelay()和ndelay()函数：前者接收一个微妙级的时间间隔作为它的参数，并在指定的延迟结束后返回；后者与前者类似，但是指定延迟的参数是纳秒级的

  6、与定时测量相关的系统调用 

  [1]time()和gettimeofday()系统调用 

  time()系统调用被gettimeofday()取代，但是，为了保持向后兼容，Linux中还包含它们。另一个被广泛使用的函数ftime()不再作为一个系统调用执行，它返回从1970年1月1日午夜(UTC)开始所走过的秒数与前1秒内锁走过的毫秒数。

  do_gettimeofday()动作：参考：p253

  拥有root权限的用户态下的进程可以用stime()和settimeofday()中任意一种系统调用来修改当前日期和时间

  [2]adjtimex()系统调用

  尽管时钟的走动确保了所有的系统最终都会从恰当的时间离开，但是，突然改变时间既是一种管理的失误也是一种危险的行为。

  调整互连PC的时钟以使所存取文件的inode中的时间标记值都保持一致


  [3]settimer()和alerm()系统调用 

  Linux允许用户态的进程激活一种叫做间隔定时器的特殊定时器，这种定时器引起了Unix信号被周期性地发送到进程。也可能激活一个间隔定时器以便在指定的延时后它仅发送一个信号。因此，间隔定时器由以下两个方面来刻画。

  发送信号所必需的频率，或者如果只需要产生一个信号，则频率为空

  在下一个信号被产生以前所剩余的时间


  7、与POSIX定时器相关的系统调用 

  POSIX1003.1b标准为用户态程序引入一种新型软定时器，尤其是针对多线程和实时应用程序。这些定时器被称作POSIX定时器。

  要执行每个POSIX定时器，必须向用户态程序提供一些POSIX时钟，也就是说，虚拟时间预定义了分辨率和属性。只要应用程序想使用POSIX定时器，它就创建一个新的定时器资源并指定一个现存的POSIX时钟来作为定时基准。参考表：6-3



  POSIX定时器比传统间隔定时器更灵活、更可靠。它们之间有两个显著区别 ：

  当传统间隔定时器到期时，内核会发送一个SIGALRM信号给进程来激活定时器。而当一个POSIX定时器到期时，内核可以发送各种信号给整个线程应用程序，也可以发送给单个指定的线程。内核还能在应用程序的某个线程上强制执行一个通告器函数，或者甚至什么也不做

  如果一个传统间隔定时器到期了很多次但用户态进程不能接收SIGALRM信号（例如由于信号被阻塞或者进程不处于运行态），那么只有第一个信号被接收到，其他所有SIGALRM信号都丢失了。对于POSIX定时器来说会发生同样的情况，但进程可以调用timer_getoverrun()系统调用来得到自第一个信号产生以来定时器到期的次数





