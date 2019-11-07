---
title: Docker学习(二)——Docker架构
categories: [Coding, Docker]
tags:
  - Docker
id: docker-architect
date: 2017-11-03 13:53:16
description: Docker详解
keywords: Docker,架构,教程
---

<blockquote class="blockquote-center">像面向对象编程一样理解Docker镜像与容器</blockquote>

上一篇文章[Docker学习(一)——Docker安装](http://fraserlife.com/2017/11/01/Docker%E5%AD%A6%E4%B9%A0-%E4%B8%80-%E2%80%94%E2%80%94Docker%E5%AE%89%E8%A3%85/#more)讲述了在Ubuntu下安装，安装完了要了解一下Docker架构与一些名词解释，这样对Docker后续的操作会有更好的理解.

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器, 也就是说我们通过Docker客户端来完成对镜像以及容器的一系列操作

<img itemprop="url image" src="/uploads/architecture.jpg" class="full-image" />

对于上面的图片出现的一系列名词可以这样理解：
<!--more-->

| 名词术语      |    解释 |
| :-------- | :--------|
| Docker 镜像(Images)  | Docker 镜像是用于创建 Docker 容器的模板 |
| Docker 容器(Container)    | 容器是独立运行的一个或一组应用 |
| Docker 客户端(Client)      |  Docker 客户端通过命令行或者其他工具使用 Docker API (https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信|
| Docker 主机(Host)    | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器 |
| Docker 仓库(Registry)    | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库，Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用 |

*关于Registry我要多说一句，应该把它理解成管理仓库的仓库，这个仓库中有Ubuntu仓库，Apache仓库，Nginx仓库等， 每个小仓库还都管理不同version的镜像，如Ubuntu1604,Ubuntu1404等.*

文章开头提到**像面向对象编程一样理解Docker镜像与容器**，看如下表格，一个类可以创建多个对象（除了单例模式），也就是说一个镜像可以有多个容器来运行.

| Docker     |    面向对象 |
| :-------- | :--------|
| 镜像	 | 类 |
| 容器    | 对象 |

充分理解一下这篇架构图以及名词解释，就要进入接下来的Docker之旅了.

### 参考
1. [Docker架构](http://www.runoob.com/docker/docker-architecture.html)
2. [几张图帮你理解 docker 基本原理及快速入门](http://www.cnblogs.com/SzeCheng/p/6822905.html)
