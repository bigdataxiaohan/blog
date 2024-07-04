---
title: Spark之RDD实战篇3
date: 2019-05-29 19:06:25
tags: Spark
categories: 大数据
---

##  键值对RDD

Spark 为包含键值对类型的 RDD 提供了一些专有的操作 在PairRDDFunctions专门进行了定义。这些 RDD 被称为 pair RDD。有很多种方式创建`pair RDD`，在输入输出章节会讲解。一般如果从一个普通的RDD转 为pair RDD时，可以调用map()函数来实现，传递的函数需要返回键值对。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529191639.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529191720.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529191855.png)

果从一个普通的RDD转为`pair RDD`时，可以调用map()函数来实现，传递的函数需要返回键值对。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529192923.png)

### 转化操作列表

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529193023.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529193034.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529193100.png)

### 聚合操作

当数据集以键值对形式组织的时候，聚合具有相同键的元素进行一些统计是很常见的操作。之前讲解过基础RDD上的fold()、combine()、reduce()等行动操作，pair RDD上则 有相应的针对键的转化操作。Spark 有一组类似的操作，可以组合具有相同键的值。这些 操作返回 RDD，因此它们是转化操作而不是行动操作。 

reduceByKey() 与 reduce() 相当类似;它们都接收一个函数，并使用该函数对值进行合并。 reduceByKey() 会为数据集中的每个键进行并行的归约操作，每个归约操作会将键相同的值合 并起来。因为数据集中可能有大量的键，所以 reduceByKey() 没有被实现为向用户程序返回一 个值的行动操作。实际上，它会返回一个由各键和对应键归约出来的结果值组成的新的 RDD。 

foldByKey() 则与 fold() 相当类似;它们都使用一个与 RDD 和合并函数中的数据类型相 同的零值作为初始值。与 fold() 一样，foldByKey() 操作所使用的合并函数对零值与另一 个元素进行合并，结果仍为该元素。 

```scala
val rdd =sc.parallelize(Array(("panda",0),("pink",3),("pirate",3),("panda",1),("pink",4)),2)
val result =rdd.mapValues(x => (x, 1)).reduceByKey((x, y) => (x._1 + y._1, x._2 + y._2))
result.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529195651.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529195711.png)

combineByKey() 是常用的基于键进行聚合的函数。大多数基于键聚合的函数都是用它实现的。和 aggregate() 一样，combineByKey() 可以让用户返回与输入数据的类型不同的 返回值。 

由于 combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就 和之前的某个元素的键相同。 

如果这是一个新的元素，combineByKey() 会使用一个叫作 createCombiner() 的函数来创建 那个键对应的累加器的初始值。需要注意的是，这一过程会在每个分区中第一次出现各个键时发生，而不是在整个 RDD 中第一次出现一个键时发生。 

如果这是一个在处理当前分区之前已经遇到的键，它会使用 mergeValue() 方法将该键的累加器对应的当前值与这个新的值进行合并。 

由于每个分区都是独立处理的，因此对于同一个键可以有多个累加器。如果有两个或者更 多的分区都有对应同一个键的累加器，就需要使用用户提供的 mergeCombiners() 方法将各 个分区的结果进行合并。 

```scala
val rdd =sc.parallelize(Array(("panda",0),("pink",3),("pirate",3),("panda",1),("pink",4)),2)
val result = rdd.combineByKey(
  (v) => (v, 1),
  (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),
  (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
)
result.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529200208.png)

### 数据分组

如果数据已经以预期的方式提取了键，groupByKey() 就会使用 RDD 中的键来对数据进行 分组。对于一个由类型 K 的键和类型 V 的值组成的 RDD，所得到的结果 RDD 类型会是 [K, Iterable[V]]。 

多个RDD分组，可以使用cogroup函数，cogroup() 的函数对多个共享同 一个键的 RDD 进行分组。对两个键的类型均为 K 而值的类型分别为 V 和 W 的 RDD 进行 cogroup() 时，得到的结果 RDD 类型为 [(K, (Iterable[V], Iterable[W]))]。如果其中的 一个 RDD 对于另一个 RDD 中存在的某个键没有对应的记录，那么对应的迭代器则为空。 cogroup() 提供了为多个 RDD 进行数据分组的方法。 

