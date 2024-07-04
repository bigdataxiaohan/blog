---
title: ElasticSearch索引
date: 2018-12-15 22:31:53
tags: Elasticsearch
categories: 数据库
---

## 简介

索引是具有相同结构的文档集合。在Elasticsearch中索引是个非常重要的内容，对Elasticsearch的大部分操作都是基于索引来完成的。同时索引可以类比关系型数据库Mysql中的数据库database

### 创建索引

创建索引的时候可以通过修改number of shards和 number of replicas参数的数量来修改分片和副本的数量。在默认的情况下分片的数量是5个，副本的数量是1个

例如创建三个主分片两个副本分片的索引

```json
PUT /secisland
{
  "settings": {
    "index":{"number_of_shards":3,"number_of_replicas":2}
  }
}
#参数可以简写为
{"settings": {"number_of_shards":3,"number_of_replicas":2}}
```

![FacGMF.md.png](https://s1.ax1x.com/2018/12/15/FacGMF.md.png)

### 修改索引

```json
PUT /test_index/_settings
{
  "number_of_replicas":1
}
```

![Fac0G6.md.png](https://s1.ax1x.com/2018/12/15/Fac0G6.md.png)

对于任何Elasticsearch文档而言，一个文档会包括一个或者多个字段，任何字段都要有自己的数据类型，例如string、integer、date等。Elasticsearch中是通过映射来进行字段和数据类型对应的。在默认的情况下Elasticsearch会自动识别字段的数据类型。同时Elasticsearch提供了 mappings参数可以显式地进行映射。

```json
PUT /test_index1
{
"settings": {"number_of_shards":3,"number_of_replicas":2},
"mappings": {"secilog": {"properties": {"logType": {"type": "string",
"index": "not_analyzed"}}}}
}
```

![FagQwd.md.png](https://s1.ax1x.com/2018/12/15/FagQwd.md.png)

### 删除索引

```json
DELETE /test_index
```

![FagdmQ.md.png](https://s1.ax1x.com/2018/12/15/FagdmQ.md.png)

### 获取索引

```json
GET /test_index1
{
  "test_index1": {
    "aliases": {},
    "mappings": {
      "secilog": {
        "properties": {
          "logType": {
            "type": "keyword"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1544886365769",
        "number_of_shards": "3",
        "number_of_replicas": "2",
        "uuid": "Iz8evLbCQ1CS85owEbKsgQ",
        "version": {
          "created": "5020299"
        },
        "provided_name": "test_index1"
      }
    }
  }
}
```

### 过滤查询

```json
索引的settings和mappings属性。可配置的属性包括 settings、 mappingswarmers 和 aliases。
GET /test_index1/_settings,_mapping, 如果索引不存在则返回404
{
  "test_index1": {
    "settings": {
      "index": {
        "creation_date": "1544886365769",
        "number_of_shards": "3",
        "number_of_replicas": "2",
        "uuid": "Iz8evLbCQ1CS85owEbKsgQ",
        "version": {
          "created": "5020299"
        },
        "provided_name": "test_index1"
      }
    },
    "mappings": {
      "secilog": {
        "properties": {
          "logType": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

```

### 关闭索引

![Fa2YNR.md.png](https://s1.ax1x.com/2018/12/15/Fa2YNR.md.png)

![Fa2R8P.md.png](https://s1.ax1x.com/2018/12/15/Fa2R8P.md.png)

### 映射管理

#### 增加映射

```json
PUT /test_index2
{
  "mappings": {
    "log": {
      "properties": {
        "message": {
          "type": "string"
        }
      }
    }
  }
#以上接口添加索引名为test_index2，文档类型为log，其中包含字段message,字段类型是字符串：
 PUT test_index2/_mapping/user
{
  "properties": {
    "name":{"type": "string"}
  }
}
# 添加文档类型为user，包含字段name,字段类型是字符串。
#设置多个索引映射时
PUT /{index}/_mapping/{type}
{index}可以有多种方式，逗号分隔：比如testl,test2，test3。
all表示所有索引3通配符*表示所有。test*表示以test开头。
{type}需要添加或更新的文档类型。
{body}需要添加的字段或字段类型
```

#### 获取映射

系统同时支持获取多个索引和类型的语法。
获取文档映射接口一次可以获取多个索引或文档映射类型。该接口通常是如下格式:
host:port/{index}/ mapping/{type}, {index}和{type}可以接受逗号（，）分隔符，也可以使用

```html
GET  test_index2/_mapping/{string}
GET /test_index2/_mapping/log,user
```

#### 判断存在

健查索引或文档类型是否存在  存在返回200 不存在返回404

```
HEAD test_index2/secilog
```

### 索引别名

Elasticsearch可以对一个或多个索引指定别名,通过别名可以查询一个或多个索引内容,在内部Elasticsearch会把别名映射在索引上,别名不可以和索引名其他索引别名重复

```json
POST /_aliases
{
  "actions":{"add":{"index":"test_index1","aliases":"othername1"}}
}
#给test_index增加索引别名othername1
```

#### 修改别名

别名没有修改的语法,当需要修改时,先删除 ,在添加

```json
POST /_aliases
{
  "actions":{"remove":{"index":"test_index1","aliases":"othername2"}}
}
```

#### 删除别名

```
DELETE  http://{host}:{port}/{index}/_alias/{name}
```

#### 查询别名

```
GET  http://{host}:{port}/{index}/_alias/{name}
```





