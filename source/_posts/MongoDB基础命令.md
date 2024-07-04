---
title: MongoDB基础命令
date: 2018-12-09 08:37:53
tags: MongoDB
categories: 数据库
---

## MongoDB 入门命令

### 查看当前数据库

```sql
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
>

-- use databaseName 选库
> use test
switched to db test
>

-- show tables/collections 查看当前库下的collection
> show tables
> show collections
>
```

### 基础操作

Mongodb的库是隐式创建,你可以use 一个不存在的库然后在该库下创建collection,即可创建库

```sql
--创建collection
--db.createCollection(‘collectionName’)  

> db.createCollection('test')
{ "ok" : 1 }
>
> show tables
test
>

--collection允许隐式创建
--Db.collectionName.insert(document);
> db.stu.insert({stu:'001',name:'xiaoming'})
WriteResult({ "nInserted" : 1 })
> show tables
stu
test

-- 删除collection
db.collectionName.drop()

-- 删除database
db.dropDatabase();
> db.dropDatabase()
{ "dropped" : "test", "ok" : 1 }
>
```



### 增

#### 插入数据

```json
> db.stu.insert({sid:"10"})
WriteResult({ "nInserted" : 1 })
> db.stu.insert({sid:"11"})
WriteResult({ "nInserted" : 1 })
```

```json
> db.stu.find()
{ "_id" : ObjectId("5c0c8a0b31a9b3cbb9df1d4f"), "sid" : "10" }
{ "_id" : ObjectId("5c0c8ac731a9b3cbb9df1d50"), "sid" : "11" }
```

添加数据时不添加任何主键,会制动生成一个主键,主键不会像关系型数据库那样自动递增(为了分布式考虑),使用的是时间戳+机器编号+进程编号+序列号来生成,保证每个id都是唯一的.id为5c0c8a0b31a9b3cbb9df1d4f,可以分解为  
5c0c8a0b 31a9b3  cbb9   df1d4f   (5c0c8a0b)表示时间戳, 31a9b3  表示机器号, cbb9   表示进程编号, df1d4f  表示序列号

我们也可以手动指定ID

```json
> db.stu.insert({_id:"001","sid":"12"})
WriteResult({ "nInserted" : 1 })
> db.stu.find()
{ "_id" : ObjectId("5c0c8a0b31a9b3cbb9df1d4f"), "sid" : "10" }
{ "_id" : ObjectId("5c0c8ac731a9b3cbb9df1d50"), "sid" : "11" }
{ "_id" : "001", "sid" : "12" }
>
```

#### 批量插入

```json
> db.user.insert([
... {username:"xiaoming",nickname:"XM",passwd:"123"} ,
... {username:"xiaogang",nickname:"XG",passwd:"111"},
... {username:"xiaohua",nickname:"XH",passwd:"123"}
... ])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 3,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
# 查看数据
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0c8f3531a9b3cbb9df1d51"),
        "username" : "xiaoming",
        "nickname" : "XM",
        "passwd" : "123"
}
{
        "_id" : ObjectId("5c0c8f3531a9b3cbb9df1d52"),
        "username" : "xiaogang",
        "nickname" : "XG",
        "passwd" : "111"
}
{
        "_id" : ObjectId("5c0c8f3531a9b3cbb9df1d53"),
        "username" : "xiaohua",
        "nickname" : "XH",
        "passwd" : "123"
}

```

执行插入数据的时候,驱动程序会把数据转换成为BSON格式,然后将数据输入数据库,数据库会解析BSON,并检验是否含有“_id”键,因为用户如果没有自定义"_id",会自动生成,而且每次插入的文档不能超过16M(插入文档的大小跟MongoDB版本有关系)

### 删除文档

##### 方式一

db.user.deleteMany({})

```json
> db.user.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 3 }
>
> db.user.remove({})
WriteResult({ "nRemoved" : 3 })
```

