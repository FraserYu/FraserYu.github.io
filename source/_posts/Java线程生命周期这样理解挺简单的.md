---
title: Java线程生命周期这样理解挺简单的
date: 2020-03-25 14:32:03
tags:
    - Java-Concurrency
categories: [Coding, Java-Concurrency]
id: java-thread-life-cycle
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgconcurrency.png
description: 了解Java线程的生命周期是编写并发程序的基础，如何透彻的理解，它们的状态如何转换，如果你始终不能理解或记忆这些问题，相信这篇文章会帮你解惑
keywords: Java线程生命周期,线程生命周期,thread lifecycle,Java并发编程
---


<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200325143022.png)</fancybox>

## 为什么要了解线程的生命周期？

之前写过 Spring Bean 生命周期三部曲：

1. [Spring Bean生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html)
2. [Spring Bean生命周期之缘尽](https://dayarch.top/p/spring-bean-lifecycle-destroy.html)
3. [Spring Aware 到底是什么？](https://dayarch.top/p/spring-aware.html)



有朋友留言说：“了解了它们的生命周期后，使用 Spring Bean 好比看到它们的行动轨迹，现在使用就一点都不慌了”。我和他一样，了解事物的生命周期目的很简单，唯【不慌】也

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img不慌.gif)</fancybox>



[Java 并发系列](https://dayarch.top/categories/Coding/Java-Concurrency/) 已经写了很多，从来还没提起过那个它【Java线程生命周期】。有了前序理论图文的铺垫，在走进源码世界之前，谈论它的时机恰好到了。因为，编写并发程序的核心之一就是正确的摆弄线程状态



## 线程生命周期的几种状态

刚接触线程生命周期时，我总是记不住，也理解不了他们的状态，可以说是比较混乱，更别说它们之间是如何进行状态转换的了。原因是我把`操作系统通用线程状态`和`编程语言封装后的线程状态` 概念混淆在一起了



### 操作系统通用线程状态

个人觉得通用线程状态更符合我们的思考习惯。其状态总共有 5 种 （如下图）。对于经常写并发程序的同学来说，其嘴里经常念的都是操作系统中的这些通用线程状态，且看

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200322162836.png)</fancybox>



除去生【初始状态】死【终止状态】，其实只是三种状态的各种转换，听到这句话是不是心情放松了很多呢？



为了更好的说明`通用线程状态`和 `Java 语言中的线程状态`，这里还是先对前者进行简短的说明 



> 初始状态

线程已被创建，但是还不被允许分配CPU执行。**注意，这个被创建其实是属于编程语言层面的，实际在操作系统里，真正的线程还没被创建**， 比如 Java 语言中的 new Thread()。



> 可运行状态

线程可以分配CPU执行，**这时，操作系统中线程已经被创建成功了**



> 运行状态

操作系统会为`处在可运行状态的线程`分配CPU时间片，被 CPU 临幸后，处在可运行状态的线程就会变为运行状态



> 休眠状态

如果处在运行状态的线程调用`某个阻塞的API`或`等待某个事件条件可用`，那么线程就会转换到休眠状态，**注意：此时线程会释放CPU使用权，休眠的线程永远没有机会获得CPU使用权，只有当等待事件出现后，线程会从休眠状态转换到可运行状态**



> 终止状态

`线程执行完`或者`出现异常` (被interrupt那种不算的哈，后续会说)就会进入终止状态，正式走到生命的尽头，没有起死回生的机会



接下来就来看看你熟悉又陌生，面试又经常被问到的Java 线程生命周期吧



### Java语言线程状态

在 Thread 的源码中，定义了一个枚举类 State，里面清晰明了的写了Java语言中线程的6种状态：

1. NEW
2. RUNNABLE
3. BLOCKED
4. WAITING
5. TIMED_WAITING
6. TERMINATED

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-03-22_16-01-54.jpg)</fancybox>



这里要做一个小调查了，你有查看过这个类和读过其注释说明吗？（欢迎留言脚印哦）

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200325112648.png)</fancybox>

