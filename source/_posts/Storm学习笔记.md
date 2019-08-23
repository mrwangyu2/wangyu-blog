---
title: Storm学习笔记
date: 2016-10-20 09:12:30
tags:
- 大数据
---

![](/assets/blog-image/storm-t.jpg)
由 王宇 原创并发布 ：

## Storm 概念
Let us now have a closer look at the components of Apache Storm −

|Components|Description|
| -------- |:----------:|
|Tuple|Tuple is the main data structure in Storm. It is a list of ordered elements. By default, a Tuple supports all data types. Generally, it is modelled as a set           of comma separated values and passed to a Storm cluster.|
|Stream|Stream is an unordered sequence of tuples.|
|Spouts|Source of stream. Generally, Storm accepts input data from raw data sources like Twitter Streaming API, Apache Kafka queue, Kestrel queue, etc. Otherwise you can write spouts to read data from datasources. “ISpout" is the core interface for implementing spouts. Some of the specific interfaces are IRichSpout, BaseRichSpout, KafkaSpout, etc.|
|Bolts|Bolts are logical processing units. Spouts pass data to bolts and bolts process and produce a new output stream. Bolts can perform the operations of filtering, aggregation, joining, interacting with data sources and databases. Bolt receives data and emits to one or more bolts. “IBolt” is the core interface for implementing bolts. Some of the common interfaces are IRichBolt, IBasicBolt, etc.|
|Topology|Spouts and bolts are connected together and they form a topology. Real-time application logic is specified inside Storm topology. In simple words, a topology is a directed graph where vertices are computation and edges are stream of data.|
|Tasks|In simple words, a task is either the execution of a spout or a bolt.|
|Nimbus|Nimbus is a master node of Storm cluster. All other nodes in the cluster are called as worker nodes. Master node is responsible for distributing data among all the worker nodes, assign tasks to worker nodes and monitoring failures.|
|Supervisor|The nodes that follow instructions given by the nimbus are called as Supervisors. A supervisor has multiple worker processes and it governs worker processes to complete the tasks assigned by the nimbus.|
|Worker process|A worker process will execute tasks related to a specific topology. A worker process will not run a task by itself, instead it creates executors and asks them to perform a particular task. A worker process will have multiple executors.|
|Executor|An executor is nothing but a single thread spawn by a worker process. An executor runs one or more tasks but only for a specific spout or bolt.|
|Task|A task performs actual data processing. So, it is either a spout or a bolt.|
|ZooKeeper framework|Apache ZooKeeper is a service used by a cluster (group of nodes) to coordinate between themselves and maintaining shared data with robust synchronization techniques. Nimbus is stateless, so it depends on ZooKeeper to monitor the working node status.

ZooKeeper helps the supervisor to interact with the nimbus. It is responsible to maintain the state of nimbus and supervisor.|
<!--more-->

![](/assets/blog-image/storm-1.jpg)
![](/assets/blog-image/storm-2.jpg)
![](/assets/blog-image/storm-3.jpg)

* Stream Grouping(消息分发策略)
    * Shuffle Grouping 随机分组
    * Fields Grouping 按字段分组
    * All Grouping 广播发送，对于每个tuple, 所有Bolts都会收到
    * Global Grouping 全局分组
    * None Grouping 同随机分组相同
    * Direct Grouping 指向分组
    * Local or shuffle Grouping 本地或随机分组
## Storm Workflow
* Local Mode
* Production Mode
##  Storm 配置
 * 步骤一： 安装JDK 并配置环境变量 JAVA_HOME CLASSPATH
 * 步骤二 :  安装ZooKeeper
  * 下载ZooKeeper 
  * 解包
  ```shell
 $ tar xzvf zookeeper-3.5.2-alpha.tar.gz
 $ mv ./zookeeper-3.5.2-alpha /opt/zookeepter
 $ cd  /opt/zookeeper
 $ mkdir data
  ```
  * 创建配置文件
  ```shell
 $ cd /opt/zookeeper
 $ vim conf/zoo.cfg

 tickTime=2000
 dataDir=/path/to/zookeeper/data
 clientPort=2181
 initLimit=5
 syncLimit=2
  ```
  * 启动ZooKeeper Seve
  ```shell
 $ bin/zkServer.sh start
  ```
 * 步骤三：在安装配置Storm
 > 下载Storm
 > 解包
  ```shell
 $ tar xvfz apache-storm-1.0.2.tar.gz
 $ mv apache-storm-1.0.2 /opt/storm
 $ cd /opt/storm
 $ mkdir data
  ```
 > 编辑Storm配置

  ```shell
 $ cd /opt/storm
 $ vim conf/storm.yaml
 storm.zookeeper.servers:
  - "localhost"
  storm.local.dir: “/path/to/storm/data(any path)”
  nimbus.host: "localhost"
  supervisor.slots.ports:
   - 6700
   - 6701
   - 6702
   - 6703
     ui.port: 6969
  ```
      > 启动 Nimbus
  ```shell
      $ cd /opt/storm
      $ ./bin/storm nimbus
  ```
      > 启动 Supervisor 
  ```shell
      $ cd /opt/storm
      $ ./bin/stormi supervisor 
  ```
      > 启动 UI 
  ```shell
      $ cd /opt/storm
      $ ./bin/storm  ui 
  ```

