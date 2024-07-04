---
title: Spark之RDD实战篇
date: 2019-05-27 21:38:53
tags: Spark
categories: 大数据
---

## RDD编程

在Spark中，RDD被表示为对象，通过对象上的方法调用来对RDD进行转换。经过一系列的transformations定义RDD之后，就可以调用action触发RDD的计算，action可以是向应用程序返回结果(count, collect等)，或者是向存储系统保存数据(saveAsTextFile等)。在Spark中，只有遇到action，才会执行RDD的计算(即延迟计算)，这样在运行时可以通过管道的方式传输多个转换。

要使用Spark，开发者需要编写一个Driver程序，它被提交到集群以调度运行Worker，Driver中定义了一个或多个RDD，并调用RDD上的action，Worker则执行RDD分区计算任务。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527214645.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527215048.png)

## 解析器集成

与Ruby和Python类似，Scala提供了一个交互式Shell (解析器)，借助内存数据带来的低延迟特性，可以让用户通过解析器对大数据进行交互式查询。Spark解析器将用户输入的多行命令解析为相应Java对象的示例如图所示

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527215246.png)

Scala解析器处理过程一般为：

①将用户输入的每一行编译成一个类；

②将该类载入到JVM 中；

③调用该类的某个函数。在该类中包含一个单利对象，对象中包含当前行的变量或函数，在初始化方法中包含处理该行的代码。例如，如果用户输入“varx=5”，在换行输入primln(x),那解析器会定义一个叫Linel的类，该类包含X，第二行编译成println (Linel.getlnstance().x)。

Spark中做了以下两个改变。
(1)类传输：为了让工作节点能够从各行生成的类中获取到字节码，通过HTTP传输。
(2)代码生成器的改动：通常各种代码生成的单例对象是由类的静态方法来提供的。也就是说，当序列化一个引用上一行定义变量的闭包（例如上面例子的Linel.x), Java不会通过检索对象树的方式去传输包含x的Linel实例。因此工作节点不能够得到x,在Spark中修改了代码生成器的逻辑，让各行对象的实例可以被字节应用。在图中显示了 Spark修改之后解析器是如何把用户输入的每一行变成Java对象的。

## 内存管理

Spark提供了 3种持久化RDD的存储策略：

1.未序列化Java对象存在内存中、

2.序列化的数据存于内存中

3.存储在磁盘中

第一个选项的性能是最优的，因为可以直接访问在Java虚拟机内存里的RDD对象；在空间有限的情况下，第二种方式可以让用户釆用比Java对象更有效的内存组织方式，但代价是降低了性能；第三种策略使用于RDD太大的情形，每次重新计算该RDD会带来额外的资源开销（如I/O等)。对于内存使用LRU回收算法来进行管理，当计算得到一个新的RDD分区，但没有足够空间来存储时，系统会从最近最少使用的RDD回收其一个分区的空间。除非该RDD是新分区对应的RDD，这种情况下Spark会将旧的分区继续保留在内存中，防止同一个RDD的分区被循环调入/调出。这点很关键，因为大部分的操作会在一个RDD的所有分区上进行，那么很有可能己经存在内存中的分区将再次被使用。

## 多用户管理

RDD模型将计算分解为多个相互独立的细粒度任务，这使得它在多用户集群能够支持多种资源共享算法。特别地，每个RDD应用可以在执行过程中动态调整访问资源。
在每个应用程序中，Spark运行多线程同时提交作业，并通过一种等级公平调度器来实现多个作业对集群资源的共享，这种调度器和Hadoop Fair Scheduler类似。该算法主 要用于创建基于针对相同内存数据的多用户应用，例如：Spark SQL引擎有一个服务 模式支持多用户并行查询。公平调度算法确保短的作业能够在即使长作业占满集群资源的情况下尽早完成。

Spark的公平调度也使用延迟调度，通过轮询每台机器的数据，在保持公平的情况下给予作业高的本地性。Spark支持多级本地化访问策略（本地化)，包括内存、磁盘和机 架。

