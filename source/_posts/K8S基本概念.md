---
title: K8S基本概念
date: 2024-11-14 14:16:27
tags: K8S
categories: Pod
---

### pod简介

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元，它是一个或多个紧密关联的容器的组合，这些容器共享同一个网络命名空间和存储卷。Pod提供了一个抽象层，它封装了容器在节点上的运行环境，例如存储、网络和运行时环境。

          +------------------ Pod ------------------+
          |                                        |
          |      +----------- Container ---------+ |
          |      |                               | |
          |      |  Nginx                        | |
          |      |  redis                        | |
          |      |  java-web                     | |
          |      |  Pause                        | |
          |      +-------------------------------+ |
          |                                        |
          +----------------------------------------+

### pod作用

- 运行单个容器应用程序。
- 运行多个相关容器应用程序。
- 运行应用程序和sidecar容器，sidecar容器提供支持应用程序所需的其他功能，如日志记录、监视和调试。
- 提供应用程序和其依赖项之间的网络通信。
- 提供应用程序和存储之间的访问。

### pod生命周期

#### Pending

Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。

#### Running

Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。

#### Succeeded

 Pod 中的所有容器都已成功终止，并且不会再重启。

#### Failed

Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止，且未被设置为自动重启。

#### Unknown

因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。

### pause 容器

Pause 容器是一种特殊类型的容器，它的主要作用是充当依赖其他容器的容器，为其他容器提供一个可靠的、隔离的运行环境。 Pause 容器是一种轻量级的容器，它本身不包含任何业务逻辑，只是为其他容器提供一个稳定、可靠的运行环境。Pause 容器的实现基于 Docker 的 pause 镜像，可以在创建其他容器之前将其加载到 Pod 中，以确保 Pod 中的其他容器在 Pause 容器的基础上运行。Pause 容器的主要作用是为其他容器提供生命周期的隔离和协调。它可以帮助管理员确保 Pod 中各个容器的启动顺序和依赖关系，避免容器之间的相互干扰和冲突。同时，Pause 容器还可以为 Pod 中的容器提供一个稳定的网络环境，确保容器的网络连接可靠性。

```yaml
# hellok8s.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.21
```

![image-20241114154509772](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241114154509772.png)

使用docker ps 观察命令

![image-20241114155751015](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241114155751015.png)

查看容器 实际为2个其中有1个名为 pause 的容器，pause 容器，是一个很特殊的容器，它又叫 `infra`容器，是每个 Pod 都会自动创建的容器，它不属于用户自定义的容器。

![image-20241114160034230](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241114160034230.png)

使用inspect 发现改容器使用的是 rancher/mirrored-pause:3.6 的镜像，大小为683kB。

#### pause容器作用

假设现在有一个 Pod，它包含两个容器（A 和 B），K8S 是通过让他们加入（join）另一个第三方容器的 network namespace 实现的共享，而这个第三方容器就是 pause 容器。

<img src="https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/0f338dee2baf60d325436ff284900d19-20241114181349529.png" alt="img" style="zoom:50%;" />

如果没有 pause 容器， A 和 B 要共享网络，要么 A 加入 B 的 network namespace，要么 B 加入 A 的 network namespace， 而无论是谁加入谁，只要 network 的 owner 退出了，该 Pod 里的所有其他容器网络都会立马异常，这显然是不合理的。
反过来，由于 pause 里只有是挂起一个容器，里面没有任何复杂的逻辑，只要不主动杀掉 Pod，pause 都会一直存活，这样一来就能保证在 Pod 运行期间同一 Pod 里的容器网络的稳定。
我们在同一 Pod 里所有容器里看到的网络视图，都是完全一样的，包括网络设备、IP 地址、Mac 地址等等，因为他们其实全是同一份，而这一份都来自于 Pod 第一次创建的这个 Infra container。

#### 模拟创建pod

一个 Pod 从表面上来看至少由一个容器组成，而实际上一个 Pod 至少要有包含两个容器，一个是应用容器，一个是 pause 容器。

