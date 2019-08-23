---
title: Kafka学习笔记
date: 2016-10-24 09:12:14
tags:
- 大数据
---

![](/assets/blog-image/kafka-t.jpg)
由 王宇 原创并发布 ：

## 消息系统(Messaging System)
* Point to Point Messaging System
![](/assets/blog-image/kafka-1.jpg)
* Publish-Subscribe Messaging System
![](/assets/blog-image/kafka-2.jpg)
两者最主要的区别是 Publish-Subscribe 模式支持多 Receiver

## Kafka  结构
<!--more-->
![](/assets/blog-image/kafka-3.jpg)

| Components | Description|
|--------------------|:-----------------|
|Topics| A stream of messages belonging to a particular category is called a topic. Data is stored in topics.Topics are split into partitions. For each topic, Kafka keeps a mini-mum of one partition. Each such partition contains messages in an immutable ordered sequence. A partition is implemented as a set of segment files of equal sizes.|
|Partition|Topics may have many partitions, so it can handle an arbitrary amount of data.|
|Partition offset|Each partitioned message has a unique sequence id called as offset.|
|Replicas of partition|Replicas are nothing but backups of a partition. Replicas are never read or write data. They are used to prevent data loss.|
|Brokers|Brokers are simple system responsible for maintaining the pub-lished data. Each broker may have zero or more partitions per topic. Assume, if there are N partitions in a topic and N number of brokers, each broker will have one partition.Assume if there are N partitions in a topic and more than N brokers (n + m), the first N broker will have one partition and the next M broker will not have any partition for that particular topic.Assume if there are N partitions in a topic and less than N brokers (n-m), each broker will have one or more partition sharing among them. This scenario is not recommended due to unequal load distri-bution among the broker.|
|Kafka cluster|Kafka’s having more than one broker are called as Kafka cluster. A Kafka cluster can be expanded without downtime. These clusters are used to manage the persistence and replication of message data.|
|Producers|Producers are the publisher of messages to one or more Kafka topics. Producers send data to Kafka brokers. Every time a producer pub-lishes a message to a broker, the broker simply appends the message to the last segment file. Actually, the message will be appended to a partition. Producer can also send messages to a partition of their choice.|
|Consumers|Consumers read data from brokers. Consumers subscribes to one or more topics and consume published messages by pulling data from the brokers.|
|Leader|Leader is the node responsible for all reads and writes for the given partition. Every partition has one server acting as a leader.|
|Follower|Node which follows leader instructions are called as follower. If the leader fails, one of the follower will automatically become the new leader.A follower acts as normal consumer, pulls messages and up-dates its own data store.|

## 集群结构（Cluster Architecture）
![](/assets/blog-image/kafka-4.jpg)

| Components | Description|
|--------------------|:-----------------|
|Broker|Kafka cluster typically consists of multiple brokers to maintain load balance. Kafka brokers are stateless, so they use ZooKeeper for maintaining their cluster state. One Kafka broker instance can handle hundreds of thousands of reads and writes per second and each bro-ker can handle TB of messages without performance impact. Kafka broker leader election can be done by ZooKeeper.|
|ZooKeeper|ZooKeeper is used for managing and coordinating Kafka broker. ZooKeeper service is mainly used to notify producer and consumer about the presence of any new broker in the Kafka system or failure of the broker in the Kafka system. As per the notification received by the Zookeeper regarding presence or failure of the broker then pro-ducer and consumer takes decision and starts coordinating their task with some other broker.|
|Producers|Producers push data to brokers. When the new broker is started, all the producers search it and automatically sends a message to that new broker. Kafka producer doesn’t wait for acknowledgements from the broker and sends messages as fast as the broker can handle.|
|Consumers|Since Kafka brokers are stateless, which means that the consumer has to maintain how many messages have been consumed by using partition offset. If the consumer acknowledges a particular message offset, it implies that the consumer has consumed all prior messages. The consumer issues an asynchronous pull request to the broker to have a buffer of bytes ready to consume. The consumers can rewind or skip to any point in a partition simply by supplying an offset value. Consumer offset value is notified by ZooKeeper.|

