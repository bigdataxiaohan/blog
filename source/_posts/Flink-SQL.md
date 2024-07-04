---
title: Flink-SQL
date: 2021-08-26 11:08:24
tags: Flink
categories: 大数据
---



Flink 为日期和时间提供了丰富的数据类型， 包括 `DATE`， `TIME`， `TIMESTAMP`， `TIMESTAMP_LTZ`， `INTERVAL YEAR TO MONTH`， `INTERVAL DAY TO SECOND` ，对多种时间类型和时区的支持使得跨时区的数据处理变得非常容易。

## TIMESTAMP 

- `TIMESTAMP(p)` 是 `TIMESTAMP(p) WITHOUT TIME ZONE` 的简写， 精度 `p` 支持的范围是0-9， 默认是6。
- `TIMESTAMP` 用于描述年， 月， 日， 小时， 分钟， 秒 和 小数秒对应的时间戳。

```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class PrintTableDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();
        TableEnvironment tableEnv = TableEnvironment.create(settings);
        //测试生成数据
        tableEnv.executeSql("CREATE TABLE datagen (\n" +
                "    f_sequence INT,\n" +
                "    f_random INT,\n" +
                "    f_random_str STRING,\n" +
                "    ts as  localtimestamp\n" +
                "  ) WITH (\n" +
                "    'connector' = 'datagen',\n" +
                "    -- optional options --\n" +
                "    'rows-per-second'='5',\n" +
                "    'fields.f_sequence.kind'='sequence',\n" +
                "    'fields.f_sequence.start'='1',\n" +
                "    'fields.f_sequence.end'='9',\n" +
                "    'fields.f_random.min'='1',\n" +
                "    'fields.f_random.max'='9',\n" +
                "    'fields.f_random_str.length'='20'\n" +
                "  )");

        //测试写入数据
        tableEnv.executeSql("  CREATE TABLE print_table (\n" +
                "    f_sequence INT,\n" +
                "    f_random INT,\n" +
                "    f_random_str STRING,\n" +
                "    ts   TIMESTAMP\n" +
                "    ) WITH (\n" +
                "    'connector' = 'print'\n" +
                "  )");

        tableEnv.executeSql("desc   print_table").print();
        //测试写入print表中
        tableEnv.executeSql("  INSERT INTO print_table select f_sequence,f_random,f_random_str,ts from datagen");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830113146.png)

 ## TIMESTAMP_LTZ 类型

- `TIMESTAMP_LTZ(p)` 是 `TIMESTAMP(p) WITH LOCAL TIME ZONE` 的简写， 精度 `p` 支持的范围是0-9， 默认是6。
- `TIMESTAMP_LTZ` 用于描述时间线上的绝对时间点， 使用 long 保存从 epoch 至今的毫秒数， 使用int保存毫秒中的纳秒数。 epoch 时间是从 java 的标准 epoch 时间 `1970-01-01T00:00:00Z` 开始计算。 在计算和可视化时， 每个 `TIMESTAMP_LTZ` 类型的数据都是使用的 session （会话）中配置的时区。
- `TIMESTAMP_LTZ` 没有字符串表达形式因此无法通过字符串来指定， 可以通过一个 long 类型的 epoch 时间来转化(例如: 通过 Java 来产生一个 long 类型的 epoch 时间 `System.currentTimeMillis()`)

```sql
SET 'sql-client.execution.result-mode' = 'tableau';
CREATE VIEW MyView1 AS SELECT LOCALTIME, LOCALTIMESTAMP, CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP, CURRENT_ROW_TIMESTAMP(), NOW(), PROCTIME();
DESC MyView1;

SET 'table.local-time-zone' = 'UTC';
SELECT * FROM MyView1;
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830142901.png)

