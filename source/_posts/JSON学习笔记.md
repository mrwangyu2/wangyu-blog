---
title: JSON学习笔记
date: 2016-12-02 09:11:24
tags:
- 大数据
---

![](/assets/blog-image/json-t.jpg)
由 王宇 原创并发布 ：

## JSON 是如何存储信息

```json
{
   "book": [
      {
         "id":"01",
         "language": "Java",
         "edition": "third",
         "author": "Herbert Schildt"
      },
      {
         "id":"07",
         "language": "C++",
         "edition": "second",
         "author": "E.Balagurusamy"
      }
   ]
}

```

## 基本语法
<!--more-->
*  语法
 >* Data is represented in name/value pairs. (key value模式)
 >* Curly braces hold objects and each name is followed by ':'(colon), the name/value pairs are separated by , (comma).（key value 分号分开）
 >* Square brackets hold arrays and values are separated by ,(comma). （逗号分开每个Key Value）
* 数据结构
 > * {} 集合(Collection) or 对象(Object)
 >*  [] 数组 (Array)

## JSON 数据类型
|Type|Description|
|---|:---|
|Number|	double- precision floating-point format in JavaScript|
|String|	double-quoted Unicode with backslash escaping|
|Boolean|	true or false|
|Array|	an ordered sequence of values|
|Value|	it can be a string, a number, true or false, null etc|
|Object|	an unordered collection of key:value pairs|
|Whitespace|	can be used between any pair of tokens|
|null|	empty|

## JSON Schema
* Schema
当我们在描述 文字链接 的时候，需要约定数据的组织方式，比如，需要知道有哪些字段，这些字段的取值如何表示等，这就是 JSON Schema 的来源。
* 用途
 1. 用于描述数据结构
 2. 用于构建人机可读的文档
 3. 用于生成模拟数据
 4. 用于校验数据，实现自动化测试
* 关键字
|Keywords|Description|
|---|:---|
|$schema|	The $schema keyword states that this schema is written according to the draft v4 specification.|
|title|	You will use this to give a title to your schema.|
|description|	A little description of the schema.|
|type|	The type keyword defines the first constraint on our JSON data: it has to be a JSON Object.|
|properties|	Defines various keys and their value types, minimum and maximum values to be used in JSON file.|
|required|	This keeps a list of required properties.|
|minimum|	This is the constraint to be put on the value and represents minimum acceptable value.|
|exclusiveMinimum|	If "exclusiveMinimum" is present and has boolean value true, the instance is valid if it is strictly greater than the value of "minimum".|
|maximum|	This is the constraint to be put on the value and represents maximum acceptable value.|
|exclusiveMaximum|	If "exclusiveMaximum" is present and has boolean value true, the instance is valid if it is strictly lower than the value of "maximum".|
|multipleOf|	A numeric instance is valid against "multipleOf" if the result of the division of the instance by this keyword's value is an integer.|
|maxLength|	The length of a string instance is defined as the maximum number of its characters.|
|minLength|	The length of a string instance is defined as the minimum number of its characters.|
|pattern|	A string instance is considered valid if the regular expression matches the instance successfully.|

## JSON 同 XML 比较
* 冗长(Verbose)
XML 比JSON 有过多的冗长因素，JSON对于程序员更容易编写
* 数组的使用
XML 结构化数据，没有JSON 的数组
* Parsing
JavaScript's eval method parses JSON. When applied to JSON, eval returns the described object.

## 环境
下载配置 json-simple.jar , 在CLASSPATH中指向此包

## JSON 同 Java 实体的对比
|JSON|JAVA|
|---|:---|
|string|	java.lang.String|
|number|	java.lang.Number|
|true,false|	java.lang.Boolean|
|null|	null|
|array|	java.util.List|
|object|	java.util.Map|

## Java 中形成JSON
* 例子
```java
import org.json.simple.JSONObject;

class JsonEncodeDemo {
   public static void main(String[] args){
      JSONObject obj = new JSONObject();
      obj.put("name", "foo");
      obj.put("num", new Integer(100));
      obj.put("balance", new Double(1000.21));
      obj.put("is_vip", new Boolean(true));
      System.out.print(obj);
   }
}

```
* 输出结果
```json
{"balance": 1000.21, "num":100, "is_vip":true, "name":"foo"}

```

## Java 中解析JSON
* 例子
```java
import org.json.simple.JSONObject;
import org.json.simple.JSONArray;
import org.json.simple.parser.ParseException;
import org.json.simple.parser.JSONParser;

class JsonDecodeDemo {
   public static void main(String[] args){
      JSONParser parser = new JSONParser();
      String s = "[0,{\"1\":{\"2\":{\"3\":{\"4\":[5,{\"6\":7}]}}}}]";
      try{
         Object obj = parser.parse(s);
         JSONArray array = (JSONArray)obj;
         System.out.println("The 2nd element of array");
         System.out.println(array.get(1));
         System.out.println();

         JSONObject obj2 = (JSONObject)array.get(1);
         System.out.println("Field \"1\"");
         System.out.println(obj2.get("1"));    

         s = "{}";
         obj = parser.parse(s);
         System.out.println(obj);

         s = "[5,]";
         obj = parser.parse(s);
         System.out.println(obj);

         s = "[5,,2]";
         obj = parser.parse(s);
         System.out.println(obj);
      }catch(ParseException pe){
         System.out.println("position: " + pe.getPosition());
         System.out.println(pe);
      }
   }
}

```

* 输出结果
```json
The 2nd element of array
{"1":{"2":{"3":{"4":[5,{"6":7}]}}}}

Field "1"
{"2":{"3":{"4":[5,{"6":7}]}}}
{}
[5]
[5,2]
```

## 参考

Useful Links on JSON
[Site on JSON](http://www.json.org/) - JSON's Official site giving link on JSON specification, news, update etc.
http://www.json.org/
[The JavaTM Tutorials](http://docs.oracle.com/javase/tutorial/index.html) -The Java Tutorials are practical guides for programmers who want to use the Java programming language to create applications.
http://docs.oracle.com/javase/tutorial/index.html
[JSON - JSON at Wikipedia](https://en.wikipedia.org/wiki/JSON), the free encyclopedia
https://en.wikipedia.org/wiki/JSON
[Free Java Download](https://www.java.com/en/download/) - Download Java for your desktop computer now!
https://www.java.com/en/download/
