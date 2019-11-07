---
title: Docker学习(一)——Docker安装
categories: [Coding, Docker]
tags:
  - Docker
id: docker-install
date: 2017-11-01 14:04:46
description: Docker安装详细教程
keywords: Docker, Docker安装
---

<blockquote class="blockquote-center">Docker, incredible virtual machine</blockquote>

<img itemprop="url image" src="/uploads/dockerIs.png" class="full-image" />

### 什么是Docker
按照惯例，介绍一些历史，说明一下来龙去脉，至少知道它是怎么来的，又为什么会来，能干什么
> 1. Docker 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 Go 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护
> 2. Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。Redhat 已经在其 RHEL6.5 中集中支持 Docker；Google 也在其 PaaS 产品中广泛应用。
> 3. Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。

<!-- more -->

### Docker与传统虚拟机的对比
通过两张图，来理解一下Docker与传统虚拟机的对比，通过下面两张图能看出来Docker节省了哪些资源即可,其他的好处在后面的学习过程中陆续介绍
<img itemprop="url image" src="/uploads/virtualization.png"/>
<img itemprop="url image" src="/uploads/docker.png"/>

Docker目前分为两个版本
1. Docker Enterprise Edition (Docker EE) 专为企业开发和IT团队设计，用于在大规模生产中构建，运送和运行关键业务应用程序Docker EE集成，认证和支持，为企业提供业界最安全的容器平台，使所有应用程序现代化
2. Docker Community Edition (Docker CE) 是开发人员和小团队的理想选择，希望开始使用Docker并尝试基于容器的应用程序Docker CE可在许多平台上使用，从桌面到云到服务器

### 环境
Docker系列学习主要通过 VirtualBox 在 Ubuntu16.04 中完成安装以及后续进阶课程的学习, Docker版本为DockerCE

### 安装
#### 安装Ubuntu16.04
VirtualBox 安装Ubuntu1604请参考[在virtualbox下安装ubuntu server 16.04](http://blog.csdn.net/liaolu2999/article/details/52081438)

#### 安装docker
按照Docker官网[Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)描述安装DockerCE

##### 检查安装条件
Docker 要求 Ubuntu 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的 Ubuntu 版本是否支持 Docker。
通过 uname -r 命令查看你当前的内核版本
```bash
    $ uname -r
```

##### 卸载旧的版本
老的Docker版本我们叫做**docker**或**docker-engine**, 如果之前安装过，这里需要先卸载, Docker CE package现在叫做docker-ce
```bash
    $ sudo apt-get remove docker docker-engine docker.io
```
##### 搭建Docker仓库
首次在宿主机上安装DockerCE，需要搭建Docker repository,搭建完成后，可以通过repository安装或更新Docker.

1. 更新apt package
```bash
    $ sudo apt-get update
```
2. 允许apt用https方式访问Docker仓库（命令太长可以用 **\\** 符号换行）
```bash
    $ sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      software-properties-common
```
3. 添加Docker的官方GPG密钥
```bash
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
4. 验证fingerprint
**9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88**, 用后八位字符即可
```bash
    $ sudo apt-key fingerprint 0EBFCD88
```
5. 设置稳定版Repository
```bash
    $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
```

##### 安装Docker CE
1. 更新apt索引
```bash
    $ sudo apt-get update
```
2. 安装最新版本的Docker CE
```bash
    $ sudo apt-get install docker-ce
```
3. 生产环境指定Docker版本
生产环境为了稳定性会选择指定版本，而不是Dokcer的最新版本，查看可用版本：
```bash
    $ apt-cache madison docker-ce

    docker-ce | 17.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages  
```
4. 安装指定Docker CE 版本
```bash
    $ sudo apt-get install docker-ce=<VERSION>
```
5. 确认Docker CE 是否安装成功
**仔细阅读docker run hello-world的输出内容，对于理解docker的工作原理有帮助**
```bash
    $ sudo docker run hello-world
    $ sudo docker version
```
6. 启动Docker
```bash
    $ systemctl start docker
```
7. 重新启动Docker
```bash
    $ systemctl restart docker
```

##### Docker命令去掉sudo
默认情况下，我们每次敲docker命令都需要sudo权限，下面的方法可以省去sudo方便很多.
1. 创建docker组
```bash
    $ sudo groupadd docker
```
2. 添加当前用户到docker组
```bash
    $ sudo usermod -aG docker $USER
```
3. 登出，重新登录shell
4. 验证docker命令是否可以运行（注意没有sudo）
```bash
    $ docker run hello-world
```
