---
title: Flink时间概念和Watermark
date: 2020-09-20 09:45:48
tags: Flink
categories: 大数据
---



在流式数据处理中，数据具有时间的属性特征，Flink根据时间产生的位置不同，时间可区分为：事件生成事件(Event Time)、事件摄取事件(Ingestion Time)、事件处理事件(Processing Time)。

## 时间概念

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200920104428.png)

- 事件时间（Event Time）是数据产生时本身携带时间，这个时间在数据进入Flink中的DataSource就一定确定，Flink可以进入DataSource的数据中提取到事件的事件时间戳，在Event Time中，时间是取决于数据本身和处理数据的系统时间无关，Event Time程序必须指定如何生成*事件时间水印*，这是表示Event Time进度的机制。

  在最理想的情况下，无论时间何时到达还是乱序传入，最后处理时间时间时都应该产生一个完全一致和确定的结果，例如，每小时Event Time窗口将包含带有落入该小时的事件Event Time的所有记录，无论它们到达的顺序如何，或者何时处理它们都应该是一个确定的结果。除非事件已知按顺序到达（按时间戳），否则事件时间处理会在等待无序事件时产生一些延迟。由于只能等待一段有限的时间，因此限制了确定性事件时间应用程序的可能性。

- 接入时间(Ingestion Time) 是数据进入Flink的系统时间，依赖于主机系统的系统时钟，每个事件将进入 Flink 中的DataSource时当时的时间作为时间戳。Ingestion Time 在概念上位于 Event Time 和 Processing Time 之间。 与 Processing Time 相比，Ingestion Time生成的代价相对较高，但是结果更为容易推测。 Ingestion Time 使用稳定的时间戳在数据进Flink的DataSource时就已分配，后续的处理与程序所在的机器时钟无关，所以对事件的不同窗口操作将使用的时间戳是固定的即为当数据进入DataSource时分配的，而在 Processing Time 中，每个窗口操作符可以将事件分配给不同的窗口，这些窗口是基于机器系统时间和到达延迟的。

  与 Event Time 相比，Ingestion Time 程序无法处理任何无序事件或延迟数据，这是因为Flink Ingestion Time 是在进入Flink就已经分配好的，程序中不必指定如何生成水印。

- 处理时间 （Processing Time） 数据操作的过程中都是基于当时机器的系统时间，每个小时Processing Time窗口将包含系统时钟制定的整个小时之间满足特定的操作的所有事件数据。Processing Time 是最简单的 "Time" 概念，不需要流和机器之间的协调，它提供了最好的性能和最低的延迟。但是这个时间存在一定的不确定性，比如消息到达处理节点延迟等影响。

简而言之：三种时间的含义如表格和图所示

| 时间类型                    | 含义                     |
| --------------------------- | ------------------------ |
| 事件时间（Event Time）      | 事件实际发生的时间       |
| 接入时间（Ingestion Time）  | 事件进入流处理框架的时间 |
| 处理时间（Processing Time） | 事件被处理的时间         |

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200920130927.png)

## 时间概念制定

在默认的情况下,Flink使用的是Processing Time，用户可以在创建StreamExecutionEnvironment后调用setStreamTimeCharacteristic设定相关的事件概念。

```java
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
		//设置为事件时间
        environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
		//设置为接入时间
        environment.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime);
		//设置为处理时间
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
```

## 水印(水位线)

通常情况下由于网络或者系统等外部因素的影响，事件数据不能及时传入Flink中，数据会乱序到达或者延迟到达。WaterMark的出现是为了解决实时计算中的数据乱序问题，在正常的中文翻译中WaterMark是水位，但是在 Flink 框架中，翻译为“水位线”更为合理，它在本质上是一个时间戳。其实现主要参考了谷歌的《The dataflow model: a practical approach to balancing correctness, latency, and cost in massive-scale, unbounded, out-of-order data processing》

{% pdf https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/p1792-Akidau.pdf %}

简单来说，Flink的水印是一种控制数据处理过程和进度的措施，比如在基于事件时间的Window创建后，如何确定数据该Window的数据元素全部到达，如果全部到达则可以对Window里的所有数据进行操作，如果数据没有全部到达，则继续等待，需要等待所有的数据全部到达后才能进行处理。Flink会读取进入系统的最新事件时间减去固定的时间间隔作为WaterMark，该事件间隔为用户配置的支持最大延迟到达的时间长度，理论上不会有事件超过该间隔到达，否则就会被认为为延迟事件或异常事件。

简单来说，当事件进入Flink中，会在Source Operator中根据当前最新事件产生Watermarks时间戳，记做X，进入Flink系统中的数据事件时间，记做Y，如果Y<X则说明 Watermark X时间戳之前的所有事件均已到达，同时满足Window中的EndTime大于WaterMark，则触发窗口计算并且输出。换句话说触发Window的计算，需要保证Window内的所有数据元素满足其事件时间 Y>=X，否则窗口会继续等待Watermark大于窗口结束时间的条件满足。有了Watermark机制，可以有效处理乱序时间的问题，保证流式结果的正确性。

