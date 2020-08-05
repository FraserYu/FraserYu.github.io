---
title: Java AQS共享式获取同步状态及Semaphore的应用分析
date: 2020-06-13 15:05:28
tags:
    - Java-Concurrency
categories: [Coding, Java-Concurrency]
id: java-aqs-acquireshared-and-semaphore
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgconcurrency.png
description: 本文详细讲述了 Java AQS 中共享式获取同步状态的过程，通过该过程了解 Semaphore 的经典应用
keywords: AQS,Semphore,acquireShared,setHeadAndPropagate
---


现陆续将Demo代码和技术文章整理在一起 [Github实践精选](https://github.com/FraserYu/learnings) ，方便大家阅读查看，本文同样收录在此，觉得不错，还请Star🌟



<fancybox>![](https://rgyb.sunluomeng.top/20200613150348.png)</fancybox>

看到本期内容这么少，是不是心动了呢？ 



## 前言

上一篇万字长文 [Java AQS队列同步器以及ReentrantLock的应用](https://dayarch.top/p/java-aqs-and-reentrantlock.html) 为我们读 JUC 源码以及其设计思想做了足够多的铺垫，接下来的内容我将重点说明差异化，如果有些童鞋不是能很好的理解文中的一些内容，强烈建议回看上一篇文章，搞懂基础内容，接下来的阅读真会轻松加愉快



---



AQS 中我们介绍了独占式获取同步状态的多种情形：

- 独占式获取锁
- 可响应中断的独占式获取锁
- 有超时限制的独占式获取锁



AQS 提供的模版方法里面还差共享式获取同步状态没有介绍，所以我们今天来揭开这个看似神秘的面纱

<fancybox>![](https://rgyb.sunluomeng.top/20200613150826.png)</fancybox>



## AQS 中的共享式获取同步状态

独占式是你中没我，我中没你的的一种互斥形式，共享式显然就不是这样了，所以他们的唯一区别就是：

> 同一时刻能否有多个线程同时获取到同步状态

简单来说，就是这样滴：

<fancybox>![](https://rgyb.sunluomeng.top/20200613104401.png)</fancybox>



我们知道同步状态 state 是维护在 AQS 中的，抛开可重入锁的概念，我在上篇文章中也提到了，独占式和共享式控制同步状态 state 的区别仅仅是这样：

<fancybox>![](https://rgyb.sunluomeng.top/20200523160705.png)</fancybox>



所以说想了解 AQS 的 xxxShared 的模版方法，只需要知道它是怎么控制 state 的就好了



### AQS共享式获取同步状态源码分析

为了帮助大家更好的回忆内容，我将上一篇文章的两个关键内容粘贴在此处，帮助大家快速回忆，关于共享式，大家只需要关注【骚紫色】就可以了



#### 自定义同步器需要重写的方法

<fancybox>![](https://rgyb.sunluomeng.top/20200523160830.png)</fancybox>



#### AQS 提供的模版方法

<fancybox>![](https://rgyb.sunluomeng.top/20200523195957.png)</fancybox>



故事就从这里说起吧 (你会发现和独占式惊人的相似)，关键代码都加了注释

```java
    public final void acquireShared(int arg) {
      	// 同样调用自定义同步器需要重写的方法，非阻塞式的尝试获取同步状态，如果结果小于零，则获取同步状态失败
        if (tryAcquireShared(arg) < 0)
          	// 调用 AQS 提供的模版方法，进入等待队列
            doAcquireShared(arg);
    }
```



进入 `doAcquireShared` 方法：



```java
    private void doAcquireShared(int arg) {
      	// 创建共享节点「SHARED」，加到等待队列中
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
          	// 进入“自旋”，这里并不是纯粹意义上的死循环，在独占式已经说明过
            for (;;) {
              	// 同样尝试获取当前节点的前驱节点
                final Node p = node.predecessor();
              	// 如果前驱节点为头节点，尝试再次获取同步状态
                if (p == head) {
                  	// 在此以非阻塞式获取同步状态
                    int r = tryAcquireShared(arg);
                  	// 如果返回结果大于等于零，才能跳出外层循环返回
                    if (r >= 0) {
                      	// 这里是和独占式的区别
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```



上面代码第 18 行我们提到和独占式获取同步状态的区别，贴心的给大家一个更直观的对比：



<fancybox>![](https://rgyb.sunluomeng.top/20200613111949.png)</fancybox>



差别只在这里，所以我们就来看看 `setHeadAndPropagate(node, r)` 到底干了什么，我之前说过 JDK 源码中的方法命名绝大多数还是非常直观的，该方法直译过来就是 【设置头并且传播/繁衍】。独占式只是设置了头，共享式除了设置头还多了一个传播，你的疑问应该已经来了：

> 啥是传播，为什么会有传播这个设置呢？

想了解这个问题，你需要先知道非阻塞共享式获取同步状态返回值的含义：

<fancybox>![](https://rgyb.sunluomeng.top/20200613114247.png)</fancybox>



这里说的传播其实说的是 `propagate > 0` 的情况，道理也很简单，当前线程获取同步状态成功了，还有剩余的同步状态可用于其他线程获取，**那就要通知在等待队列的线程，让他们尝试获取剩余的同步状态**



**如果要让等待队列中的线程获取到通知，需要线程调用 release 方法实现的**。接下来，我们走近 `setHeadAndPropagate` 一探究竟，验证一下

```java
  // 入参，node： 当前节点
	// 入参，propagate：获取同步状态的结果值，即上面方法中的变量 r
	private void setHeadAndPropagate(Node node, int propagate) {
    		// 记录旧的头部节点，用于下面的check
        Node h = head; 
    		// 将当前节点设置为头节点
        setHead(node);
        
    		// 通过 propagate 的值和 waitStatus 的值来判断是否可以调用 doReleaseShared 方法
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
          	// 如果后继节点为空或者后继节点为共享类型，则进行唤醒后继节点
    				// 这里后继节点为空意思是只剩下当前头节点了，另外这里的 s == null 也是判断空指针的标准写法
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

上面方法的大方向作用我们了解了，但是代码中何时调用 `doReleaseShared` 的判断逻辑还是挺让人费解的，为什么会有这么一大堆的判断，我们来逐个分析一下：



这里的空判断有点让人头大，我们先挑出来说明一下：

<fancybox>![](https://rgyb.sunluomeng.top/20200613120818.png)</fancybox>



排除了其他判断条件的干扰，接下来我们就专注分析 propagate 和 waitStatus 两个判断条件就可以了，这里再将 waitStatus 的几种状态展示在这里，帮助大家理解，【骚粉色】是我们一会要用到的：

<fancybox>![](https://rgyb.sunluomeng.top/20200613124631.png)</fancybox>



> propagate > 0

上面已经说过了，如果成立，直接短路后续判断，然后根据 doReleaseShared 的判断条件进行释放



> propagate > 0 不成立， h.waitStatus < 0 成立 **（注意这里的h是旧的头节点）**

什么时候 h.waitStatus < 0 呢？抛开 CONDITION 的使用，只剩下 SIGNAL 和 PROPAGATE，想知道这个答案，需要提前看一下 `doReleaseShared()` 方法了：



```java
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                  	// CAS 将头节点的状态设置为0                
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    // 设置成功后才能跳出循环唤醒头节点的下一个节点
                  	unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         // 将头节点状态CAS设置成 PROPAGATE 状态
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

从  `doReleaseShared()` 方法中可以看出：

- 如果让 h.waitStatus < 0 成立，只能将其设置成 PROPAGATE = -3 的情况，设置成功的前提是 h 头节点 expected 的状态是 0；

- 如果 h.waitStatus = 0，是上述代码第 8 行 CAS 设置成功，然后唤醒等待中的线程



所以猜测，当前线程执行到 h.waitStatus < 0 的判断前，有另外一个线程刚好执行了  `doReleaseShared()` 方法，将 waitStatus 又设置成PROPAGATE = -3



这个理解有点绕，我们还是来画个图理解一下吧：

<fancybox>![](https://rgyb.sunluomeng.top/20200613132907.png)</fancybox>



可能有同学还是不太能理解这么写的道理，我们一直说 propagate <> = 0 的情况，propagate = 0 代表的是**当时/当时/当时** 尝试获取同步状态没成功，但是之后可能又有共享状态被释放了，所以上面的逻辑是以防这种万一，你懂的，严谨的并发就是要防止一切万一，现在结合这个情景再来理解上面的判断你是否豁然开朗了呢？



继续向下看，

> 前序条件不成立，(h = head) == null ||  h.waitStatus < 0 **注意这里的h是新的头节点）**

有了上面铺垫，这个就直接画个图就更好理解啦，其实就是没有那么巧有另外一个线程掺合了

<fancybox>![](https://rgyb.sunluomeng.top/20200613133904.png)</fancybox>



相信到这里你应该理解共享式获取同步状态的全部过程了吧，至于**非阻塞共享式获取同步状态**和**带有超时时间获取同步状态**，结合本文讲的 setHeadAndPropagate 逻辑和独占式获取同步状态的实现过程过程来看，真是一毛一样，这里就不再累述了，赶紧打开你的 IDE 去验证一下吧



我们分析了AQS 的模版方法，还一直没说 `tryAcquireShared(arg)` 这个方法是如何被重写的，想要了解这个，我们就来看一看共享式获取同步状态的经典应用 Semaphore

## Semaphore 的应用及源码分析

### Semaphore 概念

Semaphore 中文多翻译为 【信号量】，我还特意查了一下剑桥辞典的英文解释：

<fancybox>![](https://rgyb.sunluomeng.top/20200613134832.png)</fancybox>



其实就是信号标志（two flags），比如红绿灯，每个交通灯产生两种不同行为

- Flag1-红灯：停车
- Flag2-绿灯：行车



在 Semaphore 里面，什么时候是红灯，什么时候是绿灯，其实就是靠 `tryAcquireShared(arg)` 的结果来表示的

- 获取不到共享状态，即为红灯
- 获取到共享状态，即为绿灯



所以我们走近 Semaphore ，来看看它到底是怎么应用 AQS 的，又是怎样重写 `tryAcquireShared(arg)`  方法的



### Semaphore 源码分析

先看一下类结构

<fancybox>![](https://rgyb.sunluomeng.top/20200613135824.png)</fancybox>



看到这里你是否有点跌眼镜，和 ReentrantLock 相似的可怕吧，如果你有些陌生，再次强烈建议你回看上一篇文章 [Java AQS队列同步器以及ReentrantLock的应用](https://dayarch.top/p/java-aqs-and-reentrantlock.html) ，这里直接提速对比看公平和非公平两种重写的 `tryAcquireShared(arg)` 方法，没有意外，公平与否，就是判断是否有前驱节点



<fancybox>![](https://rgyb.sunluomeng.top/20200613140142.png)</fancybox>



方法内部只是计算 state 的剩余值，那 state 的初始值是多少怎么设置呢？当然也就是构造方法了：

```java
		public Semaphore(int permits) {
      	// 默认仍是非公平的同步器，至于为什么默认是非公平的，在上一篇文章中也特意说明过
        sync = new NonfairSync(permits);
    }
    
    NonfairSync(int permits) {
    		super(permits);
    }
```

super 方法，就会将初始值给到 AQS 中的 state

<fancybox>![](https://rgyb.sunluomeng.top/20200613140716.png)</fancybox>



也许你发现了，当我们把 permits 设置为1 的时候，不就是 ReentrantLock 的互斥锁了嘛，说的一点也没错，我们用 Semaphore 也能实现基本互斥锁的效果

```java

static int count;
//初始化信号量
static final Semaphore s 
    = new Semaphore(1);
//用信号量保证互斥    
static void addOne() {
  s.acquire();
  try {
    count+=1;
  } finally {
    s.release();
  }
}
```



But（英文听力中的重点），Semaphore 肯定不是为这种特例存在的，它是共享式获取同步状态的一种实现。如果使用信号量，我们通常会将 permits 设置成大于1的值，不知道你是否还记得我曾在 [为什么要使用线程池?](https://dayarch.top/p/why-we-need-to-use-threadpool.html) 一文中说到的池化概念，在同一时刻，允许多个线程使用连接池，每个连接被释放之前，不允许其他线程使用。所以说 **Semaphore 可以允许多个线程访问一个临界区**，最终很好的做到一个**限流/限流/限流** 的作用





虽然 Semaphore 能很好的提供限流作用，说实话，Semaphore 的限流作用比较单一，我在实际工作中使用 Semaphore 并不是很多，如果真的要用高性能限流器，Guava RateLimiter 是一个非常不错的选择，我们后面会做分析，有兴趣的可以提前了解一下



关于 Semaphore 源码，就这么三下五除二的结束了

<fancybox>![](https://rgyb.sunluomeng.top/20200613150959.png)</fancybox>

## 总结

不知你有没有感觉到，我们的节奏明显加快了，好多原来分散的点在被疯狂的串联起来，如果按照这个方式来阅读 JUC 源码，相信你也不会一头扎进去迷失方向，然后沮丧的退出 JUC 吧，然后面试背诵答案，然后忘记，然后再背诵？



跟上节奏，关于共享式获取同步状态，Semaphore 只不过是非常经典的应用，ReadWriteLock 和 CountDownLatch 日常应用还是非常广泛的，我们接下来就陆续聊聊它们吧



## 灵魂追问

1. Semaphore 的 permits 设置成1 “等同于” 简单的互斥锁实现，那它和 ReentrantLock 的区别还是挺大的，都有哪些区别呢？
2. 你在项目中是如何使用 Semaphore 的呢？





## 参考

1. Java 并发实战
2. Java 并发编程的艺术
3. https://blog.csdn.net/anlian523/article/details/106319294


