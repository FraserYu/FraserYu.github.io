---
title: 使用 Docker 部署 Spring Boot
tags:
  - Docker
  - Spring Boot
categories: [Coding, Docker]
id: docker-deploy-spring-boot
date: 2018-04-09 15:43:42
---

### 前言
最近陆续在通过SpringBoot搭建项目，往里面逐渐添加功能，项目地址[SpringBootLearning](https://github.com/FraserYu/springbootlearning.git), 当总结Docker部署SpringBoot的时候，我遇到了一些问题，加以总结
<!--more-->
### 问题汇总
文章主要参考[Spring Boot 2.0(四)：使用 Docker 部署 Spring Boot](http://www.spring4all.com/article/992), 因为我是已有的SpringBoot项目，不是简单的SpringBoot项目，所以使用Docker部署SpringBoot的时候遇到了一些问题，现总结如下：
1. **mvn package** 打包命令不能生成jar包到target目录下
因为Docker会用到jar包，所以在用Docker部署之前，先用Maven来进行打包，保证正确，但是使用命令并不能生成jar包，发现打包中打了我的测试用例类，测试用例有持续执行操作，当然其他持续操作也有可能出现这个问题，所以我在pom.xml中加入了plugin来过滤掉Test case.
```xml
    <properties>
    <!--mvn 打包时省略test用例-->
    <skipTests>true</skipTests>
    </properties>

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <configuration>
        <skipTests>${skipTests}</skipTests>
      </configuration>
    </plugin>
```
2. 通过Maven的docker插件来部署，遇到了两个问题：
> + [ERROR] Failed to execute goal com.spotify:docker-maven-plugin:1.0.0:build (default-cli) on project demo: Exception caught: java.util.concurrent.ExecutionException: com.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: org.apache.http.conn.HttpHostConnectException: Connect to localhost:2375 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused: connect -> [Help 1]
> + [ERROR] Failed to execute goal com.spotify:docker-maven-plugin:1.0.0:build (default-cli) on project demo: Exception caught: java.util.concurrent.ExecutionException: com.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target -> [Help 1]

通过查找资料在pom.xml中maven的docker插件中添加了如下内容：
```xml
  <dockerHost>https://192.168.99.100:2376</dockerHost>
  <dockerCertPath>D:/Users/fraser.yu/.docker/machine/machines/default</dockerCertPath>
```
