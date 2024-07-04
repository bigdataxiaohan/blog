---
title: Redis持久化
date: 2018-12-07 12:03:06
tags: Redis
categories: 数据库
---
### Redi持久化方式

Redis provides a different range of persistence options:

The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.

the AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. Redis is able to rewrite the log on background when it gets too big.

If you wish, you can disable persistence at all, if you want your data to just exist as long as the server is running.

It is possible to combine both AOF and RDB in the same instance. Notice that, in this case, when Redis restarts the AOF file will be used to reconstruct the original dataset since it is guaranteed to be the most complete.
Redis 提供了2个不同形式的持久化方式。
RDB （Redis DataBase）
AOF （Append Of File）

#### RDB 
​	在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

##### 如何执行
​	Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

##### fork
在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“写时复制技术”，一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。这也是为什么推荐使用Linux而不是在windos平台上运行的原因之一.

写时复制策略也保证了在fork的时刻虽然看上去生成了两份内存副本，但实际 上内存的占用量并不会增加一倍。这就意味着当系统内存只有2 GB，而Redis数据库 的内存有1.5GB时，执行fork后内存使用量并不会增加到3 GB (超出物理内存）。 为此需要确保Linux系统允许应用程序申请超过可用内存（物理内存和交换分区）的 空间，方法是在/etc/sysctl • conf 文件加入 vm. overcommit_memory = 1，然后 重启系统或者执行sysctl vm.overcommit_memory=l确保设置生效。 
另外需要注意的是，当进行快照的过程中，如果写入操作较多，造成fork前后 数据差异较大，是会使得内存使用量显著超过实际数据大小的，因为内存中不仅保存 了当前的数据库数据，而且还保存着fork时刻的内存数据。进行内存用量估算时很容易忽略这一问题，造成内存用量超限。

##### 配置文件

###### 文件名称

在redis.conf中配置文件名称，默认为dump.rdb

![F1qxuF.png](https://s1.ax1x.com/2018/12/07/F1qxuF.png)

###### 保存路径

rdb文件的保存路径，也可以修改。默认为Redis启动时命令行所在的目录下
![F1qzB4.png](https://s1.ax1x.com/2018/12/07/F1qzB4.png)

###### 保存策略

![F1LP41.png](https://s1.ax1x.com/2018/12/07/F1LP41.png)



######  手动保存
命令save: 只管保存，其它不管，全部阻塞
save vs bgsave
当Redis无法写入磁盘的话，直接关掉Redis的写操作
![F1LYDg.png](https://s1.ax1x.com/2018/12/07/F1LYDg.png)

###### 压缩
rdbcompression yes
进行rdb保存时，将文件压缩
![F1L526.png](https://s1.ax1x.com/2018/12/07/F1L526.png)

###### 数据校验

![F1LxRP.png](https://s1.ax1x.com/2018/12/07/F1LxRP.png)

在存储快照后，还可以让Redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

###### 备份
​	先通过config
​	get dir  查询rdb文件的目录 

###### 恢复
​	关闭Redis
​	先把备份的文件拷贝到工作目录下
​	启动Redis备份数据会直接加载	 

###### 优点

​	节省磁盘空间,恢复速度快.

###### 缺点

•虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。
•在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

### AOF

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。
AOF默认不开启，需要手动在配置文件中配置.
可以在redis.conf中配置文件名称，默认为 appendonly.aof 
![F1vUyV.png](https://s1.ax1x.com/2018/12/07/F1vUyV.png)
AOF文件的保存路径，同RDB的路径一致。

#### 文件故障备份

AOF的备份机制和性能虽然和RDB不同,但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。

#### AOF文件故障恢复
AOF文件的保存路径，同RDB的路径一致。
如遇到AOF文件损坏，可通过   redis-check-aof  --fix  appendonly.aof   进行恢复

#### 同步频率

始终同步，每次Redis的写入都会立刻记入日志
每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。
把不主动进行同步，把同步时机交给操作系统。
![F1xttH.png](https://s1.ax1x.com/2018/12/07/F1xttH.png)

#### Rewrite

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

#### 重写实现

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。
重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。

![F1x7EF.png](https://s1.ax1x.com/2018/12/07/F1x7EF.png)

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,如果Redis的AOF当前大小>=
base_size+base_size*100%
(默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。

#### 优点

备份机制更稳健，丢失数据概率更低。
可读的日志文本，通过操作AOF稳健，可以处理误操作。

![F1zP4H.png](https://s1.ax1x.com/2018/12/07/F1zP4H.png)

#### 缺点

比起RDB占用更多的磁盘空间。
恢复备份速度要慢。
每次读写都同步的话，有一定的性能压力。
存在个别Bug，造成恢复不能。

### 选择

对数据不敏感，可以选单独用RDB

•不建议单独用 AOF，因为可能会出现Bug。

•如果只是做纯内存缓存，可以都不用。

### 优先级

AOF和RDB同时开启，系统默认取AOF的数据


