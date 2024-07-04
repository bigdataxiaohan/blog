---
title: Spark之SparkStreaming数据源
date: 2019-06-05 22:52:19
tags: SparkStreaming
categories: 大数据
---

DStreams输入

Spark Streaming原生支持一些不同的数据源。一些“核心”数据源已经被打包到Spark Streaming 的 Maven 工件中，而其他的一些则可以通过 `spark-streaming-kafka` 等附加工件获取。每个接收器都以 Spark 执行器程序中一个长期运行的任务的形式运行，因此会占据分配给应用的 CPU 核心。此外，我们还需要有可用的 CPU 核心来处理数据。这意味着如果要运行多个接收器，就必须至少有和接收器数目相同的核心数，还要加上用来完成计算所需要的核心数。例如，如果我们想要在流计算应用中运行 10 个接收器，那么至少需要为应用分配 11 个 CPU 核心。所以如果在本地模式运行，不要使用local或者local[1]。

### 文件数据源

文件数据流：能够读取所有HDFS API兼容的文件系统文件，通过fileStream方法进行读取。

Spark Streaming 将会监控 dataDirectory 目录并不断处理移动进来的文件，目前不支持嵌套目录。

文件需要有相同的数据格式。

文件进入 dataDirectory的方式需要通过移动或者重命名来实现。

一旦文件移动进目录，则不能再修改，即便修改了也不会读取新数据。

如果文件比较简单，则可以使用 streamingContext.textFileStream(dataDirectory)方法来读取文件。文件流不需要接收器，不需要单独分配CPU核。

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

object StreamingHDFS {
  def main(args: Array[String]): Unit = {
    //创建配置
    val sparkConf = new SparkConf().setAppName("streaming data from HDFS").setMaster("local[*]")
    //创建StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(5))
    //从HDFS接口数据
    val lines = ssc.textFileStream("hdfs://datanode1:9000/input/streaming/")

    val words = lines.flatMap(_.split(" "))

    val wordCounts = words.map((_, 1)).reduceByKey(_ + _)

    wordCounts.print()
    ssc.start()
    ssc.awaitTermination()

  }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/HDFSStreaming.gif)

### 自定义配置

通过继承Receiver，并实现onStart、onStop方法来自定义数据源采集。

```scala
import java.io.{BufferedReader, InputStreamReader}
import java.net.Socket

import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.receiver.Receiver

/**
  * Created by 清风笑丶 Cotter on 2019/6/3.
  */
class CustomerRecevicer(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {
  //接收器启动的时候子自动调用
  override def onStart(): Unit = {
    //创建线程
    new Thread("receiver") {
      override def run(): Unit = {
        //接受数据并提交给框架
        receive()
      }
    }.start()
  }

  def receive(): Unit = {

    var socket: Socket = null
    var input: String = null
    try {
      socket = new Socket(host, port)
      //生成输入流
      val reader = new BufferedReader(new InputStreamReader(socket.getInputStream))

      //接收数据
      //            input = reader.readLine()
      while (!isStopped() && (input = reader.readLine()) != null) {
        store(input)
      }
      restart("restart")

    } catch {
      case e: java.net.ConnectException => restart("restart")
      case t:Throwable => restart("restart")
    }
  }

  //接收器关闭的时候调用
  override def onStop(): Unit = {
  }
}
```

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}


object CustomerStreamingWordCount extends App {
  //创建配置
  val sparkConf = new SparkConf().setAppName("streaming word count").setMaster("local[*]")
  //创建StreamingContext
  val ssc = new StreamingContext(sparkConf, Seconds(5))
  //从socket接口数据   
  val lineDStream = ssc.receiverStream(new CustomerRecevicer("datanode1", 9999))  //自定义的使用的是receiverStream

  val wordDStream = lineDStream.flatMap(_.split(" "))
  val word2CountDStream = wordDStream.map((_, 1))
  val result = word2CountDStream.reduceByKey(_ + _)


  result.print()
  //启动
  ssc.start()
  ssc.awaitTermination()
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/CustomersparkStreamingWordCount.gif)

### RDD队列

