---
title: Unix网络编程阅读笔记一 Socket基础篇
date: 2015-02-10 09:17:59
tags:
- linux
---


由 王宇 原创并发布
一、TCP/IP UDP SCTP 等协议介绍
请参考<<TCP IP 协议详解 卷1>>

二、字符反射服务程序代码：
本文代码，在以下环境下编译通过

CentOS 6.4
Kernal version: 2.6.32
GCC version: 4.4.7
1 TCP Echo Server:
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<unistd.h>
#include<netinet/in.h>
#include<errno.h>

#define SA struct sockaddr
#define SERV_PORT 51000
#define MAXLINE 4096 
#define LISTENQ 1024

void str_echo(int socket_fd)
{
  ssize_t n;
  char buf[MAXLINE];

again:
  while( (n = read(socket_fd, buf, sizeof(buf))) > 0 )
  {
    if(write(socket_fd, buf, n) == -1 )
    {
      perror("Error:write.\n");

    }
    else
    {
      printf("Echo string: %s\n", buf);   

    }

  }

  if( n < 0 && errno == EINTR )
  {
    goto again;

  }
  else if(n < 0)
  {
    perror("ERROR: str_echo\n");

  }


}


int main()
{
  int socket_fd, connect_fd;  
  struct sockaddr_in servaddr, clientaddr;
  socklen_t client_len;
  pid_t child_pid;

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
    connect_fd = accept(socket_fd, (SA *)&clientaddr, &client_len);

    if((child_pid = fork()) == 0)
    {
      close(socket_fd);
      str_echo(connect_fd);
      exit(0);

    }

    close(connect_fd);

  }

  exit(0);

}


2 TCP Echo Client:


#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<netinet/in.h>
#include<sys/select.h>
#include<errno.h>

#define SERV_PORT 51000
#define MAXLINE 4096 
#define SA struct sockaddr

#define min(a,b)    ((a) < (b) ? (a) : (b))
#define max(a,b)    ((a) > (b) ? (a) : (b))

char* Ip_address = "127.0.0.1";


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

void str_cli(FILE *fp, int socket_fd)
{
  int n, max_fd, stdin_eof = 0;
  fd_set rset;
  char buf[MAXLINE];

  FD_ZERO(&rset);

  for(;;)
  {
    if(stdin_eof == 0)
    {
      FD_SET(fileno(fp), &rset);  

    }

    FD_SET(socket_fd, &rset);

    max_fd = max(fileno(fp), socket_fd) + 1;

    if(FD_ISSET(socket_fd, &rset) != 0 ) /* socketdable. */
    {
      if((n= read(socket_fd, buf, MAXLINE)) == 0)
      {
        if(stdin_eof == 1)
        {
          return; /* normal termination */    

        }
        else
        {
          perror("ERROR: server terminated prematurely.\n");  

        }

      }     
      write(fileno(stdout), buf, n);

    }

    if(FD_ISSET(fileno(fp), &rset) !=0 ) /* inputdable.  */
    {
      if((n =read(fileno(fp), buf, MAXLINE)) == 0)
      {
        stdin_eof = 1;
        shutdown(socket_fd, SHUT_WR); /* send FIN */
        FD_CLR(fileno(fp), &rset);
        continue;

      }

      writen(socket_fd, buf, n);

    }


  }

}


int main()
{
  int socket_fd, connect_rt;
  struct sockaddr_in servaddr;

  socket_fd = socket(AF_INET, SOCK_STREAM, 0);

  if(socket_fd == -1)
  {
    perror("Error: created socket!\n");
    exit(1);

  }

  bzero(&servaddr, sizeof(servaddr));

  servaddr.sin_family = AF_INET;
  servaddr.sin_port = htons(SERV_PORT);
  inet_pton(AF_INET, Ip_address, &servaddr.sin_addr);

  connect_rt = connect(socket_fd, (SA *)&servaddr, sizeof(servaddr));

  if(connect_rt != 0)
  {
    perror("Error: connect socket!\n");
    exit(1);

  }

  str_cli(stdin, socket_fd);


  exit(0);

}


三、Socket
1 Socket 地址结构
sockaddr_in 是 "Internet socket address structure,"   定义在<netinet/in.h>中


2 地址参数
struct in_addr {
  in_addr_t s_addr; /* 32-bit IPv4 address */
  /*  network byte ordered */

};

