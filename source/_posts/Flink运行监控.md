---

title: Flink Metrics任务监控
date: 2021-11-14 16:58:24
tags: Flink
categories: 大数据
---

## 监控指标

Flink任务提交得集群后，需要对任务进行有效监控，对Flink得监控指标可以分为系统指标和用户指标。Flink 提供的 Metrics 可以在 Flink 内部收集一些指标，通过这些指标让开发人员更好地理解作业或集群的状态。由于集群运行后很难发现内部的实际状况， Metrics 可以很好的帮助开发人员了解作业的当前状况。

系统指标：

- CPU负载
- 组件内存使用情况

用户指标

- 自定义注册监控指标
- 用户业务状态信息

在FlinkWeb中我们可以看到系统对应的TaskManager、TaskSlots以及Job相关的信息

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211114182059.png)

访问相关界面可以拿到对应的监控指标名称

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211114182300.png)

通过get方式获取到相关指标的数值

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211114182545.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211114181805.png)

这样子看不是特别方便我们这里采用Promethus的方式对来采集Flink-Metrics，并对其进行可视化展示。

## Promethus配置

对于Promethus这里不在详细介绍，使用Promethus Reporter将指标发送到Prometheus中，首先在启动集群前将 flink-metrics-prometheus_2.12-1.13.2.jar 放到 flink的lib目录下,在flink-conf.yaml中配置Promethus Reporter的相关信息。

```yaml
metrics.reporter.prom.class: org.apache.flink.metrics.prometheus.PrometheusReporter
metrics.reporter.prom.port: 9260
```

另外一种方式是使用PromethusPushGateway，将指标发送到指定的网关中，然后Promethus从该网关拉去数据，对应得配置如下所示

```yaml
metrics.reporter.promgateway.class: org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter
metrics.reporter.promgateway.host: node1
metrics.reporter.promgateway.port: 9091
metrics.reporter.promgateway.jobName: myJob
metrics.reporter.promgateway.randomJobNameSuffix: true
metrics.reporter.promgateway.deleteOnShutdown: false
metrics.reporter.promgateway.groupingKey: k1=v1;k2=v2metrics.reporter.promgateway.interval: 10 SECONDS
```

本文没有使用这种方法，这里我门使用的是第一种Promethus Reporter，在flink-conf.yaml 添加如下配置

```yaml
metrics.reporter.prom.class: org.apache.flink.metrics.prometheus.PrometheusReporter
metrics.reporter.prom.port: 9260
```

将 flink-conf.yaml 配置文件和`flink-metrics-prometheus_2.12-1.13.2.jar`分发到各个计算节点，然后重启flink集群，在promethus.yml中添加

```yaml
  - job_name: 'flink_metrics'
    static_configs:
    - targets: ['hadoop102:9260','hadoop103:9260','hadoop104:9260']
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211115104100.png)

Flink-metrics相关组件已经启动，测试作业如下：

```java
package cn.hphblog.job;

import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class HelloFlink {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);


        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "us_ac");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

        //{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));


        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });


        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws Exception {
                return Tuple2.of(t1.f0, t1.f1 + t2.f1);
            }
        }).print();

        environment.execute("Flink Metrics");
    }
}
```

```java
package cn.hphblog.job;

public class UserAction {
    private int Id;
    private String Name;
    private String Operation;
    private Long t;
    private int opt;

    public int getId() {
        return Id;
    }

    public void setId(int id) {
        Id = id;
    }

    public String getName() {
        return Name;
    }

    public void setName(String name) {
        Name = name;
    }

    public String getOperation() {
        return Operation;
    }

    public void setOperation(String operation) {
        Operation = operation;
    }

    public Long getT() {
        return t;
    }

    public void setT(Long t) {
        this.t = t;
    }

    public int getOpt() {
        return opt;
    }

    public void setOpt(int opt) {
        this.opt = opt;
    }

    public UserAction(int id, String name, String operation, Long t, int opt) {
        Id = id;
        Name = name;
        Operation = operation;
        this.t = t;
        this.opt = opt;
    }

    public UserAction() {
    }