```scala
var rddl = sc.makeRDD(Array(("A",0), ("A",2), ("B",1), ("B",2), ("Cn",1)))
rdd1.groupByKey().collect
//使用reduceByKey操作将RDD[K,V]中每个K对应的V值根据映射函数来运算                 
var rdd2 = rdd1.reduceByKey((x,y) => x + y)
//对rddl使用reduceByKey操作进行重新分区
var rdd2 = rdd1.reduceByKey (new org.apache.spark.HashPartitioner(2),(x, y) => x + y)
rdd2.collect

var rdd2 = sc.makeRDD(Array(("A","a"),("C","c"),("D","d")),2)
var rdd3 = sc.makeRDD(Array(("A","A"),("E","E")),2)
var rdd4 = rddl.cogroup(rdd2,rdd3)
rdd4.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529203008.png)

### 连接

连接主要用于多个Pair RDD的操作，连接方式多种多样:右外连接、左外连接、交 叉连接以及内连接。 

普通的 join 操作符表示内连接 2。只有在两个 pair RDD 中都存在的键才叫输出。当一个输 入对应的某个键有多个值时，生成的pair RDD会包括来自两个输入RDD的每一组相对应 的记录。 

leftOuterJoin()产生的pair RDD中，源RDD的每一个键都有对应的记录。每个 键相应的值是由一个源 RDD 中的值与一个包含第二个 RDD 的值的 Option(在 Java 中为 Optional)对象组成的二元组。 

rightOuterJoin() 几乎与 leftOuterJoin() 完全一样，只不过预期结果中的键必须出现在第二个 RDD 中，而二元组中的可缺失的部分则来自于源 RDD 而非第二个 RDD。这些连接操作都是继承了cgroup

```scala
var rdd1 = sc.makeRDD (Array(("A", "1"), ("B","2") , ("C","3")), 2)
var rdd2 = sc.makeRDD (Array(("A", "a"), ("C","c") , ("D","d")), 2)
//进行内连接操作
rdd1.join(rdd2).collect
//进行左连接操作
rdd1.leftOuterJoin(rdd2).collect
rdd1.leftOuterJoin(rdd2).collect                      
//进行右连接操作
rdd1.rightOuterJoin(rdd2).collect                                      
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529204438.png)

### 数据排序

sortByKey() 函数接收一个叫作 ascending 的参数，表示我们是否想要让结果按升序排序(默认值为 true)。 

```scala
··val rdd = sc.parallelize(Array((3,"hadoop"),(6,"hohblog"),(2,"flink"),(1,"spark")))
rdd.sortByKey(true).collect()
rdd.sortByKey(false).collect()
```

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E5%A4%A7%E6%95%B0%E6%8D%AE/Spark/RDD/1559015691057.png)

### 行动操作

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529204942.png)

### 数据分区

Spark目前支持Hash分区和Range分区，用户也可以自定义分区，Hash分区为当前的默认分区，Spark中分区器直接决定了RDD中分区的个数、RDD中每条数据经过Shuffle过程属于哪个分区和Reduce的个数，注意：

(1)只有Key-Value类型的RDD才有分区的，非Key-Value类型的RDD分区的值是None。
(2)每个RDD的分区ID范围：0~numPartitions-1，决定这个值是属于那个分区的。

```scala
val pairs =sc.parallelize(List((1,1),(2,2),(3,3)))
pairs.partitioner
val partitioned = pairs.partitionBy(new org.apache.spark.HashPartitioner(2))
partitioned.partitioner
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529210421.png)

#### Hash分区方式

对于给定的key，计算其hashCode，并除于分区的个数取余，如果余数小于0，则用余数+分区的个数，最后返回的值就是这个key所属的分区ID。

```scala
val nopar = sc.parallelize(List((1,3),(1,2),(2,4),(2,3),(3,6),(3,8)),8)
nopar.partitioner
nopar.mapPartitionsWithIndex((index,iter)=>{ Iterator(index.toString+" : "+iter.mkString("|")) }).collect
val hashpar = nopar.partitionBy(new org.apache.spark.HashPartitioner(7))
hashpar.count
hashpar.partitioner
hashpar.mapPartitions(iter => Iterator(iter.length)).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529210826.png)

