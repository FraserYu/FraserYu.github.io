---
title: Spring 数据绑定剖析
date: 2019-12-12 19:59:04
tags:
    - Spring Boot
    - Spring Data Binding
categories: [Coding, Spring-Boot]
id: spring-data-binding-mechanism
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootconverter.png
description: Spring Boot的整个请求过程，都离不开Spring 数据绑定机制，透彻理解该机制，可以让我们的编码更加整洁清晰，同时可以应对更多特殊请求处理
keywords: Spring数据绑定,Spring Data Binding,resolver
---

## 前言
在剖析完 「[Spring Boot 统一数据格式是怎么实现的？](https://dayarch.top/p/spring-boot-global-return.html) 」文章之后，一直觉得有必要说明一下 Spring's Data Binding Mechanism 「Spring 数据绑定机制」。

默认情况下，Spring 只知道如何转换简单数据类型。比如我们提交的 int、String 或 boolean类型的请求数据，它会自动绑定到与之对应的 Java 类型。但在实际项目中，远远不够，因为我们可能需要绑定更复杂的对象类型。

我们需要了解 Spring 数据绑定机制，这样我们就可以更灵活的做全局配置或自定义配置，进而让我们的 RESTful API 更简洁，可读性也更好。本文依旧先通过示例代码说明实现，然后进行源码分析，带领大家了解这个机制是如何生效的，知其所以然， Let's go......

## Spring 数据绑定

### 日期绑定
先来看下面一小段代码
```java
@RestController
@RequestMapping("/bindings/")
@Slf4j
public class BindingController {


	@GetMapping("/{date}")
	public void getSpecificDateInfo(@PathVariable LocalDateTime date) {
		log.info(date.toString());
	}
}
```

当我们用 Postman 请求这个 API
```http
http://localhost:8080/rgyb/bindings/2019-12-10 12:00:00
```

如我们所料，抛出数据类型转换异常
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_13-44-29.jpg)
因为 Spring 默认不支持将 String 类型的请求参数转换为 LocalDateTime 类型，所以我们需要自定义 converter 「转换器」完整整个转换过程

自定义转换器 `StringToLocalDateTimeConverter`，使其实现 `org.springframework.core.convert.converter.Converter<S, T>` 接口，在重写的 convert 方法中实现我们自定义的转换逻辑

```java
public class StringToLocalDateTimeConverter implements Converter<String, LocalDateTime> {
	@Override
	public LocalDateTime convert(String s) {
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss", Locale.CHINESE);
		return LocalDateTime.parse(s, formatter);
	}
}
```

将转换器注册到上下文中:
```java
@Configuration
public class UnifiedReturnConfig implements WebMvcConfigurer {
    @Override
	public void addFormatters(FormatterRegistry registry) {
		registry.addConverter(new StringToLocalDateTimeConverter());
	}
}
```

重新访问上面链接，查看控制台，按照预期得到相应转换结果:
```shell
c.e.unifiedreturn.api.BindingController  : 2019-12-10T12:00
```

知道了这个，比如我们常用的枚举类型也可以应用这种方式做数据绑定

### 枚举类型绑定
同样的套路，自定义转换器
```java
public class StringToEnumConverter implements Converter<String, Modes> {
	
	@Override
	public Modes convert(String s) {
		return Modes.valueOf(s);
	}
}
```
将其添加至上下文，请小伙伴们自行尝试吧，知道了这个，我们再也不用在 RESTful API 内部做数据转换了，我们做到了全局控制，同时让整个 API 看起来更加清晰简洁

### 绑定对象
在某些情况下，我们希望将数据绑定到对象，这时我们可能马上联想起来使用 `@RequestBody` 注解，该注解通常用于获取 POST 请求体，并将其转换相应的数据对象

在实际业务场景中，除了`请求体`中的数据，我们同样需要`请求头`中的数据，比如 `token` ，token 中包含当前登陆用户的信息，每一次 RESTful 请求我们都需要从 header 中获取 token 数据处理实际业务，这种场景，上文提到的 `Converter` 以及 `@RequestBody` 显然不能满足我们的需求，此时我们就要换另一种解决方案 : `HandlerMethodArgumentResolver`