## WorkFlow
Kafka 是一个通过topics分割到一个或多个partitions的collection. 一个Kafka partition 是一个线性有序的messages, 每一个message通过索引标识
* Workflow of Pub-Sub Messaging
    >* Producers send message to a topic at regular intervals.
    >*  Kafka broker stores all messages in the partitions configured for that particular topic. It ensures the messages are equally shared between partitions. If the producer sends two messages and there are two partitions, Kafka will store one message in the first partition and the second message in the second partition.
    >*  Consumer subscribes to a specific topic.
    >* Once the consumer subscribes to a topic, Kafka will provide the current offset of the topic to the consumer and also saves the offset in the Zookeeper ensemble.
    >* Consumer will request the Kafka in a regular interval (like 100 Ms) for new messages.
    >* Once Kafka receives the messages from producers, it forwards these messages to the consumers.
    >* Consumer will receive the message and process it.
    >* Once the messages are processed, consumer will send an acknowledgement to the Kafka broker.
    >* Once Kafka  <font color=red>receives an acknowledgement</font> , it changes the offset to the new value and updates it in the Zookeeper. Since offsets are maintained in the Zookeeper, the consumer can read next message correctly even during server outrages.
    >* This above flow will repeat until the consumer stops the request.
    >* Consumer has the option to rewind/skip to the desired offset of a topic at any time and read all the subsequent messages.
* Workflow of Queue Messaging / Consumer Group
一个consumers组有相同的Group ID 去 Subscribe 一个Topic.
    >* Producers send message to a topic in a regular interval.
    >* Kafka stores all messages in the partitions configured for that particular topic similar to the earlier scenario.
    >* A single consumer subscribes to a specific topic, assume Topic-01 with Group ID as Group-1.
    >* Kafka interacts with the consumer in the same way as Pub-Sub Messaging until new consumer subscribes the same topic, Topic-01 with the same Group ID as Group-1.
    >* Once the new consumer arrives, Kafka switches its operation to<font color=red> share mode and shares the data between the two consumers</font>. This sharing will go on until the number of con-sumers reach the number of partition configured for that particular topic.
    >* Once the number of consumer exceeds the number of partitions, the new consumer will not receive any further message until any one of the existing consumer unsubscribes. This scenario arises because each consumer in Kafka will be assigned a minimum of one partition and once all the partitions are assigned to the existing consumers, the new consumers will have to wait.
    >* This feature is also called as Consumer Group. In the same way, Kafka will provide the best of both the systems in a very simple and efficient manner.
* Role of ZooKeeper
A critical dependency of Apache Kafka is Apache Zookeeper, which is a distributed configuration and synchronization service. Zookeeper serves as the coordination interface between the Kafka brokers and consumers. The Kafka servers share information via a Zookeeper cluster. Kafka stores basic metadata in Zookeeper such as information about topics, brokers, consumer offsets (queue readers) and so on.

## 安装步骤
* 步骤一： 安装JDK 并配置环境变量 JAVA_HOME CLASSPATH
* 步骤二 :  安装ZooKeeper
 >   下载ZooKeeper 
 >    解包
  ```shell
 $ tar xzvf zookeeper-3.5.2-alpha.tar.gz
 $ mv ./zookeeper-3.5.2-alpha /opt/zookeepter
 $ cd  /opt/zookeeper
 $ mkdir data
  ```
  >  创建配置文件
  ```shell
 $ cd /opt/zookeeper
 $ vim conf/zoo.cfg

 tickTime=2000
 dataDir=/opt/zookeeper/data
 clientPort=2181
 initLimit=5
 syncLimit=2
  ```
 > 启动ZooKeeper Seve
  ```shell
 $ bin/zkServer.sh start
  ```
* 步骤三安装 SLF4J
 > 下载 SL4J :  slf4j-1.7.21.tar.gz  www.slf4j.org
 >  解包并配置CLASSPATH
 ```shell
 $ tar xvzf ./slf4j-17.21.tar.gz
 $ mv ./slf4j-17.21 /op/slf4j
 vim ~/.bashrc
 export CLASSPATH = ${CLASSPATH}:/opt/slf4j/*
 ```
* 步骤四： 安装 Kafka
 > 下载 Kafka 
 >  解包
 ```shell
 $ tar xzvf kafka_2.11-0.10.1.0.tgz
 $ mv ./kafka_2.11-0.10.1.0 /opt/kafka
 $ cd /opt/kafka
 ```
 > 启动服务
 ```shell
 $ ./bin/kafka-server-start.sh config/server.properties
 ```
* 步骤五： 停止服务
 ```
 $ ./bin/kafka-server-stop.sh config/server.properties
 ```
## Kafka 基本操作
* 启动 ZooKeeper 和 Kafka
```shell
$ ./bin/zkServer.sh start
$ ./bin/kafka-server-start.sh config/server.properties
```

* 单节点Broker 配置

