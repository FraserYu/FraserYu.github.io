---
title: Spring Bean生命周期之缘起
date: 2019-12-20 09:04:56
tags:
    - Spring Boot
    - Spring Bean
    - 生命周期
categories: [Coding, Spring-Boot]
id: spring-bean-lifecycle-creation
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: 透彻的理解Spring Bean 生命周期是掌握Spring Bean依赖的前提，了解它也可以让我们在特定的节点做特定的事情满足业务需要
keywords: Spring Bean生命周期, InitializingBean, @PostConstruct, init-method
---

Spring bean 的生命周期很容易理解。实例化 bean 时，可能需要执行一些初始化以使其进入可用 （Ready for Use）状态。类似地，当不再需要 bean 并将其从容器中移除时，可能需要进行一些清理，这就是它的生命周期

上一篇文章 [面试还不知道BeanFactory和ApplicationContext的区别？](https://dayarch.top/p/difference-between-beanfactory-and-applicationcontext.html) 中说明了接口 Beanfactory 和 Applicationcontext 可以通过 `T getBean(String name, Class<T> requiredType)` 方法从 Spring 容器中获取bean，区别是，前者是懒加载形式，后者是预加载的形式。那么问题来了：
> 这些 Spring Beans 是怎么生成出来的呢？ 

在正式回答这个问题之前，先解答一些有关 Java Bean， Spring Bean 和 Spring IoC 容器这些概念性的疑惑，我希望通过下面这个例子形象说明这些问题：
> 小学生 （Java Bean）通过提交资料申请（元数据配置）加入了少先队（Spring Ioc 容器），学习了一些精神与规定之后，变成了少先队员（Spring Bean）

从这里可以看出，Java Bean 和 Spring Bean 都是具有特定功能的对象，小学生还是那个小学生，只不过加入了少先队之后有了新的身份，新的身份要按照组织 （Spring Ioc）的规定履行特定义务

来看下图加深一下了解
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-07-01_20-14-05.jpg)</fancybox>

首先要有容器，实例化 Spring Ioc 容器是非常简单的，接口 `org.springframework.context.ApplicationContext` 表示Spring IoC容器，负责实例化，配置和组装上述 bean。 容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。 配置元数据通常以XML，Java 注解或代码的形式表示。 它允许你自己表达组成应用程序的对象以及这些对象之间丰富的相互依赖性，比如这样：

```java
ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"spring.xml", "spring1.xml"});
```

> 有了容器，我们需要做哪些处理，使其内部对象变为 `Ready for Use` 的状态?  

我们需要通过 Spring 容器实例化它们，Spring 为我们提供了三种方式：

## 三种初始化方式
### InitializingBean
Spring 为我们提供了 `InitializingBean` 接口
```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```
我们可以通过实现 `InitializingBean` 接口，在其唯一方法 `afterPropertiesSet` 内完成实例化的工作，但是 Spring Framework 官方并不建议我们通过这种方法来完成 Bean 的实例化，这是一种强耦合的方式，我们看到框架层面才会用到这个方法。

### @PostConstruct
这种方式是 Spring 非常提倡的一种方式，我们通常将其标记在方法上即可，通常习惯将这个方法起名为 `init()`
```java
@PostConstruct
public void init() {
  System.out.println("Inside init() method...");
}
```
### init-method
你应该见过这种初始化方式：
```java
public class MyClass {
   public void init() {
      // perform post-creation logic here
   }
}

@Configuration
public class AppConfig {
   @Bean(initMethod = "init")
   public MyClass myclass() {
      return new MyClass ();
   }
}
```
你也应该见过这种配置方式：

```xml
<bean id="myClass" class="com.demo.MyClass" init-method="init"/>
```

没错，这只是同样功能的不同实现方式罢了
以上就是三种初始化 Spring Beans 的方式，我们在框架中看到过三种方式在组合使用，那么组合使用的调用顺序是什么呢？
1. 首先@PostConstruct 会被最先调用
2. 其次 `InitializingBean.afterPropertiesSet()` 方法将会被调用 
3. 最后调用通过 XML 配置的 init-method 方法或通过设置 @Bean 注解 设置 initMethod 属性的方法

了解了这些，你也就了解了 Spring Bean 是怎么来的了

通过图示来说明一下：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-07-01_21-46-24.jpg)</fancybox>

这个调用顺序很难记忆吗吗？ 
> PostConstruct `(P)`，afterPropertiesSet `(A)`，init-method `(I)` ---> `PAI （圆周率π）`

## BeanPostProcessor
BeanPostProcessor 接口，大家也应该有印象，里面只有两个方法：
```java
public interface BeanPostProcessor {
    Object postProcessBeforeInitialization(Object var1, String var2) throws BeansException;

    Object postProcessAfterInitialization(Object var1, String var2) throws BeansException;
}
```
看方法名，BeforeInitialization 和 AfterInitialization，我们应该猜得出，这是在上述三种方式的前和后，算是一种全局的切面思想，我们经常会使用 `postProcessAfterInitialization` 方法，通过读取 Bean 的注解完成一些后续逻辑编写与属性的设定，现在 `Ready for Use`之前是这样：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-07-02_08-38-03.jpg)</fancybox>

在 `Ready for Use` 之前，了解这些内容，已可以基本满足日常的工作内容，但这并不是 Ready for Use 的全部内容，Spring Bean 整个生命周期的流程应该是这样的，后续文章会逐步点亮：
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-07-02_09-34-02.jpg)</fancybox>

## 灵魂追问
1. 了解了 Spring Bean 是怎么来的？那它是怎么没的呢？什么时候需要销毁他们呢？
2. Spring 框架中 XxxxAware，这些类有什么作用，能在 `Ready for Use` 之前有用处吗？
3. 你日常的工作中有充分利用今天说明的这些内容吗？懂得这些会大大方便你的编程

## 补充说明
1. 虽然当下流行以注解声明方式进行编程，甚至高版本 Spring 会将一些方法标记为过时，但文章说明依旧会使用 `XMLBeanFactory` 这类方法，包括 XML 配置。这样做，只不过为了更清晰的说明问题。
2. 另外将 Spring Bean 声明周期的讲解，进行拆分，是为了让大家有独立的思考空间，带着问题去思考、时间，而不是被动的填充，最终串联起自己的学习网络，这样理解的更深刻，具体请看之前写的文章  [程序猿为什么要看源码](https://mp.weixin.qq.com/s/V7h8O6pVFQ-nr_iA2SNqtw), 后续内容请持续关注