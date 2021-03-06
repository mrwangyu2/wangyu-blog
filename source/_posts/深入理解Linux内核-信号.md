---
title: 深入理解Linux内核-信号
date: 2012-10-22 09:24:10
tags:
- linux
---


由 王宇 原创并发布 ：



第十一章信号 

信号用于在用户态进程间通信。内核也用信号通知进程系统所发生的事情。 

1、信号的作用 

信号(signal)是很短的消息，可以被发送到一个进程或一组进程。发送给进程的唯一信息通常是一个数，以此来标识信号。

使用信号的两个主要目的 ：

让进程知道已经发生了一个特定的事件。

强迫进程执行它自己代码中的信号处理程序。 

当然，这两个目的不是互斥的，因为进程经常通过执行一个特定的例程来对某一事件作出反应。

常规信号： 前31个

实时信号： 32-64

实时信号与常规信号有很大的不同，因为它们必须排队以便发送的多个信号能被接收 到。另一方面，同种类型的常规信号并不排队 ；如果一个常规信号被连续发送多次，那么，只有其中的一个发送到接收进程。尽管Linux内核并不使用实时信号，它还是通过几个特定的系统调用完全实现了POSIX标准。

许多系统调用允许程序员发送信号，并决定他们的进程如何响应所接收的信号。

信号的一个重要特点 是它们可以随时被发送给状态经常不可预知的进程 。发送给非运行进程的信号必须由内核保存，直到进程恢复执行。

阻塞一个信号要求信号的传递拖延，直到随后解除阻塞，这使得信号产生一段时间之后才能对其传递这一问题变得更加严重。

内核区分信号传递的两个不同阶段 ：

信号产生 ：

内核更新目标进程的数据结构以表示一个新信号已被发送。

信号传递 ：

内核强迫目标进程通过以下方式对信号做出反应：或改变目标进程的执行状态，或开始执行一个特定的信号处理程序 ，或两者都是。

每个所产生的信号至多被传递一次。信号是可消费资源：一旦它们已传递出去，进程描述符中有关这个信号的所有信息都被取消。

已经产生但还没有传递的信号称为挂起信号(pendingsignal)。任何时候，一个进程仅存在给定类型的一个挂起信号，同一进程同种类型的其他信号不被排队，只被简单地丢弃。但是，实时信号是不同的：同种类型的挂起信号可以有好几个。

信号可以保留不可预知的挂起时间，必须考虑的因素：

信号通常只被当前正运行的进程传递

给定类型的信号可以由进程选择性地阻塞

当进程执行一个信号处理程序的函数时，通常“屏蔽”相应的信号，即自动阻塞这个信号到处理程序结束。因此，所处理的信号的另一次出现不能中断信号处理程序，所以，信号处理函数不必是可重入的 。

内核实现：

记住每个进程阻塞哪些信号

当从内核态切换到用户态时，对任何一个进程都要检查是否有一个信号已到达。这几乎在每个定时中断时都发生

确定是否可以忽略信号。这个发生在下列所有的条件都满足时：

目标进程没有被另一个进程跟踪

信号没有被目标进程阻塞

信号被目标进程忽略

处理这样的信号，即信号可能在进程运行期间的任一时刻请求把进程切换到一个信号处理函数，并在这个函数返回以后恢复原来执行的上下文。    

[1]传递信号之前所执行的操作 

进程以三种方式对一个信号做出应答： 

（1）显示地忽略信号 

（2）执行与信号相关的缺省操作 。由内核预定义的缺省操作取决于信号的类型，下列类型：

Terminate:进程被终止（杀死）

Dump:进程被终止(杀死)

Ignore:信号被忽略

Stop:进程被停止，即把进程置为TASK_STOPPED状态

Continue:如果进程被停止，就把它置为TASK_RUNNING状态

（3）通过调用相应的信号处理函数捕获信号 