Spark Streaming也可以使用 streamingContext.queueStream(queueOfRDDs)来创建DStream，每一个推送到这个队列中的RDD，都会作为一个DStream处理。

```scala
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.{Seconds, StreamingContext}

import scala.collection.mutable

object QueueRdd {

  def main(args: Array[String]) {

    val conf = new SparkConf().setMaster("local[2]").setAppName("QueueRdd")
    val ssc = new StreamingContext(conf, Seconds(1))

    //创建RDD队列
    val rddQueue = new mutable.SynchronizedQueue[RDD[Int]]()

    // 创建QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue)

    //处理队列中的RDD数据
    val mappedStream = inputStream.map(x => (x % 10, 1))
    val reducedStream = mappedStream.reduceByKey(_ + _)

    //打印结果
    reducedStream.print()

    //启动计算
    ssc.start()

    // Create and push some RDDs into
    for (i <- 1 to 30) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 300, 10)
      Thread.sleep(5000)

      //通过程序停止StreamingContext的运行
      //ssc.stop()
    }
  }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/QueueRdd.gif)

### 高级数据源(Kafka等)

这一类的来源需要外部接口，其中一些有复杂的依赖关系（如Kafka和Flume),因此通过这些来源创建DStreams需要明确其依赖。在工程中需要引入 Maven 工件 spark- streaming-kafka_2.10 来使用它。包内提供的 KafkaUtils 对象可以在 StreamingContext 和 JavaStreamingContext 中以你的 Kafka 消息创建出 DStream。由于 KafkaUtils 可以订阅多个主题，因此它创建出的 DStream 由成对的主题和消息组成。要创建出一个流数据，需 要使用 StreamingContext 实例、一个由逗号隔开的 ZooKeeper 主机列表字符串、消费者组的名字(唯一名字)，以及一个从主题到针对这个主题的接收器线程数的映射表来调用 createStream() 方法

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190604111236.png)

```scala
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * Created by 清风笑丶 Cotter on 2019/6/4.
  */
object KafkaStreming {
  def main(args: Array[String]): Unit = {

    //配置
    val conf = new SparkConf().setAppName("KafkaStreaming") setMaster ("local[*]")

    val ssc = new StreamingContext(conf, Seconds(5))

    //kafka的参数
    val brokers = "datanode1:9092,datanode2:9092,datanode3:9092"
    val zookeeper = "datanode1:2181,datanode2:2181,datanode3:2181"
    val sourceTopic = "source"
    val targetTopic = "target"
    val consumerGroup = "consumer"

    //封装kafka参数
    val kafkaParams = Map[String, String] {
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokers
      ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG -> "org.apache.kafka.common.serialization.StringDeserializer"
      ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG -> "org.apache.kafka.common.serialization.StringDeserializer"
      ConsumerConfig.GROUP_ID_CONFIG -> consumerGroup


    }

    val kafkaDStream = KafkaUtils.x(ssc, kafkaParams, Set(sourceTopic))
    kafkaDStream.print()

    ssc.start()
    ssc.awaitTermination()


  }
}

```

```shell
nohup /opt/module/kafka/bin/kafka-server-start.sh /opt/module/kafka/config/server.properties & #启动kafka
[hadoop@datanode1 bin]$ nohup ./kafka-manager  -java-home /opt/module/jdk1.8.0_162/  -Dconfig.file=../conf/application.conf >/dev/null 2>&1 &  #启动kafkamanager
 /opt/module/kafka/bin/kafka-topics.sh --zookeeper datanode1:2181 --create --replication-factor 2 --partitions 2 --topic source #创建一个topic
  /opt/module/kafka/bin/kafka-console-producer.sh --broker-list datanode1:9092 --topic source #启动生产者
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/KafkaSpark.gif)

