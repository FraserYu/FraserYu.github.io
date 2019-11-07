---
title: Java String基础面试
tags:
  - JVM
categories: [Coding, Java]
id: java-string-interview
date: 2019-08-31 11:57:49
description: Java String面试内容基本上都在这里
keywords: String,new String,几个对象,常量池,面试

---


关于 Java String，这是面试的基础，但是还有很多童鞋不能说清楚，所以本文将简单而又透彻的说明一下那个让你迷惑的 String

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/mountain-landscape-2031539_1920.jpg)


在 Java 中，我们有两种方式创建一个字符串
```java
String x = "abc";
String y = new String("abc");
```
你常见也常写第一种，很少见第二种，但面试还总问这类问题，双引号和构造器两种形式创建字符串到底有什么差别呢？

先来看例子
### 例子 1
```java
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
```
`a == b` 结果为 true，是因为 a 和 b 都指向 **方法区(method area)** 同一个字符串文字，内存引用是同一个

当多次创建相同的字符串文字时，只存储每个不同字符串值的一个副本。这个叫做**字符串留驻/留用**，Java 中所有编译期字符串常量都会被自动留驻

### 例子 2
```java
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d);  // False
System.out.println(c.equals(d)); // True
```
`c==d` 结果为 false，因为 c 和 d 的引用指向**堆**中不同的对象，`不同的对象肯定有不同的内存引用`

举了两个例子，文字描述有点懵？我们来试图通过图形来理解上述两种情况:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/2019-08-30-18-44-15.jpg)


也许你已经看看出来了，一个是在`方法区`，一个是在`堆`中，在 JVM 模型中这是两个不同的区域，也许你面试时也经常被问到吧，来看下图:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/2019-08-30-18-47-02.jpg)

再次提醒一下，所有 new 的对象都会在 Heap 中，这样以后你就好区分了

## 运行期字符串留驻
上面说的字符串留驻是在编译期，那么运行期可以吗？答案是肯定的，我们需要一个函数来帮忙
```java
String c = new String("abcd").intern();
String d = new String("abcd").intern();
System.out.println(c == d);  // Now true
System.out.println(c.equals(d)); // True
```
看到 `c == d` 结果为 true，你应该理解 intern (英文有拘留，软禁的意思)的作用了，通过调用 intern()方法，就好比把创建的字符串拘留在方法区一样了

在面试时甚至还会问你下面代码创建了几个对象:
```java
String d = new String("abcd")
```
1. 如果方法区已存在"abcd", 那么只创建一个 new String 的对象
2. 如果方法区没有"abcd", 那么要创建两个对象，一个在方法区，一个在堆中

所以，**正常情况**下我们没必要使用构造器创建对象，因为这很可能会产生一个额外的没用的对象，但是有例外哦，我们下面说

```java
String s = "abcd";
s = s.concat("ef");
```

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/2019-08-30-20-38-21.jpg)

当我们想在字符串 s 后面拼接字符"ef"时，会在堆中创建一个新的对象，并将 s 的引用指向新创建的对象，由于 String 创建的是不可变对象，所以 **String 类中的所有方法都不会改变它自身，而是返回一个新的字符串(快打开你的 IDE，看看是否每个操作String 的方法最后都是返回有 return new String 字样)**，到这里你也应该理解了一个道理:
>  如果我们需要一个字符串被修改，我们最好使用 StringBuffer 或者 StringBuilder，否则，由于每次操作字符串都会创建一个新的对象，而旧的对象不会有引用指向它，这样我们会浪费很多垃圾回收的时间

到这里还没完，你有没有想过为什么 String 会被设置/制造成 final？
## 为什么 String 类被 final 修饰
谈及这个问题我们需要一些倒推的或者相互约束思维来思考
### 字符串池的需求
字符串池(String intern pool)是方法区域中的一个特殊存储区域。当创建一个字符串时，如果该字符串已经存在于池中，那么返回现有字符串的引用，而不是创建一个新对象。所以说，如果一个字符串是可变的，那么改变一个引用的值，将导致原本指向该值的引用获取到错误的值

### 缓存 hashcode
字符串的hashcode在Java中经常使用。例如，在HashMap或HashSet中。不可变保证hashcode始终是相同的，这样就可以在不担心更改的情况下兑现它。这意味着，不需要每次使用hashcode时都计算它。这样更有效率。所以你会在 String 类中看到下面的成员变量的定义:
```java
/** Cache the hash code for the string */
private int hash; // Default to 0
```

### 安全性
String被广泛用作许多java类的参数，例如网络连接、打开文件等。如果字符串不是不可变的，连接或文件将被更改，这可能导致严重的安全威胁。该方法认为它连接到一台机器上，但实际上并没有。**可变字符串也可能导致反射中的安全问题，因为参数是字符串**。

### 不可变对象天生是线程安全的
由于不可变对象不能被更改，所以它们可以在多个线程之间自由共享。这消除了同步的需求。

总之，出于效率和安全性的考虑，String 被设计为不可变的。这也是为什么在一般情况下，不可变类是首选的原因。

## 附加说明
关于不可变对象和不可变引用总是有同学搞不清楚
```java
final User user = new User();
```
上面的代码指的是 user 引用不能被更改指向内存的其他地址，但是由于 User 是可变对象，我们可以调用 user 的 setter 方法修改其属性

在String类中包含很多学问，包括你对JVM模型的理解，这也就是为什么面试官为什么喜欢问String，主要考察你的基本功

## 灵魂追问
1. String 和基本类型的包装类如 Integer 和 Long 都被 final 修饰，但为什么不建议作为 synchronized 同步块的参数适用呢？
2. 基本类型自动装箱你知道发生了什么吗？和上一个问题有关系

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)

### Material Theme UI
这是一款 IDEA 的主题插件，安装后，选择 `Material Palenight` 主题，同时作出如下设置
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/2019-08-31-11-43-17.png)

设置完后，你的 IDEA 就是下面这样，引起极度舒适
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-08-30/2019-08-31-11-39-58.jpg)



--------

## 推荐阅读
+ [每天用SpringBoot，还不懂RESTful API返回统一数据格式是怎么实现的？ ](https://mp.weixin.qq.com/s/q5cyK14WGiMm-R6NIFBcTg)
+ [双亲委派模型：大厂高频面试题，轻松搞定](https://mp.weixin.qq.com/s/Dnr1jLebvBUHnziZzSfcrA)
+ [面试还不知道BeanFactory和ApplicationContext的区别？](https://mp.weixin.qq.com/s/YBQB086ADBjHUmwrFQrWew)
+ [如何设计好的RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ)
+ [红黑树，超强动静图详解，简单易懂](https://mp.weixin.qq.com/s/ilND8u_8HGSTSrJiMB4X8g)

--------
> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享
> - 高效工具汇总 | 回复「工具」
> - 面试问题分析与解答
> - 技术资料领取 | 回复「资料」

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)
