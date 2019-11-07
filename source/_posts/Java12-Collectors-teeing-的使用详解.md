---
title: Java12 Collectors.teeing 的使用详解
date: 2019-11-05 16:43:32
tags:
    - Java
    - JDK源码
categories: [Coding, Java]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
id: jdk12-collectors-teeing-api-usage
description: Java12新特性teeing的使用，merge合并两个集合的内容，提高了编程效率
keywords: teeing,Java12,JDK12,Lambda

---

## 前言
在 Java 12 里面有个非常好用但在官方 JEP 没有公布的功能，因为它只是 Collector 中的一个小改动，它的作用是 merge 两个 collector 的结果，这句话显得很抽象，老规矩，我们先来看个图:
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgtee.jpg)

管道改造经常会用这个小东西，通常我们叫它「三通」，它的主要作用就是将 downstream1 和 downstream2 的流入合并，然后从 merger 流出

有了这个形象的说明我们就进入正题吧

## Collectors.teeing
上面提到的小功能就是 Collectors.teeing API, 先来看一下 JDK 关于该 API 的说明，**看着觉得难受的直接忽略，继续向下看例子就好了:**
```java
/**
 * Returns a {@code Collector} that is a composite of two downstream collectors.
 * Every element passed to the resulting collector is processed by both downstream
 * collectors, then their results are merged using the specified merge function
 * into the final result.
 *
 * <p>The resulting collector functions do the following:
 *
 * <ul>
 * <li>supplier: creates a result container that contains result containers
 * obtained by calling each collector's supplier
 * <li>accumulator: calls each collector's accumulator with its result container
 * and the input element
 * <li>combiner: calls each collector's combiner with two result containers
 * <li>finisher: calls each collector's finisher with its result container,
 * then calls the supplied merger and returns its result.
 * </ul>
 *
 * <p>The resulting collector is {@link Collector.Characteristics#UNORDERED} if both downstream
 * collectors are unordered and {@link Collector.Characteristics#CONCURRENT} if both downstream
 * collectors are concurrent.
 *
 * @param <T>         the type of the input elements
 * @param <R1>        the result type of the first collector
 * @param <R2>        the result type of the second collector
 * @param <R>         the final result type
 * @param downstream1 the first downstream collector
 * @param downstream2 the second downstream collector
 * @param merger      the function which merges two results into the single one
 * @return a {@code Collector} which aggregates the results of two supplied collectors.
 * @since 12
 */
public static <T, R1, R2, R>
Collector<T, ?, R> teeing(Collector<? super T, ?, R1> downstream1,
                          Collector<? super T, ?, R2> downstream2,
                          BiFunction<? super R1, ? super R2, R> merger) {
    return teeing0(downstream1, downstream2, merger);
}
```
API 描述重的一句话非常关键:
>  *Every element passed to the resulting collector is processed by both downstream collectors*
>  结合「三通图」来说明就是，集合中每一个要被传入 merger 的元素都会经过 downstream1 和 downstream2 的加工处理

**其中 merger 类型是 BiFunction，也就是说接收两个参数，并输出一个值，请看它的 apply 方法**
```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);
}
```

至于可以如何处理，我们来看一些例子吧
## 例子
为了更好的说明 teeing 的使用，列举了四个例子，看过这四个例子再回看上面的 API 说明，相信你会柳暗花明了
### 计数和累加
先来看一个经典的问题，给定的数字集合，需要映射整数流中的元素数量和它们的和
```java
class CountSum {
	private final Long count;
	private final Integer sum;
	public CountSum(Long count, Integer sum) {
		this.count = count;
		this.sum = sum;
	}

	@Override
	public String toString() {
		return "CountSum{" +
				"count=" + count +
				", sum=" + sum +
				'}';
	}
}
```
通过 Collectors.teeing 处理
```java
CountSum countsum = Stream.of(2, 11, 1, 5, 7, 8, 12)
        .collect(Collectors.teeing(
                counting(),
                summingInt(e -> e),
                CountSum::new));

System.out.println(countsum.toString());
```
- downstream1 通过 Collectors 的静态方法 counting 进行集合计数
- downstream2 通过 Collectors 的静态方法 summingInt 进行集合元素值的累加
- merger 通过 CountSum 构造器收集结果

运行结果:
>  CountSum{count=7, sum=46}

我们通过 teeing 一次性得到我们想要的结果，继续向下看其他例子:

### 最大值与最小值
通过给定的集合， 一次性计算出集合的最大值与最小值，同样新建一个类 MinMax，并创建构造器用于 merger 收集结果
```java
class MinMax {
	private final Integer min;
	private final Integer max;
	public MinMax(Integer min, Integer max) {
		this.min = min;
		this.max = max;
	}

	@Override
	public String toString() {
		return "MinMax{" +
				"min=" + min +
				", max=" + max +
				'}';
	}
}
```
通过 teeing API 计算结果:
```java
MinMax minmax = Stream.of(2, 11, 1, 5, 7, 8, 12)
        .collect(Collectors.teeing(
                minBy(Comparator.naturalOrder()),
                maxBy(Comparator.naturalOrder()),
                (Optional<Integer> a, Optional<Integer> b) -> new MinMax(a.orElse(Integer.MIN_VALUE), b.orElse(Integer.MAX_VALUE))));

System.out.println(minmax.toString());
```
- downstream1 通过 Collectors 的静态方法 minBy，通过 Comparator 比较器按照自然排序找到最小值
- downstream2 通过 Collectors 的静态方法 maxBy，通过 Comparator 比较器按照自然排序找到最大值
- merger 通过 MinMax 构造器收集结果，只不过为了应对 NPE，将 BiFunction 的两个入参经过 Optional 处理

