---
title: Storm JDBC整合学习笔记
date: 2016-11-24 09:10:13
tags:
- 大数据
---

![](/assets/blog-image/storm-jdbc-t.jpg)
由 王宇 原创并发布 ：

## 插入数据库(Inserting into a database)
### ConnectionProvider
统一接口
org.apache.storm.jdbc.common.ConnectionProvider

```java
public interface ConnectionProvider extends Serializable {
    /**

     * method must be idempotent.

     */
    void prepare();


    /**

     *

     * @return a DB connection over which the queries can be executed.

     */
    Connection getConnection();


    /**

     * called once when the system is shutting down, should be idempotent.

     */
    void cleanup();
}

```
支持： HikariCP 连接池
org.apache.storm.jdbc.common.HikariCPConnectionProvider

### JdbcMapper
org.apache.storm.jdbc.mapper.JdbcMapper

<!--more-->
```java
public interface JdbcMapper  extends Serializable {

    List<Column> getColumns(ITuple tuple);

}
```
getColume方法，定义了一个storm tuple 如何映射一个数据库表的列。
<font color=red>
The order of the returned list is important. The place holders in the supplied queries are resolved in the same order as returned list.
</font>
返回的列表的顺序性是很重要的。总之就是自己在插入数据的时候，各个字段的顺序要对应一致。
例如：我们提交了一个插入查询语句insert into user(user_id, user_name, create_date) values (?,?, now())那么values（）中第一个？就对应了user_id,第二个？对应了usr_name以此类推。getColumns的返回列表也是这样。咱们的jdbc不提供任何不标准的查询语法。

### JdbcInsertBolt
* 要使用JdbcInsertBolt，我们需要用一个ConnectionProvider的实现以及JdbcMapper的实现（该实现将tuple转换成DB的行）来构建一个JdbcInsertBolt实例。
* 用withTableName方法提供表名
* withInsertQuery方法提供一个插入查询
* 设置一个查询超时时间来规定一个查询最多能花多少时间。默认与topology.message.timeout.secs一样大，如果该值为-1那就意味着不设置查询超时。我们可以设置查询超时时间<=topology.message.timeout.secs。
```java
Map hikariConfigMap = Maps.newHashMap();
hikariConfigMap.put("dataSourceClassName","com.mysql.jdbc.jdbc2.optional.MysqlDataSource");
hikariConfigMap.put("dataSource.url", "jdbc:mysql://localhost/test");
hikariConfigMap.put("dataSource.user","root");
hikariConfigMap.put("dataSource.password","password");
ConnectionProvider connectionProvider = new HikariCPConnectionProvider(hikariConfigMap);
String tableName = "user_details";
JdbcMapper simpleJdbcMapper = new SimpleJdbcMapper(tableName, connectionProvider);
JdbcInsertBolt userPersistanceBolt = new JdbcInsertBolt(connectionProvider, simpleJdbcMapper)
                                    .withTableName("user")
                                    .withQueryTimeoutSecs(30);
                                    Or
JdbcInsertBolt userPersistanceBolt = new JdbcInsertBolt(connectionProvider, simpleJdbcMapper)
                                    .withInsertQuery("insert into user values (?,?)")
                                    .withQueryTimeoutSecs(30);                   
```

### SimpleJdbcMapper
更一般化的JdbcMapper，它可以将tuple与数据库的行进行映射。SimpleJdbcMapper假设tuple字段名与你将要写入的数据库表中的列名是一样的。
### JdbcTridentState
We also support a trident persistent state that can be used with trident topologies.
```java
JdbcState.Options options = new JdbcState.Options()
        .withConnectionProvider(connectionProvider)
        .withMapper(jdbcMapper)
        .withTableName("user_details")
        .withQueryTimeoutSecs(30);
JdbcStateFactory jdbcStateFactory = new JdbcStateFactory(options);
```
可以使用 withInsertQuery 设置条件
## 查询数据库(Lookup from Database)
org.apache.storm.jdbc.mapper.JdbcLookupMapper
```java
void declareOutputFields(OutputFieldsDeclarer declarer);  
List<Column> getColumns(ITuple tuple);  
List<Values> toTuple(ITuple input, List<Column> columns);
```
* declareOutputFields
  指定输出的tuple中的字段
* getColumns
  确定查询中的占位符(?)以及它们的SQL类型和值。
* toTuple
  接收一个输入tuple并且表示数据库一行的列字段值列表作为select搜索的结果。
