---
title: Flink-CEP
date: 2021-11-29 20:24:24
tags: Flink
categories: 大数据
---
复杂事件处理(CEP)是一种基于流处理，将系统数据看作不同类型事件，通过分析事件之间的联系，简历不同的事件关系系列库，并利用过滤，关联、聚合等技术，最终由简单事件产生高级事件，通过规则模式的方式对重要信息进行追踪分析，从实时数据中发掘有价值的信息。

应用领域：

- 防范网络诈骗
- 设备故障检测
- 风险规避合和只能营销领域。

主流工具

- Esper
- Jboss 
- Drools
- MicroSoft StreamInsight（商用）

## 原理

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220209225029.png)

Flink CEP内部是用**NFA（非确定有限自动机）**来实现的，由点和边组成的一个状态图，以一个初始状态作为起点，经过一系列的中间状态，达到终态。点分为**起始状态**、**中间状态**、**最终状态**三种，边分为**take**、**ignore**、**proceed**三种。

- **take**：必须存在一个条件判断，当到来的消息满足take边条件判断时，把这个消息放入结果集，将状态转移到下一状态。
- **ignore**：当消息到来时，可以忽略这个消息，将状态自旋在当前不变，是一个自己到自己的状态转移。 
- **proceed**：又叫做状态的空转移，当前状态可以不依赖于消息到来而直接转移到下一状态。举个例子，当用户购买商品时，如果购买前有一个咨询客服的行为，需要把咨询客服行为和购买行为两个消息一起放到结果集中向下游输出；如果购买前没有咨询客服的行为，只需把购买行为放到结果集中向下游输出就可以了。 也就是说，如果有咨询客服的行为，就存在咨询客服状态的上的消息保存，如果没有咨询客服的行为，就不存在咨询客服状态的上的消息保存，咨询客服状态是由一条proceed边和下游的购买状态相连。

## 事件定义

### 简单事件

简单事件存在生活中，主要是处理单一事件，事件结果可直接观察出来活计算出来，比如用户当天的交易总额只需按照用户维度聚合求和即可。

### 复杂事件

复杂事件由多个简单事件组合而成，当特定的事件发生时就会出发某一个动作。

## 事件关系

### 时序关系

动作事件和动作事件之间，动作事件和状态变化事件之间，存在时间顺序，事件和时间的时序关系决定大部分的时序规则，如：A事件状态为1时，B事件的状态为0。

### 聚合关系

动作事件和动作事件之间，动作事件和状态变化事件之间，存在聚合关系，个体聚合形成整体，如：A事件的状态为1时为触发10次告警。

### 层次关系

动作事件和动作事件之间，动作事件和状态变化事件之间，存在层次关系，父类事件和子类事件存在层级关系，从父类到子类是具体化的，从子类到父类是泛化的。

### 依赖关系

动作事件和动作事件之间，动作事件和状态变化事件之间，存在依赖关系和约束关系，A事件发生的前提是B事件触发，则A与B之间存在依赖关系。

### 因果关系

对于完整的动作过程，结果状态为果，初始状态和动作都可以视为原因，如果A事件状态改变导致B事件的触发，则A事件为因,B事件为果。

## 事件处理

### 事件推断

利用事物状态的约束关系，从一部分状态属性值可以推断出另一个状态属性值，比如三角形两个内角为 90°，60° 可以推断出第三个角为30°。

### 事件查因

当结果状态出现时，并且知道初始状态，可以分析得出某个动作是原因。

### 事件决策

想要得到某个结果，知道初始状态，执行什么动作，某个规则符合条件触发后，执行报警等操作。

### 事件预测

知道事情的初始状态，以及想要做的动作，预测未发生的结果状态，比如气象局根据气象数据预测天气状况。

## Pattern API

Flink CEP 提供了 Pattern API 对输入流数据的复杂规则定义，从事件流中抽取事件结果,通过使用Pattern API 构建CEP应用程序，其中包括输入流的创建，Pattern接口定义，通过CEP pattern方法将定义的Pattern应用在输入的Stream上，最后使用PatternStream.select触发事件结果。

每个复杂的模式序列包括多个简单的模式，比如，寻找拥有相同属性事件序列的模式。从现在开始，我们把这些简单的模式称作**模式**， 把我们在数据流中最终寻找的复杂模式序列称作**模式序列**，你可以把模式序列看作是这样的模式构成的图， 这些模式基于用户指定的**条件**从一个转换到另外一个，比如 `event.getName().equals("end")`。 一个**匹配**是输入事件的一个序列，这些事件通过一系列有效的模式转换，能够访问到复杂模式图中的所有模式。