    @Override
    public String toString() {
        return "UserAction{" +
                "Id=" + Id +
                ", Name='" + Name + '\'' +
                ", Operation='" + Operation + '\'' +
                ", t=" + t +
                ", opt=" + opt +
                '}';
    }
}
```

## Grafna配置

Grafna如果联网配置也相对简单，https://grafana.com/grafana/dashboards/14911

只需要引入ID为14911即可

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211115105414.png)

效果如下

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211115105946.png)

## 告警

在监控告警中我们往往会遇到如下得问题。

- 阈值设定：不同业务场景，不同指标，如何衡量阈值是过于宽松，还是过于严格。
- 流量波动：在理想的世界里，流量是有起伏规律的，监控系统能够掌握这种规律，当流量上升时，告警阈值自动上升
- 瞬态告警：每个人都会遇到这样的情况，同样的问题隔段时间就出现一次，持续时间不过几分钟，来得快去得也快。说实话，你已经忙得不可开交了，近期内也不大会去排除这种问题。是忽略呢？还是忽略呢？
- 信息过载：典型的信息过载场景是，给所有需要的地方都加上了告警，以为这样即可高枕无忧了，结果随着而来的是，各种来源的告警轻松挤满你的收件箱。
- 故障定位：在相对复杂的业务场景下，一个“告警事件” 除了包含“时间”(何时发生)、“地点”(哪个服务器/组件)、“内容”(包括错误码、状态值等)外，还包含地区、机房、服务、接口等，故障定位之路道阻且长。

基于Promethus我们可以构建出业务所有服务的的监控指标和用户看板，然而这还是不够的，这个时候我们需要用到告警。

我们需要使用到 alertmanager相关的东西，这里对alertmanager 不在做过多介绍，修改promethus.yaml中修改相关配置添加

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - hadoop102:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
   - "/opt/module/prometheus-2.3.0.linux-amd64/rules/*.rules"

```

根据实际的情况修改相关配置，flink集群运行任务告警规则配置如下，对于当集群任务不存在时，发送告警相关的邮件

```yaml
groups:
- name: flink-job
  rules:
  - alert: flink-job
    expr: flink_jobmanager_numRunningJobs  == 0
    for: 10s
    labels:
      severity: 1
      team: node
    annotations:
      summary: "{{ $labels.job }} 任务已经停止"
```

同时修改alertmanager.yml配置文件

```yaml
global:
  smtp_smarthost: 'smtp.qq.com:465'       #QQ邮箱服务器
  smtp_from: '467008580@qq.com'           #发送邮箱
  smtp_auth_username: '467008580@qq.com'  #名称    
  smtp_auth_password: 'xxxxxxxxx'        # 授权码
  smtp_require_tls: false
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'email'
- name: 'email'
  email_configs:
  - to: 'xxxxx@wo.cn'
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

由于我们在Flink集群上目前没有运行任何的任务

所以告警一会儿就会触发。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211115175752.png)

当然也可以选择电话，钉钉等相关告警配置，对于Flink任务监控基本和Promethus监控类似，配置相关的告警规则即可。

## 自定义Metrics

### Metric分类

#### Counter

Counter：对一个计数器进行累加，即对于多条数据和多兆数据一直往上加的过程。

#### Gauge

反映一个值。比如要看现在 Java heap 内存用了多少，就可以每次实时的暴露一个 Gauge，Gauge 当前的值就是 heap 使用的量。

#### Meter

Meter 是指统计吞吐量和单位时间内发生“事件”的次数。它相当于求一种速率，即事件次数除以使用的时间。

#### Histogram

用于统计一些数据的分布，比如说 Quantile、Mean、StdDev、Max、Min 等。

### Metric Group

Metric 在 Flink 内部有多层结构，以 Group 的方式组织，不是一个扁平化的结构，Metric Group + Metric Name 是 Metrics 的唯一标识。

Metric Group 的层级有 TaskManagerMetricGroup 和 TaskManagerJobMetricGroup，每个 Job 具体到某一个 task 的 group，task 又分为 TaskIOMetricGroup 和 OperatorMetricGroup。Operator 下面也有 IO 统计和一些 Metrics，整个层级大概如下图所示。Metrics 不会影响系统，它处在不同的组中，并且 Flink 支持自己去加 Group，可以有自己的层级。

```apl
•TaskManagerMetricGroup
  •TaskManagerJobMetricGroup
    •TaskMetricGroup
      •TaskIOMetricGroup
      •OperatorMetricGroup
        •${User-defined Group} / ${User-defined Metrics}
        •OperatorIOMetricGroup
