---
title: Redis学习笔记
date: 2016-04-14 09:11:07
tags:
- 数据库
---

![](/assets/blog-image/redis-t.jpg)
由 王宇 原创并发布 ：


## Redis 环境

* 安装Redis
```shell
$sudo apt-get update
$sudo apt-get install redis-server
```

* 启动Redis
```shell
$ redis-server
```

* 检察Redis 是否工作正常
```shell
$ redis-cli
redis 127.0.0.1:6379> ping
PONG
```
<!--more-->

* 安装Redis 桌面管理工具

[下载地址](http://redisdesktop.com/download)：http://redisdesktop.com/download

## 配置Redis

* 配置文件：redis.conf
    * 语法：
    ```shell
    redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
    ```

    * 例子
    ```shell
    redis 127.0.0.1:6379> CONFIG GET  loglevel
    redis 127.0.0.1:6379> CONFIG GET *
    ```

* 修改配置：

    * 语法：
    ```shell
    redis 127.0.0.1:6379> CONFIG SET  CONFIG_SETTING_NAME NEW_CONFIG_VALUE
    ```

    * 例子
    ```
    redis 127.0.0.1:6379> CONFIG SET loglevel "notice"
    OK
    redis 127.0.0.1:6379> CONFIG GET loglevel
    ```

## Redis 数据类型(5种)

* Strings-字符串

例子：
```shell
redis 127.0.0.1:6379> SET name "tutorialspoint"
OK
redis 127.0.0.1:6379>GET name
"tutorialspoint"
```

* Hashes-字典

 例子：
```shell
redis 127.0.0.1:6379> HMSET user:1 username tutorialspoint password tutorialspoint points 200
OK
redis 127.0.0.1:6379> HGETALL user:1

1) "username"
2) "tutorialspoint"
3) "password"
4) "tutorialspoint"
5) "points"
6) "200

```

* Lists-列表

时间复杂度：O(N)

 例子：
```shell
redis 127.0.0.1:6379> lpush tutoriallist redis
(integer) 1
redis 127.0.0.1:6379> lpush tutoriallist mongodb
(integer) 2
redis 127.0.0.1:6379> lpush tutoriallist rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange tutoriallist 0 10

1) "rabitmq"
2) "mongodb"
3) "redis"

```

* Sets-集合

时间复杂度：O(1)

例子：
```shell
redis 127.0.0.1:6379> sadd tutoriallist redis
(integer) 1
redis 127.0.0.1:6379> sadd tutoriallist mongodb
(integer) 1
redis 127.0.0.1:6379> sadd tutoriallist rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd tutoriallist rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers tutoriallist

1) "rabitmq"
2) "mongodb"
3) "redis"
```
* Stored Sets-有序集合

例子：
```shell
redis 127.0.0.1:6379> zadd tutoriallist 0 redis
(integer) 1
redis 127.0.0.1:6379> zadd tutoriallist 0 mongodb
(integer) 1
redis 127.0.0.1:6379> zadd tutoriallist 0 rabitmq
(integer) 1
redis 127.0.0.1:6379> zadd tutoriallist 0 rabitmq
(integer) 0
redis 127.0.0.1:6379> ZRANGEBYSCORE tutoriallist 0 1000

1) "redis"
2) "mongodb"
3) "rabitmq"
```

## Redis 命令

* 服务器端

 ```shell
 $redis-cli -h host -p port -a password
 ```

## Redis 操作Keys命令

|S.N.|Command & Description|
|---|:---|
|1|	DEL key This command deletes the key, if exists|
|2|	DUMP key This command returns a serialized version of the value stored at the specified key.|
|3|	EXISTS key This command checks whether the key exists or not.|
|4|	EXPIRE key seconds Expires the key after the specified time|
|5|	EXPIREAT key timestamp Expires the key after the specified time. Here time is in Unix timestamp format|
|6|	PEXPIRE key milliseconds  Set the expiry of key in milliseconds|
|7|	PEXPIREAT key milliseconds-timestamp Set the expiry of key in unix timestamp specified as milliseconds|
|8|	KEYS pattern Find all keys matching the specified pattern|
|9|	MOVE key db Move a key to another database|
|10|	PERSIST key Remove the expiration from the key|
|11|	PTTL key Get the remaining time in keys expiry in milliseconds.|
|12|	TTL key Get the remaining time in keys expiry.|
|13|	RANDOMKEY Return a random key from redis|
|14|RENAME key newkey Change the key name|
|15|	RENAMENX key newkey Rename key, if new key doesn't exist|
|16|	TYPE key Return the data type of value stored in key.|


## Redis-Strings 命令列表
|S.N.|Command & Description|
|---|:---|
|1|	SET key value This command sets the value at the specified key|
|2|	GET key Get the value of a key.|
|3|	GETRANGE key start end  Get a substring of the string stored at a key|
|4|	GETSET key value Set the string value of a key and return its old value|
|5|	GETBIT key offset Returns the bit value at offset in the string value stored at key|
|6|	MGET key1 [key2..] Get the values of all the given keys|
|7|	SETBIT key offset value Sets or clears the bit at offset in the string value stored at key|
|8|	SETEX key seconds value Set the value with expiry of a key|
|9|	SETNX key value Set the value of a key, only if the key does not exist|
|10|	SETRANGE key offset value Overwrite part of a string at key starting at the specified offset|
|11|	STRLEN key Get the length of the value stored in a key|
|12|	MSET key value [key value ...] Set multiple keys to multiple values|
|13|	MSETNX key value [key value ...]  Set multiple keys to multiple values, only if none of the keys exist|
|14|	PSETEX key milliseconds value Set the value and expiration in milliseconds of a key|
|15|	INCR key Increment the integer value of a key by one|
|16|	INCRBY key increment Increment the integer value of a key by the given amount|
|17|	INCRBYFLOAT key increment Increment the float value of a key by the given amount|
|18|	DECR key Decrement the integer value of a key by one|
|19|	DECRBY key decrement Decrement the integer value of a key by the given number|
|20|	APPEND key value Append a value to a key|

## Redis-Hashes 命令列表
|S.N.|Command & Description|
|---|:---|
|1|	HDEL key field2 [field2]  Delete one or more hash fields|
|2|	HEXISTS key field  Determine whether a hash field exists or not|
|3|	HGET key field Get the value of a hash field stored at specified key|
|4|	HGETALL key Get all the fields and values stored in a hash at specified key|
|5|	HINCRBY key field increment Increment the integer value of a hash field by the given number|
|6|	HINCRBYFLOAT key field increment Increment the float value of a hash field by the given amount|
|7|	HKEYS key Get all the fields in a hash|
|8|	HLEN key Get the number of fields in a hash|
|9|	HMGET key field1 [field2] Get the values of all the given hash fields|
|10|	HMSET key field1 value1 [field2 value2 ] Set multiple hash fields to multiple values|
|11|	HSET key field value Set the string value of a hash field|
|12|	HSETNX key field value Set the value of a hash field, only if the field does not exist|
|13|	HVALS key Get all the values in a hash|
|14|	HSCAN key cursor [MATCH pattern] [COUNT count] Incrementally iterate hash fields and associated values|

## Redis-Lists 命令列表
|S.N.|Command & Description|
|---|:---|
|1|	BLPOP key1 [key2 ] timeout Remove and get the first element in a list, or block until one is available|
|2| BRPOP key1 [key2 ] timeout Remove and get the last element in a list, or block until one is available|
|3|	BRPOPLPUSH source destination timeout Pop a value from a list, push it to another list and return it; or block until one is available|
|4|	LINDEX key index Get an element from a list by its index|
|5| LINSERT key BEFORE|AFTER pivot value Insert an element before or after another element in a list|
|6| LLEN key Get the length of a list|
|7| LPOP key Remove and get the first element in a list|
|8	|LPUSH key value1 [value2] Prepend one or multiple values to a list|
|9	|LPUSHX key value Prepend a value to a list, only if the list exists|
|10|	LRANGE key start stop Get a range of elements from a list|
|11	|LREM key count value Remove elements from a list|
|12	|LSET key index value Set the value of an element in a list by its index|
|13	|LTRIM key start stop Trim a list to the specified range|
|14	|RPOP key Remove and get the last element in a list|
|15	|RPOPLPUSH source destination Remove the last element in a list, append it to another list and return it|
|16	|RPUSH key value1 [value2] Append one or multiple values to a list|
|17	|RPUSHX key value Append a value to a list, only if the list exists|


## Redis-Sets 命令列表
|S.N.|Command & Description|
|---|:---|
|1|	SADD key member1 [member2] Add one or more members to a set|
|2	|SCARD key Get the number of members in a set|
|3	|SDIFF key1 [key2] Subtract multiple sets|
|4	|SDIFFSTORE destination key1 [key2] Subtract multiple sets and store the resulting set in a key|
|5	|SINTER key1 [key2] Intersect multiple sets|
|6	|SINTERSTORE destination key1 [key2] Intersect multiple sets and store the resulting set in a key|
|7	|SISMEMBER key member Determine if a given value is a member of a set|
|8	|SMEMBERS key Get all the members in a set|
|9	|SMOVE source destination member Move a member from one set to another|
|10	|SPOP key Remove and return a random member from a set|
|11	|SRANDMEMBER key [count] Get one or multiple random members from a set|
|12	|SREM key member1 [member2] Remove one or more members from a set|
|13	|SUNION key1 [key2] Add multiple sets|
|14	|SUNIONSTORE destination key1 [key2] Add multiple sets and store the resulting set in a key|
|15	|SSCAN key cursor [MATCH pattern] [COUNT count] Incrementally iterate Set elements|

## Redis-Sorted Sets 命令列表
|S.N.|Command & Description|
|---|:---|
|1	|ZADD key score1 member1 [score2 member2] Add one or more members to a sorted set, or update its score if it already exists|
|2	|ZCARD key Get the number of members in a sorted set|
|3	|ZCOUNT key min max Count the members in a sorted set with scores within the given values|
|4	|ZINCRBY key increment member Increment the score of a member in a sorted set|
|5	|ZINTERSTORE destination numkeys key [key ...] Intersect multiple sorted sets and store the resulting sorted set in a new key|
|6	|ZLEXCOUNT key min max Count the number of members in a sorted set between a given lexicographical range|
|7	|ZRANGE key start stop [WITHSCORES] Return a range of members in a sorted set, by index|
|8	|ZRANGEBYLEX key min max [LIMIT offset count] Return a range of members in a sorted set, by lexicographical range|
|9	|ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] Return a range of members in a sorted set, by score|
|10	|ZRANK key member Determine the index of a member in a sorted set|
|11	|ZREM key member [member ...] Remove one or more members from a sorted set|
|12	|ZREMRANGEBYLEX key min max Remove all members in a sorted set between the given lexicographical range|
|13	|ZREMRANGEBYRANK key start stop Remove all members in a sorted set within the given indexes|
|14	|ZREMRANGEBYSCORE key min max Remove all members in a sorted set within the given scores|
|15	|ZREVRANGE key start stop [WITHSCORES] Return a range of members in a sorted set, by index, with scores ordered from high to low|
|16	|ZREVRANGEBYSCORE key max min [WITHSCORES] Return a range of members in a sorted set, by score, with scores ordered from high to low|
|17	|ZREVRANK key member Determine the index of a member in a sorted set, with scores ordered from high to low|
|18	|ZSCORE key member Get the score associated with the given member in a sorted set|
|19	|ZUNIONSTORE destination numkeys key [key ...] Add multiple sorted sets and store the resulting sorted set in a new key|
|20|	ZSCAN key cursor [MATCH pattern] [COUNT count] Incrementally iterate sorted sets elements and associated scores|


