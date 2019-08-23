---
title: Storm与Redis整合
date: 2016-11-07 09:11:42
tags:
- 大数据
---

![](/assets/blog-image/storm-redis-t.jpg)
由 王宇 原创并发布 ：

## 参考资料：http://storm.apache.org/releases/2.0.0-SNAPSHOT/storm-redis.html
## Strom为Redis提供几个标准Bolt
>* <font color=red>RedisLookupBolt</font>: 查询  
>* <font color=red>RedisStoreBolt</font>: 存储
>* <font color=red>AbstractRedisBolt</font>: 存储
## RedisLookupBolt 例子
从Redis中，查询单词的计算数量
<!--more-->
```java
class WordCountRedisLookupMapper implements RedisLookupMapper {
    private RedisDataTypeDescription description;
    private final String hashKey = "wordCount";

    public WordCountRedisLookupMapper() {
        description = new RedisDataTypeDescription(
                RedisDataTypeDescription.RedisDataType.HASH, hashKey);
    }

    @Override
    public List<Values> toTuple(ITuple input, Object value) {
        String member = getKeyFromTuple(input);
        List<Values> values = Lists.newArrayList();
        values.add(new Values(member, value));
        return values;
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("wordName", "count"));
    }

    @Override
    public RedisDataTypeDescription getDataTypeDescription() {
        return description;
    }

    @Override
    public String getKeyFromTuple(ITuple tuple) {
        return tuple.getStringByField("word");
    }

    @Override
    public String getValueFromTuple(ITuple tuple) {
        return null;
    }
}

JedisPoolConfig poolConfig = new JedisPoolConfig.Builder()
        .setHost(host).setPort(port).build();
RedisLookupMapper lookupMapper = new WordCountRedisLookupMapper();
RedisLookupBolt lookupBolt = new RedisLookupBolt(poolConfig, lookupMapper);

```
* 对例子理解
![](/assets/blog-image/storm-redis-1.jpg)
如上图，JedisPoolConfig的作用是，提供Redis相关的配置，给RedisLookupBolt
RedisLookupMapper的作用是，绘制相应的数据结构给RedisLookupBolt.
 例如 
    * getKeyFromTuple()方法， 告诉RedisLookupBolt,从输入的Tuple中，取什么样子的Key，让它去Redis中查询。
    * declareOutputFields()的作用是，定义RedisLookupBolt的输出格式。
## RedisStoreBoltBolt 例子
```java
class WordCountStoreMapper implements RedisStoreMapper {
    private RedisDataTypeDescription description;
    private final String hashKey = "wordCount";

    public WordCountStoreMapper() {
        description = new RedisDataTypeDescription(
            RedisDataTypeDescription.RedisDataType.HASH, hashKey);
    }

    @Override
    public RedisDataTypeDescription getDataTypeDescription() {
        return description;
    }

    @Override
    public String getKeyFromTuple(ITuple tuple) {
        return tuple.getStringByField("word");
    }

    @Override
    public String getValueFromTuple(ITuple tuple) {
        return tuple.getStringByField("count");
    }
}

JedisPoolConfig poolConfig = new JedisPoolConfig.Builder()
                .setHost(host).setPort(port).build();

RedisStoreMapper storeMapper = new WordCountStoreMapper();

RedisStoreBolt storeBolt = new RedisStoreBolt(poolConfig, storeMapper);
```