注意，被对一个信号的阻塞和忽略是不同的：只要信号被阻塞，它就不被传递；只有在信号解除阻塞后才传递它。而一个被忽略的信号总是被传递，只是没有进一步的操作。 

SIGKILL和SIGSTOP信号不可以被显示地忽略、捕获或阻塞，因此，通常必须执行它们的缺省操作。因此，SIGKILL和SIGSTOP允许具有适当特权的用户分别终止并停止任何进程，不管进程执行时采取怎样的防御措施。

如果信号的传递会引起内核杀死一个进程，难么这个信号对该进程就是致命的。SIGKILL信号总是致命的；而且，缺省操作为Terminate的每个信号，以及不被进程捕获的信号对该进程也是致命的。注意，如果一个被进程所捕获的信号，其对应的信号处理函数终止了这个进程，那么这个信号就不是致命的，因为进程自己选择了终止，而不是被内核杀死。

[2]POSIX信号和多线程应用 

POSIX1003.1标准对多线程应用的信号处理有一些严格的要求 ：

信号处理程序必须在多线程应用的所有线程之间共享；不过，每个线程必须有自己的挂起信号掩码和阻塞信号掩码。

POSIX库函数kill()和sigqueue()必须向所有的多线程应用而不是某个特殊的线程发送信号。所有由内核产生的信号同样如此。

每个发送给多线程应用的信号仅传送给一个线程，这个线程是由内核在从不会阻塞该信号的线程中随意选择出来的

如果向多线程应用发送了一个致命的信号，那么内核将杀死该应用的所有线程，而不仅仅是杀死接收信号的那个线程。

Linux内核2.6把多线程应用实现为一组属于同一个线程组的轻量级进程。

如果一个挂起信号被发送给了某个特定进程，那么这个信号是私有的；如果被发送给了整个线程组，它就是共享的

[3]与信号相关的数据结构 

对系统中的每个进程来说，内核必须跟踪什么信号当前正在挂起或被屏蔽，以及每个线程组是如何处理所有信号的。为了完成这些操作，内核使用几个处理器描述符可存取的数据结构：参考图11-1***



(1)信号描述符和信号处理程序描述符 

进程描述符signal字段指向信号描述符(signaldescriptor)--一个signal_struct 类型的结构，用来跟踪共享挂起信号。

除了信号描述符以外，每个进程还引用一个信号处理程序描述符(signal handler deseriplor),它是一个sighand_struct 类型的结构，用来描述每个信号必须怎样被线程组处理

(2)sigaction数据结构

一些体系结构把特性赋给仅对内核可见的信号。因此，信号的特性存放在k_sigaction结构中，k_sigaciton结构既包含对用户态进程所隐藏的特性，也包含大家熟悉的sigaction结构，该结构保存了用户态进程能看见的所有特性。实际上，在80x86平台上，信号的所有特性对用户态的进程都是可见的。因此,k_sigaction结构只不过简化为类型为sigaction的单个sa结构。字段：

sa_handler:指定要执行操作的类型。它的值可以是指向信号处理程序的一个指针,SIG_DFL（即值0，指定执行缺省操作），或者SIG_IGN（即值1，指定忽略信号）

sa_flags:是一个标志集，指定必须怎样处理信号。

sa_mask:类型为sigset_t的变量，指定当运行信号处理程序时要屏蔽的信号

(3)挂起信号队列 

