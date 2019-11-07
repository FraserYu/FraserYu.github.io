---
title: 有序性可见性，Happens-before来搞定
tags:
  - Happens-before
category: [Coding, Java-Concurrency]
summary: 'Happens-before是解决并发编程可见性有有序性的核心约束，图形与代码展示将不再抽象,让你透过现象看本质'
id: java-concurrency-happens-before-rule
date: 2019-09-12 18:24:22
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgconcurrency.png
description: Happens-before是解决并发编程可见性有有序性的核心约束，图形与代码展示将不再抽象,让你透过现象看本质
keywords: happens-before,Java并发,有序性,可见性,内存屏障
---

## 写在前面
上一篇文章[并发 Bug 之源有三，请睁大眼睛看清它们](https://mp.weixin.qq.com/s/fQcozKziqMxFMBkptpnuTw) 谈到了`可见性/原子性/有序性`三个问题，这些问题通常违背我们的直觉和思考模式，也就导致了很多并发 Bug

- 为了解决 CPU，内存，IO 的短板，增加了缓存，但这导致了可见性问题
- 编译器/处理器`擅自`优化 ( Java代码在编译后会变成 Java 字节码, 字节码被类加载器加载到 JVM 里,  JVM 执行字节码,  最终需要转化为汇编指令在 CPU 上执行) ，导致有序性问题

初衷是好的，但引发了新问题，最有效的办法就禁止缓存和编译优化，问题虽然能解决，但「又回到最初的起点，呆呆地站在镜子前」是很尴尬的，我们程序的性能就堪忧了.

## 解决方案
1. 作为我们程序猿不想写出 bug 影响 KPI，所以希望内存模型易于理解、易于编程。这就需要基于一个**强内存模型**来编写代码
2. 作为编译器和处理器不想让外人说它处理速度很慢，所以希望内存模型对他们束缚越少越好，可以由他们**擅自优化**，这就需要基于一个**弱内存模型**

> 俗话说:「没有什么事是开会解决不了的，如果有，那就再开一次」😂

JSR-133 的专家们就有了新想法，既然不能完全禁止缓存和编译优化，那就**按需**禁用缓存和编译优化，按需就是要加一些约束，约束中就包括了上一篇文章简单提到过的 **volatile，synchronized，final** 三个关键字，同时还有你可能听过的 **Happens-Before** 原则(包含可见性和有序性的约束)，Happens-before 规则也是本章的主要内容

为了满足二者的强烈需求，照顾到双方的情绪，于是乎: JMM 就对程序猿说了一个善意的谎言: 「会严格遵守 Happpen-Befores 规则，不会重排序」让程序猿放心，私下却有自己的策略:
1. 对于会改变程序执行结果的重排序,JMM要求编译器和处理器必须禁止这种重排序。
2. 对于不会改变程序执行结果的重排序, JMM对编译器和处理器不做要求 (JMM允许这种重排序)。

我们来用个图说明一下:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%A4%9A%E7%BA%BF%E7%A8%8B/_image/2019-09-05/JMM%E4%BF%9D%E8%AF%81%20%281%29.png)


> 这就是那个善意的谎言，虽是谎言，但还是照顾到了程序猿的利益，所以我们只需要了解 happens-before 规则就能得到保证 (图画了好久，不知道是否说明了谎言的所在😅，欢迎留言)

## Happens-before
Happens-before 规则主要用来约束两个操作，两个操作之间具有 happens-before 关系, 并不意味着前一个操作必须要在后一个操作之前执行，**happens-before 仅仅要求前一个操作(执行的结果)对后一个操作可见**,   (the first is **visible** to and ordered before the second)

说了这么多，先来看一小段代码带你逐步走进 Happen-Befores 原则，看看是怎样用该原则解决 **可见性** 和 **有序性** 的问题:

```java
class ReorderExample {
  int x = 0;
  boolean flag = false;
  public void writer() {
    x = 42;    //1
    flag = true;    //2
  }
  public void reader() {
    if (flag) { //3
      System.out.println(x);    //4
    }
  }
}
```

假设 A 线程执行 writer 方法，B 线程执行 reader 方法，打印出来的 x 可能会是 0，上一篇文章说明过: 因为代码 1 和 2 没有**数据依赖**关系，所以可能被重排序

```java
flag = true;    //2
x = 42;    //1
```

所以，线程 A 将 `flag = true` 写入**但没有为 x 重新赋值时**，线程 B 可能就已经打印了 x 是 0

那么为 flag 加上 volatile 关键字试一下:

```java
volatile boolean flag = false;
```

