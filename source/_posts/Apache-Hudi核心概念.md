---
title: Apache Hudi 核心概念
date: 2022-07-06 21:36:01
tags: Hudi
categories: Hudi

---

## Hudi简介

Apache Hudi 在 HDFS 的数据集上提供了插入更新和增量拉取的功能。

一般来说，我们会将大量数据存储到 HDFS，新数据增量写入，而旧数据很少有改动，特别是在经过数据清洗，放入数据仓库的场景。而且在数据仓库如 hive 中，对于 update 的支持非常有限，计算昂贵。另一方面，若是有仅对某段时间内新增数据进行分析的场景，则 hive、presto、hbase 等也未提供原生方式，而是需要根据时间戳进行过滤分析。

在此需求下，Hudi 可以提供这两种需求的实现。第一个是对 record 级别的更新，另一个是仅对增量数据的查询。且 Hudi 提供了对 Hive、presto、Spark 的支持，可以直接使用这些组件对 Hudi 管理的数据进行查询。

Hudi 是一个通用的大数据存储系统，主要特性：

- 摄取和查询引擎之间的快照隔离，包括 Apache Hive、Presto 和 Apache Spark。
- 支持回滚和存储点，可以恢复数据集。
- 自动管理文件大小和布局，以优化查询性能准实时摄取，为查询提供最新数据。
- 实时数据和列数据的异步压缩。

Apache Hudi 强调其主要支持Upserts、Deletes和Incrementa数据处理，支持三种数据写入方式：UPSERT，INSERT 和 BULK_INSERT。

## 核心概念

### Timeline

Hudi维护着一条对Hudi数据集所有操作的不同`Instant`组成的`Timeline`（时间轴），通过时间轴，用户可以轻易的进行增量查询或基于某个历史时间点的查询。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220707113410378.png)



`Timeline`在Hudi中被定义为`HoodieTimeline`接口，该接口定义了针对`Instant`的不同操作，包括`commit`、`deltacommit`、`clean`、`rollback`、`savepoint`、`compaction`、`restore`，以及对这些操作进行过滤的方法以及针对`Instant`的状态和操作类型生成不同的文件名的方法，这些操作的含义如下

- `commits`： 一次commit表示将一批数据**原子**性地写入一个表。
- `deltacommit` ：将一批记录**原子写入**到`Merge On Read`存储类型的数据集（写入增量日志log文件中）。
- `cleans` ：删除数据集中不再需要的旧版本文件。
- `rollback` ：表示当`commit/deltacommit`不成功时进行回滚，其会删除在写入过程中产生的部分文件。
- `savepoint`：将某些文件组标记为**已保存**，以便其不会被删除。在发生灾难需要恢复数据的情况下，它有助于将数据集还原到时间轴上的某个点。
- `compaction` ： 将基于行的log日志文件转变成列式parquet数据文件。`compaction`在时间轴上表现为特殊提交。
- `restore`：将从某个`savepoint`恢复。

`Timeline`与`Instant`密切相关，每条`Timeline`必须包含零或多个`Instant`。所有`Instant`构成了`Timeline`，`Instant`在Hudi中被定义为`HoodieInstant`，其主要包含三个字段

```sql
  public HoodieInstant(State state, String action, String timestamp) {
    this.state = state;
    this.action = action;
    this.timestamp = timestamp;
  }
```

State为枚举值

```java
  public enum State {
    // Requested State (valid state for Compaction) 表示某个action已经调度，但尚未执行。
    REQUESTED,
    // Inflight instant  表示action当前正在执行。
    INFLIGHT,
    // Committed instant 表示timeline上的action已经完成。
    COMPLETED,
    // Invalid instant
    INVALID
  }
```

`state`：状态，如`requested`、`inflight`、`completed`等状态，状态会转变，如当提交完成时会从`inflight`状态转变为`completed`状态。

`action`：操作，对数据集执行的操作类型，如`commit`、`deltacommit`等。

`tmiestamp`：时间戳，发生的时间戳，Hudi会保证单调递增。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707111537.png)

在时间轴上不同的操作会产生不同的instance，

`Timeline`（时间轴）是Hudi中非常重要的概念，基于历史时间点的查询及增量查询均需由`Timeline`提供支持

Ø Arrival time: 数据到达 Hudi 的时间，commit time。

Ø Event time: record 中记录的时间。

### 文件管理

