---
title: SQLECTRON-超轻量级SQL客户端
date: 2020-07-26 21:36:47
tags:
    - MySQL
categories: [Coding, 数据库-持久层-SQL]
id: SQLECTRON-sql-client
description: SQLECTRON 是一款用 Javascript 来编写的非常轻量级的 SQL client
keywords: SQLECTRON,SQL Client
---

无意间看到这个SQL客户端，瞬间被它简洁的页面吸引了， 启动画面可能是它最复杂的呈现了，爱没？

<fancybox>![](https://rgyb.sunluomeng.top/2020-07-22at10.58.56.gif)</fancybox>



## SQLECTRON

按照[官网](https://sqlectron.github.io/) (https://sqlectron.github.io/,  看 URL 发现，SQLECTRON官网都是用 Github Pages 搭建的) 的说明：

> 一个简单的轻量级SQL客户端桌面/终端，具有跨数据库和跨平台的支持

看到这你应该放心了，无论你使用的是  `Linux`， `Mac` 还是 `Windows`，都可以试一试。那它支持哪些数据库呢？一会到安装界面你就会发现了



这里我用 MAC 演示一下整个使用过程



## 安装与使用

写本文时的版本为 `v1.30.0`, 直接下载安装包——>拖拽, 一步安装完成

<fancybox>![](https://rgyb.sunluomeng.top/20200722103040.png)</fancybox>



添加 Server

<fancybox>![](https://rgyb.sunluomeng.top/2020-07-22at11.23.19.gif)</fancybox>



填写相关信息，从Database Type 中你应该已经看到了，目前支持的数据库类型有：

- MySQL
- PostgreSQL
- Microsoft SQL Server
- SQLite
- Cassandra



<fancybox>![](https://rgyb.sunluomeng.top/20200722104013.png)</fancybox>

测试连接 ——> Save 即可

<fancybox>![](https://rgyb.sunluomeng.top/20200722104034.png)</fancybox>



选择相应的 Server， 然后 Connect，执行个 SQL 试一试

<fancybox>![](https://rgyb.sunluomeng.top/2020-07-22at11.32.40.gif)</fancybox>



获取执行结果后，可以快速粘贴为 JSON 或 CSV 格式，当然也可以导出相应格式文件，非常方便

<fancybox>![](https://rgyb.sunluomeng.top/20200722113741.png)</fancybox>



日常 explain 个 SQL， 画风都不一样了

<fancybox>![](https://rgyb.sunluomeng.top/20200722114118.png)</fancybox>



说它很轻量级，我们和DataGrip 来做个比较**（这么比真是没有人性，DataGrip 的功能有多少怎么不说呢）** 如果不是重度客户端依赖的同学，SQLECTRON 还是满足基本要求的

<fancybox>![](https://rgyb.sunluomeng.top/20200722105551.png)</fancybox>

<fancybox>![](https://rgyb.sunluomeng.top/20200722105628.png)</fancybox>



由于客户端提供的功能并不复杂，所以[快捷键](https://github.com/sqlectron/sqlectron-gui/wiki/Keyboard-Shortcuts) (https://github.com/sqlectron/sqlectron-gui/wiki/Keyboard-Shortcuts)也少的可怜，大家可以自行查阅



如果你更喜欢终端形式，SQLECTRON 还有一个 `SQLECTRON-TERM`  (https://github.com/sqlectron/sqlectron-term) 客户端支持，就像这样，浓浓的 BIOS 风

<fancybox>![](https://rgyb.sunluomeng.top/sqlectron-demo-term-v1.0.0.gif)</fancybox>



只需一条命令安装即可（前提是安装 Node）

```javascript
npm install -g sqlectron-term
```



Bu~~~~~~~~~~~t



先别盲目追逐，这是一个用 Javascript 语言实现的，并且在 github 上的星标并不多

<fancybox>![](https://rgyb.sunluomeng.top/20200722115846.png)</fancybox>

因为 Owner 出于兴趣维护这个项目，但是现在兴趣没了

<fancybox>![](https://rgyb.sunluomeng.top/20200722115950.png)</fancybox>





## 总结

如果你只是做日常的基本 SQL 执行，那么 SQLECTRON 完全可以满足你的需求，你不用再找 DataGrip 或 Navicat 的注册码，同时也不会让电脑发热太多发生卡顿



如果你有兴趣看一看，并且想尝试维护这个项目，这又是一个很好的锻炼机会



