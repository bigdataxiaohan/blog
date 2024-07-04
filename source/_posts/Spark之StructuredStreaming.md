---
title: Spark之StructuredStreaming
date: 2019-06-07 22:57:46
tags:
 - Spark
 - Structured Streaming
categories: 大数据
---

## 简介

 Structured Streaming是Spark2.0版本提出的新的实时流框架，是一种基于Spark SQL引擎的可扩展且容错的流处理引擎。在内部，默认情况下，结构化流式查询使用微批处理引擎进行处理，该引擎将数据流作为一系列小批量作业处理，从而实现低至100毫秒的端到端延迟和完全一次的容错保证。自Spark 2.3以来，引入了一种称为连续处理的新型低延迟处理模式，它可以实现低至1毫秒的端到端延迟，并且具有至少一次保证。

相比于Spark Streaming，优点如下：

支持多种数据源的输入和输出
以结构化的方式操作流式数据，能够像使用Spark SQL处理离线的批处理一样，处理流数据，代码更简洁，写法更简单
基于Event-Time，相比于Spark Streaming的Processing-Time更精确，更符合业务场景
解决了Spark Streaming存在的代码升级，DAG图变化引起的任务失败，无法断点续传的问题。

## WordCount

```scala
import org.apache.spark.sql.SparkSession

/**
  * Created by 清风笑丶 Cotter on 2019/6/7.
  */
object WordCount {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("StructuredNetworkWordCount")
      .master("local[*]")
      .getOrCreate()

    val lines = spark.readStream
      .format("socket")
      .option("host", "datanode1")
      .option("port", 9999)
      .load()

    import spark.implicits._
    val words = lines.as[String].flatMap(_.split(" "))
    val wordCounts = words.groupBy("value").count()

    val query = wordCounts.writeStream
      .outputMode("complete")
      .format("console")
      .start()

    query.awaitTermination()

  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/StructuredStreaming.gif)

## 编程模型

结构化流的关键思想是将活生生的数据流看作一张正在被连续追加数据的表。产生了一个与批处理模型非常相似的新的流处理模型。可以像在静态表之上的标准批处理查询一样，Spark是使用在一张无界的输入表之上的增量式查询来执行流计算的。

![Stream as a Table](http://spark.apache.org/docs/latest/img/structured-streaming-stream-as-a-table.png)

数据流Data Stream看成了表的行数据，连续地往表中追加。结构化流查询将会产生一张结果表（Result Table）:

![Model](http://spark.apache.org/docs/latest/img/structured-streaming-model.png)

第一行是Time，每秒有个触发器，第二行是输入流，对输入流执行查询后产生的结果最终会被更新到第三行的结果表中。第四行驶输出，图中显示的输出模式是完全模式（Complete Mode）。图中显示的是无论结果表何时得到更新，我们将希望将改变的结果行写入到外部存储。输出有三种不同的模式：

（1）完全模式（Complete Mode）

整个更新的结果表（Result Table）将被写入到外部存储。这取决于外部连接决定如何操作整个表的写入。

（2）追加模式（Append Mode）

只有从上一次触发后追加到结果表中新行会被写入到外部存储。适用于已经存在结果表中的行不期望被改变的查询。

（3）更新模式（Update Mode）

只有从上一次触发后在结果表中更新的行将会写入外部存储（Spark 2.1.1之后才可用）。这种模式不同于之前的完全模式，它仅仅输出上一次触发后改变的行。如果查询中不包含聚合，这种模式与追加模式等价的。每种模式适用于特定类型的查询。下面以单词计数的例子说明三种模式的区别（单词计数中使用了聚合）

![Model](http://spark.apache.org/docs/latest/img/structured-streaming-example-model.png)

### Event-time Late Data

 Event-time是嵌入到数据本身的基于事件的时间。对于许多的应用来说，你可能希望操作这个事件-时间。例如，如果你想获得每分钟物联网设备产生的事件数量，然后想使用数据产生时的时间（也就是数据的event-time），而不是Spark接收他们的时间。每个设备中的事件是表中的一行，而事件-时间是行中的一个列值。这就允许将基于窗口的聚合（比如每分钟的事件数）看成是事件-时间列的分组和聚合的特殊类型——每个时间窗口是一个组，每行可以属于多个窗口/组。

 进一步，这个模型自然处理那些比期望延迟到达的事件-时间数据。当Spark正在更新结果表时，当有延迟数据，它就会完全控制更新旧的聚合，而且清理旧的聚合去限制中间状态数据的大小。从Spark 2.1开始，我们已经开始支持水印（watermarking ），它允许用户确定延迟的阈值，允许引擎相应地删除旧的状态。

### 窗口操作

在滑动的事件-时间窗口上的聚合对于结构化流是简单的，非常类似于分组聚合。在分组聚合中，聚合的值对用户确定分组的列保持唯一的。在基于窗口的聚合中，聚合的值对每个窗口的事件-时间保持唯一的。

 修改我们前面的单词计数的例子，现在当产生一行句子时，附件一个时间戳。我们想每5分钟统计一次10分钟内的单词数。例如，12:00 - 12:10, 12:05 - 12:15, 12:10 - 12:20等。注意到12:00 - 12:10是一个窗口，表示数据12:00之后12:10之前到达。比如12:07到达的单词，这个单词应该在12:00 - 12:10和12:05 - 12:15两个窗口中都要被统计。如图：

![img](http://spark.apache.org/docs/latest/img/structured-streaming-window.png)

```scala
import java.sql.Timestamp

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._

