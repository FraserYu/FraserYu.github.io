---
title: Spring Boot 自定义starter 全面教程
tags:
  - Spring Boot
  - Starter
categories: [Coding, Spring-Boot]
id: spring-boot-starter-custom
date: 2019-10-15 19:35:53
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootstarter.png
icons: [fas fa-star yellow, fas fa-fire accent]
description: Spring Boot 自定义starter 全面教程，模仿官方标准构建属于自己的starter
keywords: starter,自定义starter,SpringBoot,SpringBoot自定义Starter教程
---

## 写在前面
我们每次构建一个 Spring 应用程序时，我们都不希望从头开始实现具有「横切关注点」的内容；相反，我们希望一次性实现这些功能，并根据需要将它们包含到任何我们要构建的应用程序中
>#### 横切关注点
>  横切关注点: 指的是一些具有横越多个模块的行为 (来自维基百科的介绍)
>  **说白了就是多个项目或模块都可以用到的内容，比如一个 SDK**

在Spring Boot中，用于表示提供这种横切关注点的模块的术语是 **starter**，通过依赖 starter 可以轻松使用其包含的一些功能特性，**无论你的工作中是否会构建自己的 starter，你都要具有构建 「starter」的思想**，本文将结合 Spring Boot 官方标准构建一个简单的 starter

## 自定义 starter
在我们深入了解如何自定义 starter 之前，为了更好的理解我们每一步在干什么，以及 starter 是如何起作用的，我们先从宏观角度来看 starter 的结构组成到底是什么样的

通常一个完整的 starter 需要包含下面两个组件:
1. Auto-Configure Module
2. Starter Module

如果你看下面这两个组件的解释有些抽象，大概了解一下，阅读完该文章回看这里就会豁然开朗了

### Auto-Configure Module
Auto-Configure Module (自动配置模块) 是包含自动配置类的 Maven 或 Gradle 模块。通过这种方式，**我们可以构建可以自动贡献于应用程序上下文的模块，以及添加某个特性或提供对某个外部库的访问**

### Starter Module
Spring Boot Starter 是一个 Maven 或 Gradle 模块，其唯一目的是提供 "启动" 某个特性所需的所有依赖项。可以包含一个或多个 Auto-Configure Module (自动配置模块)的依赖项，以及可能需要的任何其他依赖项。这样，在Spring 启动应用程序中，我们只需要添加这个 starter 依赖就可以使用其特性

>  ⚠️: Spring 官方参考手册建议将自动配置分离，并将每个自动配置启动到一个独立的 Maven 或 Gradle 模块中，从而将自动配置和依赖项管理分离开来。如果你没有建立一个供成千上万用户使用的开源库，也可以将二者合并到一个 module 中
>  *You may combine the auto-configuration code and the dependency management in a single module if you do not need to separate those two concerns*

### 命名
来自 Spring 官方的 starter 都是 以 `spring-boot-starter` 开头，比如:
- spring-boot-starter-web
- spring-boot-starter-aop

如果我们自定义 starter 功能名称叫`acme`，那么我们的命名是这样的:
- acme-spring-boot-starter
- acme-spring-boot-autoconfigure

如果 starter 中用到了配置 keys，也要注意不要使用 Spring Boot 使用的命名空间，比如(server，management，spring)

### Parent Module 创建
先来全局看一下项目结构:
一级目录结构:
```java
.
├── pom.xml
├── rgyb-spring-boot-autoconfigure
├── rgyb-spring-boot-sample
└── rgyb-spring-boot-starter
```

二级目录结构:
```java
.
├── pom.xml
├── rgyb-spring-boot-autoconfigure
│   ├── pom.xml
│   └── src
├── rgyb-spring-boot-sample
│   ├── pom.xml
│   └── src
└── rgyb-spring-boot-starter
    ├── pom.xml
    └── src
```

创建一个空的父亲 Maven Module，主要提供依赖管理，这样 SubModule 不用单独维护依赖版本号，来看 pom.xml 内容:
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>

    <!--  添加其他全局依赖管理到这里，submodule默认不引入这些依赖，需要显式的指定  -->
