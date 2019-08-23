---
layout: linux
title: ROS(Robot OS)Tutorial 学习记录
date: 2018-03-31 07:02:38
tags:
- linux
---

![](/assets/blog-image/ros-1.jpg)
由  王宇 原创并发布 ：

注： 此文档仅仅是我阅读时的流水账，也因为本人英语水平有限，仅凭个人的自我理解所编写，不可避免的存在错误和偏差，请以官方原文为准。

官方教程：http://wiki.ros.org/ROS/Tutorials

## Beginner Level

### 1. 安装配置ROS环境

#### 安装ROS
  * 官方推荐 ROS Kinettic Kame   http://wiki.ros.org/Distributions
  * 安装步骤： http://wiki.ros.org/kinetic/Installation/Ubuntu
#### 管理环境
  * 概念 ：<font color=red>notion of combining spaces using the shell environment </font>
  * 检查环境
  ```shell
  $ printenv | grep ROS
  ```
  * 安装命令
  ```shell
  $ source /opt/ros/kinetic/setup.bash
  ```
<!--more-->

#### 创建ROS工作空间
  * Build空间
  ```shell
  $ mkdir -p ~/catkin_ws/src
  $ cd ~/catkin_ws
  $ catkin_make
  ```
  * catkin : http://wiki.ros.org/catkin
  * 设置
  ```shell
  $ source devel/setup.bash
  ```
  * 确认环境变量： $ROS_PACKAGE_PAHT
  /home/youruser/catkin_ws/src:/opt/ros/kinetic/share
### 2. ROS文件系统
#### 预备
* 安装教程预先准备的Package
```shell
$ sudo apt-get install ros-<distro>-ros-tutorials
```
#### <font color=red>Overview 文件系统概念</font>
  * Package : Package 组织ROS单元代码的抽象组织结构，每个Package能够包含库，可以执行文件，脚本和其他部分
  * Manifests（package.xml) : Manifest是对Package的描述，它定义Package的依赖，明确Package的元数据，像版本，License等。
#### 文件系统工具
  * 使用 rospack - 得到Packages中的信息
  ```shell
  $ rospack find [package_name]
  ```
  * 使用 roscd - 更改目录
  ```shell
  $ roscd [locationname [/subdir]]
  ```
  * roscd log - 切换到日志目录
  ```shell
  $ roscd log
  ```
  * 使用 rosls - 查询目录
  ```shell
  $ rosls [locationname [/subdir]]
  ```
#### Tab 补全

### 3.创建ROS Package
* Package 由什么组成的？
  * package.xml 提供Package的元数据描述
  * catkin使用的CMakeList.txt
  * 每个Package必须有它自己的目录

* 在 catkin工作空间中的Package
![](/assets/blog-image/ros-11.jpg)

* 创建catkin Package
使用catkin_create_pkg脚本创建 catkin Package
```shell
$ cd ~/catkin_ws/src
$ catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```
* 编译和设置
```shell
$ cd ~/catkin_ws
$ catkin_make
$ . ~/catkin_ws/devel/setup.bash
```
* package 依赖
* 自定义你的Package

### 4. 编译ROS Package
#### 编译 Packages
* 使用 catkin_make - 类似于cmake make工具
```shell
$ catkin_make --source my_src
$ catkon_make install --source my_src
```
* 构建你的Package
```shell
$ cd ~/catkin_ws/
$ ls src
beginner_turorials/CMakeLists.txt
$ catkin_make
```

### 5. 理解ROS Nodes节点
#### 预备- 同ROS文件系统
#### <font color=red>Overview Graph 概念</font>
* Node : 用ROS（roscore）协同其他node的可以执行体
* Message: 当使用subscribing和publishing topic时，被使用的数据类型
* Topics：subscribing和publishing 时的主题
* Master：ROS的服务名
* rosout：ROS的 stdout stderr
* roscore： Master + rosout + parameter server

#### Nodes
* ROS nodes 使用 ROS 客户端库与其他节点通讯。
* Nodes可以发布和订阅一个主题。Node
* Node能够提供和使用服务

#### 客户端库
ROS客户端库允许nodes使用不同的程序语言编写，用于通讯，例如Python和C++

#### roscore
* 运行roscore
```
$ roscore
```
* 变更权限
```
$ sudo chown -R <your_username> ~/.ros
```
* 网络设置： http://wiki.ros.org/ROS/NetworkSetup#Single_machine_configuration

#### 使用 rosnode
* 查询Node信息
```shell
$ rosnode list
```
从rosout中查看内容

* rosnode info 命令查看特定node信息

#### 使用rosrun
```shell
$ rosrun [package_name] [node_name]
```
### 6. 理解ROS Topics主题
#### 启动一个例子
* 启动 roscore
```shell
$ roscore
```

* 运行 turtlesim - <font color=red>turtlesim是package，turtlesim_node 和 turtle_turtle_teleop_key是 node</font>
```shell
$ rosrun turtlesim turtlesim_node
```
* turtle键盘遥控
```shell
$ rosrun turtlesim turtle_teleop_key
```
![](/assets/blog-image/ros-2.jpg)

#### ROS Topics
The turtlesim_node 和 turtle_teleop_key 节点使用各自的ROS Topic进行通讯。turtle_teleop_key是发布键盘消息的topic， turtlesim 订阅相同topic，去接受键盘消息。让我们使用rqt_graph(能够显示nodes和topics)去运行

* 使用 rqt_graph
rqt_graph在系统中，建立一个实时动态图像。rqt_graph是rqt package的一部分。
  * 安装 rqt_graph
  ```shell
  $ sudo apt-get install ros-<distro>-rqt
  $ sudo apt-get install ros-<distro>-rqt-common-plugins
  ```
  * rqt_graph的显示效果
  ![](/assets/blog-image/ros-3.jpg)


