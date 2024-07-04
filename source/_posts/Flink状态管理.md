---
title: Flink状态管理
date: 2020-11-03 21:42:22
tags: Flink
categories: 大数据
---

在Flink中提供了StateBackend来存储和管理Checkpoints过程中的状态数据。

### 类别

在Flink中状态可以分为三种：

1. 基于内存的MemoryStateBackend(默认使用)
2. 基于文件系统的FsStateBackend
3. 基于RockDB作为存储介质的RocksDBState-Backend

#### MemoryStateBackend

MemoryStateBackend 将 state 数据存储在JVM堆内存中，基于内存的状态管理具有非常快速和高效的特点，但是受限于内存的容量限制，存储的状态数据过多就会导致系统内存溢出，影响整个应用的正常运行。值得注意的是MemoryStateBackend每个独立的状态（state）默认限制大小为 5MB，可以通过构造函数增加容量状态的大小不能超过 akka 的 Framesize 大小（（默认：10485760bit）聚合后的状态必须能够放进 JobManager 的内存中。

```java
        MemoryStateBackend memoryStateBackend = new MemoryStateBackend(MemoryStateBackend.DEFAULT_MAX_STATE_SIZE, false);
        executionEnvironment.setStateBackend((StateBackend) memoryStateBackend);
```

使用MemoryStateBackend只需要在setStateBackend制定即可， new MemoryStateBackend(DEFAULT_MAX_STATE_SIZE,false) 中的 false 代表关闭异步快照机制。MemoryStateBackend 适用于我们本地调试使用，来记录一些状态很小的 Job 状态信息。生产环境不建议使用。

#### FsStateBackend

FsStateBackend是基于文件系统的一种状态管理器，这里的文件系统可以是本地文件系统，也可以是HDFS分布式文件系统，使用FsStateBackend时，FsStateBackend 会把状态数据保存在 TaskManager 的内存中。CheckPoint 时，将状态快照写入到配置的文件系统目录中，少量的元数据信息存储到 JobManager 的内存中。

```java
        FsStateBackend fsStateBackend = new FsStateBackend("hdfs://hadoop102:9000/flink/checkpoints", false);
        executionEnvironment.setStateBackend((StateBackend) fsStateBackend);
```

FsStateBackend中第二个Boolean类型的参数指定是否以同步的方式进行状态数据记录，默认采用异步的方式将状态数据同步到文件系统中，异步方式能够尽可能避免在Checkpoint的过程中影响流式计算任务。如果用户想采用同步的方式进行状态数据的检查点数据，则将第二个参数指定为True即可。FsStateBackend适合任务状态非常大的情况，例如应用中含有时间范围非常长的窗口计算，或Key/value State状态数据量非常大的场景，基于文件系统存储相对比较稳定，借助于像HDFS分布式文件系统，能最大程度保证状态数据的安全性，不会出现因为外部故障而导致任务无法恢复等问题。

#### RocksDBStateBackend

RocksDBStateBackend是Flink中内置的第三方状态管理器，和前面的状态管理器不同，RocksDBStateBackend需要单独引入相关的依赖包到工程中。通过初始化RockDBStateBackend类，使可以得到RockDBStateBackend实例类。使用时需要引入依赖

```java
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
    <version>1.11.2</version>
    <scope>provided</scope>
</dependency>
```

RocksDBStateBackend 将正在运行中的状态数据保存在 RocksDB 数据库中，RocksDB 数据库默认将数据存储在 TaskManager 运行节点的数据目录下。RocksDBStateBackend在性能上要比FsStateBackend高一些，主要是因为借助于RocksDB存储了最新热数据，然后通过异步的方式再同步到文件系统中。RocksDB通过JNI的方式进行数据的交互，而JNI构建在byte[]数据结构之上，因此每次能够传输的最大数据量为2^31字节，也就是说每次在RocksDBStateBackend合并的状态数据量大小不能超过2^31字节限制，否则将会导致状态数据无法同步，这是RocksDB采用JNI方式的限制，用户在使用过程中应当注意。