## 非简单Bolt
采用 AbstractRedisBolt 实现更为复杂的逻辑
```java
public static class LookupWordTotalCountBolt extends AbstractRedisBolt {
        private static final Logger LOG = LoggerFactory.getLogger(LookupWordTotalCountBolt.class);
        private static final Random RANDOM = new Random();

        public LookupWordTotalCountBolt(JedisPoolConfig config) {
            super(config);
        }

        public LookupWordTotalCountBolt(JedisClusterConfig config) {
            super(config);
        }

        @Override
        public void execute(Tuple input) {
            JedisCommands jedisCommands = null;
            try {
                jedisCommands = getInstance();
                String wordName = input.getStringByField("word");
                String countStr = jedisCommands.get(wordName);
                if (countStr != null) {
                    int count = Integer.parseInt(countStr);
                    this.collector.emit(new Values(wordName, count));

                    // print lookup result with low probability
                    if(RANDOM.nextInt(1000) > 995) {
                        LOG.info("Lookup result - word : " + wordName + " / count : " + count);
                    }
                } else {
                    // skip
                    LOG.warn("Word not found in Redis - word : " + wordName);
                }
            } finally {
                if (jedisCommands != null) {
                    returnInstance(jedisCommands);
                }
                this.collector.ack(input);
            }
        }

        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            // wordName, count
            declarer.declare(new Fields("wordName", "count"));
        }
    }
```

## RedisLookupBolt 源代码，用于理解WordCountRedisLookupMapper
注意下列代码中的：
* get
```java
package org.apache.storm.redis.bolt;

import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;
import org.apache.storm.redis.common.mapper.RedisDataTypeDescription;
import org.apache.storm.redis.common.mapper.RedisLookupMapper;
import org.apache.storm.redis.common.config.JedisClusterConfig;
import org.apache.storm.redis.common.config.JedisPoolConfig;
import redis.clients.jedis.JedisCommands;
import java.util.List;

/**

 * Basic bolt for querying from Redis and emits response as tuple.

 * <p/>

 * Various data types are supported: STRING, LIST, HASH, SET, SORTED_SET, HYPER_LOG_LOG, GEO

 */
public class RedisLookupBolt extends AbstractRedisBolt {
    private final RedisLookupMapper lookupMapper;
    private final RedisDataTypeDescription.RedisDataType dataType;
    private final String additionalKey;

    /**
     * Constructor for single Redis environment (JedisPool)
     * @param config configuration for initializing JedisPool
     * @param lookupMapper mapper containing which datatype, query key, output key that Bolt uses
     */
    public RedisLookupBolt(JedisPoolConfig config, RedisLookupMapper lookupMapper) {
        super(config);
        this.lookupMapper = lookupMapper;

        RedisDataTypeDescription dataTypeDescription = lookupMapper.getDataTypeDescription();
        this.dataType = dataTypeDescription.getDataType();
        this.additionalKey = dataTypeDescription.getAdditionalKey();
    }


    /**
     * Constructor for Redis Cluster environment (JedisCluster)
     * @param config configuration for initializing JedisCluster
     * @param lookupMapper mapper containing which datatype, query key, output key that Bolt uses
     */
    public RedisLookupBolt(JedisClusterConfig config, RedisLookupMapper lookupMapper) {
        super(config);

        this.lookupMapper = lookupMapper;

        RedisDataTypeDescription dataTypeDescription = lookupMapper.getDataTypeDescription();
        this.dataType = dataTypeDescription.getDataType();
        this.additionalKey = dataTypeDescription.getAdditionalKey();
    }

    /**

     * {@inheritDoc}

     */
    @Override
    public void execute(Tuple input) {
        String key = lookupMapper.getKeyFromTuple(input);
        Object lookupValue;

        JedisCommands jedisCommand = null;
        try {
            jedisCommand = getInstance();
            switch (dataType) {
                case STRING:
                    lookupValue = jedisCommand.get(key);
                    break;

                case LIST:
                    lookupValue = jedisCommand.lpop(key);
                    break;

                case HASH:
                    lookupValue = jedisCommand.hget(additionalKey, key);
                    break;

                case SET:
                    lookupValue = jedisCommand.scard(key);
                    break;

                case SORTED_SET:
                    lookupValue = jedisCommand.zscore(additionalKey, key);
                    break;

                case HYPER_LOG_LOG:
                    lookupValue = jedisCommand.pfcount(key);
                    break;

                case GEO:
                    lookupValue = jedisCommand.geopos(additionalKey, key);
                    break;

                default:
                    throw new IllegalArgumentException("Cannot process such data type: " + dataType);
            }

            List<Values> values = lookupMapper.toTuple(input, lookupValue);
            for (Values value : values) {
                collector.emit(input, value);
            }
            collector.ack(input);

        } catch (Exception e) {
            this.collector.reportError(e);
            this.collector.fail(input);
        } finally {
            returnInstance(jedisCommand);
        }
    }

    /**

     * {@inheritDoc}

     */

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        lookupMapper.declareOutputFields(declarer);
    }
}
```

