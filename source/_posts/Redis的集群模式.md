---
title: Redis的集群模式
date: 2018-12-07 21:58:22
tags: Redis
categories:  数据库
---

集群

即使使用哨兵，此时的Redis集群的每个数据库依然存有集群中的所有数据，从而导致集群的总数据存储量受限于可用存储内存最小的数据库节点，形成木桶效应。由于Redis中的所有数据都是基于内存存储，这一问题就尤为突出了尤其是当使用Redis做持久化存储服务使用时。
对Redis进行水平扩容，在旧版Redis中通常使用客户端分片来解决这个问题，即启动多个Redis数据库节点，由客户端决定每个键交由哪个数据库节点存储，下次客户端读取该键时直接到该节点读取。这样可以实现将整个数据分布存储在N个数据库节点中，每个节点只存放总数据量的1/N。但对于需要扩容的场景来说，在客户端分片后，如果想增加更多的节点，就需要对数据进行手工迁移，同时在迁移的过程中为了保证数据的一致性，还需要将集群暂时下线，相对比较复杂。考虑到Redis实例非常轻量的特点，可以采用预分片技术（presharding)来在一定程度上避免此问题，具体来说是在节点部署初期，就提前考虑后的存储规模，建立足够多的实例（如128个节点），初期时数据很少，所以每个节点存储的数据也非常少，但由于节点轻量的特性，数据之外的内存幵销并不大，这使得只需要很少的服务器即可运行这些实例。曰后存储规模扩大后，所要做的不过是将某些实例迁移到其他服务器上，而不需要对所有数据进行重新分片并进行集群下线和数据迁移了。
无论如何，客户端分片终归是有非常多的缺点，比如维护成本高，增加、移除节点较繁琐等。Redis3.0版的一大特性就是支持集群（Cluster,广义的“集群”相区别）功能。集群的特点在于拥有和单机实例同样的性能，同时在网络分区后能够提供一定的可访问性以及对主数据库故障恢复的支持。另外集群支持几乎所有的单机实例支持的命令，对于涉及多键的命令（如MGET),如果每个键都位于同一个节点中，则可以正常支持，否则会提示错误。除此之外集群还有一个限制是只能使用默认的0号数据库，如果执行SELECT切换数据库则会提示错误。
哨兵与集群是两个独立的功能，但从特性来看哨兵可以视为集群的子集，当不需要数据分片或者已经在客户端进行分片的场景下哨兵就足够使用了，但如果需要进行水平扩容，则集群是一个非常好的选择

###  安装步骤

#### 安装gcc

直接用yum install 安装

