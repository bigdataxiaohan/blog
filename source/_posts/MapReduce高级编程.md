---
title: MapReduce高级编程
date: 2018-12-28 10:08:07
tags: MapReduce
categories: 大数据
---

## 计数器

数据集在进行MapReduce运算过程中，许多时候，用户希望了解待分析的数据的运行的运行情况。Hadoop内置的计数器功能收集作业的主要统计信息，可以帮助用户理解程序的运行情况，辅助用户诊断故障。

```
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/12/28 10:37:46 INFO client.RMProxy: Connecting to ResourceManager at datanode3/192.168.1.103:8032
18/12/28 10:37:48 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/12/28 10:37:50 INFO input.FileInputFormat: Total input paths to process : 2
18/12/28 10:37:50 INFO mapreduce.JobSubmitter: number of splits:2
18/12/28 10:37:51 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1545964109134_0001
18/12/28 10:37:53 INFO impl.YarnClientImpl: Submitted application application_1545964109134_0001
18/12/28 10:37:54 INFO mapreduce.Job: The url to track the job: http://datanode3:8088/proxy/application_1545964109134_0001/
18/12/28 10:37:54 INFO mapreduce.Job: Running job: job_1545964109134_0001
18/12/28 10:38:50 INFO mapreduce.Job: Job job_1545964109134_0001 running in uber mode : false
18/12/28 10:38:50 INFO mapreduce.Job:  map 0% reduce 0%
18/12/28 10:39:28 INFO mapreduce.Job:  map 100% reduce 0%
18/12/28 10:39:48 INFO mapreduce.Job:  map 100% reduce 100%
18/12/28 10:39:50 INFO mapreduce.Job: Job job_1545964109134_0001 completed successfully
18/12/28 10:39:51 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=78
                FILE: Number of bytes written=353015
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=258
                HDFS: Number of bytes written=31
                HDFS: Number of read operations=9
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
        Job Counters
                Launched map tasks=2
                Launched reduce tasks=1
                Data-local map tasks=2
                Total time spent by all maps in occupied slots (ms)=67297
                Total time spent by all reduces in occupied slots (ms)=16699
                Total time spent by all map tasks (ms)=67297
                Total time spent by all reduce tasks (ms)=16699
                Total vcore-milliseconds taken by all map tasks=67297
                Total vcore-milliseconds taken by all reduce tasks=16699
                Total megabyte-milliseconds taken by all map tasks=68912128
                Total megabyte-milliseconds taken by all reduce tasks=17099776
        Map-Reduce Framework
                Map input records=8
                Map output records=8
                Map output bytes=78
                Map output materialized bytes=84
                Input split bytes=212
                Combine input records=8
                Combine output records=6
                Reduce input groups=4
                Reduce shuffle bytes=84
                Reduce input records=6
                Reduce output records=4
                Spilled Records=12
                Shuffled Maps =2
                Failed Shuffles=0
                Merged Map outputs=2
                GC time elapsed (ms)=3303
                CPU time spent (ms)=8060
                Physical memory (bytes) snapshot=470183936
                Virtual memory (bytes) snapshot=6182424576
                Total committed heap usage (bytes)=261361664
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=46
        File Output Format Counters
                Bytes Written=31

```

这些记录了该程序运行过程的的一些信息的计数，如Map input records=8，表示Map有8条记录。可以看出来这些内置计数器可以被分为若干个组，即对于大多数的计数器来说，Hadoop使用的组件分为若干类。 

### 计数器列表

| 组别                                             | 名称/类别                                                    |
| ------------------------------------------------ | ------------------------------------------------------------ |
| MapReduce任务计数器（Map-Reduce Framework）      | org.apache.hadoop.mapreduce.TaskCounter                      |
| 文件系统计数器（File System Counters）           | org.apache.hadoop.mapreduce.FiIeSystemCounter                |
| 输入文件任务计数器（File Input Format Counters） | org.apache.hadoop.mapreduce.lib.input.FilelnputFormatCounter |
| 输出文件计数器（File Output Format Counters）    | org.apache.hadoop.mapreduce.lib.output.FileOutputFormatCounter |
| 作业计数器（Job Counters）                       | org.apache.hadoop.mapreduce.JobCounter                       |

