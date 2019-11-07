---
title: Docker学习(三)——镜像
categories: [Coding, Docker]
tags:
  - Docker
id: docker-image
date: 2017-11-06 14:50:32
description: Docker镜像的创建与上传教程
keywords: Docker,镜像,教程
---

<blockquote class="blockquote-center">镜像是Docker运行容器的前提</blockquote>

上一篇文章[Docker学习(二)——Docker架构](http://fraserlife.com/2017/11/03/Docker%E5%AD%A6%E4%B9%A0-%E4%BA%8C-%E2%80%94%E2%80%94Docker%E6%9E%B6%E6%9E%84/#more)讲述了Docker的基本架构，接下来逐步拆分讲解Docker的核心内容之一——**镜像**.

> Docker 运行容器前需要本地存有对应的镜像，如果镜像不存在，Docker会尝试先从默认（Docker Hub公共注册服务器中的仓库）镜像仓库下载，用户也可以配置使用自己的仓库. 本章内容主要有：*获取镜像，查看镜像信息，搜寻镜像，删除镜像，创建镜像，存储和载入镜像，上传镜像*. 看着内容很多，其实熟悉Git的朋友，分分钟就可以理解上面的这些内容， 操作方式都是相通的, 话不多说，看命令说话.


<!--more-->

### 获取镜像
**语法: docker pull NAME[:TAG]**

1. 从Docker Hub的Ubuntu仓库下载最新版本Ubuntu镜像(如果不指定tag，默认tag是latest,即下载最新版本)
```bash
    $ docker pull ubuntu
```
2. 从Docker Hub的Ubuntu仓库下载指定版本Ubuntu镜像
```bash
    $ docker pull ubuntu:16.04
```
3. 我们前面讲过，默认是从Docker Hub的Registry中下载，所以以上的命令相当于
```bash
    $ docker pull registry.hub.docker.com/ubuntu:latest
    $ docker pull registry.hub.docker.com/ubuntu:16.04
```
4. 从其他注册服务器下载Docker镜像（如DockerPool社区，需要指定完整的注册服务器地址）
```bash
    $ docker pull dl.dockerpool.com:5000/ubuntu
```
5. 详细信息请使用下面命令自行了解
```bash
    $ docker pull --help
```

### 查看镜像信息
**语法：docker images**
通过该命令可以查看本地所有的镜像，说明一下执行该命令之后每列的含义
> 1. REPOSITORY: 来自于哪个仓库，同样是如果没有指定注册服务器的地址，那默认就是Docker Hub的仓库
> 2. TAG：镜像的标签信息，比如latest、16.04
> 3. IMAGE ID：镜像的唯一ID标识
> 4. CREATED:（镜像拉取到本地或者在本地创建镜像的时间）
> 5. VIRTUAL SIZE:镜像的大小
*repository是不同的仓库，tag是标记同一个仓库的不同的镜像*


#### Tag打标记
有时候镜像需要一个自己需要的tag标识，这会我们需要用到docker 的tag命令来方便我们管理镜像
**语法：docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]**
1. 给镜像打上自己的标签
```bash
    $ docker tag dl.dockerpool.com:5000/ubuntu:latest ubuntu:20171106_ubuntu
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker tag --help
```

执行完以上命令再次查看[**docker images**]本地所有镜像，发现我们打了标记的镜像和被打标记的镜像的镜像ID竟然一样.这说明新打上标签的镜像只是指向了同一个镜像文件，只是别名不同，标签在这里就起到了引用或快捷方式的作用.

#### 查看镜像的详细信息
**语法：docker inspect [OPTIONS] NAME|ID [NAME|ID]**
*执行[docker inspect]命令会得到一个Json格式的信息，如果我们需要关注某一个部分，可以使用[-f]参数来使用，镜像ID使用前几个字符就可以，不用输入全部*
1. 查看刚刚打了标记的Ubuntu镜像的详细信息(镜像ID因人而异)
```bash
    $ docker inspect 5506de2b643b
```
2. 查看镜像指定的详细信息
```bash
    $ docker inspect -f {{".Architecture"}} 5506
```
3. 详细信息请使用下面命令自行了解
```bash
    $ docker inspect --help
```

