---
title: Java 12 新特性一览
date: 2020-02-10 14:30:27
tags:
    - Java
categories: [Coding, Java]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
id: jdk12-new-feature-overview
description: 本文介绍了几个比较常见，又方便使用的Java12新特性，包括String API的更改，文件的比较，Switch语句的扩展等
keywords: Java12,Java12 Switch, String indent, String transform, JDK12,Lambda
---


> - 你有一个思想，我有一个思想，我们交换后，一个人就有两个思想
>
> - If you can NOT explain it simply, you do NOT understand it well enough



现陆续将Demo代码和技术文章整理在一起 [Github实践精选](https://github.com/FraserYu/learnings)，**本文同样收录在此**，方便大家阅读查看，觉得不错，还请Star🌟



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgfenge.gif)



日常工作对集合操作真的太频繁了，前端时间就写过一篇关于Java 12 集合的文章 [Java12 Collectors.teeing 的使用详解](https://dayarch.top/p/jdk12-collectors-teeing-api-usage.html) ， 有朋友留言说这个功能比较好用。个人觉得 Java12还有几个特性可以尝试使用，这篇文章就出炉了



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgJava12_new_feature.png)





如果你目前使用的Java版本不是12，也没有关系，早已为你准备好良方，[SDKMAN 统一灵活管理多版本Java](https://dayarch.top/p/multiple-java-management.html) ，可以让你快速各种尝鲜新特性



## String API 变化



### String.indent()

`indent` 「缩进」这个单词大家很熟悉了，这是Java12处理字符串的新功能，先来看方法定义：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200209135336.png)



该方法很简单，只接收一个 `int` 类型的参数表示缩进值，这里的 n 既可以是正数，也可以是负数，只不过是增加空格 `space` 和移除空格的差别，来看个例子：



```java
String result = "foo\nbar\nbar2".indent(4);
System.out.println(result);
```



打印结果是这样的（考验眼力的时候到了，仔细看截图在IDE中设置的缩进小点点😜）：



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200209141352.png)



调用 indent 方法会自动添加一个换行符号 `\n` ，在该方法的实现中也明确给了注释说明，目的是为了行终结符的规范化

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200209153328.png)



**注意：**对于 `Tab` 就是当成一个字符来看到，比如我们把上面的例子稍作修改：

```java
String result = "foo\nbar\n\tbar2".indent(4);
System.out.println(result);
```

来看打印结果，注意和上面的不同：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200209153700.png)



就是这么简单，我们继续向下看



### String.transform()

`transform` 「转换」，我们经常会遇到字符串形势转换的需求，transform方法接收一个 Function 类型的参数， 生成一个全新形式的字符串



```java
List<String> names = List.of( "   Alex", "brian");

List<String> transformedNames = new ArrayList<>();

for (String name : names){
	String transformedName = name.transform(String::strip)
															 .transform(StringUtils::toCamelCase);

	transformedNames.add(transformedName);
}
```



有朋友可能会说，这个和单纯的对字符串 trim 或者其他操作有什么区别啊？因为接受的参数是 Function类型，当Function类型作为入参时，内部的**处理逻辑**将增加更多灵活性



## Files.mismatch(Path, Path)

有时候，我们需要比较两个文件的内容是否相同，这个API就派上用场了，该方法比较两个 path 下的文件，并且返回一个 long 值，这个值表示第一处不匹配的字节位置。如果返回-1，说明两个文件相等，还是来看个例子：



```java
Path file1 = Paths.get("/Users/fraser/Documents/projects/personal/learning-demo-collection/jdk12-demo/src/file1.txt");
		Path file2 = Paths.get("/Users/fraser/Documents/projects/personal/learning-demo-collection/jdk12-demo/src/file2.txt");


		try {
			long mismatch = Files.mismatch(file1, file2);
			System.out.println(mismatch);
		} catch (IOException e) {
			e.printStackTrace();
		}
```



两个文件内容分别是：

```html
//file1.txt
大家好，我是日拱一兵，叫我拱哥就好
//file2.txt
大家好，我是日拱一兵，叫我兵兵就好
```



查看运行结果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200209203347.png)



> 建议大家查看一下 `mismatch` 的实现逻辑，有个小算法在里面的



## Support for Unicode 11 （当个了解就好了）

当下，Emoji 表情符号在社交媒体渠道上扮演着重要角色，所以支持最新的 Unicode 规范比以往任何时候都更重要。Java 12保持了同步并支持Unicode 11。Unicode 11增加了684个字符，共137,374个字符，增加了7个新脚本，共146个脚本。



## Switch Expressions(Preview)

这个更改扩展了switch语句。为什么这么说？

- 语句（我们原来那样使用）
- 表达式（不必为每个case块定义一个break语句，我们可以简单地使用箭头语法）
- 变量赋值（使用新的switch表达式，我们可以直接将switch语句分配给一个变量）





```java
boolean isWeekend = switch (day) 
{
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> false;
 
    case SATURDAY, SUNDAY -> true;
 
    default -> throw new IllegalStateException("Illegal day entry :: " + day);
};
 
System.out.println(isWeekend);
```



>  **注意：** 要使用此预览特性，请记住，我们必须在应用程序启动期间使用-enable-preview标志显式地指示JVM。



## Compact Number Formatting (紧凑的数据格式)

由用户界面或命令行工具呈现的大数字总是很难展现。使用数字的缩写形式则要直观很多。前端为了更有好的数据展现形式，很早就有相应的组件实现了。现在后端小伙伴也可以在java12中应用这个特性了


紧凑的数字表示更易于阅读，并且在不丢失原始含义的情况下，在屏幕上需要更少的空间。

> 例子：`3.6M` 比 `3,600,000` 容易读得多



Java 12 引入了一个叫做 NumberFormat.getCompactNumberInstance(Locale, NumberFormat.Style)的静态方法。用于创建紧凑数字表示形式，来看例子：



```java
		NumberFormat formatter = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);

		String formattedString = formatter.format(25000L);
		System.out.println(formattedString);
```



来看运行结果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200210132355.png)



另外，CompactNumberFormat 是 NumberFormat的子类，我们可以自定义它的实例（其实就是格式化样式等），很简单，这个大家自行查看吧



## 总结

Java近两年升级真是太快了，了解一些新功能总是没错的，大家动手实践试试吧，以后遇到类似的需求至少能避免我们重复造轮子了.....



## 灵魂追问

1. 你们项目中Java的版本是多少？
2. 项目中你会怎样建议某些工具的升级？
