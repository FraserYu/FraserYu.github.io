---
title: Spring Boot @ConfigurationProperties 注解最强使用详解
date: 2019-12-23 21:43:55
tags:
    - Spring Boot
categories: [Coding, Spring-Boot]
id: spring-boot-configurationProperties-usage
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: 在编写项目代码时，我们要求更灵活的配置，更好的模块化整合。在 Spring Boot 项目中，为满足以上要求，就要用到ConfigurationProperties注解
keywords: ConfigurationProperties注解,Configuration注解,Spring Boot
---


## 前言
在编写项目代码时，我们要求更灵活的配置，更好的模块化整合。在 Spring Boot 项目中，为满足以上要求，我们将大量的参数配置在 application.properties 或 application.yml 文件中，通过 `@ConfigurationProperties` 注解，我们可以方便的获取这些参数值

## 使用 @ConfigurationProperties 配置模块
假设我们正在搭建一个发送邮件的模块。在本地测试，我们不想该模块真的发送邮件，所以我们需要一个参数来「开关」 disable 这个功能。另外，我们希望为这些邮件配置一个默认的主题，这样，当我们查看邮件收件箱，通过邮件主题可以快速判断出这是测试邮件

在 application.properties 文件中创建这些参数:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-24-05%402x.png)

我们可以使用 `@Value` 注解或着使用 Spring `Environment` bean 访问这些属性，是这种注入配置方式有时显得很笨重。我们将使用更安全的方式(`@ConfigurationProperties` )来获取这些属性

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-24-34%402x.png)

`@ConfigurationProperties` 的基本用法非常简单:我们为每个要捕获的外部属性提供一个带有字段的类。请注意以下几点:
- 前缀定义了哪些外部属性将绑定到类的字段上
- 根据 Spring Boot 宽松的绑定规则，类的属性名称必须与外部属性的名称匹配
- 我们可以简单地用一个值初始化一个字段来定义一个默认值
- 类本身可以是包私有的
- 类的字段必须有公共 setter 方法

> ### Spring 宽松绑定规则 (relaxed binding)
> Spring使用一些宽松的绑定属性规则。因此，以下变体都将绑定到 hostName 属性上:

![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-27-10%402x.png)

如果我们将 MailModuleProperties 类型的 bean 注入到另一个 bean 中，这个 bean 现在可以以类型安全的方式访问那些外部配置参数的值。

但是，我们仍然需要让 Spring 知道我们的 @ConfigurationProperties 类存在，以便将其加载到应用程序上下文中( [面试还不知道 BeanFactory 和 ApplicationContext 的区别？](https://dayarch.top/p/difference-between-beanfactory-and-applicationcontext.html)

## 激活 @ConfigurationProperties
对于 Spring Boot，创建一个 MailModuleProperties 类型的 bean，我们可以通过下面几种方式将其添加到应用上下文中

首先，我们可以通过添加 @Component 注解让 Component Scan 扫描到
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-27-33%402x.png)

很显然，只有当类所在的包被 Spring `@ComponentScan` 注解扫描到才会生效，默认情况下，该注解会扫描在主应用类下的所有包结构

我们也可以通过 Spring 的 Java Configuration 特性实现同样的效果:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-29-20%402x.png)

只要 MailModuleConfiguration 类被 Spring Boot 应用扫描到，我们就可以在应用上下文中访问 MailModuleProperties bean

我们还可以使用 `@EnableConfigurationProperties` 注解让我们的类被 Spring Boot 所知道，在该注解中其实是用了`@Import(EnableConfigurationPropertiesImportSelector.class)` 实现，大家可以看一下
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-29-42%402x.png)


> ### 激活一个 @ConfigurationProperties 类的最佳方式是什么？
> 所有上述方法都同样有效。然而，我建议模块化你的应用程序，并让每个模块提供自己的`@ConfigurationProperties` 类，只提供它需要的属性，就像我们在上面的代码中对邮件模块所做的那样。这使得在不影响其他模块的情况下重构一个模块中的属性变得容易。
>  
> 因此，我不建议在应用程序类本身上使用 `@EnableConfigurationProperties`，如许多其他教程中所示，是在特定于模块的 @Configuration 类上使用`@EnableConfigurationProperties`，该类也可以利用包私有的可见性对应用程序的其余部分隐藏属性。