### 搜寻镜像
**语法：docker search [OPTIONS] TERM**
1. 搜寻mysql镜像
```bash
    $ docker search mysql
```
2. 说明一下执行该命令之后每列的含义
> + NAME: 镜像名称
> + DESCRIPTION：竞相描述
> + STARTS：镜像星级（代表受欢迎程度）
> + OFFICIAL:是否为官方创建和维护的
> + AUTOMATED:是否自动创建（自动创建的资源允许用户验证镜像内容以及来源）

### 删除镜像
**语法：docker rmi [OPTIONS] IMAGE [IMAGE...]**, IMAGE可以是镜像标签或者镜像ID，rmi其实就是remove image的缩写.
> 删除镜像这里的规矩稍稍多一点但是遵循以下几个原则，不用背诵以下，会在后面的实践中逐步来验证
> 1. 当本地一个镜像有多个标签时，删除某一个标签不影响其他镜像
> 2. 当本地一个镜像只有一个标签时，删除该标签会彻底删除该镜像
> 3. 当通过镜像ID删除镜像的时候，会逐个删除指向该镜像的标签，最终删除镜像.
> 4. 当有该镜像创建的容器存在时，镜像不会被删除，docker会更换该镜像ID，并且镜像没有名字.
> 5. 当有该镜像创建的容器存在时,可以使用[-f] 参数来强制删除镜像（不建议这样做）
> 6. 当有该镜像创建的容器存在时,应遵循先删除容器，然后再删除镜像

1. 删除mysql镜像
```bash
    $ docker rmi mysql:latest
```

### 创建镜像
创建镜像有三种方式：
1. 基于已有镜像的**容器**创建
2. 基于本地模板导入
3. 基于Dockerfile创建（最常用，此处先了解整个过程，后续章节经常使用的时候自然就会了）

#### 基于已有镜像的容器创建新镜像
**语法：docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]**
1. 启动镜像（通过容器运行Ubuntu16.04, 并且进入bash环境），运行后通过[docker ps -a]来查看容器信息，当下主要关注容器ID
```bash
    $ docker run -it ubuntu:1604 /bin/bash
```
2. 创建test文件并且退出容器
```bash
    touch test
    exit
```
3. 通过运行的容器创建新的镜像,执行成功后会返回新创建的镜像ID信息
```bash
    $ docker commit -m "added a new file" -a "fraser" a925cb test:latest
```
4. 详细信息请使用下面命令自行了解["-m":提交信息,"-a":作者信息,"-p":提交时暂停容器运行]
```bash
    $ docker commit --help
```

#### 基于本地模板导入
可以直接从一个操作系统模板文件导入一个镜像，推荐[OpenVZ](https://download.openvz.org/template/precreated/)提供的模板来创建(个人觉得这个应用的不是很多，了解即可)
1. 通过模板文件导入
```bash
    $ sudo cat ubuntu-16.04-x86_64.tar.gz | docker import - ubuntu:1604
```

#### 基于Dockerfile创建
Dockerfile需要讲解的内容很多，包括里面各个指令的应用，我们会单独拿出一个章节来讲解[Dockerfile](www.baidu.com).

### 存出和载入镜像
#### 存出镜像
**语法：docker save [OPTIONS] IMAGE [IMAGE...]** , 存出镜像到本地文件
1. 存出本地的ubuntu:1604镜像为ubuntu_1604.tar
```bash
    $ docker save -o ubuntu_1604.tar ubuntu:1604
```
2. 详细信息请使用下面命令自行了解
```bash
    $ docker save --help
```

#### 载入镜像
**语法：docker load [OPTIONS]** , 从存出的本地文件中再导入到本地镜像库
1. 从ubuntu_1604.tar文件导入镜像到到本地镜像列表
```bash
    $ docker load --input ubuntu_1604.tar
    或
    $ docker load < ubuntu_1604.tar
```

### 上传镜像
**语法：docker push [OPTIONS] NAME[:TAG]**
用户在DockerHub上完成注册之后，即可上传自制的镜像，但是需要打上带有**user**的标签（注册的用户名）
1. 将本地hello-world镜像加上标签
```bash
    $ docker tag hello-world:latest fraseryu/helloworld:latest
```
2. 查看本地镜像列表可以看到打上标签的镜像
```bash
    $ docker images
```
3. 登录完成DockerHub授权认证，输入刚刚在DockerHub上注册的用户名和密码
```bash
    $ docker login
```
4. push本地订制的镜像到DockerHub
```bash
    $ docker push fraseryu/helloworld:latest
```
5. 登录DockerHub看到我们的镜像已经push成功
<img itemprop="url image" src="/uploads/push.png"/>