```sql
SET 'table.local-time-zone' = 'Asia/Shanghai';
SELECT * FROM MyView1;
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830142938.png)

<font color="red">目前仅1.13.2版本之后支持，且`TIMESTAMP_LTZ` 转为String类型时设置会失效，这一版本很好的修复了之前 `PROCTIME()` 函数返回的类型是 `TIMESTAMP` ， 返回值是UTC时区下的 `TIMESTAMP`的问题 ，不需要在对时间进行处理。</font>

## SQL Demo

了解了上述的时间相关概念我们可以学习如何使用Flink SQL 完成DataStream API的操作

### PrintDemo

```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class PrintTableDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();

        TableEnvironment tableEnv = TableEnvironment.create(settings);

        //测试生成数据
        tableEnv.executeSql("CREATE TABLE datagen (\n" +
                "    f_sequence INT,\n" +
                "    f_random INT,\n" +
                "    f_random_str STRING,\n" +
                "    ts as  localtimestamp\n" +
                "  ) WITH (\n" +
                "    'connector' = 'datagen',\n" +
                "    -- optional options --\n" +
                "    'rows-per-second'='5',\n" +
                "    'fields.f_sequence.kind'='sequence',\n" +
                "    'fields.f_sequence.start'='1',\n" +
                "    'fields.f_sequence.end'='9',\n" +
                "    'fields.f_random.min'='1',\n" +
                "    'fields.f_random.max'='9',\n" +
                "    'fields.f_random_str.length'='20'\n" +
                "  )");

        //测试写入数据
        tableEnv.executeSql("  CREATE TABLE print_table (\n" +
                "    f_sequence INT,\n" +
                "    f_random INT,\n" +
                "    f_random_str STRING,\n" +
                "    ts   TIMESTAMP\n" +
                "    ) WITH (\n" +
                "    'connector' = 'print'\n" +
                "  )");

        tableEnv.executeSql("desc   print_table").print();
        //测试写入表中
        tableEnv.executeSql("  INSERT INTO print_table select f_sequence,f_random,f_random_str,ts from datagen");
    }
}

```

随机生成相关的数据，并将数据写入到print_table中

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830144319.png)

### KafkaTable2FileDemo

```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class KafkaTable2FileDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();

        TableEnvironment tableEnv = TableEnvironment.create(settings);

        //创建Kafka 数据源
        tableEnv.executeSql("CREATE TABLE KafkaTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING,\n" +
                "  `ts` TIMESTAMP(3) METADATA FROM 'timestamp'\n" +
                ") WITH (\n" +
                "  'connector' = 'kafka',\n" +
                "  'topic' = 'iris',\n" +
                "  'properties.bootstrap.servers' = 'hadoop102:9092,hadoop103:9092,hadoop104:9092',\n" +
                "  'properties.group.id' = 'test',\n" +
                "  'scan.startup.mode' = 'earliest-offset',\n" +
                "  'format' = 'json'\n" +
                ")");
        //写入文件中
        tableEnv.executeSql("CREATE TABLE MyUserTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING\n" +
                ") PARTITIONED BY (type) WITH (\n" +
                "  'connector' = 'filesystem',        \n" +
                "  'path' = '/tmp/iris_data',  \n" +
                "  'format' = 'json',                                    \n" +
                "  'partition.default-name' = 'type'\n" +
                ")");
        tableEnv.executeSql("INSERT INTO MyUserTable SELECT  sepalLength,sepalWidth,petalLength,petalWidth,type from KafkaTable");
    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830145229.png)

文件已经写入

### KafkaTable2MysqlDemo



```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class KafkaTable2MysqlDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();
        TableEnvironment tableEnv = TableEnvironment.create(settings);
        //创建Kafka 数据源
        tableEnv.executeSql("CREATE TABLE KafkaTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING,\n" +
                "  `ts` TIMESTAMP(3) METADATA FROM 'timestamp'\n" +
                ") WITH (\n" +
                "  'connector' = 'kafka',\n" +
                "  'topic' = 'iris',\n" +
                "  'properties.bootstrap.servers' = 'hadoop102:9092,hadoop103:9092,hadoop104:9092',\n" +
                "  'properties.group.id' = 'test',\n" +
                "  'scan.startup.mode' = 'earliest-offset',\n" +
                "  'format' = 'json'\n" +
                ")");
        //创建Mysql SinkTable
        tableEnv.executeSql("CREATE TABLE MysqlTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth`  DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING,\n" +
                "  PRIMARY KEY (type) NOT ENFORCED\n" +
                ") WITH (\n" +
                "  'connector' = 'jdbc',\n" +
                "  'url' = 'jdbc:mysql://hadoop102:3306/kafkasink',\n" +
                "  'driver'='com.mysql.jdbc.Driver' ,\n" +
                "  'username' = 'root',\n" +
                "  'table-name'='MysqlTable' ,\n" +
                "  'password' = '123456',\n" +
                "  'sink.buffer-flush.max-rows' = '1'\n" +
                ")");
        //写入Mysql表中
        tableEnv.executeSql("INSERT INTO MysqlTable SELECT  sepalLength,sepalWidth,petalLength,petalWidth,type from KafkaTable");

    }
}