#### Ranger分区方式

HashPartitioner分区弊端：可能导致每个分区中数据量的不均匀，极端情况下会导致某些分区拥有RDD的全部数据。

RangePartitioner分区优势：尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，一个分区中的元素肯定都是比另一个分区内的元素小或者大；

但是分区内的元素是不能保证顺序的。简单的说就是将一定范围内的数映射到某一个分区内。

RangePartitioner作用：将一定范围内的数映射到某一个分区内，在实现中，分界的算法用到了[水塘抽样算法](<https://www.cnblogs.com/krcys/p/9121487.html>)。

```scala
val nopar = sc.parallelize(List((1,3),(1,2),(2,4),(2,3),(3,6),(3,8)),8)
nopar.partitioner
nopar.mapPartitionsWithIndex((index,iter)=>{ Iterator(index.toString+" : "+iter.mkString("|")) }).collect
val rangepar = nopar.partitionBy(new org.apache.spark.RangePartitioner(2,nopar))
rangepar.count
rangepar.partitioner
rangepar.mapPartitions(iter => Iterator(iter.length)).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529211534.png)

#### 自定义分区

要实现自定义的分区器，你需要继承 org.apache.spark.Partitioner 类并实现下面三个方法。 

numPartitions: Int:返回创建出来的分区数。

getPartition(key: Any): Int:返回给定键的分区编号(0到numPartitions-1)。 

equals():Java 判断相等性的标准方法。这个方法的实现非常重要，Spark 需要用这个方法来检查你的分区器对象是否和其他分区器实例相同，这样 Spark 才可以判断两个 RDD 的分区方式是否相同。  

假设我们需要将相同后缀的数据写入相同的文件，我们通过将相同后缀的数据分区到相同的分区并保存输出来实现。

```scala
val data=sc.parallelize(List("aa.2","bb.2","cc.3","dd.3","ee.5").zipWithIndex,2)
data.collect
data.mapPartitionsWithIndex((index,iter)=>Iterator(index.toString +" : "+ iter.mkString("|"))).collect

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529215147.png)

