---
title: Redis
date: 2018-12-03 19:29:17
tags: Redis
categories: 数据库
---

## Nosql简介

NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指非关系型的数据库。随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题，包括超大规模数据的存储。

例如谷歌或Facebook每天为他们的用户收集万亿比特的数据。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

## Redis简介

### 历史与发展

2008年，意大利的一家创业公司Merzia推出了一款基于MySQL的网站实时统计系统LLOOGG，然而没过多久，公司的创始人开始对MySQL的性能感到失望，决定自己为该系统量身定做一个数据库；量身定做的数据库系统于2009年开发完成，名叫Redis（Remote Dictionary Server，远程字典服务）; Redis创始者还是很有开源精神的，它不仅仅想让Redis应用在他们公司的网站实时统计系统中，还希望更多的人使用它;在2009年将Redis开源发布，代码托管在Github上; Redis在短短几年就拥有了庞大的用户群体，Hacker News在2012年发布了一份数据库的使用情况调查，结果显示12%的公司在使用Redis； 国内的新浪微博，知乎,街旁; 国外的Github、Stack Overflow 、Flickr、暴雪和Instagram,都是Redis的用户

redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set -- 有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部分场合可以对关系数据库起到很好的补充作用。它提供了Java C/C++ C# PHP JavaScript Perl Object-C，Python，Ruby，Erlang等客户端，使用很方便。 
Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。
[官网](https://redis.io/)

### 特性

#### 存储结构

1. 字符串类型(sting)	Redis基本类型，一个key对应一个value，value最多可以是512M；String是二进制安全的（即redis的String可以包含任何数据）
2. 散列类型 (hash)    一个键值对集合，String类型的field和value映射表，类似Java里面的Map<String,Object>。
3. 列表类型(list)   字符串列表，按照插入顺序排序，底层实现是个链表。
4. 集合类型(set)  String类型的无序集合，且不允许重复的成员，通过Hashtable实现
5. 有序集合类型(zset)   ：在Set的基础上，不同的是每个元素都会关联一个double类型的分数，通过分数来为集合中的成员进行从小到大的排序。Zset的成员是唯一的，但分数（score）却可以重复

#### 常用命令

[基础命令链接](http://www.runoob.com/redis/redis-commands.html)

#### key的其他命令

```
 keys *
 exists key的名字，判断某个key是否存在
 move key db   --->当前库就没有了，被移除了
 expire key 秒钟：为给定的key设置过期时间
 ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期
 type key 查看你的key是什么类型
```

###### String其他命令

```
单值单value
set/get/del/append/strlen
Incr/decr/incrby/decrby,一定要是数字才能进行加减
getrange/setrange
getrange:获取指定区间范围内的值，类似between......and的关系   从零到负一表示全部
setrange设置指定区间范围内的值，格式是setrange key值 具体值
setex(set with expire)键秒值/setnx(set if not exist)  
				setex:设置带过期时间的key，动态设置。
				setnx:只有在 key 不存在时设置 key 的值。
 mset/mget/msetnx		
 				mset:同时设置一个或多个 key-value 对。  
 				mget:获取所有(一个或多个)给定 key 的值。
				msetnx:同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
 getset(先get再set)
 				getset:将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
				简单一句话，先get然后立即set
 						
```

###### List其他命令

```
单值多value
lpush/rpush/lrange
lpop/rpop  左删除 右删除
lindex，按照索引下标获得元素(从上到下)  通过索引获取列表中的元素 lindex key index
llen
lrem key 数字N 给定值v1    解释(删除N个值等于v1的元素)  
ltrim key 开始index 结束index，截取指定范围的值后再赋值给key
				ltrim：截取指定索引区间的元素，格式是ltrim list的key 起始索引 结束索引
rpoplpush 源列表 目的列表
				移除列表的最后一个元素，并将该元素添加到另一个列表并返回
lset key index value			
 linsert key  before/after 已有值 插入的新值
 				在list某个已有值的前后再添加具体值
 				
它是一个字符串链表，left、right都可以插入添加；
如果键不存在，创建新的链表；
如果键已存在，新增内容；
如果值全移除，对应的键也就消失了。
它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。
```

[![FMDtrd.md.png](https://s1.ax1x.com/2018/12/03/FMDtrd.md.png)](https://imgchr.com/i/FMDtrd)

###### Set其他命令

```
单值多value
sadd/smembers/sismember
scard，获取集合里面的元素个数
srem key value 删除集合中元素
srandmember key 某个整数(随机出几个数)
spop key 随机出栈
smove key1 key2 在key1里某个值      作用是将key1里已存在的某个值赋给key2

				差集：sdiff 在第一个set里面而不在后面任何一个set里面的项
数学集合类		 交集：sinter
				并集：sunion

```



###### Hash其他命令

```
KV模式不变，但V是一个键值对

hset/hget/hmset/hmget/hgetall/hdel
hlen
hexists key 在key里面的某个值的key
hkeys/hvals
hincrby/hincrbyfloat
hsetnx  		不存在赋值，存在了无效。
```

######  Zset其他命令

```
在set基础上，加一个score值。
		之前set是k1    v1 v2 v3，
		现在zset是k1 score1 v1 score2 v2
zrangebyscore key 开始score 结束score
zrem key 某score下对应的value值，作用是删除元素
				删除元素，格式是zrem zset的key 项的值，项的值可以是多个
				zrem key score某个对应值，可以是多个值
zcard key获得几个元素
				zcard ：获取集合中元素个数
				zrank： 获取value在zset中的下标位置
Zcount key score区间
				zscore：按照值获得对应的分数
				正序、逆序获得下标索引值
Zrank key values值，作用是获得下标值
Zscore key 对应值,获得分数
zrevrank key values值，作用是逆序获得下标值    正序、逆序获得下标索引值
zrevrange
zrevrangebyscore  key 结束score 开始score
```

#### 内存存储与持久化

Redis数据库中所有数据都存储在内存中.由于内存的读写速度快于硬盘,因此Redis在性能上对比其他基于硬盘存储的数据库有非常明显的优势,在一台普通的笔记本电脑上,可以在一秒内读取超过十万个键值.

```shell
redis-benchmark -q -n 100000 #在1核1Gcentos6.10上测试的Redis的吞吐量
```

[![Flca3q.png](https://s1.ax1x.com/2018/12/05/Flca3q.png)](https://imgchr.com/i/Flca3q)

####  功能丰富

Redis虽然做为数据库开发的,但是由于为其提供了丰富的功能,越来越多的人将其作用缓存.队列系统,Redis可谓是名副其实的多面手.

Redis可以为每个键设置生存时间(Time to Live,TTL)生存时间到期后键会自动被删除,这一功能配合出色的性能让Redis可以作为缓存系统来使用,而且Reids支持持久化和丰富的数据类型.

Redis是单线程模型,Memcached是多线程模型.多和服务器上Memcached的性能更高一些,但是Redis的性能已经足够优越,在绝大部分的场景下,其性能都不会成为瓶颈.所以在使用是应该个更加关心两者在功能上的区别如果是如到了*高级的数据类型*或者是持久化等功能,Redis将会是Memcached很好的替代品.

作为缓存系统,Redis可以限定数据占用的最大的内存空间,在数据达到空间先之后,可以按照一定的规则自动淘汰不需要的键,

Redis的列表类型键可以用来实现队列,并且支持阻塞式读取,可以很容易的实现一个高性能的优先级队列,同时在更高舱面上,Redis还可支持"发布/订阅"的消息模式,可以基于次构建聊天室系统.

除基本的会话token之外，Redis还提供很简便的FPC平台。回到一致性问题，即使重启了Redis实例，因为有磁盘的持久化，用户也不会看到页面加载速度的下降，这是一个极大改进，类似PHP本地FPC。

再次以Magento为例，Magento提供一个插件来使用Redis作为[全页缓存后端](https://github.com/colinmollenhour/Cm_Cache_Backend_Redis)。

此外，对WordPress的用户来说，Pantheon有一个非常好的插件  [wp-redis](https://wordpress.org/plugins/wp-redis/)，这个插件能帮助你以最快速度加载你曾浏览过的页面。

Redis在内存中对数字进行递增或递减的操作实现的非常好。集合（Set）和有序集合（Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis只是正好提供了这两种数据结构。所以，我们要从排序集合中获取到排名最靠前的10个用户–我们称之为“user_scores”，我们只需要像下面一样执行即可：

当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户及用户的分数，你需要这样执行：

ZRANGE user_scores 0 10 WITHSCORES

#### 简单稳定

Redis使用命令来读写数据,命令语句之于Redis就相当于SQL语言之对于关系数据库,列入在关系数据库中想要获取posts表内的id为1的记录的title字段的值可以使用如下SQL语句来实现。

```sql
SELECT title FROM posts WHERE id=1 LIMIT=1  #SQL语句
```

```redis
HGET post:1 title
```

Redis 提供了几十种不同编程语言的客户端库，这些库都很好的封装了Redis的命令，是的在程序中与Redis进行交互变得更加容易。有些库还提供了可以将编程语言的数据类型直接以对应的形式存储到Redis中（如将数组直接以列表类型存到Redis）的简单方法，使用起来非常方便。

同时Redis使用C语言开发，代码量只有3万多行。这降低了用户通过Redis源代码来使之更加适合自己项目需要的门槛。对于希望“榨干”数据库性能开发者而，这无疑是一个很大的吸引力。因此Redis的开发不仅仅是Salvatore Sanfilippo 和*Pieter* *Noordhuis* 有近100名开发者为Redis贡献了代码。良好的开发分为和严谨的发布机制使得Redis的版本非常可靠，如次多的公司在项目中使用后Redis也可以印证这以一点。

#### memcached与Redis比较

| 数据库    | CPU      | 内存使用率             | 持久性                   | 数据结构 | 工作环境      |
| --------- | -------- | ---------------------- | ------------------------ | -------- | ------------- |
| memcached | 支持多核 | 高                     | 无                       | 简单     | Linux/Windos  |
| Redis     | 单核     | 低（压缩比memcache高） | 有（硬盘存储，主从同步） | 复杂     | 推荐使用Linux |