大部分的Hadoop都有相应的计数器，可以对其进行追踪，方便处理运行中出现的问题，这些信息从应用角度又分为任务计数器和作业计数器：

#### 任务计数器

##### 内置MapReduce任务计数器

| 计数器名称                                          | 说明                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| map输人的记录数(MAP_INPUT_RECORDS）                 | 作业中所有map已处理的输人记录数。每次RecordReader读到一条记录并将其传给map的map()函数时，该计数器的值递增 |
| 分片（split）的原始字节数(SPLIT_RAW_BYTES)          | 由map读取的输人分片对象的字节数。这些对象描述分片元数据（文件的位移和长度），而不是分片的数据自身，因此总规模是小的 |
| map输出的记录数(MAP_OUTPUT_RECORDS)                 | 作业中所有map产生的map输出记录数。每次某一个map 的OutputCollector调用collect()方法时，该计数器的值增加 |
| map输出的字节数(MAP_OUTPUT_BYTES)                   | 作业中所有map产生的耒经压缩的输出数据的字节数·每次某一个map的OutputCollector调用collect()方法时，该计数器的值增加 |
| map输出的物化字节数（MAP_OUTPUT_MATERIALIZED_BYTES) | map输出后确实写到磁盘上的字节数；若map输出压缩功能被启用，则会在计数器值上反映出来 |
| combine输人的记录数(COMBINE_INPUT_RECORDS)          | 作业中所有combiner(如果有）已处理的输人记录数。combiner的迭代器每次读一个值，该计数器的值增加。注意：本计数器代表combiner已经处理的值的个数，并非不同的键组数（后者并无实所意文，因为对于combiner而言，并不要求每个键对应一个组。 |
| combine输出的记录数(COMBINE_OUTPUT_RECORDS)         | 作业中所有combiner（如果有）已产生的输出记录数。每当一个combiner的OutputCollector调用collect()方法时，该计数器的值增加 |
| reduce输人的组（REDUCE_INPUT_GROUPS）               | 作业中所有reducer已经处理的不同的码分组的个数。每当某一个reducer的reduce()被调用时，该计数器的值增加。 |
| reduce输人的记录数（REDUCE_INPUT_RECORDS)           | 作业中所有reducer已经处理的输人记录的个数。每当某个reducer的迭代器读一个值时，该计数器的值增加。如果所有reducer已经处理数完所有输人，則该计数器的值与计数器"map输出的记录"的值相同。 |
| reduce输出的记录数（REDUCE_OUTPUT_RECORDS）         | 作业中所有map已经产生的reduce输出记录数。每当某个reducer的OutputCollector调用collect()方法时，该计数器的值增加。 |
| reduce经过shuffle的字节数(REDUCE_SHUFFLE_BYTES)     | 由shuffle复制到reducer的map输出的字节数。                    |
| 溢出的记录数(SPILLED_RECORDS)                       | 作业中所有map和reduce任务溢出到磁盘的记录数                  |
| CPU毫秒(CPU_MILLISECONDS)                           | 一个任务的总CPU时间，以毫秒为单位，可由/proc/cpuinfo获取     |
| 物理内存字节数（PHYSICAL_MEMORY_BYTES）             | 一个任务所用的物理内存，以字节数为单位，可由/proc/meminfo获取 |
| 虚拟内存字节数(VIRTUAL_MEMORY_BYTES）               | 一个任务所用虚拟内存的字节数，由/proc/meminfo获取            |
| 有效的堆字节数(COMMITTED_HEAP_BYTES)                | 在JVM中的总有效内存最（以字节为单位），可由Runtime. getRuntime().totalMemory()获取 |
| GC运行时间毫秒数(GC_TIME_MILLIS)                    | 在任务执行过程中，垃圾收集器(garbage collection）花费的时间（以毫秒为单位），可由GarbageCollector MXBean. getCollectionTime()获取 |
| 由shuffle传输的map输出数(SHUFFLED_MAPS)             | 由shume传输到reducer的map输出文件数。                        |
| 失敗的shuffle数(FAILED_SHUFFLE)                     | shuffle过程中，发生map输出拷贝错误的次数                     |
| 被合并的map输出数（MERGED_MAP_OUTPUTS）             | shuffle过程中，在reduce端合并的map输出文件数                 |

##### 内置文件系统任务计数器

| 计数器名称                                 | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 文件系统的读字节数（BYTES_READ）           | 由map任务和reduce任务在各个文件系统中读取的字节数，各个文件系统分别对应一个计数器，文件系统可以是local、 HDFS、S3等 |
| 文件系统的写字节数(BYTES_WRITTEN）         | 由map任务和reduce任务在各个文件系统中写的字节数              |
| 文件系统读操作的数量(READ_OPS)             | 由map任务和reduce任务在各个文件系统中进行的读操作的数量（例如，open操作，filestatus操作） |
| 文件系统大规模读操作的数量(LARGE_READ_OPS) | 由map和reduce任务在各个文件系统中进行的大规模读操作（例如，对于一个大容量目录进行list操作）的数量 |
| 文件系统写操作的数量(WRITE_OPS)            | 由map任务和reduce任务在各个文件系统中进行的写操作的数量（例如，create操作，append操作） |

##### 内置的输入文件任务计数器

| 计数器名称               | 说明                                     |
| ------------------------ | ---------------------------------------- |
| 读取的字节数(BYTES_READ) | 由map任务通过FilelnputFormat读取的字节数 |

##### 内置输出文件任务计数器

| 计数器名称                | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 写的字节数(BYTES_WRITTEN) | 由map任务（针对仅含map的作业）或者reduce任务通过FileOutputFormat写的字节数 |

#### 作业计数器

##### 内置的作业计数器

| 计数器名称                                 | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 启用的map任务数（TOTAL_LAUNCHED_MAPS）     | 启动的map任务数，包括以“推测执行”方式启动的任务。            |
| 启用的reduce任务数(TOTAL_LAUNCHED_REDUCES) | 启动的reduce任务数，包括以“推测执行”方式启动的任务。         |
| 启用的uber任务数(TOTAL_LAIÆHED_UBERTASKS)  | 启用的uber任务数。                                           |
| uber任务中的map数(NUM_UBER_SUBMAPS)        | 在uber任务中的map数。                                        |
| Uber任务中的reduce数(NUM_UBER_SUBREDUCES)  | 在任务中的reduce数。                                         |
| 失败的map任务数（NUM_FAILED_MAPS）         | 失败的map任务数。                                            |
| 失败的reduce任务数(NUM_FAILED_REDUCES)     | 失败的reduce任务数                                           |
| 失败的uber任务数(NIN_FAILED_UBERTASKS)     | 失败的uber任务数。                                           |
| 被中止的map任务数（NUM_KILLED_MAPS）       | 被中止的map任务数。                                          |
| 被中止的reduce任务数(NW_KILLED_REDUCES)    | 被中止的reduce任务数。                                       |
| 数据本地化的map任务数（DATA_LOCAL_MAPS）   | 与输人数据在同一节点上的map任务数。                          |
| 机架本地化的map任务数（RACK_LOCAL_MAPS)    | 与输人数据在同一机架范围内但不在同一节点上的map任务数。      |
| 其他本地化的map任务数（OTHER_LOCAL_MAPS）  | 与输人数据不在同一机架范围内的map任务数。由于机架之间的带宽资源相对较少，Hadoop会尽量让map任务靠近输人数据执行，因此该计数器值一般比较小。 |
| map任务的总运行时间(MILLIS_MAPS)           | map任务的总运行时间，单位毫秒。包括以推测执行方式启动的任务。可参见相关的度量内核和内存使用的计数器(VCORES_MILLIS_MAPS和MB_MILLIS_MAPS） |
| reduce任务的总运行时间(MILLIS_REDUCES)     | reduce任务的总运行时间，单位毫秒。包括以推滌执行方式启动的任务。可参见相关的度量内核和内存使用的计数器(VQES_MILLIS_REARES和t*B_MILLIS_REUKES) |

#### 自定义计数器

虽然Hadoop内置的计数器比较全面，给作业运行过程的监控带了方便，但是对于那一些业务中的特定要求(统计过程中对某种情况发生进行计数统计)MapReduce还是提供了用户编写自定义计数器的方法。

##### 过程

1. 定义一个Java的枚举类型(enum)，用于记录计数器分组，其枚举类型的名称即为分组的名称，枚举类型的字段就是计数器名称。
2. 通过Context类的实例调用getCounter方法进行increment(long incr)方法，进行计数的添加。

##### 案例

###### ReportTest

```java
public enum ReportTest {                //定义枚举
    ErroWord, GoodWord, ReduceReport	//写入要记录的计数器名称
}
```

###### Mapper类

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class TxtMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        String[] words = value.toString().split(" ");

        for (String word : words) {
            if (word.equals("GoodWord")) {
                context.setStatus("GoodWord is coming");
                context.getCounter(ReportTest.GoodWord).increment(1);
            } else if (word.equals("ErroWord")) {
                context.setStatus("BadWord is coming!");
                context.getCounter(ReportTest.ErroWord).increment(1);
            } else {
                context.write(new Text(word), new IntWritable(1));
            }

        }
    }
}

```

###### Reducer类

```java

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.Iterator;

public class TxtReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;

        Iterator<IntWritable> it = values.iterator();
        while (it.hasNext()) {
            IntWritable value = it.next();
            sum += value.get();
        }
        if (key.toString().equals("hello")) {
            context.setStatus("BadKey is comming!");
            context.getCounter(ReportTest.ReduceReport).increment(1);
        }
        context.write(key, new IntWritable(sum));
    }
}
```

###### ToolRunnerJS

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counters;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;

public class ToolRunnerJS extends Configured implements Tool {
    public static void main(String[] args) throws Exception {
        ToolRunnerJS tool = new ToolRunnerJS();
        tool.run(null);
    }

    public int run(String[] args0) throws Exception {
        //Configuration:MapReduce的类,向Hadoop框架描述MapReduce执行工作
        Configuration conf = new Configuration();
        String output = "jishuqi1";

        Job job = Job.getInstance(conf);
        job.setJarByClass(ToolRunnerJS.class);
        job.setJobName("jishu");               //设置Job名称

        job.setOutputKeyClass(Text.class); //设置Job输出数据 K
        job.setOutputValueClass(IntWritable.class); //设置Job输出数据 V

        job.setMapperClass(TxtMapper.class);
        job.setReducerClass(TxtReducer.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.addInputPath(job, new Path("/input/counter/*")); //为 Job设置输入路径
        FileOutputFormat.setOutputPath(job, new Path("/output/counter_result")); //为Job设置输出路径

        job.waitForCompletion(true);
        Counters counters = job.getCounters();
        System.out.println("Counter getGroupNames:"+counters.getGroupNames());
        return 0;
    }
}
```