```scala
import org.apache.spark.{Partitioner, SparkConf, SparkContext}
class CustomerPartitioner(numParts:Int) extends org.apache.spark.Partitioner{

  //覆盖分区数
  override def numPartitions: Int = numParts

  //覆盖分区号获取函数
  override def getPartition(key: Any): Int = {
    val ckey: String = key.toString
    ckey.substring(ckey.length-1).toInt%numParts
  }
}

val result = data.partitionBy(new CustomerPartitioner(4))
result.mapPartitionsWithIndex((index,iter)=>Iterator(index.toString +" : "+ iter.mkString("|"))).collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529215215.png)

使用自定义的 Partitioner 是很容易的:只要把它传给 partitionBy() 方法即可。Spark 中有许多依赖于数据混洗的方法，比如 join() 和 groupByKey()，它们也可以接收一个可选的 Partitioner 对象来控制输出数据的分区方式。

#### 分区shuffle优化

在分布式程序中， 通信的代价是很大的，因此控制数据分布以获得最少的网络传输可以极大地提升整体性能。 

Spark 中所有的键值对 RDD 都可以进行分区。系统会根据一个针对键的函数对元素进行分组。 主要有哈希分区和范围分区，当然用户也可以自定义分区函数。

通过分区可以有效提升程序性能。如下例子：

它在内存中保存着一张很大的用户信息表—— 也就是一个由 (UserID, UserInfo) 对组成的 RDD，其中 UserInfo 包含一个该用户所订阅 的主题的列表。该应用会周期性地将这张表与一个小文件进行组合，这个小文件中存着过 去五分钟内发生的事件——其实就是一个由 (UserID, LinkInfo) 对组成的表，存放着过去五分钟内某网站各用户的访问情况。例如，我们可能需要对用户访问其未订阅主题的页面 的情况进行统计。 

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529215423.png)

代码可以正确运行，但是不够高效。这是因为在每次调用 processNewLogs() 时都会用到 join() 操作，而我们对数据集是如何分区的却一无所知。默认情况下，连接操作会将两个数据集中的所有键的哈希值都求出来，将该哈希值相同的记录通过网络传到同一台机器上，然后在那台机器上对所有键相同的记录进行连接操作。因为 userData 表比每五分钟出现的访问日志表 events 要大得多，所以要浪费时间做很多额外工作:在每次调用时都对 userData 表进行哈希值计算和跨节点数据混洗，降低了程序的执行效率。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529215459.png)

我们在构建 userData 时调用了 partitionBy()，Spark 就知道了该 RDD 是根据键的哈希值来分区的，这样在调用 join() 时，Spark 就会利用到这一点。具体来说，当调用 userData. join(events) 时，Spark 只会对 events 进行数据混洗操作，将 events 中特定 UserID 的记 录发送到 userData 的对应分区所在的那台机器上。这样，需要通过网络传输的 数据就大大减少了，程序运行速度也可以显著提升了。 

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529215606.png)

#### 基于分区进行操作

基于分区对数据进行操作可以让我们避免为每个数据元素进行重复的配置工作。诸如打开 数据库连接或创建随机数生成器等操作，都是我们应当尽量避免为每个元素都配置一次的 工作。Spark 提供基于分区的 mapPartition 和 foreachPartition，让你的部分代码只对 RDD 的每个分区运行 一次，这样可以帮助降低这些操作的代价。

#### 从分区中获益的操作

能够从数据分区中获得性能提升的操作有cogroup()、 groupWith()、join()、leftOuterJoin()、rightOuterJoin()、groupByKey()、reduceByKey()、 combineByKey() 以及 lookup()等。

### 数据读取与保存

#### 文本文件

当我们将一个文本文件读取为RDD 时，输入的每一行都会成为RDD的一个元素。也可以将多个完整的文本文件一次性读取为一个pair RDD，其中键是文件名，值是文件内容。

如果传递目录，则将目录下的所有文件读取作为RDD。

文件路径支持通配符。通过wholeTextFiles()对于大量的小文件读取效率比较高，大文件效果没有那么高。

Spark通过saveAsTextFile() 进行文本文件的输出，该方法接收一个路径，并将 RDD 中的内容都输入到路径对应的文件中。Spark 将传入的路径作为目录对待，会在那个 目录下输出多个文件。这样，Spark 就可以从多个节点上并行输出了。 

```scala
val rdd = sc.textFile("/input/test.txt")
rdd.collect
val rdd = sc.textFile("/input/*")
rdd.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529220140.png)

#### JSON文件

JSON文件中每一行就是一个JSON记录，可以通过将JSON文件当做文本文件来读取，然后利用相关的JSON库对每一条数据进行JSON解析。

```json
{"name":"Hadoop","age":13}
{"name":"Spark", "age":11}
{"name":"Flink", "age":3}
```

