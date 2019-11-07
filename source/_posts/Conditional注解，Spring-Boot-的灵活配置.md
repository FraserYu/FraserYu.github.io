---
title: '@Conditional注解，Spring Boot 的灵活配置'
categories: [Coding, Spring-Boot]
tags:
  - Spring Boot
  - 注解
id: spring-boot-condition-annotation
date: 2019-07-31 16:30:07
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootcondition.png
description: 灵活使用SpringBoot Conditonal相关注解，增加编程的灵活性
keywords: SpringBoot,ConditionalOnProperty,注解
---

上一篇文章 [你应该知道的 @ConfigurationProperties 注解的使用姿势，这一篇就够了](https://mp.weixin.qq.com/s/fvgL73R_4ANp4wsSiEhUGA) 介绍了如何通过 `@ConfigurationProperties` 注解灵活读取配置属性，这篇文章将介绍如何灵活配置 Spring Bean

## 写在前面
当我们构建一个 Spring 应用的时候，有时我们想在满足指定条件的时候才将某个 bean 加载到应用上下文中， 在Spring 4.0 时代，我们可以通过 `@Conditional` 注解来实现这类操作
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-14-13-28%402x.png)

我们看到  `@Conditional`  注解接收的参数是 extends Condition 接口的泛型类，也就是说，我们要使用 `@Conditional` 注解，只需要实现 Condition 接口并重写其方法即可:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-14-14-40%402x.png)

看到接口的 matches 方法返回的是 boolean 类型，是不是和我们自定义 validation annotation 有些类似，都是用来判断是否满足指定条件。另外注意看，以上注解和接口都在  `org.springframework.context.annotation`  package 中

终于到了 Spring Boot 时代，在这个全新的时代，Spring Boot 在  `@Conditional`  注解的基础上进行了细化，无需出示复杂的介绍信 (实现 Condition 接口)，只需要手持预定义好的 `@ConditionalOnXxxx` 注解印章的门票，如果验证通过，就会走进 Application Context 大厅

## 注解详解
Spring Boot 对 `@Conditional` 注解为我们做了细化，这些注解都定义在 `org.springframework.boot.autoconfigure.condition` package 下
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-11-07-51.png)

逐个打开这 13 个注解，我们发现这些注解上有相同的元注解:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-45-29%402x.png)

从这些标记上我们可以了解如下内容:
- 都可以应用在 TYPE 上，也就是说，Spring 自动扫描的一切类 (@Configuration, @Component, @Service, @Repository, or @Controller) 都可以通过添加相应的 `@ConditionalOnXxxx` 来判断是否加载

- 都可以应用在 METHOD 上，所以有 @Bean 标记的方法也可以应用这些注解

- 都是用了 `@Conditional` 注解来标记，OnBeanCondition 等自定义 Condition 还是实现了 Condition 接口的，换汤不换药，没什么神秘的，只不过做了更具象的封装罢了，来看类依赖图:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-13-40-16.png)

其实看这些注解字面意思已经能理解这些注解的含义，但是我们还是要说明具体的使用以及一些注意事项，我按照个人使用频次由高到低讲解:

### @ConditionalOnProperty
毫无疑问这个注解是榜首
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-49-03%402x.png)

这个条件解释是:  application.properties 或 application.yml 文件中 mybean.enable 为 true 才会加载 MyCondition 这个 Bean，如果没有匹配上也会加载，因为 matchIfMissing = true，默认值是 false。
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-49-57%402x.png)

### @ConditionalOnBean 和 ConditionalOnMissingBean
有时候我们需要某个 Bean 已经存在应用上下文时才会加载，那么我们会用到 `@ConditionalOnBean` 注解:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-51-16%402x.png)

与之相反，有时候我们需要某个 Bean 不存在于应用上下文时才会加载，那么我们会用到 `@ConditionalOnMissingBean` 注解

### @ConditionalOnClass 和 @ConditionalOnMissingClass
不要嫌我废话，和上面的一样，只不过判断某个类是否存在于 classpath 中，这就不做过多说明了

### @ConditionalOnExpression
如果我们有更复杂的多个配置属性一起判断，那么我们就可以用这个表达式了:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-54-28%402x.png)

只有当两个属性都为 true 的时候才加载 MyModule，到这里要顺便揭晓上一篇文章 [你应该知道的 @ConfigurationProperties 注解的使用姿势，这一篇就够了](https://mp.weixin.qq.com/s/fvgL73R_4ANp4wsSiEhUGA) 灵魂追问 3，其中 `:true` 就是: 如果没有为该属性设置值，则为该属性设置默认值true, 其实这就是`@Vaue` 注解的规范，一切 SpEL 都可以应用在这里.

写到这，我常用的已经用完了，还要硬着头皮介绍其他几个内容 😄，开个玩笑，咱们继续:

### @ConditionalOnSingleCandidate
这个注解和 @ConditionalOnBean 类似，为了更好的说明该注解的使用 (其实是 `才疏学浅` ) ，我只能翻译一下类的注释了
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-56-07%402x.png)