## 无法转换的属性
如果我们在 application.properties 属性上定义的属性不能被正确的解析会发生什么？假如我们为原本应该为布尔值的属性提供的值为 'foo':
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-30-09%402x.png)

默认情况下，Spring Boot 将会启动失败，并抛出异常:
```shell
Failed to bind properties under 'myapp.mail.enabled' to java.lang.Boolean:

    Property: myapp.mail.enabled
    Value: foo
    Origin: class path resource [application.properties]:1:20
    Reason: failed to convert java.lang.String to java.lang.Boolean
```
当我们为属性配置错误的值时，而又不希望 Spring Boot 应用启动失败，我们可以设置 `ignoreInvalidFields` 属性为 true (默认为 false)
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-49-56%402x.png)

这样，Spring Boot 将会设置 enabled 字段为我们在 Java 代码里设定好的默认值。如果我们没有设置默认值，enabled 将为 null，因为这里定义的是 boolean 的包装类 Boolean

## 未知的属性
和上面的情况有些相反，如果我们在 application.properties 文件提供了 MailModuleProperties 类不知道的属性会发生什么？
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-52-20%402x.png)

默认情况下，Spring Boot 会忽略那些不能绑定到 `@ConfigurationProperties` 类字段的属性

然而，当配置文件中有一个属性实际上没有绑定到 `@ConfigurationProperties` 类时，我们可能希望启动失败。也许我们以前使用过这个配置属性，但是它已经被删除了，这种情况我们希望被触发告知手动从 application.properties 删除这个属性

为了实现上述情况，我们仅需要将 `ignoreUnknownFields` 属性设置为 false (默认是 true)
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-53-28%402x.png)

现在，应用启动时，控制台会反馈我们异常信息
```shell
Binding to target [Bindable@cf65451 type = com.example.configurationproperties.properties.MailModuleProperties, value = 'provided', annotations = array<Annotation>[@org.springframework.boot.context.properties.ConfigurationProperties(value=myapp.mail, prefix=myapp.mail, ignoreInvalidFields=false, ignoreUnknownFields=false)]] failed:

    Property: myapp.mail.unknown-property
    Value: foo
    Origin: class path resource [application.properties]:3:29
    Reason: The elements [myapp.mail.unknown-property] were left unbound.
```
>  弃用警告⚠️(Deprecation Warning)
>  `ignoreUnknownFields` 在未来 Spring Boot 的版本中会被标记为 deprecated，因为我们可能有两个带有 `@ConfigurationProperties` 的类，同时绑定到了同一个命名空间 (namespace) 上，其中一个类可能知道某个属性，另一个类却不知道某个属性，这样就会导致启动失败

## 启动时校验 @ConfigurationProperties
如果我们希望配置参数在传入到应用中时有效的，我们可以通过在字段上添加 `bean validation` 注解，同时在类上添加 `@Validated` 注解
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-55-40%402x.png)

如果我们忘记在 application.properties 文件设置 enabled 属性，并且设置 defaultSubject 为空
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-56-53%402x.png)

应用启动时，我们将会得到 `BindValidationException` 
```shell
Binding to target org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'myapp.mail' to com.example.configurationproperties.properties.MailModuleProperties failed:

    Property: myapp.mail.enabled
    Value: null
    Reason: must not be null

    Property: myapp.mail.defaultSubject
    Value: null
    Reason: must not be empty
```
当然这些默认的验证注解不能满足你的验证要求，我们也可以自定义注解

如果你的验证逻辑很特殊，我们可以实现一个方法，并用 @PostConstruct 标记，如果验证失败，方法抛出异常即可, 关于 @PostConstruct，可以查看 [Spring Bean 的生命周期之缘起](https://dayarch.top/p/spring-bean-lifecycle-creation.html)

## 复杂属性类型
多数情况，我们传递给应用的参数是基本的字符串或数字。但是，有时我们需要传递诸如 List 的数据类型

### List 和 Set
假如，我们为邮件模块提供了一个 SMTP 服务的列表，我们可以添加该属性到 MailModuleProperties 类中
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-58-11%402x.png)

我们有两种方式让 Spring Boot 自动填充该 list 属性
#### application.properties
在 application.properties 文件中以数组形式书写
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-59-06%402x.png)

#### application.yml
YAML 本身支持 list 类型，所以可以在 application.yml 文件中添加:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-59-34%402x.png)

