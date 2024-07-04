---
title: Spark之SparkSQL理论篇
date: 2019-05-30 19:09:55
tags: SparKSQL
categories: 大数据
---

 {{ "Spark SQL 理论学习"}}：<Excerpt in index | 首页摘要><!-- more --> 
<The rest of contents | 余下全文>

## 简介

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191139.png)

Spark SQL是Spark用来处理结构化数据的一个模块，它提供了一个编程抽象叫做DataFrame并且作为分布式SQL查询引擎的作用。

## 特点

1）易整合

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191724.png)

2) 统一的数据访问方式

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191703.png)

3）兼容Hive

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191746.png)

4）标准的数据连接

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191824.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530191859.png)

 SparkSQL可以看做是一个转换层，向下对接各种不同的结构化数据源，向上提供不同的数据访问方式。

 在SparkSQL中Spark为我们提供了两个新的抽象，分别是DataFrame和DataSet

`RDD (Spark1.0)` —>` Dataframe(Spark1.3)` —>` Dataset(Spark1.6)`

同样的数据都给到这三个数据结构有相同的结果。不同是的他们的执行效率和执行方式。在后期的Spark版本中，DataSet会逐步取代RDD和DataFrame成为唯一的API接口。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530192332.png)

## RDD

RDD是一个懒执行的不可变的可以支持Lambda表达式的并行数据集合。简单，API设计友好。但它是一个JVM驻内存对象，受GC的限制和数据增加时Java序列化成本的升高。

```scala
 val rdd = sc.textFile("file:///opt/module/spark/README.md")
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530193912.png)

```scala
rdd.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530194214.png)

## Dataframe

在Spark中DataFrame与RDD类似，也是一个分布式数据容器。但是DataFrame更像传统数据库的二维表格，除了数据以外，还记录数据的结构信息（schema），与Hive类似，DataFrame也支持嵌套数据类型``（struct、array和map）``。DataFrame API提供的是一套高层的关系操作，比函数式的RDD API要更加友好，门槛更低。与R和Pandas的DataFrame类似。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530194628.png)

RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解Person类的内部结构。右侧的DataFrame却提供了详细的结构信息，使得Spark SQL可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。DataFrame多了数据的结构信息，即schema。RDD是分布式的Java对象的集合。DataFrame是分布式的Row对象的集合。DataFrame除了提供了比RDD更丰富的算子以外，更重要的特点是提升执行效率、减少数据读取以及执行计划的优化，比如filter下推、裁剪等。

性能上比RDD要高，主要有两方面原因： 

DataFrame是为数据提供了Schema的视图。可以把它当做数据库中的一张表来对待是同时DataFrame也是懒执行的。

定制化内存管理数据以二进制的方式存在于``非堆内存``，节省了大量空间之外，还摆脱了GC的限制。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530194934.png)

优化的执行计划查询计划通过Spark catalyst optimiser进行优化. 

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530195026.png)

举个例子

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530195702.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530195713.png)

上图展示的人口数据分析的示例，构造了两个DataFrame，join之后做了filter操作。直接地执行这个执行计划，执行效率很差。join是代价较大的操作，如果能将filter下推到 join下方，先对DataFrame进行过滤，再join过滤后的较小的结果集，可以缩短执行时间。Spark SQL的查询优化器是这样做的：逻辑查询计划优化就是一个利用基于关系代数的等价变换，将高成本的操作替换为低成本操作的过程。 

得到的优化执行计划在转换成物理执行计划的过程中，还可以根据具体的数据源的特性将过滤条件下推至数据源内。最右侧的物理执行计划中Filter之所以消失不见，就是因为溶入了用于执行最终的读取操作的表扫描节点内。 

对于普通开发者而言：即便是经验并不丰富的程序员写出的次优的查询，也可以被尽量转换为高效的形式予以执行。但是由于在编译期缺少类型安全检查，导致运行时容易出错。

##   Dataset

1）是Dataframe API的一个扩展，是Spark最新的数据抽象

2）用户友好的API风格，既具有类型安全检查也具有Dataframe的查询优化特性。

3）Dataset支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。

4）样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。

5） Dataframe是Dataset的特列，DataFrame=Dataset[Row] ，所以可以通过as方法将Dataframe转换为Dataset。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息我都用Row来表示。

6）DataSet是强类型的。比如可以有Dataset[Car]，Dataset[Person].

7）DataFrame只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，比如你可以对一个String进行减法操作，在执行的时候才报错，而DataSet不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查。就跟JSON对象和类对象之间的类比。

数据准备

```json
{"name":"Hadoop"}
{"name":"Spark", "Year":2015}
{"name":"Flink", "Year":2018}
```

```scala
case class  Bigdata(name:String,Year:Int) 
val ds = spark.sqlContext.read.json("file:///opt/module/spark/json/bigdata.json")
ds.show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530203550.png)

RDD让我们能够决定怎么做，而DataFrame和DataSet让我们决定做什么,控制的粒度不一样。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530203640.png)



## 三者共性

1）都是spark平台下的分布式弹性数据集。

2）三者都有惰性机制，在进行创建、转换，如map方法时，不会立即执行，只有在遇到Action如foreach时，三者才会开始遍历运算，极端情况下，如果代码里面有创建、转换，但是后面没有在Action中使用对应的结果，在执行时会被直接跳过

3）都有惰性机制，在进行创建、转换，如map方法时，不会立即执行，只有在遇到Action如foreach时，三者才会开始遍历运算，极端情况下，如果代码里面有创建、转换，但是后面没有在Action中使用对应的结果，在执行时会被直接跳过。

```scala
val rdd=spark.sparkContext.parallelize(Seq(("a", 1), ("b", 1), ("a", 1)))
// map不运行
// map不运行
rdd.map{line=>
  println("运行")
  line._1
}.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530204902.png)+