```shell
+-----------------------------+             +---------------------------+
|           NODE              |             |        Browser            |
|                             |             |                           |
|   +---------------------+    |             |   http://{NODE_IP}:8888  |
|   |        POD          |    |             |                           |
|   |                     |    |             |                           |
|   |  +---------+        |    |    GET      |   +-------------------+   |
|   |  |  Nginx  |        |    +------------>|   |       HTML        |   |
|   |  +----+----+        |    |             |   +-------------------+   |
|   |       | Proxy       |    |             |                           |
|   |  +----+----+        |    |             +---------------------------+
|   |  |  Ghost  |        |    |
|   |  +----+----+        |    |
|   |       | Port 80     |    |
|   +-------|-------------+    |
|           |                  |
|        +--+--+               |
|        | Pause |             |
|        +-------+             |
|           | Port 8888        |
+-----------|-------------------+
            |
            |
      Access via 
   http://{NODE_IP}:8888
```



1.首先我们先创建一个pause容器

```shell
   docker run -d -p 8888:80 \
        --ipc=shareable \
        --name mock_k8s_pod_pause \
        rancher/mirrored-pause:3.6
```

2.创建nginx 容器,先生成对应的配置文件。

```shell
cat <<EOF >> nginx.conf
error_log stderr;
events { worker_connections  1024; }
http {
    access_log /dev/stdout combined;
    server {
        listen 80 default_server;
        server_name hphblog.cn www.hphblog.cn;
        location / {
            proxy_pass http://127.0.0.1:4000;
        }
    }
}
EOF
```

3.运行对应的nginx容器

```shell
       docker run -d --name mock_k8s_pod_nginx \
        -v `pwd`/nginx.conf:/etc/nginx/nginx.conf \
        --net=container:mock_k8s_pod_pause \
        --ipc=container:mock_k8s_pod_pause \
        --pid=container:mock_k8s_pod_pause \
        nginx:1.21
```

这里的3个参数

- `--net`：指定 nginx 要 join 谁的 network namespace，当然是前面创建的mock_k8s_pod_pause
- `--ipc`：指定 ipc mode， 指定前面创建的mock_k8s_pod_pause
- `--pid`：指定 nginx 要 join 谁的 pid namespace，照旧是前面创建的mock_k8s_pod_pause

运行hexo 相关的docker

```shell
docker run -d  --name=mock_k8s_pod_hexo \
--net=container:mock_k8s_pod_pause \
--ipc=container:mock_k8s_pod_pause \
--pid=container:mock_k8s_pod_pause \
-v /Users/hanpenghui/blog:/app \
bloodstar/hexo
```

到这里我们创建了一个符合 K8S Pod 模型的 “Pod” ，只是它并不由 K8S 进行管理，这个 ”Pod” 由一个 mock_k8s_pod_pause 容器（负责提供可稳定共享的命名空间）和两个共享 mock_k8s_pod_pause 容器命名空间的两个应用容器。

如果没有K8S的Pod，启动一个hexo服务需要手动创建三个容器，当想销毁这个服务时，同样需要删除三个容器。有了 K8S 的 Pod，这三个容器在逻辑上就是一个整体，创建 Pod 就会自动创建三个容器，删除 Pod 就会删除三个容器，从管理上来讲，方便了不少。

### 创建ConfigMap

使用下面的命令创建一个 ConfigMap 对象，存储 nginx.conf 文件。

```shell
kubectl create configmap nginx-config --from-file=nginx.conf
```

查看nginx的配置nginx-config 的配置信息。

![image-20241115110604949](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241115110604949.png)

执行命令创建一个 hexo.yaml 文件

```shell

cat <<EOF > hexo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hexo
  namespace: default
spec:
  containers:
    - image: nginx:1.21
      imagePullPolicy: IfNotPresent
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
          hostPort: 8888
      volumeMounts:
        - mountPath: /etc/nginx/
          name: nginx-config
          readOnly: true
    - image: bloodstar/hexo
      imagePullPolicy: IfNotPresent
      name: hexo
      volumeMounts:
        - mountPath: /app
          name: blog-data
  volumes:
    - name: nginx-config
      configMap:
        name: nginx-config
    - name: blog-data
      hostPath:
        path: /Users/hanpenghui/blog
        type: Directory
EOF
```
执行apply -f 命令生成pod
![image-20241115112319570](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241115112319570.png)

访问192.168.194.24 即可访问hexo blog 服务
![image-20241115113312723](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/k8s/image-20241115113312723.png)

