---
title: IntelliJ IDEA 高级调试之Stream Trace
date: 2021-05-11 09:32:06
tags:
  - IntelliJ IDEA
id: idea-stream-trace
categories: [Coding, Java]
description: 自从 Java 8 开始，作为程序员的我们都离不开 Stream 相关功能的使用，书写起来那叫一个流畅（这个 feel～～）。但总是有一些时候，我们对 stream 的操作所要的结果和预期不符，这就需要我们逐步调试，定位问题, IntelliJ IDEA 的 Stream Trace 就可以轻松帮助我们可视化问题所在
keywords: Stream Trace,Intellij IDEA
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200207104108.png
---

## 前言

自从 Java 8 开始，作为程序员的我们都离不开 Stream 相关功能的使用，书写起来那叫一个流畅（这个 feel～～）。但总是有一些时候，我们对 stream 的操作所要的结果和预期不符，这就需要我们逐步调试，定位问题



## 常规调试

先来看下面这段代码：

```java
public static void main(String[] args) {
  Object[] res = Stream.of(1,2,3,4,5,6,7,8).filter( i -> i%2 == 0).filter( i -> i>3).toArray();
  System.out.println(Arrays.toString(res));
}
```

我们可以在 Stream 操作处打上断点，逐步查看结果，就像这样：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210511085423.png)



我们需要各种单步调试，不是很直观，我们迫切的需要个一览视图，让我们快速查看我们的 Stream 结果



## 可视化调试

同样先选择行断点，以 `Debug` 模式进入程序：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210511090152.png)

接下来会弹出 `Stream Trace`，整个 Stream 操作尽显眼前

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210511090325.png)

同样可以点击左下角的 `Flat Mode` 按钮，将整个视图扁平化

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210511090630.png)



在实际业务中，我们通常对集合进行各种 Stream 操作，我们再来个复杂一些的例子：

```java
		List<Optional<Customer>> customers = Arrays.asList(
				Optional.of(new Customer("日拱一兵", 18)),
				Optional.of(new Customer("卑微的小开发", 22)),
				Optional.empty(),
				Optional.of(new Customer("OOT", 21)),
				Optional.empty(),
				Optional.of(new Customer("温柔一刀", 23)),
				Optional.empty()
		);

		long numberOf65PlusCustomers = customers
				.stream()
				.flatMap(c -> c
						.map(Stream::of)
						.orElseGet(Stream::empty))
				.filter(c -> c.getAge() > 18)
				.count();

		System.out.println(numberOf65PlusCustomers);
```

同样按照上面的操作得到可视化 Stream Trace 视图，直观了解整个 Stream 流程，查看对象属性等

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210511092343.png)



## 总结

这个简单的功能，看一遍就会，相信可以在日常的调试中对你有很大帮助，接下来会介绍更多的你不曾留意又很高级调试技巧