上述命令会删除user所有的文档,不删除集合本身,原有的索引也会保留,remove函数可以接收一个查询文档作为可选参数给定参数后,可以删除指定符条件的文档。

#####  方式二

```json
> db.user.remove({passwd:"123"})
WriteResult({ "nRemoved" : 2 })
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0c91d131a9b3cbb9df1d5b"),
        "username" : "xiaogang",
        "nickname" : "XG",
        "passwd" : "111"
}

```

删除数据是永久性的不可以撤销也不能恢复。

### 更新文档

在MongoDB中更新单个文档的操作是原子性的，默认情况下如果一个update操作多个文doc，那么对于每个doc的更新是原子性的,但是对于整个update操作而言,不是原子性的可能存在前面的doc更新成功,而后面的文档更新失败,由于更新单个文档doc的操作是原子性的,如果两个更新同时发生,那么一个更新操作会足协另外一个,doc的最终结果的值是由事件靠后的更新操作删除决定的.

#### 格式

db.collection.update(critera,objNew,upset,multi)

critera:查询条件

objNew:update对象和一些更新操作符

upset:如果存在update记录,是否插入objNew这个新文档,true为插入,默认为false

multi:默认是false,值更新找到的第一条记录,如果是true,按照条件查询出看来的记录全部更新

#### save

另一个更新命令是save 格式如下

db.collection.save(object)

obj表示要更新的对象,如果内部已经存在一个和obj相同的"_id"的记录纸Mongodb会把obj对象替换集合内已存在的记录,如果不存在,则会插入obj对象.

#### 文档替换

用于一个新文档完全替代匹配的文档,这种方法先把数据读出来,之后对对象的方式完成修改,这种方式一般用在修改较大的情况下:

```json
db.user.insertOne({ 
	name:"foo",
	nickname:"bar",
	friends:12,
	enemies:2
})
```

我们希望把数据修改成为

````json
db.user.insertOne({ 
	name:"foo",
	nickname:"bar",
	relations:{
    friends:12,
	enemies:2
	}
})
````



##### 步骤

查询对象存储到u中:

```json
var u = db.user.findOne({name:"foo"})
```

设置relations的值:

```json
 u.relations = {friends:u.friends,enemies:u.enemies}
```

修改username的值:

```json
> u.username = u.name
foo
```

删除friends:

```json
> u.username = u.name
foo
```

删除enmies:

```json
> delete u.enemies
true
```

删除name:

```json
 delete u.name
```

替换对象

```json
> db.user.update({name:"foo"},u)
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

查询

```json
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0cab14d22a51c6ef9cdcee"),
        "nickname" : "bar",
        "relations" : {
                "friends" : undefined,
                "enemies" : undefined
        },
        "username" : "foo"
}

```

这种替换基于编程思想来进行的这种方式对单个对象傅咋修改比较适用

#### 使用修改器

修改文档只修改文章的部分,而不是全部,这个时候我们可以使用修改器对文档进行更新,他的主要思想是通过*$*符号来进行修改这些操作

##### 增加和减少

inc可以对数据进行增加和减少,这个操作只针对数字类型,小数或者整数.

添加一条数据:

```json
> db.topic.insertOne({title:"first",version:108})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5c0cb1ba422725fda4bd5746")
}
> db.topic.find().pretty()
{
        "_id" : ObjectId("5c0cb1ba422725fda4bd5746"),
        "title" : "first",
        "version" : 108
}

