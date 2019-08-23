---
title: Kafka与Storm整合
date: 2016-12-02 09:12:00
tags:
- 大数据
---

![](/assets/blog-image/kafka-storm-t.jpg)
由 王宇 原创并发布 ：

## 整合Storm的概念

* 整合场景

 > A spout is a source of streams. For example, <font color=red>a spout may read tuples off a Kafka Topic</font> and emit them as a stream. <font color=red>A bolt consumes input streams, process and possibly emits new streams.</font> Bolts can do anything from running functions, filtering tuples, do streaming aggregations, streaming joins, <font color=red>talk to databases</font>, and more. Each node in a Storm topology executes in parallel. A topology runs indefinitely until you terminate it. Storm will automatically reassign any failed tasks. Additionally, Storm guarantees that there will be no data loss, even if the machines go down and messages are dropped.
<!--more-->
* BrokerHosts - ZkHosts & StaticHosts
BrokerHosts is an interface and ZkHosts and StaticHosts are its two main implementations. ZkHosts is used to track the Kafka brokers dynamically by maintaining the details in ZooKeeper, while StaticHosts is used to manually / statically set the Kafka brokers and its details. ZkHosts is the simple and fast way to access the Kafka broker

* KafkaConfig API
This API is used to <font color=red>define configuration settings for the Kafka cluster</font>. The signature of Kafka Con-fig is defined as follows
public KafkaConfig(BrokerHosts hosts, string topic)

> Hosts − The BrokerHosts can be ZkHosts / StaticHosts.
> Topic − topic name

* SpoutConfig API
Spoutconfig is an <font color=red>extension of KafkaConfig</font> that supports additional ZooKeeper information.

public SpoutConfig(BrokerHosts hosts, string topic, string zkRoot, string id)
>* Hosts − The BrokerHosts can be any implementation of BrokerHosts interface
>* Topic − topic name.
>* zkRoot − ZooKeeper root path.
>* id − The spout stores the state of the offsets its consumed in Zookeeper. The id should uniquely identify your spout.

* SchemeAsMultiScheme (调度)????
* KafkaSpout API
<font color=red>KafkaSpout is our spout implementation, which will integrate with Storm. It fetches the messages from kafka topic and emits it into Storm ecosystem as tuples. KafkaSpout get its config-uration details from SpoutConfig.</font>
```java
// ZooKeeper connection string
BrokerHosts hosts = new ZkHosts(zkConnString);


//Creating SpoutConfig Object
SpoutConfig spoutConfig = new SpoutConfig(hosts, 

   topicName, "/" + topicName UUID.randomUUID().toString());

//convert the ByteBuffer to String.
spoutConfig.scheme = new SchemeAsMultiScheme(new StringScheme());

//Assign SpoutConfig to KafkaSpout.
KafkaSpout kafkaSpout = new KafkaSpout(spoutConfig);

```

## 创建 Bolt
* Bolt 接口定义参考《Storm学习笔记》
* 分割统计单词个数的Bolt例子
>* SplitBolt.java
```java
   import java.util.Map;
   import org.apache.storm.tuple.Tuple;
   import org.apache.storm.tuple.Fields;
   import org.apache.storm.tuple.Values;
   import org.apache.storm.task.OutputCollector;
   import org.apache.storm.topology.OutputFieldsDeclarer;
   import org.apache.storm.topology.IRichBolt;
   import org.apache.storm.task.TopologyContext;

   public class SplitBolt implements IRichBolt {

      private OutputCollector collector;

      @Override
      public void prepare(Map stormConf, TopologyContext context,

         OutputCollector collector) {

         this.collector = collector;

      }

      @Override
      public void execute(Tuple input) {
         String sentence = input.getString(0);
         String[] words = sentence.split(" ");

         for(String word: words) {

            word = word.trim();

            if(!word.isEmpty()) {
               word = word.toLowerCase();
               collector.emit(new Values(word));
            }
         }
         collector.ack(input);
      }

      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
         declarer.declare(new Fields("word"));
      }

      @Override
      public void cleanup() {}

      @Override
      public Map<String, Object> getComponentConfiguration() {
         return null;
      }
   }

```

>* CountBolt.java
```java

   import java.util.Map;
   import java.util.HashMap;
   import org.apache.storm.tuple.Tuple;
   import org.apache.storm.task.OutputCollector;
   import org.apache.storm.topology.OutputFieldsDeclarer;
   import org.apache.storm.topology.IRichBolt;
   import org.apache.storm.task.TopologyContext;

   public class CountBolt implements IRichBolt{
      Map<String, Integer> counters;
      private OutputCollector collector;

      @Override
      public void prepare(Map stormConf, TopologyContext context,

      OutputCollector collector) {
         this.counters = new HashMap<String, Integer>();
         this.collector = collector;
      }

      @Override
      public void execute(Tuple input) {
         String str = input.getString(0);
         if(!counters.containsKey(str)){
            counters.put(str, 1);
         }else {
            Integer c = counters.get(str) +1;
            counters.put(str, c);
         }

         collector.ack(input);
      }

      @Override
      public void cleanup() {
         for(Map.Entry<String, Integer> entry:counters.entrySet()){
            System.out.println(entry.getKey()+" : " + entry.getValue());
         }
      }
   

      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {

      }

      @Override
      public Map<String, Object> getComponentConfiguration() {
         return null;
      }
   }

```