set 集合也是这种方式的配置方式，不再重复书写。另外YAML 是更好的阅读方式，层次分明，所以在实际应用中更推荐大家使用该种方式做数据配置

### Duration
Spring Boot 内置支持从配置参数中解析 durations (持续时间)，[官网文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-conversion-duration) 给出了明确的说明
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-00-08%402x.png)

我们既可以配置毫秒数数值，也可配置带有单位的文本:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-01-01%402x.png)

官网上已明确说明，配置 duration 不写单位，默认按照毫秒来指定，我们也可已通过 @DurationUnit 来指定单位:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-01-47%402x.png)

 常用单位如下:
- `ns` for nanoseconds (纳秒)
- `us` for microseconds (微秒)
- `ms` for milliseconds (毫秒)
- `s` for seconds (秒)
- `m` for minutes (分)
- `h` for hours (时)
- `d` for days (天)

### DataSize
与 Duration 的用法一毛一样，默认单位是 byte (字节)，可以通过 @DataSizeUnit  单位指定:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-02-34%402x.png)

添加配置
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-03-17%402x.png)

但是，我测试的时候打印出来结果都是以 B (bytes) 来显示

常见单位如下:
- `B` for bytes
- `KB` for kilobytes
- `MB` for megabytes
- `GB` for gigabytes
- `TB` for terabytes

## 自定义类型
有些情况，我们想解析配置参数到我们自定义的对象类型上，假设，我们我们设置最大包裹重量:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-04-08%402x.png)

在 MailModuleProperties 中添加 Weight 属性
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-04-51%402x.png)

我们可以模仿 DataSize 和 Duration 创造自己的 converter (转换器)
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-05-39%402x.png)

将其注册到 Spring Boot 上下文中
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-07-03%402x.png)

`@ConfigurationPropertiesBinding` 注解是让 Spring Boot 知道使用该转换器做数据绑定

## 使用 Spring Boot Configuration Processor 完成自动补全
我们向项目中添加依赖:
### Maven
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-09-28%402x.png)

### Gradle
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-09-58%402x.png)

重新 build 项目之后，configuration processor 会为我们创建一个 JSON 文件:
<fancybox>![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-15-23-19.png)</fancybox>

这样，当我们在 application.properties 和 application.yml 中写配置的时候会有自动提醒:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-15-24-36.png)


## 标记配置属性为 Deprecated
configuration processor 允许我们标记某一个属性为 deprecated
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-11-03%402x.png)

我们可以通过添加 `@DeprecatedConfigurationProperty` 注解到字段的 getter 方法上，来标示该字段为 deprecated，重新 build 项目，看看 JSON 文件发生了什么？
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-17-12-06%402x.png)

当我们再编写配置文件时，已经给出了明确 deprecated 提示:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-15-52-19.png)


## 总结
Spring Boot 的 `@ConfigurationProperties` 注解在绑定类型安全的 Java Bean 时是非常强大的，我们可以配合其注解属性和  `@DeprecatedConfigurationProperty` 注解获取到更友好的编程方式，同时这样让我们的配置更加模块化。

## 附加说明
以为  `@ConfigurationProperties` 注解满足我们的全部需要了吗？其实不然，Spring 官网明确给出了该注解和 `@Value` 注解的对比:
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-02-04.png)

如果使用 SpEL 表达式，我们只能选择 `@Value` 注解

另外我之前在阅读 RabbitMQ 源码时，发现 RabbitProperties 类充分的利用了 `@ConfigurationProperties`  注解特性:
- deprecated
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-05-40.png)

- Duration
![](http://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E5%BA%94%E7%94%A8%E7%B1%BB%E6%96%87%E7%AB%A0/_image/2019-07-23/2019-07-24-16-06-48.png)

- Enum
- 嵌套属性

>  感觉自己后知后觉，最近在思考，为什么小时候要阅读和背诵古诗词，文言文等经典，因为这样写文章就可以轻松熟练的引用经典。技术也一样，各种框架的源码就是学生时代的古诗词和文言文，我们要多多查看阅读，甚至背诵编程思想，这样就可以写出越来越优雅的代码

关于 `@ConfigurationProperties`  注解的使用，这里推荐 [RabbitMQ Github 源码](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java)，只需看这一个类就可以，知道怎样充分利用这个注解.

Demo 代码获取，回复公众号「demo」，打开链接查看对应的子文件夹即可

