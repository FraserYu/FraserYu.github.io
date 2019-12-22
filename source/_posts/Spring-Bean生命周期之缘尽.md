---
title: Spring Bean生命周期之缘尽
date: 2019-12-21 17:23:42
tags:
    - Spring Boot
    - Spring Bean
    - 生命周期
categories: [Coding, Spring-Boot]
id: spring-bean-lifecycle-destroy
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: 透彻的理解Spring Bean 生命周期是掌握Spring Bean依赖的前提，了解它也可以让我们在特定的节点做特定的事情满足业务需要
keywords: Spring Bean生命周期, @PreDestroy, DisposableBean, destroy-method
---

 上一篇文章 [Spring Bean 生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html) 说明了`我是谁？` 和 `我从哪里来?` 的两大哲学问题，今天我们要讨论一下终极哲学`我要到哪里去？`


 初始化 Spring Bean 有三种方式：
 1. @PostConstruct
 2. InitializingBean.afterPropertiesSet()
 3. init-method

销毁 Spring Bean 同样有三种方式：
1. @PreDestroy
2. DisposableBean.destroy()
3. destroy-method
 
> 正所谓，天对地，雨对风； @PostConstruct 对 @PreDestroy；InitializingBean.afterPropertiesSet() 对 DisposableBean.destroy()； init-method 对 destroy-method；雷隐隐，雾蒙蒙；山花对海树，赤日对苍穹；平仄平仄平平仄，仄平仄平仄仄平，仄仄平……

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-02/%E5%B2%B3%E4%BA%91%E9%B9%8F1.gif) </fancybox>


感觉讲到这没必要讲下去了，一切清晰明了，但我还有话要说

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-02/6bad7ce4875a2946.gif) </fancybox>

当 Spring Ioc 容器要移除 bean 时，销毁相关回调方法将会被执行，这么做的目的时释放被 bean 持有的资源，或者去执行一些终极任务. 当 ApplicationContext 调用 registerShutdownHook 方法时，这些销毁方法就会被触发，但一般正常的业务中很少会用到这些方法

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-11-16-43-10%402x.png)</fancybox>


接下来具体的展示一下三种方法的使用方式
## 三种销毁 Spring Bean 的方式

### DisposableBean
Spring 为我们提供了 DisposableBean 接口

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-11-16-43-30%402x.png)</fancybox>

我们可以通过实现 `DisposableBean` 接口，在其唯一方法 `destroy` 内完成 bean 销毁的工作，但是 Spring Framework 官方并不建议我们通过这种方法来销毁 bean，这同样是一种强耦合的方式，我们看到框架层面才会用到这个方法。
###  @PreDestroy
这种方式是 Spring 非常提倡的一种方式，我们通常将其标记在方法上即可，通常习惯将这个方法起名为 `destory()`

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-11-16-43-52%402x.png)</fancybox>

### destroy-method
同样是两种方式，第一种方式：


<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-11-16-44-04%402x.png)</fancybox>

第二种方式

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-11-16-44-15%402x.png)</fancybox>


以上三种 Bean 的销毁方式也是可以组合使用的，那么组合在一起的调用顺序是什么呢？
1. 首先 @PreDestroy 会被调用
2. 其次 DisposableBean.destroy() 会被调用 
3. 最后调用通过 XML 配置的 destroy-method 方法或通过设置 @Bean 注解 设置 destroyMethod 属性的方法

用图示来说明一下调用顺序

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-02/Xnip2019-07-02_16-00-23.jpg) </fancybox>


这个调用顺序也不难记忆
> PreDestroy `(P)`，destroy `(D)`，destroy-method `(D)` ---> `PDD （那个3亿人都在 pin 的那个 APP 名称，目前 3 亿人中没有我）`

再来看看 Spring Bean 生命周期图，我们已经点亮了核心部分：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-02/Xnip2019-07-02_16-01-53.jpg) </fancybox>

我们要注意，在销毁的过程并没有类似 BeanPostProcess 那中切面的思想，这里要注意到区别。

## 灵魂追问
1. 在阅读框架源码时，哪些地方用到了 bean 的销毁方法？
2. 还没有被点亮的地方，你认为还有哪些内容没有做？