<font color=red>每个模式必须有一个独一无二的名字，你可以在后面使用它来识别匹配到的事件。</font>

<font color=red>模式的名字不能包含字符`":"`.</font>

### 单个模式

一个模式可以是一个单例或者循环模式。单例模式只接受一个事件，循环模式可以接受多个事件。 在模式匹配表达式中，模式"a b+ c? d"（或者"a"，后面跟着一个或者多个"b"，再往后可选择的跟着一个"c"，最后跟着一个"d"）， a，c?，和 d都是单例模式，b+是一个循环模式。默认情况下，模式都是单例的，你可以通过使用量词把它们转换成循环模式。 每个模式可以有一个或者多个条件来决定它接受哪些事件。

### 量词 

在FlinkCEP中，你可以通过这些方法指定循环模式：`pattern.oneOrMore()`，指定期望一个给定事件出现一次或者多次的模式（例如前面提到的`b+`模式）； `pattern.times(#ofTimes)`，指定期望一个给定事件出现特定次数的模式，例如出现4次`a`； `pattern.times(#fromTimes, #toTimes)`，指定期望一个给定事件出现次数在一个最小值和最大值中间的模式，比如出现2-4次`a`。

你可以使用`pattern.greedy()`方法让循环模式变成贪心的，但现在还不能让模式组贪心。 你可以使用`pattern.optional()`方法让所有的模式变成可选的，不管是否是循环模式。

对一个命名为`start`的模式，以下量词是有效的：

```java
// 期望出现4次
start.times(4);

// 期望出现0或者4次
start.times(4).optional();

// 期望出现2、3或者4次
start.times(2, 4);

// 期望出现2、3或者4次，并且尽可能的重复次数多
start.times(2, 4).greedy();

// 期望出现0、2、3或者4次
start.times(2, 4).optional();

// 期望出现0、2、3或者4次，并且尽可能的重复次数多
start.times(2, 4).optional().greedy();

// 期望出现1到多次
start.oneOrMore();

// 期望出现1到多次，并且尽可能的重复次数多
start.oneOrMore().greedy();

// 期望出现0到多次
start.oneOrMore().optional();

// 期望出现0到多次，并且尽可能的重复次数多
start.oneOrMore().optional().greedy();

// 期望出现2到多次
start.timesOrMore(2);

// 期望出现2到多次，并且尽可能的重复次数多
start.timesOrMore(2).greedy();

// 期望出现0、2或多次
start.timesOrMore(2).optional();

// 期望出现0、2或多次，并且尽可能的重复次数多
start.timesOrMore(2).optional().greedy();
```

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class IndividualPatterns {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        //创建数据流
        KeyedStream<OrderEvent, String> input = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("1", "create", 3000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("3", "create", 6003L),
                        new OrderEvent("4", "create", 6003L),
                        new OrderEvent("4", "pay", 6003L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                ).keyBy(x -> x.orderId);

        // 规则 create 出现2次得
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent) throws Exception {
                        return orderEvent.eventType.equals("create");
                    }
                    //次数
                }).within(Time.seconds(5)).times(2);
        
        PatternStream<OrderEvent> patternedStream = CEP.pattern(input, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("create");
                collector.collect("条件获取到得订单ID:" + order.iterator().next().orderId + ":" + order.iterator().next().eventType);

            }
        }).print();


        env.execute();
    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220219184907.png)

### 组合模式

组合模式得序列由初始模式作为开头，如下所示：

```java
Pattern<Event, ?> start = Pattern.<Event>begin("start");
```

FlinkCEP支持事件之间如下形式的连续策略:

| 策略                 | 含义                                                         | 关键字          |
| -------------------- | ------------------------------------------------------------ | --------------- |
| **严格连续**         | 期望所有匹配的事件严格的一个接一个出现，中间没有任何不匹配的事件。 | next()          |
| **松散连续**         | 忽略匹配的事件之间的不匹配的事件。                           | followedBy()    |
| **不确定的松散连续** | 更进一步的松散连续，允许忽略掉一些匹配事件的附加匹配。       | followedByAny() |
|                      | 如果不想后面直接连着一个特定事件                             | notNext()       |
|                      | 如果不想一个特定事件发生在两个事件之间的任何地方。           | notFollowedBy() |

