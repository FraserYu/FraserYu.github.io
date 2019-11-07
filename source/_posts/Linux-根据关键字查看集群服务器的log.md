---
title: Linux 根据关键字查看集群服务器的log
tags:
  - Linux
categories: [Coding, Linux]
id: linux-search-keyword-script
date: 2017-03-29 15:22:34
---

### 背景
现在项目多数会采用集群部署的形式，进行负载均衡，来分担客户端大量请求的压力，当客户端请求发生异常，异常log只能在某一个服务器节点上产生，当服务器节点比较少时候，我们可以根据log的关键字来单个查找，但是当服务器节点多的时候我们还这么查找真的有些浪费时间而且不准确.

<!-- more -->

### 解决方案
在某一台release 服务器上执行下面脚本即可完成多个节点的log信息的查询
**check_log_in_console.sh**脚本内容如下

	#!/bin/bash

	if [ $# -lt 1 ]; then
	  echo "please provide the content you want to find"
	  echo "usage: $0 [content] [date in yyyymmdd format]"
	  exit 0
	fi

	if [ "$2" == "" ]; then
	  d=`date +%Y%m%d`
	else
	  d=$2
	fi

	s=prd-hybris-pad

	for((i=1;i<=4;i++))
	do
		server=$s$i.bb.com
		echo "============check logs on $server===================="

		ssh app@$server "grep '$1' /data/workspace/hybris/hybris/log/tomcat/console-$d.log"
	        sleep 1
	done

	echo "done"
	#exit 0

这样我们只需要在该文件目录中执行

	./check_log_in_console.sh "你要搜索的关键字"  [回车]

就会去以上节点中四个服务器中去查找你要搜索的关键字，打印出类似如下的log信息：

	============check logs on prd-hybris-pad1.bb.com==========
	============check logs on prd-hybris-pad2.bb.com==========
	============check logs on prd-hybris-pad3.bb.com==========
	============check logs on prd-hybris-pad4.bb.com==========