### 顺序事件中的Watermarks

如果数据元素的事件时间是有序的，Watermark的时间戳会随着数据元素的事件时间按照顺序生成，水位线的变化和事件时间保持一致，在WaterMark时间大于Window结束事件就会触发Window计算，并创建新的Window，并把事件时间元素分配到新的Window中。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200920192729.png)

在顺序实践中Watermarks只是对Stream数据做了简单的周期标记,Watermark的价值不大，会出现因为超期时间导致延迟输出计算结果。不过现实的情况是往往数据是乱序的。

### 乱序时间中的Watermarks

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200920193225.png)

现实情况往往是这种数据的乱序。在Flink处理数据中会根据设定的延迟时间分别计算出Watermark(11)和Watermark(17),这两个Watermark到达一个Operator中后，会马上调整算子基于事件时间的时钟，与当前的Watermark的值做匹配，然后出发相应的计算。

### 并行数据流中的Watermarks

Watermark会在Source Operator中生成，并且在每个Source Operator中的子Task独立生成Watermark，子任务生成后，更新该Task的Watermark，并且会逐步更新下游算子中的Watermark水位线，随后一致保持在该并发之中,直到下一次Watermarks的生成，并对之前的Watermarks覆盖。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200920194349.png)

W(17)水位线已经将Source算子和Map算子的子任务时钟时间全部更新为17，并且一致会随着时间向后推移更新下游算子中的事件时间，多个Watermarkl同是更新当前事件时间，Flink会选择小的水位线来更新，当Window算子Task中水位线大于了Window结束事件，就会立即出发窗口计算。

### 指定 Timestamps和生成Watermarks

如果使用了EvenTime时间处理数据，除了在setStreamTimeCharacteristic指定TimeCharacteristic为EventTime外，还需要在Flink程序中制定EventTime时间戳在数据中心的信息，在Flink程序运行时会通过制定字段抽取对应的事件时间，这个过程叫 Timestamps  Assigning,这是告诉Flink那个字段是时间，指定完Timestamps  后，要创建相应的Watermarks，用户可以Timestamps  计算出Watermarks的生成策略。目前有两种方式定义。

#### Source Function中自定义

在数据进入Flink中就直接分配EventTime和Watermark，用户复写SourceFunction接口中的run方法实现数据生成逻辑，同时需要调用collectWithTimestamp生成Eventime的时间戳，调用emitWatermark生成WaterMark。代码如下：

```java
package com.hph.time;

import com.google.gson.Gson;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.streaming.api.watermark.Watermark;


import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Random;

public class MyWaterMarkAndTimestamps {
    private static transient KafkaConsumer<String, String> consumer;

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);


        DataStreamSource<String> dataStreamSource = environment.addSource(new RichSourceFunction<String>() {
            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);

            }

            boolean isRunning = true;

            @Override
            public void run(SourceContext<String> ctx) throws Exception {
                //随机产生数据为 A-Z附带时间戳
                while (isRunning) {
                    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
                    Thread.sleep(1000);
                    char i = (char) (65 + new Random().nextInt(25));
                    long timeMills = (System.currentTimeMillis());
                    //格式化数据为A_Z的字符 + 时间
                    String element = i + "_" + simpleDateFormat.format(new Date(timeMills));
                    //设置时间戳
                    ctx.collectWithTimestamp(element, timeMills);
                    //水印时间=时间戳+1S
                    ctx.emitWatermark(new Watermark(timeMills + 1000));
                }
            }

            @Override
            public void cancel() {
                isRunning = false;
            }
        });
        dataStreamSource.print();


        //将数据转化为 Tuple2 之后对数据进行分组，同是创建一个窗口 每5秒滑动一次
        dataStreamSource.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                String word = value.split("_")[0];
                return Tuple2.of(word, 1);
            }
        }).keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        }).timeWindow(Time.seconds(5)).sum(1).print();

        environment.execute("MyWaterMarkAndTimestamps");
    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200921223344.gif)

从计算结果来看数据每秒产生一次，并且在Flink中Window的窗口为每5s划分一个，因此当水印时间小于或等于这批时间范围的最大值时即触发计算。如何理解水印呢，以生活中的例子为例，我们上班的事件为9:00，但是公司比较人性化，你可以稍微晚来一些，这个都不算做迟到，因此我们可以把这个水印理解成为打卡上班，而水印的值即为出发窗口计算的时间点。

#### Flink自带的TimeAssigner制指定

在用户定义好了外部的书卷连接器后，就不能在实现Source Function接口制定Event Time和 Watermark了，这时候就需要使用TimeAssigner，TimeAssigner一般在DataSource算子后面指定，也可以在后面的算子中指定，但是需要保证TimeAssigner在第一个时间相关的Operatpr之前即可，TimeAssigner可以覆盖用户在SourceFunction定义的逻辑。

Flink自带的TimeAssigner分为两种：

1. 基于时间间隔周期的Periodic Watermarks。在Flink中Periodic Watermarks可分为升序模式，提取数据中的特定字段为Timestamp，并且使用当前时间为Watermark，比较适合于事件多为顺序生成的，另一种是通过设置固定时间间隔指定WaterMark落后于Timestamp的区间长度，也就是最大容忍迟到多长时间内的数据到达系统。
2. 基于数量生成的Punctuated Watermarks。

在在flink 1.11之前的版本中，提供了两种生成水印（Watermark）的策略，分别是AssignerWithPunctuatedWatermarks和AssignerWithPeriodicWatermarks，这两个接口都继承自TimestampAssigner接口。用户想使用不同的水印生成方式，则需要实现不同的接口，但是这样引发了一个问题，对于想给水印添加一些通用的、公共的功能则变得复杂，因为我们需要给这两个接口都同时添加新的功能，这样还造成了代码的重复。在Flink1.11中对水印和时间提取的方式进行了重构,可以通过实现WatermarkGenerator接口解析EventTime的时间戳和设置水印。

```java
package com.hph.time;


