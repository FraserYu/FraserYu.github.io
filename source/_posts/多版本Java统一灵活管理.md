---
title: SDKMAN 统一灵活管理多版本Java
date: 2019-11-14 08:51:02
tags:
    - Java
    - JDK
categories: [Coding, Java]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
id: multiple-java-management
description: 面对快速升级的Java，我们应该灵活的管理多个Java版本，不再去做枯燥的Java环境变量的配置
keywords: sdkman,多版本Java,多版本JDK, Java环境变量
---


近两年，Java 版本升级频繁，感觉刚刚掌握 Java8，写本文时，已听到 java14 的消息，无论是尝鲜新特性([Java12 中超级香的一个功能](https://dayarch.top/p/jdk12-collectors-teeing-api-usage.html))，还是由于项目升级/兼容需要，我们可能都要面临管理多个 Java 版本的情

> 另外 Oracle 自 Java11 开始，更改了用户协议，任何商用都会收费。在写本文时，得到消息「微软宣布加入 OpenJDK」，打不过就选择 OpenJDK。随便 G 一下，当个故事了解就可以

配置单个 Java 环境变量本身没什么技术含量可言，但当需要管理多个 Java 版本，重复配置环境变量显然是非常枯燥的，按照传统的配置方式我们又不能灵活的切换 Java 版本

那要如何轻松管理与使用多个版本 Java？

## 多版本 Java 管理
显然我们不是第一个有这种困境的人，我所知道的现有方案有三种:
1. [Jabba](https://github.com/shyiko/jabba)
2. [jenv](https://github.com/jenv/jenv)
3. [sdkman](https://sdkman.io/)

本文主要说明如何通过 sdkman 打破我们面临的困境，帮助我们灵活配置与使用 Java

## sdkman 介绍
SDKMAN 是一个用于在大多数基于 Unix 系统上管理 **多个软件开发工具包** (Java, Groovy, Scala, Kotlin and Ceylon. Ant, Gradle 等) 的并行版本的工具 。

它提供了一个方便的命令行接口 (CLI) 和 API，用于`安装`、`切换`、`删除`和`列出候选对象`。本文主要通过管理 Java 来说明 sdkman 的使用

## sdkman 安装
在类 unix 平台上安装 sdkman 非常容易。它可以顺利的安装在Mac OSX、Linux、WLS、Cygwin、Solaris和FreeBSD 上，同时还支持Bash和 ZSH shell。

只需打开一个新终端机并输入:
```shell
$ curl -s "https://get.sdkman.io" | bash
```

按照相应的指令提示，完成相应的操作后继续输入:
```shell
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

到这里我们就可以验证 sdk 的安装版本了:
```shell
$ sdk version
```

![sdk version](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_14-45-11.jpg)

上图红色框标记显示我当前 sdkman 的版本，每次执行 `sdk version` 命令时，都会检查是否会有新版本，如果要更新输入 `y` 就可以

有些系统发行版本不包含 `zip` 和 `unzip`，如果安装时遇到相关错误，可以输入如下命令安装 `zip` 和 `unzip`

```shell
$ sudo apt-get install zip unzip
```

从上面的安装命令上可以看出，sdkman 默认的安装路径是在`$HOME/.sdkman` 下，我们也可以自定义安装路径，只需要指定 `SDKMAN_DIR` 变量值就好了:

```shell
$ export SDKMAN_DIR="/usr/local/sdkman" && curl -s "https://get.sdkman.io" | bash

```

到这里 sdkman 的安装就结束了，我们来看看如何使用

## sdkman 使用教程
命令行下学习一个新玩意当然是查看它的 help 命令，输入:
```shell
$ sdkman help
```
![sdk help](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_15-35-22.jpg)

感觉上图按颜色区分内容后，sdkman 的使用说明也就结束了，我们按照上面的图来详细说明一下使用教程

### sdk list
先来输入:
```
$ sdk list
```
![sdk list](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_16-07-35.jpg)

绿色的标记就是 sdkman 集成的所有可用的 candidate，通过按回车「enter」按键，会看到更多可用 candidate

我们指定 candidate，输入:
```shell
$ sdk list java
```

![sdk list java](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_16-14-36.jpg)

从上图中可以看到所有 java 可用的版本 version，以及标识 indentifier，以及状态 status，我已经安装了 java 12 和 11

有了这些信息做铺垫，我们可以安装任意 sdkman 内置的软件开发包了，继续以 java 为例

### sdk install 
回看 sdkman help 命令的输出，使用 install 命令，我们再安装一个 Java 最新 `13.0.1.j9` 版本

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_16-28-17.jpg)

从上图你可以看出，绿色标记的内容是 list 命令结果中的 version 值，但是报错不可用，输入`indentifier` 编号才能正常下载，这里需要注意

安装完后，status 就会编程 installed 状态

### sdk current
当安装多个版本的 java 时，我们输入下面命令获取当前正在用 candidate 的版本
```shell
$ sdk current java
```

![sdk current java](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_17-53-25.jpg)

### sdk use
了解了当前使用版本，如果我们想切换到其他版本, 可以输入:
```shell
$ sdk use java 12.0.2.j9-adpt
```
注意⚠️: 这里同样是指定的 indentifier 的值
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_17-56-22.jpg)

### sdk default
如果我们想指定某个版本为默认版本，可以输入:
```shell
$ sdk default java jdk1.8.0_162.jdk
```
注意⚠️: 这里同样是指定的 indentifier 的值

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_18-00-41.jpg)


### sdk uninstall
当我们想卸载某个版本可以输入:
```
$ sdk uninstall java 12.0.2.j9-adpt
```
注意⚠️: 这里同样是指定的 indentifier 的值

### sdk upgrade
如果我们想升级某个 candidate，可以输入:
```shell
$ sdk upgrade java
```

### sdk flush
使用 sdkman 时间变长也会慢慢产生很多缓存内容，我们可以输入 
清理广播消息:
```shell
$ sdk flush broadcast
```

清理下载的 sdk 二进制文件(长时间使用后清理，可以节省出很多空间):
```shell
$ sdk flush archives
```

清理临时文件内容:
```shell
$ sdk flush temp

```

到这里 sdkman 的基本使用就已经介绍完了，其实这些命令都不用急，想不起来的时候执行 `sdk help` 来临时查看一下就好

## sdkman 卸载
如果我们不喜欢 sdkman 了，我们也可以轻松的卸载掉它:
```shell
$ tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
$ rm -rf ~/.sdkman
```

最后打开你的 .bashrc、.bash_profile 和/或者 .profile，找到并删除下面这几行。

```shell
#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```

我用的 zshrc，找到 .zshrc 文件删除掉上面内容即可

到这里基于 Unix 系统的，有关 sdkman 的安装，使用及下载都已经介绍完了，可以上手试一试了，相信很多小伙伴用的是 Windows，除了 jenv， sdkman 和 Jabba 都有 windows 用户的解决方案:

## sdkman windows 解决方案
在 sdkman 官网首页同样为 windows 用户提供了解决方案，小伙伴们找到如下位置查看即可

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-13_18-19-21.jpg)

我看了一下过程，也是很简单，由于手头没有 windows 电脑，这个请小伙伴们自行尝试吧，有问题欢迎留言 



## 灵魂追问
1. 你现在用的 Java 版本是多少？
2. 有跟随新版本尝试更多新特性吗？
3. **你的灯还亮着吗？**