由于任务相互独立，调度器还支持取消作业来为高优先级的作业腾出资源。Spark中可以使用Mesos来实现细粒度的资源共享，这使得Spark应用能相互之间或在不同的计算框架之间实现资源的动态共享。Spark使用Sparrow系统扩展支持分布式调度，该调度允许多个Spark应用以去中心化的方式在同一集群上排队工作，同时提供数据本地性、低延迟和公平性。

## RDD创建

### 集合中创建RDD

从已有的集合中创建RDD

```scala
val rdd1 = sc.parallelize(Array(1,2,3,4,5,6,7,8,9,10))
```
![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527222920.png)

```scala
//并行化操作  
def parallelize[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope { //默认是多少呢
    assertNotStopped()
    new ParallelCollectionRDD[T](this, seq, numSlices, Map[Int, Seq[String]]())
  }
//本地模式下  
override def defaultParallelism(): Int =
    scheduler.conf.getInt("spark.default.parallelism", totalCores)  
//CoarseGrainedSchedulerBackend
 override def defaultParallelism(): Int = {
    conf.getInt("spark.default.parallelism", math.max(totalCoreCount.get(), 2))
  }
//stanlone继承了CoarseGrainedSchedulerBackend 因此绝大部分的情况下并行化处理数据的并行度为CPU的核数

//makeRDD本质上还是调用了parallelize
  def makeRDD[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope {
    parallelize(seq, numSlices)
  }
```

```scala
/**
 * Distribute a local Scala collection to form an RDD, with one or more
 * location preferences (hostnames of Spark nodes) for each object.
 * Create a new partition for each collection item.
 */
//
def makeRDD[T: ClassTag](seq: Seq[(T, Seq[String])]): RDD[T] = withScope {
  assertNotStopped()
  val indexToPrefs = seq.zipWithIndex.map(t => (t._2, t._1._2)).toMap
  new ParallelCollectionRDD[T](this, seq.map(_._1), math.max(seq.size, 1), indexToPrefs)
}
```

```scala
val test1 = sc.parallelize(List(1,2,3,4))
val seq = List((1,List("datanode1")),(2,List("datanode2")))  //可以提供位置信息
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527230912.png)

`def parallelize[T: ClassTag]` 和`def makeRDD[T: ClassTag]`返回的都是ParallelCollectionRDD,而且这个makeRDD的实现不可以自己指定分区的数量，而是固定为seq参数的size大小。

### 外部存储系统的数据集创建

包括本地的文件系统，还有所有Hadoop支持的数据集，比如HDFS、Cassandra、HBase等

```scala
val datasets =sc.textFile("hdfs://datanode1:9000/input/test.txt")
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527232248.png)

## RDD转换

### map()

map操作时对RDD中的每一个素都执行一个指定的函数来产生一个新的RDD,任何元RDD中的元素在新RDD中都有且只有一个元素与之对应。





```scala
val data = sc.parallelize(1 to 10).collect()
val map = data.map(_ * 2)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190527232843.png)

### mapPartitions()

类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是`Iterator[T] => Iterator[U]`。假设有N个元素，有M个分区，那么map的函数的将被调用N次,而mapPartitions被调用M次,一个函数一次处理所有分区。其中`preservesPartitioning`表示是否保留父RDD的partitiones分区信息，如果在映射过程中需要频繁创建对象，使用mapPartitions操作要比map操作高 效得多。比如，将RDD中的所有数据通过JDBC连接写入数据库，如果使用map函数，可能要为每一个元素都创建一个connection,这样开销很大。如果使用mapPartitions，那么只需要针对每一个分区建立一个connectiono mapPartitionsWithlndex操作作用类似于mapPartitions,只是输入参数多了一个分区索引。

```scala
 def mapPartitions[U: ClassTag](
      f: Iterator[T] => Iterator[U],
      preservesPartitioning: Boolean = false): RDD[U] = withScope {
    val cleanedF = sc.clean(f)
    new MapPartitionsRDD(
      this,
      (context: TaskContext, index: Int, iter: Iterator[T]) => cleanedF(iter),
      preservesPartitioning)
  }
```

```scala
//创建RDD使RDD有两个分区
 var rdd1= sc.makeRDD(1 to 10,2)
