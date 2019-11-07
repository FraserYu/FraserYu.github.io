---
title: Jenkins自动化部署Hybris
tags:
  - Jenkins
categories: [Coding, DevOps]
id: jenkins-hybris-deploy
date: 2017-03-08 15:03:17
---

### 前言
项目开发时间不算紧张，知道项目应用Jenkins 持续集成实现自动化部署，这个部署是如何实现的，向架构师取经研究之后做此记录.

### 部署架构拓扑图
以下是项目部署架构简易拓扑图，不包括Solr服务器的部署, Prod Env采用Cluster Deployment的方式.  
<!-- more -->
![部署拓扑图](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_tp.png)

> 运维人员搭建Jenkins服务器之后，每位被授权的开发人员都可以访问Jenkins 工作主页， 当push代码之后，统一在指定时间点build (Jenkins没有采用有push代码自动build的方式 ).
> 1. 开发人员Push 代码
> 2. Build QA1, 功能测试通过之后进行下一步
> 3. Build QA2, 功能测试通过之后进行下一步
> 4. 等待发布版本时间节点
> 5. Build Stage环境，功能测试通过之后进行下一步 （拓扑图中没有显示Stage环境）
> 6. Build Prod 环境， 功能测试通过之后成功发布.

开发人员只需要点击Jenkins控制台的Build，即可完成自动化部署，这正是Jenkins 带来的便利，接下来就解释一下Jenkins 整个工作过程.

### Jenkins 持续集成过程详解
生产环境是采用集群部署，和QA环境是有所区别的，下面针对生产环境和QA环境分别作解释

#### Common 环境解释
Jenkins 的Home Directory 是  

	/var/lib/jenkins

#### QA环境部署
> 以QA1 环境为例，item名称为 **qa_honor_hybris**,  workspace的路径是Jenkins 的workspace路径与item名称的拼接，这样当前item的**$workspace**的路径为

	/var/lib/jenkins/workspace/qa_honor_hybris

1. 点击build之后，Jenkins会按照该item配置好的Git Repository 来拉取最新代码到workspace中
2. 执行自定义Shell 脚本(Jenkins 控制台写好的脚本)

		# 1. 移除掉工作区旧代码
		rm -rf /data/qa_honor_hybris/hybris/bin/custom
		# 2. copy 拉取下来的最新代码到工作区中
		cp -r $WORKSPACE/bin/custom /data/qa_honor_hybris/hybris/bin
		# 3. copy local.properties  和 localextensions.xml 到 工作区config目录下
		cp $WORKSPACE/config/local.properties /data/qa_honor_hybris/hybris/config
		cp $WORKSPACE/config/localextensions.xml /data/qa_honor_hybris/hybris/config
		# 5. 进入工作区platform 目录
		cd /data/qa_honor_hybris/hybris/bin/platform
		# 6. 设置基本环境变量，当然可以Jenkins 设置好
		export JAVA_HOME=/usr/local/jdk1.8
		export JRE_HOME=/usr/local/jdk1.8/jre
		export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
		export PATH=$JAVA_HOME/bin:$PATH
		# 7. 指定ant 目录并且build，执行该步骤是为了检测项目是否能build成功
		. ./setantenv.sh
		ant clean all
		# 8. 切换到工作区build 目录下
		cd /data/qa_honor_hybris/build
		./build_and_deploy.sh

3. **./build_and_deploy.sh** 脚本内容

		#!/bin/bash
		# 1. 进入工作区目录
		cd /var/lib/jenkins/workspace/qa1-honor-hybris
		echo "build for honor hybris qa1"
		# 删除workspace 历史内容
		echo "create honor source code tar ball"
		rm -rf honor
		rm -rf honor.tar.gz
		# 创建文件夹，等待打包
		mkdir honor
		# copy workspace 最新代码拷贝到 刚刚新建的honor 文件夹中
		cp -r /var/lib/jenkins/workspace/qa1-honor-hybris/bin honor
		cp -r /var/lib/jenkins/workspace/qa1-honor-hybris/config/qa honor
		# 将honor 文件夹压缩打包
		tar -zcvf honor.tar.gz honor

		# 将打包的honor.tar.gz 远程copy到 QA1 服务器的workspace 中
		echo "======copy tar ball to qa1 server(10.11.12.233)====="
		scp honor.tar.gz app@10.11.12.233:/data/build_workspace
		# 进入到QA1 环境并且进入workspace, 停掉QA1 的服务
		echo "===build hybris and start hybris on qa1 server==="
		ssh app@10.11.12.233 'cd /data/build_workspace;. ./stop.sh'
		# 执行QA1相应路径下的脚本.
		ssh app@10.11.12.233 'cd /data/build_workspace;. ./build_and_start.sh'

		echo "build_and_deploy done"
		exit 0