```scala
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.{SparkConf, TaskContext}
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.kafka010.{HasOffsetRanges, KafkaUtils, OffsetRange}
import org.apache.spark.streaming.{Seconds, StreamingContext}


/**
  * Created by 清风笑丶 Cotter on 2019/6/5.
  */
object StreamingWithKafka {

  private val brokeList = "datanode1:9092,datanode2:9092,datanode2:9092"
  private val topic = "topic-spark"
  private val group = "group-spark"
  private val checkpointDir = "/opt/kafka/checkpoint"

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamingWithKafka")

    val ssc = new StreamingContext(sparkConf, Seconds(5))
    ssc.checkpoint(checkpointDir)

    //创建kafka的连接对象
    val kafkaParams = Map[String, Object](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokeList, //Kafka集群
      ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG -> classOf[StringDeserializer], //序列化
      ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG -> classOf[StringDeserializer], //序列化
      ConsumerConfig.GROUP_ID_CONFIG -> group, //消费者组
      ConsumerConfig.AUTO_OFFSET_RESET_CONFIG -> "latest", //latest自动重置偏移量为最新的偏移量
      ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG -> (false: java.lang.Boolean) //是否自动提交
    )
    //创建DStream,发挥接受的消息
    val stream = KafkaUtils.createDirectStream[String, String](
      ssc,
      PreferConsistent,
      Subscribe[String, String](List(topic), kafkaParams))

    val value = stream.map(record => {
      val intVal = Integer.valueOf(record.value())
      println(intVal)        // 打印输入数字
      intVal
    }).reduce(_ + _)   //相加
    value.print()   //输出

    stream.foreachRDD(rdd => {
      val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      rdd.foreachPartition { iter =>
        val o: OffsetRange = offsetRanges(TaskContext.getPartitionId())
        print(s"${o.topic} ${o.partition} ${o.fromOffset} ${o.untilOffset}")

      }
    }
    )
    
    ssc.start
    ssc.awaitTermination
  }

}
```

```shell
[hadoop@datanode1 kafka]$ bin/kafka-topics.sh --zookeeper datanode1:2181 --create --replication-factor 3 --partitions 1 --topic-spark
bin/kafka-console-producer.sh --broker-list datanode1:9092 --topic topic-spark

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/KafkaSparkStreaming.gif)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190605200456.png)

因为在本地运行E盘放了我们程序相当于E盘就是根目录了,可以指定HDFS.

#### 两种连接方式

Spark对于Kafka的连接主要有两种方式，一种是DirectKafkaInputDStream，另外一种是KafkaInputDStream。DirectKafkaInputDStream 只在 driver 端接收数据，所以继承了 InputDStream，是没有 receivers 的。

主要通过KafkaUtils.createDirectStream以及KafkaUtils.createStream这两个 API 来创建，除了要传入的参数不同外，接收 kafka 数据的节点、拉取数据的时机也完全不同。

##### createStream[Receiver-based]

这种方法使用一个 Receiver 来接收数据。在该 Receiver 的实现中使用了 Kafka high-level consumer API。Receiver 从 kafka 接收的数据将被存储到 Spark executor 中，随后启动的 job 将处理这些数据。

在默认配置下，该方法失败后会丢失数据（保存在 executor 内存里的数据在 application 失败后就没了），若要保证数据不丢失，需要启用 WAL（即预写日志至 HDFS、S3等），这样再失败后可以从日志文件中恢复数据。

在该函数中，会新建一个 KafkaInputDStream对象，KafkaInputDStream继承于 ReceiverInputDStream。KafkaInputDStream实现了getReceiver方法，返回接收器的实例：

```scala
  def getReceiver(): Receiver[(K, V)] = {
    if (!useReliableReceiver) {
      //< 不启用 WAL
      new KafkaReceiver[K, V, U, T](kafkaParams, topics, storageLevel)
    } else {
      //< 启用 WAL
      new ReliableKafkaReceiver[K, V, U, T](kafkaParams, topics, storageLevel)
    }
  }