```

将数字减少3

```json
> db.topic.update({"title" : "first"},{$inc:{version:-3}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.topic.find().pretty()
{
        "_id" : ObjectId("5c0cb1ba422725fda4bd5746"),
        "title" : "first",
        "version" : 105
}
```

#### $set修改器

使用  set 可以完成的顶的需求修改

```
原始数据
> db.author.find().pretty()
{
        "_id" : ObjectId("5c0cb444422725fda4bd5747"),
        "name" : "foo",
        "age" : 20,
        "gender" : "male",
        "intro" : "student"
}
```

将intro 修改为 teacher

```json
> db.author.update({name:"foo"},{$set:{intro:"teacher"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.author.find().pretty()
{
        "_id" : ObjectId("5c0cb444422725fda4bd5747"),
        "name" : "foo",
        "age" : 20,
        "gender" : "male",
        "intro" : "teacher"
}
>
```



#### $push修改器

使用push可以完成数组的插入,会在最后一条插入,如果没有这个key会自动创建一长条插入

```json
> db.post.find().pretty()
{
        "_id" : ObjectId("5c0cc527422725fda4bd5748"),
        "title" : "a blog",
        "content" : "...",
        "author" : "foo"
}
#s使用push插入数组

db.post.update({title:"a blog"},{$push:{comments:{name:"lina",email:"lina@email.com",content:"lina replay"}}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.post.find().pretty()
{
        "_id" : ObjectId("5c0cc527422725fda4bd5748"),
        "title" : "a blog",
        "content" : "...",
        "author" : "foo",
        "comments" : [
                {
                        "name" : "lina",
                        "email" : "lina@email.com",
                        "content" : "lina replay"
                }
        ]
}

```



#### addToSet修改器

使用addToSet可以向一个数组添加元素,有一个限定条件,如果存在了就不添加.

```json
{
        "_id" : ObjectId("5c0cc9f3cdb93a655448eec5"),
        "name" : "foo",
        "age" : 12,
        "email" : [
                "foo@example.com",
                "foo@163.com"
        ]
}
## 添加集合
> db.user.update({name:"foo"},{$addToSet:{email:"foo@qq.com"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
查询结果
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0cc9f3cdb93a655448eec5"),
        "name" : "foo",
        "age" : 12,
        "email" : [
                "foo@example.com",
                "foo@163.com",
                "foo@qq.com"
        ]
}
>
## 插入一个存在的数据
> db.user.update({name:"foo"},{$addToSet:{email:"foo@qq.com"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
> db.user.find().pretty()
{
        "_id" : ObjectId("5c0cc9f3cdb93a655448eec5"),
        "name" : "foo",
        "age" : 12,
        "email" : [
                "foo@example.com",
                "foo@163.com",
                "foo@qq.com"
        ]
}
```

nModified键的值为 0 ,因为已经添加了,所以执行添加语句的时候不会重复添加的,这种机制减少了数据库的冗余数据.

### 更新多个文档

```json
> db.clllections.update({x:1},{x:99})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.clllections.find().pretty()
{ "_id" : ObjectId("5c0ccc58cdb93a655448eec6"), "x" : 99 }
{ "_id" : ObjectId("5c0ccc61cdb93a655448eec7"), "x" : 1 }
{ "_id" : ObjectId("5c0ccc65cdb93a655448eec8"), "x" : 1 }
{ "_id" : ObjectId("5c0ccc6acdb93a655448eec9"), "x" : 2 }
>
只有第一条匹配了 采用如下命令
> db.clllections.update({x:1},{$set:{x:99}},false,true)
WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
> db.clllections.find().pretty()
{ "_id" : ObjectId("5c0ccc58cdb93a655448eec6"), "x" : 99 }
{ "_id" : ObjectId("5c0ccc61cdb93a655448eec7"), "x" : 99 }
{ "_id" : ObjectId("5c0ccc65cdb93a655448eec8"), "x" : 99 }
{ "_id" : ObjectId("5c0ccc6acdb93a655448eec9"), "x" : 2 }

首先我们要将修改的数据赋值给$set,$set是一个修改器,我们将在上文详细讲解过,然后后面多了两个参数,第一个flase表示如果不存在update记录,是否将我们要更新的新文档插入,true 表示插入 false 表示不插入,第二个true表示是否更新全部属性的文章,false表示值更新第一条记录,true表示更新所有查到的文档.
```
















