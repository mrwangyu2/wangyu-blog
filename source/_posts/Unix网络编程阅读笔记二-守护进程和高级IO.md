---
title: Unix网络编程阅读笔记二 守护进程和高级IO
date: 2015-03-05 09:17:35
tags:
- linux
---

由 王宇 原创并发布
本文代码，在以下环境下编译通过
CentOS 6.4
Kernal version: 2.6.32
GCC version: 4.4.7
一、Dasmon 进程和 'inetd' 超级服务
daemon 是后台运行的程序，一般随系统的启动自动地启动且在用户logoff后仍然能够继续运行。该daemon进程一般在启动后需要与父进程断开关系，并使进程没有控制终端（tty）。因为daemon程序在后台执行，不需要于终端交互，通常关闭STDIN、STDOUT和STDER。daemon无法输出信息，可以使用syslog或自己的日志系统进行日志处理。

1、daemon_init函数：
#include "unp.h"
#include <syslog.h>
#define MAXFD 64
extern int daemon_proc; /* defined in error.c */
  int
daemon_init(const char *pname, int facility)
{
  int i;
  pid_t pid;
  if ( (pid = Fork()) < 0)
    return (-1);
  else if (pid)
    _exit(0); /* parent terminates */
  /* child 1 continues... */
  if (setsid() < 0) /* become session leader */
    return (-1);
  Signal(SIGHUP, SIG_IGN);
  if ( (pid = Fork()) < 0)
    return (-1);
  else if (pid)
    _exit(0); /* child 1 terminates */
  /* child 2 continues... */
  daemon_proc = 1; /* for err_XXX() functions */
  chdir("/"); /* change working directory */
  /* close off file descriptors */
  for (i = 0; i < MAXFD; i++)
    close(i);
  /* redirect stdin, stdout, and stderr to /dev/null */
  open("/dev/null", O_RDONLY);
  open("/dev/null", O_RDWR);
  open("/dev/null", O_RDWR);

  openlog(pname, LOG_PID, facility);

  return (0);  /* sucess */
}

fork
首先调用fork，然后终止父进程，留下子进程继续运行。

setsid
setsid 是一个POSIX函数，用于创建一个新的会话(session).当前进程变为新会话的会话头进程以及新进程组的进程组头进程，从而不再有控制终端。

忽略SIGHUP信号并再次fork
再次fork函数返回时，父进程实际上是上一次调用fork产生的子进程，它被终止掉，留下新的子进程继续运行。再次fork的目的是确保本守护进程将来即使打开了一个终端设备，也不会自动获得控制终端。当没有控制终端的一个会话头进程打开一个终端设备时，该终端自动成为这个会话头进程的控制终端。然而再次调用fork之后，我们确保新的子进程不再是一个会话头进程，从而不能自动获得一个控制终端。
这里必须忽略SIGHUP信号，因为当会话头进程终止时，其会话中的所有进程都收到SIGHUG信号。

为错误处理函数设置标识
把全局变量daemon_proc置为非0值，这个外部变量由我们的err_XXX函数定义，其值非0是在告知它们改为调用syslog, 以取代fprintf到标准错误输出。

改变工作目录
把工作目录改到根目录，不过有些守护进程另有原因需改到其个目录。

关闭所有打开的描述符
关闭本守护进程从执行它的进程继承来的所有打开着的描述符。问题是怎样检测正在使用的最大描述符。我们的解决办法是干脆关闭前64个描述符。

将stdin stdout和stderr重定向到/dev/null
打开/dev/null作为本守护进程的标准输入、标准输出、标准错误输出。这一点保证这些常用描述符是打开的，针对它们的read系统调用换回0(EOF),write系统调用则由内核丢弃所写数据。

使用syslogd处理错误
调用openlog。Unix系统通常通过一个系统初始化脚本来启动一个叫做ssylogd的 daemon. 我们能够发送log 信息给它。

2、inetd 守护进程
  子进程为客户提供服务，父进程则继续等待下一个客户请求。这个模型存在两个问题：