/**
  * Created by 清风笑丶 Cotter on 2019/6/7.
  */
object WindowOnEventTime {

  case class TimeWord(word: String, timestamp: Timestamp)

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("Structrued-Streaming")
      .master("local[*]")
      .getOrCreate()
      
    val lines = spark.readStream
      .format("socket")
      .option("host", "datanode1")
      .option("port", 9999)
      .option("includeTimestamp", true) //添加时间戳
      .load()
    import spark.implicits._

    val words = lines.as[(String, Timestamp)]
      .flatMap(line => line._1.split(" ")
        .map(word => TimeWord(word, line._2))).toDF()

    // 计数
    val windowedCounts = words.groupBy(
      window($"timestamp","10 seconds" ,"5 seconds"), $"word"
    ).count().orderBy("window")

    // 查询
    val query = windowedCounts.writeStream
      .outputMode("complete")
      .format("console")
      .option("truncate", "false")
      .start()
    query.awaitTermination()

  }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/StructuredStreaming_WindowandEvent.gif)

### 容错语义

提供end-to-end exactly-once语义是structured streaming设计背后的关键目标之一。 为了实现这一点，Spark设计了structured streaming的sources，sinks和执行引擎，可靠地跟踪处理进程的准确进度，以便它可以通过重新启动和/或重新处理来解决任何类型的故障。 假设每个Streaming源具有跟踪流中读取位置的偏移（类似于Kafka偏移或Kinesis序列号）。 引擎使用检查点和WAL（write ahead logs）记录每个触发器中正在处理的数据的偏移范围。 Streaming sinks为了解决重复计算被设计为幂等。 一起使用可重放sources和幂等sinks，Structured Streaming可以在任何故障下确保end-to-end exactly-once的语义。

###  Watermarking

现在考虑如果其中一个事件延迟到达应用程序会发生什么。 例如，说在12:04（即事件时间）生成的一个单词可以在12:11被应用程序接收。 应用程序应该使用时间12:04而不是12:11更新12:00 - 12:10的窗口的较旧计数。 这在我们基于窗口的分组中自然发生 - Structured Streaming可以长时间维持部分聚合的中间状态，以便迟到的数据可以正确地更新旧窗口的聚合，如下所示。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190608100551.png)

为了持续几天运行这个查询，系统必须限制其累积的内存中间状态的数量。这意味着系统需要知道什么时候可以从内存状态中删除旧的聚合，因为应用程序不会再为该集合接收到较晚的数据。为了实现这一点，在Spark 2.1中引入了watermarking，让引擎自动跟踪数据中的当前event-time，并尝试相应地清理旧状态。您可以通过指定事件时间列来定义查询的watermarking，并根据事件时间指定数据的延迟时间的阈值。对于从时间T开始的特定窗口，引擎将保持状态，并允许延迟数据更新状态，直到引擎看到最大事件时间-迟到的最大阈值。换句话说，阈值内的迟到数据将被聚合，但是比阈值晚的数据将被丢弃。让我们以一个例子来理解这一点。我们可以使用Watermark（）轻松定义上一个例子中的watermarking ，

```scala
val windowedCounts = words
    .withWatermark("timestamp", "10 minutes")
    .groupBy(
        window($"timestamp", "10 minutes", "5 minutes"),
        $"word")
    .count()
```

