---
title: Spark之RDD实战2
date: 2019-05-28 22:57:46
tags: 
- Spark
- RDD
categories: 大数据
---

依赖: RDD和它依赖的父RDD（s）的关系有两种不同的类型，即窄依赖（narrow dependency）和宽依赖（wide dependency）。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528225959.png)

### 窄依赖

窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用。

### 宽依赖

窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用,窄依赖我们形象的比喻为独生子女

### Lineage

RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage（即血统）记录下来，以便恢复丢失的分区。RDD的Lineage会记录RDD的元数据信息和转换行为，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528230120.png)

```scala
val text = sc.textFile("/input/test.txt")
val words = text.flatMap(_.split(" "))
val date = words.map((_,1))
val result = date.reduceByKey(_+_)
date.dependencies
result.dependencies
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528231029.png)

## DAG的生成

DAG(Directed Acyclic Graph)叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此宽依赖是划分Stage的依据。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528231121.png)

## RDD相关概念关系

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/RDD/20190528231142.png)

输入可能以多个文件的形式存储在HDFS上，每个File都包含了很多块，称为Block。当Spark读取这些文件作为输入时，会根据具体数据格式对应的InputFormat进行解析，一般是将若干个Block合并成一个输入分片，称为InputSplit，注意InputSplit不能跨越文件。随后将为这些输入分片生成具体的Task。InputSplit与Task是一一对应的关系。随后这些具体的Task每个都会被分配到集群上的某个节点的某个Executor去执行。

1)      每个节点可以起一个或多个Executor。

2)      每个Executor由若干core组成，每个Executor的每个core一次只能执行一个Task。

3)      每个Task执行的结果就是生成了目标RDD的一个partiton。

注意: 这里的core是虚拟的core而不是机器的物理CPU核，可以理解为就是Executor的一个工作线程。而 Task被执行的并发度 = Executor数目 * 每个Executor核数。至于partition的数目：

1)      对于数据读入阶段，例如sc.textFile，输入文件被划分为多少InputSplit就会需要多少初始Task。

2)      在Map阶段partition数目保持不变。

3)      在Reduce阶段，RDD的聚合会触发shuffle操作，聚合后的RDD的partition数目跟具体操作有关，例如repartition操作会聚合成指定分区数，还有一些算子是可配置的。

RDD在计算的时候，每个分区都会起一个task，所以rdd的分区数目决定了总的的task数目。申请的计算节点（Executor）数目和每个计算节点核数，决定了你同一时刻可以并行执行的task。

比如的RDD有100个分区，那么计算的时候就会生成100个task，你的资源配置为10个计算节点，每个两2个核，同一时刻可以并行的task数目为20，计算这个RDD就需要5个轮次。如果计算资源不变，你有101个task的话，就需要6个轮次，在最后一轮中，只有一个task在执行，其余核都在空转。如果资源不变，你的RDD只有2个分区，那么同一时刻只有2个task运行，其余18个核空转，造成资源浪费。这就是在spark调优中，增大RDD分区数目，增大任务并行度的做法。