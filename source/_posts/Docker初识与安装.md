---
title: Docker初识与安装
date: 2019-04-04 19:48:24
tags: Docker
categories: 容器
---

## 简介

Docker就是虚拟化的一种轻量级替代技术。Docker的容器技术不依赖任何语言、框架或系统，可以将App变成一种标准化的、可移植的、自管理的组件，并脱离服务器硬件在任何主流系统中开发、调试和运行。

通俗来说Docker是在Linux系统上迅速创建一个容器（类似虚拟机）并在容器上部署和运行应用程序，通过配置文件可以轻松实现应用程序的自动化安装、部署和升级，非常方便。因为使用了容器，所以可以很方便的把生产环境和开发环境分开，互不影响。

![A2IMcj.png](https://s2.ax1x.com/2019/04/04/A2IMcj.png)

## Docker的基本概念

Docker是开发人员和系统管理员使用容器开发，部署和运行应用程序的平台。 使用Linux容器部署应用程序称为容器化。 容器不是新生的概念，但它们可以用于轻松部署应用程序。

容器化越来越受欢迎，因为容器是：

灵活：即使是最复杂的应用也可以容器化化。

轻量级：容器利用并共享主机内核。 

可互换：您可以即时部署更新和升级。 

便携式：您可以在本地构建，部署到云，并在任何地方运行。 

可扩展：您可以增加并自动分发容器副本。 

可堆叠：您可以垂直和即时堆叠服务。

![A2OPmQ.png](https://s2.ax1x.com/2019/04/04/A2OPmQ.png)

### Image

- Docker Image是一个极度精简版的Linux程序运行环境，vi这种基本的工具没有，官网的Java镜像包括的东西更少，除非是镜像叠加方式的， 如Centos+Java7。
- Docker Image是需要定制化Build的一个“安装包”，包括基础镜像+应用的二进制部署包 
- Docker Image内不建议有运行期需要修改的配置文件 
- Dockerfile用来创建一个自定义的image,包含了用户指定的软件依赖等。 当前目录下包含Dockerfile,使用命令build来创建新的image 
- Docker Image的最佳实践之一是尽量重用和使用网上公开的基础镜像 

![ARdOQe.png](https://s2.ax1x.com/2019/04/05/ARdOQe.png)

![ARw9FP.png](https://s2.ax1x.com/2019/04/05/ARw9FP.png)

### Container

- Docker Container是Image的实例，共享内核。 
- Docker Container里可以运行不同Os的Image，比如Ubuntu的或者 Centos 。
- Docker Container不建议内部开启一个SSHD服务，1.3版本后新增了 docker exec命令进入容器排查问题。 
-  Docker Container没有IP地址，通常不会有服务端口暴露，是一个封闭的 “盒子/沙箱”。

通过运行镜像启动容器。 镜像是一个可执行包，包含运行应用程序所需的所有内容代码，库、环境变量和配置文件。 容器是镜像的运行时实例，镜像在执行时在内存中变为什么（有状态的镜像或用户进程）。 您可以使用命令docker ps查看正在运行的容器列表，就像在Linux中一样。

容器和虚拟机：

容器在Linux上本机运行，并与其他容器共享主机的内核。它运行一个独立的进程，不占用任何其他可执行文件的内存，使其轻量级。

相比之下，虚拟机（VM）运行一个完整的“客户”操作系统，通过虚拟机管理程序对主机资源进行虚拟访问。通常，VM提供的环境比大多数应用程序需要的资源更多。

![A2XBKU.png](https://s2.ax1x.com/2019/04/04/A2XBKU.png)



#### 生命周期

![ARN6PO.png](https://s2.ax1x.com/2019/04/05/ARN6PO.png)

[相关资料](https://blog.csdn.net/u010278923/article/details/78751306)

### Daemon

Docker Daemon是创建和运行Container的Linux守护进程，也是Docker 最主要的核心组件。

Docker Daemon 可以理解为Docker Container的Container 。

Docker Daemon可以绑定本地端口并提供Rest API服务，用来远程访问和控制 。

### Registry/Hub

Docker之所以这么吸引人，除了它的新颖的技术外，围绕官方Registry（Docker Hub）的生态圈也是相当吸引人眼球的地方。在Docker Hub上你可以很轻松下载 到大量已经容器化好的应用镜像，即拉即用。这些镜像中，有些是Docker官方维 护的，更多的是众多开发者自发上传分享的。而且你还可以在Docker Hub中绑定你的代码托管系统（目前支持Github和Bitbucket）配置自动生成镜像功能，这样 Docker Hub会在你代码更新时自动生成对应的Docker镜像。 

![ARwymd.png](https://s2.ax1x.com/2019/04/05/ARwymd.png)

![ARwqkq.png](https://s2.ax1x.com/2019/04/05/ARwqkq.png)

### 组件关系

C/S架构：支持IPV4  IPV6 Unix Socket

![ARe2vD.png](https://s2.ax1x.com/2019/04/04/ARe2vD.png)

容器与镜像是Docker主机上非常重要的部分Docker的images来自于Registry（Docker镜像仓库默认Docket Hub）。

从Docker hub 上pull镜像时默认的时https,docker的镜像仓库在国外，docker-cn加速不是很好，因此我们可是使用阿里云的docker加速。

## 安装Docker

本人使用的是Centos7

### 系统准备

要安装Docker CE，您需要CentOS 7的维护版本。不支持或测试存档版本。 必须启用centos-extras存储库。 默认情况下，此存储库已启用，但如果已将其禁用，则需要重新启用它。 建议使用overlay2存储驱动程序。

### 卸载旧版本

较旧版本的Docker被称为docker或docker-engine。如果已安装这些，请卸载它们以及相关的依赖项。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



如果yum报告没有安装这些软件包，那就没关系。 保留/ var / lib / docker /的内容，包括图像，容器，卷和网络。 Docker CE包现在称为docker-ce。

### 安装Docker CE

在新主机上首次安装Docker CE之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。

安装所需的包。 yum-utils提供yum-config-manager实用程序，devicemapper存储驱动程序需要device-mapper-persistent-data和lvm2。

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

使用以下命令设置稳定存储库。

```shell
sudo yum-config-manager --add-repo   https://download.docker.com/linux/centos/docker-ce.repo
```

安装最新版本的Docker CE和containerd，或者安装特定版本。

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

如果提示接受GPG密钥，请确认指纹符合060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35，如果符合，则接受该指纹。

如果您启用了多个Docker存储库，则无需在yum install或yum update命令中指定版本进行安装或更新，则始终安装尽可能高的版本，这可能不适合您的稳定性需求。

要安装特定版本的Docker CE，请在repo中列出可用版本，然后选择并安装：

```shell
[root@localhost ~]# yum list docker-ce --showduplicates | sort -r
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirrors.tuna.tsinghua.edu.cn
docker-ce.x86_64            3:18.09.4-3.el7                    docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                    @docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                    docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                    docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                    docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                   docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
 * base: mirrors.aliyun.com
Available Packages
```

返回的列表取决于启用的存储库，并且特定于您的CentOS版本（在此示例中以.el7后缀表示）

通过其完全限定的包名称安装特定版本，包名称（docker-ce）加上从第一个冒号（:)开始的版本字符串（第2列），直到第一个连字符，用连字符分隔（ - ）。例如 ` docker-ce-18.09.1`

```
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

Docker已安装但尚未启动。已创建docker组，但未向该组添加任何用户。

由于网络问题下载速度可能会稍微慢一些，可以耐心等待。

### 启动Docker

```shell
 sudo systemctl start docker
```

### 拉取helloworld

```shell
[root@localhost ~]# docker pull hello-world
Using default tag: latest
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

这里出现了错误我们解决下

首先安装

```shell
yum install bind-utils
```

```shell
dig @192.168.1.2 registry-1.docker.io
```

```shell
[root@localhost ~]# dig @192.168.1.2 registry-1.docker.io  #此处填写CDN

; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> @192.168.1.2 registry-1.docker.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11981
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; MBZ: 0005 , udp: 4096
;; QUESTION SECTION:
;registry-1.docker.io.          IN      A

;; ANSWER SECTION:
registry-1.docker.io.   5       IN      A       54.175.43.85
registry-1.docker.io.   5       IN      A       34.197.189.129
registry-1.docker.io.   5       IN      A       34.199.40.84
registry-1.docker.io.   5       IN      A       54.210.105.17
registry-1.docker.io.   5       IN      A       100.24.246.89
registry-1.docker.io.   5       IN      A       34.199.77.19
registry-1.docker.io.   5       IN      A       54.88.231.116
registry-1.docker.io.   5       IN      A       34.201.196.144
```

修改/etc/hosts `docker.io`相关的域名解析到其它可用IP

```shell
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

配置加速

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

重启docker

```shell
service docker restart
```

pull helloworld镜像

```shell
[root@localhost ~]# docker pull hello-world
Using default tag: latest

latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
Status: Downloaded newer image for hello-world:latest
```

![ARZgmj.png](https://s2.ax1x.com/2019/04/04/ARZgmj.png)

此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条信息性消息并退出。

### 便捷安装

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## 推荐安装方法(使用清华镜像)

1.下载清华大学Docker CE的repo文件

```shell
cd /etc/yum.repos.d/
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

2.我们打开文件发现依旧指向的时Docker的地址这里我们修改为清华大学的地址

```shell
:%s@https://download.docker.com/@https://mirrors.tuna.tsinghua.edu.cn/docker-ce/@
```

3.安装Docker CE

```shell
yum install docker-ce
```

## 配置Docker加速器

1.可以使用docker cn加速不过效果不太理想

```shell
vim /etc/docker/daemon.json
```

```json

{            
     "registry-mirrors": ["https://registry.docker-cn.com"]    
 }             
```

2.可以使用阿里云等云服务厂商的加速。

## 卸载Docker CE

卸载Docker包

```shell
sudo yum remove docker-ce
```

主机上的镜像，容器，卷或自定义配置文件不会自动删除。要删除所有镜像，容器和卷：

```shell
sudo rm -rf /var/lib/docker
```



## 核心

![A2I1un.png](https://s2.ax1x.com/2019/04/04/A2I1un.png)

### cgroups

Linux系统中经常有个需求就是希望能限制某个或者某些进程的分配资源。于是就出现了cgroups的概念， cgroup就是controller group ，在这个group中，有分配好的特定比例的cpu时间，IO时间，可用内存大小等。 cgroups是将任意进程进行分组化管理的Linux内核功能。最初由google的工程师提出，后来被整合进Linux内核中。 
cgroups中的 重要概念是“子系统”，也就是资源控制器，每种子系统就是一个资源的分配器，比如cpu子系 统是控制cpu时间分配的。首先挂载子系统，然后才有control group的。比如先挂载memory子系统，然后在 memory子系统中创建一个cgroup节点，在这个节点中，将需要控制的进程id写入，并且将控制的属性写入， 这就完成了内存的资源限制。 
cgroups 被Linux内核支持，有得天独厚的性能优势，发展势头迅猛。在很多领域可以取代虚拟化技术分割资源。 cgroup默认有诸多资源组，可以限制几乎所有服务器上的资源：cpu mem iops,iobandwide,net,device acess等 

资料： [kernel文档](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt)  [infoQ相关](https://www.infoq.cn/article/docker-kernel-knowledge-cgroups-resource-isolation)

### LXC

LXC是Linux containers的简称，是一种基于容器的操作系统层级的虚拟化技术。借助于namespace的隔离机制和cgroup限额功能，LXC提供了一套统一的API和工具来建立和管理container。LXC跟其他操作系统层次的虚拟化技术相比，最大的优势在于LXC被整合进内核，不用单独为内核打补丁。

LXC旨在提供一个共享kernel的 OS级虚拟化方法，在执行时不用重复加载Kernel, 且container的kernel与host 共享，因此可以大大加快container的启动过程，并显著减少内存消耗，容器在提供隔离的同时，还通过共享这些资源节省开销，这意味着容器比真正的虚拟化的开销要小得多。 在实际测试中，基于LXC的虚拟化方法的IO和 CPU性能几乎接近 baremetal 的性能。 

虽然容器所使用的这种类型的隔离总的来说非常强大，然而是不是像运行在hypervisor上的虚拟机那么强壮仍具有争议性。如果内核停止，那么所有的容器就会停止运行。 

 性能方面：LXC>>KVM>>XEN 

内存利用率：LXC>>KVM>>XEN 

隔离程度： XEN>>KVM>>LXC 

[baremetal相关](https://blog.csdn.net/BtB5e6Nsu1g511Eg5XEg/article/details/81091459)

### AUFS

AuFS是一个能透明覆盖一或多个现有文件系统的层状文件系统(类比PhotoShop图层)。 支持将不同目录挂载到同一个虚拟文件系统下，可以把不同的目录联合在一起，组成一个单一的目录。这种是一种虚拟的文件系统，文件系统不用格式化，直接挂载即可。 
Docker最初使用AuFS作为容器的文件系统。当一个进程需要修改一个文件时，AuFS创建该文件的一个副本。 AuFS可以把多层合并成文件系统的单层表示。这个过程称为写入复制（ copy on write ）。 它目前作仍然作为存储后端之一来支持。

aufs的竞争产品时overlayfs，后者在3.18版本开始合并到Linux内核；

docker的分层镜像，除了aufs，docker还支持btrfs，devicemapper和vfs等。在Ubuntu系统下docker默认的时aufs；在Centos7上默认的时devicemapper（ 性能比较差）。

![ARw3y4.png](https://s2.ax1x.com/2019/04/05/ARw3y4.png)



AuFS允许Docker把某些镜像作为容器的基础。例如，你可能有一个可以作为很多不同容器的基础的CentOS 系统镜像。多亏AuFS，只要一个CentOS镜像的副本就够了，这样既节省了存储和内存，也保证更快速的容器部署。 
使用AuFS的另一个好处是Docker的版本容器镜像能力。每个新版本都是一个与之前版本的简单差异改动， 有效地保持镜像文件最小化。但，这也意味着你总是要有一个记录该容器从一个版本到另一个版本改动的审计跟踪。

### App打包

LXC的基础上, Docker额外提供的Feature包括：标准统一的 打包部署运行方案 为了最大化重用Image，加快运行速度，减少内存和磁盘 footprint, Docker container运行时所构造的运行环境，实际 上是由具有依赖关系的多个Layer组成的。例如一个apache的运行环境可能是在基础的rootfs image的基础上，叠加了包含例如Emacs等各种工具的image，再叠加包含apache及其相关依赖library的image，这些image由AUFS文件系统加载合并到统一路径中，以只读的方式存在，最后再叠加加载 一层可写的空白的Layer用作记录对当前运行环境所作的修改。 
有了层级化的Image做基础，理想中，不同的APP就可以既可能的共用底层文件系统，相关依赖工具等，同一个APP的 不同实例也可以实现共用绝大多数数据，进而以copy on write的形式维护自己的那一份修改过的数据等。

![A2HISS.png](https://s2.ax1x.com/2019/04/04/A2HISS.png)

### 生命周期开发模式

Docker正在迅速改变云计算领域的运作规则，并彻底颠覆云技术的发展前景。 从持续集成/持续交付到微服务、开源协作乃至DevOps，Docker一路走来已经给应用程序开发生命周期以及云工程技术实践带来了巨大变革。 

![A2HUd1.png](https://s2.ax1x.com/2019/04/04/A2HUd1.png)