///使用mapPartitions对rddl进行重新分区
  var rdd2 = rdd1.mapPartitions{ x => {
      var result = List[Int]()
      var i = 0
      while(x.hasNext){
          i += x.next()
    }
     result.::(i).iterator
   }}
  //rdd2将rddl中每个分区中的数值累加
  rdd2.collect

//重新对rdd1分区
var rdd3 = rdd1.mapPartitionsWithIndex{
     (x,iter) => {
       var result = List[String]()
       var i = 0
       while(iter.hasNext){
          i += iter.next()
         }
        result.::(x + "|" + i).iterator
   }
}
rdd3.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190528083325.png)

### glom()

RDD中每一个分区所有类型为T的数据转变成元素类型为T的数组[Array[T]].

```scala
var rdd = sc.parallelize(1 to 16,4)
rdd.glom().collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190528083705.png)

### flatMap()

flatMap操作原RDD中的每一个元素生成一个或多个元素来构建新的RDD

```scala
val rdd1 = sc.parallelize(1 to 5)
val flatMap = rdd1.flatMap(1 to _)
flatMap.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528084629.png)

### filter()

返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成.

```scala
var sourceFilter = sc.parallelize(Array("hadoop","spark","flink","hphblog"))
val filter = sourceFilter.filter(_.contains("h"))
filter.collect
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528084943.png)

### mapPartitionsWithIndex()

类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是`(Int, Interator[T]) =>Iterator[U]`

```scala
//创建RDD使RDD有两个分区
 var rdd1= sc.makeRDD(1 to 10,2)
///使用mapPartitions对rddl进行重新分区
  var rdd2 = rdd1.mapPartitions{ x => {
      var result = List[Int]()
      var i = 0
      while(x.hasNext){
          i += x.next()
    }
     result.::(i).iterator
   }}
  //rdd2将rddl中每个分区中的数值累加
  rdd2.collect

//重新对rdd1分区
var rdd3 = rdd1.mapPartitionsWithIndex{
     (x,iter) => {
       var result = List[String]()
       var i = 0
       while(iter.hasNext){
          i += iter.next()
         }
        result.::(x + "|" + i).iterator
   }
}
rdd3.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/20190528083325.png)

###  sample(withReplacement, fraction, seed)

以指定的随机种子随机抽样出数量为fraction的数据，withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样，seed用于指定随机数生成器种子。例子从RDD中随机且有放回的抽出50%的数据，随机种子值为2（即可能以1 2 3的其中一个起始值）

```scala
val rdd = sc.parallelize(1 to 10)
rdd.collect
var sample1 = rdd.sample(true,0.5,2)
sample1.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528091220.png)

### distinct([numTasks]))

对源RDD进行去重后返回一个新的RDD. 默认情况下，只有8个并行任务来操作，但是可以传入一个可选的numTasks参数改变它。

```scala
 val rdd = sc.parallelize(List(1,2,2,3,3,4,4,5,5,5,6,6,7,7,8))
 val rdd1 = rdd.distinct()
 rdd1.collect

 val rdd3 = rdd1.distinct(10)
 rdd3.collect
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528093127.png)

### partitionBy

对RDD进行分区操作，如果原有的partionRDD和现有的partionRDD是一致的话就不进行分区， 否则会生成ShuffleRDD。

```scala
val rdd = sc.parallelize(Array((1,"hadoop"),(2,"spark"),(3,"flink"),(4,"hphblog")),4)
rdd.partitions.size
var rdd2 = rdd.partitionBy(new org.apache.spark.HashPartitioner(2))
rdd.collect
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528093534.png)

###  coalesce((numPartitions, shuffle)

缩减分区数，用于大数据集过滤后，提高小数据集的执行效率。shuffle默认关闭.

```scala
val rdd = sc.parallelize(1 to 10000,4)
val coalesceRDD = rdd.coalesce(2)
val shuffleRDD = rdd.coalesce(2,true)
shuffleRDD.collect
rdd.collect
coalesceRDD.partitions.size
shuffleRDD.partitions.size
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528094743.png)

### repartition(numPartitions)