在这个例子中，我们正在定义“timestamp”列的查询的watermark ，并将“10分钟”定义为允许数据延迟的阈值。 如果此查询在更新输出模式下运行（稍后在“输出模式”部分中讨论），则引擎将继续更新Resule表中窗口的计数，直到窗口比watermark 旧，滞后于当前事件时间列“ timestamp“10分钟。 

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190608101146.png)

与之前的更新模式类似，引擎维护每个窗口的中间计数。 但是，部分计数不会更新到结果表，也不写入sink。 引擎等待“10分钟”接收迟到数据，然后丢弃窗口（watermark）的中间状态，并将最终计数附加到结果表sink。 例如，窗口12:00 - 12:10的最终计数仅在watermark更新到12:11之后才附加到结果表中。 
watermarking 清理聚合状态的条件重要的是要注意，为了清理聚合查询中的状态，必须满足以下条件（从Spark 2.1.1开始，以后再进行更改）。 

- 输出模式必须是追加或更新。 完整模式要求保留所有聚合数据，因此不能使用watermarking 去掉中间状态。 有关每种输出模式的语义的详细说明，请参见“输出模式”部分。 
- 聚合必须具有事件时间列或事件时间列上的窗口。 
- 必须在与聚合中使用的时间戳列相同的列上使用withWatermark 。 例如，df.withWatermark（“time”，“1 min”）.groupBy（“time2”）.count（）在附加输出模式中无效，因为watermark 在不同的列上定义为聚合列。 
- 必须在聚合之前调用withWatermark才能使用watermark 细节。 例如，在附加输出模式下，``df.groupBy（“time”）.count（）.withWatermark（“time”，“1 min”）``无效。

### Join操作

Streaming DataFrames可以与静态 DataFrames连接，以创建新的Streaming DataFrames。 例如下面的例子。

```scala
val staticDf = spark.read. ...
val streamingDf = spark.readStream. ... 

streamingDf.join(staticDf, "type")          // inner equi-join with a static DF
streamingDf.join(staticDf, "type", "right_join")  // right outer join with a static DF
```


在Spark 2.3中，Spark添加了对流 - 流 Join的支持，也就是说，您可以加入两个 streaming Datasets/DataFrames。 在两个数据流之间生成连接结果的挑战是，在任何时间点，dataset的view对于连接的两侧都是不完整的，这使得在输入之间找到匹配更加困难。 从一个输入流接收的任何行的数据都可以与来自另一个输入流的未来输入的任何一条数据匹配，尚未接收的行匹配。 因此，对于两个输入流，我们将过去的输入缓冲为流状态，以便我们可以将每个未来输入与过去的输入相匹配，从而生成Join结果。 此外，类似于流聚合，Spark自动处理迟到的无序数据，并可以使用水印限制状态。 

```scala
mport org.apache.spark.sql.functions.expr

val impressions = spark.readStream. ...
val clicks = spark.readStream. ...

// Apply watermarks on event-time columns
val impressionsWithWatermark = impressions.withWatermark("impressionTime", "2 hours")
val clicksWithWatermark = clicks.withWatermark("clickTime", "3 hours")

// Join with event-time constraints
impressionsWithWatermark.join(
  clicksWithWatermark,
  expr("""
    clickAdId = impressionAdId AND
    clickTime >= impressionTime AND
    clickTime <= impressionTime + interval 1 hour
    """)
)
```

## 不支持的操作

有几个DataFrame / Dataset操作不支持streaming DataFrames / Datasets。 其中一些如下。 
- streaming Datasets不支持多个streaming聚合（即streaming DF上的聚合链）。 
- 流数据集不支持limit和取前N行。 
- Streaming Datasets不支持Distinct 操作。 
- 只有在在完全输出模式的聚合之后，streaming Datasets才支持排序操作。 
- 有条件地支持Streaming和静态Datasets之间的外连接。 
不支持与 streaming Dataset的Full outer join 
不支持streaming Dataset 在右侧的Left outer join 
不支持streaming Dataset在左侧的Right outer join

- 两个streaming Datasets之间的任何种类型的join都不受支持

