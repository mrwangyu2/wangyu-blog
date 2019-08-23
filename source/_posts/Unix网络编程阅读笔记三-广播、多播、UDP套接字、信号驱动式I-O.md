---
title: Unix网络编程阅读笔记三 广播、多播、UDP套接字、信号驱动式I/O
date: 2015-03-05 09:17:10
tags:
- linux
---

由 王宇 原创并发布
本文代码，在以下环境下编译通过
CentOS 6.4
Kernal version: 2.6.32
GCC version: 4.4.7
一、广播
1、用途和寻址方式
广播的用途之一是在本地子网定位一个服务器主机，前提是已知或认定这个服务器主机位于本地子网，但是不知道它的单播IP地址。这种操作也称为资源发现。另一个用途是在有多个客户主机与单个服务器主机通信的局域网环境中尽量减少分组流通。

图 20-1





TCP 只支持单播
IPv6 不支持广播
IPv4 是可选的
广播和多播要求用于UDP或原始IP
2、单播和广播的比较
图 20-3





图 20-4





单播IP数据报仅由通过目的IP地址指定的单个主机接收，子网上的其他主机都不受任何影响。 广播存在的根本问题，子网上未参加相应广播应用的所有主机也不得不沿协议栈一路向上完整地处理收取的UDP广播数据报，直到该数据报历经UDP层时被丢弃为止。另外，子网上所有非IP的主机也不得不在数据链路层接收完整的帧，然后再丢弃它，这有可能严重影响子网上这些其他主机的工作。

  3、使用广播的dg_cli函数
void dg_cli( FILE *fp, int socket_fd, const SA *server_addr, socklen_t server_len)
{
  int n;
  const int on = 1;   
  char send_line[MAXLINE], recv_line[MAXLINE + 1];
  struct sockaddr *preply_addr;
  socklen_t len;

  preply_addr = (struct sockaddr*)malloc(server_len);
  if(preply_addr == NULL)
  {
    perror("Error: malloc.\n");
    return; 
  }

  if(setsockopt(socket_fd, SOL_SOCKET, SO_BROADCAST, &on, sizeof(on)) == -1)/*广播标识：SO_BROADCAST*/
  {
    perror("Error: setsockopt\n");
    return; 
  }

  if(signal(SIGALRM, recvfrom_alarm) == SIG_ERR)
  {
    perror("Error: signal\n");
    return; 
  }

  while(fgets(send_line, MAXLINE, fp) != NULL)
  {
    if(sendto(socket_fd, send_line, strlen(send_line), 0, server_addr, server_len) == -1)
    {
      perror("Error: sendto\n");
      return; 
    }

    alarm(5);

    for(;;)
    {
      len = server_len;
      n = recvfrom(socket_fd, recv_line, MAXLINE, 0, preply_addr, &len);

      if(n < 0)
      {
        if(errno == EINTR)
        {
          break;  
        }
        else
        {
          perror("Error: recvfrom\n");    
        }
      }
      else
      {
        recv_line[n] = 0;
        printf("from %s: %s\n", sock_ntop_host(preply_addr, len), recv_line );
      }
    }

  }

  free(preply_addr);
}

二、 多播
单播地址标识单个IP接口，广播地址标识某个子网的所有IP接口，多播地址标识一组IP接口。

1、IPv4 的D类地址
IPv4的D类地址（从224.0.0.0到239.255.255.255）是IPv4多播地址。D类地址的低序28位构成多播组ID，整个32位地址则称为组地址

图A-3

图21-1





2、局域网上多播和广播的比较
图21-4





3、使用多播的dg_cli函数
通过简单地去掉setsockopt调用来修改广播的dg_cli函数，则发送多播数据报无需设置任何多播套接字选项。

三、高级UDP套接字编程
1、何时用UDP代替TCP
UDP的优势：

UDP支持广播和多播。事实上如果应用程序使用广播或多播，那就必须使用UDP
UDP没有连接建立和拆除，减小资源开销
UDP无法提供的TCP特性，需注意的是，不是所有应用程序都需要TCP的所有这些特性：

正面确认，丢失分组重传，重复分组检测，给被网络打乱次序的分组排序。TCP确认所有数据，以便检测出丢失的分组。这些特性的实现要求每个TCP数据分节都包含一个能被对端确认的序列号。这些特性还要求TCP为每个连接估算重传超时值，该值应随着两个端系统之间分组流通的变化持续更新。
窗口式流量控制
慢启动和拥塞避免
对于简单的请求-应答应用程序可以使用UDP，不过错误检测功能必须加到应用程序内部。
对于海量数据传输（例如文件传输）不应该使用UDP
2、给UDP应用增加可靠性
在客户程序中增加两个特性：

超时和重传：用于处理丢失的数据报
处理超时和重传的老式方法是先发送一个请求并等待N秒钟。如果期间没有收到应答，那就重新发送同一个请求并再等待N秒钟。如此发生一定次数后放弃发送。
序列号：供客户验证一个应答是否匹配相应的请求
客户为每个请求冠以一个序列号，服务器必须在返送给客户的应答中回射这个序列号。
图：22-5





