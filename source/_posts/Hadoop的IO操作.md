---
title: Hadoop的I/O操作
date: 2018-12-20 13:54:37
tags: Hadoop
categories: 大数据
---

Hadoop自带的一条原子操作作用域数据I/O操作,其中有一些技术比Hadoop更常用,如数据完整性保持和压缩在处理好几个TB级别的数据集时值得关注.

## 数据完整性

Hadoop用户不希望在存储和处理数据时丢失或损坏任何数据，但是当系统中需要处理数据量达到Hadoop处理极限时，数据被损坏不可避免。

检验数据是否损坏常见的措施是：在第一次数据被引入系统时计算校验和(checksum)并在数据通过一个不可靠的通道进行传输时再次计算校验和，如果再次计算的校验和和原来的校验和不匹配，我们可以认为数据已坏，该技术只是检测无法恢复，因此推荐使用ECC内存（实现错误检查和纠正技术的内存条），当然校验和也有可能是损坏的，但是几率比较低。

最常用的错误检测码时CRC-32（32位循环冗余校验）任何大小的数据计算得到一个32位的整数校验和。HadoopCheckFileSysterm使用CRC-32计算校验和。HDFS用于校验和计算的则是一个更有效的变体CRC-32C。

## HDFS数据完整性

HDFS对写入的数据计算校验和，读取数据时验证，它追对每个由dfs.bytes-per-checksum指定字节的数据计算校验和。默认情况下位512个字节，由于CRC-32是4个字节，所以存储校验和的额外开销低于1%。

datanode负责收到数据后存储该数据及其校验和之前的数据进行验证，它在收到客户端的数据或复制其他datanode的数据时执行这个操作。正在写数据的客户端以及校验和发送到由一系列datanode组成的管线，管线最后一个datanode负责校验和，每个datanode都保存用于验证校验日志和的日志(persistent log of checksum verification)，所以它知道每个数据块的最后一次验证时间，客户端验证一个数据块后，就会告诉这个datanode，datanode由此更新日志，保存这些统计信息对于检测损坏的磁盘很有价值。

不只是客户端读取数据的时候会验证校验和，每个datanode也会有个后台线程运行DataBlockScanner，从而定期验证存储在datanode上的所有数据块。

HDFS是多副本存储，因此可以通过数据副本来修复损坏的数据块，从而获取完好的、新的副本，如果检查到错误，先向Namenode报告这个数据块副本标记为已损坏以及正在尝试读操作的这个datanode抛出ChecksumExcepion，这样它不将客户端请求资源直接发送到这个节点。或者尝试将这个副本复制到其他datanode。之后安排这个数据块的一个副本复制到另外一个datanode，如此以来数据块的复制因子(replication factor)又回到了期望水平。此后一个损坏的数据块副本便被删除了。

在使用open()方法读取文件之前，将false值传递给FileSystem对象的setVeryfyChecksum()方法，既可以禁用校验和验证。在命令解释器中使用-get选项中的-ignoreCrc命令或者使用等价的-copyToLocal命令，也可以达到同样的效果，如果一个损坏的文件需要检查并决定如何处理，这个特性是十分有用的。例如你在删除该文件之前尝试看看是否能够恢复部分数据。

可以hadoop的命令 fs -checksum 来检查一个文件的校验和。

## 压缩

文件压缩有两大好处；减少存储文件所需要的磁盘空间，加速数据在网络和磁盘上的传输，在处理大量数据的时候相当重要。

### 格式

| 压缩格式 | 工具  | 算法    | 文件拓展名 | 可切分    |
| -------- | ----- | ------- | ---------- | --------- |
| DEFLATE  | 无    | DEFLATE | .deflate   | 否        |
| gzip     | gzip  | DEFLATE | .gz        | 否        |
| bzip2    | bzip2 | bzip2   | .bz2       | 是        |
| LZO      | lzop  | LZO     | .lzo       | 是:被索引 |
| LZ4      | 无    | LZ4     | .lz4       | 否        |
| Snappy   | 无    | Snappy  | .snappy    | 否        |

### 性能

```
测试环境:
8 core i7 cpu 
8GB memory
64 bit CentOS
1.4GB Wikipedia Corpus 2-gram text input
```

