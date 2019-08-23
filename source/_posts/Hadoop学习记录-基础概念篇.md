---
title: Hadoop学习记录-基础概念篇
date: 2016-07-07 09:13:22
tags:
- 大数据
---

![](/assets/blog-image/dlb-gb-t.jpg)
由 王宇 原创并发布 ：
 
## 一 Hadoop 概念
 
### 1. 参考资料
 
<< Distributed Systems Principles and Paradigms 2nd edition >>
<< Hadoop- The Definitive Guide, 4th Edition>>
<< Hadoop Real-World Solutions Cookbook>>

 
### 2. Hadoop 结构
<!--more-->
#### 2.1 Hadoop 整体结构
Hadoop Common: Hadoop Java 库和实用工具
Hadoop YARN: Job 调度和集群资源管理框架
Hadoop Distributed File System(HDFS): 分布式文件系统
Hadoop MapReduce: 基于YARN的大数据集合并行处理
 
#### 2.1.1 MapReduce
The Map Task: This is the first task, which takes input data and converts it into a set of data, where individual elements are broken down into tuples (key/value pairs).
The Reduce Task: This task takes the output from a map task as input and combines those data tuples into a smaller set of tuples. The reduce task is always performed after the map task.
JobTracker: The master is responsible for resource management, tracking resource consumption/availability and scheduling the jobs component tasks on the slaves, monitoring them and re-executing the failed tasks.
TaskTracker: he slaves TaskTracker execute the tasks as directed by the master and provide task-status information to the master periodically

 
#### 2.1.2 HDFS
NameNode: manages the file system metadata and one or more slave
DataNode: store the actual data
 
### 2.2 HDFS
#### 2.2.1 HDFS 功能
It is suitable for the distributed storage and processing.
Hadoop provides a command interface to interact with HDFS.
The built-in servers of namenode and datanode help users to easily check the status of cluster.
Streaming access to file system data.
HDFS provides file permissions and authentication.

 
#### 2.2.2 HDFS 结构
Namenode 任务
Manages the file system namespace.
Regulates client’s access to files.
It also executes file system operations such as renaming, closing, and opening files and directories.
Datanode
Datanodes perform read-write operations on the file systems, as per client request.
They also perform operations such as block creation, deletion, and replication according to the instructions of the namenode.
Block
Generally the user data is stored in the files of HDFS. The file in a file system will be divided into one or more segments and/or stored in individual data nodes. These file segments are called as blocks. In other words, the minimum amount of data that HDFS can read or write is called a Block. The default block size is 64MB, but it can be increased as per the need to change in HDFS configuration.

 
### 2.3 MapReduce
MapReduce 是一种处理技术，是一种基于JAVA的分布式程序模型。MapReduce算法包含两个重要的任务，Map 和 Reduce。

 
#### 2.3.1 算法
Generally MapReduce paradigm is based on sending the computer to where the data resides!
MapReduce program executes in three stages, namely map stage, shuffle stage, and reduce stage.
Map stage : The map or mapper’s job is to process the input data. Generally the input data is in the form of file or directory and is stored in the Hadoop file system (HDFS). The input file is passed to the mapper function line by line. The mapper processes the data and creates several small chunks of data.
Reduce stage : This stage is the combination of the Shuffle stage and the Reduce stage. The Reducer’s job is to process the data that comes from the mapper. After processing, it produces a new set of output, which will be stored in the HDFS.
During a MapReduce job, Hadoop sends the Map and Reduce tasks to the appropriate servers in the cluster.
The framework manages all the details of data-passing such as issuing tasks, verifying task completion, and copying data around the cluster between the nodes.
Most of the computing takes place on nodes with data on local disks that reduces the network traffic.
After completion of the given tasks, the cluster collects and reduces the data to form an appropriate result, and sends it back to the Hadoop server.

#### 2.3.2 Inputs and Outpus
#### 2.3.3 术语
PayLoad - Applications implement the Map and the Reduce functions, and form the core of the job.
Mapper - Mapper maps the input key/value pairs to a set of intermediate key/value pair.
NamedNode - Node that manages the Hadoop Distributed File System (HDFS).
DataNode - Node where data is presented in advance before any processing takes place.
MasterNode - Node where JobTracker runs and which accepts job requests from clients.
SlaveNode - Node where Map and Reduce program runs.
JobTracker - Schedules jobs and tracks the assign jobs to Task tracker.
Task Tracker - Tracks the task and reports status to JobTracker.
Job - A program is an execution of a Mapper and Reducer across a dataset.
Task - An execution of a Mapper or a Reducer on a slice of data.
Task Attempt - A particular instance of an attempt to execute a task on a SlaveNode.
 
## 二 Hadoop环境配置
 