有几个系统调用能产生发送给整个线程组的信号，如kill()和rt_sigqueueinfo() ，而其他的一些则产生发送给特定进程的信号，如tkill()和tgkill()

  为了跟踪当前的挂起信号是什么，内核把两个挂起信号队列与每个进程相关联：

  共享挂起信号队列，它位于信号描述符的shared_pending字段，存放整个线程组的挂起信号

  私有挂起信号队列，它位于进程描述符的pending字段，存放特定进程的挂起信号

  [4]在信号数据结构上的操作 ：参考p429-430的函数列表 



  2、产生信号 

  很多内核函数都会产生信号：它们完成信号处理第一步的工作，即根据需要更新一个或多个进程的描述符。 它们不直接执行第二步的信号传递操作，而是可能根据信号的类型和目标进程的状态唤醒一些进程，并促使这些进程接收信号。

  当发送给进程一个信号时，这个信号可能来自内核，也可能来自另一个进程。内核通过对如表11-9所示的某个函数进行调用而产生信号

  当一个信号被发往整个线程组时，这个信号可能来自内核，也可能来自另一个进程。内核通过对如表11-10所示的某个函数进行调用而产生信号

  [1]specific_send_sig_info()函数：向指定进程发送信号，步骤：参考p433

  [2]send_signal()函数:在挂起信号队列中插入一个新元素,步骤：参考p434

  [3]group_send_sig_info()函数:向整个线程组发送信号，步骤：参考p435-437 



  3、传递信号 

  为确保进程的挂起信号得到处理内核所执行的操作。

  内核在允许进程恢复用户态下的执行之前，检查进程TIF_SIGPENDING标志的值。每当内核处理完一个中断或异常时，就检查是否存在挂起信号 

  为了处理非阻塞的挂起信号，内核调用do_signal()函数

  通常只是在CPU要返回到用户态时才调用do_signal()函数

  do_signal()函数的核心由重复调用dequeue_signal()函数的循环组成，直到在私有挂起信号队列和共享挂起信号队列中都没有非阻塞的挂起信号时，循环才结束。

  dequeue_singal()函数首先考虑私有挂起信号队列中的所有信号，并从最低编号的挂起信号开始。然后考虑共享队列中的信号。它更新数据结构以表示信号不再是挂起的，并返回它的编号。

  do_signal()函数如何处理每一个挂起的信号，其编号由dequeue_signal()返回。首先，它检查current接收进程是否正受到其他一些进程的监控；在肯定的情况下，do_signal()调用do_notify_parent_cldstop()和schedule()让监控进程知道进程的信号处理。

  然后，do_signal()把要处理信号的k_sigaction数据结构的地址赋给局部变量ka;根据ka的内容可以执行三种操作：忽略信号、执行缺省操作或执行信号处理程序。如果显式忽略被传递的信号，那么do_signal()函数仅仅继续执行循环，并由此考虑另一个挂起信号

  [1]执行信号的缺省操作 

  如果ka->sa.sa_handler等于SIG_DFL,do_signal()就必须执行信号的缺省操作。唯一的例外是当接收进程是init时，这个信号被丢弃。

  SIGSTOP与其他信号的差异比较微妙：SIGSTOP总是停止线程组，而其他信号只停止不在“孤儿进程组”中的线程组。POSIX标准规定，只要进程组中一个进程有父进程，尽管进程处于不同的进程组中但在同一个会话中，那么这个进程组就不是孤儿。因此，如果父进程死亡，但启动该进程的用户并登录在线，那么该进程组就不是一个孤儿。

  缺省操作为Dump的信号可以在进程的工作目录中创建一个“转储”文件，这个文件列出进程地址空间和CPU寄存器的全部内容

  [2]捕获信号 

  如果信号有一个专门的处理程序，do_signal()就函数必须强迫该处理程序执行。这是通过调用handle_signal()进行的 

  注意do_signal()的处理了一个单独的信号后怎样返回。直到下一次调用do_signal()时才考虑其他挂起的信号。这种方式确保了实时信号将以适当的顺序得到处理

  执行一个信号处理程序是件相当复杂的任务，因此在用户态和内核态之间切换时需要谨慎地处理栈中的内容 。我们将正确地解释这里所承担的任务

  信号处理程序是用户态进程所定义的函数，并包含在用户态的代码段中。handle_signal()函数运行在内核态，而信号处理程序运行在用户态，这就意味着在当前进程恢复“正常”执行之前，它必须首先执行用户态的信号处理程序。此外，当内核打算恢复进程的正常执行时，内核态堆栈不再包含被中断程序的硬件上下文，因此每当从内核态向用户态转换时，内核态堆栈都被清空。而另外一个复杂性是因为信号处理程序可以调用系统调用，在这种情况下，执行了系统调用的服务例程以后，控制权必须返回到信号处理程序而不是到被中断程序的正常代码流。

  linux所采用的解决方法是把保存在内核态堆栈中的硬件上下文拷贝到当前进程的用户态堆栈中。用户态堆栈也以这样的方式被修改，即当信号处理程序终止时，自动调用sigreturn()系统调用把这个硬件上下文拷贝回到内核态堆栈中，并恢复用户态堆栈中原来的内容。

  图11-2说明了有关捕捉一个信号的函数的执行流：

  一个非阻塞的信号发送给一个进程。当中断或异常发生时，进程切换到内核态。

  正要返回到用户态前，内核执行do_signal()函数，