4. **./build_and_start.sh** 脚本内容

		#!/bin/bash
		# 1. 删除掉honor 文件夹（确保文件夹删除，下面其实已经删除了一次）
		rm -rf honor
		# 2. 赋予可读可写可执行的权限
		chmod 755 honor.tar.gz
		# 3. 解压刚刚传输过来的压缩包，并赋予可读可写可执行的权限
		tar -zxvf honor.tar.gz
		chmod -R 755 honor

		# 4. 删除掉旧的Hybris代码
		echo "remove old honor hybris source code..."
		rm -rf /data/build_workspace/honor-hybris/hybris/bin/custom
		rm -rf /data/build_workspace/honor-hybris/hybris/config/local.properties
		rm -rf /data/build_workspace/honor-hybris/hybris/config/localextensions.xml

		# 5. copy刚刚解压出来的honor文件夹的内容到工作区
		echo "copy new honor source code..."
		cd /data/build_workspace
		cp -r honor/* honor-hybris/hybris
		cp honor/qa/local_qa1.properties honor-hybris/hybris/config/local.properties
		cp honor/qa/localextensions.xml honor-hybris/hybris/config/localextensions.xml

		# 6. 删除解压出来的文件夹
		rm -rf honor
		# 7. 设置环境变量
		echo "begin to start qa1..."
		export JAVA_HOME="/usr/local/jdk1.8.0_101"
		export PATH=$JAVA_HOME/bin:$PATH
		cd /data/build_workspace/honor-hybris/hybris/bin/platform
		. ./setantenv.sh
		ant clean all
		# 8. 直接以debug模式启动（方便远程调试，但是Jenkins build进度条始终是红色，不影响使用，还没找到这个解决办法）
		. ./hybrisserver.sh debug
		exit 0

以上是QA环境还不涉及到多node的情况，接下来会介绍Prod 的情况，大体流程和QA环境相似，只不过是多了处理node的情况.

#### Prod 环境部署
> Jenkins 在build之后会将最新的代码pull到release服务器， 然后在release服务器上执> 行脚本，将代码发布到各个服务器上, 在release服务器的**/data** 目录下有两个目录
> 1. **/data/install**: 将环境所需的软件、Licence等存放在此，这些只是第一次发布的时候会用到
> 2. **/data/build**: 这是真正build的所需要的目录


1. 点击build之后，Jenkins会按照该item配置好的Git Repository 来拉取最新代码到workspace中
2. 执行自定义Shell 脚本(Jenkins 控制台写好的脚本, 和QA环境类似，注意目录的改变)

		rm -rf /data/prod_honor_hybris/hybris/bin/custom
		cp -r $WORKSPACE/bin/custom /data/prod_honor_hybris/hybris/bin
		cp $WORKSPACE/config/local.properties /data/prod_honor_hybris/hybris/config
		cp $WORKSPACE/config/localextensions.xml /data/prod_honor_hybris/hybris/config
		cd /data/prod_honor_hybris/hybris/bin/platform
		export JAVA_HOME=/usr/local/jdk1.8
		export JRE_HOME=/usr/local/jdk1.8/jre
		export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
		export PATH=$JAVA_HOME/bin:$PATH
		. ./setantenv.sh
		ant clean all
		cd /data/prod_honor_hybris/build
		./build_and_deploy.sh

3. **./build_and_deploy.sh** 脚本内容

		#!/bin/bash
		cd /var/lib/jenkins/workspace/prod-honor-hybris
		echo "build for honor hybris product"

		echo "create honor source code tar ball"
		rm -rf honor
		rm -rf honor.tar.gz
		mkdir honor
		cp -r /var/lib/jenkins/workspace/prod-honor-hybris/bin honor
		cp -r /var/lib/jenkins/workspace/prod-honor-hybris/config/prod honor
		tar -zcvf honor.tar.gz honor

		# 1. 生产上要备份原有的压缩包，以防万一进行恢复版本
		echo "======backup original tar in release server(10.11.12.232)====="
		ssh app@10.11.12.232 "cp /data/build/product/honor.tar.gz /data/build/product/honor.tar.gz_backup"
		echo "======copy tar ball to release server(10.11.12.232)====="
		scp honor.tar.gz app@10.11.12.232:/data/build/product
		echo "build_and_deploy done"
		exit 0


