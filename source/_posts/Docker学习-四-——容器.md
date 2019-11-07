---
title: Docker学习(四)——容器
categories: [Coding, Docker]
tags:
  - Docker
id: docker-container
date: 2017-11-07 10:45:33
description: Docker容器的创建，启动与退出
keywords: Docker,容器,container,教程
---

<blockquote class="blockquote-center">容器是镜像的一个运行实例，所不同的是，它有额外的可写文件层</blockquote>

上一篇文章[Docker学习(三)——镜像](http://fraserlife.com/2017/11/06/Docker%E5%AD%A6%E4%B9%A0-%E4%B8%89-%E2%80%94%E2%80%94%E9%95%9C%E5%83%8F/#more)介绍了Docker的镜像，
包括 *获取镜像，查看镜像信息，搜寻镜像，删除镜像，创建镜像，存储和载入镜像，上传镜像*，接下来逐步拆分讲解Docker的核心内容之一——**容器**.

<!--more-->

> Docker的容器十分轻量级，用户可以随时创建或删除容器

### 创建容器
#### 单纯创建容器
**语法: docker create [OPTIONS] IMAGE [COMMAND] [ARG...]**
1. 为本地Ubuntu镜像创建容器
```bash
    $ docker create -it ubuntu:latest
```
2. 查看本地启动的容器（注意观察容器的状态是Created,换句话说这个容器是终止状态，没有运行）
```bash
    $ docker ps -a
```
3.  容器信息解释
> + CONTAINER ID： 容器ID
> + IMAGE: 容器所运行镜像的名称
> + COMMAND: 运行容器时的命令
> + CREATED: 容器的创建时间
> + STATUS: 容器当前状态

4. 详细信息请使用下面命令自行了解
```bash
    $ docker create --help
```

#### 启动容器方法一
**语法: docker start [OPTIONS] CONTAINER [CONTAINER...]**
1. 查看刚刚创建的容器ID，通过start命令来启动
```bash
    $ docker start 93f
```

2. 重新查看容器信息（注意当前容器状态是Exited，即容器运行终止）
```bash
    $ docker ps -a
```

3. 详细信息请使用下面命令自行了解
```bash
    $ docker start --help
```

#### 启动容器方法二
创建容器和启动容器有一种更方便的命令来完成上面create 和 start 的操作.
**语法: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]**
1. 为本地Ubuntu镜像在此创建容器并启动
```bash
    $ docker run -it ubuntu:latest
```
3. 详细信息请使用下面命令自行了解（和create是一样的）
```bash
    $ docker run --help
```
#### 退出容器
输入**exit**或者**Ctrl+D**来退出容器，对于所创建的bash容器，当使用exit命令退出之后，该容器就出于终止状态了，这是因为对于Docker容器来说，当运行的应用（此处以bash为例）退出后，容器也就没有继续运行的必要了.

#### 守护态运行
更多的时候，需要让Docker容器在后台以守护态（Daemonized）形式运行，用户可以通过添加**-d** option来实现.
1. 运行容器，每隔一秒打印出hello world
```bash
    $ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
2. 获取容器的输出信息
```bash
    $ docker logs ce90
```
3. 详细信息请使用下面命令自行了解
```bash
    $ docker logs --help
```

### 终止容器
**语法: docker stop [OPTIONS] CONTAINER [CONTAINER...]**
执行上述命令后，终止容器分为两个步骤，先向容器发送一个SIGTERM信号，10秒钟后会再次发送一个SIGKILL信号终止容器
1. 终止上文中启动的容器
```bash
    $ docker stop ce90
```
2. 查看终止的容器
```bash
    $ docker ps -a -q
```
3. 启动容器
```bash
    $ docker start ce90
```
4. 重启容器（将运行中的容器终止，并重新启动）
```bash
    $ docker restart ce90
```
5. 详细信息请使用下面命令自行了解
```bash
    $ docker stop --help
```


### 进入容器
上文中提到使用**-d**参数可以使Docker成守护态运行，但是用户无法看到容器的内部信息，而很多时候我们需要进入到容器内进行操作，Docker也提供了集中方法来实现这个功能
#### docker attach
**语法: docker attach [OPTIONS] CONTAINER**, 运行该命令时，该容器必须是运行状态（UP）,如果没有运行，使用**start**命令，attach命令可以不用指定命令参数，默认执行上次run的命令参数**/bin/bash**, 但是当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示，当某个窗口因为某一个命令阻塞时，其他窗口就无法执行其他操作了，所以不建议使用.
1. 进入刚刚以守护态启动的容器
```bash
    $ docker attach 9d89
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker attach --help
```

#### docker exec
**语法: docker exec [OPTIONS] CONTAINER COMMAND [ARG...]**, 该命令至少需要一个命令参数
1. 进入刚刚一守护态启动的容器
```bash
    $ docker exec -it 9d89 /bin/bash
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker exec --help
```

### 删除容器
**语法: docker rm [OPTIONS] CONTAINER [CONTAINER...]** 删除处于 **终止状态** 的容器，同时可以输入多个容器ID批量删除容器
1. 删除处于终止状态的容器
```bash
    $ docker rm 9d89
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker rm --help
```

### 导入和导出容器
#### 导出容器
导出容器是指导出一个已经创建的容器到一个文件，不管该容器是否是处于运行状态
**语法: docker export [OPTIONS] CONTAINER**
1. 导出9d89容器到ubuntu_export.tar文件
```bash
    $ docker export 9d89 > ubuntu_export.tar
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker export --help
```

#### 导入容器
导出的文件又可以使用**docker import** 命令导入，成为镜像
1. 导入上文中的ubuntu_export.tar文件成为镜像
```bash
    $ cat ubuntu_export.tar | docker import - test/ubuntu:v1.0
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker import --help
```

我们在上一节中讲过载入镜像 **docker load** 命令导入镜像文件到本地镜像库，这节通过 **docker import** 命令来导入一个容器快照到本地镜像库，但是容器快照文件将丢弃所有历史数据和元数据（仅仅保存容器当前的快照状态）， 而镜像文件形式保存完整记录，体积要大，此外，容器快照导入标签可以重新指定标签元数据等，向上文中的test/ubuntu:v1.0
