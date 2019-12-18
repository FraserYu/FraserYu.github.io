---
title: Java equals 和 hashCode 面试那点事
date: 2019-12-17 20:41:51
tags:
    - Java
    - 面试
categories: [Coding, Java]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.pngs
id: java-equals-hashcode
description: Java基础面试中经常会被问到 equals 和 hashcode 之间的关系及他们的约束，其实这些内容不是通过死记硬背的，我们可以通过渐进明细的形式分析其背后的道理
keywords: 重写equals方法, 重写hashCode方法, 重写equals和hashCode方法
---

## 前言
上一篇文章 [如何妙用 Spring 数据绑定？](https://dayarch.top/p/spring-data-binding-mechanism.html) ，**灵魂追问** 环节留下了一个有关 equals 和 hashcode 问题 。基础面试经常会碰到与之相关的问题，这不是一个复杂的问题，但很多朋友都苦于说明他们二者的关系和约束，于是写本文做单独说明，本篇文章将循序渐进 ( *通过举例，让记忆与理解更轻松* ) 说明这些让你有些苦恼的问题，Let's go .......

## 面试问题

### 1. Java 里面有了 == 运算符，为什么还需要 equals ？
>  `==` 比较的是对象地址，`equals` 比较的是对象值

先来看一看 `Object` 类中 `equals` 方法:

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

我们看到 `equals` 方法同样是通过 `==` 比较对象地址，并没有帮我们比较值。Java 世界中 `Object` 绝对是"老祖宗" 的存在，`==` 号我们没办法改变或重写。但 `equals` 是方法，这就给了我们重写 `equals` 方法的可能，让我们实现其对值的比较: 

```java
@Override
public boolean equals(Object obj) {
    //重写逻辑
}
```

新买的电脑，每个电脑都有唯一的序列号，通常情况下，两个一模一样的电脑放在面前，你会说由于序列号不一样，这两个电脑不一样吗？

如果我们要说两个电脑一样，通常是比较其「品牌/尺寸/配置 」(值) ，比如这样:

```java
@Override
public boolean equals(Object obj) {
    return 品牌相等 && 尺寸相等 && 配置相等
}
```

当遇到如上场景时，我们就需要重写 `equals` 方法。这就解释了 Java 世界为什么有了 `==` 还有`equals` 这个问题了. 

### 2.  `equals`相等 和 `hashcode` 相等问题
关于二者，你经常会碰到下面的两个问题:
- 两个对象 `equals` 相等，那他们 `hashCode` 相等吗？
- 两个对象 `hashCode` 相等，那他们 `equals` 相等吗？

*为了说明上面两个问题的结论，这里举一个不太恰当的例子，只为方便记忆*，我们将 `equals` 比作一个单词的拼写；`hashCode` 比作一个单词的发音，在相同语境下:

> sea / sea 「大海」，两个单词拼写一样，所以 `equals` 相等，他们读音 `/siː/` 也一样，所以 `hashCode` 就相等，这就回答了第一个问题:
> #### 两个对象 `equals` 相等，那他们 `hashCode` 一定也相等
>  sea / see 「大海/看」，两个单词的读音  `/siː/` 一样，显然单词是不一样的，这就回答了第二个问题:
> #### 两个对象 `hashCode` 相等，那他们 `equals` 不一定相等

查看 `Object` 类的 `hashCode` 方法:
```java
public native int hashCode();
```
继续查看该方法的注释，明确写明关于该方法的约束
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-16_21-06-16.jpg)</fancybox>

其实在这个结果的背后，还有的是关于重写 `equals` 方法的约束

### 3. 重写 `equals` 有哪些约束？
关于重写 `equals` 方法的约束，同样在该方法的注释中写的很清楚了，我在这里再说明一下:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-16_20-49-14.jpg)</fancybox>

**赤橙红绿青蓝紫，七彩以色列；哆来咪发唆拉西, 一曲安哥拉** ，这些规则不是用来背诵的，只是在你需要重写 `equals` 方法时，打开 JDK 查看该方法，按照准则重写就好

### 4. 什么时候需要我们重写 `hashCode`？
为了比较值，我们重写 `equals` 方法，那什么时候又需要重写 `hashCode` 方法呢？
> 通常只要我们重写 `equals` 方法就要重写 `hashCode` 方法

为什么会有这样的约束呢？按照上面讲的原则，两个对象 `equals` 相等，那他们的 `hashCode` 一定也相等。如果我们只重写 `equals` 方法而不重写 `hashCode` 方法，看看会发生什么，举个例子来看:

