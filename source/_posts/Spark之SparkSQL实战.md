---
title: Spark之SparkSQL实战
date: 2019-05-30 22:09:06
tags: SparKSQL
categories: 大数据
---

 {{ "DataFrames 基本操作和 DSL SQL风格 UDF函数 以及数据源"}}：<Excerpt in index | 首页摘要><!-- more --> 
<The rest of contents | 余下全文>

## SparkSQL查询

Json数据准备

```json
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

```scala
val df =spark.read.json("/input/sparksql/json/people.json")
df.show()
df.filter($"age">21).show();
df.createOrReplaceTempView("person")
spark.sql("SELECT * FROM person").show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190530221923.png)

## IDEA创建SparkSQL程序

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.1.1</version>
    <scope>provided</scope>
</dependency>
```

```scala
package com.hph.sql

import org.apache.spark.sql.SparkSession


object HelloWorld {

  def main(args: Array[String]) {
    //创建SparkConf()并设置App名称

    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.some.config.option", "some-value")
      .master("local[*]")
      .getOrCreate()

    // For implicit conversions like converting RDDs to DataFrames
    import spark.implicits._

    val df = spark.read.json("F:\\spark\\examples\\src\\main\\resources\\people.json")

    // Displays the content of the DataFrame to stdout
    df.show()

    df.filter($"age" > 21).show()

    df.createOrReplaceTempView("persons")

    spark.sql("SELECT * FROM persons where age > 21").show()

    spark.stop()
  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190531121301.png)

## SparkSession

老的版本中，SparkSQL提供两种SQL查询起始点，一个叫SQLContext，用于Spark自己提供的SQL查询，一个叫HiveContext，用于连接Hive的查询，SparkSession是Spark最新的SQL查询起始点，实质上是SQLContext和HiveContext的组合，所以在SQLContext和HiveContext上可用的API在SparkSession上同样是可以使用的。SparkSession内部封装了sparkContext，所以计算实际上是由sparkContext完成的。

```scala
import org.apache.spark.sql.SparkSession


object HelloWorld {

  def main(args: Array[String]) {
    //创建SparkConf()并设置App名称

    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.some.config.option", "some-value")
      .master("local[*]")
      .getOrCreate()

    // 用于将DataFrames隐式转换成RDD，使df能够使用RDD中的方法。
    import spark.implicits._

    val df = spark.read.json("hdfs://datanode1:9000/input/sparksql/people.json")

    // Displays the content of the DataFrame to stdout
    df.show()

    df.filter($"age" > 21).show()

    df.createOrReplaceTempView("persons")

    spark.sql("SELECT * FROM persons where age > 21").show()

    spark.stop()
  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601104448.png)



## 创建DataFrames

SparkSession是创建DataFrames和执行SQL的入口，创建DataFrames有三种方式，一种是可以从一个存在的RDD进行转换，还可以从Hive Table进行查询返回，或者通过Spark的数据源进行创建。

```scala
val df = spark.read.json("/input/sparksql/people.json")	
df.show()
val peopleRdd = sc.textFile("/input/sparksql/people.txt")
val peopleDF = peopleRdd.map(_.split(",")).map(paras => (paras(0),paras(1).trim().toInt)).toDF("name","age")
peopleDF.show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601123919.png)

### 常用操作

###  DSL风格语法

```scala
df.printSchema() 
df.select("name").show()
df.select($"name", $"age" + 1).show()  
df.filter($"age" > 21).show()
df.groupBy("age").count().show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601124441.png)

###  SQL风格语法

```scala
df.createOrReplaceTempView("people")
val sqlDF = spark.sql("SELECT * FROM people")
sqlDF.show()
df.createGlobalTempView("people")
spark.sql("SELECT * FROM global_temp.people").show()
spark.newSession().sql("SELECT * FROM global_temp.people").show()
```

临时表是Session范围内的，Session退出后，表就失效了。如果想应用范围内有效，可以使用全局表。注意使用全局表时需要全路径访问，如：global_temp.people

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601124700.png)

## 创建DataSet

注意: Case classes in Scala 2.10只支持 22 字段. 你可以使用自定义的 classes 来实现对字段的映射
`case class Person(name: String, age: Long)`

```scala
case class Person(name: String, age: Long)
val caseClassDS = Seq(Person("Andy", 32)).toDS()
caseClassDS.show()
import spark.implicits._ 
val primitiveDS = Seq(1, 2, 3).toDS() 
primitiveDS.map(_ + 1).collect()

val path = "/input/sparksql/people.json"
val peopleDS = spark.read.json(path).as[Person]
peopleDS.show()
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601151413.png)

## 相互转化

