---
title: Lombok 使用教程详解
date: 2019-11-25 13:34:12
tags:
    - Java
    - Lombok
categories: [Coding, Java]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
id: lombok-usage
description: Lombok 是一种 Java™ 实用工具，可用来帮助开发人员消除 Java 的冗长，尤其是对于简单的 Java 对象（POJO）
keywords: Lombok, @Data, Lombok安装, @RequiredArgsConstructor, @AllArgsConstructor
---


## 前言
在 Java 应用程序中存在许多重复相似的、生成之后几乎不对其做更改的代码，但是我们还不得不花费很多精力编写它们来满足 Java 的编译需求

比如，在 Java 应用程序开发中，我们几乎要为所有 Bean 的成员变量添加 get() ,set() 等方法，这些相对固定但又不得不编写的代码浪费程序员很多精力，同时让类内容看着更杂乱，我们希望将有限的精力关注在更重要的地方。

Lombok 已经诞生很久了，甚至在 Spring Boot Initalizr 中都已加入了 Lombok 选项，
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-25_11-53-49.jpg)

这里我们将 Lombok 做一下详细说明:

## Lombok

> **官网的介绍**：Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java. Never write another getter or equals method again. Early access to future java features such as val, and much more.

**直白的说：** Lombok 是一种 Java™ 实用工具，可用来帮助开发人员消除 Java 的冗长，尤其是对于简单的 Java 对象（POJO）。它通过注解实现这一目的，且看:

### Bean 的对比
传统的 POJO 类是这样的
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage001.png)

通过Lombok改造后的 POJO 类是这样的
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage003.png)

一眼可以观察出来我们在编写 Employee 这个类的时候通过 `@Data` 注解就已经实现了所有成员变量的 `get()` 与 `set()` 方法等，同时 Employee 类看起来更加清晰简洁。Lombok 的神奇之处不止这些，丰富的注解满足了我们开发的多数需求。

### Lombok的安装
查看下图，@Data的实现，我们发现这个注解是应用在编译阶段的
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage005.png)

这和我们大多数使用的注解，如 Spring 的注解（在运行时，通过反射来实现业务逻辑）是有很大差别的，如Spring 的@RestController 注解
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage007.png) 

一个更直接的体现就是，普通的包在引用之后一般的 IDE 都能够自动识别语法，但是 Lombok 的这些注解，一般的 IDE 都无法自动识别，因此如果要使用 Lombok 的话还需要配合安装相应的插件来支持 IDE 的编译，防止IDE 的自动检查报错，下面以 IntelliJ IDEA 举例安装插件。

在Repositories中搜索Lombok，安装后重启IDE即可
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage009.png)

在Maven或Gradle工程中添加依赖
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-25_11-32-32.jpg)
 
至此我们就可以应用 Lombok 提供的注解干些事情了。

## Lombok注解详解
Lombok官网提供了许多注解，但是 “劲酒虽好，可不要贪杯哦”，接下来逐一讲解官网推荐使用的注解(有些注解和原有Java编写方式没太大差别的也没有在此处列举，如@ Synchronized等)
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage013.png)

### @Getter和@Setter
该注解可应用在类或成员变量之上，和我们预想的一样，`@Getter` 和 `@Setter` 就是为成员变量自动生成 get 和 set 方法，默认生成访问权限为 public 方法，当然我们也可以指定访问权限 protected 等，如下图：
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage015.png) 

成员变量name指定生成set方法，并且访问权限为protected；boolean类型的成员变量 female 只生成get方法，并修改方法名称为 isFemale()。当把该注解应用在类上，默认为所有非静态成员变量生成 get 和 set 方法，也可以通过 AccessLevel.NONE 手动禁止生成get或set方法，如下图：
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage017.png)
 

### @ToString
该注解需应用在类上，为我们生成 Object 的 toString 方法，而该注解里面的几个属性能更加丰富我们想要的内容, `exclude` 属性禁止在 toString 方法中使用某字段，而of属性可以指定需要使用的字段，如下图：

 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage019.png)

查看编译后的Employee.class得到我们预期的结果，如下图
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage021.png)

### @EqualsAndHashCode
该注解需应用在类上，使用该注解，lombok会为我们生成 equals(Object other) 和 hashcode() 方法，包括所有非静态属性和非transient的属性，同样该注解也可以通过 exclude 属性排除某些字段，of 属性指定某些字段，也可以通过 callSuper 属性在重写的方法中使用父类的字段，这样我们可以更灵活的定义bean的比对，如下图：
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage023.png)
 
查看编译后的Employee.class文件，如下图：
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage025.png)
 