### SimpleJdbcLookupMapper
针对单表简单查询
* SimpleJdbcMapper认为tuple中的字段与你作为占位符的字段名是一致的
```java
Fields outputFields = new Fields("user_id", "user_name", "create_date");  
List<Column> queryParamColumns = Lists.newArrayList(new Column("user_id", Types.INTEGER));  
this.jdbcLookupMapper = new SimpleJdbcLookupMapper(outputFields, queryParamColumns);
```
### JdbcLookupBolt
注意超时设置要<=topology.message.timeout.secs
```java
String selectSql = "select user_name from user_details where user_id = ?";  
SimpleJdbcLookupMapper lookupMapper = new SimpleJdbcLookupMapper(outputFields, queryParamColumns)  
JdbcLookupBolt userNameLookupBolt = new JdbcLookupBolt(connectionProvider, selectSql, lookupMapper)  
        .withQueryTimeoutSecs(30);  
```
### JdbcTridentState for Lookup
```java
JdbcState.Options options = new JdbcState.Options()
        .withConnectionProvider(connectionProvider)
        .withJdbcLookupMapper(new SimpleJdbcLookupMapper(new Fields("user_name"), Lists.newArrayList(new Column("user_id", Types.INTEGER))))
        .withSelectQuery("select user_name from user_details where user_id = ?");
        .withQueryTimeoutSecs(30);
```
## 例子
### maven中加入storm-jdbc和mysql的connector
```xml
<dependency>
     <groupId>org.apache.storm</groupId>
     <artifactId>storm-jdbc</artifactId>
     <version>0.10.0</version>
     <scope>provided</scope>
    </dependency>
    <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-Java</artifactId>
     <version>5.1.31</version>
 </dependency>
```
### 表结构
```sql
CREATE TABLE `userinfo` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(50) DEFAULT NULL,
  `resource_id` varchar(10) DEFAULT NULL,
  `create_date` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `count` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### Github中的例子
storm/external/storm-jdbc/src/test/java/org/apache/storm/jdbc/bolt/
storm/examples/storm-jdbc-examples/src/main/java/org/apache/storm/jdbc/

### 实例代码
```java
import java.sql.Types;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
import org.apache.storm.guava.collect.Lists;  
import org.apache.storm.jdbc.bolt.JdbcInsertBolt;  
import org.apache.storm.jdbc.bolt.JdbcLookupBolt;  
import org.apache.storm.jdbc.common.Column;  
import org.apache.storm.jdbc.common.ConnectionProvider;  
import org.apache.storm.jdbc.common.HikariCPConnectionProvider;  
import org.apache.storm.jdbc.mapper.JdbcMapper;  
import org.apache.storm.jdbc.mapper.SimpleJdbcLookupMapper;  
import org.apache.storm.jdbc.mapper.SimpleJdbcMapper;  
import backtype.storm.tuple.Fields;  

public class PersistentBolt {  
    private static Map<String,Object> hikariConfigMap = new HashMap<String, Object>(){{  
        put("dataSourceClassName", "com.mysql.jdbc.jdbc2.optional.MysqlDataSource");  
        put("dataSource.url", "jdbc:mysql://localhost/storm");  
        put("dataSource.user","user");  
        put("dataSource.password", "password");  
    }};  

    public static ConnectionProvider connectionProvider = new HikariCPConnectionProvider(hikariConfigMap);  

    public static JdbcInsertBolt getJdbcInsertBolt(){  
        //使用tablename进行插入数据，需要指定表中的所有字段  
        /*String tableName="userinfo";
        JdbcMapper simpleJdbcMapper = new SimpleJdbcMapper(tableName, connectionProvider);
        JdbcInsertBolt jdbcInsertBolt = new JdbcInsertBolt(connectionProvider, simpleJdbcMapper)
                                        .withTableName("userinfo")
                                        .withQueryTimeoutSecs(50);*/  

        //使用schemaColumns，可以指定字段要插入的字段  
        List<Column> schemaColumns = Lists.newArrayList(new Column("user_id", Types.VARCHAR),  
                new Column("resource_id", Types.VARCHAR), new Column("count", Types.INTEGER));  
        JdbcMapper simpleJdbcMapper = new SimpleJdbcMapper(schemaColumns);  
        JdbcInsertBolt jdbcInsertBolt = new JdbcInsertBolt(connectionProvider,simpleJdbcMapper)  
                                        .withInsertQuery("insert into userinfo(id,user_id,resource_id,count) values(?,?,?)")  
                                        .withQueryTimeoutSecs(50);  
        return jdbcInsertBolt;  
    }  

    public static JdbcLookupBolt getJdbcLookupBlot(){  
        //查询  
        //指定bolt的输出字段  
        Fields outputFields = new Fields("user_id","resource_id","count");  

        //指定查询条件字段  
        List<Column> queryColumns = Lists.newArrayList(new Column("user_id", Types.VARCHAR),new Column("resource_id",Types.VARCHAR));  
        String selectSql = "select count from userinfo where user_id=? and resource_id=?";  
        SimpleJdbcLookupMapper lookupMapper = new SimpleJdbcLookupMapper(outputFields, queryColumns);  
        JdbcLookupBolt jdbcLookupBolt  = new JdbcLookupBolt(connectionProvider, selectSql, lookupMapper);  
        return jdbcLookupBolt;  
    }  
}  

```






