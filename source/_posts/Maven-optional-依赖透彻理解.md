---
title: Maven 依赖传递性透彻理解
tags:
  - Maven
categories: [Coding, DevOps]
id: maven-dependency-optional-transitive
date: 2019-10-11 15:36:38
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgmaven.png
description: 通过图形了解Maven Optional传递性背后的道理
keywords: Maven,optional,option=true,依赖,传递性
---

## 写在前面
本来想写一篇「如何自定义Spring Boot Starter」，但是为了更好理解 Starter 的一些设计理念和其中的关键点，所以提前将一些细节内容单独提取出来讲解说明

在 Maven pom.xml 中，你经常会看到依赖项中有类似下面的代码:
```xml
<dependency>
  <groupId>sample.ProjectA</groupId>
  <artifactId>Project-A</artifactId>
  <version>1.0</version>
  <scope>compile</scope>
  <optional>true</optional> 
</dependency>
```
这里的 `<optional>true</optional> ` 是什么意思呢？

## optional 关键字的奥秘
老规矩，画个图说明问题:


![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-11/maven%20optional%20%283%29.png)


由于 project C 使用到了两个来自 project A 的类 (OptionalFeatureAClass) 和 project B 的类 (OptionalFeatureBClass). 如果 project C 没有依赖 packageA 和 packageB，那么编译将会失败。

project D 依赖 project C，但是对于 project D 来说，类 (OptionalFeatureAClass) 和类 (OptionalFeatureBClass) **是可选的特性**，所以为了让最终的 war/ejb package 不包含不必要的依赖，使用`<optional>` 声明当前依赖是可选的, 默认情况下也不会被其他项目继承(好比 Java 中的 final 类，不能被其他类继承一样)

如果 project D 确实需要用到 project C 中的 OptionalFeatureAClass 怎么办呢？那我们就需要在 project D 的 pom.xml 中显式的添加声明 project A 依赖，继续看下图:


![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-11/maven%20optional%202.png)



Project D 需要用到 Project A 的 OptionalFeatureAClass，那么需要在 Project D 的 pom.xml 文件中显式的添加对 Project A 的依赖

到这也就很好理解为什么 Maven 为什么要设计 optional 关键字了，假设一个关于数据库持久化的项目(Project C), 为了适配更多类型的数据库持久化设计，比如 Mysql 持久化设计(Project A) 和 Oracle 持久化设计(Project B)，当我们的项目(Project D) 要用的 Project C 的持久化设计，不可能既引入 mysql 驱动又引入 oracle 驱动吧，所以我们要显式的指定一个，就是这个道理了

## 实际案例
在 [spring-boot-actuator](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-actuator/1.3.3.RELEASE/spring-boot-actuator-1.3.3.RELEASE.pom) pom.xml 文件中，有超过 20 个依赖是 optional

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-11/2019-10-11-20-34-58.png)


因为 Spring Boot 不可能将没必要的依赖也打包到你最终的 jar package 中，所以用到 spring boot actuator 的项目最终生成的 jar package 中不会包含这 20 多个依赖 jar，如果你要用到哪一个，显式的加入到你的项目就好了

在接下来的文章，自定义 Spring Boot Starter 也是这个策略，因为 starter 是包含特定功能为其他项目服务用的，类似本文的 Project C 的角色了，到这里你理解 optional 的奥秘了吗？

## 反向应用
如果 Project C 引入的依赖没有加 `<optional>true</optional>`，Project D 又需要依赖 Project C，但只用到 Project A 的类怎么办呢？Maven 也是有解决办法的，使用 exclusion 关键字，不多说，上一段代码就懂了:
```xml
<dependencies>
    <dependency>
      <groupId>top.dayarch.demo</groupId>
      <artifactId>Project-C</artifactId>
      <exclusions>
        <exclusion>
          <groupId>top.dayarch.demo</groupId>
          <artifactId>Project-B</artifactId>
        </exclusion>
      </exclusions> 
    </dependency>
</dependencies>
```

## 总结
到这里，在你今后设计功能性依赖时，你应该明白怎样设计依赖关系了, 我这里推荐使用 optional 的形式，简单来说，你设计的依赖什么菜都有，想吃什么菜自己 "抱蔡明" 就好，接下来我们就模拟官方标准创建自定义的 starter......

## 灵魂追问
1. 有很多童鞋项目组用的构建工具时 Gradle，你知道 Gradle 中是怎样表示的吗？
2. 自定义 starter，你知道官方标准 starter 的结构是什么样的吗？

## 提高效率工具

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png) 


--------

## 推荐阅读
1. [这次走进并发的世界，请不要错过](https://dayarch.top/p/d7e125a1.html)
2. [学并发编程，透彻理解这三个核心是关键](https://dayarch.top/p/86243a5b.html)
3. [并发Bug之源有三，请睁大眼睛看清它们](https://dayarch.top/p/6d12b8cf.html)
4. [可见性有序性，Happens-before来搞定](https://dayarch.top/p/815d7647.html)
5. [解决原子性问题？你首先需要的是宏观理解](https://dayarch.top/p/32b8e26a.html)
6. [面试并发volatile关键字时，我们应该具备哪些谈资？](https://dayarch.top/p/fb3c7eeb.html)


--------

> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享 
> - 高效工具汇总 | 回复「工具」
> - 面试问题分析与解答 
> - 技术资料领取 | 回复「资料」

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)