首先我们需要自定义一个注解 `LoginUser` (**运行时生效，作用于参数上**)
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface LoginUser {
}
```

然后自定义 `LoginUserArgumentResolver` ，使其实现 `HandlerMethodArgumentResolver` 接口

```java
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
	@Override
	public boolean supportsParameter(MethodParameter methodParameter) {
        //判断参数是否有自定义注解 LoginUser 修饰
		return methodParameter.hasParameterAnnotation(LoginUser.class);
	}

	@Override
	public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {

		HttpServletRequest request = (HttpServletRequest) nativeWebRequest.getNativeRequest();

		LoginUserVo loginUserVo = new LoginUserVo();

		String token = request.getHeader("token");
		if (Strings.isNotBlank(token)){
			//通常这里需要编写 token 解析逻辑，并将其放到 LoginUserVo 对象中
			//logic
		}

		//在此为了快速简洁的做演示说明，省略掉解析 token 部分，直接从 header 指定 key 中获取数据
		loginUserVo.setId(Long.valueOf(request.getHeader("userId")));
		loginUserVo.setName(request.getHeader("userName"));
		return loginUserVo;
	}
}
```

依旧将自定义的 `LoginUserArgumentResolver` 添加到上下文中
```java
@Override
public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(new LoginUserArgumentResolver());
}
```

编写 API:
```java
@GetMapping("/id")
public void getLoginUserInfo(@LoginUser LoginUserVo loginUserVo) {
    log.info(loginUserVo.toString());
}
```

通过 Postman 请求，在 header 中设置好相应的 K-V，如下图
```http
http://localhost:8080/rgyb/bindings/id
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_15-32-15.jpg)

发送请求，查看控制台，得到预期结果

```shell
c.e.unifiedreturn.api.BindingController  : LoginUserVo(id=111111, name=rgyb)
```

相信到这里，你已经了解了基本的使用，接下来我们进行源码分析，透过现象看本质 (希望可以打开 IDE 跟着步骤查看)

## Spring 数据绑定源码分析
首先我们需要了解我们自定义的  `LoginUserArgumentResolver` 是如何被加载到上下文中的，在你看过  [HttpMessageConverter转换原理解析](https://dayarch.top/p/spring-boot-messageconverter.html)  和 [Springboot返回统一JSON数据格式是怎么实现的？](https://dayarch.top/p/spring-boot-global-return.html)后，你也许已经有了眉目，同加载 MessageConverter 如出一辙，在 `RequestMappingHandlerAdapter` 类中，同样有添加 ArgumentResolver 的方法，该方法会把系统内置的 resolver 和用户自定义的 resolver 都加载到上下文中，关键代码展示如下:

```java
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
    List<HandlerMethodArgumentResolver> resolvers = new ArrayList();
    resolvers.add(new RequestParamMethodArgumentResolver(this.getBeanFactory(), false));
    //其他内置 resolver

    resolvers.add(new RequestResponseBodyMethodProcessor(this.getMessageConverters(), this.requestResponseBodyAdvice));
    ...
    ...

    if (this.getCustomArgumentResolvers() != null) {
        resolvers.addAll(this.getCustomArgumentResolvers());
    }

    ...
    ...
    return resolvers;
}
```

在 [HttpMessageConverter转换原理解析](https://dayarch.top/p/spring-boot-messageconverter.html) 文章中有一段调用栈跟踪，我再次粘贴在此处，并用红框做出标记，其实我们在分析 messageConverter 时已经悄悄的路过了我们本节要说的内容
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_16-00-32.jpg)

我们进入相应的类中瞧一瞧:
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_16-02-47.jpg)

到这里你应该猛的了解这背后的道理了吧

接下来，我们来验证我们天天用的 `@RequestBody` 注解是不是这个套路呢？
处理该注解的类是 `RequestResponseBodyMethodProcessor`，查看其类图，发现其依旧实现了 `HandlerMethodArgumentResolver` 接口

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_16-09-04.jpg)

打开该类，你会看到下图代码，重点地方我已标记出来
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_16-18-36.jpg)

整体处理流程如出一辙，只不过在里面调用了 messageConverter 来解析 JSON 数据。

## 总结
本文说的 Converter 和 ArgumentResolver 以及在 Spring MVC 中常用的 `@InitBinder` 注解整体过程都如出一辙，大家都可以按照这个思路来查看具体的实现。另外，在我们完成日常编码工作时，都可以从 Spring 现有的处理方式中摸索到一些解决方案，但前提是你了解 Spring 底层的一些调用过程

最后希望小伙伴打开 IDE 切实查看相应代码，你一定还会有新发现，我们可以一起探讨。本文代码已上传，公众号回复「demo」，打开链接查看 「spring-boot-unified-return」文件夹内容即可，也可以顺路回顾以前 Spring Boot 统一返回格式的代码实现

- - - - - 

## 灵魂追问
![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2019-12-12_18-12-09.jpg)
1. 如上图所示，在追中源码时，发现`HandlerMethodArgumentResolverComposite` 是 `HandlerMethodArgumentResolver` 的实现类之一，其中有一个 Map 类型的成员变量，通常我们使用 Map，key 的类型多数为 String 类型，但看到这个 Map 中有这样的 key 你马上想到的是什么？基础面试经常会问 equals 和 hashcode 的问题，下一篇文章会借着这个类来分析说明一下你总困惑的这件小事
2. 对于 Spring Boot 的整个调用过程，你能描述出整体流程吗？
3. Spring 内置多少个 Resolver？你可以跟踪调试获取到

