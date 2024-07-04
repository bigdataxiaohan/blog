---
title: Elasticsearch简介
date: 2019-04-12 18:06:58
tags: Elasticsearch
categories: NOSQL
---

## 简介

Elasticsearch (ES)是一个基于Lucene构建的开源、分布式、[RESTful](https://baike.baidu.com/item/RESTful/4406165?fr=aladdin)接口全文搜索引擎Elasticsearch还是一个分布式文档数据库，其中每个字段均是被索引的数据且可被搜索，它能够扩展至数以百计的服务器存储以及处理PB级的数据。它可以在很短的时间内存储、搜索和分析大量的数据。它通常作为具有复杂搜索场景情况下的核心发动机。Elasticsearch就是为高可用和可扩展而生的。可以通过购置性能更强的服务器来完成，称为垂直扩展或者向上扩展（Vertical Scale/Scaling Up)，或增加更多的服务器来完成，称为水平扩展或者向外扩展（Horizontal Scale/Scaling Out)尽管ES能够利用更强劲的硬件，垂直扩展毕竟还是有它的极限。真正的可扩展性来自于水平扩展，通过向集群中添加更多的节点来分担负载，增加可靠性。在大多数数据库中，水平扩展通常都需要你对应用进行一次大的重构来利用更多的节点。而ES天生就是分布式的：它知道如何管理多个节点来完成扩展和实现高可用性。这也意味着你的应用不需要做任何的改动。

详细ElasticSearch内容可以在本站搜索阅读,如果有错,请及时提醒,避免误导他人。

## 准备

安装Elasticsearch

![AqntvF.png](https://s2.ax1x.com/2019/04/12/AqntvF.png)

注意看dockerhub中是否存在latest标签的镜像，因为没有这个镜像耽误了一些时间。

![AquFr4.png](https://s2.ax1x.com/2019/04/12/AquFr4.png)

因为elasticsearch在运行中会默认占用2个G，我们本机的资源够用，这里我们可以通过docker限制docker的资源。

![AquRzT.png](https://s2.ax1x.com/2019/04/12/AquRzT.png)

这里我们 制定了初始化内存和最大堆内存为512MB。暴露外部通信端口9200集群节点通信端口9300。

![AquqW6.png](https://s2.ax1x.com/2019/04/12/AquqW6.png)

启动成功。

![AqKc0H.png](https://s2.ax1x.com/2019/04/12/AqKc0H.png)

为什么Json编程这个样子了，因为我安装了JSON VIEW

Elasticsearch 是 *面向文档* 的，意味着它存储整个对象或 *文档_。Elasticsearch 不仅存储文档，而且 _索引*每个文档的内容使之可以被检索。在 Elasticsearch 中，你 对文档进行索引、检索、排序和过滤--而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

Elasticsearch 使用 JavaScript Object Notation 或者 [*JSON*](http://en.wikipedia.org/wiki/Json) 作为文档的序列化格式。JSON 序列化被大多数编程语言所支持，并且已经成为 NoSQL 领域的标准格式。 它简单、简洁、易于阅读。

考虑一下这个 JSON 文档，它代表了一个 user 对象：

```json
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
```

我们受雇于 *Megacorp* 公司，作为 HR 部门新的 *“热爱无人机”* （_"We love our drones!"_）激励项目的一部分，我们的任务是为此创建一个雇员目录。该目录应当能培养雇员认同感及支持实时、高效、动态协作，因此有一些业务需求：

- 支持包含多值标签、数值、以及全文本的数据
- 检索任一雇员的完整信息
- 允许结构化搜索，比如查询 30 岁以上的员工
- 允许简单的全文搜索以及较复杂的短语搜索
- 支持在匹配文档内容中高亮显示搜索片段
- 支持基于数据创建和管理分析仪表盘

![Aqlg74.png](https://s2.ax1x.com/2019/04/12/Aqlg74.png)

## 数据准备

  我们可以把索引类比为MYSQL中的数据库而类型类比成为数据库中的表，文档则时数据库中的记录。

```json
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

![Aq1MbF.png](https://s2.ax1x.com/2019/04/12/Aq1MbF.png)

这里使用的软件为Postman

保存2号员工

```json
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```

![Aq1Ygx.png](https://s2.ax1x.com/2019/04/12/Aq1Ygx.png)

```json
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

![Aq172q.png](https://s2.ax1x.com/2019/04/12/Aq172q.png)

## 检索数据

![Aq1xIJ.png](https://s2.ax1x.com/2019/04/12/Aq1xIJ.png)

如果将GET换成DELETE则是删除。

![Aq3qfA.png](https://s2.ax1x.com/2019/04/12/Aq3qfA.png)

删除后GET的效果

将 HTTP 命令由 PUT 改为 GET 可以用来检索文档，同样的，可以使用 DELETE 命令来删除文档，以及使用 HEAD 指令来检查文档是否存在。如果想更新已存在的文档，只需再次 PUT 。

![Aq8emT.png](https://s2.ax1x.com/2019/04/12/Aq8emT.png)

![AqGSjx.png](https://s2.ax1x.com/2019/04/12/AqGSjx.png)

![AqGMb8.png](https://s2.ax1x.com/2019/04/12/AqGMb8.png)

### 查询规则

```json
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

![AqGUK0.png](https://s2.ax1x.com/2019/04/12/AqGUK0.png)

和_search?q=last_name:Smith效果相同

```json
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 33 } 
                }
            }
        }
    }
```

![AqJsw8.png](https://s2.ax1x.com/2019/04/12/AqJsw8.png)

### 全文检索

```json
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

![AqJzm6.png](https://s2.ax1x.com/2019/04/12/AqJzm6.png)

### 短语搜索

短语搜索和全文搜索的区别在于短语搜索会完全匹配

```json
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

![AqtZ8J.png](https://s2.ax1x.com/2019/04/12/AqtZ8J.png)

### 高亮搜索

```json
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

![Aqt35D.png](https://s2.ax1x.com/2019/04/12/Aqt35D.png)

![Aqt62j.png](https://s2.ax1x.com/2019/04/12/Aqt62j.png)

如何理解高亮搜扫，类似于百度百科搜索的关键词会又`<em>`标签。