```scala
package com.hph

import org.apache.spark.{SparkConf, SparkContext}
import org.json4s.ShortTypeHints
import org.json4s.jackson.JsonMethods._
import org.json4s.jackson.Serialization

object TestJson {

  case class BigData(name:String,year:Int)

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setMaster("local[*]").setAppName("JSON")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    implicit val formats = Serialization.formats(ShortTypeHints(List()))
    val input = sc.textFile("D:\\input\\people.json")

    input.collect().foreach(x => {var c = parse(x).extract[BigData];println(c.name + "," + c.year)})

  }

}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529225006.png)

####  CSV文件

读取 CSV/TSV 数据和读取 JSON 数据相似，都需要先把文件当作普通文本文件来读取数据，然后通过将每一行进行解析实现对CSV的读取。CSV/TSV数据的输出也是需要将结构化RDD通过相关的库转换成字符串RDD，然后使用 Spark 的文本文件 API 写出去。

#### Sequence文件

 SequenceFile文件是[Hadoop](http://lib.csdn.net/base/hadoop)用来存储二进制形式的key-value对而设计的一种平面文件(Flat File)。

 Spark 有专门用来读取 SequenceFile 的接口。在 SparkContext 中，可以调用 sequenceFile[ keyClass, valueClass](path)。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529225226.png)

#### 对象文件

对象文件是将对象序列化后保存的文件，采用Java的序列化机制。可以通过objectFile[k,v](path) 函数接收一个路径，读取对象文件，返回对应的 RDD，也可以通过调用saveAsObjectFile() 实现对对象文件的输出。因为是序列化所以要指定类型。

```scala
val data=sc.parallelize(List((1,"hphblog"),(2,"Spark"),(3,"Flink"),(4,"SpringBoot"),(5,"SpringCloud")))
data.saveAsObjectFile("hdfs://datanode1:9000/objfile")
val objrdd:org.apache.spark.rdd.RDD[(Int,String)] = sc.objectFile[(Int,String)]("hdfs://datanode1:9000/objfile/p*")
objrdd.collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529231917.png)

#### HDFS

Spark的整个生态系统与Hadoop是完全兼容的,所以对于Hadoop所支持的文件类型或者数据库类型,Spark也同样支持.另外,由于Hadoop的API有新旧两个版本,所以Spark为了能够兼容Hadoop所有的版本,也提供了两套创建操作接口.对于外部存储创建操作而言,hadoopRDD和newHadoopRDD是最为抽象的两个函数接口,主要包含以下四个参数.

1）输入格式(InputFormat): 制定数据输入的类型,如TextInputFormat等,新旧两个版本所引用的版本分别是org.apache.hadoop.mapred.InputFormat和org.apache.hadoop.mapreduce.InputFormat(NewInputFormat)

2）键类型: 指定[K,V]键值对中K的类型

3）值类型: 指定[K,V]键值对中V的类型

4）分区值: 指定由外部存储生成的RDD的partition数量的最小值,如果没有指定,系统会使用默认值defaultMinSplits

其他创建操作的API接口都是为了方便最终的Spark程序开发者而设置的,是这两个接口的高效实现版本.例如,对于textFile而言,只有path这个指定文件路径的参数,其他参数在系统内部指定了默认值

分为新旧API，**注意:**

1.在Hadoop中以压缩形式存储的数据,不需要指定解压方式就能够进行读取,因为Hadoop本身有一个解压器会根据压缩文件的后缀推断解压算法进行解压.

2.如果用Spark从Hadoop中读取某种类型的数据不知道怎么读取的时候,上网查找一个使用map-reduce的时候是怎么读取这种这种数据的,然后再将对应的读取方式改写成上面的hadoopRDD和newAPIHadoopRDD两个类就行了

```scala
val data = sc.parallelize(Array((1,"Hadoop"), (2,"Spark"), (3,"Flink")))
data.saveAsHadoopFile("hdfs://datanode1:9000/output/hdfs_spark",classOf[Text],classOf[IntWritable],classOf[TextOutputFormat[Text,IntWritable]])
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529233324.png)

```scala
val data = sc.parallelize(Array(("Hadoop",1), ("Spark",2), ("Flink",3)))
data.saveAsNewAPIHadoopFile("hdfs://datanod1:9000/output/NewAPI/",classOf[Text],classOf[IntWritable] , classOf[org.apache.hadoop.mapreduce.OutputFormat[Text,IntWritable]])
```

#### 文件系统

Spark 支持读写很多种文件系统， 像本地文件系统、Amazon S3、HDFS等甚至是腾讯和阿里的COS等。

#### 数据库

支持通过Java JDBC访问关系型数据库。需要通过JdbcRDD进行，不过需要我们把驱动包放入Spark的

```scala
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by 清风笑丶 Cotter on 2019/5/30.
  */
