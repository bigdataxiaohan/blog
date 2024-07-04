---
title: Hadoop分布式文件系统HDFS
date: 2018-12-17 16:14:55
tags: Hadoop
categories: 大数据
---

## HDFS

HDFS是 Hadoop Distribute File System的缩写，是谷歌GFS分布式文件系统的开源实现，Apache Hadoop的一个子项目，HDFS基于流数据访问模式的分布式文件系统，支持海量数据的存储，允许用户将百千台组成存储集群，HDFS运行在低成本的硬件上，提供高吞吐量，高容错性的数据访问。

### 优点

- 可以处理超大文件(TB、PB)。
- 流式数据访问 一次写入多次读取，数据集一旦生成，会被复制分发到不同存储节点上，响应各种数据分析任务请求。
- 商用硬件  可以运行在低廉的商用硬件集群上。
- 异构软硬件平台的可移植性，HDFS在设计的时候就考虑到了平台的可以移植性，这种特性方便了 HDFS在大规模数据应用平台的推广。

###  缺点

- 低延迟的数据访问，要求地时间延迟数据访问的应用不是在HDFS上运行，可以用Hbase，高吞吐量的同时会以提高时间延迟为代价。
- 大量小文件 由于namenode将文件的元数据存储在内存中，因此受制于namenode的内存容量，一般来说目录块和数据库的存储信息约占150个字节也就是说如果有100万个文件，每一个文件占一个数据块，则需要300MB内存，但是存储十亿个就超出了当前的硬件能力了。
- 多用户写入，任意修改文件。HDFS中的文件写入只支持一个用户，且只能以添加的方式正在文件末尾添加数据，不支持多个写操作，不支持文件的任意位置修改。

## 数据块

 HDFS默认的块大小是128M，与单一磁盘上的文件系统相似,HDFS也被划分为块大小的多个分块。HDFS中小于一个块大小的文件不会占据整个块的空间，当一个1M的文件存储在一个128M的块中时，文件只占1M的磁盘空间，而不是128M。HDFS的块比磁盘的块大，目的时为了最小化的寻址开销，假设寻址时间约为10ms，传输速度为100M/s，为了使寻址时间仅占传输时间的1%，我们要将块设置成100MB，默认块的大小为128M，这个块的大小也不能设置得过大。MapReduce中的map任务通常一次只处理一个数据块的数据，因此如果任务数量太少了（少于集群中的节点数量），作业的运行速度就会比较慢。

## NameNode

元数据节点（NameNode)的作用是管理分布式文件的命名空间，一个集群只有一个NameNode节点，主要负责HDFS文件系统的管理工作包括命名空间管理和文件Block管理，在HDFS内部，一个文件被分成为一个或者多个Block的所有元数据信息，主要包括“文件名 —>>数据块的映射“，”数据块 —>>DataNode“的映射列表，该列表通过DataNode上报给NameNode建立，NameNode决定文件数据块到具体的DataNode节点的映射。

NameNode管理文件系统的命名空间(namespace)，它维护这文件系统树以及文件树中所有的文件（文件夹）的元数据。管理这些信息的文件有两个，分别是命名空间镜像文件(namespace image)和操作日志文件(edit log),这些信息被缓存在RAM中，也会持被持久化存储在硬盘。

为了周其性地将元数据节点地镜像文件fsimage和日志edits合并，以防止日志文件过大，HDFS定义了辅助元数据节点（Secondary NameNode）上也保存了一份fsimage文件，以确保在元数据文件中地镜像文件失败时可以恢复。

### 容错机制

#### NFS

备份组成文件系统元数据持久状态的文件，将本地持久状态写入磁盘地同时，写入一个远程挂载地文件系统（NFS）

#### 辅助节点

运行一个辅助的Namenode，不作为Namenode使用，定期合并编辑日志和命名空间镜像，一般在一台单独的物理计算机上运行，因为它需要占用大量CPU时间，并且需要与namenode一样多的内存来执行合并操作，会保存Namenode合并后的命名空间和镜像文件，一般会滞后于NameNode节点的。