## 提交到 Topology
* Topology概念参考《Storm学习笔记》
* KafkaStormSample.java
```java
   import org.apache.storm.Config;
   import org.apache.storm.LocalCluster;
   import org.apache.storm.topology.TopologyBuilder;
   import java.util.ArrayList;
   import java.util.List;
   import java.util.UUID;
   import org.apache.storm.spout.SchemeAsMultiScheme;
   import org.apache.storm.kafka.trident.GlobalPartitionInformation;
   import org.apache.storm.kafka.ZkHosts;
   import org.apache.storm.kafka.Broker;
   import org.apache.storm.kafka.StaticHosts;
   import org.apache.storm.kafka.BrokerHosts;
   import org.apache.storm.kafka.SpoutConfig;
   import org.apache.storm.kafka.KafkaConfig;
   import org.apache.storm.kafka.KafkaSpout;
   import org.apache.storm.kafka.StringScheme;

   public class KafkaStormSample {
      public static void main(String[] args) throws Exception{
         Config config = new Config();
         config.setDebug(true);
         config.put(Config.TOPOLOGY_MAX_SPOUT_PENDING, 1);
         String zkConnString = "localhost:2181";
         String topic = "my-first-topic";
         BrokerHosts hosts = new ZkHosts(zkConnString);

         SpoutConfig kafkaSpoutConfig = new SpoutConfig (hosts, topic, "/" + topic, UUID.randomUUID().toString());
         kafkaSpoutConfig.bufferSizeBytes = 1024 * 1024 * 4;
         kafkaSpoutConfig.fetchSizeBytes = 1024 * 1024 * 4;

         // kafkaSpoutConfig.forceFromStart = true;
         kafkaSpoutConfig.scheme = new SchemeAsMultiScheme(new StringScheme());
         TopologyBuilder builder = new TopologyBuilder();

         builder.setSpout("kafka-spout", new KafkaSpout(kafkaSpoutConfig));
         builder.setBolt("word-spitter", new SplitBolt()).shuffleGrouping("kafka-spout");
         builder.setBolt("word-counter", new CountBolt()).shuffleGrouping("word-spitter");

         LocalCluster cluster = new LocalCluster();
         cluster.submitTopology("KafkaStormSample", config, builder.createTopology());

         Thread.sleep(10000);
         cluster.shutdown();
      }
   }
```

## 版本
* Zookeeper: zookeeper-3.5.2-alpha.tar.gz
* Curator: <font color=red>2.9.1</font>
* SLF4j: slf4j-1.7.21.tar.gz
* Kafka: kafka_2.11-0.10.1.0
* Storm : apache-storm-1.0.2.tar.gz
* JSON: json-simple-1.1.1.jar
* JDK: 1.8.0

## 编译
* 依赖包
 >* Curator
Before moving compilation, Kakfa-Storm integration <font color=red>needs curator ZooKeeper client java library</font>. 这个包的路径，要加到CLASSPATH中 
<font color=red>
curator-client-2.9.1.jar
curator-framework-2.9.1.jar
</font>

>* JSON
 json-simple-1.1.1.jar
>* Storm-Kafka
json-simple-1.1.1.jar
>* Kafka Lib
>* Storm Lib
>* SLF4J
当CLASSPATH中，包含了Kafka 和Storm 的lib 后， SLF4j会产生冲突，将Kafka中的SLF4j 移除CLASSPATH即可

* 编译命令

 ```shell

 $ javac *.java

 ```



## 执行

* 开启服务： Zookeeper Kafka Storm
```shell
$cd /opt/zookeeper
$.bin/zkServer.sh start

$cd /opt/kafka
$./bin/kafaka-server-start.sh config/server.properties

$ cd /opt/storm
$./bin/storm nimbus
$./bin/storm supervisor
```

* 创建Topic:  "my-first-topic"
```shell
$ ./bin/kafktopics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic my-first-topic
$ ./bin/kafktopics.sh --list --zookeeper localhost:2181
```

* 在Kafka  Producer CLI 输入message
```shell
$ ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-first-topic
hello
kafka
storm
spark
test message
anther test message
```

* 执行例子
```shell
$ java KafkaStormSample
```

* 输出结果
```shell
storm : 1
test : 2
spark : 1
another : 1
kafka : 1
hello : 1
message : 2
```