>  只有指定类已存在于 BeanFactory 中，并且可以确定单个候选项才会匹配成功
>
> BeanFactory 存在多个 bean 实例，但是有一个 primary 候选项被指定(通常在类上使用 `@Primary` 注解)，也会匹配成功。实质上，如果自动连接具有定义类型的 bean 匹配就会成功
>
> 目前，条件只是匹配已经被应用上下文处理的 bean 定义，本身来讲，强烈建议仅仅在 auto-configuration 类中使用这个条件，如果候选 bean 被另外一个 auto-configuration 创建，确保使用该条件的要在其后面运行

### @ConditionalOnResource
如果我们要加载的 bean 依赖指定资源是否存在于 classpath 中，那么我们就可以使用这个注解
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-57-04%402x.png)

看到这个 logback.xml 是不是很亲切，在我们引入第三方工具类如 [Dozer](https://mp.weixin.qq.com/s/rE_cX0Z7nccY5D6z3O7mUQ) 等都可以添加类似的开关

接下来的是真冷门，大家有个印象，如果有需要，至少能想到用这些注解实现灵活配置就好了

### @ConditionalOnJndi
只有指定的资源通过 JNDI 加载后才加载 bean
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-58-52%402x.png)

### @ConditionalOnJava
只有运行指定版本的 Java 才会加载 Bean
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-16-59-48%402x.png)

### @ConditionalOnWebApplication 和 @ConditionalOnNotWebApplication
只有运行在 web 应用里才会加载这个 bean
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-00-34%402x.png)

与之相反，在非 web 环境才加载 bean
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-01-29%402x.png)

### @ConditionalOnCloudPlatform
这个注解冷的我呼吸都要停止了，只有运行在指定的云平台上才加载指定的 bean，CloudPlatform 是 `org.springframework.boot.cloud` 下一个 enum 类型的类，大家可以打开自行看看:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-02-42%402x.png)


到此，Spring Boot 为我们提供的这 13 个注解就介绍完了，但是没有结束，下面的一些冷门知识，你需要知道:
## 组合条件
### 组合条件 AND
如果我们想多个条件一起应用，并且条件的关系是 `and`，我们只需要在类上使用多个`@ConditionalOnXxxx` 就可以了 (你这也叫冷门？) ，当然也可以继承 `AllNestedConditions`类封装我们多个条件
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-05-03%402x.png)

这样就有了组合 and 条件，只有内部所有条件都满足，才加载指定 bean

### 组合条件 OR
如果我们希望组合的条件是 `or` 的关系，我们该怎么办呢？ 我们可以通过继承 `AnyNestedCondition` 来完成这一要求，示例代码和上面一样，大家自行打开 `AnyNestedCondition` 类，查看类说明即可


### 条件组合 NONE
有 and 和 or 就肯定有 non(非)，我们可以通过继承 `NoneNestedConditions` 完成这一要求，大家自行查看即可

 ## 自定义注解
通过组合方式实现了多条件逻辑应用，我们需要应用这些组合条件也就要自定义注解，其实文章开头已经讲过了，模仿内置的 13 个注解写就好了:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-05-50%402x.png)

只需要通过`@Conditional`注解指定我们自定义的 condition 类就好了，然后应用到你想用的地方就好了
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-07-30/2019-07-30-17-06-30%402x.png)

还是推荐大家看 RabbitMq 的 `RabbitAutoConfiguration` 类，这个类里面主流的注解都是用了 (只看这一个类就好了)，大家看框架理解学习这些注解是更好的方式:
>  https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitAutoConfiguration.java


## 总结
到这里，你已经了解了如何灵活配置 bean，结合之前的文章，相信的定义会更加灵活，希望大家打开 IDE，自行查看这些注解，了解更多具体内容

## 灵魂追问
1. SpringBoot 添加了 web starter，有哪些方法将其更改为非 web 应用？
2. Java8 Stream 也有 findAny，findAll 这类的操作，这都是匹配，你有使用过吗？
3. 看下面这段代码，如果 classpath 中没有 MyBean class，编译会报错，那这个注解什么用呢？
```java
@Configuration    
@ConditionalOnClass(MyBean.class)
public class MyConfiguration{
    // omitted       
}
```

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)

--------

## 推荐阅读
+ [红黑树，超强动静图详解，简单易懂](https://mp.weixin.qq.com/s/ilND8u_8HGSTSrJiMB4X8g)
+ [双亲委派模型：大厂高频面试题，轻松搞定](https://mp.weixin.qq.com/s/Dnr1jLebvBUHnziZzSfcrA)
+ [面试还不知道BeanFactory和ApplicationContext的区别？](https://mp.weixin.qq.com/s/YBQB086ADBjHUmwrFQrWew)
+ [如何设计好的RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ)
+ [程序猿为什么要看源码？](https://mp.weixin.qq.com/s/V7h8O6pVFQ-nr_iA2SNqtw)

--------
> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享
> - 高效工具汇总
> - 面试问题分析与解答
> - 技术资料领取
**后续会推出 「多线程」以及「ElasticSearch」等连载内容**
> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)