<font color=red>不能以`notFollowedBy()`结尾。</font>

<font color=red>一个`NOT`模式前面不能是可选的模式。</font>

```java
// 严格连续
Pattern<Event, ?> strict = start.next("middle").where(...);

// 松散连续
Pattern<Event, ?> relaxed = start.followedBy("middle").where(...);

// 不确定的松散连续
Pattern<Event, ?> nonDetermin = start.followedByAny("middle").where(...);

// 严格连续的NOT模式
Pattern<Event, ?> strictNot = start.notNext("not").where(...);

// 松散连续的NOT模式
Pattern<Event, ?> relaxedNot = start.notFollowedBy("not").where(...);
```

松散连续意味着跟着的事件中，只有第一个可匹配的事件会被匹配上，而不确定的松散连接情况下，有着同样起始的多个匹配会被输出。 举例来说，模式"a b"，给定事件序列"a"，"c"，"b1"，"b2"，会产生如下的结果：

"a"和"b"之间严格连续： {} （没有匹配），"a"之后的"c"导致"a"被丢弃。

"a"和"b"之间松散连续： {a b1}，松散连续会"跳过不匹配的事件直到匹配上的事件"。

"a"和"b"之间不确定的松散连续： {a b1}, {a b2}，这是最常见的情况。

也可以为模式定义一个有效时间约束。 例如，你可以通过pattern.within()方法指定一个模式应该在10秒内发生。 这种时间模式支持处理时间和事件时间.

<font color=red>  一个模式序列只能有一个时间限制。如果限制了多个时间在不同的单个模式上，会使用最小的那个时间限制。</font >

```java
next.within(Time.seconds(10));
```

 连续性会被运用在被接受进入模式的事件之间。 用这个例子来说明上面所说的连续性，一个模式序列`"a b+ c"`（`"a"`后面跟着一个或者多个（不确定连续的）`"b"`，然后跟着一个`"c"`） 输入为`"a"，"b1"，"d1"，"b2"，"d2"，"b3"，"c"`，输出结果如下：

1. **严格连续**: `{a b3 c}` – `"b1"`之后的`"d1"`导致`"b1"`被丢弃，同样`"b2"`因为`"d2"`被丢弃。
2. **松散连续**: `{a b1 c}`，`{a b1 b2 c}`，`{a b1 b2 b3 c}`，`{a b2 c}`，`{a b2 b3 c}`，`{a b3 c}` - `"d"`都被忽略了。
3. **不确定松散连续**: `{a b1 c}`，`{a b1 b2 c}`，`{a b1 b3 c}`，`{a b1 b2 b3 c}`，`{a b2 c}`，`{a b2 b3 c}`，`{a b3 c}` - 注意`{a b1 b3 c}`，这是因为`"b"`之间是不确定松散连续产生的。

对于循环模式（例如`oneOrMore()`和`times()`)），默认是*松散连续*。如果想使用*严格连续*，你需要使用`consecutive()`方法明确指定， 如果想使用*不确定松散连续*，你可以使用`allowCombinations()`方法。

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class StrictConsecutive {
    public static void main(String[] args) throws Exception {


        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);


        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "C", 1L),
                        new OrderEvent("1", "D", 2L),
                        new OrderEvent("1", "A", 3L),
                        new OrderEvent("1", "A", 4L),
                        new OrderEvent("1", "A", 5L),
                        new OrderEvent("1", "D", 6L),
                        new OrderEvent("1", "B", 7L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);

        //C D A1 A2 A3 D A4 B

        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("start").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("c");
                    }
                })
                .followedBy("middle").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("a");
                    }
                }).oneOrMore().consecutive()
                .followedBy("end1").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("b");
                    }
                }).within(Time.seconds(5));


        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);
        patternedStream.inProcessingTime().process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                collector.collect(map);
            }
        }).print();

        env.execute("StrictConsecutive");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220220085606.png)

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class AllowCombinationsConsecutive {
    public static void main(String[] args) throws Exception {


        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);


        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "C", 1L),
                        new OrderEvent("1", "D", 2L),
                        new OrderEvent("1", "A", 3L),
                        new OrderEvent("1", "A", 4L),
                        new OrderEvent("1", "A", 5L),
                        new OrderEvent("1", "D", 6L),
                        new OrderEvent("1", "B", 7L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);

        //C D A1 A2 A3 D A4 B

        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("start").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("c");
                    }
                })
                .followedBy("middle").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("a");
                    }
                }).oneOrMore().allowCombinations()
                .followedBy("end1").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("b");
                    }
                }).within(Time.seconds(5));


        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);
        patternedStream.inProcessingTime().process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                collector.collect(map);
            }
        }).print();

        env.execute("StrictConsecutive");

    }
}
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220220091353.png)

