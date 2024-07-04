---
title: Elasticsearch增删改查
date: 2018-12-16 10:27:25
tags: Elasticsearch
categories: 数据库
---

document数据格式 

1. 应用系统的数据结构都是面向对象的，复杂的
2. 对象数据存储到数据库中，只能拆解开来，变为扁平的多张表，每次查询的时候还得还原回对象格式，相当麻烦
3. ES是面向文档的，文档中存储的数据结构，与面向对象的数据结构是一样的，基于这种文档数据结构，es可以提供复杂的索引，全文检索，分析聚合等功能
4. es的document用json数据格式来表达

### Java数据

```java
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
```

### 数据库数据

#### employee

| id   | email            | first_name | last_name | join_date  |
| ---- | ---------------- | ---------- | --------- | ---------- |
| 001  | hangsan@sina.com | san        | zhang     | 2017/01/01 |

#### employee_info

| employee_id | bio                | age  | interests   |
| :---------- | ------------------ | ---- | ----------- |
| 001         | curious and modest | 30   | bike, climb |

### Json数据

```json
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

### 集群管理

```json
GET /_cat/health?v 
```

green：每个索引的primary shard和replica shard都是active状态的
yellow：每个索引的primary shard都是active状态的，但是部分replica shard不是active状态，处于不可用的状态
red：不是所有索引的primary shard都是active状态的，部分索引有数据丢失了

现在只启动动了一个es进程，相当于就只有一个node。现在es中有一个index，就是kibana自己内置建立的index。由于默认的配置是给每个index分配5个primary shard和5个replica shard，而且primary shard和replica shard不能在同一台机器上（为了容错）。现在kibana自己建立的index是1个primary shard和1个replica shard。当前就一个node，所以只有1个primary shard被分配了和启动了，但是一个replica shard没有第二台机器去启动。只要启动第二个es进程，就会在es集群中有2个node，然后那1个replica shard就会自动分配过去，然后cluster status就会变成green状态。

### 新增

```json
#语法
PUT /index/type/id
{
  "json数据"
}
# 添加商品1
PUT /ecommerce/product/1
{
    "name" : "gaolujie yagao",                        #商品名称
    "desc" :  "gaoxiao meibai",                       #商品描述
    "price" :  30,								   #商品价格
    "producer" :      "gaolujie producer",            #生厂厂家
    "tags": [ "meibai", "fangzhu" ]                   #产品标签
}
#添加商品2
PUT /ecommerce/product/2
{
    "name" : "jiajieshi yagao",
    "desc" :  "youxiao fangzhu",
    "price" :  25,
    "producer" :      "jiajieshi producer",
    "tags": [ "fangzhu" ]
}
#添加商品3
PUT /ecommerce/product/3
{
    "name" : "zhonghua yagao",
    "desc" :  "caoben zhiwu",
    "price" :  40,
    "producer" :      "zhonghua producer",
    "tags": [ "qingxin" ]
}

```

es会自动建立index和type，不需要提前创建，而且es默认会对document每个field都建立倒排索引，让其可以被搜索

### 查询

```
#语法
GET /index/type/id
GET /ecommerce/product/1
```

```json
{
  "_index": "ecommerce",
  "_type": "product",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "name": "gaolujie yagao",
    "desc": "gaoxiao meibai",
    "price": 30,
    "producer": "gaolujie producer",
    "tags": [
      "meibai",
      "fangzhu"
    ]
  }
}
```

### 修改

```json
PUT /ecommerce/product/1
{
    "name" : "jiaqiangban gaolujie yagao",
    "desc" :  "gaoxiao meibai",
    "price" :  30,
    "producer" :      "gaolujie producer",
    "tags": [ "meibai", "fangzhu" ]
}
```

### 删除

```
DELETE /ecommerce/product/1
```



## 查询

### query string search

query string search的由来:因为search参数都是以http请求的query string来附带的

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
    "hits": {
    "total": 3,
    "max_score": 1,
    "hits": 
  ......
  {
        "_index": "ecommerce",
        "_type": "product",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "zhonghua yagao",
          "desc": "caoben zhiwu",
          "price": 40,
          "producer": "zhonghua producer",
          "tags": [
            "qingxin"
          ]
       ......
}
```

took：耗费了几毫秒
timed_out：是否超时，这里是没有
_shards：数据拆成了5个分片，所以对于搜索请求，会打到所有的primary shard（或者是它的某个replica shard）
hits.total：查询结果的数量，3个document
hits.max_score：score的含义，就是document对于一个search的相关度的匹配分数，越相关，就越匹配，分数也高
hits.hits：包含了匹配搜索的document的详细数据

按售价降序排列

````json
GET /ecommerce/product/_search?q=name:yagao&sort=price:desc
````

#### 适用场景

适用于临时的在命令行使用一些工具，比如curl，快速的发出请求，来检索想要的信息；如果查询请求很复杂，是很难去构建的在生产环境中，几乎很少使用query string search

### query DSL

DSL：Domain Specified Language，特定领域的语言
http request body：请求体，可以用json的格式来构建查询语法，比较方便，可以构建各种复杂的语法，比query string search肯定强大多了

#### 查询所有

```json
GET /ecommerce/product/_search
{
  "query": { "match_all": {} }
}
```

#### 条件查询

查询名称包含yagao的商品，同时按照价格降序排序

````json
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "name" : "yagao"
        }
    },
    "sort": [
        { "price": "desc" }
    ]
}
````

#### 分页查询

```json
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}
```

#### 指定查询

更加适合生产环境的使用，可以构建复杂的查询

```json
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "price"]
  }
```

###  query filter

#### 过滤查询

搜索商品名称包含yagao，而且售价大于25元的商品

```json
GET /ecommerce/product/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "name" : "yagao" 
                }
            },
            "filter" : {
                "range" : {
                    "price" : { "gt" : 25 } 
                }
            }
        }
    }
}
```

### full-text search（全文检索）

```
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "yagao producer"
        }
    }
}
```

producer这个字段，会先被拆解，建立倒排索引

| special   |         | 4    |
| --------- | ------- | ---- |
| yagao     |         | 4    |
| producer  | 1,2,3,4 |      |
| gaolujie  | 1       |      |
| zhognhua  | 3       |      |
| jiajieshi | 2       |      |

yagao producer 会被拆解为 yagao和producer

### phrase search（短语搜索）

跟全文检索相对应，相反，全文检索会将输入的搜索串拆解开来，去倒排索引里面去一一匹配，只要能匹配上任意一个拆解后的单词，就可以作为结果返回
phrase search，要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，才可以算匹配，才能作为结果返回

```
GET /ecommerce/product/_search
{
    "query" : {
        "match_phrase" : {
            "producer" : "yagao producer"
        }
    }
}
```

### highlight search（高亮搜索结果）

```
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "producer"
        }
    },
    "highlight": {
        "fields" : {
            "producer" : {}
        }
    }
}
```

搜索结果会有一个<em> 标签 类似于这种效果

![Fd9gg0.md.png](https://s1.ax1x.com/2018/12/16/Fd9gg0.md.png)
