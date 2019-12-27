---
title: 在Spring Boot启动时执行代码的几种方法
date: 2019-12-25 16:13:15
tags:
    - Spring Boot
    - Spring Bean
    - 生命周期
categories: [Coding, Spring-Boot]
id: spring-boot-execute-on-startup
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: 我们经常会碰到在Spring Boot启动阶段执行一些代码片段需求，Spring 为我们提供了至少五种方式，比如：MyCommandLineRunner，ApplicationRunner， ApplicationListener， PostConstruct， afterPropertiesSet，但是他们之间还是有很大区别的，本文将带你透彻理解他们的区别，在实战中选择最适合你的那一个
keywords: Spring Bean生命周期, InitializingBean, @PostConstruct, init-method
---


## 前言
有时候我们需要在应用启动时执行一些代码片段，这些片段可能是仅仅是为了记录 log，也可能是在[启动时检查与安装证书](https://dayarch.top/p/pkix-certificate-import.html) ，诸如上述业务要求我们可能会经常碰到

Spring Boot 提供了至少 5 种方式用于在应用启动时执行代码。我们应该如何选择？本文将会逐步解释与分析这几种不同方式


## CommandLineRunner
`CommandLineRunner` 是一个接口，通过实现它，我们可以在 `Spring 应用成功启动之后` 执行一些代码片段

```java
@Slf4j
@Component
@Order(2)
public class MyCommandLineRunner implements CommandLineRunner {
	
	@Override
	public void run(String... args) throws Exception {
		log.info("MyCommandLineRunner order is 2");
		if (args.length > 0){
			for (int i = 0; i < args.length; i++) {
				log.info("MyCommandLineRunner current parameter is: {}", args[i]);
			}
		}
	}
}
```

当 Spring Boot 在应用上下文中找到 `CommandLineRunner` bean，它将会在应用成功启动之后调用 `run()`  方法，并传递用于启动应用程序的命令行参数

通过如下 maven 命令生成 jar 包:
```bash
mvn clean package
```

通过终端命令启动应用，并传递参数:
```bash
java -jar springboot-application-startup-0.0.1-SNAPSHOT.jar --foo=bar --name=rgyb
```

查看运行结果:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-23_20-32-41.jpg)</fancybox>

到这里我们可以看出几个问题:
1. 命令行传入的参数并没有被解析，而只是显示出我们传入的字符串内容 `--foo=bar`，`--name=rgyb`，我们可以通过 `ApplicationRunner` 解析，我们稍后看
2. 在重写的 `run()` 方法上有 `throws Exception` 标记，Spring Boot 会将 `CommandLineRunner` 作为应用启动的一部分，如果运行 `run()` 方法时抛出 Exception，应用将会终止启动
3. 我们在类上添加了 `@Order(2)` 注解，当有多个 `CommandLineRunner` 时，将会按照 `@Order` 注解中的数字从小到大排序 (数字当然也可以用复数)

> ### ⚠️不要使用 `@Order` 太多
> 看到 order 这个 "黑科技" 我们会觉得它可以非常方便将启动逻辑按照指定顺序执行，但如果你这么写，说明多个代码片段是有相互依赖关系的，为了让我们的代码更好维护，我们应该减少这种依赖使用

### 小结
如果我们只是想简单的获取以空格分隔的命令行参数，那 `MyCommandLineRunner` 就足够使用了

 

## ApplicationRunner
上面提到，通过命令行启动并传递参数，`MyCommandLineRunner` 不能解析参数，如果要解析参数，那我们就要用到 `ApplicationRunner` 参数了

```java
@Component
@Slf4j
@Order(1)
public class MyApplicationRunner implements ApplicationRunner {

	@Override
	public void run(ApplicationArguments args) throws Exception {
		log.info("MyApplicationRunner order is 1");
		log.info("MyApplicationRunner Current parameter is {}:", args.getOptionValues("foo"));
	}
}
```

重新打 jar 包，运行如下命令:
```bash
java -jar springboot-application-startup-0.0.1-SNAPSHOT.jar --foo=bar,rgyb
```

运行结果如下:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-23_21-10-14.jpg)</fancybox>

到这里我们可以看出: 
1. 同 `MyCommandLineRunner` 相似，但 `ApplicationRunner` 可以通过 run 方法的 `ApplicationArguments` 对象解析出命令行参数，并且每个参数可以有多个值在里面，因为 `getOptionValues` 方法返回 List<String> 数组
2. 在重写的 `run()` 方法上有 `throws Exception` 标记，Spring Boot 会将 `CommandLineRunner` 作为应用启动的一部分，如果运行 `run()` 方法时抛出 Exception，应用将会终止启动
3. `ApplicationRunner` 也可以使用 `@Order` 注解进行排序，从启动结果来看，它与 `CommandLineRunner` 共享 order 的顺序，稍后我们通过源码来验证这个结论

### 小结
如果我们想获取复杂的命令行参数时，我们可以使用 `ApplicationRunner` 

 

## ApplicationListener
如果我们不需要获取命令行参数时，我们可以将启动逻辑绑定到 Spring 的 `ApplicationReadyEvent` 上

```java
@Slf4j
@Component
@Order(0)
public class MyApplicationListener implements ApplicationListener<ApplicationReadyEvent> {

	@Override
	public void onApplicationEvent(ApplicationReadyEvent applicationReadyEvent) {
		log.info("MyApplicationListener is started up");
	}
}
```

运行程序查看结果:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-23_21-27-09.jpg)</fancybox>

