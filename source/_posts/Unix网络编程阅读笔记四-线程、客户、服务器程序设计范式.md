---
title: Unix网络编程阅读笔记四 线程、客户、服务器程序设计范式
date: 2015-03-05 09:16:10
tags:
- linux
---

由 王宇 原创并发布
本文代码，在以下环境下编译通过
CentOS 6.4
Kernal version: 2.6.32
GCC version: 4.4.7
一、 线程
父进程accept一个连接，fork一个子进程，该子进程处理与该连接对端的客户之间的通信，这种范式多少年来一直用的挺好，fork调用却存在一些问题：

fork是昂贵的。
fork返回之后父子进程之间信息的传递需要进程间通信（IPC）机制。调用fork之前父进程向尚未存在的子进程传递信息相当容易，因为子进程将从父进程数据空间及所有描述符的一个副本开始运行。然而从子进程往父进程返回信息却比较费力。
线程有助于解决这两个问题。线程有时称为轻权进程（lightweight process）。同一进程内的所有线程共享相同的全局内存。这使得线程之间易于共享信息，然而伴随这种容易性而来的却是同步问题。

线程除了共享全局变量外还共享：

进程指令
  大多数数据
打开的文件(即描述符)
  信号处理函数和信号处置
  当前工作目录
  用户ID和组ID
  不过每个线程有各自的：

  线程ID
  寄存器集合，包括程序计数器和栈指针
  栈（用于存放局部变量和返回地址）
  errno
  信号掩码
  优先级
  1、基本线程函数：创建和终止
  pthread_create 函数 创建并启动一个线程
  pthread_join 函数 等待给定线程终止
  pthread_self 函数 获得线程自身线程ID
pthread_detach 函数 转变为脱离状态(detached)
  pthread_exit 函数 终止线程
  2、使用线程的str_cli函数
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<netinet/in.h>
#include<pthread.h>
#include<errno.h>

#define SERV_PORT 51000
#define MAXLINE 4096 
#define SA struct sockaddr

  char* Ip_address = "192.168.153.130";

  static int socket_fd;
  static FILE *fp;


size_t readline(int fd, void *vptr, size_t maxlen)
{
  size_t n, rc;
  char c, *ptr;

  ptr = vptr;

  for(n = 1; n < maxlen; n++)
  {
again:
    if( (rc = read(fd, &c, 1)) == 1)
    {
      *ptr++ = c;     
      if(c == '\n')
      {
        break;
      }
    }
    else if(rc == 0)
    {
      return(n - 1);
    }
    else
    {
      if(errno == EINTR)
      {
        goto again; 
      }     
      return (-1);
    }

  }

  return(n);
}

ssize_t writen(int fd, const void *vptr, size_t n)
{
  size_t nleft;
  ssize_t nwritten;
  const char *ptr;

  ptr = vptr;
  nleft = n;
  while(nleft > 0)
  {
    if((nwritten = write(fd, ptr, nleft)) <= 0)
    {
      if(nwritten < 0 && errno == EINTR)
      {
        nwritten = 0;
      }
      else
      {
        return(-1);
      }

    }

    nleft -= nwritten;
    ptr += nwritten;    
  }

  return(n);
}

void* copyto(void *arg)
{
  char sendline[MAXLINE];

  while(fgets(sendline, MAXLINE, fp) != NULL )
  {
    if(writen(socket_fd, sendline, strlen(sendline)) == -1)
    {
      perror("Error: write socket!\n");
    }

  }
  if(shutdown(socket_fd, SHUT_WR) == -1)
  {
    perror("Error: shutdown!\n");
  }

  return(NULL);
}


void str_cli(FILE *fp_arg, int socket_fd_arg)
{
  char recvline[MAXLINE];
  pthread_t tid;
  socket_fd = socket_fd_arg;
  fp = fp_arg;

  if(pthread_create(&tid, NULL, copyto, NULL) != 0)
  {
    perror("Error: pthread_create!\n");
  }

  while(readline(socket_fd, recvline, MAXLINE) > 0)
  {
    fputs(recvline, stdout);
  }
}


