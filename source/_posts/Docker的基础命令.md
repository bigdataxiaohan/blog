---
title: Docker的基础命令
date: 2019-04-05 09:58:06
tags: Docker
categories: 容器
---

## docker

```shell
[root@localhost ~]# docker

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit
 
Management Commands:     #管理命令
  builder     Manage builds
  config      Manage Docker configs  #管理配置
  container   Manage containers      #管理容器 
  engine      Manage the docker engine   # 管理引擎
  image       Manage images               #管理插件
  network     Manage networks
  node        Manage Swarm nodes            #管理节点
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

## docker 兼容了新旧的使用格式 建议使用分组管理.

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

 ## 查看版本

```shell
[root@localhost ~]# docker version
Client:              #客户端
 Version:           18.09.4    #docker版本
 API version:       1.39     #Docker接口版本
 Go version:        go1.10.8  # Docker Go版本
 Git commit:        d14af54266
 Built:             Wed Mar 27 18:34:51 2019
 OS/Arch:           linux/amd64  #  系统
 Experimental:      false        # 是否为企业版
 
Server: Docker Engine - Community   
 Engine:
  Version:          18.09.4
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       d14af54
  Built:            Wed Mar 27 18:04:46 2019
  OS/Arch:          linux/amd64
  Experimental:     false
You have new mail in /var/spool/mail/root
```

## 详细信息

```shell
[root@localhost ~]# docker info
Containers: 0   #容器个数
 Running: 0    # 运行
 Paused: 0    # 暂停
 Stopped: 0   # 停止
Images: 0      # 镜像
Server Version: 18.09.4  #服务器版本
Storage Driver: overlay2  #存储驱动后端 很重要  分层构建和联合挂载 需要专门的文件驱动在之前可能是使用的DM 性能奇差
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:         #  插件
 Volume: local         #卷
 Network: bridge host macvlan null overlay  # 网络插件
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog  #日志插件
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: bb71b10fd8f58240ca47fbb579b9d1028eea7c84
runc version: 2b18fe1d885ee5083ef9f0838fee39b62d653e30
init version: fec3683
Security Options:  #安全选项
 seccomp     
  Profile: default  #默认选项