### 重启策略

Flink支持不同的重启策略，这些重启策略控制着job失败后如何重启。集群可以通过默认的重启策略来重启，这个默认的重启策略通常在未指定重启策略的情况下使用，而如果Job提交的时候指定了重启策略，这个重启策略就会覆盖掉集群的默认重启策略。

| 重启策略     | 重启策略值   |
| ------------ | ------------ |
| Fixed delay  | fixed-delay  |
| Failure rate | failure-rate |
| No restart   | None         |

```java
package com.hph.state;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class RestartStrateegiesDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        //开启checkpointing
        env.enableCheckpointing(5000);
        //默认的策略是固定时间内无限重启
        DataStreamSource<String> input = env.socketTextStream("hadoop102", 8888);

        SingleOutputStreamOperator<String> upperCase = input.map(new MapFunction<String, String>() {
            @Override
            public String map(String s) throws Exception {
                if (s.startsWith("bug")) {
                    throw new RuntimeException("程序出现BUG");
                }
                return s.toUpperCase();
            }
        });
        upperCase.print();

        env.execute("RestartStrateegiesDemo");

    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105211547.png)

当程序出现异常时会一直处于重启状态，这是程序默认的策略。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105212010.png)

此时任务做了checkpoints，向JobManager汇报自己的情况。并重新启动任务。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105212319.png)

同是我们可以设置其他的重启策略如：尝试重启2次，重启间隔为2s。

```java
package com.hph.state;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import javax.print.DocFlavor;

public class RestartStrateegiesDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        //开启checkpointing
        env.enableCheckpointing(5000);

        env.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));
        //默认的策略是固定时间内无限重启
        DataStreamSource<String> input = env.socketTextStream("hadoop102", 8888);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = input.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {


                if (s.startsWith("bug")) {
                    throw new RuntimeException("程序出现BUG");
                }
                return Tuple2.of(s, 1);
            }
        });
        wordAndOne.keyBy(s -> s.f0).sum(1).print();

        env.execute("RestartStrateegiesDemo");

    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105214138.png)

当任务出现问题重启时，Flink依旧计算正确，这是基于内存的。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105214247.png)

此时任务依旧执行正确。

当程序出现地三次的报错时，程序不再重启，Flink的重启策略和状态管理保证了Flink程序的健壮性。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105214336.png)

```java
package com.hph.state;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.runtime.state.StateBackend;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import javax.print.DocFlavor;

public class RestartStrateegiesDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        //开启checkpointing
        env.enableCheckpointing(5000);

        env.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));
        //默认的策略是固定时间内无限重启
        DataStreamSource<String> input = env.socketTextStream("hadoop102", 8888);
        env.setStateBackend((StateBackend) new FsStateBackend("file:///D:/FlinkBackend"));

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = input.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {


                if (s.startsWith("bug")) {
                    throw new RuntimeException("程序出现BUG");
                }
                return Tuple2.of(s, 1);
            }
        });
        wordAndOne.keyBy(s -> s.f0).sum(1).print();

        env.execute("RestartStrateegiesDemo");

    }
}

```

### checkpoting

Flink中基于异步轻量级的分布式快照技术提供了Checkpoints容错机制，分布式快照可以将同一时间点Task/Operator的状态数据全局统一快照处理，包括Keyed State和Operator State。Flink会在输入的数据集上间隔性地生成checkpoint barrier，通过栅栏（barrier）将间隔时间段内的数据划分到相应的checkpoint中。当应用出现异常时，Operator就能够从上一次快照中恢复所有算子之前的状态，从而保证数据的一致性。例如在KafkaConsumer算子中维护Offset状态，当系统出现问题无法从Kafka中消费数据时，可以将Offset记录在状态中，当任务重新恢复时就能够从指定的偏移量开始消费数据。对于状态占用空间比较小的应用，快照产生过程非常轻量，高频率创建且对Flink任务性能影响相对较小。checkpoint过程中状态数据一般被保存在一个可配置的环境中，通常是在JobManager节点或HDFS上。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201105220059.png)

