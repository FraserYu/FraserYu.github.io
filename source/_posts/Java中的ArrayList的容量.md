---
title: Java中的ArrayList的容量
tags:
  - ArrayList
categories: [Coding, Java]
id: java-arraylist-capacity
date: 2017-03-16 11:33:44
description: ArrayList使用必须要给定初始容量，节省内存开销
keywords: ArrayList,容量
---

1. List接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。
2. ArrayList继承于List接口，除继承过来的方法外，还提供一些方法来操作内部用来存储列表的数组的大小。
3. 每个ArrayList实例都有一个容量。该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。并未指定增长策略的细节，因为这不只是添加元素会带来分摊固定时间开销那样简单。
<!-- more -->
4. ArrayList是经常会被用到的，一般情况下，使用的时候会像这样进行声明：

		List arrayList = new ArrayList();
如果像上面这样使用默认的构造方法，初始容量被设置为10。当ArrayList中的元素超过10个以后，会重新分配内存空间，使数组的大小增长到16。
可以通过调试看到动态增长的数量变化：10->16->25->38->58->88->...

5. 也可以使用下面的方式进行声明：

		List arrayList = new ArrayList(4);
将ArrayList的默认容量设置为4。当ArrayList中的元素超过4个以后，会重新分配内存空间，使数组的大小增长到7。
可以通过调试看到动态增长的数量变化：4->7->11->17->26->...

6. 那么容量变化的规则是什么呢？请看下面的公式：

		((旧容量 * 3) / 2) + 1
注：这点与C#语言是不同的，C#当中的算法很简单，是翻倍。 一旦容量发生变化，就要带来额外的内存开销，和时间上的开销。所以，在已经知道容量大小的情况下，推荐使用下面方式进行声明：List arrayList = new ArrayList(CAPACITY_SIZE); 即指定默认容量大小的方式。

感谢原文作者：[原文](http://blog.csdn.net/arui319/article/details/3557743)