</dependencyManagement>
```

### Auto-Configure Module 构建
新建类 `GreetingAutoConfiguration`

```java
@Configuration
public class GreetingAutoConfiguration {

	@Bean
	public GreetingService greetingService(GreetingProperties greetingProperties){
		return new GreetingService(greetingProperties.getMembers());
	}
}
```

我们用 `@Configuration` 注解标记类 GreetingAutoConfiguration，作为 starter 的入口点。这个配置包含了我们需要提供starter特性的所有 `@Bean` 定义，在本例中，为了简单阐述问题，我们只将 GreetingService Bean 添加到应用程序上下文

GreetingService 内容如下:
```java
@AllArgsConstructor
public class GreetingService {

	private List<String> members = new ArrayList<>();

	public void sayHello(){
		members.forEach(s -> System.out.println("hello " + s));
	}
}
```

在 resources 目录下新建文件 `META-INF/spring.factories` (如果目录 `META-INF` 不存在需要手工创建)，向文件写入内容:
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
 top.dayarch.autoconfigure.GreetingAutoConfiguration
```

Spring 启动时会在其 classpath 中所有的 `spring.factoreis` 文件，并加载里面的声明配置，GreetingAutoConfiguration 类就绪后，我们的 Spring Boot Starter 就有了一个自动激活的入口点

到这里这个 "不完全的 starter" 已经可以使用了。但因为它是自动激活的，为了个让其灵活可用，我们需要让其按照我们的意愿来激活使用，所以我们需要条件注解来帮忙

#### 条件配置
为类添加两个条件注解:

```java
@Configuration
@ConditionalOnProperty(value = "rgyb.greeting.enable", havingValue = "true")
@ConditionalOnClass(DummyEmail.class)
public class GreetingAutoConfiguration {
    ...
}
```

- 通过使用 `@ConditionalOnProperty` 注解，我们告诉 Spring，只有属性 `rgyb.greeting.enable` 值被设置为 true 时，才将 GreetingAutoConfiguration (以及它声明的所有 bean ) 包含到应用程序上下文中
- 通过使用 `@ConditionalOnClass` 注解，我们告诉Spring 只有类 DummyEmail.class 存在于 classpath 时，才将 GreetingAutoConfiguration (以及它声明的所有 bean ) 包含到应用程序上下文中

多个条件是 `and/与`的关系，既只有满足全部条件时，才会加载 GreetingAutoConfiguration

如果你对条件注解的使用还不是很明确，可以查看我之前的文章: [@Conditional注解，灵活配置 Spring Boot](https://dayarch.top/p/d66d57ce.html)

#### 配置属性管理
上面使用了 `@ConditionalOnProperty` 注解，实际 starter 中可能有非常多的属性，所以我们需要将这些属性集中管理:
```java
@Data
@ConfigurationProperties(prefix = "rgyb.greeting")
public class GreetingProperties {

	/**
	 * GreetingProperties 开关
	 */
	boolean enable = false;

	/**
	 * 需要打招呼的成员列表
	 */
	List<String> members = new ArrayList<>();
}
```

我们知道这些属性是要在 application.yml 中使用的，当我们需要使用这些属性时，为了让 IDE 给出更友好的提示，我们需要在 pom.xml 中添加依赖:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

这样当我们 mvn compile 时，会在生成一个名为 `spring-configuration-metadata.json` JSON 文件，文件内容如下:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-11-17-51.png)


生成的内容在接下来的内容中用到，且看

#### 提升启动时间
对于类路径上的每个自动配置类，Spring Boot 必须计算 `@Conditional…` 条件值，用于决定是否加载自动配置及其所需的所有类，根据 Spring 启动应用程序中 starter 的大小和数量，这可能是一个非常昂贵的操作，并且会影响启动时间，为了提升启动时间，我们需要在 pom.xml 中添加另外一个依赖:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure-processor</artifactId>
    <optional>true</optional>