4)都会根据spark的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出

5)都有partition的概念

6)都有partition的概念

7)对DataFrame和Dataset进行操作许多操作都需要这个包进行支持 `import spark.implicits._`

8)DataFrame和Dataset均可使用模式匹配获取各个字段的值和类型

## 三者区别

1) RDD一般和spark mlib同时使用，但是不支持sparksql操作

2) DataFrame:与RDD和Dataset不同，DataFrame每一行的类型固定为Row，只有通过解析才能获取各个字段的值，每一列的值没法直接访问

```scala
val testDF = spark.read.json("file:///opt/module/spark/json/bigdata.json")
testDF.foreach{
  line =>
    val col1=line.getAs[String]("name")
    val col2=line.getAs[Long]("Year")
}
```

3) DataFrame与Dataset一般不与spark ml同时使用

4) DataFrame与Dataset均支持sparksql的操作，比如select，groupby之类，还能注册临时表/视窗，进行sql语句操作：

```scala
val testDF = spark.read.json("file:///opt/module/spark/json/bigdata.json")
testDF.createOrReplaceTempView("tmp")
spark.sql("select * from tmp").show(100,false)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530211720.png)

5) DataFrame与Dataset支持一些特别方便的保存方式，比如保存成csv，可以带上表头，这样每一列的字段名一目了然

```scala
"","Sepal.Length","Sepal.Width","Petal.Length","Petal.Width","Species"
"1",5.1,3.5,1.4,0.2,"setosa"
"2",4.9,3,1.4,0.2,"setosa"
"3",4.7,3.2,1.3,0.2,"setosa"
"4",4.6,3.1,1.5,0.2,"setosa"
"5",5,3.6,1.4,0.2,"setosa"
"6",5.4,3.9,1.7,0.4,"setosa"
"7",4.6,3.4,1.4,0.3,"setosa"
"8",5,3.4,1.5,0.2,"setosa"
"9",4.4,2.9,1.4,0.2,"setosa"
"10",4.9,3.1,1.5,0.1,"setosa"
```

```scala
//读取
val options = Map("header" -> "true", "delimiter" -> "\t", "path" -> "file:///opt/module/spark/csv/iris.csv")
val datarDF= spark.read.options(options).format("csv").load()
datarDF.show()
//保存
val saveoptions = Map("header" -> "true", "delimiter" -> "\t", "path" -> "hdfs://datanode1:9000/test/saveToCSV")
datarDF.write.format("csv").mode(org.apache.spark.sql.SaveMode.Overwrite).options(saveoptions).save()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530214629.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530214810.png)

### 三者转换

### RDD => DataFrame

#### 手动确定

```scala
val peopleRDD = sc.textFile("/input/sparksql/people.txt")
val name2AgeRDD = peopleRDD.map{x => val para = x.split(",");(para(0).trim, para(1).trim.toInt) }
name2AgeRDD.collect
import spark.implicits._
val df = name2AgeRDD.toDF("name","age")
df.show
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190531202646.png)

#### 反射确定

利用case class

```scala
val peopleRDD = sc.textFile("/input/sparksql/people.txt")
class classPeople(name:String,age:Int)
val df = peopleRDD.map{x => val para = x.split(",");People(para(0).trim, para(1).trim.toInt) }.toDS
df.show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601095101.png)

####   编程方式

1)准备Scheam

```scala
import org.apache.spark.sql.types._
val schema = StructType( StructField("name",StringType)::StructField("age",org.apache.spark.sql.types.IntegerType)::Nil)
```

2)准备Data   【需要Row类型】

```scala
val peopleRDD = sc.textFile("/input/sparksql/people.txt")
import org.apache.spark.sql._
val data = peopleRDD.map{ x => val para = x.split(",");Row(para(0),para(1).trim.toInt)}
```

3)生成DataFrame

```scala
val dataFrame = spark.createDataFrame(data, schema)
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601101135.png)

### DataFrame => RDD

直接DataFrame.rdd即可

```scala
dataFrame.rdd
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601101408.png)

### RDD  <==>DataSet

#### RDD -》  DataSet

```scala
val peopleRDD = sc.textFile("/input/sparksql/people.txt")
case class People(name:String, age:Int)  //case class 确定schema
val ds = peopleRDD.map{x => val para = x.split(",");People(para(0), para(1).trim.toInt)}.toDS
ds.show()
```

![1559355499746](C:\Users\Schindler\AppData\Roaming\Typora\typora-user-images\1559355499746.png)

#### DataSet -》 RDD

```scala
val dsRDD = ds.rdd
dsRDD.collect
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601102125.png)

### DataFrame  <==> DataSet

#### DataSet  => DataFrame

```scala
val ds2df = ds.totoDF
ds2df.show
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601102502.png)

#### DataFrame  =>DataSet

```scala
case class People(name:String, age:Int)
val df2ds = ds2df.as[People]
df2ds.show
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601102723.png)

## 参考资料

关于SparkSQL原理深入的学习可以参考《图解Spark》