object JDBCRdd {
  def main (args: Array[String] ) {
    val sparkConf = new SparkConf ().setMaster ("local[*]").setAppName ("JdbcApp")
    val sc = new SparkContext (sparkConf)
    val rdd = new org.apache.spark.rdd.JdbcRDD (
      sc,
      () => {
        Class.forName ("com.mysql.jdbc.Driver").newInstance()
        java.sql.DriverManager.getConnection ("jdbc:mysql://datanode1:3306/rdd", "root", "123456")
      },
      "select * from rddtable where id >= ? and id <= ?;",  //SQL
      1,   // 下界
      10, //上界
      1, //分区数
      r => (r.getInt(1), r.getString(2)))

    println (rdd.count () )
    rdd.foreach (println (_) )
    sc.stop ()
  }

}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530094256.png)![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530094311.png)

Mysql写入：

```scala
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by 清风笑丶 Cotter on 2019/5/30.
  */
object JDBCRDD2MySQL {
  def main(args: Array[String]) {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HBaseApp")
    val sc = new SparkContext(sparkConf)
    val data = sc.parallelize(List("JDBC2Mysql", "JDBCSaveToMysql","RDD2Mysql"))

    data.foreachPartition(insertData)
  }

  def insertData(iterator: Iterator[String]): Unit = {
    Class.forName ("com.mysql.jdbc.Driver").newInstance()
    val conn = java.sql.DriverManager.getConnection("jdbc:mysql://datanode1:3306/rdd", "root", "hive")
    iterator.foreach(data => {
      val ps = conn.prepareStatement("insert into rddtable(name) values (?)")
      ps.setString(1, data)
      ps.executeUpdate()
    })
  }

}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530095104.png)

JdbcRDD 接收这样几个参数。 

• 首先，要提供一个用于对数据库创建连接的函数。这个函数让每个节点在连接必要的配置后创建自己读取数据的连接。 

• 接下来，要提供一个可以读取一定范围内数据的查询，以及查询参数中`lowerBound`和 `upperBound` 的值。这些参数可以让 Spark 在不同机器上查询不同范围的数据，这样就不会因尝试在一个节点上读取所有数据而遭遇性能瓶颈。

• 这个函数的最后一个参数是一个可以将输出结果从转为对操作数据有用的格式的函数。如果这个参数空缺，Spark会自动将每行结果转为一个对象数组。 

```scala
create 'fruit','info'
put 'fruit','1001','info:name','Apple'
put 'fruit','1001','info:color','Read'
put 'fruit','1002','info:name','Banana'
put 'fruit','1002','info:color','Yelow'
```

````scala
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by 清风笑丶 Cotter on 2019/5/30.
  */
object ReadHBase {
  def main(args: Array[String]) {


    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HBaseApp")
    val sc = new SparkContext(sparkConf)

    sc.setLogLevel("ERROR")
    val conf = HBaseConfiguration.create()
    conf.set("hbase.zookeeper.quorum", "192.168.1.101");
    conf.set("hbase.zookeeper.property.clientPort", "2181")
    //HBase中的表名
    conf.set(TableInputFormat.INPUT_TABLE, "fruit")

    val hBaseRDD = sc.newAPIHadoopRDD(conf, classOf[TableInputFormat],
      classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable],
      classOf[org.apache.hadoop.hbase.client.Result])

    val count = hBaseRDD.count()
    println("hBaseRDD RDD Count:" + count)
    hBaseRDD.cache()
    hBaseRDD.foreach {
      case (_, result) =>
        val key = Bytes.toString(result.getRow)
        val name = Bytes.toString(result.getValue("info".getBytes, "name".getBytes))
        val color = Bytes.toString(result.getValue("info".getBytes, "color".getBytes))
        println("Row key:" + key + " Name:" + name + " Color:" + color)
    }
    sc.stop()
  }
}
````



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530104837.png)

```scala
import org.apache.hadoop.hbase.client.{HBaseAdmin, Put}
import org.apache.hadoop.hbase.{HBaseConfiguration, HColumnDescriptor, HTableDescriptor, TableName}
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.mapred.TableOutputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.mapred.JobConf
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by 清风笑丶 Cotter on 2019/5/30.
  */