## Redis-HyperLogLog-计算基数

* 例子
```
redis 127.0.0.1:6379> PFADD tutorials "redis"
1) (integer) 1
redis 127.0.0.1:6379> PFADD tutorials "mongodb"
1) (integer) 1
redis 127.0.0.1:6379> PFADD tutorials "mysql"
1) (integer) 1
redis 127.0.0.1:6379> PFCOUNT tutorials
(integer) 3
```

|S.N.|Command & Description|
|---|:---|
|1|	PFADD key element [element ...]  Adds the specified elements to the specified HyperLogLog.|
|2|	PFCOUNT key [key ...]  Return the approximated cardinality of the set(s) observed by the HyperLogLog at key(s).|
|3|	PFMERGE destkey sourcekey [sourcekey ...]  Merge N different HyperLogLogs into a single one.|



## Redis - Publish Subscribe

Redis的pub sub 实现了发送、接受message 的系统。

* 例子

```shell

redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

```shell
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
(integer) 1
redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by tutorials point"
(integer) 1
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by tutorials point"
```

|S.N.|Command & Description|
|---|:---|
|1|	PSUBSCRIBE pattern [pattern ...] Subscribe to channels matching the given patterns.|
|2|	PUBSUB subcommand [argument [argument ...]] Tells the state of pubsub system eg which clients are active on the server.|
|3|	PUBLISH channel message Post a message to a channel.|
|4|	PUNSUBSCRIBE [pattern [pattern ...]]  Stop listening for messages posted to channels matching the given patterns.|
|5|	SUBSCRIBE channel [channel ...]  Listen for messages published to the given channels.|
|6|	UNSUBSCRIBE [channel [channel ...]]  Stop listening for messages posted to the given channels.|


## Redis-Transaction 事务

执行一组命令

* 例子
```shell
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> SET tutorial redis
QUEUED
redis 127.0.0.1:6379> GET tutorial
QUEUED
redis 127.0.0.1:6379> INCR visitors
QUEUED
redis 127.0.0.1:6379> EXEC