## 在Storm上开发实现一个统计任务
* 场景 - 统计移动电话的数量.
    > 在Spout中，准备4个电话号码和电话之间随机通话数量。
    > 分别创建不同的Bolt，用于统计
    > 使用 Topology 将 Spout 和 Bolt 关联起来
* 以下程序在Ubuntu 16.04  64位 JDK1.8 环境下编译执行通过 
* 创建 Spout 组件
  Spout 需要继承 IRichSpout 接口， 接口描述如下：
    > open − Provides the spout with an environment to execute. The executors will run this method to initialize the spout.
    > nextTuple − Emits the generated data through the collector.
    > close − This method is called when a spout is going to shutdown.
    > declareOutputFields − Declares the output schema of the tuple.
    > ack − Acknowledges that a specific tuple is processed
    > fail − Specifies that a specific tuple is not processed and not to be reprocessed.

  ```java
    import java.util.*;
    //import storm tuple packages
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Values;   
    //import Spout interface packages
    import org.apache.storm.topology.IRichSpout;
    import org.apache.storm.topology.OutputFieldsDeclarer;
    import org.apache.storm.spout.SpoutOutputCollector;
    import org.apache.storm.task.TopologyContext;

    //Create a class FakeLogReaderSpout which implement IRichSpout interface to access functionalities

    public class FakeCallLogReaderSpout implements IRichSpout {
        //Create instance for SpoutOutputCollector which passes tuples to bolt.
       private SpoutOutputCollector collector;
       private boolean completed = false;

       //Create instance for TopologyContext which contains topology data.
       private TopologyContext context;

       //Create instance for Random class.
       private Random randomGenerator = new Random();
       private Integer idx = 0;

        @Override
           public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
              this.context = context;
              this.collector = collector;
           }

           @Override
               public void nextTuple() {
                  if(this.idx <= 1000) {
                         List<String> mobileNumbers = new ArrayList<String>();
                             mobileNumbers.add("1234123401");
                             mobileNumbers.add("1234123402");
                             mobileNumbers.add("1234123403");
                             mobileNumbers.add("1234123404");
                             Integer localIdx = 0;
                             while(localIdx++ < 100 && this.idx++ < 1000) {
                                    String fromMobileNumber = mobileNumbers.get(randomGenerator.nextInt(4));
                                    String toMobileNumber = mobileNumbers.get(randomGenerator.nextInt(4));

                                        while(fromMobileNumber == toMobileNumber) {
                                               toMobileNumber = mobileNumbers.get(randomGenerator.nextInt(4));
                                }

                                Integer duration = randomGenerator.nextInt(60);
                                this.collector.emit(new Values(fromMobileNumber, toMobileNumber, duration));
                     }
                  }
               }

           @Override
             public void declareOutputFields(OutputFieldsDeclarer declarer) {
              declarer.declare(new Fields("from", "to", "duration"));
               }    

           //Override all the interface methods
           @Override
           public void close() {}

           public boolean isDistributed() {
              return false;
           }

           @Override
           public void activate() {}

           @Override 
           public void deactivate() {}

           @Override
           public void ack(Object msgId) {}

           @Override
           public void fail(Object msgId) {}

           @Override
           public Map<String, Object> getComponentConfiguration() {
              return null;
           }
        }
  ```