### 模式组

也可以定义一个模式序列作为`begin`，`followedBy`，`followedByAny`和`next`的条件。这个模式序列在逻辑上会被当作匹配的条件， 并且返回一个`GroupPattern`，可以在`GroupPattern`上使用`oneOrMore()`，`times(#ofTimes)`， `times(#fromTimes, #toTimes)`，`optional()`，`consecutive()`，`allowCombinations()`。

| 模式操作                             |                             描述                             |
| :----------------------------------- | :----------------------------------------------------------: |
| **begin(#name)**                     | 定义一个开始的模式：```java Pattern start = Pattern.begin("start"); ``` |
| **begin(#pattern_sequence)**         | 定义一个开始的模式：```java Pattern start = Pattern.begin( Pattern.begin("start").where(...).followedBy("middle").where(...) ); ``` |
| **next(#name)**                      | 增加一个新的模式。匹配的事件必须是直接跟在前面匹配到的事件后面（严格连续）：```java Pattern next = start.next("middle"); ``` |
| **next(#pattern_sequence)**          | 增加一个新的模式。匹配的事件序列必须是直接跟在前面匹配到的事件后面（严格连续）：```java Pattern next = start.next( Pattern.begin("start").where(...).followedBy("middle").where(...) ); ``` |
| **followedBy(#name)**                | 增加一个新的模式。可以有其他事件出现在匹配的事件和之前匹配到的事件中间（松散连续）：```java Pattern followedBy = start.followedBy("middle"); ``` |
| **followedBy(#pattern_sequence)**    | 增加一个新的模式。可以有其他事件出现在匹配的事件序列和之前匹配到的事件中间（松散连续）：```java Pattern followedBy = start.followedBy( Pattern.begin("start").where(...).followedBy("middle").where(...) ); ``` |
| **followedByAny(#name)**             | 增加一个新的模式。可以有其他事件出现在匹配的事件和之前匹配到的事件中间， 每个可选的匹配事件都会作为可选的匹配结果输出（不确定的松散连续）：```java Pattern followedByAny = start.followedByAny("middle"); ``` |
| **followedByAny(#pattern_sequence)** | 增加一个新的模式。可以有其他事件出现在匹配的事件序列和之前匹配到的事件中间， 每个可选的匹配事件序列都会作为可选的匹配结果输出（不确定的松散连续）：```java Pattern followedByAny = start.followedByAny( Pattern.begin("start").where(...).followedBy("middle").where(...) ); ``` |
| **notNext()**                        | 增加一个新的否定模式。匹配的（否定）事件必须直接跟在前面匹配到的事件之后（严格连续）来丢弃这些部分匹配：```java Pattern notNext = start.notNext("not"); ``` |
| **notFollowedBy()**                  | 增加一个新的否定模式。即使有其他事件在匹配的（否定）事件和之前匹配的事件之间发生， 部分匹配的事件序列也会被丢弃（松散连续）：```java Pattern notFollowedBy = start.notFollowedBy("not"); ``` |
| **within(time)**                     | 定义匹配模式的事件序列出现的最大时间间隔。如果未完成的事件序列超过了这个事件，就会被丢弃：```java pattern.within(Time.seconds(10)); ``` |

```java
Pattern<Event, ?> start = Pattern.begin(
    Pattern.<Event>begin("start").where(...).followedBy("start_middle").where(...)
);

// 严格连续
Pattern<Event, ?> strict = start.next(
    Pattern.<Event>begin("next_start").where(...).followedBy("next_middle").where(...)
).times(3);

// 松散连续
Pattern<Event, ?> relaxed = start.followedBy(
    Pattern.<Event>begin("followedby_start").where(...).followedBy("followedby_middle").where(...)
).oneOrMore();

// 不确定松散连续
Pattern<Event, ?> nonDetermin = start.followedByAny(
    Pattern.<Event>begin("followedbyany_start").where(...).followedBy("followedbyany_middle").where(...)
).optional();

```

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class OrderCEPDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);
        // 5s 创建订单并下单
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.eventType.equals("create");
                    }
                })
                .next("pay")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.eventType.equals("pay");
                    }
                })
                .within(Time.seconds(5));

        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("pay");
                collector.collect("正常支付的订单号为:" + order.iterator().next().orderId);

            }
        }).print();


        env.execute();
    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220207174926.png)