1) OK
2) "redis"
3) (integer) 1
```

|S.N.|Command & Description|
|---|:---|
|1|	DISCARD Discard all commands issued after MULTI|
|2|	EXEC Execute all commands issued after MULTI|
|3|	MULTI Mark the start of a transaction block|
|4|	UNWATCH Forget about all watched keys|
|5|	WATCH key [key ...] Watch the given keys to determine execution of the MULTI/EXEC block|

## Redis-Scripting

* 语法

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]

```

* 例子

```
redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second

1) "key1"
2) "key2"
3) "first"
4) "second"
```

|S.N.|Command & Description|
|---|:---|
|1|	EVAL script numkeys key [key ...] arg [arg ...]  Execute a Lua script.|
|2|	EVALSHA sha1 numkeys key [key ...] arg [arg ...] Execute a Lua script.|
|3|	SCRIPT EXISTS script [script ...]  Check existence of scripts in the script cache.|
|4|	SCRIPT FLUSH Remove all the scripts from the script cache.|
|5|	SCRIPT KILL Kill the script currently in execution.|
|6|	SCRIPT LOAD script Load the specified Lua script into the script cache.|



## Redis - Connections

* 例子

```
redis 127.0.0.1:6379> AUTH "password"
OK
redis 127.0.0.1:6379> PING
PONG

```

