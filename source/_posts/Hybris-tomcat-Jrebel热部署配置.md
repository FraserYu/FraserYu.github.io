---
title: Hybris tomcat Jrebel热部署配置
tags:
  - Hybris
  - Jrebel
categories: [Coding, Others]
id: hybris-jrebel
date: 2017-03-30 15:31:33
description: Hybris通过Jrebel插件实现热部署，提高编程调试效率
keywords: Hybris,Jebrel,热部署

---

### 背景
Jrebel是非常强大的实现项目热部署的软件，当然也比较消耗电脑的性能，当如Hybris的这种启动需要一段时间的项目，对于后端开发人员来说，改动一些东西就要重启服务真的是一种折磨，浪费时间，接下来说明一下Jrebel在Hybris项目中的配置

### 配置Jrebel
Jrebel可以通过IntelliJ IDEA来下载安装集成好的插件，也可以单独下载Jrebel的包 （请支持正版）

<!-- more -->

#### 方法一：通过IntelliJ IDEA 插件
如下图下载安装InteliJ IDEA 插件
![Jrebel](http://7xkyc7.com1.z0.glb.clouddn.com/jrebel_jrebel.png)

安装好之后我们就可以在IntelliJ IDEA的安装目录中看到该插件，接下来在Hybris的local.properties文件中来配置tomcat

	#JRebel
	tomcat.javaoptions=-agentpath:"D:/Users/fraser/.IntelliJIdea2016.2/config/plugins/jr-ide-idea/lib/jrebel6/lib/jrebel64.dll"  -DforceANSI=true
	tomcat.debugjavaoptions=-agentpath:"D:/Users/fraser/.IntelliJIdea2016.2/config/plugins/jr-ide-idea/lib/jrebel6/lib/jrebel64.dll" -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8000,suspend=n -DforceANSI=true

#### 方法二：通过安装包
下载Jrebel安装包来，将安装包解压到某一个目录下，然后同样配置local.properties 文件如下：

	#JREBEL
	tomcat.javaoptions=-Xverify:none -javaagent:"E:/MyProjectSpace/jrebel_5.6.0/jrebe/jrebel.jar=de.hybris.tomcat.HybrisWebappClassLoader60"
	tomcat.debugjavaoptions=-Xverify:none -javaagent:"E:/MyProjectSpace/jrebel_5.6.0/jrebe/jrebel.jar=de.hybris.tomcat.HybrisWebappClassLoader60"-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8060,suspend=y

	#可不配
	tomcat.generaloptions=-Xmx2G -Xms2G -XX:PermSize=500M -XX:MaxPermSize=300M -Xss256K -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+CMSPermGenSweepingEnabled -XX:+CMSClassUnloadingEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:+CMSParallelRemarkEnabled -XX:+ParallelRefProcEnabled -XX:+CMSScavengeBeforeRemark  -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Xloggc:"${HYBRIS_LOG_DIR}/tomcat/java_gc.log" -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dorg.tanukisoftware.wrapper.WrapperManager.mbean=true -Djava.endorsed.dirs=..b/endorsed -Dcatalina.base=%CATALINA_BASE% -Dcatalina.home=%CATALINA_HOME% -Dfile.encoding=UTF-8 -Dlog4j.configuration=log4j_init_tomcat.properties -Djava.util.logging.config.file=jdk_logging.properties -Djava.io.tmpdir="${HYBRIS_TEMP_DIR}" -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000

### 测试
启动Hybris服务器如看到如下信息就说明Jrebel配置成功：
![jrebel1](http://7xkyc7.com1.z0.glb.clouddn.com/jrebel_jrebel1.png)

这样我们执行**ant build** 命令等待build成功，就可以实现热部署，如果遇到如下问题：
![jrebel3](http://7xkyc7.com1.z0.glb.clouddn.com/jrebel_jrebel2.png)

找到**D:\projects\hybrisHn\hybris\bin\ext-backoffice\backoffice\buildcallbacks.xml** 文件，按照如下注释掉**<backoffice_remove_web_fragments/>** 即可

	<macrodef name="backoffice_after_build">
        <sequential>
            <!--<backoffice_remove_web_fragments/>-->
            <backoffice_create_web_fragments/>
        </sequential>
    </macrodef>

重新尝试**ant build** 命令应该就可以build成功，快尝试一下吧