•JobManagerMetricGroup
  •JobManagerJobMetricGroup
```

#### Counter 

```java
package cn.hphblog.job;

import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.metrics.Counter;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class MyCustomCounter {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);

        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "my_counter");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

//{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));

        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });

        //flink Counter Metrics
        flatMap.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            private transient Counter counter;

            @Override
            public void open(Configuration parameters) throws Exception {
                this.counter = getRuntimeContext()
                        .getMetricGroup()
                        .counter("myCounter");
            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                this.counter.inc();
                return value;
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).
                reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws
                            Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();

        environment.execute("Flink Metrics Counter");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211116111454.png)

#### Gauge

```java
package cn.hphblog.job;

import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.metrics.Gauge;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class MyCustomGauge {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);


        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "my_counter");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

         //{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));


        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });

        //flink Counter Metrics
        flatMap.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            private transient int valueToExpose = 0;

            @Override
            public void open(Configuration parameters) throws Exception {
                getRuntimeContext()
                        .getMetricGroup()
                        .gauge("MyGauge", new Gauge<Integer>() {
                            @Override
                            public Integer getValue() {
                                return valueToExpose;
                            }
                        });
            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                valueToExpose++;
                return value;
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).
                reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws
                            Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();

        environment.execute("Flink Metrics Gauge");

    }
}

```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211116113457.png)

####  Meter

```java
package cn.hphblog.job;

import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.dropwizard.metrics.DropwizardMeterWrapper;
import org.apache.flink.metrics.Meter;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class MyCustomMeter {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);


        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "my_meter");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

//{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));


        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });

        //flink Counter Metrics
        flatMap.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            private transient Meter meter;

            @Override
            public void open(Configuration parameters) throws Exception {
                com.codahale.metrics.Meter dropwizardMeter = new com.codahale.metrics.Meter();
                this.meter = getRuntimeContext()
                        .getMetricGroup()
                        .meter("myMeter", new DropwizardMeterWrapper(dropwizardMeter));

            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                this.meter.markEvent();
                return value;
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).
                reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws
                            Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();

        environment.execute("Flink Metrics Meter");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211116133445.png)

加速发送Kafka数据到us_ac 这一topic中即刻看到相关的趋势变化。

#### Histogram

```java
package cn.hphblog.job;

import com.codahale.metrics.SlidingWindowReservoir;
import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.dropwizard.metrics.DropwizardHistogramWrapper;
import org.apache.flink.metrics.Histogram;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class MyCustomHistogram {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);


        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "my_histogram");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

//{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));


        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });
        //flink Histogram  Metrics
        flatMap.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            private transient Histogram histogram;

            @Override
            public void open(Configuration parameters) throws Exception {
                com.codahale.metrics.Histogram dropwizardHistogram =
                        new com.codahale.metrics.Histogram(new SlidingWindowReservoir(500));

                this.histogram = getRuntimeContext()
                        .getMetricGroup()
                        .histogram("myHistogram", new DropwizardHistogramWrapper(dropwizardHistogram));

            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                this.histogram.update(value.f1);
                return value;
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).
                reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws
                            Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();

        environment.execute("Flink Metrics Histogram");

    }
}
```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211116141427.png)

### 火焰图

火焰图（Flame Graph）是由 Linux 性能优化大师 Brendan Gregg 发明的，和所有其他的 profiling 方法不同的是，火焰图以一个全局的视野来看待时间分布，它从底部往顶部，列出所有可能导致性能瓶颈的调用栈。

- 每一列代表一个调用栈，每一个格子代表一个函数
- 纵轴展示了栈的深度，按照调用关系从下到上排列。最顶上格子代表采样时，正在占用 cpu 的函数。
- 横轴的意义是指：火焰图将采集的多个调用栈信息，通过按字母横向排序的方式将众多信息聚合在一起。需要注意的是它并不代表时间。
- 横轴格子的宽度代表其在采样中出现频率，所以一个格子的宽度越大，说明它是瓶颈原因的可能性就越大。
- 火焰图格子的颜色是随机的暖色调，方便区分各个调用信息。
- 其他的采样方式也可以使用火焰图， on-cpu 火焰图横轴是指 cpu 占用时间，off-cpu 火焰图横轴则代表阻塞时间。

Flink在1.13.1版本之后已经支持火焰图的开启，只需要在flink-conf.yaml 中指定即可

```yaml
rest.flamegraph.enabled : true
```
Flink作业如下

```java
package cn.hphblog.job;