*即便加上了 volatile 关键字，这个问题在 java1.5 之前还是没有解决，但 java1.5 和其之后的版本对 volatile 语义做了增强*，问题得以解决，这就离不开 Happens-before 规则的约束了，总共有 6 个规则，且看

### 程序顺序性规则
>  **一个线程中**的每个操作, happens-before 于该线程中的任意后续操作
第一感觉这个原则是一个在理想状态下的"废话"，并且和上面提到的会出现重排序的情况是矛盾的，注意这里是一个线程中的操作，其实隐含了「as-if-serial」语义: 说白了就是只要执行结果不被改变，无论怎么"排序"，都是对的

这个规则是一个基础规则，happens-before 是多线程的规则，所以要和其他规则约束在一起才能体现出它的顺序性，别着急，继续向下看

### volatile变量规则
>  对一个 volatile 域的写, happens-before 于任意后续对这个 volatile 域的读

我将上面的程序添加两行代码作说明:

```java
public class ReorderExample {

	private int x = 0;
	private int y = 1;
	private volatile boolean flag = false;

	public void writer(){
		x = 42;	//1
		y = 50;	//2
		flag = true;	//3
	}

	public void reader(){
		if (flag){	//4
			System.out.println("x:" + x);	//5
			System.out.println("y:" + y);	//6
		}
	}
}
```

这里涉及到了 volatile 的内存增强语义，先来看个表格:

| 能否重排序  |第二个操作   |第二个操作   |第二个操作   |
|---|---|---|---|
| 第一个操作  |普通读/写   | volatile 读  | volatile 写  |
|普通读/写   |  - | -  | NO  |
|volatile 读   | NO  | NO  | NO  |
| volatile 写  | -  | NO  | NO  |

从这个表格 **最后一列** 可以看出:
>  如果第二个操作为 volatile 写，不管第一个操作是什么，都不能重排序，**这就确保了 volatile 写之前的操作不会被重排序到 volatile 写之后**
拿上面的代码来说，代码 1 和 2 不会被重排序到代码 3 的后面，但代码 1 和 2 可能被重排序 (没有依赖也不会影响到执行结果)，说到这里和 **程序顺序性规则**是不是就已经关联起来了呢？

从这个表格的 **倒数第二行** 可以看出:
>  如果第一个操作为 volatile 读，不管第二个操作是什么，都不能重排序，**这确保了 volatile 读之后的操作不会被重排序到 volatile 读之前**
拿上面的代码来说，代码 4 是读取 volatile 变量，代码 5 和 6 不会被重排序到代码 4 之前

*volatile 内存语义的实现是应用到了 「内存屏障」，因为这完全够单独写一章的内容，这里为了不掩盖主角 Happens-before 的光环，保持理解 Happens-before 的连续性，先不做过多说明*

到这里，看这个规则，貌似也没解决啥问题，因为它还要联合第三个规则才起作用
### 传递性规则
>  如果 A happens-before B, 且 B happens-before C, 那么 A happens-before C
直接上图说明一下上面的例子

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%A4%9A%E7%BA%BF%E7%A8%8B/_image/2019-09-05/%E7%BA%BF%E7%A8%8B%E5%88%87%E6%8D%A2%E5%9B%BE.png)

从上图可以看出
- `x =42` 和 `y = 50` Happens-before `flag = true`, 这是**规则 1**
- 写变量(代码 3) `flag=true` Happens-before 读变量(代码 4) `if(flag)`，这是**规则 2**

根据**规则 3**传递性规则，`x =42` Happens-before 读变量 `if(flag)`

> **谜案要揭晓了**: 如果线程 B 读到了 flag 是 true，那么 `x =42` 和 `y = 50` 对线程 B 就一定可见了，这就是 Java1.5 的增强 (之前版本是可以普通变量写和 volatile 变量写的重排序的)

通常上面三个规则是一种联合约束，到这里你懂了吗？规则还没完，继续看

### 监视器锁规则
>  对一个锁的解锁 happens-before 于随后对这个锁的加锁

这个规则我觉得你应该最熟悉了，就是解释 synchronized 关键字的，来看

```java
public class SynchronizedExample {
	private int x = 0;

	public void synBlock(){
		// 1.加锁
		synchronized (SynchronizedExample.class){
			x = 1; // 对x赋值
		}
		// 3.解锁
	}

	// 1.加锁
	public synchronized void synMethod(){
		x = 2; // 对x赋值
	}
	// 3. 解锁
}
```

先获取锁的线程，对 x 赋值之后释放锁，另外一个再获取锁，一定能看到对 x 赋值的改动，就是这么简单，请小伙伴用下面命令查看上面程序，看同步块和同步方法被转换成汇编指令有何不同？

```shell
javap -c -v SynchronizedExample
```