## Kafka + Storm + Redis 完整例子
* 场景
    参考 《Storm与Kafka整合记录》中，统计单词数量的例子。在这个例子基础之上，将统计到单词结果保存到Redis中。
* 修改Topology，增加RedisStoreBolt

```java
public class KafkaStormSample {
	public static void main(String[] args) throws Exception {
		Config config = new Config();
		config.setDebug(true);
		config.put(Config.TOPOLOGY_MAX_SPOUT_PENDING, 1);
		String zkConnString = "localhost:2181";
		String topic = "my-first-topic";
		BrokerHosts hosts = new ZkHosts(zkConnString);

		SpoutConfig kafkaSpoutConfig = new SpoutConfig(hosts, topic, "/" + topic, UUID.randomUUID().toString());
		kafkaSpoutConfig.bufferSizeBytes = 1024 * 1024 * 4;
		kafkaSpoutConfig.fetchSizeBytes = 1024 * 1024 * 4;
		// kafkaSpoutConfig.forceFromStart = true;
		kafkaSpoutConfig.scheme = new SchemeAsMultiScheme(new StringScheme());

		// Creating RedisStoreBolt
		String host = "localhost";
		int port = 6379;

		JedisPoolConfig poolConfig = new JedisPoolConfig.Builder().setHost(host).setPort(port).build();

		RedisStoreMapper storeMapper = new WordCountStoreMapper();
		RedisStoreBolt storeBolt = new RedisStoreBolt(poolConfig, storeMapper);

		// Assemble with topology
		TopologyBuilder builder = new TopologyBuilder();
		builder.setSpout("kafka-spout", new KafkaSpout(kafkaSpoutConfig));
		builder.setBolt("word-spitter", new SplitBolt()).shuffleGrouping("kafka-spout");
		builder.setBolt("word-counter", new CountBolt()).shuffleGrouping("word-spitter");
		builder.setBolt("redis-store-bolt", storeBolt).shuffleGrouping("word-counter");

		LocalCluster cluster = new LocalCluster();
		cluster.submitTopology("KafkaStormSample", config, builder.createTopology());

		Thread.sleep(10000);
		cluster.shutdown();
	}
}

```
* 修改 CountBolt, emit to RedisStoreBolt
```java
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

      // emit to redis-store-bolt
      Integer result = counters.get(str);
      this.collector.emit(new Values(str, String.valueOf(result)));

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
      declarer.declare(new Fields("word", "count"));
   }

   @Override
   public Map<String, Object> getComponentConfiguration() {
      return null;
   }
}

```
## 依赖包
jedis-2.9.0.jar
commons-pool2-2.4.2.jar
将以上两个包，增加到CLASSPATH路径中
## 编译执行
* 启动服务
    * 启动 zookeeper
        ```shell
            $ cd /opt/zookeeper
            $ ./bin/zkServer.sh start
        ```
    * 启动 redis

        ```shell
            $ redis-server
        ```
    * 启动 kafka
        ```shell
           $ cd /opt/kafka
           $ ./bin/kafka-server-start.sh config/server.properties
        ```    
    * 启动 storm
        ```shell         
            $ cd /opt/storm
            $./bin/storm nimbus
            $./bin/storm supervisor
        ```
* 输入数据
```shell
$cd /opt/kafka
$./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-first-topic
Hello
My first message
My second message
```
* 执行例子
```shell
    $ java KafkaStormSample
```

* 在Redis-cli 中查询结果

```shell
$ redis-cli
127.0.0.1:6379> hvals wordCount
```