4. 发布内容前端服务器，进入到Prod Release 服务器相应目录执行脚本**app_release.sh**
> 主要动态循环IP地址，将文件传送到不同的服务器，执行相应服务器的脚本.

		#!/bin/bash

		if [ $# -lt 2 ]; then
		  echo "please provide server start index and end index"
		  echo "usage: $0 [start] [end]"
		 # exit 0
		fi

		s=prd-hybris-pad
		baseIp=11.12.126.
		let ipStartNum=$1+54
		echo "release artifact to hybris server from $s$1 to $s$2"

		for((i=$1;i<=$2;i++))
		do
			server=$s$i.xxx.com
			ipAddr=$baseIp$ipStartNum
			echo "============deploy on $server===================="
			echo "copy Hybris production Deployment ZIP to $server"
			scp /data/build/product/honor.tar.gz app@$server:/data/workspace

			echo "build hybris on $server and start hybris"
			ssh app@$server "cd /data/workspace;./stop.sh; nohup ./build_and_start.sh $ipAddr  >./build.log 2>&1 & "
			let ipStartNum=$ipStartNum+1
		        sleep 1
		done

		echo "done"
		#exit 0

5. Prod 服务器目录结构

![目录结构](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_prod1.png)

6. 进入到Prod 服务器之一 查看**./stop.sh** 脚本内容

		#!/bin/bash
		cd /data/workspace/hybris/hybris/bin/platform
		. ./setantenv.sh && ./hybrisserver.sh stop

7. 查看**./build_and_start.sh**内容（内容有些复杂）

		#!/bin/bash
		rm -rf honor
		chmod 755 honor.tar.gz
		tar -zxvf honor.tar.gz
		chmod -R 755 honor

		echo "remove old honor hybris source code..."
		rm -rf /data/workspace/hybris/hybris/bin/custom
		rm -rf /data/workspace/hybris/hybris/config/local.properties
		rm -rf /data/workspace/hybris/hybris/config/localextensions.xml

		echo "copy new honor source code..."
		cp -r honor/* hybris/hybris
		cp honor/prod/local.properties hybris/hybris/config/local.properties
		cp honor/prod/localextensions.xml hybris/hybris/config/localextensions.xml
		cp template.tomcat.context.tpl hybris/hybris/config/tomcat/tomcat_context.tpl
		sed -c -i 's/.*name="backoffice".*/<!-- <extension name="backoffice" \/> -->/g' hybris/hybris/config/localextensions.xml
		sed -c -i 's/.*requires-channel="https".*/<intercept-url pattern="\/\*\*" requires-channel="https"\/>/g' hybris/hybris/bin/custom/honor/honorcommercewebservices/web/webroot/WEB-INF/config/v2/security-v2-spring.xml
		  sed -c -i 's/.*name="mcc".*/<!-- <extension name="mcc" \/> -->/g' hybris/hybris/config/localextensions.xml

		#echo cluster.id=$1 >> hybris/hybris/config/local.properties
		echo -e "\ncluster.broadcast.method.jgroups.tcp.bind_addr=$1" >> hybris/hybris/config/local.properties
		echo -e "\ntask.processing.enabled=fase" >> hybris/hybris/config/local.properties
		echo -e "\nhac.webroot=/hac" >> hybris/hybris/config/local.properties
		rm -rf honor

		echo "begin to start server..."
		export JAVA_HOME="/usr/local/jdk1.8.0_101"
		export PATH=$JAVA_HOME/bin:$PATH
		cd /data/workspace/hybris/hybris/bin/platform
		. ./setantenv.sh
		ant clean all
		. ./hybrisserver.sh start
		exit 0


8. **template.tomcat.context.tpl** 文件内容

		#if (
		  $contextPath != '/hmc' &&
		  $contextPath != '/hac' &&
		  $contextPath != '/backoffice' &&
		  $contextPath != '/advancedexport' &&
		  $contextPath != '/admincockpit' &&
		  $contextPath != '/mcc' &&
		  $contextPath != '/reportcockpit' &&   
		  $contextPath != '/importcockpit' &&  
		  $contextPath != '/cscockpit' &&  
		  $contextPath != '/cmscockpit' &&
		  $contextPath != '/cmscockpitRegular' &&
		  $contextPath != '/productcockpit' &&  
		  $contextPath != '/chinacheckoutaddon' &&
		  $contextPath != '/instore' &&
		  $contextPath != '/virtualjdbc' &&
		  $contextPath != '/erpintegration' &&
		  $contextPath != '/acceleratorservices' &&
		  $contextPath != '/commercesearchsampledata' &&
		  $contextPath != '/solrfacetsearch'
		)

		#if($contextDescription && $contextDescription != '')
					<!-- $contextDescription -->
		#end
					<Context path="$contextPath" docBase="$contextDocBase" $!contextAdditionalAttributes>
						<Manager pathname="" />
		#if($contextLoader && $contextLoader != '')
						$contextLoader
		#end
		#if($contextAdditionalElements && $contextAdditionalElements != '')
						$contextAdditionalElements
		#end
					</Context>
		#end