#### RAID

使用磁盘陈列NameNode节点上冗余备份Namenode的数据。



## DataNode

数据节点只负责存储数据，一个Block会在多个DataNode中进行冗余备份，一个块在一个DataNode上最多只有一个备份，DataNode上存储了数据块ID和数据块内容以及他们的映射关系。

DataNode定时和NameNode通信，接收NameNode的指令，默认的超时时长为10分钟+30s。 NameNode上不永久保存DataNode上有哪些数据块信息，通过DataNode上报的方式更新NameNode上的映射表，DataNode和NameNode建立连接后，会不断和 NameNode保持联系，包括NameNode第DataNode的一些命令，如删除数据或把数据块复制到另一个DataNode上等，DataNode通常以机架的形式组织，机架通过一个交互机将所有的系统链接起来。机架内部节点之间传输速度快于机架间节点的传输速度。

DataNode同时作为服务器接收客户端的范围呢，处理数据块的读、写请求。DataNode之间还会相互他通信，执行数据块复制任务，在数据块复制任务，在客户端执行写操作时，DataNode之间需要相互通信配合，保持写操作的一致性，DataNode的功能包括：

1. 保存Block，每个块对应原数据信息文件，描述这个块属于那个文件，第几个块等信息。
2. 启动DataNode线程，向NameNode定期汇报Block信息。
3. 定期向NameNode发送心跳保持联系。如果10分钟没有接收到心跳，则认为其lost，将其上的Block复制到其他DataNode节点上。

## SecondaryNameNode

SecondaryNameNode定期地创建命名空间地检查点。首先从NameNode下载fsimage和edit.log然后在本地合并他们，将合并后地fsimage和edit.log上传回到NameNode，这样减少了NameNode重新启动NameNode时合并fsimage和edits.log花费地时间，定期合并fsimage和edit.log文件，使得edits.log大小保持在限定返回内并起到*冷备份*的作用，在NameNode失效的时候，可以恢复fsimage。SecondaryNameNode与NameNode通常运行在不同的机器上，内存呢与NameNode的内存一样大。

参数dfs.namenode.secondary.http-address设置SecondaryNamenode 的通信地址，SecondaryNamenode上的检查点进程开始由两个配置参数控制，第一个参数dfs.namenode.checkpoint.period，指定两个连续检查点之间的时间差默认1小时。dfs.namenode.checkpoint.txns，定义NameNode上新增事务的数量默认1百万条。即使没有到达设定的时间也会启动fsimage和edits的合并。

### 工作流程

1. SecondaryNameNode会定期和NameNode通信，请求其停止使用edits文件，暂时将更新的操作写道一个新的文件edits.new上来。这个操作时瞬间完成的，上层写日志的函数时完全赶不到差别。

2. SecondaryNameNode通过HTTP GET的方式从NameNode上获取fsimage和edits.log文件,并下载到相应的目录下。

3. SecondaryNameNode将下载下来的fsimage载入到内存，然后一条一条的执行edits文件中的哥哥更新操作，使内存中的fsimage保持最新；这个过程就是edits和fsimage文件合并。

4. SecondaryNameNode 执行完3操作后，会通过POST的方式新的fsimage文件发送到NameNode节点上。