此外，还有一些Dataset方法将不适用于streaming Datasets。 它们是立即运行查询并返回结果的操作，这在streaming Datasets上没有意义。 相反，这些功能可以通过显式启动streaming查询来完成（参见下一节）。 
\- count() - 无法从流数据集返回单个计数。 而是使用ds.group By.count（）返回一个包含running count的streaming Dataset 。 
\- foreach() - 而是使用ds.writeStream.foreach（…）（见下一节）。 
\- show() - Instead use the console sink (see next section).

## 流式查询

### 输出模式

- Append mode (default) - 这是默认模式，其中只有从上次触发后添加到结果表的新行将被输出到sink。 只有那些添加到“结果表”中并且从不会更改的行的查询才支持这一点。 因此，该模式保证每行只能输出一次（假定容错sink）。 例如，只有select，where，map，flatMap，filter，join等的查询将支持Append模式。 
- Complete mode -每个触发后，整个结果表将被输出到sink。 聚合查询支持这一点。 
- Update mode - （自Spark 2.1.1以来可用）只有结果表中自上次触发后更新的行才会被输出到sink。 更多信息将在以后的版本中添加。

| Type                                            | Supported Output Modes   | 备注                                                         |
| ----------------------------------------------- | ------------------------ | ------------------------------------------------------------ |
| 没有聚合的查询                                  | Append, Update           | 不支持完整模式，因为将所有数据保存在结果表中是不可行的。     |
| 有聚合的查询：使用watermark对event-time进行聚合 | Append, Update, Complete | 附加模式使用watermark 来降低旧聚合状态。 但是，窗口化聚合的输出会延迟“withWatermark（）”中指定的晚期阈值，因为模式语义可以在结果表中定义后才能将结果表添加到结果表中（即在watermark 被交叉之后）。 有关详细信息，请参阅后期数据部分。更新模式使用水印去掉旧的聚合状态。完全模式不会丢弃旧的聚合状态，因为根据定义，此模式保留结果表中的所有数据。 |
| 有聚合的查询：其他聚合                          | Complete, Update         | 由于没有定义watermark （仅在其他类别中定义），旧的聚合状态不会被丢弃。不支持附加模式，因为聚合可以更新，从而违反了此模式的语义。 |

### Output Sinks

File sink-将输出存储到目录

```scala
writeStream
    .format("parquet")        // 也可以是 "orc", "json", "csv", 等等.
    .option("path", "path/to/destination/dir")
    .start()
```

Foreach sink - 对输出中的记录运行任意计算。 

```scala
writeStream
    .foreach(...)
    .start()
```

Console sink (for debugging) 

每次触发时将输出打印到控制台/ stdout。 都支持“Append ”和“Complete ”输出模式。 这应该用于低数据量的调试目的，因为在每次触发后，整个输出被收集并存储在驱动程序的内存中。

```scala
writeStream
    .format("console")
    .start()
```

Memory sink (for debugging) 

输出作为内存表存储在内存中。 都支持“Append ”和“Complete ”输出模式。 由于整个输出被收集并存储在驱动程序的内存中，所以应用于低数据量的调试目的。 因此，请谨慎使用。

```scala
writeStream
    .format("memory")
    .queryName("tableName")
    .start()
```

| sink         | Supported Output Modes    | Options                                                      | Fault-tolerant                                           | Notes                                     |
| ------------ | ------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- | ----------------------------------------- |
| File Sink    | Append                    | path：输出目录的路径，必须指定。 maxFilesPerTrigger：每个触发器中要考虑的最大新文件数（默认值：无最大值） latestFirst：是否首先处理最新的新文件，当有大量的文件积压（default：false）时很有用 有关特定于文件格式的选项，请参阅DataFrameWriter（Scala / Java / Python）中的相关方法。 例如。 对于“parquet”格式选项请参阅DataFrameWriter.parquet（） | yes                                                      | 支持对分区表的写入。 按时间划分可能有用。 |
| Foreach Sink | Append, Update, Compelete | None                                                         | 取决于ForeachWriter的实现                                | 更多细节在下一节                          |
| Console Sink | Append, Update, Complete  | numRows：每次触发打印的行数（默认值：20）truncate：输出太长是否截断（默认值：true） | no                                                       |                                           |
| Memory Sink  | Append, Complete          | None                                                         | 否。但在Complete模式下，重新启动的查询将重新创建整个表。 | 查询名就是表名                            |

您必须调用start（）来实际启动查询的执行。 这将返回一个StreamingQuery对象，它是连续运行执行的句柄。 您可以使用此对象来管理查询