</dependency>
```

这个注解会生成一个名为 `spring-autoconfigure-metadata.properties` Property 文件，其内容如下:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-11-26-42.png)


这样，Spring Boot 在启动期间读取这些元数据，可以过滤出不满足条件的配置，而不必实际检查这些类，提升启动速度

到这里关于 Auto-Configure Module 就构建完了，我们需要继续完成 Starter Module 的构建

### Starter Module 构建
Starter Module 的构建很简单了，你可以认为它就是一个空 module，除了依赖 Auto-Configure Module，其唯一作用就是为了使用 starter 功能特性提供所有必须依赖，所以我们为 starter module 的 pom.xml 文件添加如下内容:
```xml
<dependencies>
    <dependency>
        <groupId>top.dayarch.learnings</groupId>
        <artifactId>rgyb-spring-boot-autoconfigure</artifactId>
        <version>1.0.0.RELEASE</version>
    </dependency>

    <!-- 在此处添加其他必要依赖，保证starter可用 -->
</dependencies>
```

同样在 resources 目录下新建文件 `META-INF/spring.providers` , 其内容如下:
```properties
providers: rgyb-spring-boot-autoconfigure
```
该文件主要作用是说明 starter module 的依赖信息，多个依赖以逗号分隔就好，该文件不会影响 starter 的使用，可有可无

Starter Module 就可以这么简单，将两个 module 分别 mvn install 到本地 Maven Repository，接下来我们创建 sample module 引入这个 starter 依赖时就会从本地 Maven Repository 中拉取

## 创建 Sample Module
我们可以通过 Spring Initializr 正常初始化一个 Spring Boot 项目 (rgyb-spring-boot-sample)，引入我们刚刚创建的 starter 依赖，在 sample pom.xml 中添加依赖:
```xml
<dependency>
    <groupId>top.dayarch.learnings</groupId>
    <artifactId>rgyb-spring-boot-starter</artifactId>
    <version>1.0.0.RELEASE</version>
</dependency>
```

接下来配置 application.yml 属性
```yml
rgyb:
  greeting:
    enable: true
    members:
      - 李雷
      - 韩梅梅
```
在我们配置 YAML 的时候，会出现下图的提示，这样会更友好，当然为了规范，属性描述最好也用英文描述，这里为了说明问题用了中文描述:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-11-43-14.png)


## 编写测试类
我们编写测试用例:
```java
@Autowired(required = false)
private GreetingService greetingService;

@Test
public void testGreeting() {
    greetingService.sayHello();
}
```
测试结果如下:
```console
hello 李雷
hello 韩梅梅
```

## 总结
到这里完整的 starter 开发就结束了，希望大家了解其构建过程，目录结构及命名等标准，这样有相应的业务需求时都可以开发自己的 starter 被其他人应用起来

starter 开发好了，别人可以手动添加依赖引入 starter 的相关功能，那我们如何像 Spring Initializr 一样，通过下来菜单选择我们的 starter 呢，这样直接初始化好整个项目，接下来的文章我们会模仿 Spring Initializr 自定义我们自的 Initializr

## 知识点说明
### Dependency optinal
> 为什么 Auto-Configure Module 的 dependency 都是 optional = true 呢？
这涉及到 Maven 传递性依赖的问题，详情请看 [Maven 依赖传递性透彻理解](https://dayarch.top/p/b674819.html)

### spring.factories
>  Spring Boot 是如何加载这个文件并找到我们的配置类的
下图是 Spring Boot 应用程序启动的调用栈的一部分，我添加了断点:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-12-38-42.png)


打开 SpringFactoriesLoader 类，映入眼帘的就是这个内容:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-12-39-32.png)

这两张图应该足够说明问题了，是 SPI 的一种加载方式，更细节的内容请大家自己去发现吧

## 实际案例
这里推荐查看 [mybatis-spring-boot-starter](https://github.com/mybatis/spring-boot-starter) 这个非 Spring 官方的案例，从中我们:
- 模仿其目录结构
- 模仿其设计理念
- 模仿其编码规范

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-10-14/2019-10-15-12-51-17.png)


另外，本文的案例我已上传，公众号回复「demo」，打开链接，查看 customstarter 目录下内容即可

## 灵魂追问
1. 在生成 spring-autoconfigure-metadata.properties 文件时，为什么 @ConditionalOnProperty 的内容没有被写进去
3. 如果我们要将依赖上传至 remote central repository，你知道怎样搭建自己的 maven repository 吗？

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
