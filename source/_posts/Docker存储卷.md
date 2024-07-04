---
title: Docker存储卷
date: 2019-04-06 12:17:16
tags: Docker
categories: 容器
---

## 简介

Docker镜像是由多个只读层添加而成，启动容器时，Docker加载值读镜像层并在镜像栈顶部添加一个读写层。

如果运行中的容器修改了现有的一个已经存在的文件，那么该文件将会在读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写曾中该文件的副本所隐层，这就是我们所说的写时复制，Elasticsearch也是用了写时复制。和这个略有不同。

默认情况下，容器不使用任何 volume，此时，容器的数据被保存在容器之内，它只在容器的生命周期内存在，关闭并重启容器，其数据不会受到影响，但是删除Docker容器，数据将会全部丢失。当然，也可以使用 docker commit 命令将它持久化为一个新的镜像。

![AW0cOe.png](https://s2.ax1x.com/2019/04/06/AW0cOe.png)

## 问题

- 存储于联合联合文件系统中，不易于宿主机访问；
- 容器间数据共享不便。
- 删除容器会导致数据全部丢失。

## 卷(volume)

“卷”是容器上的一个或者多个目录，此目录可以绕过联合文件系统，与宿主机上的某目录“绑定(关联)”。

Volume于容器化初始化之时即会创建，由base image提供的卷中的数据会于此期间完成复制。

数据卷可以在容器之间共享和重用。

对数据卷的更改是直接进行的。

更新映像时，不包括对数据卷的更改。

即使删除容器本身，数据卷也会保持

Volume的初衷时独立于容器的生命周期实现数据持久化，因此删除容器的时候既不会删除卷，也不会对哪怕未被引用的卷当作垃圾回收操作。

![AWBmp6.png](https://s2.ax1x.com/2019/04/06/AWBmp6.png)

卷为docker提供了独立于容器的数据管理机制，我们可以把“镜像”看作成为静态的文件，例如“程序”，把卷类比为动态内容，比如“数据”，镜像可以重用，而卷可以共享；

卷实现了“程序(镜像)”和“数据(卷)”分离，以及“程序(镜像)”和“制作镜像的主机”分离，用户制作镜像时无需再考虑镜像运行的容器所在的主机的环境。

![AWgrsP.png](https://s2.ax1x.com/2019/04/06/AWgrsP.png)

考虑到容器应用是需要持久存储数据的，可能是有状态的，如果考虑使用NFS做反向代理是没必要存储数据的，应用可以分为有状态和无状态，有状态是当前这次连接请求处理一定此前的处理是有关联的，无状态是前后处理是没有关联关系的，大多数有状态应用都是数据持久存储的，如mysql,redis有状态应用，在持久存储，如nginx作为反向代理是无状态应用，tomcat可以是有状态的，但是它有可能不需要持久存储数据，因为它的session都是保存在内存中就可以的，会导致节点宕机而丢失。session，如果有必要应该让它持久，这也算是有状态的。

![AWgtaD.png](https://s2.ax1x.com/2019/04/06/AWgtaD.png)

对于Docker来说有两种类型的卷，每种类型都在容器中创业中存在一个挂载点，但其再宿主机上上的位置会有所不同：

Bind mount volume(绑定挂载卷)：宿主机上的路径需要人工指定。容器中的路径需要指定。

Dokcer Managed volume(Docker管理卷)：Docker守护进程在宿主机文件系统中所拥有的一部分中创建托管卷。自动自动

### Docker管理管卷

```shell
[root@localhost ~]# docker run --name b2 -it -v /data  busybox
```

```shell
[root@localhost ~]# docker inspect b2
```

![AWHtds.png](https://s2.ax1x.com/2019/04/06/AWHtds.png)

![AWHDQU.png](https://s2.ax1x.com/2019/04/06/AWHDQU.png)

容器中：

![AWHcw9.png](https://s2.ax1x.com/2019/04/06/AWHcw9.png)

![AWHz6S.png](https://s2.ax1x.com/2019/04/06/AWHz6S.png)

![AWbM79.png](https://s2.ax1x.com/2019/04/06/AWbM79.png)

### 绑定挂载卷

```shell
[root@localhost ~]# docker run --name b2 -it --rm -v /data/volumes/b2:/data  busybox  #运行一个b2名称的busybox的容器 再宿主机上的位置为/data/volumes/b2 再docker容器中的位置为data 
```

```shell
[root@localhost ~]# docker inspect b2
```

![AWby1f.png](https://s2.ax1x.com/2019/04/06/AWby1f.png)

宿主机：

![AWbhAs.png](https://s2.ax1x.com/2019/04/06/AWbhAs.png)

Docker：

![AWb5hq.png](https://s2.ax1x.com/2019/04/06/AWb5hq.png)

Docker查询

![AWbvNR.png](https://s2.ax1x.com/2019/04/06/AWbvNR.png)

![AWqsa9.png](https://s2.ax1x.com/2019/04/06/AWqsa9.png)

![AWLGLD.png](https://s2.ax1x.com/2019/04/06/AWLGLD.png)

![AWLLk9.png](https://s2.ax1x.com/2019/04/06/AWLLk9.png)

### 复制卷设置

当我们希望练习比较紧密的容器放置在一起的时候我们可以使用复制卷设置，让他们共享Network、IPC、UTS还可以共享存储卷。比如 Tomcat和Nginx这种的，所以再以后我们可以专门做一个基础卷。

```shell
[root@localhost ~]# docker run --name infracon -it  -v /data/infracon/volume/:/data/web/html busybox  #创建基础卷
```

```shell
[root@localhost b2]# docker run --name nginx --network container:infracon --volumes-from infracon -it busybox          #复制infracon的卷配置
```

```shell
[root@localhost ~]# docker inspect infracon
```

![AWXBqS.png](https://s2.ax1x.com/2019/04/06/AWXBqS.png)

![AWXhrT.png](https://s2.ax1x.com/2019/04/06/AWXhrT.png)

```shell
[root@localhost ~]# docker inspect nginx
```



![AWXrVg.png](https://s2.ax1x.com/2019/04/06/AWXrVg.png)

这样基础支持的容器我们就创建起来。









