这个函数又依次处理信号（通过调用handle_signal()）和建立用户态堆栈(通过调用setup_frame()或setup_rt_frame())

  当进程又切换到用户态时，因为信号处理程序的起始地址被强制放进程序计数器中，因此开始执行信号处理程序。

  当处理程序终止时，setup_frame()或setup_rt_frame()函数放在用户态堆栈中的返回代码就被执行。这个代码调用sigreturn()或rt_sigrenturn()系统调用，相应的服务例程把正常程序的用户态堆栈硬件上下文拷贝到内核堆栈，并把用户态堆栈恢复到它原来的状态(通过调用restore_sigcongtext()).当这个系统调用结束时，普通进程就因此能恢复自己的执行

  图：11-2***





  (1)建立帧 

为了适当地建立进程的用户态堆栈，handle_signal()函数或者调用setup_frame()或者调用setup_rt_frame()

  setup_frame()函数把一个叫做帧(frame)的数据结构推进用户态堆栈中，这个帧含有处理信号所需要的信息，并确保正确返回到handle_signal()函数

  setup_frame()函数把保存在内核态堆栈的段寄存器内容重新设置成它们的缺省值以后才结束。现在，信号处理程序所有需的信息就在用户态堆栈的顶部。

  (2)检查信号标志 

  建立了用户态堆栈以后,handle_signal()函数检查与信号相关的标志值。如果信号没有设置SA_NODEFER标志，在sigaction表中sa_make字段对应的信号就必须在信号处理程序执行期间被阻塞，然后，handle_signal()返回到do_signal()，do_signal()也立即返回

  (3)开始执行信号处理程序 

  do_signal()返回时，当前进程恢复它在用户态的执行。由于如前所述setup_frame()的准备，eip寄存器指向信号处理程序的第一条指令，而esp指向已推进用户态堆栈顶的帧的第一个内存单元。因此，信号处理程序被执行。

  (4)终止信号处理程序 

  信号处理程序结束时，返回栈顶地址，该地址指向帧的pretcode字段所引用的vsyscall页中的代码。因此，信号编号(即帧的sig字段)被从栈中丢弃，然后调用sigreturn()系统调用

  sys_rt_sigreturn()服务例程把来自扩展帧的进程硬件上下文拷贝到内核态堆栈，并通过从用户态堆栈删除扩展帧以恢复用户态堆栈原来的内容。

  (5)系统调用的重新执行 

  内核并不总是能立即满足系统调用发出的请求，在这种情况发生时，把发出系统调用的进程置为TASK_INTERRUPTIBLE或TASK_UNINTERRUPTIBLE状态

  如果进程处于TASK_INTERRUPTIBLE状态，并且某个进程向它发送了一个信号，那么，内核不完成系统调用就把进程置成TASK_RUNNING状态。当切换回用户态时信号被传递给进程。当这种情况发生时，系统调用服务例程没有完成它的工作，但返回EINTR,ERESTARTNOHAND,ERESTART_RESTARTBLOCK,ERESTARTSYS或ERESTARTNOINTR错误码。实际上，这种情况下用户态进程获得的唯一错误码是EINTR,这个错误码表示系统调用还没有执行完。内核内部使用剩余的错误码来指定信号处理程序结束后是否自动重新执行系统调用。

  与未完成的系统调用相关的出错码及这些出错码对信号三种可能的操作产生的影响。

  Terminate:不会自动重新执行系统调用

  Reexecut:内核强迫用户态进程把系统调用号重新装入eax寄存器，并重新执行int$0x80指令或sysenter指令。进程意识不到这种重新执行，因此出错码也不传递给进程。

  Depends:只有被传递信号的SA_RESTART标志被设置，才重新执行系统调用；否则系统调用-EINTER出错码结束

  当传递信号时，内核在试图重新执行一个系统调用前必须确定进程确实发出过这个系统调用。这就是regs硬件上下文的orig_eaz字段起重要作用之处    

  a、重新执行被未捕获信号中断的系统调用

  如果信号被显式地忽略，或者如果它的缺省操作已被强制执行，do_signal()就分析系统调用的出错码，并如表11-11中所说明的那样决定是否重新自动执行未完成的系统调用。如果必须重新开始执行系统调用，那么do_signal()就修改regs硬件上下文，以便在进程返回到用户态时，eip指向int$0x80指令或sysenter指令，且eax包含系统调用号

  b、为所捕获的信号重新执行系统调用

  如果信号被捕获，那么handle_signal()分析出错码，也可能分析sigaction表的SA_RESTART标志来决定是否必须重新执行未完成的系统调用

  如果系统调用必须被重新开始执行，handle_signal()就与do_signal()完全一样地继续执行；否则，它向用户态进程返回一个出错码-ENTR

  4、与信号处理相关的系统调用 

  在用户态运行的进程可以发送和接收信号。这意味着必须定义一组系统调用用来完成这些操作。遗憾的是，由于历史的原因，已经存在几个具有相同功能的系统调用，因此，其中一些系统调用从未被调用。例如：系统调用sys_sigaction()和sys_rt_sigaciton()几乎是相同的，因此C库中封装函数sigaction()调用sys_rt_sigaction()而不是sys_sigaction()。

  [1]kill()系统调用 

  一般用kill(pid,sig) 系统调用向普通进程或多线程应用发送信号，其相应的服务例程是sys_kill()函数

  kill()系统调用能发送任何信号，即使编号在32-64之间的实时信号。kill()系统调用不能确保把一个新的元素加入到目标进程的挂起信号队列，因此，挂起信号的多个实例可能被丢失。实时信号应该当通过rt_siggueueinfo()系统调用进行发送

  [2]tkill和gkill()系统调用 

  tkill()和tgkill() 系统调用向线程组中的指定进程发送信号

  [3]改变信号的操作 

  sigaction(sig,act,oact) 系统调用允许用户为信号指定一个操作。当然，如果没有自定义的信号操作，那么内核执行与传递的信号相关的缺省操作

  [4]检查挂起的阻塞信号 

  sigpending( )系统调用允许进程检查挂起的阻塞信号的集合，也就是说，检查信号被阻塞时已产生的那些信号

  [5]修改阻塞信号的集合 

  sigprocmask() 系统调用允许进程修改阻塞信号的集合。这个系统调用只应用于常规信号

  [6]挂起进程 

  sigsuspend() 系统调用把进程置为TASK_INTERRUPTIBLE状态，当然这是把mask参数指向的位掩码数组所指定的标准信号阻塞以后设置的。只有当一个非忽略、非阻塞的信号发送到进程以后，进程才被唤醒

  [7]实时信号的系统调用 

  系统调用只应用到标准信号，因此，必须引入另外的系统调用来允许用户态进程处理实时信号

实时信号的几个系统调用：rt_sigaction()rt_sigpending()rt_sigprocmask()rt_sigsuspend()