9. 发布内容后端服务器，进入到Prod Release 服务器相应目录执行脚本**backend_release.sh**

		#!/bin/bash

		if [ $# -lt 2 ]; then
		  echo "please provide server start index and end index"
		  echo "usage: $0 [start] [end]"
		 # exit 0
		fi

		s=prd-hybris
		baseIp=10.0.126.
		let ipStartNum=$1+52
		echo "release artifact to hybris server from $s$1 to $s$2"

		for((i=$1;i<=$2;i++))
		do
			server=$s$i.xxxx.com
			ipAddr=$baseIp$ipStartNum

			echo "============deploy on $server===================="
			echo "copy Hybris production Deployment ZIP to $server"
			scp /data/build/product/honor.tar.gz app@$server:/data/workspace

			echo "build hybris on $server and start hybris"
			ssh app@$server "cd /data/workspace;./stop.sh; nohup ./build_and_start.sh  $ipAddr >./build.log 2>&1 & "
			let ipStartNum=$ipStartNum+1
		        sleep 1
		done

		echo "done"
		#exit 0


10. **security-v2-spring.xml** 文件内容

		<?xml version="1.0" encoding="UTF-8"?>
		<!--
		 [y] hybris Platform

		 Copyright (c) 2000-2016 SAP SE or an SAP affiliate company.
		 All rights reserved.

		 This software is the confidential and proprietary information of SAP
		 ("Confidential Information"). You shall not disclose such Confidential
		 Information and shall use it only in accordance with the terms of the
		 license agreement you entered into with SAP.
		-->
		<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			   xmlns:security="http://www.springframework.org/schema/security"
			   xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/security
				http://www.springframework.org/schema/security/spring-security.xsd">

			<http pattern="/v2/**" entry-point-ref="oauthAuthenticationEntryPointV2"
				  access-decision-manager-ref="webSecurityAccessDecisionManager"
				  xmlns="http://www.springframework.org/schema/security" create-session="stateless">

				<anonymous username="anonymous" granted-authority="ROLE_ANONYMOUS"/>
				<!--<session-management session-authentication-strategy-ref="fixation"/>-->

		<intercept-url pattern="/**" requires-channel="https"/>

				<port-mappings>
					<port-mapping http="#{configurationService.configuration.getProperty('tomcat.http.port')}"
								  https="#{configurationService.configuration.getProperty('tomcat.ssl.port')}"/>
					<port-mapping http="#{configurationService.configuration.getProperty('embeddedserver.http.port')}"
								  https="#{configurationService.configuration.getProperty('embeddedserver.ssl.port')}" />
				</port-mappings>

				<custom-filter ref="resourceServerFilter" before="PRE_AUTH_FILTER"/>
				<access-denied-handler ref="oauthAccessDeniedHandlerV2"/>

				<headers >
					<content-type-options />
					<hsts include-subdomains="true" max-age-seconds="16070400" />
					<xss-protection />
					<security:frame-options disabled="true"/>
				</headers>
				<security:csrf disabled="true"/>
			</http>

			<bean id="oauthAuthenticationEntryPointV2" parent="oauthAuthenticationEntryPoint">
				<property name="exceptionRenderer" ref="oAuth2ExceptionRendererV2"/>
			</bean>

			<bean id="oauthAccessDeniedHandlerV2" parent="oauthAccessDeniedHandler">
				<property name="exceptionRenderer" ref="oAuth2ExceptionRendererV2"/>
			</bean>

			<bean id="oAuth2ExceptionRendererV2" parent="oAuth2ExceptionRenderer">
				<property name="messageConverters" ref="messageConvertersV2"/>
			</bean>
		</beans>



大家看到了好处，也看到了Jenkins 不同服务器的部署基本类似，就是苦力活了， 接下来会将docker的部署陆续添加进来.