默认情况下Flink不开启检查点的，需要在程序中通过调用enable-Checkpointing(n)方法配置和开启检查点，其中n为检查点执行的时间间隔，单位为毫秒。

可以选择exactly-once语义保证整个应用内端到端的数据一致性，这种情况比较适合于数据要求比较高，不允许出现丢数据或者数据重复，与此同时，Flink的性能也相对较弱，而at-least-once语义更适合于时廷和吞吐量要求非常高但对数据的一致性要求不高的场景。如下通过setCheckpointingMode()方法来设定语义模式，默认情况下使用的是exactly-once模式。

超时时间指定了每次Checkpoint执行过程中的上限时间范围，一旦Checkpoint执行时间超过该阈值，Flink将会中断Checkpoint过程，并按照超时处理。该指标可以通过setCheckpointTimeout方法设定，默认为10分钟

```java
        env.getCheckpointConfig().setCheckpointTimeout(10000);
```

检查点之间最小时间间隔该参数主要目的是设定两个Checkpoint之间的最小时间间隔，防止出现例如状态数据过大而导致Checkpoint执行时间过长，从而导致Checkpoint积压过多，最终Flink应用密集地触发Checkpoint操作，会占用了大量计算资源而影响到整个应用的性能。

```java
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
```

通过setMaxConcurrentCheckpoints()方法设定能够最大同时执行的Checkpoint数量。在默认情况下只有一个检查点可以运行，根据用户指定的数量可以同时触发多个Checkpoint，进而提升Checkpoint整体的效率。

```java
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
```

设定周期性的外部检查点，然后将状态数据持久化到外部系统中，使用这种方式不会在任务正常停止的过程中清理掉检查点数据，而是会一直保存在外部系统介质中，另外也可以通过从外部检查点中对任务进行恢复。

```java
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107110858.png)

Flink会周期性的将数据同步到文件系统中，

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107111045.png)

### 从checkpoting中恢复计算的中间结果

这里我们提交了一个任务，并在socket中输入了 hadoop spark flink 三个字符

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107201312.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107201337.png)

此时的checkpoting执行到第五阶段，

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107201416.png)

我们停掉任务从第五阶段开始恢复计算。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107202522.png)

重新提交任务制定checkpoting点，并且设置savepotion的位置，作业即可从上次的状态中恢复过来，是不是很强大。当然也可是使用命令行提交任务制定从某处恢复数据。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107202221.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107202156.png)

### 使用RocksDB

RocksDBStateBackend需要单独引入相关的依赖包。RocksDB是使用C++编写的嵌入式kv存储引擎，其键值均允许使用二进制流。RocksDb会充分挖掘 Flash or RAM 硬件的读写特性，支持单个 KV 的读写以及批量读写。RocksDB 自身采用的一些数据结构如 LSM/SKIPLIST 等结构使得其有读放大、写放大和空间使用放大的问题。

```java
package com.hph.state.manage;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.contrib.streaming.state.RocksDBStateBackend;
import org.apache.flink.runtime.state.StateBackend;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class RocksdbStateAndToleration {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //开启checkpointing 每隔50分钟
        env.enableCheckpointing(1000 * 50);
        env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
        env.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));
        //默认的策略是固定时间内无限重启
        DataStreamSource<String> input = env.socketTextStream("hadoop102", 8888);

        //从界面获取参数
        env.setStateBackend((StateBackend) new RocksDBStateBackend(args[0], false));

        //程序异常或者认为Cannel掉不删除checkpoting 里面的数据
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        env.getCheckpointConfig().setCheckpointTimeout(10000);
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);

        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOne = input.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String s) throws Exception {

                if (s.startsWith("bug")) {
                    throw new RuntimeException("程序出现BUG");
                }
                return Tuple2.of(s, 1);
            }
        });
        wordAndOne.keyBy(s -> s.f0).sum(1).print();

        env.execute("RestartStrateegiesDemo");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20201107204550.png)

提交任务时制定文件系统路径,即可完成对RocksDB的使用，仅在Maven中引入即可。