根据分区数，从新通过网络随机洗牌所有数据。底层调用的是`coalesce(numPartitions, shuffle = true)`

```scala
val rdd = sc.parallelize(1 to 10000,4)
rdd.partitions.size
val rerdd = rdd.repartition(2)
rerdd.partitions.size
val rerdd = rdd.repartition(4)
rerdd.partitions.size
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528100745.png)

### repartitionAndSortWithinPartitions

repartitionAndSortWithinPartitions函数是repartition函数的变种，与repartition函数不同的是，repartitionAndSortWithinPartitions在给定的partitioner内部进行排序，性能比repartition要高。 

```scala
def repartitionAndSortWithinPartitions(partitioner: Partitioner): RDD[(K, V)] = self.withScope {
  new ShuffledRDD[K, V, V](self, partitioner).setKeyOrdering(ordering)
}
```

###  sortBy([ascending], [numTasks])

```scala
def sortBy[K](
    f: (T) => K,
    ascending: Boolean = true,
    numPartitions: Int = this.partitions.length)
```

```scala
val rdd =sc.parallelize(List(1,2,3,4,5,6,7,8,9))
rdd.sortBy(x => x ,ascending=false).collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528102152.png)

###  union(otherDataset)

对源RDD和参数RDD求并集后返回一个新的RDD `不去重`

```scala
val rdd1 = sc.parallelize(1 to 10)
val rdd2 = sc.parallelize(5 to 15)
val rdd3 = rdd1.union(rdd2)
rdd3.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528102902.png)

### subtract (otherDataset)

计算差的一种函数，去除两个RDD中相同的元素，不同的RDD将保留下来 

```scala
 val rdd1 = sc.parallelize(1 to 10)
 val rdd2 = sc.parallelize(5 to 15)
 val rdd3 = rdd1.subtract(rdd2)
 rdd3.collect
 val rdd4 =rdd2.subtract(rdd1)
 rdd4.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/1559010918532.png)

### intersection(otherDataset)

对源RDD和参数RDD求交集后返回一个新的RDD

```scala
val rdd1 = sc.parallelize(1 to 10)
val rdd2 = sc.parallelize(5 to 15)
val rdd3 = rdd1.intersection(rdd2)
rdd3.collect
val rdd4 = rdd2.intersection(rdd1)
rdd4.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528103820.png)

###  cartesian(otherDataset)

笛卡尔积

```scala
val rdd1 = sc.parallelize(1 to 3)
val rdd2 = sc.parallelize(2 to 5)
rdd1.cartesian(rdd2).collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528104041.png)

###  pipe(command, [envVars])

管道，对于每个分区，都执行一个perl或者shell脚本，返回输出的RDD

```shell
#!/bin/sh
echo "hello Spark This is Linux bash"
while read LINE; do
   echo ">>>"${LINE}
done
```

```scala
val rdd = sc.parallelize(List("hi","Hello","hadoop","spark","flink","hphblog"),1)
rdd.pipe("/home/hadoop/pipe.sh").collect()
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528104554.png)

###  join(otherDataset, [numTasks])

在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD

```scala
val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd.join(rdd1).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528104857.png)

###  cogroup(otherDataset,[numTasks])

在类型为(K,V)和(K,W)的RDD上调用，返回一个``(K,(Iterable<V>,Iterable<W>))``类型的RDD

```scala
val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd.cogroup(rdd1).collect()
val rdd2 = sc.parallelize(Array((4,4),(2,5),(3,6)))
rdd.cogroup(rdd2).collect()
val rdd3 = sc.parallelize(Array((1,"a"),(1,"d"),(2,"b"),(3,"c")))
rdd3.cogroup(rdd2).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528110138.png)

### reduceByKey(func, [numTasks])

在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，reduce任务的个数可以通过第二个可选的参数来设置。

```scala
val rdd = sc.parallelize(List(("hadoop",1),("spark",5),("spark",5),("flink",3)))
val reduce = rdd.reduceByKey((x,y)=>(x+y))
reduce.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528110533.png)

### groupByKey

groupByKey也是对每个key进行操作，但只生成一个sequence

