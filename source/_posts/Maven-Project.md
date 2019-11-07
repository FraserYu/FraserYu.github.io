---
title: Maven Project
tags: Maven
categories: [Coding, DevOps]
id: maven-project-structure
date: 2016-01-21 17:28:06
---

# 创建Maven Web Project
-----------------------------------------------
### 什么是Maven
在[Apache Maven](http://maven.apache.org/) 官网上的说明是：*Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.*   翻译过来的基本意思就是：**Maven是基于项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工 具。**


<!-- more -->

### 使用Maven管理项目的好处
1.  对第三方依赖库进行统一的版本管理。
> 只要用了Maven就不用再为每个项目复制spring.jar和hibernate.jar了，Maven会在你需要的时候，自动把这些第三方依赖库找到，你需要编译，Maven就把这些jar包放到classpath里，你需要打包，Maven2就帮你把需要的jar包都复制到WEB- INF/lib/目录下。

2.  统一项目的目录结构
>  可以保证所有项目的目录结构都是一样的，目录结构统一的好处就是，你要找源代码就去src/main/java/下，你要找需要放到classpath下的资源，就去src/main/resources/下，你要找单元测试对应的代码和资源，就去src/test/下。每个目录下放什么东西，程序编译，发布的时候，每个目录起什么作用都很清楚明了，不会出现打开项目找不到要找的文件的情况。

3. 统一软件构建阶段
> Maven2把软件开发的过程划分成了几个经典阶段，比如你先要生成一些java代码，再把这些代码复制到特定位置，然后编译代码，复制需要放到 classpath下的资源，再进行单元测试，单元测试都通过了才能进行打包，发布。

4. 支持多种插件
> 在http://maven.apache.com/和http://mojo.codehaus.org/上可以找到大量的Maven2插件，通过这些插件可以完成多种多样的扩展功能。

以上只是概念性的东西，陆续会用实例证明以上好处

### 下载及安装Maven
1. 访问[Maven](http://maven.apache.org/) 下载：
![down load Maven](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_download_maven.png)
<br/>                 
2. 以Windows 环境为例，将下载的**apache-maven-3.3.9-bin.zip** 解压到某一磁盘目录下，**apache-maven-3.3.9** 作为**MAVEN_HOME** 目录，
<br/>       
3. 新建本地Maven仓库（Repository），编辑**apache-maven-3.3.9\conf\setting.xml** 文件，在本地建立仓库（任意一次盘文件夹下即可）
![Maven Repo](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_mavenRepo.png)
<br/>       
4. 环境变量可配置也可不配置，如果用到命令行必须要配置环境变量。
	> 1. 新建系统变量MAVEN_HOME, 并指定目录：E:\Java_Web\apache-maven-3.3.9-bin\apache-maven-3.3.9
	> 2. 编辑系统变量Path，增加%MAVEN_HOME%\bin;
	> 3. 命令行下（cmd）检测Maven 版本： mvn -v  
	>
	> ![maven_env_configuration](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_mavenEnv.png)

### Eclipse下配置Maven
1. 现在新版本的Eclipse已经集成了Maven插件，在Windows-Preferences-Maven节点下的Installations添加Maven, 最后**勾选上添加的Maven**。
![Add Maven](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_addMaven.png)
<br/>       
2. 在Windows-Preferences-Maven节点下的User Setting设置setting.xml 文件
![User Setting](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_mavenSetting.png)
<br/>       
### 新建Maven Web Project
1.  鼠标邮件New Maven Project
![step1](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_new_maven1.png)
<br/>
2.  选择Artifact ID
![step2](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_new_maven2.png)
<br/>
3. 填写required text
![step3](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_new_maven3.png)
<br/>
4. pom.xml 上鼠标邮件run as **Maven install**, 会发现关于这个项目的war包在Maven仓库中已经生成
![step4](http://7xkyc7.com1.z0.glb.clouddn.com/blog_3_Maven_new_maven4.png)

### 常用MVN命令
> 1. mvn  help:system 自动在本用户下创建   ~/.m2/repository
> 2. mvn clean compile     清理编译
> 3. mvn clean test  清理测试
> 4. mvn clean package 清理打包
> 5. mvn clean install  清理将打包好的jar存入 本地仓库  注意是本地仓库
> 6. mvn archetype:generate 使用Archetype生成项目骨架
> 7. mvn clean deploy  根据pom中的配置信息将项目发布到远程仓库中


### Maven项目目录结构说明
> 2. src/main/java :正式内容包路径
> 3. src/mian/resources :正式的配置文件路径
> 4. src/test/java :测试包路径
> 5. src/test/resources :测试的配置文件路径
> 6. src/main/webapp : war 资源目录

### 更多
更多关于[IntelliJ IDEA 下创建Maven项](http://mark.leanote.com/post/%E4%BD%BF%E7%94%A8IntelliJ-IDEA-14%E5%92%8CMaven%E5%88%9B%E5%BB%BAjava-web%E9%A1%B9%E7%9B%AE)