(1) 所有这些守护进程含有几乎相同的启动代码，既表现在创建套接字上，也表现在演变成守护进程上(类似我们的daemon_init函数)
  (2) 每个守护进程在进程表中占据一个表项，然而它们大部分时间处于睡眠状态。
  inetd守护进程使上述问题得到简化，它是这样解决上述两个问题：
  通过由inetd处理普通守护进程的大部分启动细节以简化守护程序的编写。这么一来每个服务器不再有调用daemon_init函数的必要。
  单个进程(inetd)就能为多个服务等待外来的客户请求，以此取代每个服务一个进程的做法。这么做减少了系统中的进程总数。
  inetd 通常是读取/etc/inetd.conf 来指定本超级服务器处理那些服务以及当一个服务请求到达时该怎么做。
  小结
  许多Unix服务器由inetd守护进程启动。它处理全部守护进程化所需的步骤，当启动真正的服务器时，套接字已在标准输入、标准输出和标准错误输出上打开。这样我们无需调用soket bind listen accept 因为这些步骤已由inetd处理。

  inetd守护进程的工作流程（如下图所示）




  1． 在启动阶段，读入配置文件(/etc/inetd.conf /etc/xinetd.conf)，对于配置文件中的每个服务创建一个适当类型（TCP或UDP）的套接口。新创建的每个套接口都被加入到将由某个select调用使用的一个描述字集中。、 xinetd(inetd)的配置文件中包含服务的类型（如上例中的stream、tcp），服务模式（如上例中的nowait），服务程序（如上例中的/root/echo）以及访问控制信息、日志等信息。
  2． 为每个套接口调用bind（根据/etc/services中的配置项）。
  3． 对于每个TCP套接口，调用listen以接受外来的连接请求；
  4． 创建完毕所有套接口后，调用select等待其中任何一个套接口变为可读。inetd的大部分时间阻塞于select调用内部，等待某个套接口变为可读。
  5． 当select返回指出某个套接口可读以后，如果该套接口是TCP套接口，而且其服务器为nowait类型，则调用accept接受这个连接。
  6． inetd调用fork派生进程，并由子进程处理服务请求。 l 子进程关闭要处理的套接口描述字之外的所有描述字（对于TCP为accept返回的套接口，对于UDP为最初创建的套接口），子进程三次调用dup2，把待处理套接口的描述字复制到描述字0、1、2上；然后关闭原套接口描述字。因此，子进程打开的描述字只有0、1、2。子进程从标准输入读，相当于从所处理的套接口读；子进程往标准输出或标准错误上写，相当于往所处理套接口写。 l 子进程根据login-name(user)的配置值，如果不是root，子进程则调用setgid和setuid把自身改为指定的用户。 l 子进程调用exec执行由配置文件指定的程序（ 如上例中的/root/echo）来具体处理请求。
  7． 如果5中返回的是TCP套接口，则父进程先关闭接受请求产生的连接套接口。父进程在此调用select，等待下一个变为可读的套接口。
  二、高级I/O函数
  1、Socket Timeouts
  Socket的I/O操作上设置超时的方法有以下3种：
  (1) 调用alarm, 它的指定超时期满时产生SIGALRM信号。这个方法涉及信号处理，而信号处理在不同的实现上存在差异，而且可能干涉进程中现有的alarm调用。
  (2) 在select中阻塞等待I/O（select有内置的时间限制），以此代替直接阻塞在read或write调用上
  (3) 使用较新的SO_RCVTIMEO和SO_SNDTIMEO套接字选项。这个方法的问题在于并非所有实现都支持这两个套接字选项。
  2、recv和send函数
#include <sys/socket.h>
  ssize_t recv(int sockfd,void *buff,size_tnbytes,int flags) ;
  ssize_t send(int sockfd,const void *buff,size_tnbytes,int flags) ;
  //Both return: number of bytes read or written if OK, –1 on error



  recv和send的前3个参数等同于read和write的3个参数。flags参数，参考下表
  表

  3、readv和writev函数
  readv和writev允许单个系统调用读入到或写出自一个或多个缓冲区。