Kernel Version: 3.10.0-957.10.1.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 3.683GiB
Name: localhost.localdomain
ID: BUTI:XQW2:K5UQ:SNVB:EC67:GBOV:LHZD:FI2G:UFL2:TU4H:FSCN:PQAU
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 https://registry.docker-cn.com/  #加速镜像配置
Live Restore Enabled: false
Product License: Community Engine
```

##  Docker Search

搜索 Docker hub 中的 nginx 镜像

```shell
docker search nginx
```

![ARtYjA.png](https://s2.ax1x.com/2019/04/05/ARtYjA.png)

## Docker pull

多个层级每一层分别下载

![ARtaHP.png](https://s2.ax1x.com/2019/04/05/ARtaHP.png)

## docker images

last表示时最新的

```shell
[root@localhost docker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              2bcb04bdb83f        9 days ago          109MB
[root@localhost docker]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              2bcb04bdb83f        9 days ago          109MB
```

##  docker 卸载镜像

```shell
[root@localhost docker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              af2f74c517aa        2 days ago          1.2MB
nginx               latest              2bcb04bdb83f        9 days ago          109MB
[root@localhost docker]# docker image rm busybox     #第一种命令
Untagged: busybox:latest 
Untagged: busybox@sha256:f79f7a10302c402c052973e3fa42be0344ae6453245669783a9e16da3d56d5b4
Deleted: sha256:af2f74c517aac1d26793a6ed05ff45b299a037e1a9eefeae5eacda133e70a825
Deleted: sha256:0b97b1c81a3200e9eeb87f17a5d25a50791a16fa08fc41eb94ad15f26516ccea
[root@localhost docker]# docker rmi nginx      #第二种命令
Untagged: nginx:latest
Untagged: nginx@sha256:dabecc7dece2fff98fb00add2f0b525b7cd4a2cacddcc27ea4a15a7922ea47ea
Deleted: sha256:2bcb04bdb83f7c5dc30f0edaca1609a716bda1c7d2244d4f5fbbdfef33da366c
Deleted: sha256:dfce9ec5eeabad339cf90fce93b20f179926d5819359141e49e0006a52c066ca
Deleted: sha256:166d13b0f0cb542034a2aef1c034ee2271e1d6aaee4490f749e72d1c04449c5b
Deleted: sha256:5dacd731af1b0386ead06c8b1feff9f65d9e0bdfec032d2cd0bc03690698feda
[root@localhost docker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@localhost docker]#

```

## Docker 创建一个容器

```shell
[root@localhost docker]# docker container

Usage:  docker container COMMAND

Manage containers

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  inspect     Display detailed information on one or more containers
  kill        Kill one or more running containers
  logs        Fetch the logs of a container
  ls          List containers
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  prune       Remove all stopped containers
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  run         Run a command in a new container
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker container COMMAND --help' for more information on a command.
```

```shell
#启动一个nginx容器
[root@localhost ~]# docker run --name nginx-docker1 -d nginx  #-d为后台运行
aaed74bf582677ed39f186a3967f4a4efb11836a164b78f1d0b157fb2f26dcaa
[root@localhost ~]# docker inspect nginx-docker1             #查看docker绑定地址 为172.17.0.2
```

![ARU8OA.png](https://s2.ax1x.com/2019/04/05/ARU8OA.png)

![ARUcT0.png](https://s2.ax1x.com/2019/04/05/ARUcT0.png)

### Redis

即使本地没有Redis的镜像我们依然可以安装。

```shell
docker run --name Redis -d redis:4-alpine
```

进入终端

```shell
docker exec -it kvsotre /bin/sh     #进入容器中
```

![ARdMKH.png](https://s2.ax1x.com/2019/04/05/ARdMKH.png)

## 查看日志

```shell
docker logs kvstore  #产看kvstore的日志
```

![ARd3VI.png](https://s2.ax1x.com/2019/04/05/ARd3VI.png)

## 常用命令

![ARddMQ.png](https://s2.ax1x.com/2019/04/05/ARddMQ.png)

## 镜像生成途径

![AR0fER.png](https://s2.ax1x.com/2019/04/05/AR0fER.png)

### 基于容器

1.创建一个busybox的容器

```shell
[root@localhost ~]# docker run --name b1 -it busybox
/ # mkdir -p /data/html
/ # vi /data/html/index.html
<h1>Busybox httpd Server </h1>
```

```shell
[root@localhost ~]# docker commit  -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message   
  -p, --pause            Pause container during commit (default true) #暂停镜像文件
```

![ARIFLn.png](https://s2.ax1x.com/2019/04/05/ARIFLn.png)

为了引用方便我们可以给镜像打上标签.

```shell
[root@localhost ~]# docker tag --help

Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```

![ARIz01.png](https://s2.ax1x.com/2019/04/05/ARIz01.png)

![ARo11g.png](https://s2.ax1x.com/2019/04/05/ARo11g.png)

 ![ARo2Ax.png](https://s2.ax1x.com/2019/04/05/ARo2Ax.png)

```json
docker inspect busybox
"Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"sh\"]"
            ],
```

 ![ARTFU0.png](https://s2.ax1x.com/2019/04/05/ARTFU0.png)

docker运行镜像生成了原先保存的镜像文件。

```shell
[root@localhost ~]# docker commit -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")  # 附加作者信息
  -c, --change list      Apply Dockerfile instruction to the created image  #修改原有基础镜像的命令
  -m, --message string   Commit message    # 提交信息
  -p, --pause            Pause container during commit (default true)  # 暂停容器
```

```shell
docker commit -a "bigdataxiaohan <467008580@qq.com>" -c 'CMD ["/bin/httpd","-f","-h","/data/html"]' -p t1 hphblog/httpd:v0.2
```

![ARTsG8.png](https://s2.ax1x.com/2019/04/05/ARTsG8.png)

启动基于v0.2版本的服务

![ART2rj.png](https://s2.ax1x.com/2019/04/05/ART2rj.png)

![ARTfZn.png](https://s2.ax1x.com/2019/04/05/ARTfZn.png)

dcoker服务可以直接访问。

由于将docker镜像推导Docker hub 需要相同的组织类名 而我在DockerHub 上创建的仓库为bigdataxiaohan/httpd 我们把镜像更改以下名称

```shell
docker tag hphblog/httpd:v0.2 bigdataxiaohan/httd:v0.2
```

然后登录

```shell
[root@localhost ~]# docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
```

![AR7wy4.png](https://s2.ax1x.com/2019/04/05/AR7wy4.png)

网络不是很好建议多尝试几次。

```shell
docker tag hphblog/httpd:v0.2 bigdataxiaohan/httpd:v0.2
```

(⊙﹏⊙) 还是放弃吧  我们可以使用腾讯云的Docker镜像仓库

![ARjd4s.png](https://s2.ax1x.com/2019/04/05/ARjd4s.png)

![ARjhCR.png](https://s2.ax1x.com/2019/04/05/ARjhCR.png)

![ARvla4.png](https://s2.ax1x.com/2019/04/05/ARvla4.png)

哎呀妈呀QAQ 终于push成功了

![ARv8i9.png](https://s2.ax1x.com/2019/04/05/ARv8i9.png)

![ARvJR1.png](https://s2.ax1x.com/2019/04/05/ARvJR1.png)

#### Docker 打包

 ```shell
[root@localhost ~]# docker  save --help

Usage:  docker save [OPTIONS] IMAGE [IMAGE...]

Save one or more images to a tar archive (streamed to STDOUT by default)

Options:
  -o, --output string   Write to a file, instead of STDOUT
 ```

![ARvTWn.png](https://s2.ax1x.com/2019/04/05/ARvTWn.png)

#### Dokcer加载

```shell
[root@localhost ~]# docker load --help

Usage:  docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output
```

![ARx7Ae.png](https://s2.ax1x.com/2019/04/05/ARx7Ae.png)

![ARxL9A.png](https://s2.ax1x.com/2019/04/05/ARxL9A.png)

```shell
docker load -i myimages.gz
```

![ARxjjP.png](https://s2.ax1x.com/2019/04/05/ARxjjP.png)

![ARzSHS.png](https://s2.ax1x.com/2019/04/05/ARzSHS.png)