```scala
val words = Array("hadoop", "spark", "spark", "flink", "flink", "flink")
val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))
val group = wordPairsRDD.groupByKey()
group.collect()
val result = group.map(t => (t._1, t._2.sum))
result.collect
val map = group.map(t => (t._1, t._2.sum))
map.collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528111123.png)

###  combineByKey[C]

(  createCombiner: V => C,  mergeValue: (C, V) => C,  mergeCombiners: (C, C) => C) 对相同K，把V合并成一个集合。

createCombiner: combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就 和之前的某个元素的键相同。如果这是一个新的元素,combineByKey() 会使用一个叫作 createCombiner() 的函数来创建 
 那个键对应的累加器的初始值

mergeValue: 如果这是一个在处理当前分区之前已经遇到的键， 它会使用 mergeValue() 方法将该键的累加器对应的当前值与这个新的值进行合并

mergeCombiners: 由于每个分区都是独立处理的， 因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器， 就需要使用用户提供的 mergeCombiners() 方法将各个分区的结果进行合并。

```scala
val scores = Array(("Fred", 88), ("Fred", 95), ("Fred", 91), ("Wilma", 93), ("Wilma", 95), ("Wilma", 98))
val input = sc.parallelize(scores)
val combine = input.combineByKey(
          (v)=>(v,1),
          (acc:(Int,Int),v)=>(acc._1+v,acc._2+1),
          (acc1:(Int,Int),acc2:(Int,Int))=>(acc1._1+acc2._1,acc1._2+acc2._2)
)

val result = combine.map{
         case (key,value) => (key,value._1/value._2.toDouble)
}
result.collect()
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528111935.png)

### aggregateByKey

`(zeroValue:U,[partitioner: Partitioner]) (seqOp:(U, V) => U,combOp: (U, U) => U) `

在kv对的RDD中，，按key将value进行分组合并，合并时，将每个value和初始值作为seq函数的参数，进行计算，返回的结果作为一个新的kv对，然后再将结果按照key进行合并，最后将每个分组的value传递给combine函数进行计算（先将前两个value进行计算，将返回结果和下一个value传给combine函数，以此类推），将key与计算结果作为一个新的kv对输出。

seqOp函数用于在每一个分区中用初始值逐步迭代value，combOp函数用于合并每个分区中的结果。

```scala
val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)
val agg = rdd.aggregateByKey(0)(math.max(_,_),_+_)
agg.collect()
agg.partitions.size
val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),1)
val agg = rdd.aggregateByKey(0)(math.max(_,_),_+_).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528114417.png)

### foldByKey

`(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]`aggregateByKey的简化操作，seqop和combop相同

```scala
val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)
val agg = rdd.foldByKey(0)(_+_)
agg.collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528115239.png)

### sortByKey([ascending], [numTasks])

在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD

```scala
val rdd = sc.parallelize(Array((3,"hadoop"),(6,"hohblog"),(2,"flink"),(1,"spark")))
rdd.sortByKey(true).collect()
rdd.sortByKey(false).collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/1559015691057.png)

### mapValues

针对于(K,V)形式的类型只对V进行操作 

```scala
val rdd = sc.parallelize(Array((3,"hadoop"),(6,"hohblog"),(2,"flink"),(1,"spark")))
rdd.mapValues(_+"==> www.hphblog.cn").collect()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528115636.png)

## RDD行动算子

### reduce(func)

通过func函数聚集RDD中的所有元素，这个功能必须是可交换且可并联的

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.reduce(_+_)
val rdd1 = sc.makeRDD(Array(("a",1),("a",3),("c",3),("d",5)))
rdd1.reduce((x,y)=>(x._1 + y._1,x._2 + y._2))
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528120250.png)