正常支付的订单为1 和2 

## 条件

对每个模式你可以指定一个条件来决定一个进来的事件是否被接受进入这个模式。

### 迭代条件

这是最普遍的条件类型。使用它可以指定一个基于前面已经被接受的事件的属性或者它们的一个子集的统计数据来决定是否接受时间序列的条件。能够对模式之前所有接收的事件进行处理。

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.IterativeCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class OrderIteratrCEPDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);
        // 规则 先create 后 pay
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        if (Integer.valueOf(orderEvent.orderId) < 100) {
                            return true;
                        }
                        return false;
                    }
                }).next("pay").where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        if (orderEvent.eventType.equals("pay") && Integer.valueOf(orderEvent.orderId) >= 2) {
                            return true;
                        }
                        return false;
                    }
                }).within(Time.seconds(5));

        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("pay");
                collector.collect("正常支付的订单号为:" + order.iterator().next().orderId);

            }
        }).print();


        env.execute();
    }
}

```

### 简单条件

这种类型的条件扩展了前面提到的`IterativeCondition`类，它决定是否接受一个事件只取决于事件自身的属性。

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class OrderSimpleCEPDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);
        // 简单条件
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.eventType.equals("create");
                    }
                })
                .next("pay")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.eventType.equals("pay");
                    }
                })
                .within(Time.seconds(5));

        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("pay");
                collector.collect("正常支付的订单号为:" + order.iterator().next().orderId);

            }
        }).print();

        env.execute();
    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220219170537.png)

###  停止条件

如果使用循环模式（`oneOrMore()`和`oneOrMore().optional()`），你可以指定一个停止条件，例如，接受事件的值大于5直到值的和小于50。

为了更好的理解它，看下面的例子。给定

- 模式如`"(a+ until b)"` (一个或者更多的`"a"`直到`"b"`)
- 到来的事件序列`"a1" "c" "a2" "b" "a3"`
- 输出结果会是： `{a1 a2} {a1} {a2} {a3}`.

你可以看到`{a1 a2 a3}`和`{a2 a3}`由于停止条件没有被输出。

| 模式操作                        |                             描述                             |
| :------------------------------ | :----------------------------------------------------------: |
| **where(condition)**            | 为当前模式定义一个条件。为了匹配这个模式，一个事件必须满足某些条件。 多个连续的where()语句取与组成判断条件：```java pattern.where(new IterativeCondition() { @Override public boolean filter(Event value, Context ctx) throws Exception { return ... // 一些判断条件 } }); ``` |
| **or(condition)**               | 增加一个新的判断，和当前的判断取或。一个事件只要满足至少一个判断条件就匹配到模式：```java pattern.where(new IterativeCondition() { @Override public boolean filter(Event value, Context ctx) throws Exception { return ... // 一些判断条件 } }).or(new IterativeCondition() { @Override public boolean filter(Event value, Context ctx) throws Exception { return ... // 替代条件 } }); ``` |
| **until(condition)**            | 为循环模式指定一个停止条件。意思是满足了给定的条件的事件出现后，就不会再有事件被接受进入模式了。只适用于和`oneOrMore()`同时使用。**NOTE:** 在基于事件的条件中，它可用于清理对应模式的状态。```java pattern.oneOrMore().until(new IterativeCondition() { @Override public boolean filter(Event value, Context ctx) throws Exception { return ... // 替代条件 } }); ``` |
| **subtype(subClass)**           | 为当前模式定义一个子类型条件。一个事件只有是这个子类型的时候才能匹配到模式：```java pattern.subtype(SubEvent.class); ``` |
| **oneOrMore()**                 | 指定模式期望匹配到的事件至少出现一次。.默认（在子事件间）使用松散的内部连续性。 关于内部连续性的更多信息可以参考[连续性](https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/libs/cep/#consecutive_java)。**NOTE:** 推荐使用`until()`或者`within()`来清理状态。```java pattern.oneOrMore(); ``` |
| **timesOrMore(#times)**         | 指定模式期望匹配到的事件至少出现**#times**次。.默认（在子事件间）使用松散的内部连续性。 关于内部连续性的更多信息可以参考[连续性](https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/libs/cep/#consecutive_java)。```java pattern.timesOrMore(2); ``` |
| **times(#ofTimes)**             | 指定模式期望匹配到的事件正好出现的次数。默认（在子事件间）使用松散的内部连续性。 关于内部连续性的更多信息可以参考[连续性](https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/libs/cep/#consecutive_java)。```java pattern.times(2); ``` |
| **times(#fromTimes, #toTimes)** | 指定模式期望匹配到的事件出现次数在**#fromTimes**和**#toTimes**之间。默认（在子事件间）使用松散的内部连续性。 关于内部连续性的更多信息可以参考[连续性](https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/libs/cep/#consecutive_java)。```java pattern.times(2, 4); ``` |
| **optional()**                  | 指定这个模式是可选的，也就是说，它可能根本不出现。这对所有之前提到的量词都适用。```java pattern.oneOrMore().optional(); ``` |
| **greedy()**                    | 指定这个模式是贪心的，也就是说，它会重复尽可能多的次数。这只对量词适用，现在还不支持模式组。```java pattern.oneOrMore().greedy(); ``` |

### 组合条件

你可以把`subtype`条件和其他的条件结合起来使用。这适用于任何条件，你可以通过依次调用`where()`来组合条件。 最终的结果是每个单一条件的结果的逻辑**AND**。如果想使用**OR**来组合条件，你可以像下面这样使用`or()`方法

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.IterativeCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class OrderCombinCEPDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L),
                        new OrderEvent("4", "cancel", 4003L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);


        // 规则 先create 后 pay
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .or(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {

                        return !orderEvent.eventType.equals("");
                    }
                }).or(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        return (Integer.valueOf(orderEvent.orderId) ==3);
                    }
                }).within(Time.seconds(5));

        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("create");
                collector.collect("Or条件获取到得订单ID:" + order.iterator().next().orderId);

            }
        }).print();


        env.execute();
    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220219174500.png)