5. Namenode从SecondaryNameNode接收到更新的fsimage替换旧的fsimage文件，同时将edits.new更名为edits文件，这个过程中edits就变小了。

    ![F0h1j1.png](https://s1.ax1x.com/2018/12/17/F0h1j1.png)

## HDFS工作机制

### 机架感知

在HDFS中，数据块被复制成副本，存放在不同的节点上，副本的存放是HDFS可靠性和性能的关键，优化副本存放策略使HDFS区分于其他大部分分布式文件系统的共同要特征，HDFS采用了一种称为机架感知的（rck-aware）的策略来改进数据的可靠性，可用性和网络带宽的利用率。HDFS实例一般运行在多个机架的计算机组成的集群上，不同机架上的两台机器之间的通信时通过交互机。在多数情况下，一个机架上的两台机器间的带宽会比不同机架的两台机器的带宽大。

通过一个机架感知的过程,NameNode可以确定每一个Datanode所属的机架id，一个简单的策略是将副本放在不同的机架，可以有效防止一个机架失效时数据的丢失，并且允许读数据的时候充分利用各个机架的带宽，这种策略设置可以将副本均匀分布在集群中，有利于当组件失效的情况下的负载均衡，但是 ，因为这种策略的一个写操作需要传输数据块到多个机架这样增加了写的代价。

多数情况下副本系数时3，HDFS的存放策略时将一个副本存放在本地的机架上，一个副本放在同一机架的另外一个节点上 ，最后一个副本放在不同机架的节点上，这种策略减少了机架间的数据传输，提高了写操作的效率。机架间的错误远比节点的错误少，这种策略提减少了读取数据时需要的网络传输总带宽。这种策略下，副本并不是均匀分布在同一个机架上。三分之一的副本在一个节点，三分之二的副本在一个机架上，其他副本均匀分布在剩下的机架中，这种策略在不损害数据可靠性和可读性能的情况下改进了写的性能。

![F0hvvR.png](https://s1.ax1x.com/2018/12/17/F0hvvR.png)



### 分配原理

- 有了机架感知，NameNode就可以画出下图所示的datanode网络拓扑图,

![F04CVK.png](https://s1.ax1x.com/2018/12/17/F04CVK.png)

- 最底层是Hx是 datanode, 则H1的rackid=/D1/R1/H1，H1的parent是R1，R1的是D1，有了这些rackid信息就可以计算出任意两台datanode之间的距离

```
distance(/D1/R1/H1,/D1/R1/H1)=0  相同的datanode
distance(/D1/R1/H1,/D1/R1/H2)=2  同一rack下的不同datanode
distance(/D1/R1/H1,/D1/R1/H4)=4  同一IDC下的不同datanode
distance(/D1/R1/H1,/D2/R3/H7)=6  不同IDC下的datanode
```

- 写文件时根据策略输入 dn 节点列表，读文件时按与client由近到远距离返回 dn 列表

### 文件读取

1. HDFS Client 通过FileSystem对象的Open方法打开要读取的文件。

2. DistributeFileSystem负责向远程的元数据（NameNode）发起RPC调用，得到文件的数据信息。返回数据块列表，对于每个数据块，NameNode返回该数据块的DataNode地址。

3. DistributeFileSystem返回一个 FSDataInputSteam对象给客户端，客户端调用FSdataInputSteam对象的read()方法开始读取数据。

4. 通过对数据流反复调用read()方法，把数据从数据节点传输到客户端。

5. 当数据读取完毕时，DFSInputSteam对象会关闭此数据节点的链接，连接此文件下一个数据块的最近数据节点。

6. 当客户端读取完数据时，调用FSDataInputSteam对象的close()关闭输入流。

    ![FB9AJJ.md.png](https://s1.ax1x.com/2018/12/18/FB9AJJ.md.png)

#### API

| 方法名                                         | 返回值 | 说明                                                         |
| ---------------------------------------------- | ------ | ------------------------------------------------------------ |
| read(ByteBuffer buf)                           | int    | 读取数据放到buf缓冲区,返回所读取的字节数。                   |
| read(long pos,byte[] buf, int offset ,int len) | int    | 输入流的指定位置开始把数据读取到缓冲区中，pos指定从 输入文件读取的位置，offset数据写入缓冲区位置的（偏移量）len读操作的最大的字节数。 |
| readFully(long pos,byte[] buff[])              | void   | 从指定位置，读取所有数据到缓冲区                             |
| seek(long set)                                 | void   | 指向输入流的第offset字节                                     |
| relaseBuffer(ByteBuffer buff)                  | void   | 删除                                                         |

#### 代码

```java
package hdfs;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.net.URI;


public class DataInputStream {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(new URI("hdfs://192.168.1.101:9000"),conf,"hadoop");
        Path src = new Path("/input/test");
        FSDataInputStream dis = fs.open(src);
        String str = dis.readLine();
        while (str.length() > 0) {
            System.out.println(str);
            str = dis.readLine();
            if (str == null) break;
        }
        dis.close();
    }
}

```

### 文件写入

1. 客户端调用DistributedFileSystem对象的create方法创建一个文件输出流对象
2. DistributedFileSystem向远程的NameNode系欸但发起一次RPC调用，NameNode检查该文件是否存在，如果存在将会覆盖写入，以及客户端是否有权限新建文件。
3. 客户端调用的FSDataOutputStream对象的write()方法写数据，首先数据先被写入到缓冲区，在被切分为一个个数据包。
4. 每个数据包发送到用NameNode节点分配的一组数据节点的一个节点上，在这组数据节点组成的管线上依次传输数据包。
5. 管线上的数据接按节点顺序反向发挥确认信息(ack)最终由管线中的第一个数据节点将整条管线确认信息发给客户端。

![FBPdPg.md.png](https://s1.ax1x.com/2018/12/18/FBPdPg.md.png)

#### 代码

```Java
package hdfs;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.net.URI;

public class DataOutputStream {
    public static void main(String[] args) throws Exception {

        Configuration conf = new Configuration();
        //设置 hdfs 地址 conf  用户名称
        FileSystem fs = FileSystem.get(new URI("hdfs://192.168.1.101:9000"), conf, "hadoop");

        //文件路径
        Path path = new Path("/input/write.txt");

        //字符
        byte[] buff = "hello world".getBytes();
        FSDataOutputStream dos = fs.create(path);
        dos.write(buff);
        dos.close();
        fs.close();

    }
}
```

## 数据容错

### 数据节点

每个DataNode节点定期向NameNode发送心跳信号，网络割裂会导致DataNode和NameNode失去联系，NameNode通过心跳信号的缺失来检测DataNode是否宕机。当DataNode宕机时不再将新的I/O请求发给他们。DataNode的宕机会导致数据块的副本低于指定值，NameNode不断检测这些需要复制的数据块，一旦发现低于设定副本数就启动复制操作。在某个DataNode节点失效，或者文件的副本系数增大时都可能需要重新复制。

### 名称节点

名称节点保存了所有的元数据信息，其中最核心的两大数据时fsimage和edits.logs，如果这两个文件损坏，那么整个HDFS实例失效。Hadoop采用了两种机制来确保名称节点安全。

1. 把名称节点上的元数据同步到其他的文件系统比如（远程挂载的网络文件系统NFS）
2.  运行一个第二名称节点SecondaryNameNode当名称节点宕机后，可以使用第二名称节点元数据进行数据恢复，但会丢失一部分数据。

因此会把两种方式结合起来一起使用，当名称节点发生宕机的时候，首先到远程挂载的网络文件系统中获取备份的元数据信息，放到第二名称节点上进行恢复并把第二名称节点作为名称节点来使用。

### 数据出错

某个DataNode获取的数据块可能是损坏的，损坏可能是由DataNode的存储设备错误，网络错误或者软件bug造成的。HDFS使用校验和判断数据块是否损坏。当客户端创建一个新的HDFS会计算这个每个数据块的校验和。并将校验和作为一个单独的隐藏文件保存在同一个HDFS命名空间下。当客户端获取文件内容后，它会检验从DataNode获取的数据和对应的校验是否匹配，如果不匹配客户端从其他Datanode获取该数据块的副本。HDFS的每个DataNode还保存了检查校验和日志，客户端的每一次检验都会记录到日志中。