### 1. 搭建虚拟机
VirualBox 5.0.24, 用户手册
Linux OS : Puppy Slacko6.3.2 64bit
在VirualBox 上安装三台虚拟机，机器名分别为：
hadoop-master
hadoop-slave-1
hadoop-slave-2

 
### 2. 软件安装及环境配置
下载JDK
安装Eclipse
下载Hadoop 2.7.2
安装ssh
在~/.bashrc 中配置环境变量
```shell
JAVA_HOME=/opt/jdk1.8.0_92
CLASSPATH=.:${JAVA_HOME}/lib/
export JAVA_HOME=${JAVA_HOME}
export CLASSPATH=${CLASSPATH}:${CLASSPATH}tools.jar:${CLASSPATH}dt.jar
export PATH=.:${PATH}:${JAVA_HOME}/bin:${CLASSPATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```
在 ~/.profile 最后一行加入： source ~/.bashrc 否则在ssh登录后，无法执行.bashrc, 使得环境变量无效。

 
### 3. 配置SSH
使用Package Manager 下载 OpenSSH
生成 Key
```shell
     ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
     ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
     ssh-keyigen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
```

配置 /etc/ssh/sshd_config
```shell
     PermitRootLogin yes
     TCPKeepAlive yes
     ClientAliveInterval 60
     ClientAliveCountMax 3
     MaxStartups 10:30:100
```

修改 root 密码

```shell
    passwd root
```
启动SSH服务
```shell
    /usr/sbin/sshd
```
开机自动启动SSH服务
在 ~/Startup 目录中，增加 launchSSH.sh

#!/usr/bin/bash
/usr/sbin/sshd
可以采用如下命令，调试SSH
```shell
    ssh -v ip
```
配置Putty
 
4. 配置 Hadoop
在每个节点上配置SSH Key
```shell
$ ssh-keygen -t rsa 
$ ssh-copy-id -i ~/.ssh/id_rsa.pub tutorialspoint@hadoop-master 
$ ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop_tp1@hadoop-slave-1 
$ ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop_tp2@hadoop-slave-2 
```
在每个节点上编辑hosts
```shell
# vi /etc/hosts
enter the following lines in the /etc/hosts file.
192.168.67.113 hadoop-master
192.168.67.27 hadoop-slave-1
192.168.67.98 hadoop-slave-2
```
core-site.xml
```xml
<configuration>
        <property>
                <name>fs.default.name</name>
                <value>hdfs://localhost:9000/</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
</configuration>
```

hdfs-site.xml
```xml
<configuration>
   <property>
      <name>dfs.data.dir</name>
      <value>/opt/hadoop-2.7.2/dfs/name/data</value>
      <final>true</final>
   </property>

   <property>
      <name>dfs.name.dir</name>
      <value>/opt/hadoop-2.7.2/dfs/name</value>
      <final>true</final>
   </property>

   <property>
      <name>dfs.replication</name>
      <value>1</value>
   </property>
</configuration>
```
mapred-site.xml
```xml
<configuration>
   <property>
      <name>mapred.job.tracker</name>
      <value>hadoop-master:9001</value>
   </property>
</configuration>
```

hadoop-env.sh
将下列环境变量更换为绝对路径：
```shell
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/opt/jdk1.8.0_92

#export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/etc/hadoop"}
export HADOOP_CONF_DIR="/opt/hadoop-2.7.2/etc/hadoop"
```
 
5. 安装Hadoop到 Slave 服务器上
```shell
$ cd /opt 
$ scp -r hadoop-2.7.2 hadoop-slave-1:/opt/hadoop-2.7.2
$ scp -r hadoop-2.7.2 hadoop-slave-2:/opt/hadoop-2.7.2
```
 
6. 配置 Master 服务器
$ cd /opt/hadoop-2.7.2

// Configuring Master Node
$ vi etc/hadoop/masters
hadoop-master

// Configuring Slave Node
$ vi etc/hadoop/slaves
hadoop-master
hadoop-slave-1 
hadoop-slave-2
 
7. 在Master 服务器上格式化 Name Node
$ cd /opt/hadoop-2.7.2
$ bin/hadoop namenode –format
 
8. 启动 Hadoop 服务
$ cd /opt/hadoop-2.7.2
$ sbin/start-all.sh
$ jps
 

通过logs/hadoop-root-namenode-puppypc3200.log 查看异常

 
9. 观察 Hadoop 的端口服务
$ cd /opt/hadoop-2.7.2
$ netstat -apnt | grep 'java'

 
10. 使用浏览器访问Hadoop
访问Hadoop

http://master ip:50070
确认集群上的所有应用

http://master ip:8088
 
11. 操作HDFS
HDFS 命令清单


使用 help 参数，查询命令的详细信息

 
三 Hadoop MapReduce 开发实例
 
1. WordCount 此例的功能是计算输入文件中的单词数量
```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```
 
2. 编译WordCount 并创建JAR
$ bin/hadoop com.sun.tools.javac.Main WordCount.java
$ jar cf wc.jar WordCount*.class
 
3. 编辑输入文件file01 file02
 
file01 输入：Hello World Bye World
file02 输入：Hello Hadoop Goodbye Hadoop

 
将输入文件复制到HDFS 的 /data/input 目录下

$ bin/hadoop fs -copyFromLocal file0* /data/input/.
 
4. 执行应用
$ bin/hadoop jar wc.jar WordCount /data/input /data/output
 
5. 查看结果
 



 
  
 

 