>* 创建一个 Kafka Topic
```shell
 语法：
 $ ./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-fator 1 --partitions 1 --topic  topic-name
 例子：
 $./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic Hello-Kafka
 输出：
  Created topic "Hello-Kafka"
```
>* List of Topics
```shell 
语法：
 $ ./bin/kafka-topics.sh --list --zookeeper localhost:2181
 输出：
 Hello-Kafka 
```
>* 启动 Producer 去发 Messages
```shell
 语法：
 $ ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic  topic-name
 例子：
 $./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic Hello-Kafka
 命令行下手动输入如下信息：
    Hello
    My first message
    My second message
```
>* 启动 Consumer 去接受 Messages
```shell   
 语法：
   $ ./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic topic-name --from-beginning
  例子：
   $ ./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic Hello-Kafka --from-beginning 
   输出：
    Hello
    My first message
    My second message
```
* 多节点Broker配置
<font color=red>待续</font>
* 基本Topic 操作

 >* 修改 Topic

  ```shell
    语法：
    $ ./bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic topic-name --parti-tions count
    例子：
    $ ./bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic Hello-kafka --parti-tions 2
    输出：
     WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affectedAdding partitions succeeded!
  ```

 >* 删除 Topic

  ```shell
    语法：
    $ ./bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic topic-name
    例子：
    $ ./bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic Hello-kafka 
    输出：
    Topic Hello-kafka marked for deletion
  ``` 

>* 删除数据

  ```shell
  Stop the Apache Kafka daemon
  Delete the topic data folder: rm -rf /tmp/kafka-logs/MyTopic-0
  ```
## Simple Producer Example
* Kafka Producer API
  >* KafkaProducer class provides send method to send messages <font color=red>asynchronously</font> to a topic. The signature of send() is as follows
 ```java
producer.send(new ProducerRecord<byte[],byte[]>(topic, partition, key1, value1) , callback);
 ```
 >* ProducerRecord − The producer manages <font color=red>a buffer of records waiting to be sent</font>.
 >* Callback − A user-supplied callback to execute when the record has been acknowl-edged by the server (null indicates no callback).
 >* KafkaProducer class provides a flush method to <font color=red>ensure all previously sent messages have been actually completed. </font>Syntax of the flush method is as follows −
 ```java
 public void flush()
 ```
 >* KafkaProducer class provides partitionFor method, which helps in getting the partition metadata for a given topic. This can be used for custom partitioning. The signature of this method is as follows −
 ```java
 public Map metrics()
 It returns the map of internal metrics maintained by the producer.
 ```
 >* public void close() − KafkaProducer class provides close method blocks until all previously sent requests are completed.
* Producer API
 >* Send

    ```java
    public void send(KeyedMessaget<k,v> message) - sends the data to a single topic,par-titioned by key using either sync or async producer.
    public void send(List<KeyedMessage<k,v>>messages)- sends data to multiple topics.
    Properties prop = new Properties();
    prop.put(producer.type,”async”)
    ProducerConfig config = new ProducerConfig(prop);
    There are two types of producers – Sync and Async.
    ```
 >* Close
 public void close()
Producer class provides close method to close the producer pool connections to all Kafka bro-kers.
* Configuration Settings
   |name|Description|
   |------------|:------------|
   |client.id|identifies producer application |
   |producer.type|either sync or async|
   |acks|The acks config controls the criteria under producer requests are con-sidered complete.|
   |retries|If producer request fails, then automatically retry with specific value.|
   |bootstrap.servers|bootstrapping list of brokers.|
   |linger.ms|if you want to reduce the number of requests you can set linger.ms to something greater than some value.|
   |key.serializer|Key for the serializer interface.|
   |value.serializer|value for the serializer interface|
   |batch.size|Buffer size.|
   |buffer.memory|controls the total amount of memory available to the producer for buff-ering.|
* ProducerRecord API
 public ProducerRecord (string topic, int partition, k key, v value)

   >* Topic − user defined topic name that will appended to record.
   >* Partition − partition count
   >* Key − The key that will be included in the record.
   >* Value − Record contents

 public ProducerRecord (string topic, k key, v value)
    ProducerRecord class constructor is used to create a record with key, value pairs and without partition.

    >* Topic − Create a topic to assign record.
    >* Key − key for the record.
    >* Value − record contents.

 public ProducerRecord (string topic, v value)
    ProducerRecord class creates a record without partition and key.
    >* Topic − create a topic.
    >* Value − record contents.

    |Class Method|Description|
    |---|:-----|
    |public string topic()|Topic will append to the record.|
    |public K key()|Key that will be included in the record. If no such key, null will be re-turned here.|
    |public V value()|Record contents.|
    |partition()|Partition count for the record|