- Hudi将HDFS上的数据集组织到基本路径下的目录结构。
- 数据集为多个分区,这些分区和HIVE表非常相似是包含该分区数据文件的文件夹。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707114723.png)



在每一个分区内，文件被组织称为一个文件组，由文件id充当唯一标识。每个文件组包含多个文件切片，其中每个切片包含在某个即时时间的提交/压缩生成的基本列文件(.parquet)以及一组日志文件(.log) ，该文件包含自主生成基本文件以来对基本文件的插入/更新。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707115048.png)

Hudi的 base file (parquet文件)在footer的meta去记录了 record key 组成的BloomFilter，用于在file base index 的实现高效的key contains 检测。

Hudi的log(avro文件)是自己编码的，通过积攒数据buffer以LogBlock为单位写出，每个LogBlock为单位写出，每个LogBlock包含magic number、size、content、footer等信息，用户数据读、校验和和过滤。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707144639.png)

### 索引Index

- Hudi通过索引机制提供高效的upsert操作，该机制会将RecordKey+PartitionPath组合的方式作为唯一标识符映射到一个文件ID，而且这个唯一标识和文件组/文件ID之间的映射自记录被写入文件组开始就不会在改变。
  - 全局索引：在全表的所有分区范围下强制要求键有且只有一个对应的记录（适用于小表）。
  - 非全局索引：仅在表的某一个分区内强制要求键保持唯一，它依靠写入器为同一个记录的更删提供一致（适用于大表）。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/数据湖/20221207095347.png)

有了索引之后，更新的数据可以快速被定位到对应的 File Group。上图为例，白色是基本文件，黄色是更新数据，有了索引机制，可以做到：避免读取不需要的文件、避免更新不必要的文件、无需将更新数据与历史数据做分布式关联，只需要在 File Group 内做合并。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707145853.png)

#### 索引类别

| Index 类别              | 原理                                                         | 优点                                                      | 缺点                                   |
| ----------------------- | ------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| Bloom Index             | 默认配置，使用布隆过滤器来判断记录存在与否，也可选使用record key的范围裁剪需要的文件 | 效率高，不依赖外部系统，数据和索引保持一致性              | 因假阳性问题，还需回溯原文件再查找一遍 |
| Simple Index            | 把update/delete操作的新数据和老数据进行join                  | 实现最简单，无需额外的资源                                | 性能比较差                             |
| HBase Index             | 把index存放在HBase里面。在插入 File Group定位阶段所有task向HBase发送 Batch Get 请求，获取 Record Key 的 Mapping 信息 | 对于小批次的keys，查询效率高                              | 需要使用HBase，增加了运维负担          |
| Flink State-based Index | HUDI 在 0.8.0 版本中实现的 Flink witer，采用了 Flink 的 state 作为底层的 index 存储，每个 records 在写入之前都会先计算目标 bucket ID。 | 不同于 BloomFilter Index，避免了每次重复的文件 index 查找 |                                        |

#### 索引选择

##### 对事实表的延迟更新

共享出行的行程表、股票买卖记录的表、和电商的订单表。这些表通常一直在增长，大部分的更新随机发生在较新的记录上，旧记录有着长尾分布型的更新。这通常是源于交易关闭或者数据更正的延迟性，大部分更新会发生在最新的几个分区上而小部分会在旧的分区。

![image-20221207101222204](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221207101222204.png)

查询索引可以靠设置得当的布隆过滤器来裁剪很多数据文件,成的键可以以某种顺序排列,Hudi用所有文件的键域来构造区间树，这样能来高效地依据输入的更删记录的键域来排除不匹配的文件。Hudi缓存了输入记录并使用了自定义分区器和统计规律来解决数据的偏斜。有时，如果布隆过滤器的假阳性率过高，查询会增加数据的打乱操作。Hudi支持动态布隆过滤器（设置hoodie.bloom.index.filter.type=DYNAMIC_V0）。它可以根据文件里存放的记录数量来调整大小从而达到设定的假阳性率。

##### 事件表去重

从Apache Kafka或其他类似的消息总线发出的事件数通常是事实表大小的10-100倍，如物联网的事件流、点击流数据、广告曝光数，这些大部分都是仅追加的数据，插入和更新只存在于最新的几个分区中。重复事件可能发生在整个数据管道的任一节点。可以在入库之前用数据湖进行驱虫

![image-20221207102052130](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221207102052130.png)

