---
title: Dokcer网络
date: 2019-04-06 10:46:41
tags: Docker
categories: 容器
---

## Docker网络功能介绍

Docker的网络实现是利用Linux上的网络命名空间、网桥和虚拟网络设备 ( VETH ）等来实现。 默认情况下， Docker安装完成后会创建一个网桥docker0。 Docker中的网络接口默认都是虚拟的网络接口。Docker 容器网络在本地主机和容器内分别创建个虚拟接口，并让它们彼此连通。对于docker0我们可以把它当成虚拟的交换机或者虚拟的网卡。

![AWUige.png](https://s2.ax1x.com/2019/04/06/AWUige.png)

关于命名空间更加详细的内容可以参考[InfoQ](https://www.infoq.cn/article/docker-kernel-knowledge-namespace-resource-isolation)相关内容。

## 创建容器步骤

1. 创建一对虚拟接口，分别放到本地主机和新容器中。
2.  本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如veth526a417。
3.  容器一端放到新容器中，并修改名称为eth0，这个接口只在容器的命名空间可见。
4.  从网桥可用地址段中获取一个空闲地址IP，分配给容器的eth0，并配置默认路由到桥接网卡 veth526a417。 

![AWUKC8.png](https://s2.ax1x.com/2019/04/06/AWUKC8.png)

5. 完成上述步骤之后， 容器就可以使用eth0虚拟网卡来连接其他容器和其他网络。


![AWUEDA.png](https://s2.ax1x.com/2019/04/06/AWUEDA.png)

当Docker 启动时，会自动在主机上创建一个docker0 虚拟网桥，实际上是Linux 的一个bridge，可以理解为一个软件交换机。它会在挂载到它的接口之间进行转发，同时Docker随机分配一个本地未占用的私有网段 （ 在RFCl918中定义 ）中 的一个地址给 docker0接口 。 例如典型的172.17.42.1，掩码为 255.255.0.0。此后启动的容器内的接口也会自动分配一个同一网段 （ 172.17.0.0/16 ）的地址。 当创建一个Docker容器的时候，同时会创建一对veth pair 接口 （ 当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包 ）。 这对接口一端在容器内，即eth0; 另一端在本地，并被挂载到 docker0网桥，名称以 veth开头（ 例如veth526a417） 通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。 此时， Docke 就创建了主机和所有容器之间的一个虚拟共享网络。

![AWaJQe.png](https://s2.ax1x.com/2019/04/06/AWaJQe.png)

由于Docker网络比较复杂,因此本人只学习了其基本原理。

## 案例(Docker安装Tomcat)

```shell
docker pull tomcat  #拉取镜像
docker images|grep tomcat  #查看镜像
```

![AWw6Vs.png](https://s2.ax1x.com/2019/04/06/AWw6Vs.png)

![AWwjxO.png](https://s2.ax1x.com/2019/04/06/AWwjxO.png)

















