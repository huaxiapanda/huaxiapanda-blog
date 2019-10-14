---
title: elasticsearch入门
permalink: elasticsearch入门
date: 2019-10-14 17:48:15
categories: 大数据
tags: elasticsearch
---
@[toc]
# 1. elasticsearch核心概念
[elk下载地址](https://www.elastic.co/cn/downloads/past-releases)
## 1.1 Near Realtime(NRT)
&emsp;&emsp;近实时，两个意思，从写入数据到数据可以被搜索到有一个小延迟（大概1秒）；基于es执行搜索和分析可以达到秒级.
## 1.2 Cluster
&emsp;&emsp;集群，包含多个节点，每个节点属于哪个集群是通过一个配置（集群名称，默认是elasticsearch）来决定的，对于中小型应用来说，刚开始一个集群就一个节点很正常.**可以在elasticsearch.yml中修改这个名称**
## 1.3 Node
&emsp;&emsp;节点，集群中的一个节点，节点也有一个名称（默认是随机分配的），节点名称很重要（在执行运维管理操作的时候），默认节点会去加入一个名称为"elasticsearch"的集群，如果直接启动一堆节点，那么它们会自动组成一个elasticsearch集群，当然一个节点也可以组成一个elasticsearch集群.
## 1.4 Document&field
&emsp;&emsp;文档，es中的最小数据单元，一个document可以是一条客户数据，一条商品分类数据，一条订单数据，通常用JSON数据结构表示，每个index下的type中，都可以去存储多个document。一个document里面有多个field，每个field就是一个数据字段。<br />
例如一个product document
```javascript
// product document
{
    "product_id":"3",
    "product_name":"华为荣耀P30 Pro",
    "product_desc":"50倍放大，清晰",
    "category_id":"2",
    "category_name":"手机数码"
}
```
**注：index其实就是一个数据库，可以存储很多type，type其实就是一张表,可以存储很多Document，Document其实就是一行数据,一行数据有很多Filed，Filed其实就是一行中的一个字段。**
## 1.5 Index
&emsp;&emsp;索引，包含一堆有相似结构的文档数据，比如可以有一个客户索引，商品分类索引，订单索引，索引有一个名称。一个index包含很多document，一个index就代表了一类类似的或者相同的document。比如说建立一个product index，商品索引，里面可能就存放了所有的商品数据，所有的商品document。
## 1.6 Type
&emsp;&emsp; 类型，每个索引里都可以有一个或多个type，type是index中的一个逻辑数据分类，一个type下的document，都有相同的field，比如博客系统，有一个索引，可以定义用户数据type，博客数据type，评论数据type。<br />
&emsp;&emsp;商品index，里面存放了所有的商品数据，商品document,但是商品分很多种类，每个种类的document的field可能不太一样，比如说电器商品，可能还包含一些诸如售后时间范围这样的特殊field；生鲜商品，还包含一些诸如生鲜保质期之类的特殊field <br>

example:<br />
&emsp;&emsp;type,日化商品type，电器商品type，生鲜商品type<br/>
&emsp;&emsp;日化商品type：product_id，product_name，product_desc，category_id，category_name<br/>
&emsp;&emsp;电器商品type：product_id，product_name，product_desc，category_id，category_name，service_period<br/>
&emsp;&emsp;生鲜商品type：product_id，product_name，product_desc，category_id，category_name，eat_period<br/>

&emsp;&emsp;每一个type里面，都会包含一堆document
```javascript
{
  "product_id": "2",
  "product_name": "长虹电视机",
  "product_desc": "4k高清",
  "category_id": "3",
  "category_name": "电器",
  "service_period": "1年"
}
{
  "product_id": "3",
  "product_name": "基围虾",
  "product_desc": "纯天然，冰岛产",
  "category_id": "4",
  "category_name": "生鲜",
  "eat_period": "7天"
}
```
## 1.7 shard
&emsp;&emsp;单台机器无法存储大量数据，es可以将一个索引中的数据切分为多个shard，分布在多台服务器上存储。有了shard就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。每个shard都是一个lucene index。
## 1.8 replica
&emsp;&emsp;任何一个服务器随时可能故障或宕机，此时shard可能就会丢失，因此可以为每个shard创建多个replica副本。replica可以在shard故障时提供备用服务，保证数据不丢失，多个replica还可以提升搜索操作的吞吐量和性能。primary shard（建立索引时一次设置，不能修改，默认5个），replica shard（随时修改数量，默认1个），默认每个索引10个shard，5个primary shard，5个replica shard，最小的高可用配置，是2台服务器。
## 1.9 share和replica图解
![shared和replica图解](elasticsearch入门/1.png)
## 1.10 elasticsearch核心概念 vs 数据库核心概念
| Elasticsearch    | 数据库     |
| :-------------: | :-------------: |
| Document       | 行/row       |
| Type     | 表/table       |
| Index       | 库/database       |

# 2 基础API
## 2.1 Elasticsearch之document数据格式
&emsp;&emsp;elasticsearch是面向文档的搜索分析引擎<br/>
+ 应用系统的数据结构都是面向对象的，复杂的
+ 对象数据存储到数据库中，只能拆解开来，变为扁平的多张表，每次查询的时候还得还原回对象格式，相当麻烦
+ ES是面向文档的，文档中存储的数据结构，与面向对象的数据结构是一样的，基于这种文档数据结构，es可以提供复杂的索引，全文检索，分析聚合等功能.
+ es的document用json数据格式来表达<br/>

**传统的关系型数据库的数据格式:**
```Java
public class Employee {
  private String email;
  private String firstName;
  private String lastName;
  private EmployeeInfo info;
  private Date joinDate;
}

private class EmployeeInfo {
  private String bio; // 性格
  private Integer age;
  private String[] interests; // 兴趣爱好
}

EmployeeInfo info = new EmployeeInfo();
info.setBio("curious and modest");
info.setAge(30);
info.setInterests(new String[]{"bike", "climb"});

Employee employee = new Employee();
employee.setEmail("zhangsan@sina.com");
employee.setFirstName("san");
employee.setLastName("zhang");
employee.setInfo(info);
employee.setJoinDate(new Date());
// employee对象：里面包含了Employee类自己的属性，还有一个EmployeeInfo对象
// 两张表：employee表，employee_info表，将employee对象的数据重新拆开来，变成Employee数据和EmployeeInfo数据
// employee表：email，first_name，last_name，join_date，4个字段
// employee_info表：bio，age，interests，3个字段；此外还有一个外键字段，比如employee_id，关联着employee表
```
**elasticsearch数据格式为:**
```javascript
{
    "email":      "zhangsan@sina.com",
    "first_name": "san",
    "last_name": "zhang",
    "info": {
        "bio":         "curious and modest",
        "age":         30,
        "interests": [ "bike", "climb" ]
    },
    "join_date": "2017/01/01"
}
```
## 2.2 基础API
### 2.2.1 快速查看集群的健康状态
es提供了一套api,叫做cat api,可以查看es中各种各样的数据:
```
GET /_cat/health?v
```
![健康状态结果](elasticsearch入门/2.png)
+ 如何快速了解集群的健康状态?green、yellow、red?
  + green:每个索引的primary shard和replica shard都是active状态的。
  + yellow:每个索引的primary shard都是active状态的，但是部分replica shard不是active状态，处于不可用的状态。
  + red:不是所有索引的primary shard都是active状态的，部分索引有数据丢失了。
+ 为什么现在会处于一个yellow状态？<br/>
&emsp;&emsp;我们现在就一个笔记本电脑，就启动了一个es进程，相当于就只有一个node。现在es中有一个index，就是kibana自己内置建立的index。由于默认的配置是给每个index分配5个primary shard和5个replica shard，而且primary shard和replica shard不能在同一台机器上（为了容错）。现在kibana自己建立的index是1个primary shard和1个replica shard。当前就一个node，所以只有1个primary shard被分配了和启动了，但是一个replica shard没有第二台机器去启动。