耳边响起五环之歌，Java中线程状态竟然比通用线程状态的 5 种多1种，变成了 6 种。这个看似复杂，其实并不是你想的那样，Java在通用线程状态的基础上，有裁剪，也有丰富，整体来说是少一种。再来看个图，注意颜色区分哦

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage-20200322163335824.png)</fancybox>


Java 语言中

- 将通用线程状态的`可运行状态`和`运行状态`合并为 `Runnable`，
- 将`休眠状态`细分为三种 (`BLOCKED`/`WAITING`/`TIMED_WAITING`); 反过来理解这句话，就是这三种状态在操作系统的眼中都是休眠状态，同样**不会获得CPU使用权**



看上图右侧【Java语言中的线程状态】，进一步简洁的说，除去线程生死，我们只要玩转 `RUNNABLE` 和`休眠状态`的转换就可以了，编写并发程序也多数是这两种状态的转换。所以我们需要了解，有哪些时机，会触发这些状态转换



远看看轮廓， 近看看细节。我们将上面Java语言中的图进行细化，将触发的节点放到图中 (这看似复杂的图，其实三句话就能分解的，所以别慌)，且看：



<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgstateoverview.png)</fancybox>







#### RUNNABLE与BLOCKED状态转换

**当且仅有（just only）一种情况会从 RUNNABLE 状态进入到 BLOCKED 状态**，就是线程在等待 synchronized 内置隐式锁；如果等待的线程获取到了 synchronized 内置隐式锁，也就会从 BLOCKED 状态变为 RUNNABLE 状态了



> **注意：**
>
> 上面提到，以操作系统通用状态来看，线程调用阻塞式 API，会变为休眠状态（释放CPU使用权），但在JVM层面，Java线程状态不会发生变化，也就是说Java线程的状态依旧会保持在 RUNNABLE 状态。JVM并不关心操作系统调度的状态。在JVM看来，等待CPU使用权（操作系统里是处在可执行状态）与等待I/O（操作系统是处在休眠状态），都是等待某个资源，所以都归入了RUNNABLE 状态 
>
> ​                                                         —— 摘自《Java并发编程实战》



#### RUNNABLE与WAITING状态转换

调用**不带时间参数的等待API**，就会从RUNNABLE状态进入到WAITING状态；当被唤醒就会从WAITING进入RUNNABLE状态



#### RUNNABLE与 TIMED-WAITING 状态转换

调用**带时间参数的等待API**，自然就从 RUNNABLE 状态进入 TIMED-WAITING 状态；当被唤醒或超时时间到就会从TIMED_WAITING进入RUNNABLE状态



看图中的转换 API 挺多的，其实不用担心，后续分析源码章节，自然就会记住的，现在有个印象以及知道状态转换的节点就好了



---



相信到这里，你看Java线程生命周期的眼神就没那么迷惑了，重点就是RUNNABLE与休眠状态的切换，接下来我们看一看，如何查看线程中的状态，以及具体的代码触发点



## 如何查看线程处在什么状态

### 程序中调用 `getState()` 方法

Thread 类中同样存在  `getState()` 方法用于查看当前线程状态，该方法就是返回上面提到的枚举类 `State`

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200322202334.png)</fancybox>



### 

#### NEW

就是上面提到， 编程语言中特有的，通过继承 Thread 或实现 Runnable 接口定义线程后，这时的状态都是 NEW

```java
Thread thread = new Thread(() -> {});
System.out.println(thread.getState());
```





#### RUNNABLE

调用了 `start()` 方法之后，线程就处在 RUNNABLE 状态了

```java
Thread thread = new Thread(() -> {});
thread.start();
//Thread.sleep(1000);
System.out.println(thread.getState());
```



#### BLOCKED

等待 synchronized 内置锁，就会处在 BLOCKED 状态

```java
public class ThreadStateTest {

	public static void main(String[] args) throws InterruptedException {
		Thread t1 = new Thread(new DemoThreadB());
		Thread t2 = new Thread(new DemoThreadB());

		t1.start();
		t2.start();

		Thread.sleep(1000);

		System.out.println((t2.getState()));
		System.exit(0);
	}
}

class DemoThreadB implements Runnable {
	@Override
	public void run() {
		commonResource();
	}

	public static synchronized void commonResource() {
		while(true) {
			
		}
	}
}
```



