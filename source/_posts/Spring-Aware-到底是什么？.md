---
title: Spring Aware 到底是什么？
date: 2019-12-22 19:20:52
tags:
    - Spring Aware
    - Spring Bean
    - 生命周期
categories: [Coding, Spring-Boot]
id: spring-aware
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: Spring 源码中大量应用Aware，通过本文你将透彻理解 Spring Aware背后的道理，以及如何应用它
keywords: Spring Aware，BeanNameAware，BeanFactoryAware
---

通过如下前序两篇文章:
1. [Spring Bean 生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html)
2. [Spring Bean 生命周期之缘尽](https://dayarch.top/p/spring-bean-lifecycle-destroy.html)
我们了解了 Spring Bean 的生命周期核心内容，bean 是如何被初始化变为 `Ready for Use` 的状态，当资源被回收时又是如何被 destroy 的，但 `Spring Bean Life Cycle`图并未被全部点亮，这篇文章将点亮剩余内容，同时说说你常见的 XxxxAware 接口

为什么要说 Spring Bean 生命周期又说 Aware 呢？下来点亮剩下内容你也许就明白了：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-03/Xnip2019-07-03_13-45-04.jpg)</fancybox>

1. 在 Spring Bean `Ready for Use`之前的起源当然是要调用构造器，所以 Constructor 毋庸置疑是创建 Spring Bean 的第一步
2. 通过 Setter 方法完成依赖注入，SDI （Setter Dependency Injection）
3. 依赖注入一旦结束，`BeanNameAware.setBeanName()` 会被调用，它设置该 bean 在 Bean Factory 中的名称
4. 接下来调用 `BeanClassLoaderAware.setBeanClassLoader() `，为 bean 实例提供类加载器，我们知道所有类都是要通过类加载器加载到上下文的，关于类的加载机制/双亲委派模型（大厂都爱问的面试题）内容会在后续给出来，让你透彻的了解
5. 然后 `BeanFactoryAware.setBeanFactory()` 会被调用为 bean 实例提供其所拥有的 factory

关于 1、2 两点我要额外多说一些内容，请看下面代码：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-17-44%402x.png)</fancybox>

这里，我们尝试通过构造器访问自动注入的 field `Environment env`，当构造器被调用时，Spring Bean 还没被完全初始化，这就会导致 `NullPointerExceptions`；
我们变换一下方式：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-19-37%402x.png)</fancybox>

这种方式，Environment 实例被安全注入之后才调用 `@PostConstruct`标记的方法，这样就不会抛出 `NullPointerException` 了。

这会回看周期图，有没有豁然开朗？

> 等所有 Spring Bean 都完成依赖注入（周期图中的 Setter Methods 部分）再使用 bean 的引用才是安全的方式，

后续会有一个章节专门说一说面试经常被问起的 `Spring 有几种依赖注入方式`的尴尬问题，请关注后续文章

到这里终于可以说一说 Aware 了，且看

## Aware
Aware 是 Spring 中的一个根接口，继承该接口的子接口有很多，比如周期图中的那三个 Aware，但是该接口没有任何方法，所以大家可以把它理解成一个标记接口：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-20-29%402x.png)</fancybox>

Aware 翻译过来可以理解为"察觉的；注意到的；感知的" ，XxxxAware 也就是对....感知的，没有 Aware 就是无感知的吗？对喽

> Spring 的依赖注入最大亮点就是所有的 Bean 对 Spring 容器的存在是没有意识的，拿 [Spring Bean 生命周期之“我从哪里来”？]() 文章中“小学生入少先队”为例子说明，小学生还是那个小学生，加入少先队还是加入共青团只不过规则不一样罢了
但是在实际项目中，我们不可避免的要用到 Spring 容器本身提供的资源（难免要有事情需要少先队组织的帮助），这时候要让 Bean 主动意识到 Spring 容器的存在，才能调用 Spring 所提供的资源，这就是 Spring Aware. 其实 Spring Aware 是 Spring 设计为框架内部使用的，若使用了，你的 Bean 将会和 Spring 框架耦合，所以自己不单独使用，但是在读框架源码时希望你不再模糊.

常见的 Spring Aware 接口


| Aware子接口    |描述     |
| :-- | :-- |
|  BeanNameAware   | 获取容器中 Bean 的名称    |
|   BeanFactoryAware  |   获取当前 BeanFactory ，这样可以调用容器的服务  |
|  ApplicationContextAware   |  同上，在[BeanFactory 和 ApplicationContext 的区别]() 中已明确说明   |
|   MessageSourceAware  | 获取 Message Source 相关文本信息    |
|  ApplicationEventPublisherAware   | 发布事件    |
|   ResourceLoaderAware  |  获取资源加载器，这样获取外部资源文件   |



来看类关系图：

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-03/Xnip2019-07-03_14-39-56.jpg)</fancybox>

当然不止以上这些 Aware， 通常使用 Spring Aware 的目的是为了让 Bean 获得 Spring 容器的服务。

## 代码示例
### BeanNameAware
自定义 bean 实现 BeanNameAware

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-21-09%402x.png)</fancybox>

注册 bean

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-21-51%402x.png)</fancybox>

运行

<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07/2019-07-09-19-22-45%402x.png)</fancybox>

和预想一样，Bean Name 输出结果为 `myCustomBeanName`，如果移除掉 @Bean 注解的 name 属性， 输出结果为 `getMyBeanName`

## 总结
在大多数情况下，我们应该避免使用任何 Aware 接口，除非我们需要它们。实现这些接口会将代码耦合到Spring框架，但是希望看过本节内容之后阅读框架源码思维更加清晰

## 灵魂追问
1. 框架中有哪些经典的 Aware 应用？
2. 到现在你能很好的理解 Spring Bean 的生命周期吗？

## Demo代码
涉及到 Spring Bean 生命周期的测试代码由于内容较多，没有写在此处，关注公众号并回复 「demo」获取相关代码，请自行尝试运行结果