到这我们可以看出:
1. `ApplicationReadyEvent` **当且仅当** 在应用程序就绪之后才被触发，甚至是说上面的 Listener 要在本文说的所有解决方案都执行了之后才会被触发，最终结论请稍后看
2. 代码中我用 `Order(0)` 来标记，显然 ApplicationListener 也是可以用该注解进行排序的，按数字大小排序，应该是最先执行。但是，这个顺序仅用于同类型的 ApplicationListener 之间的排序，与前面提到的 `ApplicationRunners` 和 `CommandLineRunners` 的排序并不共享

### 小结
如果我们不需要获取命令行参数，我们可以通过 `ApplicationListener<ApplicationReadyEvent>` 创建一些全局的启动逻辑，我们还可以通过它获取 [Spring Boot 支持的 configuration properties 环境变量参数](https://dayarch.top/p/spring-boot-configurationProperties-usage.html)

 ---

如果你看过我之前写的 **Spring Bean 生命周期三部曲:**
- [Spring Bean 生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html)
- [Spring Bean 生命周期之缘尽](https://dayarch.top/p/spring-bean-lifecycle-destroy.html)
- [Spring Aware 到底是什么？](https://dayarch.top/p/spring-aware.html)

那么你会对下面两种方式非常熟悉了

## @PostConstruct
创建启动逻辑的另一种简单解决方案是提供一种在 bean 创建期间由 Spring 调用的初始化方法。我们要做的就只是将 `@PostConstruct` 注解添加到方法中：

```java
@Component
@Slf4j
@DependsOn("myApplicationListener")
public class MyPostConstructBean {

	@PostConstruct
	public void testPostConstruct(){
		log.info("MyPostConstructBean");
	}
}
```

查看运行结果:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-25_15-34-52.jpg)</fancybox>

从上面运行结果可以看出:
1. Spring 创建完 bean之后 (**在启动之前**)，便会立即调用 `@PostConstruct` 注解标记的方法，因此我们无法使用 `@Order ` 注解对其进行自由排序，因为它可能依赖于 `@Autowired` 插入到我们 bean 中的其他 Spring bean。
2. 相反，它将在依赖于它的所有 bean 被初始化之后被调用，如果要添加人为的依赖关系并由此创建一个排序，则可以使用 `@DependsOn` 注解（虽然可以排序，但是不建议使用，理由和 `@Order` 一样）

### 小结
`@PostConstruct` 方法固有地绑定到现有的 Spring bean，因此应仅将其用于此单个 bean 的初始化逻辑；

 

## InitializingBean
与 `@PostConstruct` 解决方案非常相似，我们可以实现 `InitializingBean` 接口，并让 Spring 调用某个初始化方法:

```java
@Component
@Slf4j
public class MyInitializingBean implements InitializingBean {


	@Override
	public void afterPropertiesSet() throws Exception {
		log.info("MyInitializingBean.afterPropertiesSet()");
	}
}
```

查看运行结果:
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-25_15-30-31.jpg)</fancybox>

从上面的运行结果中，我们得到了和 `@PostConstruct` 一样的效果，但二者还是有差别的

> ### ⚠️ `@PostConstruct` 和 `afterPropertiesSet` 区别
> 1. afterPropertiesSet，顾名思义「在属性设置之后」，调用该方法时，该 bean 的所有属性已经被 Spring 填充。如果我们在某些属性上使用 `@Autowired`（常规操作应该使用构造函数注入），那么 Spring 将在调用`afterPropertiesSet` 之前将 bean 注入这些属性。但 `@PostConstruct` 并没有这些属性填充限制
> 2. 所以 `InitializingBean.afterPropertiesSet` 解决方案比使用 `@PostConstruct` 更安全，因为如果我们依赖尚未自动注入的 `@Autowired` 字段，则 `@PostConstruct` 方法可能会遇到 NullPointerExceptions 

### 小结
如果我们使用构造函数注入，则这两种解决方案都是等效的

 

## 源码分析
请打开你的 IDE (重点代码已标记注释):

> `MyCommandLineRunner` 和 `ApplicationRunner` 是在何时被调用的呢？

打开 `SpringApplication.java` 类，里面有 `callRunners` 方法
```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
    List<Object> runners = new ArrayList<>();
    //从上下文获取 ApplicationRunner 类型的 bean
    runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());

    //从上下文获取 CommandLineRunner 类型的 bean
    runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());

    //对二者进行排序，这也就是为什么二者的 order 是可以共享的了
    AnnotationAwareOrderComparator.sort(runners);

    //遍历对其进行调用
    for (Object runner : new LinkedHashSet<>(runners)) {
        if (runner instanceof ApplicationRunner) {
            callRunner((ApplicationRunner) runner, args);
        }
        if (runner instanceof CommandLineRunner) {
            callRunner((CommandLineRunner) runner, args);
        }
    }
}
```

**强烈建议完整看一下 `SpringApplication.java` 的全部代码，Spring Boot 启动过程及原理都可以从这个类中找到一些答案**

 

## 灵魂追问
1. 上面程序运行结果， `afterPropertiesSet` 方法调用先于 `@PostConstruct` 方法，但这和我们在 [Spring Bean 生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html) 中的调用顺序恰恰相反，你知道为什么吗？
2. `MyPostConstructBean` 通过 `@DependsOn("myApplicationListener")` 依赖了 MyApplicationListener，为什么调用结果前者先与后者呢？
3. 为什么不建议 `@Autowired` 形式依赖注入

在写 Spring Bean 生命周期时就有朋友问我与之相关的问题，显然他们在概念上有一些含混，所以，仔细理解上面的问题将会帮助你加深对 Spring Bean 生命周期的理解


## Spring Boot应用启动执行代码概览图
最后画一张图用来总结这几种方式
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgSpringBoot_exec_in_start1.png)</fancybox>