```scala
val noAggDF = deviceDataDf.select("device").where("signal > 10")   


// Print new data to console
noAggDF
  .writeStream
  .format("console")
  .start()


// Write new data to Parquet files
noAggDF
  .writeStream
  .format("parquet")
  .option("checkpointLocation", "path/to/checkpoint/dir")
  .option("path", "path/to/destination/dir")
  .start()

// ========== DF with aggregation ==========
val aggDF = df.groupBy("device").count()


// Print updated aggregations to console
aggDF
  .writeStream
  .outputMode("complete")
  .format("console")
  .start()


// Have all the aggregates in an in-memory table 
aggDF
  .writeStream
  .queryName("aggregates")    // this query name will be the table name
  .outputMode("complete")
  .format("memory")
  .start()


spark.sql("select * from aggregates").show()   // interactively query in-memory table
```

### Foreach和ForeachBatch

foreach和foreachBatch操作允许您在流式查询的输出上应用任意操作和编写逻辑。 它们的用例略有不同 - 虽然foreach允许在每一行上自定义写入逻辑，foreachBatch允许在每个微批量的输出上进行任意操作和自定义逻辑。

foreachBatch（）允许您指定在流式查询的每个微批次的输出数据上执行的函数。 从Spark 2.4开始，Scala，Java和Python都支持它。 它需要两个参数：DataFrame或Dataset，它具有微批次的输出数据和微批次的唯一ID。

```scala
streamingDF.writeStream.foreachBatch { (batchDF: DataFrame, batchId: Long) =>
  // Transform and write batchDF 
}.start()
```

#### foreachBatch

重用现有的批处理数据源 - 对于许多存储系统，可能还没有可用的流式接收器，但可能已经存在用于批量查询的数据写入器。使用foreachBatch，您可以在每个微批次的输出上使用批处理数据编写器。

写入多个位置 - 如果要将流式查询的输出写入多个位置，则可以简单地多次写入输出DataFrame / Dataset。但是，每次写入尝试都会导致重新计算输出数据（包括可能重新读取输入数据）。要避免重新计算，您应该缓存输出DataFrame / Dataset，将其写入多个位置，然后将其解除。这是一个大纲。

```scala
streamingDF.writeStream.foreachBatch { (batchDF: DataFrame, batchId: Long) => batchDF.persist() batchDF.write.format(…).save(…) // location 1 batchDF.write.format(…).save(…) // location 2 batchDF.unpersist() }
```

应用其他DataFrame操作 - 流式DataFrame中不支持许多DataFrame和Dataset操作，因为Spark不支持在这些情况下生成增量计划。使用foreachBatch，您可以在每个微批输出上应用其中一些操作。但是，您必须自己解释执行该操作的端到端语义。注意：默认情况下，foreachBatch仅提供至少一次写保证。但是，您可以使用提供给该函数的batchId作为重复数据删除输出并获得一次性保证的方法。 foreachBatch不适用于连续处理模式，因为它从根本上依赖于流式查询的微批量执行。如果以连续模式写入数据，请改用foreach。

注意：默认情况下，foreachBatch仅提供至少一次写保证。 但是，您可以使用提供给该函数的batchId作为重复数据删除输出并获得一次性保证的方法。 foreachBatch不适用于连续处理模式，因为它从根本上依赖于流式查询的微批量执行。 如果以连续模式写入数据，请改用foreach。

#### Foreach

如果foreachBatch不是一个选项（例如，相应的批处理数据写入器不存在，或连续处理模式），那么您可以使用foreach表达自定义编写器逻辑。 具体来说，您可以通过将数据划分为三种方法来表达数据写入逻辑：打开，处理和关闭。 从Spark 2.4开始，foreach可用于Scala，Java和Python。

```scala
treamingDatasetOfString.writeStream.foreach(
  new ForeachWriter[String] {

    def open(partitionId: Long, version: Long): Boolean = {
      // Open connection
    }

    def process(record: String): Unit = {
      // Write string to connection
    }

    def close(errorOrNull: Throwable): Unit = {
      // Close the connection
    }
  }
).start()
```

执行语义启动流式查询时，Spark以下列方式调用函数或对象的方法：

此对象的单个副本负责查询中单个任务生成的所有数据。换句话说，一个实例负责处理以分布式方式生成的数据的一个分区。