这和 synchronized 的语义相关，小伙伴可以先自行了解一下，锁的内容时会做详细说明

### start()规则
>  如果线程 A 执行操作 ThreadB.start() (启动线程B),  那么 A 线程的 ThreadB.start() 操作 happens-before 于线程 B 中的任意操作，也就是说，主线程 A 启动子线程 B 后，子线程 B 能看到主线程在启动子线程 B 前的操作，看个程序就秒懂了

```java
public class StartExample {
	private int x = 0;
	private int y = 1;
	private boolean flag = false;

	public static void main(String[] args) throws InterruptedException {
		StartExample startExample = new StartExample();

		Thread thread1 = new Thread(startExample::writer, "线程1");
		startExample.x = 10;
		startExample.y = 20;
		startExample.flag = true;

		thread1.start();

		System.out.println("主线程结束");
	}

	public void writer(){
		System.out.println("x:" + x );
		System.out.println("y:" + y );
		System.out.println("flag:" + flag );
	}
}
```

运行结果:

```shell
主线程结束
x:10
y:20
flag:true

Process finished with exit code 0
```

线程 1 看到了主线程调用 thread1.start() 之前的所有赋值结果，这里没有打印「主线程结束」，你知道为什么吗？这个守护线程知识有关系

### join()规则
> 如果线程 A 执行操作 ThreadB.join() 并成功返回, 那么线程 B 中的任意操作 happens-before 于线程 A 从 ThreadB.join() 操作成功返回，**和 start 规则刚好相反**，主线程 A 等待子线程 B 完成，当子线程 B 完成后，主线程能够看到子线程 B 的赋值操作，将程序做个小改动，你也会秒懂的

```java
public class JoinExample {
	private int x = 0;
	private int y = 1;
	private boolean flag = false;

	public static void main(String[] args) throws InterruptedException {
		JoinExample joinExample = new JoinExample();

		Thread thread1 = new Thread(joinExample::writer, "线程1");
		thread1.start();

		thread1.join();

		System.out.println("x:" + joinExample.x );
		System.out.println("y:" + joinExample.y );
		System.out.println("flag:" + joinExample.flag );
		System.out.println("主线程结束");
	}

	public void writer(){
		this.x = 100;
		this.y = 200;
		this.flag = true;
	}
}
```

运行结果:

```shell
x:100
y:200
flag:true
主线程结束

Process finished with exit code 0
```

「主线程结束」这几个字打印出来喽，依旧和线程何时退出有关系


## 总结
1. **Happens-before 重点是解决前一个操作结果对后一个操作可见**，相信到这里，你已经对 Happens-before 规则有所了解，这些规则解决了多线程编程的可见性与有序性问题，但还没有完全解决原子性问题(除了 synchronized)
2. start 和 join 规则也是解决主线程与子线程通信的方式之一
3. 从内存语义的角度来说, volatile 的`写-读`与锁的`释放-获取`有相同的内存效果；volatile 写和锁的释放有相同的内存语义; volatile 读与锁的获取有相同的内存语义，⚠️⚠️⚠️(敲黑板了) volatile 解决的是可见性问题，synchronized 解决的是原子性问题，这绝对不是一回事，后续文章也会说明

## 附加说明
1. 个人博客会首发原创文章，如文章有密码保护，通过查看文章的**创建日期**，如果是 1 月份，请公众号回复当前月份数值即可，如回复「1」
2. 多线程系列文章整体会按照我的大纲节奏来写，但是如果大家有什么疑问，也欢迎到我单独建立的[多线程系列问题留言汇总](https://dayarch.top/2019/09/11/duo-xian-cheng-xi-lie-wen-ti-liu-yan-hui-zong/) 文章中留言，我会统一回复，如果共通疑问很多，我会插入相关章节单独做说明
3. 并发文章的相关代码，我也会同步上传到代码库，公众号回复「demo」，点击链接，找到concurrency 子项目即可
4. 如果文章对你有帮助，烦请小伙伴转发分享给更多朋友，我们一起进步

## 灵魂追问
1. 同步块和同步方法在编译成 CPU 指令后有什么不同？
2. 线程有 Daemon(守护线程)和非 Daemon 线程，你知道线程的退出策略吗？
3. 关于 Happens-before 你还有哪些疑惑呢？

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)

### MarkDown 表格生成器
本文的好多表格是从官网粘贴的，如何将其直接转换成 MD table 呢？那么 https://www.tablesgenerator.com/markdown_tables 就可以帮到你了，无论是生成 MD table，还是粘贴内容生成 table 和内容都是极好的，当然了不止 MD table，自己发现吧，更多工具，公众号回复 「工具」获得

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-10-52-55.png)

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