![FrZhtI.md.png](https://s1.ax1x.com/2018/12/20/FrZhtI.md.png)

可以看出压缩比越高，压缩时间越长，压缩比：Snappy < LZ4 < LZO < GZIP < BZIP2

`gzip: `
优点：压缩比在四种压缩方式中较高；hadoop本身支持，在应用中处理gzip格式的文件就和直接处理文本一样；有hadoop native库；大部分linux系统都自带gzip命令，使用方便。 
缺点：不支持split。

`lzo压缩 `
优点：压缩/解压速度也比较快，合理的压缩率；支持split，是hadoop中最流行的压缩格式；支持hadoop native库；需要在linux系统下自行安装lzop命令，使用方便。 
缺点：压缩率比gzip要低；hadoop本身不支持，需要安装；lzo虽然支持split，但需要对lzo文件建索引，否则hadoop也是会把lzo文件看成一个普通文件（为了支持split需要建索引，需要指定inputformat为lzo格式）。

`snappy压缩 `
优点：压缩速度快；支持hadoop native库。 
缺点：不支持split；压缩比低；hadoop本身不支持，需要安装；linux系统下没有对应的命令。

`bzip2压缩 `
优点：支持split；具有很高的压缩率，比gzip压缩率都高；hadoop本身支持，但不支持native；在linux系统下自带bzip2命令，使用方便。 

缺点：压缩/解压速度慢；不支持native。

## 使用

MapReduce 可以在三个阶段中使用压缩。

输入压缩文件。如果输入的文件是压缩过的，那么在被 MapReduce 读取时，它们会被自动解压。

MapReduce 作业中，对 Map 输出的中间结果集压缩。实现方式如下：

可以在 core-site.xml 文件中配置，代码

```xml
<property>
  	   <name>mapred.compress.map.output</name>
       <value>true</value>
</property>
```

Java代码

```java
conf.setCompressMapOut(true);
conf.setMapOutputCompressorClass(GzipCode.class);
```

最后一行代码指定 Map 输出结果的编码器。

MapReduce 作业中，对 Reduce 输出的最终结果集压。实现方式如下：

可以在 core-site.xml 文件中配置，代码如下

```xml
<property>
  	   <name>mapred.output.compress</name>
       <value>true</value>
</property>
```
可以在 core-site.xml 文件中配置，代码如下

```java
conf.setBoolean(“mapred.output.compress”,true);
conf.setClass(“mapred.output.compression.codec”,GzipCode.class,CompressionCodec.class);
```
最后一行同样指定 Reduce 输出结果的编码器。

## 压缩框架

前面已经提到过关于压缩的使用方式，其中第一种就是将压缩文件直接作为入口参数交给 MapReduce 处理，MapReduce 会自动根据压缩文件的扩展名来自动选择合适解压器处理数据。那么到底是怎么实现的呢？如下图所示：

![FrmJZ4.png](https://s1.ax1x.com/2018/12/20/FrmJZ4.png)

[以下引自](http://www.cnblogs.com/xuxm2007/archive/2012/06/15/2550996.html)[xuxm2007](http://www.cnblogs.com/xuxm2007/archive/2012/06/15/2550996.html):

**CompressionCodec对流进行压缩和解压缩**

CompressionCodec有两个方法可以用于轻松地压缩或解压缩数据。要想对正在被写入一个输出流的数据进行压缩，我们可以使用 createOutputStream(OutputStreamout)方法创建一个CompressionOutputStream（未压缩的数据将 被写到此)，将其以压缩格式写入底层的流。相反，要想对从输入流读取而来的数据进行解压缩，则调用 createInputStream(InputStreamin)函数，从而获得一个CompressionInputStream,，从而从底层的流 读取未压缩的数据。CompressionOutputStream和CompressionInputStream类似干 java.util.zip.DeflaterOutputStream和java.util.zip.DeflaterInputStream，前两者 还可以提供重置其底层压缩和解压缩功能，当把数据流中的section压缩为单独的块时，这比较重要。比如SequenceFile。

**下例中说明了如何使用API来压缩从标谁输入读取的数据及如何将它写到标准输出：**

```java
public class StreamCompressor 
{
     public static void main(String[] args) throws Exception 
     {
          String codecClassname = args[0];
          Class<?> codecClass = Class.forName(codecClassname); // 通过名称找对应的编码/解码器
          Configuration conf = new Configuration();
CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(codecClass, conf);
 // 通过编码/解码器创建对应的输出流
          CompressionOutputStream out = codec.createOutputStream(System.out);
 // 压缩
          IOUtils.copyBytes(System.in, out, 4096, false);
          out.finish();
     }
} 
```

用CompressionCodecFactory方法来推断CompressionCodecs

阅读一个压缩文件时，我们通常可以从其扩展名来推断出它的编码/解码器。以.gz结尾的文件可以用GzipCodec来阅读，如此类推。每个压缩格式的扩展名如第一个表格;

CompressionCodecFactory提供了getCodec()方法，从而将文件扩展名映射到相应的CompressionCodec。此方法接受一个Path对象。下面的例子显示了一个应用程序，此程序便使用这个功能来解压缩文件。

```java
public class FileDecompressor {
    public static void main(String[] args) throws Exception {
       String uri = args[0];
       Configuration conf = new Configuration();
       FileSystem fs = FileSystem.get(URI.create(uri), conf);
       Path inputPath = new Path(uri);
       CompressionCodecFactory factory = new CompressionCodecFactory(conf);
       CompressionCodec codec = factory.getCodec(inputPath);
       if (codec == null) {
           System.err.println("No codec found for " + uri);
           System.exit(1);
       }
       String outputUri =
       CompressionCodecFactory.removeSuffix(uri, codec.getDefaultExtension());
       InputStream in = null;
       OutputStream out = null;
       try {
           in = codec.createInputStream(fs.open(inputPath));
           out = fs.create(new Path(outputUri));
           IOUtils.copyBytes(in, out, conf);
       } finally {
           IOUtils.closeStream(in);
           IOUtils.closeStream(out);
       }
    }
}
```

编码/解码器一旦找到，就会被用来去掉文件名后缀生成输出文件名（通过CompressionCodecFactory的静态方法removeSuffix()来实现）。这样，如下调用程序便把一个名为file.gz的文件解压缩为file文件:

```
% hadoop FileDecompressor file.gz
```

CompressionCodecFactory 从io.compression.codecs配置属性定义的列表中找到编码/解码器。默认情况下，这个列表列出了Hadoop提供的所有编码/解码器 (见表4-3)，如果你有一个希望要注册的编码/解码器(如外部托管的LZO编码/解码器)你可以改变这个列表。每个编码/解码器知道它的默认文件扩展 名，从而使CompressionCodecFactory可以通过搜索这个列表来找到一个给定的扩展名相匹配的编码/解码器(如果有的话)。

| 属性名                | 类型           | 默认值                                                       |
| :-------------------- | -------------- | ------------------------------------------------------------ |
| io.compression.codecs | 逗号分隔的类名 | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, |
|                       |                | org.apache.hadoop.io.com                                     |

**本地库**

考虑到性能，最好使用一个本地库（native library）来压缩和解压。例如，在一个测试中，使用本地gzip压缩库减少了解压时间50%，压缩时间大约减少了10%(与内置的Java实现相比 较)。表4-4展示了Java和本地提供的每个压缩格式的实现。井不是所有的格式都有本地实现(例如bzip2压缩)，而另一些则仅有本地实现（例如 LZO）。

| 压缩格式 | Java实现 | 本地实现 |
| -------- | -------- | -------- |
| DEFLATE  | 是       | 是       |
| gzip     | 是       | 是       |
| bzip2    | 是       | 否       |
| LZO      | 否       | 是       |

Hadoop带有预置的32位和64位Linux的本地压缩库，位于库/本地目录。对于其他平台，需要自己编译库，具体请参见Hadoop的维基百科http://wiki.apache.org/hadoop/NativeHadoop。

本地库通过Java系统属性java.library.path来使用。Hadoop的脚本在bin目录中已经设置好这个属性，但如果不使用该脚本，则需要在应用中设置属性。

默认情况下，Hadoop会在它运行的平台上查找本地库，如果发现就自动加载。这意味着不必更改任何配置设置就可以使用本地库。在某些情况下，可能 希望禁用本地库，比如在调试压缩相关问题的时候。为此，将属性hadoop.native.lib设置为false，即可确保内置的Java等同内置实现 被使用(如果它们可用的话)。

CodecPool(压缩解码池)

如果要用本地库在应用中大量执行压缩解压任务，可以考虑使用CodecPool，从而重用压缩程序和解压缩程序，节约创建这些对象的开销。

下例所用的API只创建了一个很简单的压缩程序，因此不必使用这个池。此应用程序使用一个压缩池程序来压缩从标准输入读入然后将其写入标准愉出的数据：

```java
public class PooledStreamCompressor 
 {
    public static void main(String[] args) throws Exception 
    {
        String codecClassname = args[0];
        Class<?> codecClass = Class.forName(codecClassname);
        Configuration conf = new Configuration();
CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(codecClass, conf);
        Compressor compressor = null;
        try {
compressor = CodecPool.getCompressor(codec);//从缓冲池中为指定的CompressionCodec检索到一个Compressor实例
CompressionOutputStream out = codec.createOutputStream(System.out, compressor);
            IOUtils.copyBytes(System.in, out, 4096, false);
            out.finish();
        } finally
        {
            CodecPool.returnCompressor(compressor);
        }
      }
} 
```

我 们从缓冲池中为指定的CompressionCodec检索到一个Compressor实例，codec的重载方法 createOutputStream()中使用的便是它。通过使用finally块，我们便可确保此压缩程序会被返回缓冲池，即使在复制数据流之间的字 节期间抛出了一个IOException。

**压缩和输入分割**

在考虑如何压缩那些将由MapReduce处理的数据时，考虑压缩格式是否支持分割是很重要的。考虑存储在HDFS中的未压缩的文件，其大小为1GB. HDFS块的大小为128MB ，所以文件将被存储为8块，将此文件用作输入的MapReduce会创建8个输入分片(split，也称为"分块")，每个分片都被作为一个独立map任务的输入单独进行处理。

现在假设，该文件是一个gzip格式的压缩文件，压缩后的大小为1GB。和前面一样，HDFS将此文件存储为 8块。然而，针对每一块创建一个分块是没有用的因为不可能从gzip数据流中的任意点开始读取，map任务也不可能独立于其分块只读取一个分块中的数据。gZlp格式使用DEFLATE来存储压缩过的数据，DEFLATE 将数据作为一系列压缩过的块进行存储。问题是，每块的开始没有指定用户在数据流中任意点定位到下一个块的起始位置，而是其自身与数据流同步。因此，gzip不支持分割(块)机制。在这种情况下，MapReduce不分割gzip格式的文件，因为它知道输入的是gzip格式(通过文件扩展名得知)，而gzip压缩机制不支持分割机制。这样是以牺牲本地化为代价:一个map任务将处理8个HDFS块，大都不是map的本地数据。与此同时，因为map任务少，所以作业分割的粒度不够细，从而导致运行时间变长。

在我们假设的例子中，如果是一个LZO格式的文件，我们会碰到同样的问题，因为基本压缩格式不为reader 提供方法使其与流同步。但是，bzip2格式的压缩文件确实提供了块与块之间的同步标记(一个48位的π近似值) 因此它支持分割机制。对于文件的收集，这些问题会稍有不同。ZIP是存档格式，因此t可以将多个文件合并为一个ZIP文件。每个文单独压缩，所有文档的存储位置存储在ZIP文件的尾部。这个属性表明Z l P 文件支持文件边界处分割，每个分片中包括ZIP压缩文件中的一个或多个文件。

ZIP格式文件的结构如下图：ZIP文件结构请查看[wiki](https://en.wikipedia.org/wiki/Zip_%28file_format%29#File_headers)

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601164323.png)

**在MapReduce 中使用压缩**

如前所述，如果输入的文件是压缩过的.那么在被MapReduce 读取时，它们会被自动解压，根据文件扩展名来决定应该使用哪一个压缩解码器。如果要压缩MapReduce作业的输出.请在作业配置文件中将mapred.output.compress属性设置为true，将mapred.output.compression.codec属性设置为自己打算使用的压缩编码/解码器的类名。

```java
 public class MaxTemperatureWithCompression {
 
        public static void main(String[] args) throws Exception {
            if (args.length != 2) {
                System.err.println("Usage: MaxTemperatureWithCompression <input path> "
                        + "<output path>");
                System.exit(-1);
            }
            Job job = new Job();
            job.setJarByClass(MaxTemperature.class);
            FileInputFormat.addInputPath(job, new Path(args[0]));
            FileOutputFormat.setOutputPath(job, new Path(args[1]));
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(IntWritable.class);
            FileOutputFormat.setCompressOutput(job, true);
            FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class);
            job.setMapperClass(MaxTemperatureMapper.class);
            job.setCombinerClass(MaxTemperatureReducer.class);
            job.setReducerClass(MaxTemperatureReducer.class);
            System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }
```

我们使用压缩过的输入来运行此应用程序(其实不必像它一样使用和输入相同的格式压缩输出)，如下所示:

% hadoop MaxTemperatur eWithCompression input/ ncdc/sample.txt.gz

output

量生终输出的每部分都是压缩过的.在本例中，只有一部分:

% gunzip -c output/part-OOOOO.gz

1949 111

1950 22

如果为输出使用了一系列文件， 可以设置mapred.output.compresson.type 属性来控制压缩类型。默认为RECORD，它压缩单独的记录。将它改为BLOCK，可以压缩一组记录，由于有更好的压缩比，所以推荐使用。

**map 作业输出结果的压缩**

即使MapReduce 应用使用非压缩的数据来读取和写入，我们也可以受益于压缩map阶段的中阔输出。因为map作业的输出会被写入磁盘并通过网络传输到reducer节点，所以如果使用LZO之类的快速压缩，能得到更好的性能，因为传输的数据量大大减少了.表4-5显示了启用map输出压缩和设置压缩格式的配置属性.

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/大数据/Spark/SQL/20190601164358.png)

下面几行代码用于在map 作业中启用gzip 格式来压缩输出结果:

```
Configuration conf = new Configuration();
conf.setBoolean("mapred.compress.map.output", true);
conf.setClass("mapred.map.output.compression.codec", GzipCodec.class,
CompressionCodec.class);
Job job = new Job(conf);
```

旧的API要这样配置:

```java
conf.setCompressMapOutput(true);
conf.setMapOutputCompressorClass(GzipCodec.class);
```

压缩就到此为止了。总之编码和解码在hadoop有着关键的作用。