此对象必须是可序列化的，因为每个任务都将获得所提供对象的新的序列化反序列化副本。 因此，强烈建议在调用open（）方法之后完成用于写入数据的任何初始化（例如，打开连接或启动事务），这表示任务已准备好生成数据。

```scala
- 方法的生命周期如下：
-   For each partition with partition_id:
    - For each batch/epoch of streaming data with epoch_id:
        - open(partitionId, epochId) 被调用
        - 如果open（...）返回true，则对于partition和 batch/epoch中的每一行，将调用方法process(row)
        - 调用方法close（错误），在处理行时看到错误（如果有的话话）。
```

当失败导致某些输入数据的重新处理时，open（）方法中的partitionId和epochId可用于对生成的数据进行重复数据删除。 这取决于查询的执行模式。 如果以微批处理模式执行流式查询，则保证由唯一元组（partition_id，epoch_id）表示的每个分区具有相同的数据。 因此，（partition_id，epoch_id）可用于对数据进行重复数据删除和/或事务提交，并实现一次性保证。 但是，如果正在以连续模式执行流式查询，则此保证不成立，因此不应用于重复数据删除。

### 触发器

流式查询的触发器设置定义了流式数据处理的时间，查询是作为具有固定批处理间隔的微批量查询还是作为连续处理查询来执行。 以下是支持的各种触发器。

| Trigger Type                                           | Description                                                  |
| :----------------------------------------------------- | :----------------------------------------------------------- |
| 未指定（默认）                                         | 如果未明确指定触发设置，则默认情况下，查询将以微批处理模式执行，一旦前一个微批处理完成处理，将立即生成微批处理。 |
| **Fixed interval micro-batches**                       | 查询将以微批处理模式执行，其中微批处理将以用户指定的间隔启动。<br/>如果先前的微批次在该间隔内完成，则引擎将等待该间隔结束，然后开始下一个微批次。<br/>如果前一个微批次需要的时间长于完成的间隔（即如果错过了间隔边界），则下一个微批次将在前一个完成后立即开始（即，它不会等待下一个间隔边界） ）。<br/>如果没有可用的新数据，则不会启动微批次。 |
| One-time micro-batch                                   | 查询将执行*仅一个*微批处理所有可用数据，然后自行停止。 这在您希望定期启动集群，处理自上一个时间段以来可用的所有内容，然后关闭集群的方案中非常有用。 在某些情况下，这可能会显着节省成本。 |
| **Continuous with fixed checkpoint interval** *(实验)* | 查询将以新的低延迟，连续处理模式执行。在下面的连续处理部分中阅读更多相关信息。 [Continuous Processing section](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#continuous-processing-experimental) |

```scala
import org.apache.spark.sql.streaming.Trigger

// Default trigger (runs micro-batch as soon as it can)
df.writeStream
  .format("console")
  .start()

// ProcessingTime trigger with two-seconds micro-batch interval
df.writeStream
  .format("console")
  .trigger(Trigger.ProcessingTime("2 seconds"))
  .start()

// One-time trigger
df.writeStream
  .format("console")
  .trigger(Trigger.Once())
  .start()

// Continuous trigger with one-second checkpointing interval
df.writeStream
  .format("console")
  .trigger(Trigger.Continuous("1 second"))
  .start()
```

## 管理流式查询

启动查询时创建的StreamingQuery对象可用于监视和管理查询。

```scala
val query = df.writeStream.format("console").start()   // get the query object

query.id          // get the unique identifier of the running query that persists across restarts from checkpoint data

query.runId       // get the unique id of this run of the query, which will be generated at every start/restart

query.name        // get the name of the auto-generated or user-specified name

query.explain()   // print detailed explanations of the query

query.stop()      // stop the query

query.awaitTermination()   // block until query is terminated, with stop() or with error

query.exception       // the exception if the query has been terminated with error

query.recentProgress  // an array of the most recent progress updates for this query

query.lastProgress    // the most recent progress update of this streaming query
```

您可以在单个SparkSession中启动任意数量的查询。 它们将同时运行，共享群集资源,您可以使用`sparkSession.streams()`来获取可用于管理当前活动的查询的StreamingQueryManager

```scala
val spark: SparkSession = ...

spark.streams.active    // get the list of currently active streaming queries

spark.streams.get(id)   // get a query object by its unique id

spark.streams.awaitAnyTermination()   // block until any one of them terminates
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190608115959.png)

更多请参考Spark官方网站

## 参考资料

Spark官方网站