#include <sys/uio.h>
  ssize_t readv(int filedes,const struct iovec *iov,int iovcnt) ;
  ssize_t writev(int filedes,const struct iovec *iov,int iovcnt) ;
  //Both return: number of bytes read or written, –1 on error



  4、recvmsg和sendmsg函数
  这两个函数是最通用的I/O函数。实际上我们可以把所有read、readv、recv和recvfrom调用替换成recvmsg调用。类似地，各种输出函数调用也可以替换成sendmsg调用。

#include <sys/socket.h>
  ssize_t recvmsg(int sockfd,struct msghdr *msg,int flags) ;
  ssize_t sendmsg(int sockfd,struct msghdr *msg,int flags) ;
  //Both return: number of bytes read or written if OK, –1 on error



  msghdr结构体：

  struct msghdr {
    void *msg_name; /* protocol address */
    socklen_t msg_namelen; /* size of protocol address */
    struct iovec *msg_iov; /* scatter/gather array */
    int msg_iovlen; /* # elements in msg_iov */
    void *msg_control; /* ancillary data (cmsghdr struct) */
    socklen_t msg_controllen; /* length of ancillary data */
    int msg_flags; /* flags returned by recvmsg() */
  };



5、辅助数据(ancillary data)
  可以通过调用sendmsg和recvmsg这两个函数，使用msghdr结构中的msg_control和msg_controllen这两个成员发送和接收。辅助数据的另一个称谓是控制信息（control information）

  6、有多少数据在排队？
  有时候我们想要在不真正读取数据的前提下知道一个Socket上已有多少数据排队等着读取。有3个技术可用于获悉已排队的数据量：

  (1)使用非阻塞式I/O
  (2)如果我们即想查看数据，又想数据仍然留在接收队列中以供本进程其他部分稍后读取，那么可以使用MSG_PEEK标志
  (3)一些实现支持ioctl的FIONREAD命令。该命令的第三个ioctl参数是指向某个整数的一个指针，内核通过该整数返回的值就是Socket接收队列的当前字节数。
  7、Socket和标准I/O
  执行I/O的另一个方法是使用标准I/O函数库

  使用标准库需要考虑以下几个问题：
  通过调用fdopen,可以从任何一个描述符创建出一个标准I/O流。类似地，通过调用fileno，可以获取一个给定标准I/O流对应的描述符。
  TCP和UDP套接字是全双工的。标准I/O流也可以是全双工的：只要一r+类型打开流即可，r+意味着读写。然而在这样的流上，我们必须在调用一个输出函数之后插入一个fflush、fseek、fsetpos或rewind调用才能接着调用一个输入函数。
  解决上述读写问题的最简单方法是为一个给定套接字打开两个标准I/O流：一个用于读，一个用于写。
void str_echo(int sockfd)
{
  char line[MAXLINE];
  FILE *fpin, *fpout;
  fpin = Fdopen(sockfd, "r");
  fpout = Fdopen(sockfd, "w");
  while (Fgets(line, MAXLINE, fpin) != NULL)
    Fputs(line, fpout);
}

8、高级轮询技术
(1) /dev/poll接口
Solaris上名为/dev/poll的特殊文件提供了一个可扩展的轮询大量描述符的方法。select和poll存在的一个问题是，每次调用它们都得传递待查询的文件描述符。轮询设备能在调用之间维持状态，因此轮询进程可以预先设置好待查询描述符的列表，然后进入一个循环等待事件发生，每次循环回来时不必再次设置该列表

  (2) kqueue接口
本接口允许进程向内核注册描述所关注kqueue事件的是事件过滤器(event filter)

  三、非阻塞式I/O
  Socket的默认状态是阻塞的。这就意味着当发出一个不能立即完成的套接字调用时，其进程被投入睡眠，等待相应操作完成。可能阻塞的Socke调用可分为以下四类：

  (1) 输入操作，包括read、readv、recvfrom和recvmsg共5个函数
  (2) 输出操作，包括write、writev、send、sendto和sendmsg共5个函数
  (3) 接受外来连接，即accept函数
  (4) 发起外出连接，即用于TCP的connect函数