object Write2Hbase {
  def main(args: Array[String]) {
    val sparkConf = new SparkConf().setMaster("local[2]").setAppName("HBaseApp")
    val sc = new SparkContext(sparkConf)

    sc.setLogLevel("ERROR")
    val conf = HBaseConfiguration.create()
    conf.set("hbase.zookeeper.quorum", "192.168.1.101");
    conf.set("hbase.zookeeper.property.clientPort", "2181")

    val jobConf = new JobConf(conf)
    jobConf.setOutputFormat(classOf[TableOutputFormat])
    jobConf.set(TableOutputFormat.OUTPUT_TABLE, "fruit_spark")

    val fruitTable = TableName.valueOf("fruit_spark")
    val tableDescr = new HTableDescriptor(fruitTable)
    tableDescr.addFamily(new HColumnDescriptor("info".getBytes))

    val admin = new HBaseAdmin(conf)
    if (admin.tableExists(fruitTable)) {
      admin.disableTable(fruitTable)
      admin.deleteTable(fruitTable)
    }
    admin.createTable(tableDescr)

    def convert(triple: (Int, String, Int)) = {
      val put = new Put(Bytes.toBytes(triple._1))
      put.addImmutable(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes(triple._2))
      put.addImmutable(Bytes.toBytes("info"), Bytes.toBytes("price"), Bytes.toBytes(triple._3))
      (new ImmutableBytesWritable, put)
    }
    val initialRDD = sc.parallelize(List((1,"apple",11), (2,"banana",12), (3,"pear",13)))
    val localData = initialRDD.map(convert)

    localData.saveAsHadoopDataset(jobConf)
  }

}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530105359.png)

### 共享变量

####  累加器

一个全局共享变量,可以完成对信息进行操作,相当于MapReduce中的计数器, Spark 传递函数时，比如使用 map() 函数或者用 filter() 传条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本， 更新这些副本的值也不会影响驱动器中的对应变量。 如果我们想实现所有分片处理时更新共享变量的功能，那么累加器可以实现我们想要的效果。

```scala
val notice = sc.textFile("file:///opt/module/spark/README.md")
 val blanklines = sc.accumulator(0)
val tmp = notice.flatMap(line => {
         if (line == "") {
            blanklines += 1
         }
         line.split(" ")
      })
tmp.count()
blanklines.value
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530111120.png)

```scala
  @deprecated("use AccumulatorV2", "2.0.0")
  def accumulator[T](initialValue: T)(implicit param: AccumulatorParam[T]): Accumulator[T] = {
    val acc = new Accumulator(initialValue, param)
    cleaner.foreach(_.registerAccumulatorForCleanup(acc.newAcc))
    acc
  }
```

通过在驱动器中调用`SparkContext.accumulator(initialValue)`方法，创建出存有初始值的累加器。返回值为
`org.apache.spark.Accumulator[T]`对象，T 是初始值 initialValue 的类型。Spark闭包里的执行器代码可以使用累加器的 += 方法(在Java中是 add)增加累加器的值。 驱动器程序可以调用累加器的value属性(在Java中使用value()或setValue())来访问累加器的值。 

为什么有了reduce()这样的聚合操作了,还要累加器呢?因为RDD本身提供的同步机制力度太粗,尤其是在转换操作中变量状态不能同步,累加器可以对那些与RDD本身的范围和粒度不一样的值进行聚合,只不过它是一个只写变量,无法读取这个值,只能在驱动程序中读取累加器的值。

#### 自定义累加器

在2.0版本后，累加器的易用性有了较大的改进，而且官方还提供了一个新的抽象类：AccumulatorV2来提供更加友好的自定义类型累加器的实现方式。实现自定义类型累加器需要继承AccumulatorV2并至少覆写下例中出现的方法，下面这个累加器可以用于在程序运行过程中收集一些文本类信息，最终以Set[String]的形式返回。

```scala
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.JavaConversions._

class LogAccumulator extends org.apache.spark.util.AccumulatorV2[String, java.util.Set[String]] {
  private val _logArray: java.util.Set[String] = new java.util.HashSet[String]()

  override def isZero: Boolean = {
    _logArray.isEmpty
  }

  override def reset(): Unit = {
    _logArray.clear()
  }

