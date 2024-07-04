---
title: Git使用
date: 2019-01-11 17:49:04
tags: Git
categories:  工具使用
---
## 简介

​	Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。  Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Torvalds 开始着手开发 Git 是为了作为一种过渡方案来替代 BitKeeper。

## 功能

![FX2NtA.png](https://s2.ax1x.com/2019/01/11/FX2NtA.png)

## 对比

### 集中管理型版本管理

![FX22hn.png](https://s2.ax1x.com/2019/01/11/FX22hn.png)

代表：CVS、VSS、SVN

优点：实现了大部分开发中对版本管理的需求  结构简单，上手容易。

问题：

1、版本管理的服务器一旦崩溃，硬盘损坏，代码如何恢复？

2、程序员上传到服务器的代码要求是完整版本，但是程序员开发过程中想做小版本的管理，以便追溯查询，怎么办？

3、系统正在上线运行，时不时还要修改bug，要增加好几个功能要几个月，如何管理几个版本？

4、如何管理一个分布在世界各地、互不相识的大型开发团队？

## 解决方案

![FX2vjK.png](https://s2.ax1x.com/2019/01/11/FX2vjK.png)

## 安装

![FXRbqS.png](https://s2.ax1x.com/2019/01/11/FXRbqS.png)

![FXRX5j.png](https://s2.ax1x.com/2019/01/11/FXRX5j.png)

![FXRz2q.png](https://s2.ax1x.com/2019/01/11/FXRz2q.png)

![FXWCrT.png](https://s2.ax1x.com/2019/01/11/FXWCrT.png)

![FXWka4.png](https://s2.ax1x.com/2019/01/11/FXWka4.png)

​	选择Git命令的执行环境，这里推荐选择第一个，就是单独用户Git自己的命令行窗口。不推荐和windows的命令行窗口混用。

![FXWmxx.png](https://s2.ax1x.com/2019/01/11/FXWmxx.png)

在“Configuring the line ending conversions”选项中，

第一个选项：如果是跨平台项目，在windows系统安装，选择；

第二个选项：如果是跨平台项目，在Unix系统安装，选择；

第三个选项：非跨平台项目，选择。

![FXWlZD.png](https://s2.ax1x.com/2019/01/11/FXWlZD.png)

![FXWtzt.png](https://s2.ax1x.com/2019/01/11/FXWtzt.png)

![FXWdL8.png](https://s2.ax1x.com/2019/01/11/FXWdL8.png)

![FXfpYd.png](https://s2.ax1x.com/2019/01/11/FXfpYd.png)

Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识。

![FXfW9A.png](https://s2.ax1x.com/2019/01/11/FXfW9A.png)

## 使用

- 工作区(Working Directory):就是你电脑本地硬盘目录
- 本地库(Repository):工作区有个隐藏目录.git，它就是Git的本地版本库
- 暂存区(stage):一般存放在"git目录"下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。

![FXhi4J.png](https://s2.ax1x.com/2019/01/11/FXhi4J.png)

1. 创建项目文件夹设置文件夹属性

任意位置创建空文件夹，作为项目文件夹设置文件夹属性，可查看隐藏文件

2. 创建本地版本仓库

在项目文件夹内右键打开git bash窗口  输入命令:  git  init

![FvJ6HK.png](https://s2.ax1x.com/2019/01/13/FvJ6HK.png)

![FvJf9H.png](https://s2.ax1x.com/2019/01/13/FvJf9H.png)

### 提交文件


| 命令                        | 作用                               |
| --------------------------- | ---------------------------------- |
| git  init                   | 初始化本地仓库                     |
| git add 文件名              | 将添加到暂存区                     |
| git rm  - - cached <文件名> | 删除暂存区的文件                   |
| git rm <文件名>             | 和git没什么关系，就相当于linux命令 |
| git commit                  | 编写注释，完成提交文件到本地库     |
| git commit  –m “注释内容”   | 直接带注释提交(推荐使用)           |

下面进行更新操作

```shell
echo 11111 >> hello.txt
git add hello.txt
git commit -m "update 1"

echo 22222 >> hello.txt
git add hello.txt
git commit -m "update 2"

echo 33333 >> hello.txt
git add hello.txt
git commit -m "update 3"

echo 44444 >> hello.txt
git add hello.txt
git commit -m "update 4"

echo 55555 >> hello.txt
git add hello.txt
git commit -m "update 5"
```

### 查看提交记录

| 命令                               | 功能               |
| ---------------------------------- | ------------------ |
| git log 文件名                     | 查看仓库的历史记录 |
| git log  --pretty=oneline   文件名 | 查看简易信息       |

![FvYTz9.png](https://s2.ax1x.com/2019/01/13/FvYTz9.png)

### 回退历史

| 命令                     | 功能                 |
| ------------------------ | -------------------- |
| git reset  --hard HEAD^  | 回退到上一次提交     |
| git reset  --hard HEAD~n | 回退n次操作          |
| git reset 文件名         | 撤销文件缓存区的状态 |

![Fvtnzj.png](https://s2.ax1x.com/2019/01/13/Fvtnzj.png)

![FvtKQs.png](https://s2.ax1x.com/2019/01/13/FvtKQs.png)

### 版本穿越

| 命令                       | 功能                 |
| -------------------------- | -------------------- |
| git reflog 文件名          | 查看历史记录的版本号 |
| git  reset  --hard  版本号 | 穿越到特定版本       |

![FvtXXn.png](https://s2.ax1x.com/2019/01/13/FvtXXn.png)

### 还原文件

| 命令                 | 功能                                               |
| -------------------- | -------------------------------------------------- |
| git  checkout 文件名 | 仓库中的文件依然存在，所以可以从本地仓库中还原文件 |

### 删除文件

 删除项目文件夹中的文件

git  add 文件名 （而是把上面的操作添加进git）

git  commit, 真正地删除仓库中的文件

•注意：删除只是这一次操作的版本号没有了，其他的都可以恢复。

![FvUcMd.png](https://s2.ax1x.com/2019/01/13/FvUcMd.png)

###  分支管理

#### 创建分支

| 命令                       | 功能     |
| -------------------------- | -------- |
| git    branch  <分支名>    | 创建分支 |
| git    branch  -v 查看分支 | 查看分支 |

![FvaeJO.png](https://s2.ax1x.com/2019/01/13/FvaeJO.png)

#### 切换分支

| 命令                            | 功能                       |
| ------------------------------- | -------------------------- |
| git     checkout  <分支名>      | 创建分支                   |
| git      checkout  –b  <分支名> | 创建分支，切换分支一起完成 |

#### 合并分支

| 命令                   | 功能         |
| ---------------------- | ------------ |
| git  checkout   master | 切换到master |
| git  merge  <分支名>   | 合并分支     |

#### 冲突解决

冲突：冲突一般指同一个文件同一位置的代码，在两种版本合并时版本管理软件无法判断到底应该保留哪个版本，因此会提示该文件发生冲突，需要程序员来手工判断解决冲突。

合并时冲突：程序合并时发生冲突系统会提示**CONFLICT**关键字，命令行后缀会进入**MERGING**状态，表示此时是解决冲突的状态。

![Fv4MeP.png](https://s2.ax1x.com/2019/01/13/Fv4MeP.png)

解决冲突：此时通过git diff 可以找到发生冲突的文件及冲突的内容。

![Fv43FS.png](https://s2.ax1x.com/2019/01/13/Fv43FS.png)

手动解决：

![Fv47SH.png](https://s2.ax1x.com/2019/01/13/Fv47SH.png)

## GitHub使用

GitHub是什么

HUB是一个多端口的转发器，在以HUB为中心设备时，即使网络中某条线路产生了故障，并不影响其它线路的工作。

GitHub是一个Git项目托管网站,主要提供基于Git的版本托管服务

![Fv529g.png](https://s2.ax1x.com/2019/01/13/Fv529g.png)

创建项目,现在私仓也免费了。

![Fv56N8.png](https://s2.ax1x.com/2019/01/13/Fv56N8.png)

### 添加远程地址

• git remote add  远端代号   远端地址 。

• 远端代号 是指远程链接的代号，一般直接用origin作代号，也可以自定义。

•远端地址  默认远程链接的url

•例： git  remote  add  origin  https://github.com/xxxxxx.git

![FvIiCD.png](https://s2.ax1x.com/2019/01/13/FvIiCD.png)

![FvIAvd.png](https://s2.ax1x.com/2019/01/13/FvIAvd.png)

### 克隆项目

![FvIYbq.png](https://s2.ax1x.com/2019/01/13/FvIYbq.png)

从GitHub上克隆（复制）一个项目

git  clone   远端地址   新项目目录名。

• 远端地址 是指远程链接的地址。

•项目目录名  是指为克隆的项目在本地新建的目录名称，可以不填，默认是GitHub的项目名。

•命令执行完后，会自动为这个远端地址建一个名为origin的代号。

•例 git  clone  https://github.com/xxxxxxx.git   文件夹名

### 添加合作伙伴

![FvIsM9.png](https://s2.ax1x.com/2019/01/13/FvIsM9.png)

### 拷贝连接

![Fvony9.png](https://s2.ax1x.com/2019/01/13/Fvony9.png)

![FvoZz4.png](https://s2.ax1x.com/2019/01/13/FvoZz4.png)

![FvouLR.png](https://s2.ax1x.com/2019/01/13/FvouLR.png)

![FvoDFf.png](https://s2.ax1x.com/2019/01/13/FvoDFf.png)

### 更新项目



从GitHub更新项目

•git  pull   远端代号   远端分支名。

• 远端代号 是指远程链接的代号。

•远端分支名是指远端的分支名称，如master。 

例 git pull origin  master

![FvohT0.png](https://s2.ax1x.com/2019/01/13/FvohT0.png)

### 协作冲突

![。FvTU9U.png](https://s2.ax1x.com/2019/01/13/FvTU9U.png)

仍需要手动去解决。