![F3cN8A.md.png](https://s1.ax1x.com/2018/12/08/F3cN8A.md.png)

#### 解压redis 包

![F324NF.md.png](https://s1.ax1x.com/2018/12/08/F324NF.md.png)

#### make

![F3R0Dx.md.png](https://s1.ax1x.com/2018/12/08/F3R0Dx.md.png)

如果出现   Jemalloc/jemalloc.h：没有那个文件
执行make  distclean之后再make
注意 *Redis Test(可以不用执行)*

#### make install 

执行完make后，跳过Redis test 继续执行make install

![F3Rg8H.png](https://s1.ax1x.com/2018/12/08/F3Rg8H.png)



#### 目录解析

查看默认安装目录：usr/local/bin
Redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何(服务启动起来后执行)
Redis-check-aof：修复有问题的AOF文件
Redis-check-dump：修复有问题的dump.rdb文件
Redis-sentinel：Redis集群使用
redis-server：Redis服务器启动命令
redis-cli：客户端，操作入口

**为了演示方便下面我用配好的集群虚拟机**

首先要解决一个环境问题 点击链接 即可获取 这里就不写了

[链接](https://blog.csdn.net/qq_2439107956/article/details/81089711)

####  创建redis-cluster

 在 /opt 下执行 mkdir redis-cluster/
![F3WpIU.png](https://s1.ax1x.com/2018/12/08/F3WpIU.png)

#### 修改节点配置

注意要修改六个哦 当然也可以偷懒 

开启daemonize yes
cluster-enabled yes
Pid文件名字
指定端口
Log文件名字
Dump.rdb名字
Appendonly 关掉或者换名字

#### star-all脚本

```shell
#!/bin/bash

cd redis01
redis-server redis.conf
cd ..
cd redis02
redis-server redis.conf
cd ..
cd redis03
redis-server redis.conf
cd ..
cd redis04
redis-server redis.conf
cd ..
cd redis05
redis-server redis.conf
cd ..
cd redis06
redis-server redis.conf
cd ..
```



#### shutdown-all 脚本

```shell
#!/bin/bash

redis-cli -p 7001 shutdown
redis-cli -p 7002 shutdown
redis-cli -p 7003 shutdown
redis-cli -p 7004 shutdown
redis-cli -p 7005 shutdown
redis-cli -p 7006 shutdown
```

### 启动节点

![F3W5l9.md.png](https://s1.ax1x.com/2018/12/08/F3W5l9.md.png)

#### 拼接集群

```
在redis 安装目录下  src下执行命令
./redis-trib.rb  create  --replicas  1    127.0.0.1:7001   127.0.0.1:7002    127.0.0.1:7003  127.0.0.1:7004   127.0.0.1:7005 127.0.0.1:7006
```

#### 查看节点情况

![F3fK00.png](https://s1.ax1x.com/2018/12/08/F3fK00.png)

#### 分析

一个集群至少要有三个主节点。选项--replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上。

### 增加节点

redis-trib.rb是使用CLUSTERMEET命令来使每个节点认识集群中的其他节点的，可想而知如果想要向集群中加入新的节点，也需要使用CLUSTERMEET命令实现。加入新节点非常简单，只需要向新节点（以下记作A)发送如下命令即可：
CLUSTERMEETipport
ip和port是集群中任意一个节点的地址和端口号，A接收到客户端发来的命令后，会与该地址和端口号的节点B进行握手，使B将A认作当前集群中的一员。当B与A握手成功后，B会使用Gossip协议将节点A的信息通知给集群中的每一个节点。通过这一方式,即使集群中有多个节点，也只需要选择MEET其中任意一个节点，即可使新节点最终加入整个集群中。

### 插槽分配

````
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
````

•一个Redis集群包含16384个插槽（hashslot），数据库中的每个键都属于这16384个插槽的其中一个，集群使用公式CRC16(key)%16384来计算键key属于哪个槽，其中CRC16(key)语句用于计算键key的CRC16校验和。

#### 算法

代码如下

````c
*Copyright 2001-2010 Georges Menie (www.menie.org)
*Copyright 2010 Salvatore SanfHippo (adapted to Redis coding style)
*All rights reserved.
*Redistribution and use in source and binary forms, with or without
*modification, are permitted provided that the following conditions are met:

	Redistributions of source code must retain the above copyright
	noticethis list of conditions and the following disclaimer.
	Redistributions in binary form must reproduce the above copyright
	notice, this list of conditions and the following disclaimer in the
	documentation and/or other materials provided with the distribution.
	Neither the name of the University of California, Berkeley nor the
	names of its contributors may be used to endorse or promote products
	derived from this software without specific prior written permission.
        
* THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ''AS IS, ， AND ANY
* EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
* WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
* DISCLAIMED. IN NO EVENT SHALL THE REGENTS AND CONTRIBUTORS BE LIABLE FOR ANY
* DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
* (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
* LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
* ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
* (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
* SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

/* CRC16 implementation according to CCITT standards.
* Note by Qantirez: this is actually the XMODEM CRC 16 algorithm, using the
* following parameters:
*Name 			 		:"XMODEM", also known as "ZMODEM", "CRC-16/ACORN”
* Width					:16 bit
* Poly					:1021 (That is actually k^16 + x八12 + x^5 + 1)					
* Initialization		 :0000	
* Reflect Input byte		:False	
* Reflect Output CRC		:False
* Xor constant to output CRC 	: 0000
* Output for n123456789"		: 31C3
*/
staticc0nstuintl6_tcrcl6tab[256]={
0x0000,0x1021,0x2042,0x3063,0x4084,0x50a5,0x60c6,0x70e7,
0x8108,0x9129,0xal4a,0xbl6b,0xcl8c,0xdlad,0xelce,0xflef,
0x1231,0x0210,0x3273,0x2252,0x52b5,0x4294,0x72f7,0x62d6,
0x9339,0x8318,0xb37b,0xa35a,0xd3bd,0xc39c,0xf3ff,0xe3de,
0x2462,0x3443,0x0420,0x1401,0x64e6,0x74c7,0x44a4,0x5485,
0xa56a,0xb54b,0x8528,0x9509,0xe5ee,0xf5cf,0xc5ac,0xd58d,
0x3653,0x2672,0x1611,0x0630,0x76d7,0x66f6,0x5695,0x46b4,
0xb75b,0xa77a,0x9719,0x8738,0xf7df,0xe7fe,0xd79d,0xc7bc,
0x48c4,0x58e5,0x6886,0x78a7,0x0840,0x1861,0x2802,0x3823,
0xc9cc,0xd9ed,0xe98e,0xf9af,0x8948,0x9969,0xa90a,0xb92b,
0x5af5,0x4ad4,0x7ab7,0x6a96,0xla71,0x0a50,0x3a33,0x2al2,
0xdbfd,0xcbdc,0xfbbf,0xeb9e,0x9b79,0x8b58,0xbb3b,0xabla,
0x6ca6,0x7c87,0x4ce4,0x5cc5,0x2c22,0x3c03,0x0c60,0xlc41,
0xedae,0xfd8f,0xcdec,0xddcd,0xad2a,0xbd0b,0x8d68,0x9d49,
0x7e97,0x6eb6,0x5ed5,0x4ef4,0x3el3,0x2e32,0xle51,0x0e70,
0xff9f,0xefbe,0xdfdd,0xcffc,0xbflb,0xaf3a,0x9f59,0x8f78,
0x9188,0x81a9,0xblca,0xaleb,0xdl0c,0xcl2d,0xf14e,0xel6f,
0x1080,0x00al,0x30c2,0x20e3,0x5004,0x4025,0x7046,0x6067,
0x83b9,0x9398,0xa3fb,0xb3da,0xc33d,0xd31c,0xe37f,0xf35e,
0x02bl,0x1290,0x22f3,0x32d2,0x4235,0x5214,0x6277,0x7256,
0xb5ea,0xa5cb,0x95a8,0x8589,0xf56e,0xe54f,0xd52c,0xc50d,
0x34e2,0x24c3,0xl4a0,0x0481,0x7466,0x6447,0x5424,0x4405,
0xa7db,0xb7fa,0x8799,0x97b8,0xe75f,0xf77e,0xc71d,0xd73c,
0x26d3,0x36f2,0x0691,0xl6b0,0x6657,0x7676,0x4615,0x5634,
0xd94c,0xc96d,0xf90e,0xe92f,0x99c8,0x89e9,0xb98a,0xa9ab,
0x5844,0x4865,0x7806,0x6827,0xl8c0,0x08el,0x3882,0x28a3,
0xcb7d,0xdb5c,0xeb3f,0xfble,0x8bf9,0x9bd8,0xabbb,0xbb9a,
0x4a75,0x5a54,0x6a37,0x7al6,0x0afl,0xlad0,0x2ab3,0x3a92,
0xfd2e,0xed0f,0xdd6c,0xcd4d,0xbdaa,0xad8b,0x9de8,0x8dc9,
0x7c26,0x6c07,0x5c64,0x4c45,0x3ca2,0x2c83,0xlce0,0x0ccl,
0xeflf,0xff3e,0xcf5d,0xdf7c,0xaf9b,0xbfba,0x8fd9,0x9ff8,
0x6el7,0x7e36,0x4e55,0x5e74,0x2e93,0x3eb2,0x0edl,0xlef0
};
uintl6_t crcl6(const char *buf, int len) {
	int counter;
	uintl6_t crc = 0;
	for (counter = 0; counter < len; counter++)
		crc = (crc«8) A crcl6tab[ ( (crc»8) A *buf++) &0x00FF];
	return crc;
}
    
````