```

根据是否启用 WAL，receiver 分为KafkaReceiver 和 ReliableKafkaReceiver。下图描述了 KafkaReceiver 接收数据的具体流程：

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190605201219.png)

Kafka Topic 的 partitions 与RDD 的 partitions 没有直接关系，不能一一对应。如果增加 topic 的 partition 个数的话仅仅会增加单个 Receiver 接收数据的线程数。事实上，使用这种方法只会在一个 executor 上启用一个 Receiver，该 Receiver 包含一个线程池，线程池的线程个数与所有 topics 的 partitions 个数总和一致，每条线程接收一个 topic 的一个 partition 的数据。而并不会增加处理数据时的并行度。

对于一个 topic，可以使用多个 groupid 相同的 input DStream 来使用多个 Receivers 来增加并行度，然后 union 他们；对于多个 topics，除了可以用上个办法增加并行度外，还可以对不同的 topic 使用不同的 input DStream 然后 union 他们来增加并行度

如果你启用了 WAL，为能将接收到的数据将以 log 的方式在指定的存储系统备份一份，需要指定输入数据的存储等级为 StorageLevel.MEMORY_AND_DISK_SER 或 StorageLevel.MEMORY_AND_DISK_SER_2

##### createDirectStream[WithOut Receiver]

自 Spark-1.3.0 起，提供了不需要 Receiver 的方法。替代了使用 receivers 来接收数据，该方法定期查询每个 topic+partition 的 lastest offset，并据此决定每个 batch 要接收的 offsets 范围。

createDirectStream调用中，会新建DirectKafkaInputDStream，DirectKafkaInputDStream#compute(validTime: Time)会从 kafka 拉取数据并生成 RDD，流程如下：

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190605201632.png)

该函数主要做了以下三个事情：

确定要接收的 partitions 的 offsetRange，以作为第2步创建的 RDD 的数据来源

创建 RDD 并执行 count 操作，使 RDD 真实具有数据

以 streamId、数据条数，offsetRanges 信息初始化 inputInfo 并添加到 JobScheduler 中

```scala
进一步看 KafkaRDD 的 getPartitions 实现：

  override def getPartitions: Array[Partition] = {

    offsetRanges.zipWithIndex.map { case (o, i) =>

        val (host, port) = leaders(TopicAndPartition(o.topic, o.partition))

        new KafkaRDDPartition(i, o.topic, o.partition, o.fromOffset, o.untilOffset, host, port)

    }.toArray

  }