#### 查看

![FWk6ln.png](https://s1.ax1x.com/2018/12/28/FWk6ln.png)





通过Web界面也可以查看但是需要设置设置

```xml
 <property>
           <name>mapreduce.jobhistory.address</name>
	       <value>datanode1:10020</value>
           <description>MapReduce  JobHistory Server IPC host:port</description>
</property>

<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>datanode1:19888</value>
        <description>MapReduce JobHistory Server Web UI host:port</description>
</property>
```

启动服务

```shell
mr-jobhistory-daemon.sh start historyserver
```

web界面查看

![FWk5Y4.png](https://s1.ax1x.com/2018/12/28/FWk5Y4.png)

## 最值

最大值、最小值、平均值、均方差、众数、中位数等都是统计学中经典的数值统计，也是常用的统计属性字段，如果想知道最大的10个数，最小的10个数，这涉及到Top N/Bottom N 问题。

###  单一最值

常用的统计属性的字段在MapReduce的求解过程中，由一个大任务分解成若干个Mapper任务，最后会进行Reducer合并，比传统计算求解略显复杂，在MaoReduce框架中，会以Key进行分区、分组、排序的操作，在进行这些数值的操作时哦，只要设定合理的key，整个问题也就简单化了。使用Combiner可以减少Shuffle到Reduce端中间的K V的数目，减轻网络和IO的目的。

#### 求解最大值最小值

数据

```
2017-10 300
2017-10 100
2017-10 200
2017-11 320
2017-11 200
2017-11 280
2017-12 290
2017-12 270
```

#### MinMaxWritable

```java
import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class MinMaxWritable implements Writable {
    private int min;//记录最大值
    private int max;//记录最小值

    public int getMin() {
        return min;
    }

    public void setMin(int min) {
        this.min = min;
    }

    public int getMax() {
        return max;
    }

    @Override
    public String toString() {
        return min + "\t" + max;
    }

    public void setMax(int max) {
        this.max = max;
    }

    public void write(DataOutput out) throws IOException {
        out.writeInt(max);
        out.writeInt(min);
    }

    public void readFields(DataInput in) throws IOException {
        min = in.readInt();
        max = in.readInt();

    }
}
```

#### MinMaxMapper

```java
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class MinMaxMapper extends Mapper<Object, Text, Text, MinMaxWritable> {
    private MinMaxWritable outTuple = new MinMaxWritable();

    @Override
    protected void map(Object key, Text value, Context context) throws IOException, InterruptedException {

        String[] words = value.toString().split(" ");
        String data = words[0]; //定义记录的日期的自定义变量data
        if (data == null) {
            return;  //如果该日期为空，返回
        }
        outTuple.setMin(Integer.parseInt(words[1]));
        outTuple.setMax(Integer.parseInt(words[1]));
        context.write(new Text(data), outTuple);  //将结果写入到context
    }
}
```

#### MinMaxReducer

```java
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class MinMaxReducer extends Reducer<Text, MinMaxWritable, Text, MinMaxWritable> {
    private MinMaxWritable result = new MinMaxWritable();

    @Override
    protected void reduce(Text key, Iterable<MinMaxWritable> values, Context context) throws IOException, InterruptedException {
        result.setMax(0);
        result.setMin(0);
        //按照key迭代输出value的值
        for (MinMaxWritable value : values) {
            //最小值放入结果集
            if (result.getMin() == 0 || value.getMin() < result.getMin()) {
                result.setMin(value.getMin());
            }
            //最大值放入结果集
            if (result.getMax() == 0 || value.getMax() > result.getMax()) {
                result.setMax(value.getMax());
            }
        }
        context.write(key, result);
    }
}
```

#### MinMaxJob

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class MinMaxJob {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage:MinMaxMapper<in><out>");
            System.exit(2);
        }
        Job job = Job.getInstance(conf);
        job.setJarByClass(MinMaxJob.class);
        job.setMapperClass(MinMaxMapper.class);
        //启用Combiner 减少网络传输的数据量 
        job.setCombinerClass(MinMaxReducer.class);
        job.setReducerClass(MinMaxReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(MinMaxWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

#### 计算过程

![FfaTAK.png](https://s1.ax1x.com/2018/12/29/FfaTAK.png)

在一个MapReduce计算的过程中，Mapper任务相对于Reduce任务是大量的，因此少量的Reducer处理大量数据的并不明智，所以通过在Shuffle阶段引入Combiner，并把Reducer作为它的计算类，大大减少了Reducer端数据的输入，整个计算过程变得合理可靠。
