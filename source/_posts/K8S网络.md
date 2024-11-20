---
title: K8S网络
date: 2024-11-20 16:46:06
tags: K8S
---

### 网络模型

Kubernetes 的网络模型假定了所有 Pod 都在一个可以直接连通的扁平的网络空间中，这在GCE(Google Compute Engine)里面是现成的网络模型，Kubernetes 假定这个网络已经存在。而在私有云里搭建 Kubernetes 集群，就不能假定这个网络已经存在了。我们需要自己实现这个网络假设，将不同节点上的 Docker 容器之间的互相访问先打通，然后运行 Kubernetes.

### 网络模型原则

- 任意节点上的Pod可以在不借助NAT的情况下与任意节点上的任意Pod进行通信；
- 节点上的代理（诸如系统守护进程、kubelet等）可以在不借助NAT的情况下与该节点上的任意Pod进行通信；
- 处于一个节点的主机网络中的Pod可以在不借助NAT的情况下与任意节点上的任意Pod进行通信（当且仅当支持Pod运行在主机网络的平台上，比如Linux等）；
- 不论在Pod内部还是外部，该Pod的IP地址和端口信息都是一致的。

在上述要求得到保证后，Kubernetes网络主要聚焦于两个任务—**IP地址管理和路由**，并致力于解决如下问题：

- 同一个Pod内多个容器之间如何通信
- 同一个Node节点中多个Pod之间如何通信
- 不同Node节点上的多个Pod之间如何通信
- Pod和Service之间如何通信
- Pod和集群外的实体如何通信
- Service和集群外的实体如何通信

### CNI

Kubernetes刚开始仅关注和负责容器编排领域的相关事宜，网络管理并不是它最核心的工作。起初，Kubernetes通过开发Kubenet来实现网络管理功能以提供满足要求的集群网络。Kubenet是一个非常简单、基础的网络插件实现。但它本身并不支持任何跨节点之间的网络通信和网络策略等高级功能，且仅适用于Linux系统，所以Kubernetes试图找到一个更优秀的方案来替代Kubenet。为了解决这个问题，CoreOs公司和Docker各自推出了**CNI**（Container Network Interface）和**CNM** (Container Network Model）规范，CNI以其完善的规范和优雅的设计击败了CNM，并成为了Kubernetes首选的网络插件接口规范。

CNI的基本思想是在容器运行时环境中创建容器时，先创建好网络命名空间（netns），然后调用CNI插件为这个网络命名空间配置网络，之后再启动容器内的进程。CNI通过Json Schema定义了容器运行环境和网络插件之间的接口声明，描述当前容器网络的配置和规范，尝试通过一种普适的方式来实现容器网络的标准化 。它仅专注于在创建容器时分配网络资源（IP、网卡、网段等）和在容器被回收时如何删除网络资源两个方面的能力。`CNI`作为Kubernetes和底层网络之间的一个抽象存在，屏蔽了底层网络实现的细节、实现了Kubernetes和具体网络实现方案的解耦，同时也克服了Kubenet不能实现跨主机容器间的相互通信等不足和短板。

借助 CNI 标准，Kubernetes 可以实现容器网络问题的解决。通过插件化的方式来集成各种网络插件，实现集群内部网络相互通信，只要实现CNI标准中定义的核心接口操作ADD，将容器添加到网络;DEL从网络中删除一个容器;CHECK，检查容器的网络是否符合预期等)。CNI插件通常聚焦在容器到容器的网络通信。

![img](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/K8s-CNI-Structure.png)

![3e21950f-e076-42b4-8f3b-8605e79aee3b.jpg](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/3e21950f-e076-42b4-8f3b-8605e79aee3b.jpg)

CNI的接口并不是指 HTTP，gRPC 这种接口，CNI接口是指对可执行程序的调用(exec)可执行程序,Kubernetes 节点默认的 CNI 插件路径为/opt/cni/bin
