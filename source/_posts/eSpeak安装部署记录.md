---
title: eSpeak安装部署记录
date: 2013-04-22 09:21:40
tags:
- 应用实践
---


 


由 王宇 原创并发布 ：



一、环境



1、Win7 64位 +  VMware Player 4.0.1 build-528992



2、Open SUSE 11.04 Kernel version ：2.6.37.1-1.1-default



3、gcc g++ version 4.5.1

gdb (7.2-3.3)

  glibc :2.11.3

  make:3.82

  vim version 7.3



  4、下载eSpeak  :

  espeak-1.47.05-source.zip :http://espeak.sourceforge.net/download.html

  portaudio_v18.zip               :www.portaudio.com

portaudio_v19_20111121.tgz (此版本不兼容espeak)



  二、编译调试



  1、首先安装PortAudio



  eSpeak 支持两种音频框架，一种是PortAudio,另一种是PulseAudio,后者过于复杂，由于时间原因，没有深入调试



  (1) 解包：

  unzip portaudio_v18.zip

  cd poraudio_v18



  (2)查看Linux系统中的音频驱动：

  ll /dev/ | grep 'audio'

  输出如下：



  rw-rw---- 1 root audio    14,  12 Apr 22 11:17 adsp
  crw-rw---- 1 root audio    14,   4 Apr 22 11:17 audio
  crw-rw---- 1 root audio    14,   9 Apr 22 11:16 dmmidi
  crw-rw---- 1 root audio    14,   3 Apr 22 11:17 dsp
  crw-rw---- 1 root audio    14,   2 Apr 22 11:16 midi
  crw-rw---- 1 root audio    14,   0 Apr 22 11:17 mixer


  (3) 修改PortAudio 驱动程序

  cd ./pa_unix_oss/

  vim ./pa_unix_osss.c

  133行：#define DEVICE_NAME_BASE            "/dev/dsp"   将此处注释掉，新插入一行：

#define DeVICE_NAME_BASE            "/dev/adsp"



  此处的修改原因是：PortAudio无法打开音频驱动dsp,会导致PortAudio的初始化错误。我尝试了audio依然无法使用。最后尝试使用音频驱动adsp，通过测试，成功运行了PortAudio



  (4)编译

  cd portaudio_v18

  make

  make libsintall



  (5)测试PortAudio

  cd pa_unix_oss

  vim Makefile

  10-29行为PortAudio的测试项目，选择第一个patest_sine.c 。将此行的注释去掉，make编译后运行：

make run (会听到类似噪音的测试音效，太难听了。。)

  至此PortAudio 安装调试成功。



  (6) 版本兼容问题：                      

  如果使用PortAudio V19 ,编译eSpeak时，会出现undefined reference to `Pa_StreamActive'的错误。                



  2、编译安装eSpeak



  (1) 解压： 

  unzip espeak-1.47.05-source.zip



  (2) 修改Makefile 同 PortAudio 关联上

  cd ./espeak-1.47.05-source/src

  vim Makefile

  30行：AUDIO = portaudio  注释掉

  31行：AUDIO = portaudio0  注释打开

  53行：LIB_AUDIO=/usr/lib/libportaudio.so.0   注释掉  插入一行

  LIB_AUDIO=/usr/liblibportaudio.so

  如果以上方法认为过于复杂，可以采用链接(link)文件的方法，重要的是在编译eSpeak时，准确的使用PortAudio 的libportaudio.so动态库，即可。



  (3) 编译

  cd ./espeak-1.47.05-source/src

  make

  make install      //执行此命令时注意，在src的上一级目录中，一定要有espeak-data目录





  三、运行和日志



  运行：

  espeak "I am programmer good job"

  执行后可以听到一个男老外的声音，呵呵。。。



  日志：

  打开文件/tmp/espeak.log   可以看到执行时的日志记录



  调试：

  打开debug.h 文件

  4行：//#define DEBUG_ENABLED    去掉注释，编译eSpeak后，可以进入调试模式，但是运行调试的时候会出现Segmentation fault (段错误，通常是非法指针或空指针的问题)