struct sockaddr_in {
  uint8_t sin_len; /* length of structure (16) */ 
  sa_family_t sin_family; /* AF_INET */ 
  in_port_t sin_port; /* 16-bit TCP or UDP port number */
  /* network byte ordered */
  struct in_addr sin_addr; /* 32-bit IPv4 address */ 
  /* network byte ordered */ 
  char sin_zero[8]; /* unused */ 

};

Socket 函数 bind connect 和 sendto 使用"socket address structure" ,即struct sockaddr， 但是"Internet socket address structure" 和 "socket address structure" 长度相互对应，所以可以通过C语言的强制类型转换：

#define struct sockaddr
(SA *)&servaddr
3 字节顺序
2字节，16位二进制整数，在内存中有两种排列顺序：

The most significant bit (MSB) 从左到右，高位到低位
The least significant bit(LSB) 从左到右，低位到高位
为了兼容主机字节顺序(Host byte order)和网络字节顺序(Network byte order)，提供了如下转换函数：

#include <netinet/in.h>

//Both return: value in network byte order
uint16_t htons(uint16_t host16bitvalue) ; // 主机字节顺序转网络字节顺序
uint32_t htonl(uint32_t host32bitvalue) ;


//Both return: value in host byte order
uint16_t ntohs(uint16_t net16bitvalue) ; // 网络字节顺序转主机字节顺序
uint32_t ntohl(uint32_t net32bitvalue) ;

4 字节操作函数
#include <strings.h>

//Returns: 0 if equal, nonzero if unequal  
void bzero(void *dest,size_tnbytes);                        //清零   
void bcopy(const void *src,void *dest,size_tnbytes);        //复制  
int bcmp(const void *ptr1,const void *ptr2,size_tnbytes);   //比较

5 地址转换函数
地址转换函数是网络地址(Internet addresss)与ASCII 字符之间的转换

#include <arpa/inet.h>

//Returns: 1 if string was valid, 0 on error
int inet_aton(const char *strptr,struct in_addr *addrptr);

//Returns: 32-bit binary network byte ordered IPv4 address; INADDR_NONE if error
in_addr_t inet_addr(const char *strptr);

//Returns: pointer to dotted-desimal string
char *inet_ntoa(struct in_addrinaddr);



6 表达(Presentation)与二进制数值（Numeric）转换函数
#include <arpa/inet.h>

//Returns: 1 if OK, 0 if input not a valid presentation format, -1 on error
int inet_pton(intfamily,const char *strptr,void *addrptr);

//Returns: pointer to result if OK, NULL on error
const char *inet_ntop(intfamily,const void *addrptr,char *strptr,size_tlen);



例如将字符型Ip地址转换成二进制地址：

char* Ip_address = "127.0.0.1";

inet_pton(AF_INET, Ip_address, &servaddr.sin_addr);



7 TCP Client/Server的初级函数
图：





四、I/O 复用(Multiplexing): The 'select' and 'poll' Functions
                            当client进程运行到fgets() 时，此进程被内核阻塞，此时如果server发送一个FIN给client，它将不会得到一个正确的回复。为了解决这个问题，I/O mutiplexing 提供了select 和 poll 函数。

                               1 I/O 模式
                               阻塞 I/O
  非阻塞 I/O
I/O 复用(multiplexing) (select and poll)
  信号驱动式 I/O
  异步 I/O
  2 阻塞 I/O
  阻塞调用是指调用结果返回之前，当前线程会被挂起（线程进入非可执行状态，在这个状态下，cpu不会给线程分配时间片，即线程暂停运行）。函数只有在得到结果之后才会返回。

  图：



  3 非阻塞 I/O
  当我们设置socket 为非阻塞时，我们正是在告诉内核：I/O 操作没有完成时（底层资源没有准备好），不需要将进程处于睡眠状态，而是直接返回一个错误作为替代。

  图：



4 I/O 复用(multiplexing) (select and poll)
  I/O复用模型会用到select、poll、epoll函数，这几个函数也会使进程阻塞，但是和阻塞I/O所不同的的，这两个函数可以同时阻塞多个I/O操作。而且可以同时对多个读操作，多个写操作的I/O函数进行检测，直到有数据可读或可写时，才真正调用I/O操作函数。

  例如：同时阻塞client 的fgets() 和socket

  图：