  override def add(v: String): Unit = {
    _logArray.add(v)
  }

  override def merge(other: org.apache.spark.util.AccumulatorV2[String, java.util.Set[String]]): Unit = {
    other match {
      case o: LogAccumulator => _logArray.addAll(o.value)
    }

  }

  override def value: java.util.Set[String] = {
    java.util.Collections.unmodifiableSet(_logArray)
  }

  override def copy():org.apache.spark.util.AccumulatorV2[String, java.util.Set[String]] = {
    val newAcc = new LogAccumulator()
    _logArray.synchronized{
      newAcc._logArray.addAll(_logArray)
    }
    newAcc
  }
}

// 过滤掉带字母的
object LogAccumulator {
  def main(args: Array[String]) {
    val conf=new SparkConf().setAppName("LogAccumulator").setMaster("local[*]")
    val sc=new SparkContext(conf)

    val accum = new LogAccumulator
    sc.register(accum, "logAccum")
    val sum = sc.parallelize(Array("1", "2a", "3", "4b", "5", "6", "7cd", "8", "9"), 2).filter(line => {
      val pattern = """^-?(\d+)"""
      val flag = line.matches(pattern)
      if (!flag) {
        accum.add(line)
      }
      flag
    }).map(_.toInt).reduce(_ + _)  //1+3+5+6+7+8+9 =32

    println("sum: " + sum)
    for (v <- accum.value) print(v + "")
    println()
    sc.stop()
  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530124720.png)

#### 广播变量

Spark的算子逻辑是发送到Executor中运行的，数据是分区的，因此当Executor中需要引用外部变量的时候，就需要我们用到广播变量(Broadcast)

累加器相当于统筹大变量，通常用于计数，统计广播变量允许程序员缓存一个只读的变量在每一台机器上（worker）上，而不是每一个任务保存一份备份。利用广播变量可以以更有效的方式将大数据量输入集合的副本分配到每一个节点。

广播变量通过两方面提高数据共享效率：

1)集群重的每一个节点(物理机器)只有一个副本，默认的闭包是每一个任务一个副本；

2)广播传输时通过BT下载模式实现的，也就是P2P下载的，在集群很多的情况下可以极大地提高数据传输速率。广播变量修改后，不会反馈到其他节点。

在Spark中，它会自动把所有音容变量发送到工作节点是，虽然很方便，但是效率比较低：

1)默认地任务发射机制时专门为小任务进行优化的。

2)实际过程中可能会在多个并行操作中使用同一个变量，而Spark会分别为每个操作发送这个变量。

```scala
val broadcastVar = sc.broadcast(Array(1,2,3,4,5))
broadcastVar.value
sc.parallelize(Array(1,2,3,4,5,6,7,8)).flatMap(x => (broadcastVar.value)).collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530182120.png)

广播变量内部存储地数据量较小地时候可以进行高效地广播，当这个变量变得非常大地时候，例如:在广播规则库的时候，规则库比较大，从主节点发送这样的一个规则数组非常消耗内存，如果之后还需要用到规则库这个变量，则需要再向每个节点发送一遍，同时如果一个节点的Executor中多个Task都用到这个变量，那么每个Task中都需要从driver端发送一份规则库的变量，最终导致占用的内存空间很大，如果变量为外部变量，进行广播前要进行collect操作。

```scala
val broadcas = sc.textFile("file:///opt/module/spark/README.md")
val broadcasRDD = broadcas.collect
val c = sc.broadcast(broadcasRDD)
c.value
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190530184944.png)

我们通过调用一个对象SparkContext.broadcast创建一个Broadcast对象，任何可以序列化对象都可以这样实现。需要注意的是，如果变量是从外部读取的，需要先进行collect操作，再进行广播，给如果广播的值比较大，可以选择即快又好的序列化格式。在Scala和Java API中默认使用Java序列化库，对于除基本的数组以外的任何对象都比较低效，我们可以使用`spark.serialler`属性选择另外一种序列化库来优化序列化的过程(也可以使用reduce()方法为Python的pickle库自定义序列化)