* SimpleProducer 应用实例
  >* 在创建SimpleProducer之前，采用Kafka的命令创建Topic
 ```java
   //import util.properties packages
   import java.util.Properties;

   //import simple producer packages
   import org.apache.kafka.clients.producer.Producer;

   //import KafkaProducer packages
   import org.apache.kafka.clients.producer.KafkaProducer;

   //import ProducerRecord packages
   import org.apache.kafka.clients.producer.ProducerRecord;

   //Create java class named “SimpleProducer”
   public class SimpleProducer {
      public static void main(String[] args) throws Exception{
         // Check arguments length value
         if(args.length == 0){
            System.out.println("Enter topic name");
            return;
         }

         //Assign topicName to string variable
         String topicName = args[0].toString();

         // create instance for properties to access producer configs   
         Properties props = new Properties();

         //Assign localhost id
         props.put("bootstrap.servers", "localhost:9092");

         //Set acknowledgements for producer requests.      
         props.put("acks", "all");

         //If the request fails, the producer can automatically retry,
         props.put("retries", 0);

         //Specify buffer size in config
         props.put("batch.size", 16384);

         //Reduce the no of requests less than 0   
         props.put("linger.ms", 1);

         //The buffer.memory controls the total amount of memory available to the producer for buffering.   
         props.put("buffer.memory", 33554432);

         props.put("key.serializer", 
            "org.apache.kafka.common.serialization.StringSerializer");

         props.put("value.serializer", 
            "org.apache.kafka.common.serialization.StringSerializer");

         Producer<String, String> producer = new KafkaProducer <String, String>(props);

         for(int i = 0; i < 10; i++)
             producer.send(new ProducerRecord<String, String>(topicName, 
                         Integer.toString(i), Integer.toString(i)));

         System.out.println("Message sent successfully");
         producer.close();
      }
   }
 ```

 >* 编译
    ```shell
        $ javac SimpleProducer 
    ```
 >* 执行
    ```shell
        $ java SimpleProducer <topic-name>
    ```
 >* 输出结果
    ```shell
     Message sent successuflly
     执行下列语句，接收数据
     $ ./bin/kafka-console-consumer.sh --zookeeper localhost:2181 -topic <topic-name> --from-beginning
    ```
## Simple Consumer Example
* KafkaConsumer class
* ConsumerRecordAPI
* ConsumerRecords API
* Conifguration Settings
* SimpleConsumer 应用实例
    >* 接收Topic数据代码
   ```java
   import java.util.Properties;
   import java.util.Arrays;
   import org.apache.kafka.clients.consumer.KafkaConsumer;
   import org.apache.kafka.clients.consumer.ConsumerRecords;
   import org.apache.kafka.clients.consumer.ConsumerRecord;

   public class SimpleConsumer {
      public static void main(String[] args) throws Exception {
         if(args.length == 0){
            System.out.println("Enter topic name");
            return;
         }

         //Kafka consumer configuration settings
         String topicName = args[0].toString();
         Properties props = new Properties();

         props.put("bootstrap.servers", "localhost:9092");
         props.put("group.id", "test");
         props.put("enable.auto.commit", "true");
         props.put("auto.commit.interval.ms", "1000");
         props.put("session.timeout.ms", "30000");
         props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
         props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
         KafkaConsumer<String, String> consumer = new KafkaConsumer <String, String>(props);

         //Kafka Consumer subscribes list of topics here.
         consumer.subscribe(Arrays.asList(topicName));

         //print the topic name
         System.out.println("Subscribed to topic " + topicName);
         int i = 0;

         while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);

            for (ConsumerRecord<String, String> record : records)
            // print the offset,key and value for the consumer records.
            System.out.printf("offset = %d, key = %s, value = %s\n", 
               record.offset(), record.key(), record.value());
         }
      }
   }
   ```
 >* 编译
    ```shell
    $ javac SimpleConsumer
    ```
 >* 执行
    ```shell
    $ java SimpleConsumer <topic-name>
    ```
 >* 输出结果
    ```shell
    Subscribed to topic Hello-Kafka
    offset = 3, key = null, value = Hello Consumer
    ```
## 参考
Useful Links on Apache Kafka
[Apache Kafka Official Website](http://kafka.apache.org/) − Official Website for Apache Kafka
http://kafka.apache.org/
[Apache Kafka Wiki](https://en.wikipedia.org/wiki/Apache_Kafka) − Wikipedia Reference for Apache Kafka
https://en.wikipedia.org/wiki/Apache_Kafka