## 匹配后跳过策略

 对于一个给定的模式，同一个事件可能会分配到多个成功的匹配上。为了控制一个事件会分配到多少个匹配上，你需要指定跳过策略`AfterMatchSkipStrategy`。 有五种跳过策略，如下：

- ***NO_SKIP***: 每个成功的匹配都会被输出。
- ***SKIP_TO_NEXT***: 丢弃以相同事件开始的所有部分匹配。
- ***SKIP_PAST_LAST_EVENT***: 丢弃起始在这个匹配的开始和结束之间的所有部分匹配。
- ***SKIP_TO_FIRST***: 丢弃起始在这个匹配的开始和第一个出现的名称为*PatternName*事件之间的所有部分匹配。
- ***SKIP_TO_LAST\***: 丢弃起始在这个匹配的开始和最后一个出现的名称为*PatternName*事件之间的所有部分匹配。

注意当使用*SKIP_TO_FIRST*和*SKIP_TO_LAST*策略时，需要指定一个合法的*PatternName*.

例如，给定一个模式`b+ c`和一个数据流`b1 b2 b3 c`，不同跳过策略之间的不同如下：

| 跳过策略                 |             结果              |                             描述                             |
| :----------------------- | :---------------------------: | :----------------------------------------------------------: |
| **NO_SKIP**              | `b1 b2 b3 c` `b2 b3 c` `b3 c` |         找到匹配`b1 b2 b3 c`之后，不会丢弃任何结果。         |
| **SKIP_TO_NEXT**         | `b1 b2 b3 c` `b2 b3 c` `b3 c` | 找到匹配`b1 b2 b3 c`之后，不会丢弃任何结果，因为没有以`b1`开始的其他匹配。 |
| **SKIP_PAST_LAST_EVENT** |         `b1 b2 b3 c`          |     找到匹配`b1 b2 b3 c`之后，会丢弃其他所有的部分匹配。     |
| **SKIP_TO_FIRST**[`b`]   | `b1 b2 b3 c` `b2 b3 c` `b3 c` | 找到匹配`b1 b2 b3 c`之后，会尝试丢弃所有在`b1`之前开始的部分匹配，但没有这样的匹配，所以没有任何匹配被丢弃。 |
| **SKIP_TO_LAST**[`b`]    |      `b1 b2 b3 c` `b3 c`      | 找到匹配`b1 b2 b3 c`之后，会尝试丢弃所有在`b3`之前开始的部分匹配，有一个这样的`b2 b3 c`被丢弃。 |