具体的转换可以参考: [三者共性](https://www.hphblog.cn/2019/05/30/Spark之SparkSQL/#三者共性)

## UDF函数

```scala
import org.apache.spark.sql.SparkSession

object UDF {
  def main(args: Array[String]): Unit = {
    //创建SparkConf()并设置App名称
    val spark = SparkSession.builder()
      .appName("Spark SQL UDF example")
      .master("local[*]")
      .getOrCreate()
    //读取数据
    val df = spark.read.json("hdfs://datanode1:9000/input/sparksql/people.json")
    df.show()

    //注册UDF函数
    spark.udf.register("AddOne", (age: Int) => age + 1)

    df.createOrReplaceTempView("people")
    //SQL语句
    spark.sql("Select name,AddOne(age), age from people").show()

    spark.stop()

  }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601162725.png)

## 自定义聚合函数

强类型的Dataset和弱类型的DataFrame都提供了相关的聚合函数， 如 count()，countDistinct()，avg()，max()，min()。除此之外，用户可以设定自己的自定义聚合函数。

弱类型用户自定义聚合函数：通过继承`UserDefinedAggregateFunction`来实现用户自定义聚

```scala
package com.hph.sql

import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._

//自定义UDAF函数需要继承UserDefinedAggregateFunction
class AverageSal extends UserDefinedAggregateFunction {
  //输入数据
  override def inputSchema: StructType = StructType(StructField("salary", LongType) :: Nil)

  //定义每一个分区中的共享变量
  override def bufferSchema: StructType = StructType(StructField("sum", LongType) :: StructField("count", IntegerType) :: Nil)

  //表示UDAF函数的输出类型
  override def dataType: DataType = DoubleType

  //如果有相同的输入是否会存在相同的输出,如果是则为True
  override def deterministic: Boolean = true

  //初始化 每一个分区中的共享变量
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = 0L; //  sum
    buffer(1) = 0 //   sum
  }

  //每一个分区中的每一条数据聚合的时候需要调用该方法
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    //获取这一行中的工资,然后将工资加入到sum里
    buffer(0) = buffer.getLong(0) + input.getLong(0)
    //需要将工资的个数加1
    buffer(1) = buffer.getInt(1) + 1
  }

  //将每一个没去的输出合并形成最后的数据
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    //合并总的工资
    buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
    //合并总的工资个数
    buffer1(1) = buffer1.getInt(1) + buffer2.getInt(1)
  }

  //给出计算结果
  override def evaluate(buffer: Row): Any = {
    //取出总的工资  / 总的工资个数
    buffer.getLong(0).toDouble / buffer.getInt(1)
  }
}

object AverageSal {
  def main(args: Array[String]): Unit = {

    //创建SparkConf()并设置App名称
    val spark = SparkSession.builder()
      .appName("Spark SQL UDF example")
      .master("local[*]")
      .getOrCreate()

    val employee = spark.read.json("hdfs://datanode1:9000/input/sparksql/employees.json")
    employee.createOrReplaceTempView("employee")
      
    spark.udf.register("average", new AverageSal)
    spark.sql("select average(salary)  from employee").show()
    spark.stop()
    
  }
}

```

### 强类型函数

```scala
package com.hph.sql

import org.apache.spark.sql.{Encoder, Encoders, SparkSession}
import org.apache.spark.sql.expressions.Aggregator


case class Employee(name: String, salary: Long)

case class Aver(var sum: Long, var count: Int)

class Average extends Aggregator[Employee, Aver, Double] {
  //初始化方法:初始化每一个分区的共享变量
  override def zero: Aver = Aver(0L, 0)

  //每一个分区的每一条数据聚合的时候需要回调该方法
  override def reduce(b: Aver, a: Employee): Aver = {
    b.sum = b.sum + a.salary
    b.count = b.count + 1
    b
  }

  //将每一个分区的输出 合并 形成最后的数据
  override def merge(b1: Aver, b2: Aver): Aver = {
    b1.sum = b1.sum + b2.sum
    b1.count = b1.count + b2.count
    b1
  }

  //输出计算结果
  override def finish(reduction: Aver): Double = {
    reduction.sum.toDouble / reduction.count
  }

  override def bufferEncoder: Encoder[Aver] = Encoders.product

  override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
}

object Average {
  def main(args: Array[String]): Unit = {
    //创建SparkConf()并设置App名称
    val spark = SparkSession.builder()
      .appName("Spark SQL Strong Type UDF example")
      .master("local[*]")
      .getOrCreate()
    import spark.implicits._
    val employee = spark.read.json("hdfs://datanode1:9000/input/sparksql/employees.json").as[Employee]
    val aver = new Average().toColumn.name("average")
    employee.select(aver).show()

    spark.close()
  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601200128.png)

