用一个键值存储来实现去重（即HBase索引），但索引存储的消耗会随着事件数增长而线性增长以至于变得不可行。事实上，有范围裁剪功能的布隆索引是最佳的解决方案。我们可以利用作为首类处理对象的时间来构造由事件时间戳和事件id（event_ts+event_id)组成的键，这样插入的记录就有了单调增长的键。

##### 维度表的随机更删

随机写入的作业场景下，更新操作通常会触及表里大多数文件从而导致布隆过滤器依据输入的更新对所有文件标明阳性。最终会导致，即使采用了范围比较，也还是检查了所有文件。使用简单索引对此场景更合适，因为它不采用提前的裁剪操作，而是直接和所有文件的所需字段连接。如果额外的运维成本可以接受的话，也可以采用HBase索引，其对这些表能提供更加优越的查询效率。

当使用全局索引时，也可以考虑通过设置hoodie.bloom.index.update.partition.path=true或hoodie.simple.index.update.partition.path=true来处理 的情况；例如对于以所在城市分区的用户表，会有用户迁至另一座城市的情况。这些表也非常适合采用Merge-On-Read表型。

![image-20221207102210212](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20221207102210212.png)

### 数据表类型

- 写时复制（copy on write）：仅使用列式文件（parquet）存储数据。在写入/更新数据时，直接同步合并原文件，生成新版本的文件（需要重写整个列数据文件，即使只有一个字节的新数据被提交）。此存储类型下，写入数据非常昂贵，而读取的成本没有增加，所以适合频繁读的工作负载，因为数据集的最新版本在列式文件中始终可用，以进行高效的查询。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707153649.png)

- 读时合并（merge on read）：使用列式（parquet）与行式（avro）文件组合，进行数据存储。在更新记录时，更新到增量文件中（avro），然后进行异步（或同步）的compaction，创建列式文件（parquet）的新版本。此存储类型适合频繁写的工作负载，因为新记录是以appending 的模式写入增量文件中。但是在读取数据集时，需要将增量文件与旧文件进行合并，生成列式文件。

  ![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220707153721.png)

  

| Trade-off           | CopyOnWrite                     | MergeOnRead                              |
| :------------------ | :------------------------------ | :--------------------------------------- |
| Data Latency        | Higher                          | Lower                                    |
| Query Latency       | Lower                           | Higher                                   |
| Update cost (I/O)   | Higher (rewrite entire parquet) | Lower (append to delta log)              |
| Parquet File Size   | Smaller (high update(I/0) cost) | Larger (low update cost)                 |
| Write Amplification | Higher                          | Lower (depending on compaction strategy) |

### 写流程（UPSERT）

#### Copy On Write

（1）先对 records 按照 record key 去重

（2）首先对这批数据创建索引 (HoodieKey => HoodieRecordLocation)；通过索引区分哪些 records 是 update，哪些 records 是 insert（key 第一次写入）

（3）对于 update 消息，会直接找到对应 key 所在的最新 FileSlice 的 base 文件，并做 merge 后写新的 base file (新的 FileSlice)  ==> parquet 文件

（4）对于 insert 消息，会扫描当前 partition 的所有 SmallFile（小于一定大小的 base file），然后 merge 写新的 FileSlice；如果没有 SmallFile，直接写新的 FileGroup + FileSlice

#### Merge On Read

（1）先对 records 按照 record key 去重（可选）

（2）首先对这批数据创建索引 (HoodieKey => HoodieRecordLocation)；通过索引区分哪些 records 是 update，哪些 records 是 insert（key 第一次写入）

（3）如果是 insert 消息，如果 log file 不可建索引（默认），会尝试 merge 分区内最小的 base file （不包含 log file 的 FileSlice），生成新的 FileSlice；如果没有 base file 就新写一个 FileGroup + FileSlice + base file；如果  log file 可建索引，尝试 append 小的 log file，如果没有就新写一个  FileGroup + FileSlice + base file

（4）如果是 update 消息，写对应的 file group + file slice，直接 append 最新的 log file（如果碰巧是当前最小的小文件，会 merge base file，生成新的 file slice）

（5）log file 大小达到阈值会 roll over 一个新的

### 写流程（INSERT）

#### Copy On Write

（1）先对 records 按照 record key 去重（可选）

（2）不会创建 Index