在看另外一个例子来说明NO_SKIP和SKIP_TO_FIRST之间的差别： 模式： `(a | b | c) (b | c) c+.greedy d`，输入：`a b c1 c2 c3 d`，结果将会是：

| 跳过策略                |                     结果                     |                             描述                             |
| :---------------------- | :------------------------------------------: | :----------------------------------------------------------: |
| **NO_SKIP**             | `a b c1 c2 c3 d` `b c1 c2 c3 d` `c1 c2 c3 d` |       找到匹配`a b c1 c2 c3 d`之后，不会丢弃任何结果。       |
| **SKIP_TO_FIRST**[`c*`] |        `a b c1 c2 c3 d` `c1 c2 c3 d`         | 找到匹配`a b c1 c2 c3 d`之后，会丢弃所有在`c1`之前开始的部分匹配，有一个这样的`b c1 c2 c3 d`被丢弃。 |

为了更好的理解NO_SKIP和SKIP_TO_NEXT之间的差别，看下面的例子： 模式：`a b+`，输入：`a b1 b2 b3`，结果将会是：

| 跳过策略         |             结果              |                             描述                             |
| :--------------- | :---------------------------: | :----------------------------------------------------------: |
| **NO_SKIP**      | `a b1` `a b1 b2` `a b1 b2 b3` |            找到匹配`a b1`之后，不会丢弃任何结果。            |
| **SKIP_TO_NEXT** |            `a b1`             | 找到匹配`a b1`之后，会丢弃所有以`a`开始的部分匹配。这意味着不会产生`a b1 b2`和`a b1 b2 b3`了。 |

想指定要使用的跳过策略，只需要调用下面的方法创建`AfterMatchSkipStrategy`：

| 方法                                              |                          描述                          |
| :------------------------------------------------ | :----------------------------------------------------: |
| `AfterMatchSkipStrategy.noSkip()`                 |                  创建**NO_SKIP**策略                   |
| `AfterMatchSkipStrategy.skipToNext()`             |                创建**SKIP_TO_NEXT**策略                |
| `AfterMatchSkipStrategy.skipPastLastEvent()`      |            创建**SKIP_PAST_LAST_EVENT**策略            |
| `AfterMatchSkipStrategy.skipToFirst(patternName)` | 创建引用模式名称为*patternName*的**SKIP_TO_FIRST**策略 |
| `AfterMatchSkipStrategy.skipToLast(patternName)`  | 创建引用模式名称为*patternName*的**SKIP_TO_LAST**策略  |

可以通过调用下面方法将跳过策略应用到模式上：

```java
AfterMatchSkipStrategy skipStrategy = ...
Pattern.begin("patternName", skipStrategy);
```

使用SKIP_TO_FIRST/LAST时，有两个选项可以用来处理没有事件可以映射到对应的变量名上的情况。 默认情况下会使用NO_SKIP策略，另外一个选项是抛出异常。 可以使用如下的选项：

```java
AfterMatchSkipStrategy.skipToFirst(patternName).throwExceptionOnMiss()
```

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.nfa.aftermatch.AfterMatchSkipStrategy;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class AfterMatchSkipStrategyDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "C", 1L),
                        new OrderEvent("1", "D", 2L),
                        new OrderEvent("1", "A", 3L),
                        new OrderEvent("1", "A", 4L),
                        new OrderEvent("1", "A", 5L),
                        new OrderEvent("1", "D", 6L),
                        new OrderEvent("1", "B", 7L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);

        //C D A1 A2 A3 D A4 B


        //
        AfterMatchSkipStrategy skipStrategy = AfterMatchSkipStrategy.noSkip();


        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("start", skipStrategy).where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("c");
                    }
                })
                .followedBy("middle").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("a");
                    }
                }).oneOrMore().allowCombinations()
                .followedBy("end1").where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getEventType().equalsIgnoreCase("b");
                    }
                }).within(Time.seconds(5));


        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);
        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                collector.collect(map);
            }
        }).print();

        env.execute("AfterMatchSkipStrategyDemo");
    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220222210232.png)



## 检测模式

在指定了要寻找的模式后，该把它们应用到输入流上来发现可能的匹配了。为了在事件流上运行你的模式，需要创建一个`PatternStream`。 给定一个输入流`input`，一个模式`pattern`和一个可选的用来对使用事件时间时有同样时间戳或者同时到达的事件进行排序的比较器`comparator`， 你可以通过调用如下方法来创建`PatternStream`：