* 链接命令
|S.N.|Command & Description|
|---|:---|
|1|	AUTH password Authenticate to the server with given password|
|2|	ECHO message Print the given string|
|3|	PING Check whether server is running or not|
|4|	QUIT Close the current connection|
|5	|SELECT index Change the selected database for the current connection|



## Redis - Server

* 例子

```
redis 127.0.0.1:6379> INFO

```

* Redis server 命令列表


|S.N.|Command & Description|
|---|:---|
|1|	BGREWRITEAOF Asynchronously rewrite the append-only file|
|2|	BGSAVE Asynchronously save the dataset to disk|
|3|	CLIENT KILL [ip:port] [ID client-id] Kill the connection of a client|
|4|	CLIENT LIST Get the list of client connections connection to the server|
|5|	CLIENT GETNAME Get the name of current connection|
|6|	CLIENT PAUSE timeout Stop processing commands from clients for specified time|
|7|	CLIENT SETNAME connection-name Set the current connection name|
|8|	CLUSTER SLOTS Get array of Cluster slot to node mappings|
|9|	COMMAND Get array of Redis command details|
|10|	COMMAND COUNT Get total number of Redis commands|
|11|	COMMAND GETKEYS Extract keys given a full Redis command|
|12|	BGSAVE Asynchronously save the dataset to disk|
|13|	COMMAND INFO command-name [command-name ...] Get array of specific Redis command details|
|14|	CONFIG GET parameter Get the value of a configuration parameter|
|15|	CONFIG REWRITE Rewrite the configuration file with the in memory configuration|
|16|	CONFIG SET parameter value Set a configuration parameter to the given value|
|17|	CONFIG RESETSTAT Reset the stats returned by INFO|
|18|	DBSIZE Return the number of keys in the selected database|
|19|	DEBUG OBJECT key Get debugging information about a key|
|20|	DEBUG SEGFAULT Make the server crash|
|21|	FLUSHALL Remove all keys from all databases|
|22|	FLUSHDB Remove all keys from the current database|
|23|	INFO [section] Get information and statistics about the server|
|24|	LASTSAVE Get the UNIX time stamp of the last successful save to disk|
|25|	MONITOR Listen for all requests received by the server in real time|
|26|	ROLE Return the role of the instance in the context of replication|
|27|	SAVE Synchronously save the dataset to disk|
|28|	SHUTDOWN [NOSAVE] [SAVE] Synchronously save the dataset to disk and then shut down the server|
|29|	SLAVEOF host port Make the server a slave of another instance, or promote it as master|
|30|	SLOWLOG subcommand [argument] Manages the Redis slow queries log|
|31|	SYNC command used for replication|
|32|	TIME Return the current server time|


