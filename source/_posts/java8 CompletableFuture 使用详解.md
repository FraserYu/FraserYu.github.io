---
title: java8 CompletableFuture 使用详解
date: 2020-07-19 21:17:30
tags:
    - Java-Concurrency
categories: [Coding, Java-Concurrency]
id: java8-completablefuture-tutorial
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgconcurrency.png
description: Future 已经为获取多线程执行结果带来了很好的帮助，但是它依旧存在很多短板，在java1.8的版本中，CompletableFuture 的出现彻底改变了这一情况，结合Lambda的使用，让异步编程，分分钟就可以起飞
keywords: CompletableFuture,Future,Java8
---
![](https://rgyb.sunluomeng.top/puppy-5388151_1280.jpg)

---

<fancybox>![](https://rgyb.sunluomeng.top/20200719211506.png)</fancybox>

## 前言

上一篇文章 [不会用Java Future，我怀疑你泡茶没我快 ](https://dayarch.top/p/java-future-and-callable.html) 全面分析了 Future，通过它我们可以获取线程的执行结果，它虽然解决了 Runnable 的 “三无” 短板，但是它自身还是有短板：

> 不能手动完成计算

假设你使用 Future 运行子线程调用远程 API 来获取某款产品的最新价格，服务器由于洪灾宕机了，此时如果你想手动结束计算，而是想返回上次缓存中的价格，这是 Future 做不到的

> 调用 get() 方法会阻塞程序

Future 不会通知你它的完成，它提供了一个get()方法，程序调用该方法会阻塞直到结果可用为止，没有办法利用回调函数附加到Future，并在Future的结果可用时自动调用它

> 不能链式执行

烧水泡茶中，通过构造函数传参做到多个任务的链式执行，万一有更多的任务，或是任务链的执行顺序有变，对原有程序的影响都是非常大的

> 整合多个 Future 执行结果方式笨重

假设有多个 Future 并行执行，需要在这些任务全部执行完成之后做后续操作，Future 本身是做不到的，需要借助工具类 `Executors` 的方法

```java
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
```

> 没有异常处理

Future 同样没有提供很好的异常处理方案



<fancybox>![](https://rgyb.sunluomeng.top/20200719174354.png)</fancybox>



上一篇文章看 Future 觉得是发现了新天地，这么一说有感觉回到了解放前

<fancybox>![](https://rgyb.sunluomeng.top/20200719174523.png)</fancybox>



对于 Java 后端的同学，在 Java1.8 之前想实现异步编程，还想避开上述这些烦恼，[ReactiveX](http://reactivex.io/intro.html) 应该是一个常见解决方案（做Android 的应该会有了解）。如果熟悉前端同学， ES6 Promise（男朋友的承诺）也解决了异步编程的烦恼



天下语言都在彼此借鉴相应优点，Java 作为老牌劲旅自然也要解决上述问题。又是那个男人，并发大师 Doug Lea 忧天下程序员之忧，解天下程序员之困扰，在 Java1.8 版本（Lambda 横空出世）中，新增了一个并发工具类 **CompletableFuture**，它的出现，让人在泡茶过程中，品尝到了不一样的味道......



## 几个重要 Lambda 函数

**CompletableFuture** 在 Java1.8 的版本中出现，自然也得搭上 Lambda 的顺风车，为了更好的理解 **CompletableFuture**，这里我需要先介绍一下几个 Lambda 函数，我们只需要关注它们的以下几点就可以：

- 参数接受形式
- 返回值形式
- 函数名称



### Runnable

Runnable 我们已经说过无数次了，无参数，无返回值

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```



### Function

Function<T, R> 接受一个参数，并且有返回值

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```



### Consumer

Consumer<T> 接受一个参数，没有返回值

```java
@FunctionalInterface
public interface Consumer<T> {   
    void accept(T t);
}
```



### Supplier

Supplier<T> 没有参数，有一个返回值

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

### BiConsumer

BiConsumer<T, U> 接受两个参数（Bi， 英文单词词根，代表两个的意思），没有返回值

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
```



好了，我们做个小汇总

<fancybox>![](https://rgyb.sunluomeng.top/20200719175939.png)</fancybox>



有些同学可能有疑问，为什么要关注这几个函数式接口，因为 **CompletableFuture** 的函数命名以及其作用都是和这几个函数式接口高度相关的，一会你就会发现了

<fancybox>![](https://rgyb.sunluomeng.top/20200719180154.png)</fancybox>



前戏做足，终于可以进入正题了 **CompletableFuture**

## CompletableFuture

### 类结构

老规矩，先从类结构看起：

<fancybox>![](https://rgyb.sunluomeng.top/20200719153822.png)</fancybox>



#### 实现了 Future 接口

实现了 Future 接口，那就具有 Future 接口的相关特性，请脑补 Future 那少的可怜的 5 个方法，这里不再赘述，具体请查看 [不会用Java Future，我怀疑你泡茶没我快 ](https://dayarch.top/p/java-future-and-callable.html) 



#### 实现了 CompletionStage 接口

CompletionStage 这个接口还是挺陌生的，中文直译过来是【竣工阶段】，如果将烧水泡茶比喻成一项大工程，他们的竣工阶段体现是不一样的

<fancybox>![](https://rgyb.sunluomeng.top/20200705191033.png)</fancybox>



1. 单看线程1 或单看线程 2 就是一种串行关系，做完一步之后做下一步

2. 一起看线程1 和 线程 2，它们彼此就是并行关系，两个线程做的事彼此独立互不干扰

3. 泡茶就是线程1 和 线程 2 的汇总/组合，也就是线程 1 和 线程 2 都完成之后才能到这个阶段（当然也存在线程1 或 线程 2 任意一个线程竣工就可以开启下一阶段的场景）



所以，CompletionStage 接口的作用就做了这点事，所有函数都用于描述任务的时序关系，总结起来就是这个样子：

<fancybox>![](https://rgyb.sunluomeng.top/20200719181233.png)</fancybox>



**CompletableFuture** 既然实现了两个接口，自然也就会实现相应的方法充分利用其接口特性，我们走进它的方法来看一看

<fancybox>![](https://rgyb.sunluomeng.top/20200719182536.png)</fancybox>



**CompletableFuture** 大约有50种不同处理串行，并行，组合以及处理错误的方法。小弟屏幕不争气，方法之多，一个屏幕装不下，看到这么多方法，是不是瞬间要直接 `收藏——>吃灰` 2连走人？别担心，我们按照相应的命名和作用进行分类，分分钟搞定50多种方法

<fancybox>![](https://rgyb.sunluomeng.top/20200719183446.png)</fancybox>



### 串行关系

`then` 直译【然后】，也就是表示下一步，所以通常是一种串行关系体现, then 后面的单词（比如 run /apply/accept）就是上面说的函数式接口中的抽象方法名称了，它的作用和那几个函数式接口的作用是一样一样滴

```java
CompletableFuture<Void> thenRun(Runnable action)
CompletableFuture<Void> thenRunAsync(Runnable action)
CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)
  
<U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
<U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
<U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
  
CompletableFuture<Void> thenAccept(Consumer<? super T> action) 
CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)
  
<U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)  
<U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn)
<U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor)
```



### 聚合 And 关系

`combine... with...` 和 `both...and...` 都是要求两者都满足，也就是 and 的关系了

```java
<U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
<U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
<U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)