#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<fcntl.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<sys/select.h>
#include<sys/time.h>
#include<netinet/in.h>
#include<setjmp.h>
#include<errno.h>
#include<signal.h>

#define SERV_PORT 51001
#define MAXLINE 4096 
#define SA struct sockaddr

char* Ip_address = "192.168.153.130";

struct rtt_info
{
  float    rtt_rtt;     /* most recent measured RTT, in seconds */
  float    rtt_srtt;    /* smoothed RTT estimator, in seconds */
  float    rtt_rttvar;  /* smoothed mean deviation, in seconds */
  float    rtt_rto;     /* current RTO to use, in seconds */
  int      rtt_nrexmt;  /* times retransmitted: 0, 1, 2, ... */
  uint32_t rtt_base;    /* sec since 1/1/1970 at start */
};

#define RTT_RXTMIN    2    /* min retransmit timeout value, in seconds */
#define RTT_RXTMAX    60   /* max retransmit timeout value, in seconds */
#define RTT_MAXNREXMT 3    /* max times to retransmit */

/*
 * Calculate the RTO value based on current estimators:
 *   smoothed RTT plus four times the deviation
 */

#define RTT_RTOCALC(ptr) ((ptr)->rtt_srtt + (4.0 * (ptr)->rtt_rttvar))


static struct rtt_info rttinfo;
static int rttinit =0;
static struct msghdr msgsend, msgrecv;
static struct hdr
{
  uint32_t seq;  /* sequence */
  uint32_t ts;   /* timestamp when sent */

}sendhdr, recvhdr;

static sigjmp_buf jmpbuf;

int rtt_d_flag = 0;

static float rtt_minmax(float rto)
{
  if(rto < RTT_RXTMIN)
  {
    rto = RTT_RXTMIN;
  }
  else if(rto > RTT_RXTMAX)
  {
    rto = RTT_RXTMAX;   
  }

  return(rto);
}


void rtt_init(struct rtt_info *ptr)
{
  struct timeval tv;
  if(gettimeofday(&tv, NULL) == -1)
  {
    perror("Error: gettimeofday in rtt_init!\n");
  }

  ptr->rtt_base = tv.tv_sec;

  ptr->rtt_rtt = 0;
  ptr->rtt_srtt = 0;
  ptr->rtt_rttvar = 0.75;
  ptr->rtt_rto = rtt_minmax(RTT_RTOCALC(ptr));
  /* first RTO at (srtt + (4 * rttvar)) = 3 secounds */
}


uint32_t rtt_ts(struct rtt_info *ptr)
{
  uint32_t ts;
  struct timeval tv;

  if(gettimeofday(&tv, NULL) == -1)
  {
    perror("Error: gettimeofday in rtt_ts!\n");
  }
  ts = ((tv.tv_sec - ptr->rtt_base) * 1000) + (tv.tv_usec /1000);
  return(ts);
}


void rtt_newpack(struct rtt_info *ptr)
{
  ptr->rtt_nrexmt = 0;
}


int rtt_start(struct rtt_info *ptr)
{
  return((int)(ptr->rtt_rto + 0.5)); /* round float ot int*/
  /* return value can be used as: alarm(rtt_start(&foo)) */
}


void rtt_stop(struct rtt_info *ptr, uint32_t ms)
{
  double delta;

  ptr->rtt_rtt = ms / 1000.0; /* measured RTT in seconds*/

  delta = ptr->rtt_rtt - ptr->rtt_srtt;
  ptr->rtt_srtt += delta / 8; /* g = 1/8*/

  if(delta < 0.0)
  {
    delta = -delta;
  }

  ptr->rtt_rttvar += (delta - ptr->rtt_rttvar) /4; /* h = 1/4 */

  ptr->rtt_rto = rtt_minmax(RTT_RTOCALC(ptr));
}


int rtt_timeout(struct rtt_info *ptr)
{
  ptr->rtt_rto *=2; /* next RTO */

  if(++ptr->rtt_nrexmt > RTT_MAXNREXMT)
  {
    return -1; /* time to give up for this packet */
  }

  return 0;
}


static void sig_alarm(int signo)
{
  siglongjmp(jmpbuf, 1);
}


