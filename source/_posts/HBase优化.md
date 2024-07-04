---
title: HBase优化
date: 2019-01-0 23:08:43
tags: HBase
categories: 大数据
---

## 高可用

HBase中Hmaster负责监控RegionServer的生命周期，均衡RegionServer的负载，如果Hmaster挂掉了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对Hmaster的高可用配置。

### 步骤

关闭HBase集群（如果没有开启则跳过此步）

```
stop-hbase.sh
```

在conf目录下创建backup-masters文件

```
touch conf/backup-masters
```

在backup-masters文件中配置高可用HMaster节点

```
echo datanode2 > conf/backup-masters
```

将整个conf目录scp到其他节点

```
scp -r conf/ datanode1:/opt/module/hbase/    
scp -r conf/ datanode3:/opt/module/hbase/
```

### Web界面

[![FTMBfH.png](https://s2.ax1x.com/2019/01/04/FTMBfH.png)](https://s2.ax1x.com/2019/01/04/FTMBfH.png)

## 预分区

每一个region维护着startRow与endRowKey，如果加入的数据符合某个region维护的rowKey范围，则该数据交给这个region维护。依照这个原则，我们可以将数据所要投放的分区提前大致的规划好，以提高HBase性能。

### 手动设定预分区

```
create 'staff1','info','partition1',SPLITS => ['1000','2000','3000','4000']
```

### 生成16进制序列预分区

```
create 'staff2','info','partition2',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```

### 按照文件中设置的规则预分区

```
[hadoop@datanode1 hbase]$ vim splits.txt  #在hbase创建该文件
aaaa
bbbb
cccc
dddd
create 'staff2','info','partition2',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```

### 使用JavaAPI创建预分区

```
//自定义算法，产生一系列Hash散列值存储在二维数组中
byte[][] splitKeys = 某个散列值函数
//创建HBaseAdmin实例
HBaseAdmin hAdmin = new HBaseAdmin(HBaseConfiguration.create());
//创建HTableDescriptor实例
HTableDescriptor tableDesc = new HTableDescriptor(tableName);
//通过HTableDescriptor实例和散列值二维数组创建带有预分区的HBase表
hAdmin.createTable(tableDesc, splitKeys);
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.util.Bytes;
 
public class create_table_sample {
    public static void main(String[] args) throws Exception {
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "192.168.1.101,192.168.1.102,192.168.1.103");
        Connection connection = ConnectionFactory.createConnection(conf);
        Admin admin = connection.getAdmin();
 
        TableName table_name = TableName.valueOf("Java_Table");
        if (admin.tableExists(table_name)) {
            admin.disableTable(table_name);
            admin.deleteTable(table_name);
        }
 
        HTableDescriptor desc = new HTableDescriptor(table_name);
        HColumnDescriptor family1 = 
           new HColumnDescriptor(Bytes.toBytes("info"));
        family1.setTimeToLive(3 * 60 * 60 * 24);     //过期时间
        family1.setMaxVersions(3);                   //版本数
        desc.addFamily(family1);
 
        byte[][] splitKeys = {
            Bytes.toBytes("row01"),
            Bytes.toBytes("row02"),
        };
 
        admin.createTable(desc, splitKeys);
        admin.close();
        connection.close();
        System.out.println("创建成功!!!");
    }
}
```

## RowKey设计

一条数据的唯一标识就是rowkey，那么这条数据存储于哪个分区，取决于rowkey处于哪个一个预分区的区间内，设计rowkey的主要目的，就是让数据均匀的分布于所有的region中，在一定程度上防止数据倾斜。接下来我们就谈一谈rowkey常用的设计方案。

### 生成随机数、hash、散列值

原本rowKey为1001的，SHA1后变成：dd01903921ea24941c26a48f2cec24e0bb0e8cc7

原本rowKey为3001的，SHA1后变成：49042c54de64a1e9bf0b33e00245660ef92dc7bd

原本rowKey为5001的，SHA1后变成：7b61dec07e02c188790670af43e717f0f46e8913

在做此操作之前，一般我们会选择从数据集中抽取样本，来决定什么样的rowKey来Hash后作为每个分区的临界值。

### 字符串反转

20170524000001转成10000042507102

20170524000002转成20000042507102

常见的是将URL反转比如 www.baidu.com 转成 com.baidu.wwww

### 字符串拼接

20170524000001_a12e

20170524000001_93i7

主要是为了防止数据倾斜比如电话号码作为Rowkey的时候是有规律可循的，在进行MapReduce作业的时候会导致数据倾斜。

## 内存优化

HBase操作过程中需要大量的内存开销，Table是可以缓存在内存中的，一般会分配整个可用内存的70%给HBase的Java堆。但是不建议分配非常大的堆内存，因为GC过程持续太久会导致RegionServer处于长期不可用状态，一般16~48G内存就可以了，如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

## 基础优化

- 允许在HDFS的文件中追加内容

hdfs-site.xml、hbase-site.xml

属性：dfs.support.append 解释：开启HDFS追加同步，可以优秀的配合HBase的数据同步和持久化。默认值为true。

- 优化DataNode允许的最大文件打开数

hdfs-site.xml

属性：dfs.datanode.max.transfer.threads 解释：HBase一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为4096或者更高。默认值：4096

- 优化延迟高的数据操作的等待时间

hdfs-site.xml

属性：dfs.image.transfer.timeout 解释：如果对于某一次数据操作来讲，延迟非常高，socket需要等待更长的时间，建议把该值设置为更大的值（默认60000毫秒），以确保socket不会被timeout掉。

- 优化数据的写入效率

mapred-site.xml

属性： mapreduce.map.output.compress mapreduce.map.output.compress.codec 解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec或者其他压缩方式。

- 优化DataNode存储

    属性：dfs.datanode.failed.volumes.tolerated 解释： 默认为0，意思是当DataNode中有一个磁盘出现故障，则会认为该DataNode shutdown了。如果修改为1，则一个磁盘出现故障时，数据会被复制到其他正常的DataNode上，当前的DataNode继续工作。

- 设置RPC监听数量

hbase-site.xml

属性：hbase.regionserver.handler.count 解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

- 优化HStore文件大小

hbase-site.xml

属性：hbase.hregion.max.filesize 解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

- 优化hbase客户端缓存

hbase-site.xml

属性：hbase.client.write.buffer 解释：用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

- 指定scan.next扫描HBase所获取的行数

hbase-site.xml

属性：hbase.client.scanner.caching 解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

- flush、compact、split机制

当MemStore达到阈值，将Memstore中的数据Flush进Storefile；compact机制则是把flush出来的小文件合并成大的Storefile文件。split则是当Region达到阈值，会把过大的Region一分为二。

**涉及属性：**

即：128M就是Memstore的默认阈值

hbase.hregion.memstore.flush.size：134217728

即：这个参数的作用是当单个HRegion内所有的Memstore大小总和超过指定值时，flush该HRegion的所有memstore。RegionServer的flush是通过将请求添加一个队列，模拟生产消费模型来异步处理的。那这里就有一个问题，当队列来不及消费，产生大量积压请求时，可能会导致内存陡增，最坏的情况是触发OOM。

hbase.regionserver.global.memstore.upperLimit：0.4 hbase.regionserver.global.memstore.lowerLimit：0.38

即：当MemStore使用内存总量达到hbase.regionserver.global.memstore.upperLimit指定值时，将会有多个MemStores flush到文件中，MemStore flush 顺序是按照大小降序执行的，直到刷新到MemStore使用内存略小于lowerLimit

## 扩展

### 商业项目中的能力

1. 消息量：发送和接收的消息数超过60亿
2. 将近1000亿条数据的读写
3. 高峰期每秒150万左右操作
4. 整体读取数据占有约55%，写入占有45%
5. 超过2PB的数据，涉及冗余共6PB数据
6. 数据每月大概增长300千兆字节。

### 布隆过滤器

在日常生活中，包括在设计计算机软件时，我们经常要判断一个元素是否在一个集合中。比如在字处理软件中，需要检查一个英语单词是否拼写正确（也就是要判断它是否在已知的字典中）；在 FBI，一个嫌疑人的名字是否已经在嫌疑名单上；在网络爬虫里，一个网址是否被访问过等等。最直接的方法就是将集合中全部的元素存在计算机中，遇到一个新元素时，将它和集合中的元素直接比较即可。**一般来讲，计算机中的集合是用哈希表（hash table）来存储的。它的好处是快速准确，缺点是费存储空间。** 当集合比较小时，这个问题不显著，但是当集合巨大时，哈希表存储效率低的问题就显现出来了。比如说，一个像 Yahoo,Hotmail 和 Gmai 那样的公众电子邮件（email）提供商，总是需要过滤来自发送垃圾邮件的人（spamer）的垃圾邮件。一个办法就是记录下那些发垃圾邮件的 email 地址。由于那些发送者不停地在注册新的地址，全世界少说也有几十亿个发垃圾邮件的地址，将他们都存起来则需要大量的网络服务器。如果用哈希表，每存储一亿个 email 地址， 就需要 1.6GB 的内存（用哈希表实现的具体办法是将每一个 email 地址对应成一个八字节的信息指纹googlechinablog.com/2006/08/blog-post.html，然后将这些信息指纹存入哈希表，由于哈希表的存储效率一般只有 50%，因此一个 email 地址需要占用十六个字节。一亿个地址大约要 1.6GB， 即十六亿字节的内存）。因此存贮几十亿个邮件地址可能需要上百 GB 的内存。除非是超级计算机，一般服务器是无法存储的。

 **布隆过滤器只需要哈希表 1/8 到 1/4 的大小就能解决同样的问题。**

**Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。**Bloom Filter的这种高效是有一定代价的：在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false positive）。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom Filter通过极少的错误换取了存储空间的极大节省。

下面我们具体来看Bloom Filter是如何用位数组表示集合的。初始状态时，Bloom Filter是一个包含m位的位数组，每一位都置为0，如图9-5

[![FTDAyV.png](https://s2.ax1x.com/2019/01/04/FTDAyV.png)](https://s2.ax1x.com/2019/01/04/FTDAyV.png)

为了表达S={x1, x2,…,xn}这样一个n个元素的集合，Bloom Filter使用k个相互独立的哈希函数（Hash Function），它们分别将集合中的每个元素映射到{1,…,m}的范围中。对任意一个元素x，第i个哈希函数映射的位置hi(x)就会被置为1（1≤i≤k）。注意，如果一个位置多次被置为1，那么只有第一次会起作用，后面几次将没有任何效果。如图9-6所示，k=3，且有两个哈希函数选中同一个位置（从左边数第五位）。

[![FTDuFJ.png](https://s2.ax1x.com/2019/01/04/FTDuFJ.png)](https://s2.ax1x.com/2019/01/04/FTDuFJ.png)

在判断y是否属于这个集合时，我们对y应用k次哈希函数，如果所有hi(y)的位置都是1（1≤i≤k），那么我们就认为y是集合中的元素，否则就认为y不是集合中的元素。如图9-7所示y1就不是集合中的元素。y2或者属于这个集合，或者刚好是一个false positive。

[![FTDMWR.png](https://s2.ax1x.com/2019/01/04/FTDMWR.png)](https://s2.ax1x.com/2019/01/04/FTDMWR.png)

为了add一个元素，用k个hash function将它hash得到bloom filter中k个bit位，将这k个bit位置1。

为了query一个元素，即判断它是否在集合中，用k个hash function将它hash得到k个bit位。若这k bits全为1，则此元素在集合中；若其中任一位不为1，则此元素比不在集合中（因为如果在，则在add时已经把对应的k个bits位置为1）。

· 不允许remove元素，因为那样的话会把相应的k个bits位置为0，而其中很有可能有其他元素对应的位。因此remove会引入false negative，这是绝对不被允许的。

布隆过滤器决不会漏掉任何一个在黑名单中的可疑地址。但是，它有一条不足之处，也就是**它有极小的可能将一个不在黑名单中的电子邮件地址判定为在黑名单中，因为有可能某个好的邮件地址正巧对应一个八个都被设置成一的二进制位。好在这种可能性很小，我们把它称为误识概率。**

布隆过滤器的好处在于快速，省空间，但是有一定的误识别率，常见的补救办法是在建立一个小的白名单，存储那些可能个别误判的邮件地址。

布隆过滤器具体算法高级内容，如错误率估计，最优哈希函数个数计算，位数组大小计算，请参见<http://blog.csdn.net/jiaomeng/article/details/1495500>。