<U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action)
<U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action)
<U> CompletableFuture<Void> thenAcceptBothAsync( CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action, Executor executor)
  
CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action)
CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action)
CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor)
```



### 聚合 Or 关系

`Either...or...` 表示两者中的一个，自然也就是 Or 的体现了

```java
<U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn)
<U> CompletableFuture<U> applyToEitherAsync(、CompletionStage<? extends T> other, Function<? super T, U> fn)
<U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn, Executor executor)

CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)

CompletableFuture<Void> runAfterEither(CompletionStage<?> other, Runnable action)
CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action)
CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action, Executor executor)
```



### 异常处理

```java
CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
CompletableFuture<T> exceptionallyAsync(Function<Throwable, ? extends T> fn)
CompletableFuture<T> exceptionallyAsync(Function<Throwable, ? extends T> fn, Executor executor)
        
CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action)
CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action)
CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action, Executor executor)
        
       
<U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn)
<U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn)
<U> CompletableFuture<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor)
```



这个异常处理看着还挺吓人的，拿传统的 try/catch/finally 做个对比也就瞬间秒懂了

<fancybox>![](https://rgyb.sunluomeng.top/20200719185042.png)</fancybox>



whenComplete 和 handle 的区别如果你看接受的参数函数式接口名称你也就能看出差别了，前者使用Comsumer, 自然也就不会有返回值；后者使用 Function，自然也就会有返回值



这里并没有全部列举，不过相信很多同学已经发现了规律：

> CompletableFuture 提供的所有回调方法都有两个异步（Async）变体，都像这样

```java
// thenApply() 的变体
<U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
<U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
<U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```

> 另外,方法的名称也都与前戏中说的函数式接口完全匹配，按照这中规律分类之后，这 50 多个方法看起来是不是很轻松了呢？

<fancybox>![](https://rgyb.sunluomeng.top/20200719185324.png)</fancybox>





基本方法已经罗列的差不多了，接下来我们通过一些例子来实际演示一下：



### 案例演示

#### 创建一个 CompletableFuture 对象

创建一个 CompletableFuture 对象并没有什么稀奇的，依旧是通过构造函数构建

```java
CompletableFuture<String> completableFuture = new CompletableFuture<String>();
```

这是最简单的 CompletableFuture 对象创建方式，由于它实现了 Future 接口，所以自然就可以通过 get() 方法获取结果

```java
String result = completableFuture.get();
```

文章开头已经说过，get()方法在任务结束之前将一直处在阻塞状态，由于上面创建的 Future 没有返回，所以在这里调用 get() 将会永久性的堵塞

<fancybox>![](https://rgyb.sunluomeng.top/20200719190106.png)</fancybox>

这时就需要我们调用 complete() 方法手动的结束一个 Future

```java
completableFuture.complete("Future's Result Here Manually");
```

这时，所有等待这个 Future 的 client 都会返回手动结束的指定结果



#### runAsync

使用 `runAsync` 进行异步计算

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    System.out.println("运行在一个单独的线程当中");
});

future.get();
```