## Redis - 备份

* 语法 和例子
```
127.0.0.1:6379> SAVE
OK

```

* 恢复Redis数据
* 通过CONFIG 命令获得数据的存储路径：

   ```shell
   127.0.0.1:6379> CONFIG get dir
   1) "dir"
   2) "/user/tutorialspoint/redis-2.8.13/src"
   ```



 * 恢复（BGSAVE）

  ```shell
   127.0.0.1:6379> BGSAVE
   Background saving started
  ```

## Redis - 安全

默认状态下，Redis没有密码，要在设置密码：

```shell
127.0.0.1:6379> CONFIG set requirepass "tutorialspoint"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "tutorialspoint"
127.0.0.1:6379> AUTH "tutorialspoint"
OK
127.0.0.1:6379> SET mykey "Test value"
OK
127.0.0.1:6379> GET mykey
"Test value"
```

## Redis 监控执行效率(Benchmarks)

* 语法

```shell
redis-benchmark [option] [option value]
例如：
redis-benchmark -n 100000
```

|S.N.|Option| Description|Default Value|
|---|:---|:---|:---|
|1|	-h|	Specifies server host name|	127.0.0.1|
|2|	-p|	Specifies server port|	6379|
|3|	-s|	Specifies server socket	||
|4|	-c|	Specifies number of parallel connections|	50|
|5|	-n|	Specifies total number of requests|	10000|
|6|	-d|	Specifies data size of SET/GET value in bytes	|2|
|7|	-k|	1=keep alive 0=reconnect|	1|
|8|	-r|	Use random keys for SET/GET/INCR, random values for SADD	||
|9|	-p|	Pipeline <numreq> requests|	1|
|10|	-q|	Forces Quiet to redis. Just show query/sec values|	|
|11|	--csv|	Output in CSV format|	|
|12|	-l|	Generates loop, Run the tests forever	||
|13|	-t|	Only run the comma-separated list of tests.	||
|14|	-I|	Idle mode. Just open N idle connections and wait.	||

 * 例子

```shell
redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 100000 -q
SET: 146198.83 requests per second
LPUSH: 145560.41 requests per second

```


## Redis - 客户端连接(Client Connection)

* 客户端最大连接数

```shell
config get maxclients
1) "maxclients"
2) "10000"
```