### @NonNull
该注解需应用在方法或构造器的参数上或属性上，用来判断参数的合法性，默认抛出 NullPointerException 异常

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage027.png) 

查看NonNullExample.class文件，会为我们抛出空指针异常，如下图：
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage029.png)

当然我们可以通过指定异常类型抛出其他异常，lombok.nonNull.exceptionType = [NullPointerException | IllegalArgumentException] , 为实现此功能我们需要在项目的根目录新建lombok.config文件：

 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage031.png)

重新编译NonNullExample类，已经为我们抛出非法参数异常：
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage033.png)

### @NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor
以上三个注解分别为我们生成无参构造器，指定参数构造器和包含所有参数的构造器，默认情况下，`@RequiredArgsConstructor`, `@AllArgsConstructor` 生成的构造器会对所有标记 `@NonNull` 的属性做非空校验。

无参构造器很好理解，我们主要看看后两种，先看 `@RequiredArgsConstructor`
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-25_13-03-17.jpg)

从上图中我们可以看出， `@RequiredArgsConstructor` 注解生成有参数构造器时只会包含有 final 和 `@NonNull` 标识的 field，同时我们可以指定 staticName 通过生成静态方法来构造对象
 
查看Employee.class文件
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-25_13-06-48.jpg)

当我们把 staticName 属性去掉我们来看遍以后的文件:
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-11-25_13-08-48.jpg)

相信你已经注意到细节

`@AllArgsConstructor`  就更简单了，请大家自行查看吧


### @Data
介绍了以上的注解，再来介绍 `@Data` 就非常容易懂了，`@Data` 注解应用在类上，是`@ToString`, `@EqualsAndHashCode`, `@Getter / @Setter` 和 `@RequiredArgsConstructor`合力的体现，如下图：
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage039.png)

### @Builder
函数式编程或者说流式的操作越来越流行，应用在大多数语言中，让程序更具更简介，可读性更高，编写更连贯，`@Builder`就带来了这个功能，生成一系列的builder API，该注解也需要应用在类上，看下面的例子就会更加清晰明了。

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage041.png)
 
编译后的Employee.class文件如下：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage043.png)
 
妈妈再也不用担心我 set 值那么麻烦了，流式操作搞定：
 ![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage045.png) 

### @Log
该注解需要应用到类上，在编写服务层，需要添加一些日志，以便定位问题，我们通常会定义一个静态常量Logger，然后应用到我们想日志的地方，现在一个注解就可以实现：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage047.png)

查看class文件，和我们预想的一样：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage049.png)
 
Log有很多变种，CommonLog，Log4j，Log4j2，Slf4j等，lombok依旧良好的通过变种注解做良好的支持：
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage051.png) 

我实际使用的是 `@Slf4j` 注解

### val
熟悉 Javascript 的同学都知道，var 可以定义任何类型的变量，而在 java 的实现中我们需要指定具体变量的类型，而 val 让我们摆脱指定，编译之后就精准匹配上类型，默认是 final 类型，就像 java8 的函数式表达式，()->System.out.println(“hello lombok”);  就可以解析到Runnable函数式接口。
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage053.png)
查看解析后的class文件：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage055.png)
 

### @Cleanup
当我们对流进行操作，我们通常需要调用 close 方法来关闭或结束某资源，而 @Cleanup 注解可以帮助我们调用 close 方法，并且放到 try/finally 处理块中，如下图：
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage057.png)

编译后的class文件如下，我们发现被try/finally包围处理，并调用了流的close方法
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage059.png)

其实在 JDK1.7 之后就有了 `try-with-resource`，不用我们显式的关闭流，这个请大家自行看吧
 
## 总结

Lombok的基本操作流程是这样的：
1. 定义编译期的注解
2. 利用JSR269 api(Pluggable Annotation Processing API )创建编译期的注解处理器
3. 利用tools.jar的javac api处理AST(抽象语法树)
4. 将功能注册进jar包
 
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage061.jpg)


> Lombok 当然还有很多注解，我推荐使用以上就足够了，这个工具是带来便利的，而不能被其捆绑，“弱水三千只取一瓢饮，代码千万需抓重点看”，Lombok 能让我更加专注有效代码排除意义微小的障眼代码（get，set等），另外Lombok生成的代码还能像使用工具类一样方便（@Builder）。

更多内容请查看官网：https://www.projectlombok.org/

- - - - - 

## 灵魂追问
1. 为什么只有一个整体 `@EqualsAndHashCode` 注解？而不是 `@Equals` 和 `@HashCode`？这涉及到一个规范哦
2. 如果把三种构造器方式同时应用又加上了 `@Builder` 注解，会发生什么？
3. **你的灯还亮着吗？**

- - - - - 