* 介绍rostopic
rostopic 工具允许你获得关于ROS topics的信息
```shell
$ rostopic -h
```
* 使用 Using rostopic echo
rostopic echo显示在topic上的被发布数据
```shell
$ rostopic echo [topic]
```
* 使用 rostopic list
rostopic list返回所有当前发布和订阅主题列表
```shell
$ rostopic list -h
```
#### ROS Message
在nodes之间，topics上凭借发送的ROS Message去通讯。因为发布和订阅通讯，它们必须发送接收相同类型的信息。这意味着，topic类型凭借message 类型发布被定义。

* 使用 rostopic type
rostopic type 返回topic发布的message类型。
```shell
$ rostopic type [topic]
```
#### rostopic continued
* 使用 rostopic pub
rostopic pub 发布数据给当前被公布的topic
```shell
$ rostopic pub [topic] [msg_type] [args]
```
* 使用rostopic hz
rostopic 报告被发布数据的频率。
```shell
$ rostopic hz [topic]
```
* 使用rqt_plot
rqt_plot显示一个在topic上被发布数据的实时滚动图
![](/assets/blog-image/ros-4.jpg)


### 7. 理解ROS 的服务和参数
#### ROS 服务
<font color=red>服务是使node与其他node通讯的另外一种途径。服务允许node发送一个请求和接收一个回应</font>

#### 使用rosservice
rosservice能够轻松的使用serives附加到ROS C/S框架结构。 rosservice有许多能够使用在topics上的命令。
![](/assets/blog-image/ros-55.jpg)


### 8. 使用rqt_console and roslaunch
ROS使用rqt_console 和 rqt_logger_level进行调试，使用roslaunch一次启动多个node

#### 预备
#### 使用rqt_console 和rqt_logger_level
rqt_console依附于ROS的日志框架，用于显示从node的输出信息。rqt_logger_level允许我去改变运行时node的调试级别（DEBUG WARN INFO ERROR）

### 9. 使用rosed在ROS中编辑文件
#### 使用rosed
rosbash : http://wiki.ros.org/rosbash
rosed是rosbash套件中的一部分。它允许你直接在package内编辑文件。
```shell
$ rosed [package_name] [filename]
```

#### 使用rosed 用 tab补全
```shell
$ rosed [package_name] <tab><tab>
```
#### 编辑器
rosed默认编辑器是vim。 更加友好的编辑器nano通过默认Ubuntu 安装时被包含。你能够通过编辑你的~/.bashr文件去使用它们。
```shell
export EDITOR='nano -w'
```

### 10. 创建ROS msg 和 srv
####  <font color=red>介绍msg 和 srv </font>
* msg：msg files是描述ROS信息字段的样例文本文件。它们用于生成不同语言信息的源代码
* srv：srv file表示一个服务。由两部分组成： a request , a response

msg files 被保存在package 的msg目录中， svr files 被保存在srv目录中

* 字段类型
![](/assets/blog-image/ros-5.jpg)

在ROS还有一些特殊的类型： Hear，它包含一个时间戳和在ROS中被使用的坐标参照信息。你将在msg file中第一行频繁的看到Hearder
例子：
![](/assets/blog-image/ros-6.jpg)

srv file就像msg files，除了他们包含两部分内容：request，respone。这两部分内容被“-------”行分开

#### 使用msg
* 创建msg
让我们定义一个在之前的教程被创建的package中的msg
```shell
$ roscd beginner_tutorials
$ mkdir msg
$ echo "int64 num" > msg/Num.msg
```
<font color=red> 这部分内容待续</font>
* 使用rosmsg
```shell
$ rosmsg show [message type]
```

#### 使用srv
* 创建srv

<font color=red> 这部分内容待续</font>
* 使用rossrv
```shell
$ rossrv show <service type>
$ rossrv show begineer_turoials/AddTwoInts
```

#### 用于msg 和srv 常见步骤
* 改变CMakeList.txt

#### 获得帮助
```shell
$ rosmsg -h
$ rosmsg show -h
```
### 11. 用C++写一个Publisher和Subscriber的例子
 http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29

### 12. 用Python写一个Publisher和Subscriber的例子
http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29

### 13. 测试Publisher和Subscriber的例子
http://wiki.ros.org/ROS/Tutorials/ExaminingPublisherSubscriber

### 14. 用C++写一个Service 和 Client的例子
http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29
### 15. 用Python写一个Service和Client的例子
http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28python%29
### 16. 测试Service和Client的例子
http://wiki.ros.org/ROS/Tutorials/ExaminingServiceClient
### 17. 记录和播放数据
#### 记录数据-创建一个bag 文件
* 启动roscore，运行两个node，一个是显示乌龟，一个是键盘操作
  * Terminal1 : $ roscore 
  * Terminal2 : $ rosrun turtlesim turtlesim_node
  * Terminal3 : $ rosrun turtlesim tutle_teleop_key

* 记录所有的 publisher topics
```shell
$ mkdir ~/bagfiles
$ cd ~/bagfiles
$ rosbag record -a
```

#### 测试和播放bag file
```shell
$ rosbag info <your bagfile>
```
#### 记录数据的子集
```
$ rosbag record -O subset /turtle1/cmd_vel /turtle1/pose
```

#### rosbag 记录播放限制-（没搞懂）

### 18. 用roswtf开始
roswtf： 诊断ROS运行时错误的工具
http://wiki.ros.org/roswtf

### 19. 导航ROS wiki

## 总结
![](/assets/blog-image/ros-7.jpg)