### collect()

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.collect()
```

在驱动程序中，以数组的形式返回数据集的所有元素

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528120451.png)

###  count()

返回RDD的元素个数

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd. count()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528120547.png)

### first()

返回RDD的第一个元素（类似于take(1)）

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.first()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528120819.png)

###  take(n)

返回一个由数据集的前n个元素组成的数组

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.take(10)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528120952.png)

### takeSample(withReplacement,num, [seed])

返回一个数组，该数组由从数据集中随机采样的num个元素组成，可以选择是否用随机数替换不足的部分，seed用于指定随机数生成器种子

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.takeSample(true,10,2)
rdd.takeSample(false,10,2)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528121504.png)

### takeOrdered(n)

返回前几个的排序

```scala
val rdd = sc.makeRDD(1 to 100,2)
rdd.take(10)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528121702.png)

### aggregate

`(zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)`aggregate函数将每个分区里面的元素通过seqOp和初始值进行聚合，然后用combine函数将每个分区的结果和初始值(zeroValue)进行combine操作。这个函数最终返回的类型不需要和RDD中元素类型一致。

```scala
val rdd = sc.makeRDD(1 to 10,2)
rdd.aggregate(1)(
     {(x : Int,y : Int) => x + y}, 
      {(a : Int,b : Int) => a + b}
      )
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528183859.png)

为什么是58呢:

```scala
rdd.mapPartitionsWithIndex{
    (partid,iter)=>{
        var part_map = scala.collection.mutable.Map[String,List[Int]]()
        var part_name = "part_" + partid
        part_map(part_name) = List[Int]()
        while(iter.hasNext){
            part_map(part_name) :+= iter.next()//:+= 列表尾部追加元素
        }
        part_map.iterator
    }
}.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528185402.png)

遍历第一个分区的数据我们知道第一个分区的数据是(1,2,3,4,5),第二个分区的数据是(6,7,8,9,10)首先在每一个分区执行`(x : Int,y : Int) => x + y`我们传入的zeroValue的值为1,即在`part_0中zeroValue+5+4+3+2+1=19`,在`part_1中zeroValue+6+7+8+9+10=41`,在将连个分局的结果合并`(a : Int,b : Int) => a + b`,并且使用zeroValue的值1即`zeroValue+part_0+part_1=1+16+41=58`因此结果为58.

```scala
rdd.aggregate(1)(
     {(x : Int,y : Int) => x * y},
      {(a : Int,b : Int) => a + b}
      )
```

相同的我们可以刻分析出来

首先在每一个分区执行`(x : Int,y : Int) => x * y`我们传入的zeroValue的值为1,即在`part_0中zeroValue*5*4*3*2*1=120`,在`part_1中zeroValue*6*7*8*9*10=30240`,在将连个分局的结果合并`(a : Int,b : Int) => a + b`,并且使用zeroValue的值1即`zeroValue+part_0+part_1=1+120+30240=30361`因此结果为30361.

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528192803.png)

### fold(num)(func)

折叠操作，aggregate的简化操作，seqop和combop一样。

```scala
val rdd = sc.makeRDD(1 to 10,2)
rdd.aggregate(1)(
     {(x : Int,y : Int) => x + y},
      {(a : Int,b : Int) => a + b}
      )
rdd.fold(1)(_+_)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528201005.png)

### saveAsTextFile(path)

将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本

```scala
 val rdd = sc.makeRDD(1 to 10,2)
 rdd.saveAsTextFile("hdfs://datanode1:9000/spark/saveAsTextFile/")
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528201529.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528201656.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528201735.png)

### saveAsSequenceFile(path) 

将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。

### saveAsObjectFile(path) 

用于将RDD中的元素序列化成对象，存储到文件中。

###  countByKey()

针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。

```scala
val rdd = sc.parallelize(List(("hadoop",3),("spark",2),("hphblog",3),("flink",9),("flink",9),("spark",10)),3)
rdd.countByKey()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528202419.png)

### foreach(func)

在数据集的每一个元素上，运行函数func进行更新。注意foreach遍历RDD,将函数f应用于每一个元素.要注意如果对RDD执行foreach,智慧在Executor端有效,而不是Driver.比如rdd.collect().foreach(println),只会在Executor端有效,Driver端是看不到的.

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528202856.png)

### sortBy(funct)

```scala
var rdd = sc.makeRDD(Array(("A",2),("D",5), ("A",1), ("B",6), ("B",3), ("E", 7),("C",4)))
rdd.sortBy(x => x).collect
rdd.sortBy(x => x._2,false).collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528204821.png)