```



从上面的代码可以很明显看到，KafkaRDD 的 partition 数据与 Kafka topic 的某个 partition 的 o.fromOffset 至 o.untilOffset 数据是相对应的，也就是说 KafkaRDD 的 partition 与 Kafka partition 是一一对应的

该方式相比使用 Receiver 的方式有以下好处：

简化并行：不再需要创建多个 kafka input DStream 然后再 union 这些 input DStream。使用 directStream，Spark Streaming会创建与 Kafka partitions 相同数量的 paritions 的 RDD，RDD 的 partition与 Kafka 的 partition 一一对应，这样更易于理解及调优

高效：在方式一中要保证数据零丢失需要启用 WAL（预写日志），这会占用更多空间。而在方式二中，可以直接从 Kafka 指定的 topic 的指定 offsets 处恢复数据，不需要使用 WAL

恰好一次语义保证：基于Receiver方式使用了 Kafka 的 high level API 来在 Zookeeper 中存储已消费的 offsets。这在某些情况下会导致一些数据被消费两次，比如 streaming app 在处理某个 batch  内已接受到的数据的过程中挂掉，但是数据已经处理了一部分，但这种情况下无法将已处理数据的 offsets 更新到 Zookeeper 中，下次重启时，这批数据将再次被消费且处理。基于direct的方式，使用kafka的简单api，Spark Streaming自己就负责追踪消费的offset，并保存在checkpoint中。Spark自己一定是同步的，因此可以保证数据是消费一次且仅消费一次。这种方式中，只要将 output 操作和保存 offsets 操作封装成一个原子操作就能避免失败后的重复消费和处理，从而达到恰好一次的语义（Exactly-once）

通过以上分析，我们可以对这两种方式的区别做一个总结：

createStream会使用 Receiver；而createDirectStream不会

createStream使用的 Receiver 会分发到某个 executor 上去启动并接受数据；而createDirectStream直接在 driver 上接收数据

createStream使用 Receiver 源源不断的接收数据并把数据交给 ReceiverSupervisor 处理最终存储为 blocks 作为 RDD 的输入，从 kafka 拉取数据与计算消费数据相互独立；而createDirectStream会在每个 batch 拉取数据并就地消费，到下个 batch 再次拉取消费，周而复始，从 kafka 拉取数据与计算消费数据是连续的，没有独立开

createStream中创建的KafkaInputDStream 每个 batch 所对应的 RDD 的 partition 不与 Kafka partition 一 一对应；而createDirectStream中创建的 DirectKafkaInputDStream 每个 batch 所对应的 RDD 的 partition 与 Kafka partition 一 一对应

#### Flume

Spark提供两个不同的接收器来使用Apache Flum 两个接收器简介如下。 

推式接收器该接收器以 Avro 数据池的方式工作，由 Flume 向其中推数据。 

拉式接收器该接收器可以从自定义的中间数据池中拉数据，而其他进程可以使用 Flume 把数据推进 该中间数据池。 

两种方式都需要重新配置 Flume，并在某个节点配置的端口上运行接收器(不是已有的 Spark 或者 Flume 使用的端口)。要使用其中任何一种方法，都需要在工程中引入 Maven 工件 spark-streaming-flume_2.10。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/20190605202028.png)

Avro 数据池的方式工作，我们需要配置 Flume 来把数据发到 Avro 数据池。我们提供的 FlumeUtils 对象会把接收器配置在一个特定的工作节点的主机名及端口号上。这些设置必须和 Flume 配置相匹配。 

虽然这种方式很简洁，但缺点是没有事务支持。这会增加运行接收器的工作节点发生错误 时丢失少量数据的几率。不仅如此，如果运行接收器的工作节点发生故障，系统会尝试从 另一个位置启动接收器，这时需要重新配置 Flume 才能将数据发给新的工作节点。这样配 置会比较麻烦。 

较新的方式是拉式接收器(在Spark 1.1中引入)，它设置了一个专用的Flume数据池供 Spark Streaming读取，并让接收器主动从数据池中拉取数据。这种方式的优点在于弹性较好，Spark Streaming通过事务从数据池中读取并复制数据。在收到事务完成的通知前，这些数据还保留在数据池中。 

我们需要先把自定义数据池配置为 Flume 的第三方插件。安装插件的最新方法请参考 Flume 文档的相关部分([链接](https://flume.apache.org/FlumeUserGuide.html#installing-third-party- plugins))。由于插件是用 Scala 写的，因此需要把插件本身以及 Scala 库都添加到 Flume 插件 中。

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-flume-sink_2.11</artifactId>
    <version>1.2.0</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>2.11.11</version>
</dependency>
```

当你把自定义 Flume 数据池添加到一个节点上之后，就需要配置 Flume 来把数据推送到这个数据池中， 

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# source
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /home/hadoop/flumedata
a1.sources.r1.fileHeader = true

# Describe the sink
a1.sinks.k1.type = org.apache.spark.streaming.flume.sink.SparkSink
#表示从这里拉数据
a1.sinks.k1.hostname = 192.168.1.101
a1.sinks.k1.port = 4444

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```

```scala
import org.apache.spark.SparkConf
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.flume.{FlumeUtils, SparkFlumeEvent}
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
  * Created by 清风笑丶 Cotter on 2019/6/5.
  */
object FlumeStream {
  def main(args: Array[String]): Unit = {
    //spark配置
    val conf = new SparkConf().setMaster("local[*]").setAppName("Flume Spark Streaming")

    val ssc = new StreamingContext(conf, Seconds(5))

    val inputDstream:ReceiverInputDStream[SparkFlumeEvent]
    = FlumeUtils.createPollingStream(ssc, "192.168.1.101", 4444, StorageLevel.MEMORY_AND_DISK)
    val words = inputDstream.flatMap(x => new String(x.event.getBody.array()).split(" "))
    val wordAndOne : DStream[(String,Int)] = words.map((_,1))
    val result : DStream[(String,Int)] = wordAndOne.reduceByKey(_+_)

    result.print()

    // 启动流
    ssc.start()
    ssc.awaitTermination()
  }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SparkStreaming/flume_Spark.gif)