1、非阻塞读和写：str_cli函数(修订版)
  在基础篇中的str_cli，使用了select的版本仍使用阻塞式I/O。举例来说，如果在标准输入有一行文本可读，我们就调用read读入它，再调用write把它发送给服务器。然而如果Socket发送缓冲区已满，write调用将会阻塞。在进程阻塞与write调用期间，可能有来自套接字接收缓冲区的数据可供读取。

  我们维护着两个缓冲区：to容纳从标准输入到服务器去的数据，fr容纳自服务器到标准输出来的数据。 图16-1

  其中toiptr指针指向标准输入读入的数据可以存放的下一个字节。tooptr指向下一个必须写到套接字的字节。有（topiptr-tooptr）个字节需写到套接字。可从标准输入读入的字节数是（&to[MAXLINE]-toiptr）.一旦tooptr移动到toiptr，这两个指针就一起恢复到缓冲区开始处

void str_cli(FILE *fp, int socket_fd)
{
  int fcntl_return_value, stdin_eof, max_fd;
  char to[MAXLINE], fr[MAXLINE];
  char *toiptr, *tooptr, *friptr, *froptr;
  fd_set rset, wset;
  ssize_t nwritten, n;

  /* Set noblocking for socket */
  fcntl_return_value = fcntl(socket_fd, F_GETFL, 0);
  if(fcntl_return_value == -1)
  {
    perror("Error: get socket flag\n");
    return;     
  }

  fcntl_return_value = fcntl(socket_fd, F_SETFL, fcntl_return_value | O_NONBLOCK); /* 使用非阻塞标识：O_NONBLOCK*/

  if(fcntl_return_value == -1)
  {
    perror("Error: set socket noblocking flag\n");
    return;     
  }

  /* Set noblocking for standard I/O noblocking */
  fcntl_return_value = fcntl(STDIN_FILENO, F_GETFL, 0);
  if(fcntl_return_value == -1)
  {
    perror("Error: get standard in I/O flag\n");
    return;     
  }

  fcntl_return_value = fcntl(STDIN_FILENO, F_SETFL, fcntl_return_value | O_NONBLOCK);

  if(fcntl_return_value == -1)
  {
    perror("Error: set standard in I/O  noblocking flag\n");
    return;     
  }

  fcntl_return_value = fcntl(STDOUT_FILENO, F_GETFL, 0);
  if(fcntl_return_value == -1)
  {
    perror("Error: get standard out I/O flag\n");
    return;     
  }

  fcntl_return_value = fcntl(STDOUT_FILENO, F_SETFL, fcntl_return_value | O_NONBLOCK);

  if(fcntl_return_value == -1)
  {
    perror("Error: set standard out I/O  noblocking flag\n");
    return;     
  }

  /* Initilaize buffer pointers */
  toiptr = tooptr = to;
  friptr = froptr = fr;
  stdin_eof = 0;

  max_fd = max(max(STDIN_FILENO, STDOUT_FILENO), socket_fd) + 1;

  for(;;)
  {

    FD_ZERO(&rset);
    FD_ZERO(&wset);
    /* read from stdin  */
    if ( stdin_eof == 0 && toiptr < &to[MAXLINE] ) 
    {
      FD_SET(STDIN_FILENO, &rset);    
    }

    /* read from socket  */
    if ( friptr < &fr[MAXLINE] ) 
    {
      FD_SET(socket_fd, &rset);   
    }

    /* write data to socket */
    if ( tooptr != toiptr ) 
    {
      FD_SET(socket_fd, &wset);   
    }

    /* write data to standard out */
    if ( froptr != friptr ) 
    {
      FD_SET(STDOUT_FILENO, &wset);   
    }

    /* select to loop file descriptor for kernel preparing source. */
    select(max_fd, &rset, &wset, NULL, NULL);

    /* read stdin is readable. */
    if(FD_ISSET(STDIN_FILENO, &rset) != 0 ) 
    {
      if((n= read(STDIN_FILENO, toiptr, &to[MAXLINE] - toiptr)) < 0)
      {
        if ( errno != EWOULDBLOCK ) 
        {
          perror("Error: read error on stdin.\n");
        }
      } 
      else if ( n == 0)
      {
        printf("EOF on stdin.\n");
        stdin_eof = 1;

        if( tooptr == toiptr)
        {
          shutdown(socket_fd, SHUT_WR);   
        }
      }
      else
      {
        printf("Read %d bytes from stdin.\n", n);
        toiptr += n;
        FD_SET(socket_fd, &wset);
      }
    }

    /* read socket is readable. */
    if(FD_ISSET(socket_fd, &rset) !=0 )
    {
      if((n =read(socket_fd, friptr, &fr[MAXLINE] - friptr)) < 0)
      {
        if ( errno != EWOULDBLOCK ) 
        {
          perror("Error: read error on socket.\n");
        }
      }
      else if ( n == 0)
      {
        printf("EOF on socket\n");

        if(stdin_eof !=0)
        {
          return; 
        }
        else
        {
          printf("Error: server terminalated prematurely.");  
        }
      }
      else
      {
        printf("Read %d bytes from socket.\n", n);
        friptr += n;
        FD_SET(STDOUT_FILENO, &wset);
      }
    }

    /* write stdout is readable. */
    if(FD_ISSET(STDOUT_FILENO, &wset) && ((n = friptr - froptr) > 0)) 
    {
      if((nwritten = write(STDOUT_FILENO, froptr, n)) < 0)
      {
        if ( errno != EWOULDBLOCK ) 
        {
          perror("Error: write error on stdout.\n");
        }
      } 
      else
      {
        printf("Wrote %d bytes to stdout.\n", n);
        froptr += nwritten;

        if(froptr == friptr)
        {
          froptr = friptr = fr;
        }
      }
    }

    /* write socket is readable. */
    if(FD_ISSET(socket_fd, &wset) && ((n = toiptr - tooptr) > 0)) 
    {
      if((nwritten = write(socket_fd, tooptr, n)) < 0)
      {
        if ( errno != EWOULDBLOCK ) 
        {
          perror("Error: write error on socket.\n");
        }
      } 
      else
      {
        printf("Wrote %d bytes to socket.\n", n);
        tooptr += nwritten;

        if(tooptr == toiptr)
        {
          tooptr = toiptr = to;
        }
        if(stdin_eof !=0 )
        {
          shutdown(socket_fd, SHUT_WR);   
        }
      }
    }



  }

}