由于使用的是 Runnable 函数式表达式，自然也不会获取到结果

<fancybox>![](https://rgyb.sunluomeng.top/20200719190859.png)</fancybox>



#### supplyAsync

使用 `runAsync` 是没有返回结果的，我们想获取异步计算的返回结果需要使用 `supplyAsync()` 方法

```java
		CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
			try {
				TimeUnit.SECONDS.sleep(3);
			} catch (InterruptedException e) {
				throw new IllegalStateException(e);
			}
			log.info("运行在一个单独的线程当中");
			return "我有返回值";
		});

		log.info(future.get());
```

由于使用的是 Supplier 函数式表达式，自然可以获得返回结果

<fancybox>![](https://rgyb.sunluomeng.top/20200719191341.png)</fancybox>



我们已经多次说过，get() 方法在Future 计算完成之前会一直处在 blocking 状态下，对于真正的异步处理，我们希望的是可以通过传入回调函数，在Future 结束时自动调用该回调函数，这样，我们就不用等待结果



```java
CompletableFuture<String> comboText = CompletableFuture.supplyAsync(() -> {
  		//可以注释掉做快速返回 start
			try {
				TimeUnit.SECONDS.sleep(3);
			} catch (InterruptedException e) {
				throw new IllegalStateException(e);
			}
			log.info("👍");
  		//可以注释掉做快速返回 end
			return "赞";
		})
				.thenApply(first -> {
					log.info("在看");
					return first + ", 在看";
				})
				.thenApply(second -> second + ", 转发");

		log.info("三连有没有？");
		log.info(comboText.get());
```

<fancybox>![](https://rgyb.sunluomeng.top/20200719194326.png)</fancybox>



对 thenApply 的调用并没有阻塞程序打印log，也就是前面说的通过回调通知机制， 这里你看到 thenApply  使用的是supplyAsync所用的线程，如果将supplyAsync 做快速返回，我们再来看一下运行结果：

<fancybox>![](https://rgyb.sunluomeng.top/20200719194537.png)</fancybox>

thenApply 此时使用的是主线程，所以：

> **串行的后续操作并不一定会和前序操作使用同一个线程**



#### thenAccept

如果你不想从回调函数中返回任何结果，那可以使用 thenAccept 

```java
		final CompletableFuture<Void> voidCompletableFuture = CompletableFuture.supplyAsync(
				// 模拟远端API调用，这里只返回了一个构造的对象
				() -> Product.builder().id(12345L).name("颈椎/腰椎治疗仪").build())
				.thenAccept(product -> {
					log.info("获取到远程API产品名称 " + product.getName());
				});
		voidCompletableFuture.get();
```



#### thenRun

`thenAccept` 可以从回调函数中获取前序执行的结果，但thenRun 却不可以，因为它的回调函数式表达式定义中没有任何参数

```java
CompletableFuture.supplyAsync(() -> {
    //前序操作
}).thenRun(() -> {
    //串行的后需操作，无参数也无返回值
});
```



我们前面同样说过了，每个提供回调方法的函数都有两个异步（Async）变体，异步就是另外起一个线程

```java
		CompletableFuture<String> stringCompletableFuture = CompletableFuture.supplyAsync(() -> {
			log.info("前序操作");
			return "前需操作结果";
		}).thenApplyAsync(result -> {
			log.info("后续操作");
			return "后续操作结果";
		});
```

<fancybox>![](https://rgyb.sunluomeng.top/20200719195510.png)</fancybox>



到这里，相信你串行的操作你已经非常熟练了

#### thenCompose

日常的任务中，通常定义的方法都会返回 CompletableFuture 类型，这样会给后续操作留有更多的余地，假如有这样的业务（X呗是不是都有这样的业务呢？）：

```java
//获取用户信息详情
	CompletableFuture<User> getUsersDetail(String userId) {
		return CompletableFuture.supplyAsync(() -> User.builder().id(12345L).name("日拱一兵").build());
	}

	//获取用户信用评级
	CompletableFuture<Double> getCreditRating(User user) {
		return CompletableFuture.supplyAsync(() -> CreditRating.builder().rating(7.5).build().getRating());
	}
```

这时，如果我们还是使用 thenApply() 方法来描述串行关系，返回的结果就会发生 CompletableFuture 的嵌套

```java
		CompletableFuture<CompletableFuture<Double>> result = completableFutureCompose.getUsersDetail(12345L)
				.thenApply(user -> completableFutureCompose.getCreditRating(user));
```

显然这不是我们想要的，如果想“拍平” 返回结果，thenCompose 方法就派上用场了

```java
CompletableFuture<Double> result = completableFutureCompose.getUsersDetail(12345L)
				.thenCompose(user -> completableFutureCompose.getCreditRating(user));
```

这个和 Lambda 的map 和 flatMap 的道理是一样一样滴



#### thenCombine

如果要聚合两个独立 Future 的结果，那么 thenCombine 就会派上用场了

```java
		CompletableFuture<Double> weightFuture = CompletableFuture.supplyAsync(() -> 65.0);
		CompletableFuture<Double> heightFuture = CompletableFuture.supplyAsync(() -> 183.8);
		
		CompletableFuture<Double> combinedFuture = weightFuture
				.thenCombine(heightFuture, (weight, height) -> {
					Double heightInMeter = height/100;
					return weight/(heightInMeter*heightInMeter);
				});

		log.info("身体BMI指标 - " + combinedFuture.get());
```

<fancybox>![](https://rgyb.sunluomeng.top/20200719201750.png)</fancybox>



当然这里多数时处理两个 Future 的关系，如果超过两个Future，如何处理他们的一些聚合关系呢？



#### allOf ｜ anyOf

相信你看到方法的签名，你已经明白他的用处了，这里就不再介绍了

```java
static CompletableFuture<Void>	 allOf(CompletableFuture<?>... cfs)
static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```



接下来就是异常的处理了

#### exceptionally

```java
		Integer age = -1;

		CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
			if( age < 0 ) {
				throw new IllegalArgumentException("何方神圣？");
			}
			if(age > 18) {
				return "大家都是成年人";
			} else {
				return "未成年禁止入内";
			}
		}).thenApply((str) -> {
			log.info("游戏开始");
			return str;
		}).exceptionally(ex -> {
			log.info("必有蹊跷，来者" + ex.getMessage());
			return "Unknown!";
		});

		log.info(maturityFuture.get());
```

<fancybox>![](https://rgyb.sunluomeng.top/20200719204002.png)</fancybox>

exceptionally 就相当于 catch，出现异常，将会跳过 thenApply 的后续操作，直接捕获异常，进行一场处理



#### handle

用多线程，良好的习惯是使用 try/finally 范式，handle 就可以起到 finally 的作用，对上述程序做一个小小的更改， handle 接受两个参数，一个是正常返回值，一个是异常

> **注意：handle的写法也算是范式的一种**

```java
		Integer age = -1;

		CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
			if( age < 0 ) {
				throw new IllegalArgumentException("何方神圣？");
			}
			if(age > 18) {
				return "大家都是成年人";
			} else {
				return "未成年禁止入内";
			}
		}).thenApply((str) -> {
			log.info("游戏开始");
			return str;
		}).handle((res, ex) -> {
			if(ex != null) {
				log.info("必有蹊跷，来者" + ex.getMessage());
				return "Unknown!";
			}
			return res;
		});

		log.info(maturityFuture.get());
```



到这里，关于 `CompletableFuture` 的基本使用你已经了解的差不多了，不知道你是否注意，我们前面说的带有 Sync 的方法是单独起一个线程来执行，但是我们并没有创建线程，这是怎么实现的呢？

<fancybox>![](https://rgyb.sunluomeng.top/20200719205144.png)</fancybox>



细心的朋友如果仔细看每个变种函数的第三个方法也许会发现里面都有一个 Executor 类型的参数，用于指定线程池，因为实际业务中我们是严谨手动创建线程的，这在 [我会手动创建线程，为什么要使用线程池?](https://dayarch.top/p/why-we-need-to-use-threadpool.html)文章中明确说明过；如果没有指定线程池，那自然就会有一个默认的线程池，也就是 ForkJoinPool

```java
private static final Executor ASYNC_POOL = USE_COMMON_POOL ?
    ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
```

ForkJoinPool 的线程数默认是 CPU 的核心数。但是，在前序文章中明确说明过：

> **不要所有业务共用一个线程池**，因为，一旦有任务执行一些很慢的 I/O 操作，就会导致线程池中所有线程都阻塞在 I/O 操作上，从而造成线程饥饿，进而影响整个系统的性能



## 总结

`CompletableFuture` 的方法并没有全部介绍完全，也没必要全部介绍，相信大家按照这个思路来理解 `CompletableFuture` 也不会有什么大问题了，剩下的就交给`实践/时间`以及自己的体会了



## 后记

你以为 JDK1.8 *CompletableFuture* 已经很完美了是不是，但追去完美的道路上永无止境，Java 9 对*CompletableFuture* 又做了部分升级和改造

## 

1. 添加了新的工厂方法

2. 支持延迟和超时处理

   ```java
   orTimeout()
   completeOnTimeout()
   ```

3. 改进了对子类的支持

详情可以查看： [Java 9 CompletableFuture API Improvements](https://www.baeldung.com/java-9-completablefuture). 怎样快速的切换不同 Java 版本来尝鲜？[SDKMAN 统一灵活管理多版本Java](https://dayarch.top/p/multiple-java-management.html) 这篇文章的方法送给你



最后咱们再泡一壶茶，感受一下新变化吧



## 灵魂追问

1. 听说 ForkJoinPool 线程池效率更高，为什么呢？
2. 如果批量处理异步程序，有什么可用的方案吗？



## 参考

1. Java 并发编程实战
2. Java 并发编程的艺术
3. Java 并发编程之美
4. https://www.baeldung.com/java-completablefuture
5. https://www.callicoder.com/java-8-completablefuture-tutorial/

