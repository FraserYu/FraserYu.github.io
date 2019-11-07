---
title: Windows下删除过长文件路径或过长文件名的文件
categories: [Coding, Others]
tags:
  - Windows
  - Node
id: delete-long-name-file-in-windows
date: 2016-09-05 15:28:25
---

在Windows下，删除文件时报错"文件路径太长或者文件名太长"，无论是Shift+Del、进入子目录一点点删除都不可以以及使用Windows内置命令都不可以，但是我电脑中装有node，于是安装**【rimraf】**——是专门用于删除模块的插件
> 官方描述：A deep deletion module for node (like rm -rf)

	npm install -g rimraf

使用方法：
<!-- more -->

1. 进入project目录

		cd project/

2. 删除hybris目录及目录下所有文件

		rimraf hybris

可以看到，hybris目录及其所有子目录及文件全部删除干净，非常爽。

感谢[原作者](http://blog.csdn.net/crper/article/details/50458369)的分享