import org.apache.flink.api.common.eventtime.*;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.text.SimpleDateFormat;

import java.util.Date;
import java.util.Random;

public class MyWaterMarkAndTimestampsFlink11 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);


        DataStreamSource<String> dataStreamSource = environment.addSource(new RichSourceFunction<String>() {
            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);

            }


            boolean isRunning = true;

            @Override
            public void run(SourceContext<String> ctx) throws Exception {
                //随机产生数据为 A-Z附带时间戳
                while (isRunning) {

                    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
                    Thread.sleep(1000);
                    char i = (char) (65 + new Random().nextInt(25));
                    long timeMills = (System.currentTimeMillis());
                    //格式化数据为A_Z的字符 + 时间
                    String element = i + "_" + timeMills;
                    ctx.collect(element);

                }
            }

            @Override
            public void cancel() {
                isRunning = false;
            }
        });


        SingleOutputStreamOperator<String> map = dataStreamSource.map(new MapFunction<String, String>() {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");

            @Override
            public String map(String value) throws Exception {
                String[] data = value.split("_");
                String time = simpleDateFormat.format(new Date(Long.valueOf(data[1])));
                return data[0] + "_" + time;
            }
        });

        //打印处理后的数据方便查看
        map.print();


        //提取数据中的时间戳和设置水位线
        SingleOutputStreamOperator<String> watermarks = dataStreamSource.assignTimestampsAndWatermarks(new WatermarkStrategy<String>() {

            @Override
            public WatermarkGenerator<String> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
                return new WatermarkGenerator<String>() {
                    @Override
                    public void onEvent(String event, long eventTimestamp, WatermarkOutput output) {
                        String[] data = event.split("_");
                        String word = data[0];
                        String timeStamp = data[1];
                        //设置为提前1s中触发
                        output.emitWatermark(new Watermark(Long.valueOf(timeStamp) - 1000));
                    }

                    @Override
                    public void onPeriodicEmit(WatermarkOutput output) {

                    }
                };

            }

            //提取时间戳
            @Override
            public TimestampAssigner<String> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
                return new TimestampAssigner<String>() {
                    @Override
                    public long extractTimestamp(String element, long recordTimestamp) {
                        String[] data = element.split("_");
                        Long timeStamp = Long.valueOf(data[1]);
                        return timeStamp;
                    }
                };
            }
        });


        //分组  窗口每5s生成一次
        watermarks.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                String[] data = value.split("_");
                return Tuple2.of(data[0], 1);
            }
        }).keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        }).timeWindow(Time.seconds(5)).sum(1).print();

        environment.execute("MyWaterMarkAndTimestampsFlink11");


    }


}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20200922235741.gif)

可以看出每5s窗口会重新计算一次数据。

在上代码中：

- onEvent ：每个元素都会调用这个方法，如果我们想依赖每个元素生成一个水印，然后发射到下游(可选，就是看是否用output来收集水印)，我们可以实现这个方法.
- onPeriodicEmit : 如果数据量比较大的时候，我们每条数据都生成一个水印的话，会影响性能，所以这里还有一个周期性生成水印的方法。这个水印的生成周期可以这样设置：environment.getConfig().setAutoWatermarkInterval(5000L);

通过对Flink事件概念的了解和水位线的了解我想你应该明白了Flink的设计精妙之处和Google 论文的奇思妙想。