ssize_t dg_send_recv(int fd, const void *outbuff, size_t outbytes,
    void *inbuff, size_t inbytes,
    const SA *destaddr, socklen_t destlen)
{
  ssize_t n;
  struct iovec iovsend[2], iovrecv[2];

  if(rttinit == 0)
  {
    rtt_init(&rttinfo);
    rttinit = 1;
    rtt_d_flag = 1;
  }

  sendhdr.seq++;

  msgsend.msg_name = destaddr;
  msgsend.msg_namelen = destlen;
  msgsend.msg_iov = iovsend;
  msgsend.msg_iovlen = 2;
  iovsend[0].iov_base = &sendhdr;
  iovsend[0].iov_len = sizeof(struct hdr);
  iovsend[1].iov_base = outbuff;
  iovsend[1].iov_len = outbytes;

  msgrecv.msg_name = NULL;
  msgrecv.msg_namelen = 0;
  msgrecv.msg_iov = iovrecv;
  msgrecv.msg_iovlen = 2;
  iovrecv[0].iov_base = &recvhdr;
  iovrecv[0].iov_len = sizeof(struct hdr);
  iovrecv[1].iov_base = inbuff;
  iovrecv[1].iov_len = inbytes;

  if(signal(SIGALRM, sig_alarm) == SIG_ERR)
  {
    perror("Error: signal\n");
    return; 
  }

  rtt_newpack(&rttinfo); /* iinitialize for this packet */

sendagain:
  sendhdr.ts = rtt_ts(&rttinfo);
  sendmsg(fd, &msgsend, 0);

  alarm(rtt_start(&rttinfo)); /* calc timeout value & start timer */

  if (sigsetjmp(jmpbuf, 1) !=0  )
  {

    if ( rtt_timeout(&rttinfo) < 0 ) 
    {
      printf("dg_send_recv: no response from server, giving up");
      rttinit = 0; /* reinit in case we're called again */
      errno = ETIMEDOUT;
      return(-1);
    }

    goto sendagain;
  }

  do {
    n = recvmsg(fd, &msgrecv, 0);
  } while (n < sizeof(struct hdr) || recvhdr.seq != sendhdr.seq );

  alarm(0);
  /* calculate & store new RTT estimator values */
  rtt_stop(&rttinfo, rtt_ts(&rttinfo) - recvhdr.ts);
  return(n -sizeof(struct hdr)); /* return size of received datagram */
}


void dg_cli( FILE *fp, int socket_fd, const SA *server_addr, socklen_t server_len)
{
  int n;
  char send_line[MAXLINE], recv_line[MAXLINE + 1];

  while(fgets(send_line, MAXLINE, fp) != NULL)
  {
    n = dg_send_recv(socket_fd, send_line, strlen(send_line),
        recv_line, MAXLINE, server_addr, server_len);

    recv_line[n] = 0;
    fputs(recv_line, stdout);
  }
}


int main()
{
  int socket_fd, connect_rt;
  struct sockaddr_in server_addr;

  bzero(&server_addr, sizeof(server_addr));

  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(SERV_PORT);
  inet_pton(AF_INET, Ip_address, &server_addr.sin_addr);

  socket_fd = socket(AF_INET, SOCK_DGRAM, 0);

  if(socket_fd == -1)
  {
    perror("Error: created socket!\n");
    exit(1);
  }

  dg_cli(stdin, socket_fd, (SA *)&server_addr, sizeof(server_addr));

  exit(0);
}



3、并发UDP服务器
对于UDP, 我们必须应对两种不同类型的服务器： 

(1) 读入一个客户请求并发送一个应答后，与这个客户就不再相关了。这种情况下，读入客户请求的服务器可以fork一个子进程并让子进程去处理该请求。


(2) UDP服务器与客户交互多个数据报。问题是客户知道的服务器端口号只有服务器的一个荣所周知端口。一个客户发送其请求的第一个数据报到这个端口，但是服务器如何区分这是来自该客户同一个请求的后续数据报还是来自其他客户请求的数据报呢？这个问题典型的解决办法是让服务器为每个客户创建一个新的套接字，在其上bind一个临时端口，然后使用该套接字发送对该客户的所有应答。这个办法要求客户查看服务器第一个应答中的源端口号，并把本请求的后续数据报发送到该端口。

图22-19







四、 信号驱动式I/O
信号驱动式I/O是指进程预先告之内核，使得当某个描述符上发生某事时，内和使用信号通知相关进程。信号驱动式I/O不是真正的异步I/O, 非阻塞式I/O同样不是异步I/O

1、针对一个套接字使用信号驱动式I/O要求进程执行以下3个步骤：
建立SIGIO信号的信号处理函数。
设置该套接字的属主，通常使用fcntl的F_SETOWN命令设置
开启改套接字的信号驱动式I/O, 通常通过使用fcntl的F_SETFL命令打开O_ASYNC标志完成。
2、对于UDP套接字的SIGIO信号
SIGIO信号在发生以下事件时产生：

数据报到达套接字；
套接字上发生异步错误；
因此当捕获对于某个UDP套接字的SIGIO信号时，我们调用recvfrom或者读入到达的数据报，或者获取发生的异步错误

3、对于TCP套接字的SIGIO信号
不幸的是，信号驱动式I/O对于TCP套接字近乎无用。问题在于该信号产生得过于频繁，并且它的出现并没有告诉我们发生了什么事件。

4、使用SIGIO的UDP回射服务器程序