定义学生类，并通过 IDE 只帮我们生成 `equals` 方法:
```java
public class Student {

	private String name;

	private int age;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;
		Student student = (Student) o;
		return age == student.age &&
				Objects.equals(name, student.name);
	}
}
```

编写测试代码:
```java
Student student1 = new Student();
student1.setName("日拱一兵");
student1.setAge(18);

Student student2 = new Student();
student2.setName("日拱一兵");
student2.setAge(18);

System.out.println("student1.equals(student2)的结果是：" + student1.equals(student2));

Set<Student> students = new HashSet<Student>();
students.add(student1);
students.add(student2);
System.out.println("Student Set 集合长度是：" + students.size());

Map<Student, java.lang.String> map = new HashMap<Student, java.lang.String>();
map.put(student1, "student1");
map.put(student2, "student2");
System.out.println("Student Map 集合长度是：" + map.keySet().size());
```

查看运行结果:
```console
student1.equals(student2)的结果是：true
Student Set 集合长度是：2
Student Map 集合长度是：2
```

很显然，按照集合 Set 和 Map 加入元素的标准来看，student1 和 student2 是两个对象，因为在调用他们的 put (Set add 方法的背后也是 HashMap 的 put)方法时， 会先判断 hash 值是否相等，这个小伙伴们打开 JDK 自行查看吧

所以我们继续重写 Student 类的 `hashCode` 方法:
```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```
重新运行上面的测试，查看结果:
```console
student1.equals(student2)的结果是：true
Student Set 集合长度是：1
Student Map 集合长度是：1
```

得到我们预期的结果，这也就是为什么通常我们重写 `equals` 方法为什么最好也重写 `hashCode` 方法的原因

- 如果你在使用 Lombok，不知道你是否注意到 Lombok 只有一个 `@EqualsAndHashCode` 注解，而没有拆分成  @Equals 和 @HashCode 两个注解，想了解更多 Lombok 的内容，也可以查看我之前写的文章 [Lomok 使用详解](https://dayarch.top/p/lombok-usage.html)

- 另外通过 IDE 快捷键生成重写方法时，你也会看到这两个方法放在一起，而不是像 getter 和 setter 那样分开
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-17_11-36-14.jpg)

以上两点都是隐形的规范约束，希望大家也严格遵守这个规范，以防带来不必要的麻烦，记忆的方式有多样，如果记不住这个文字约束，脑海中记住上面的图你也就懂了

### 5. 重写 `hashCode` 为什么总有 31 这个数字？
细心的朋友可能注意到，我上面重写 `hashCode`的方法很简答， 就是用了 `Objects.hash`  方法，进去查看里面的方法:
```java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```
这里通过 31 来计算对象 hash 值

在 [如何妙用 Spring 数据绑定？](https://dayarch.top/p/spring-data-binding-mechanism.html) 文章末尾提到的在 `HandlerMethodArgumentResolverComposite` 类中有这样一个成员变量:

```java
private final Map<MethodParameter, HandlerMethodArgumentResolver> argumentResolverCache =
			new ConcurrentHashMap<MethodParameter, HandlerMethodArgumentResolver>(256);
```

Map 的 key 是 `MethodParameter` ，根据我们上面的分析，这个类一定也会重写 `equals` 和 `hashCode` 方法，进去查看发现，hashCode 的计算也用到了 31 这个数字
```java
@Override
public boolean equals(Object other) {
    if (this == other) {
        return true;
    }
    if (!(other instanceof MethodParameter)) {
        return false;
    }
    MethodParameter otherParam = (MethodParameter) other;
    return (this.parameterIndex == otherParam.parameterIndex && getMember().equals(otherParam.getMember()));
}

@Override
public int hashCode() {
    return (getMember().hashCode() * 31 + this.parameterIndex);
}
```

为什么计算 hash 值要用到 31 这个数字呢？我在网上看到一篇不错的文章，分享给大家，作为科普，可以简单查看一下:
[String hashCode 方法为什么选择数字31作为乘子](https://www.cnblogs.com/nullllun/p/8350178.html)

## 总结
如果还对`equals` 和 `hashCode` 关系及约束含混，我们只需要按照上述步骤逐步回忆即可，更好的是直接查看 JDK 源码；另外拿出实际的例子来反推验证是非常好的办法。如果你还有相关疑问，也可以留言探讨. 


## 灵魂追问
1. Thread 类就没有重写 `equals` 方法，你还知道哪些情况没必要重写 `equals` 方法吗？
2. 从上面 HandlerMethodArgumentResolverComposite 类中定义的 Map 成员变量，你注意到哪些知识点，比如 final，ConcurrentHashMap，初识容量，为什么要这样写？你能解释出原因吗？


- - - - - 