```java
DataStream<Event> input = ...
Pattern<Event, ?> pattern = ...
EventComparator<Event> comparator = ... // 可选的

PatternStream<Event> patternStream = CEP.pattern(input, pattern, comparator);
```

## PatternProcessFunction 抽取数据

获得到一个`PatternStream`之后，可以应用处理事件序列。推荐使用`PatternProcessFunction`。

`PatternProcessFunction`有一个`processMatch`的方法在每找到一个匹配的事件序列时都会被调用。 它按照`Map<String, List<IN>>`的格式接收一个匹配，映射的键是你的模式序列中的每个模式的名称，值是被接受的事件列表（`IN`是输入事件的类型）。 模式的输入事件按照时间戳进行排序。为每个模式返回一个接受的事件列表的原因是当使用循环模式（比如`oneToMany()`和`times()`）时， 对一个模式会有不止一个事件被接受。

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.functions.PatternProcessFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.IterativeCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;

public class PatternProcessFunctionDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L),
                        new OrderEvent("4", "cancel", 4006L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);


        // 规则 先create 后 pay
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        return orderEvent.eventType.equals("create");
                    }
                })
                .next("pay").where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        return orderEvent.eventType.equals("pay");
                    }
                }).within(Time.seconds(5));


        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);

        patternedStream.process(new PatternProcessFunction<OrderEvent, Object>() {
            @Override
            public void processMatch(Map<String, List<OrderEvent>> map, Context context, Collector<Object> collector) throws Exception {
                List<OrderEvent> order = map.get("pay");
                collector.collect("已经支付的订单ID为:" + order.iterator().next().orderId);

            }
        }).print();


        env.execute();
    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220222211922.png)





## TimeOutPatternProcessFunctionDemo

```java
package com.hph.cep;

import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternFlatSelectFunction;
import org.apache.flink.cep.PatternFlatTimeoutFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.IterativeCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

import java.util.List;
import java.util.Map;

public class TimeOutPatternProcessFunctionDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //创建数据流
        KeyedStream<OrderEvent, String> keyedStream = env.fromElements(
                        new OrderEvent("1", "create", 2000L),
                        new OrderEvent("2", "create", 3000L),
                        new OrderEvent("1", "pay", 4000L),
                        new OrderEvent("2", "pay", 4001L),
                        new OrderEvent("3", "create", 4003L),
                        new OrderEvent("4", "create", 4003L),
                        new OrderEvent("4", "cancel", 4006L),
                        new OrderEvent("3", "pay", 9006L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<OrderEvent>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<OrderEvent>() {
                                    @Override
                                    public long extractTimestamp(OrderEvent element, long recordTimestamp) {
                                        return element.eventTime;
                                    }
                                })
                )
                .keyBy(x -> x.orderId);


        // 规则 先create 后 pay
        Pattern<OrderEvent, OrderEvent> pattern = Pattern
                .<OrderEvent>begin("create")
                .where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        return orderEvent.eventType.equals("create");
                    }
                })
                .next("pay").where(new IterativeCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent orderEvent, Context<OrderEvent> context) throws Exception {
                        return orderEvent.eventType.equals("pay");
                    }
                }).within(Time.seconds(5));


        PatternStream<OrderEvent> patternedStream = CEP.pattern(keyedStream, pattern);
        OutputTag<String> outputTag = new OutputTag<String>("side-output") {
        };


        SingleOutputStreamOperator<String> result = patternedStream.flatSelect(outputTag, new PatternFlatTimeoutFunction<OrderEvent, String>() {
            @Override
            public void timeout(Map<String, List<OrderEvent>> map, long l, Collector<String> collector) throws Exception {

                collector.collect("超时订单ID:" + map.get("create").iterator().next().getOrderId());
            }

        }, new PatternFlatSelectFunction<OrderEvent, String>() {
            @Override
            public void flatSelect(Map<String, List<OrderEvent>> map, Collector<String> collector) throws Exception {
                collector.collect("正常订单ID:" + map.get("pay").iterator().next().getOrderId());
            }
        });
        result.print();
        result.getSideOutput(outputTag).print();

        env.execute("TimeOutPatternProcessFunctionDemo");
    }

}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20220222215330.png)



## 参考资料

[1.Apache Flink CEP 实战](https://developer.aliyun.com/article/738454)

[2.Apache Flink 官网](https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/libs/cep/)