（3）如果有小的 base file 文件，merge base file，生成新的 FileSlice + base file，否则直接写新的 FileSlice + base file

#### Merge On Read

（1）先对 records 按照 record key 去重（可选）

（2）不会创建 Index

（3）如果 log file 可索引，并且有小的 FileSlice，尝试追加或写最新的 log file；如果 log file 不可索引，写一个新的 FileSlice + base file 

### 写流程（INSERT OVERWRITE）

#### Copy On Write

在同一分区中创建新的文件组集。现有的文件组被标记为 "删除"。根据新记录的数量创建新的文件组

| 在插入分区之前                               | 插入相同数量的记录覆盖                                       | 插入覆盖更多的记录                                           | 插入重写1条记录                                              |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 分区包含file1-t0.parquet，file2-t0.parquet。 | 分区将添加file3-t1.parquet，file4-t1.parquet。file1, file2在t1后的元数据中被标记为无效。 | 分区将添加file3-t1.parquet，file4-t1.parquet，file5-t1.parquet，...，fileN-t1.parquet。file1, file2在t1后的元数据中被标记为无效 | 分区将添加file3-t1.parquet。file1, file2在t1后的元数据中被标记为无效。 |

#### Merge On Read

| 在插入分区之前                                             | 插入相同数量的记录覆盖                                       | 插入覆盖更多的记录                                           | 插入重写1条记录                                              |
| ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 分区包含file1-t0.parquet，file2-t0.parquet。.file1-t00.log | file3-t1.parquet，file4-t1.parquet。file1, file2在t1后的元数据中被标记为无效。 | file3-t1.parquet, file4-t1.parquet...fileN-t1.parquetfile1, file2在t1后的元数据中被标记为无效 | 分区将添加file3-t1.parquet。file1, file2在t1后的元数据中被标记为无效。 |

####  优点

（1）COW和MOR在执行方面非常相似。不干扰MOR的compaction。

（2）减少parquet文件大小。

（3）不需要更新关键路径中的外部索引。索引实现可以检查文件组是否无效（类似于在HBaseIndex中检查commit是否无效的方式）。

（4）可以扩展清理策略，在一定的时间窗口后删除旧文件组。

####  缺点

（1）需要转发以前提交的元数据。

- 在t1，比如file1被标记为无效，我们在t1.commit中存储 "invalidFiles=file1"(或者在MOR中存储deltacommit)

- 在t2，比如file2也被标记为无效。我们转发之前的文件，并在t2.commit中标记 "invalidFiles=file1, file2"（或MOR的deltacommit）

（2）忽略磁盘中存在的parquet文件也是Hudi的一个新行为, 可能容易出错,我们必须认识到新的行为，并更新文件系统的所有视图来忽略它们。这一点可能会在实现其他功能时造成问题。

### 查询类型

- Snapshot Queries(快照查询):
  - 查询某个增量提交操作中数据集的最新快照，先进行动态合并最新的基本文件(Parquet)和增量文件(Avro)来提供近实时数据集(通常会存在几分钟的延迟)
  - 读取所有partition下每个FileGroup`最新的FileSlice中的文件`，Copy On Write 表读取parquet文件，Merge On Read读取parquet+log文件。
- Incremental Queries(增量查询)
  - 仅查询新写入数据集的文件，需要指定一个Commit/Compaction的即时时间(位于Timeline的某个Instant) 作为条件，来查询此条件之后的新数据。
  - 可查看自给定commit/delta commit 即时操作以来新写入的数据，有效提供变更流来启用增量数据管道。
- **Read Optimized Queries** （读优化查询）
  - 直接查询基本文件(数据集的最新快照)，其实就是列式文件(Parquet)。并保证和非Hudi的数据集相比，具有相同的列式查询性能。
  - 可查看给定的commit/compact即时操作的表的最新快照。
  - 读优化查询和快照查询相同仅访问基本文件，提供给顶文件片自上次执行压缩操作以来的数据，通常查询数据的最新的保证取决于压缩策略。

| Trade-off     | Snapshot                                                     | Read Optimized                               |
| :------------ | :----------------------------------------------------------- | :------------------------------------------- |
| Data Latency  | Lower                                                        | Higher                                       |
| Query Latency | Higher (merge base / columnar file + row based delta / log files) | Lower (raw base / columnar file performance) |

## 参考

黑马大数据Hudi教程

https://zhuanlan.zhihu.com/p/97433744