```

从kafka的监控看到iris中的数据为300条，这里为了效果设置为1条就写入Mysql。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830145604.png)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/%E5%8A%A8%E7%94%BB.gif)

数据已经写入

### KafkaTable2HbaseDemo

```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class KafkaTable2HbaseDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();

        TableEnvironment tableEnv = TableEnvironment.create(settings);

        //创建Kafka 数据源
        tableEnv.executeSql("CREATE TABLE KafkaTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING,\n" +
                "  `ts` TIMESTAMP(3) METADATA FROM 'timestamp'\n" +
                ") WITH (\n" +
                "  'connector' = 'kafka',\n" +
                "  'topic' = 'iris',\n" +
                "  'properties.bootstrap.servers' = 'hadoop102:9092,hadoop103:9092,hadoop104:9092',\n" +
                "  'properties.group.id' = 'test',\n" +
                "  'scan.startup.mode' = 'earliest-offset',\n" +
                "  'format' = 'json'\n" +
                ")");


        //注册Hbase表
        tableEnv.executeSql("CREATE TABLE hTable (\n" +
                " type VARCHAR,\n" +
                " f1 ROW<sepalLength DOUBLE, sepalWidth DOUBLE,petalLength DOUBLE, petalWidth DOUBLE>\n" +
                ") WITH (\n" +
                " 'connector' = 'hbase-2.2',\n" +
                " 'table-name' = 'kafka2hbase',\n" +
                " 'sink.buffer-flush.max-rows'='1' , \n" +
                " 'zookeeper.quorum' = 'hadoop102:2181,hadoop103:2181,hadoop104:2181'\n" +
                ")\n");
        tableEnv.executeSql("INSERT INTO hTable SELECT  type,ROW(sepalLength,sepalWidth,petalLength,petalWidth) from KafkaTable");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/20210830153149.png)

这里之所以保留3条是因为数据本身的ROW为 Iris数据集的类型 改数据集中只有3个类型 因此实际保留的数据只有3条。

### KafkaTable2EsDemo

```java
package com.hph.sql.demo;

import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

public class KafkaTable2EsDemo {
    public static void main(String[] args) {

        EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();

        TableEnvironment tableEnv = TableEnvironment.create(settings);

        //创建Kafka 数据源
        tableEnv.executeSql("CREATE TABLE KafkaTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING,\n" +
                "  `ts` TIMESTAMP(3) METADATA FROM 'timestamp'\n" +
                ") WITH (\n" +
                "  'connector' = 'kafka',\n" +
                "  'topic' = 'iris',\n" +
                "  'properties.bootstrap.servers' = 'hadoop102:9092,hadoop103:9092,hadoop104:9092',\n" +
                "  'properties.group.id' = 'test',\n" +
                "  'scan.startup.mode' = 'earliest-offset',\n" +
                "  'format' = 'json'\n" +
                ")");
        //写入ES表中
        tableEnv.executeSql("CREATE TABLE esTable (\n" +
                "  `sepalLength` DOUBLE,\n" +
                "  `sepalWidth` DOUBLE,\n" +
                "  `petalLength` DOUBLE,\n" +
                "  `petalWidth`  DOUBLE,\n" +
                "   `type`       STRING\n" +
                ") WITH (\n" +
                "    'connector' = 'elasticsearch-7',\n" +
                "    'hosts' = 'http://192.168.2.123:9200',\n" +
                "    'index' = 'iris')");
        tableEnv.executeSql("INSERT INTO esTable SELECT  sepalLength,sepalWidth,petalLength,petalWidth,type from KafkaTable");

    }
}
```

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/Flink/%E5%86%99%E5%85%A5ES.gif)