#### WAITING

调用线程的 `join()` 等方法，从 RUNNABLE 变为 WAITING 状态

```java
public static void main(String[] args) throws InterruptedException {
		Thread main = Thread.currentThread();

		Thread thread2 = new Thread(() -> {
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
        Thread.currentThread().interrupt();
				e.printStackTrace();
			}
			System.out.println(main.getState());
		});
		thread2.start();
		thread2.join();
	}
```



#### TIMED-WAITING

调用了 `sleep(long)` 等方法，线程从 RUNNABLE 变为 TIMED-WAITING 状态

```java
public static void main(String[] args) throws InterruptedException {
		Thread thread3 = new Thread(() -> {
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
        // 为什么要调用interrupt方法？
				Thread.currentThread().interrupt();
				e.printStackTrace();
			}
		});
		thread3.start();

		Thread.sleep(1000);
		System.out.println(thread3.getState());
	}
```

#### TERMINATED

线程执行完自然就到了 TERMINATED 状态了

```java
Thread thread = new Thread(() -> {});
thread.start();
Thread.sleep(1000);
System.out.println(thread.getState());
```



以上是程序中查看线程，自己写写测试看看状态还好，现实中的程序怎么可能允许你加这么多无用代码，所以，翠花，上酸菜（**jstack**）

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200325114400.png)</fancybox>



### jstack 命令查看

相信你听说过这玩意，jstack 命令就比较强大了，不仅能查看线程当前状态，还能看调用栈，锁等线程栈信息



大家可以随意写一些程序，这里我用了上面 WAITING 状态的代码, 修改睡眠时间 Thread.sleep(100000)，然后在终端按照下图标示依次执行下图命令



<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage-20200324211507627.png)</fancybox>



更多功能还请大家自行查看，后续会单独写文章来教大家如何使用jstack查看线程栈信息



### Arthas 

这个利器，无须多言吧，线上找茬监控没毛病，希望你可以灵活使用这个工具，攻克疑难杂症

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200324213310.png)</fancybox>



查看线程栈详细信息，非常方便：https://alibaba.github.io/arthas/thread.html

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200324213720.png)</fancybox>



相信你已经和Arthas确认了眼神 

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200324214443.png)



关于线程生命周期状态整体就算说完了，编写并发程序时多问一问自己：

> 调用某个API会将你的线程置为甚么状态？

多问自己几次，自然就记住上面的图了



## 灵魂追问

1. 为什么调用 Thread.sleep, catch异常后，调用了Thread.currentThread().interrupt();

2. 进入 BLOCKED只有一种情况，就是等待 synchronized 监视器锁，那调用 JUC 中的 Lock.lock() 方法，如果某个线程等待这个锁，这个线程状态是什么呢？为什么？

```java
public class ThreadStateTest {

public static void main(String[] args) throws InterruptedException {
    TestLock testLock = new TestLock();

    Thread thread2 = new Thread(() -> {
        testLock.myTestLock();
    }, "thread2");

    Thread thread1 = new Thread(() -> {
            testLock.myTestLock();
        }, "thread1");

    thread1.start();
    Thread.sleep(1000);

    thread2.start();
    Thread.sleep(1000);

    System.out.println("****" + (thread2.getState()));

    Thread.sleep(20000);
    }   
}

@Slf4j
class TestLock{
    private final Lock lock = new ReentrantLock();

    public void myTestLock(){
        lock.lock();
        try{
            Thread.sleep(10000);
            log.info("testLock status");
        } catch (InterruptedException e) {
            log.error(e.getMessage());
        } finally {
            lock.unlock();
        }
    }
}
```

   

3. synchronized 和 Lock 有什么区别？

---

我这面也在逐步总结常见的并发面试问题（总结ing......）答案整理好后会通知大家，请持续关注

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200315120104.png)</fancybox>



同时，这里也整理了一点 Java 硬核资料，有需要的就公众号回复【资料】/【666】吧

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200310160835.png)</fancybox>