运行结果:
>  MinMax{min=1, max=12}

为了验证一下 Optional，我们将集合中添加一个 null 元素，并修改一下排序规则来看一下排序结果:
```java
MinMax minmax = Stream.of(null, 2, 11, 1, 5, 7, 8, 12)
				.collect(Collectors.teeing(
						minBy(Comparator.nullsFirst(Comparator.naturalOrder())),
						maxBy(Comparator.nullsLast(Comparator.naturalOrder())),
						(Optional<Integer> a, Optional<Integer> b) -> new MinMax(a.orElse(Integer.MIN_VALUE), b.orElse(Integer.MAX_VALUE))));
```
- downstream1 处理规则是将 null 放在排序的最前面
- downstream2 处理规则是将 null 放在排序的最后面
- merger 处理时，都会执行 optional.orElse 方法，分别输出最小值与最大值

运行结果:
>  MinMax{min=-2147483648, max=2147483647}

### 瓜的总重和单个重量
接下来举一个更贴合实际的操作对象的例子
```java
// 定义瓜的类型和重量
class Melon {
	private final String type;
	private final int weight;
	public Melon(String type, int weight) {
		this.type = type;
		this.weight = weight;
	}

	public String getType() {
		return type;
	}

	public int getWeight() {
		return weight;
	}
}

// 总重和单个重量列表
class WeightsAndTotal {
	private final int totalWeight;
	private final List<Integer> weights;
	public WeightsAndTotal(int totalWeight, List<Integer> weights) {
		this.totalWeight = totalWeight;
		this.weights = weights;
	}

	@Override
	public String toString() {
		return "WeightsAndTotal{" +
				"totalWeight=" + totalWeight +
				", weights=" + weights +
				'}';
	}
}
```

通过 teeing API 计算总重量和单个列表重量
```java
List<Melon> melons = Arrays.asList(new Melon("Crenshaw", 1200),
    new Melon("Gac", 3000), new Melon("Hemi", 2600),
    new Melon("Hemi", 1600), new Melon("Gac", 1200),
    new Melon("Apollo", 2600), new Melon("Horned", 1700),
    new Melon("Gac", 3000), new Melon("Hemi", 2600)
);


WeightsAndTotal weightsAndTotal = melons.stream()
    .collect(Collectors.teeing(
            summingInt(Melon::getWeight),
            mapping(m -> m.getWeight(), toList()),
            WeightsAndTotal::new));

System.out.println(weightsAndTotal.toString());
```
- downstream1 通过 Collectors 的静态方法 summingInt 做重量累加
- downstream2 通过 Collectors 的静态方法 mapping 提取出瓜的重量，并通过流的终结操作 toList() 获取结果
- merger 通过 WeightsAndTotal 构造器获取结果

运行结果:
>  WeightsAndTotal{totalWeight=19500, weights=[1200, 3000, 2600, 1600, 1200, 2600, 1700, 3000, 2600]}

继续一个更贴合实际的例子吧:
### 预约人员列表和预约人数
```java
class Guest {
	private String name;
	private boolean participating;
	private Integer participantsNumber;

	public Guest(String name, boolean participating, Integer participantsNumber) {
		this.name = name;
		this.participating = participating;
		this.participantsNumber = participantsNumber;
	}
	public boolean isParticipating() {
		return participating;
	}

	public Integer getParticipantsNumber() {
		return participantsNumber;
	}

	public String getName() {
		return name;
	}
}

class EventParticipation {
	private List<String> guestNameList;
	private Integer totalNumberOfParticipants;

	public EventParticipation(List<String> guestNameList, Integer totalNumberOfParticipants) {
		this.guestNameList = guestNameList;
		this.totalNumberOfParticipants = totalNumberOfParticipants;
	}

	@Override
	public String toString() {
		return "EventParticipation { " +
				"guests = " + guestNameList +
				", total number of participants = " + totalNumberOfParticipants +
				" }";
	}
}
```

通过 teeing API 处理

```java
var result = Stream.of(
                new Guest("Marco", true, 3),
                new Guest("David", false, 2),
                new Guest("Roger",true, 6))
                .collect(Collectors.teeing(
                        Collectors.filtering(Guest::isParticipating, Collectors.mapping(Guest::getName, Collectors.toList())),
                        Collectors.summingInt(Guest::getParticipantsNumber),
                        EventParticipation::new
                ));
System.out.println(result);
```
- downstream1 通过 filtering 方法过滤出确定参加的人，并 mapping 出他们的姓名，最终放到 toList 集合中
- downstream2 通过 summingInt 方法计数累加
- merger 通过 EventParticipation 构造器收集结果

其中我们定义了 var result 来收集结果，并没有指定类型，这个语法糖也加速了我们编程的效率

运行结果:
>  EventParticipation { guests = [Marco, Roger], total number of participants = 11 }

## 总结
其实 teeing API 就是灵活应用 Collectors 里面定义的静态方法，将集合元素通过 downstream1 和 downstream2 进行处理，最终通过 merger 收集起来，当项目中有同时获取两个收集结果时，是时候应用我们的 teeing API 了

## 灵魂追问
1. Collectors 里面的静态方法你应用的熟练吗？
2. 项目中你们在用 JDK 的版本是多少？
3. Lambda 的使用熟练吗？
4. 你的灯还亮着吗？