* 创建 Bolt 组件
Bolt 需要继承 IRichBolt 接口， 接口描述如下
    > prepare − Provides the bolt with an environment to execute. The executors will run this method to initialize the spout.
    > execute − Process a single tuple of input.
    > cleanup − Called when a bolt is going to shutdown.
    > declareOutputFields − Declares the output schema of the tuple.
  ```java
  //import util packages
  import java.util.HashMap;
  import java.util.Map;

  import org.apache.storm.tuple.Fields;
  import org.apache.storm.tuple.Values;
  import org.apache.storm.task.OutputCollector;
  import org.apache.storm.task.TopologyContext;

  //import Storm IRichBolt package
  import org.apache.storm.topology.IRichBolt;
  import org.apache.storm.topology.OutputFieldsDeclarer;
  import org.apache.storm.tuple.Tuple;

  //Create a class CallLogCreatorBolt which implement IRichBolt interface
  public class CallLogCreatorBolt implements IRichBolt {

    //Create instance for OutputCollector which collects and emits tuples to produce output
    private OutputCollector collector;

    @Override
      public void prepare(Map conf, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
      }

    @Override
      public void execute(Tuple tuple) {
        String from = tuple.getString(0);
        String to = tuple.getString(1);
        Integer duration = tuple.getInteger(2);
        collector.emit(new Values(from + " - " + to, duration));
      }


    @Override
      public void cleanup() {}

    @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("call", "duration"));
      }

    @Override
      public Map<String, Object> getComponentConfiguration() {
        return null;
      }
  }

  ```


  ```java
    import java.util.HashMap;
    import java.util.Map;
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Values;
    import org.apache.storm.task.OutputCollector;
    import org.apache.storm.task.TopologyContext;
    import org.apache.storm.topology.IRichBolt;    
    import org.apache.storm.topology.OutputFieldsDeclarer;
    import org.apache.storm.tuple.Tuple;

    public class CallLogCounterBolt implements IRichBolt {
       Map<String, Integer> counterMap;
       private OutputCollector collector;

       @Override
       public void prepare(Map conf, TopologyContext context, OutputCollector collector) {
          this.counterMap = new HashMap<String, Integer>();
          this.collector = collector;
       }

       @Override
       public void execute(Tuple tuple) {
          String call = tuple.getString(0);
          Integer duration = tuple.getInteger(1);

          if(!counterMap.containsKey(call)){
             counterMap.put(call, 1);
          }else{
             Integer c = counterMap.get(call) + 1;
             counterMap.put(call, c);
          }

          collector.ack(tuple);
       }

       @Override
       public void cleanup() {
          for(Map.Entry<String, Integer> entry:counterMap.entrySet()){
             System.out.println(entry.getKey()+" : " + entry.getValue());
          }
       }

       @Override
       public void declareOutputFields(OutputFieldsDeclarer declarer) {
          declarer.declare(new Fields("call"));
       }

       @Override
       public Map<String, Object> getComponentConfiguration() {
          return null;
       }
    }
  ```

* 创建 Topology  和 Local Cluster
  ```java
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Values;
    //import storm configuration packages
    import org.apache.storm.Config;
    import org.apache.storm.LocalCluster;
    import org.apache.storm.topology.TopologyBuilder;

    //Create main class LogAnalyserStorm submit topology.
    public class LogAnalyserStorm {
       public static void main(String[] args) throws Exception{
          //Create Config instance for cluster configuration
          Config config = new Config();
          config.setDebug(true);

          TopologyBuilder builder = new TopologyBuilder();
          builder.setSpout("call-log-reader-spout", new FakeCallLogReaderSpout());

          builder.setBolt("call-log-creator-bolt", new CallLogCreatorBolt())
             .shuffleGrouping("call-log-reader-spout");

          builder.setBolt("call-log-counter-bolt", new CallLogCounterBolt())
             .fieldsGrouping("call-log-creator-bolt", new Fields("call"));

          LocalCluster cluster = new LocalCluster();
          cluster.submitTopology("LogAnalyserStorm", config, builder.createTopology());
          Thread.sleep(10000);

          //Stop the topology

          cluster.shutdown();
       }
    }

```

* 远程模式
http://storm.apache.org/releases/current/Distributed-RPC.html
* 编译并运行应用
```shell
$ cd /opt/storm/my-example
$ javac  *.java
$ java LogAnalyserStorm
```
* 输出结果
```shell
1234123402 - 1234123401 : 78
1234123402 - 1234123404 : 88
1234123402 - 1234123403 : 105
1234123401 - 1234123404 : 74
1234123401 - 1234123403 : 81
1234123401 - 1234123402 : 81
1234123403 - 1234123404 : 86
1234123404 - 1234123401 : 63
1234123404 - 1234123402 : 82
1234123403 - 1234123402 : 83
1234123404 - 1234123403 : 86
1234123403 - 1234123401 : 93
```

## 参考
[Storm 官网](http://storm.apache.org/) : http://storm.apache.org
[教程](https://www.tutorialspoint.com/apache_storm/index.htm) : https://www.tutorialspoint.com/apache_storm/index.htm
[Storm-Java Doc](http://storm.apache.org/releases/current/javadocs/index.html) http://storm.apache.org/releases/current/javadocs/index.html

* PDF
《Storm Applied》
《Getting Started with Storm》
《Storm Real-time Processing Cookbook》
《Learning Storm》
《Storm Blueprints:Patterns for Distributed Real-time Computation》
《Hadoop The Definitive Guide》