##  RDD持久化

Spark速度非常快的原因之一，就是在不同操作中可以在内存中持久化或缓存个数据集。当持久化某个RDD后，每一个节点都将把计算的分片结果保存在内存中，并在对此RDD或衍生出的RDD进行的其他动作中重用。这使得后续的动作变得更加迅速。RDD相关的持久化和缓存，是Spark最重要的特征之一。可以说，缓存是Spark构建迭代式算法和快速交互式查询的关键。如果一个有持久化数据的节点发生故障，Spark 会在需要用到缓存的数据时重算丢失的数据分区。如果 希望节点故障的情况不会拖累我们的执行速度，也可以把数据备份到多个节点上。

### 缓存方式

RDD通过persist方法或cache方法可以将前面的计算结果缓存，默认情况下 persist() 会把数据以序列化的形式缓存在 JVM 的堆空 间中。 

但是并不是这两个方法被调用时立即缓存，而是触发后面的action时，该RDD将会被缓存在计算节点的内存中，并供后面重用。

```scala
/**
 * Persist this RDD with the default storage level (`MEMORY_ONLY`).
 */
def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)  //默认的持久化是内存中

/**
 * Persist this RDD with the default storage level (`MEMORY_ONLY`).
 */
def cache(): this.type = persist()   //cache最终也是调用了persist方法
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528224007.png)

在存储级别的末尾加上“_2”来把持久化数据存为两份

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528224100.png)

缓存有可能丢失，或者存储存储于内存的数据由于内存不足而被删除，RDD的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于RDD的一系列转换，丢失的数据会被重算，由于RDD的各个Partition是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部Partition。

```scala
val rdd = sc.makeRDD(1 to 10)
val nocache = rdd.map(_.toString+"["+System.currentTimeMillis+"]")
val cache =  rdd.map(_.toString+"["+System.currentTimeMillis+"]")
cache.cache
nocache.collect
nocache.collect
cache.collect
cache.collect
cache.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528224651.png)

我们发现持久化的内存时间戳没有变化,未持久化的内存时间戳是有变化的

##  RDD检查点机制

Spark中对于数据的保存除了持久化操作之外，还提供了一种检查点的机制，检查点（本质是通过将RDD写入Disk做检查点）是为了通过lineage做容错的辅助，lineage过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做Lineage，就会减少开销。检查点通过将数据写入到HDFS文件系统实现了RDD的检查点功能。

cache 和 checkpoint 是有显著区别的，  缓存把 RDD 计算出来然后放在内存中，但是RDD 的依赖链（相当于数据库中的redo 日志）， 也不能丢掉， 当某个点某个 executor 宕了，上面cache 的RDD就会丢掉， 需要通过依赖链重放计算出来， 不同的是， checkpoint是把 RDD 保存在 HDFS中， 是多副本可靠存储，所以依赖链就可以丢掉了，就斩断了依赖链， 是通过复制实现的高容错。

如果存在以下场景，则比较适合使用检查点机制：

1）DAG中的Lineage过长，如果重算，则开销太大（如在PageRank中）。

2）在宽依赖上做Checkpoint获得的收益更大。

为当前RDD设置检查点。该函数将会创建一个二进制的文件，并存储到checkpoint目录中，该目录是用SparkContext.setCheckpointDir()设置的。在checkpoint的过程中，该RDD的所有依赖于父RDD中的信息将全部被移出。对RDD进行checkpoint操作并不会马上被执行，必须执行Action操作才能触发。

```scala
val data = sc.parallelize(1 to 1000 , 5)
sc.setCheckpointDir("hdfs://datanode1:9000/checkpoint")
data.checkpoint
data.count
val ch1 = sc.parallelize(1 to 20)
val ch2 = ch1.map(_.toString+"["+System.currentTimeMillis+"]")
val ch3 = ch1.map(_.toString+"["+System.currentTimeMillis+"]")
ch3.checkpoint
ch2.collect
ch2.collect
ch2.collect
ch3.collect
ch3.collect
ch3.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190529115207.png)