2、非阻塞connect
非阻塞的connect有三个用途：

(1)我们可以把三路握手叠加在其他处理上。完成一个connect要花一个RRT时间，而RRT波动范围很大，从局域网上的几个毫秒到几百个毫秒甚至是广域网上的几秒。这段时间内也许有我们想要执行的其他处理工作可执行。
(2)我们可以使用这个技术同时建立多个连接。这个用途已随着web浏览器变得流行起来
(3)既时使用select等待连接的建立，我们可以给select指定一个时间限制，使得我们能够缩短connect的超时
3 非阻塞accept
(1) 当使用select获悉某个监听套接字上何时有已完成连接准备好被accept时，总是把这个监听套接字设置为非阻塞
(2) 在后续的accept调用中忽略以下错误： EWOULDBLOCK ECONNABORTED EPROTO EINTR
四、ioctl操作
网络程序(特别是服务器程序)经常在程序启动执行后使用ioctl获取所在主机全部网络接口的信息，包括：接口地址、是否支持广播、是否支持多播

ioctl函数
#include <unistd.h>
int ioctl(intfd,intrequest,... /* void *arg*/ );
Returns:0 if OK, -1 on error
我们可以把和网络相关的请求(request)划分为6类：

套接字操作
文件操作
接口操作
ARP高速缓存操作
路由表操作
流系统




