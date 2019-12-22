---
title: 面试还不知道BeanFactory和ApplicationContext的区别？
date: 2019-12-19 21:37:30
tags:
    - 面试
    - Spring Boot
categories: [Coding, Spring-Boot]
id: difference-between-beanfactory-and-applicationcontext
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: BeanFactory和ApplicationContext是Sring容器的核心，面试经常会问二者区别，本文会透彻说明
keywords: BeanFactory, ApplicationContext, Spring容器, Spring懒加载
---

## 前言
> 接口 BeanFactory 和 ApplicationContext 都是用来从容器中获取 Spring beans 的，但是，他们二者有很大不同

我看到过很多问 BeanFactory 和 ApplicationContext 不同点的问题，考虑到这，我应该使用前者还是后者从 Spring 容器中获取 beans 呢？请向下看

## 什么是 Spring Bean
这是一个非常简单而又很复杂的问题，通常来说，Spring beans 就是被 Spring 容器所管理的 Java 对象，来看一个简单的例子

```java
package com.zoltanraffai;  
public class HelloWorld { 
   private String message;  
   public void setMessage(String message){ 
      this.message  = message; 
   }  
   public void getMessage(){ 
      System.out.println("My Message : " + message); 
   } 
}
```

在基于 XML 的配置中， beans.xml 为 Spring 容器管理 bean 提供元数据

## 什么是 Spring 容器
Spring 容器负责实例化，配置和装配 Spring beans，下面来看如何为 IoC 容器配置我们的 HelloWorld POJO

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <bean id = "helloWorld" class = "com.zoltanraffai.HelloWorld">
      <property name = "message" value = "Hello World!"/>
   </bean>
</beans>
```

现在，它已经被 Spring 容器管理了，接下来的问题是：我们怎样获取它？

## BeanFactory 和 ApplicationContext 的不同点
### BeanFactory 接口
这是一个用来访问 Spring 容器的 root 接口，要访问 Spring 容器，我们将使用 Spring 依赖注入功能，使用 BeanFactory 接口和它的子接口

**特性：**
- Bean 的实例化/串联
通常情况，BeanFactory 的实现是使用懒加载的方式，这意味着 beans 只有在我们通过 getBean() 方法直接调用它们时才进行实例化
实现 BeanFactory 最常用的 API 是 XMLBeanFactory
这里是如何通过 BeanFactory 获取一个 bean 的例子：

```java
package com.zoltanraffai;  
import org.springframework.core.io.ClassPathResource;  
import org.springframework.beans.factory.InitializingBean; 
import org.springframework.beans.factory.xml.XmlBeanFactory; 
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      XmlBeanFactory factory = new XmlBeanFactory (new ClassPathResource("beans.xml")); 
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");    
      obj.getMessage();    
   }
}
```

### ApplicationContext 接口
ApplicationContext 是 Spring 应用程序中的中央接口，用于向应用程序提供配置信息
它继承了 BeanFactory 接口，所以 ApplicationContext 包含 BeanFactory 的所有功能以及更多功能！它的主要功能是支持大型的业务应用的创建

**特性：**
- Bean instantiation/wiring
- Bean 的实例化/串联
- 自动的 BeanPostProcessor 注册
- 自动的 BeanFactoryPostProcessor 注册
- 方便的 MessageSource 访问（i18n）
- ApplicationEvent 的发布

与 BeanFactory 懒加载的方式不同，它是预加载，所以，每一个 bean 都在 ApplicationContext 启动之后实例化

这里是 ApplicationContext 的使用例子：

```java
package com.zoltanraffai;  
import org.springframework.core.io.ClassPathResource;  
import org.springframework.beans.factory.InitializingBean; 
import org.springframework.beans.factory.xml.XmlBeanFactory; 
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml"); 
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");    
      obj.getMessage();    
   }
}
```

## 总结
ApplicationContext 包含 BeanFactory 的所有特性，通常推荐使用前者。但是也有一些限制情形，比如移动应用内存消耗比较严苛，在那些情景中，使用更轻量级的 BeanFactory 是更合理的。然而，在大多数企业级的应用中，ApplicationContext 是你的首选。

## 灵魂追问
- 如何使用 BeanPostProcessor 和 BeanFactoryPostProcessor ？
- 你了解 Spring Bean 的生命周期吗？了解了这些对与 bean 的使用将有非常大的帮助.