import com.codahale.metrics.SlidingWindowReservoir;
import com.google.gson.Gson;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.dropwizard.metrics.DropwizardHistogramWrapper;
import org.apache.flink.metrics.Histogram;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

public class MyFlameGraph {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);


        Properties props = new Properties();
        //指定kafka的Broker地址
        props.setProperty("bootstrap.servers", "hadoop102:9092,hadoop103:9092,hadoop104:9092");
        //设置组ID
        props.setProperty("group.id", "My_FlameGraph");
        props.setProperty("auto.offset.reset", "earliest");
        //kafka自动提交偏移量，
        props.setProperty("enable.auto.commit", "false");

//{"Id":295,"Name":"Nmae_295","Operation":"add","t":1636887111,"opt":0}
        FlinkKafkaConsumer<String> kafkaSouce = new FlinkKafkaConsumer<>("us_ac",  //指定Topic
                new SimpleStringSchema(),  //指定Schema，生产中一般使用Avro
                props);                     //Kafka配置

        //checkpoint开启
        environment.enableCheckpointing(5000);
        //重试策略开启
        environment.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));


        DataStreamSource<String> streamSource = environment.addSource(kafkaSouce);


        SingleOutputStreamOperator<Tuple2<String, Integer>> flatMap = streamSource.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                UserAction userAction = new Gson().fromJson(s, UserAction.class);
                collector.collect(Tuple2.of(userAction.getOperation(), userAction.getOpt()));
            }
        });
        //flink Histogram  Metrics
        flatMap.map(new RichMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            private transient Histogram histogram;

            @Override
            public void open(Configuration parameters) throws Exception {
                com.codahale.metrics.Histogram dropwizardHistogram =
                        new com.codahale.metrics.Histogram(new SlidingWindowReservoir(500));

                this.histogram = getRuntimeContext()
                        .getMetricGroup()
                        .histogram("myHistogram", new DropwizardHistogramWrapper(dropwizardHistogram));

            }

            @Override
            public Tuple2<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                this.histogram.update(value.f1);
                return value;
            }
        });

        KeyedStream<Tuple2<String, Integer>, String> tuple2StringKeyedStream = flatMap.keyBy(x -> x.f0);
        tuple2StringKeyedStream.timeWindow(org.apache.flink.streaming.api.windowing.time.Time.seconds(5)).
                reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> t1, Tuple2<String, Integer> t2) throws
                            Exception {
                        return Tuple2.of(t1.f0, t1.f1 + t2.f1);
                    }
                }).print();

        environment.execute("Flink FlameGraph ");

    }
}

```



![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20211116150153.png)

### 火焰图分析技巧

1. 纵轴代表调用栈的深度（栈桢数），用于表示函数间调用关系：下面的函数是上面函数的父函数。
2. 横轴代表调用频次，一个格子的宽度越大，越说明其可能是瓶颈原因。
3. 不同类型火焰图适合优化的场景不同，比如 on-cpu 火焰图适合分析 cpu 占用高的问题函数，off-cpu 火焰图适合解决阻塞和锁抢占问题。
4. 无意义的事情：横向先后顺序是为了聚合，跟函数间依赖或调用关系无关；火焰图各种颜色是为方便区分，本身不具有特殊含义

### 参考资料

https://www.infoq.cn/article/a8kmnxdhbwmzxzsytlga

https://nightlies.apache.org/flink/flink-docs-release-1.13/zh/docs/ops/metrics/

https://cloud.tencent.com/developer/news/473085

https://www.cyningsun.com/03-28-2020/site-reliability-engineering.html