int main()
{
  int socket_fd, connect_rt;
  struct sockaddr_in servaddr;
  char err_message[MAXLINE];

  socket_fd = socket(AF_INET, SOCK_STREAM, 0);

  if(socket_fd == -1)
  {
    printf("Error: created socket!\n");
    exit(1);
  }

  bzero(&servaddr, sizeof(servaddr));

  servaddr.sin_family = AF_INET;
  servaddr.sin_port = htons(SERV_PORT);
  inet_pton(AF_INET, Ip_address, &servaddr.sin_addr);

  connect_rt = connect(socket_fd, (SA *)&servaddr, sizeof(servaddr));

  if(connect_rt != 0)
  {
    perror(err_message);    
    printf("Error: connect socket!\n");
    printf("%s\n", err_message);
    exit(1);
  }

  str_cli(stdin, socket_fd);


  exit(0);
}



3、使用线程的TCP回射服务程序
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<unistd.h>
#include<netinet/in.h>
#include<errno.h>
#include<pthread.h>

#define SA struct sockaddr
#define SERV_PORT 51000
#define MAXLINE 4096 
#define LISTENQ 1024


void str_echo(int socket_fd)
{
  ssize_t n;
  char buf[MAXLINE];

again:
  while( (n = read(socket_fd, buf, sizeof(buf))) > 0)
  {
    if(write(socket_fd, buf, n) == -1 )
    {
      perror("Error:write.\n");
    }
  }

  if( n < 0 && errno == EINTR)
  {
    goto again;
  }
  else if(n < 0)
  {
    perror("ERROR: str_echo\n");
  }

}


static void* doit(void *arg)
{
  int conn_fd;
  conn_fd = *((int *)arg);
  free(arg);

  pthread_detach(pthread_self());
  str_echo(conn_fd);

  close(conn_fd);
  return (NULL);
}


int main()
{
  int socket_fd, *connect_fd; 
  struct sockaddr_in servaddr, clientaddr;
  socklen_t client_len;
  pid_t child_pid;
  pthread_t tid;

  socket_fd = socket(AF_INET, SOCK_STREAM, 0);

  if(socket_fd == -1)
  {
    perror("Error: created socket!\n");
    exit(1);
  }

  bzero(&servaddr, sizeof(servaddr));
  servaddr.sin_family = AF_INET;
  servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
  servaddr.sin_port = htons(SERV_PORT);

  if((bind(socket_fd, (SA *)&servaddr, sizeof(servaddr))) == -1)
  {
    perror("Error: Bind port!\n");
    exit(1);
  }

  if((listen(socket_fd, LISTENQ)) == -1)
  {
    perror("Error: Listen!\n");
    exit(1);
  }

  for(;;)
  {
    client_len = sizeof(clientaddr);
    connect_fd = (int*)malloc(sizeof(int));

    if(connect_fd == NULL)
    {
      perror("Error: malloc!\n");
      exit(1);
    }

    *connect_fd = accept(socket_fd, (SA *)&clientaddr, &client_len);

    if(pthread_create(&tid, NULL, &doit, connect_fd) == -1)
    {
      perror("Error: pthread!\n");
      exit(1);
    }

  }

  exit(0);
}



4、线程特定数据
把一个未线程化的程序转换成使用线程的版本时，有时会碰到因其中有函数使用静态变量而引起的一个常见编程错误。 有以下几种解决方案：

使用线程特定数据。这个办法并不简单，而且转换成了只能在支持线程的系统上工作的函数。本办法的优点是调用顺序无需变动，所有变动都体现在库函数中而非调用这些函数的应用程序中。
改变调用顺序
改变接口的结构，避免使用静态变量，这样函数就可以是线程安全的。
每个系统支持有限数量的线程特定数据元素。POSIX要求这个限制不小于128（每个进程）

5、互斥锁
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mptr);
int pthread_mutex_unlock(pthread_mutex_t *mptr);
Both return: 0 if OK, positive Exxx value on error

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<errno.h>
#include<pthread.h>

#define NLOOP 200
int counter;
pthread_mutex_t counter_mutex = PTHREAD_MUTEX_INITIALIZER;


void* doit(void *vptr)
{
  int i, value;

  for(i = 0; i < NLOOP; i++)
  {
    if(pthread_mutex_lock(&counter_mutex) !=0)  
    {
      perror("Error: lock\n");
    }

    value = counter;
    printf("%d: %d\n", pthread_self(), value + 1);
    counter = value + 1;

    if(pthread_mutex_unlock(&counter_mutex) !=0)    
    {
      perror("Error: unlock\n");
    }
  }

  return(NULL);
}