* Client 命令列表
|S.N.|Command| Description|
|---|:---|:---|
|1|	CLIENT LIST	|Returns the list of clients connected to redis server|
|2|	CLIENT SETNAME	|Assigns a name to the current connection|
|3|	CLIENT GETNAME	|Returns the name of the current connection as set by CLIENT SETNAME.|
|4|	CLIENT PAUSE	|This is a connections control command able to suspend all the Redis clients for the specified amount of time (in milliseconds).|
|5|	CLIENT KILL	|This command closes a given client connection.|


## Redis 管道(Pipelining)
请求-应答模式

## Redis - 分区(Partitioning)
将数据分布到不同的redis实例中

* 分区动机
> 更好的利用多台机器的存储资源横向扩展，毕竟单台机器的存储资源是有限的。
> 更好的利用多台计算机的处理器资源、网络带宽资源。



## Redis - Java

* 安装下载: jedis.jar
    https://github.com/xetorthio/jedis
    https://github.com/xetorthio/jedis/releases
   jedis-jedis-2.9.0.tar.gz



* 连接 Redis Server

```java
import redis.clients.jedis.Jedis;

public class RedisJava {

   public static void main(String[] args) {

      //Connecting to Redis server on localhost

      Jedis jedis = new Jedis("localhost");

      System.out.println("Connection to server sucessfully");

      //check whether server is running or not

      System.out.println("Server is running: "+jedis.ping());

 }

}



$javac RedisJava.java

$java RedisJava

Connection to server sucessfully

Server is running: PONG

```

* Redis Java String Example

```java
import redis.clients.jedis.Jedis;

public class RedisStringJava {

   public static void main(String[] args) {

      //Connecting to Redis server on localhost

      Jedis jedis = new Jedis("localhost");

      System.out.println("Connection to server sucessfully");

      //set the data in redis string

      jedis.set("tutorial-name", "Redis tutorial");

     // Get the stored data and print it

     System.out.println("Stored string in redis:: "+ jedis.get("tutorial-name"));

 }

}



$javac RedisStringJava.java

$java RedisStringJava

Connection to server sucessfully

Stored string in redis:: Redis tutorial

```

* Redis Java List Example

```java
import redis.clients.jedis.Jedis;

public class RedisListJava {

   public static void main(String[] args) {

      //Connecting to Redis server on localhost

      Jedis jedis = new Jedis("localhost");

      System.out.println("Connection to server sucessfully");

      //store data in redis list

      jedis.lpush("tutorial-list", "Redis");

      jedis.lpush("tutorial-list", "Mongodb");

      jedis.lpush("tutorial-list", "Mysql");

     // Get the stored data and print it

     List<String> list = jedis.lrange("tutorial-list", 0 ,5);

     for(int i=0; i<list.size(); i++) {

       System.out.println("Stored string in redis:: "+list.get(i));

     }

 }

}

$javac RedisListJava.java

$java RedisListJava

Connection to server sucessfully

Stored string in redis:: Redis

Stored string in redis:: Mongodb

Stored string in redis:: Mysql

```



* Redis Java Keys Example

```java

import redis.clients.jedis.Jedis;

public class RedisKeyJava {

   public static void main(String[] args) {

      //Connecting to Redis server on localhost

      Jedis jedis = new Jedis("localhost");

      System.out.println("Connection to server sucessfully");

      //store data in redis list

     // Get the stored data and print it

     List<String> list = jedis.keys("*");

     for(int i=0; i<list.size(); i++) {

       System.out.println("List of stored keys:: "+list.get(i));

     }

   }

}



$javac RedisKeyJava.java

$java RedisKeyJava

Connection to server sucessfully

List of stored keys:: tutorial-name

List of stored keys:: tutorial-list

```



## 参考资料

* Useful Links on Redis

[Redis Documentation](http://redis.io/) − Redis's official website for its latest docs and updates.

http://redis.io/

[Redis Downloads](http://redis.io/download) − Official website to download latest release of Redis.

http://redis.io/download

[Wiki page on Redis](https://en.wikipedia.org/wiki/Redis) − A short tutorial on Redis.



* PDF 

《Redis IN ACTION》

《Redis The Definitive Guide》

《Redis Cookbook》



