int main()
{

  pthread_t t_id_A, t_id_B;

  pthread_create(&t_id_A, NULL, &doit, NULL);
  pthread_create(&t_id_B, NULL, &doit, NULL);

  pthread_join(t_id_A, NULL);
  pthread_join(t_id_B, NULL);

  exit(0);
}

6、条件变量
修改以上代码，使counter变量每增加10，则打印一个分隔线：

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<errno.h>
#include<pthread.h>

#define NLOOP 100
int counter;
pthread_mutex_t counter_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void* doit(void *vptr)
{
  int i, value;

  for(i = 0; i < NLOOP; i++)
  {
    if(pthread_mutex_lock(&counter_mutex) !=0)  
    {
      perror("Error: lock\n");
    }

    value = counter;
    printf("%d: %d\n", pthread_self(), value + 1);
    counter = value + 1;

    if((counter%10) == 0)
    {
      pthread_cond_signal(&cond);
    }

    if(pthread_mutex_unlock(&counter_mutex) !=0)    
    {
      perror("Error: unlock\n");
    }

    sleep(1);
  }

  return(NULL);
}

void* do_split(void *vptr)
{
  for(;;)
  {
    if(pthread_mutex_lock(&counter_mutex) !=0)  
    {
      perror("Error: lock\n");
    }

    pthread_cond_wait(&cond, &counter_mutex);

    if(counter == (NLOOP * 2))
    {
      pthread_exit(0);
    }

    printf("%d: ----------\n", pthread_self());

    if(pthread_mutex_unlock(&counter_mutex) !=0)    
    {
      perror("Error: unlock\n");
    }
  }
}


int main()
{

  pthread_t t_id_A, t_id_B, t_id_C;

  pthread_create(&t_id_A, NULL, &do_split, NULL);
  pthread_create(&t_id_B, NULL, &doit, NULL);
  pthread_create(&t_id_C, NULL, &doit, NULL);

  pthread_join(t_id_A, NULL);
  pthread_join(t_id_B, NULL);
  pthread_join(t_id_C, NULL);

  exit(0);
}


二、客户/服务器程序设计范式
1、服务器程序设计范式：
1、迭代服务器（无进程控制，用作测量基准）
2、并发服务器，每个客户请求fork一个子进程
3、预先派生子进程，每个子进程无保护地调用accept
4、预先派生子进程，使用文件上锁保护accept
5、预先派生子进程，使用线程互斥锁上锁保护accept
6、预先派生子进程，父进程向子进程传递套接字描述符
7、并发服务器，每个客户请求创建一个线程
8、预先创建线程服务器，使用互斥锁保护accept
9、预先创建线程服务器，由主线程调用accept
2、总结：
当系统负载较轻时，每来一个客户请求现场派生一个子进程为之服务的传统并发服务器程序模型就足够了。这个模型甚至可以与inetd结合使用，也就是inetd处理每个连接的接受。我们的其他意见是就重负荷运行的服务器而言的，譬如Web服务器。
相比传统的每个客户fork一次设计范式，预先创建一个子进程池或一个线程池的设计范式能够把进程控制CPU时间降低10倍或以上。编写这些范式的程序并不复杂，不过需超越本章所给例子的是：监视闲置子进程个数，随着所服务客户数的动态变化而增加或减少这个数目
某些实现允许多个子进程或线程阻塞在同一个accept调用中，另一些实现却要求包绕accept调用安置某种类型的锁加以保护。文件锁或Pthread互斥锁都可以使用。
让所有子进程或线程自行调用accept通常比让父进程或主线程独自调用accept并把描述符传递给子进程或线程来的简单而快速。
由于潜在select冲突的原因，让所有子进程或线程阻塞在同一个accept调用中比让它们阻塞在同一个select调用中更可取。
使用线程通常远快于使用进程。不过选择每个客户一个子进程还是每个客户一个线程取决于操作系统提供什么支持，还可能取决于为服务每个客户需激活其他什么程序（若有其他程序需激活的话)举例子说，如果accept客户连接的服务器调用fork和exec，那么fork一个单线程的进程可能快于fork一个多线程的进程




