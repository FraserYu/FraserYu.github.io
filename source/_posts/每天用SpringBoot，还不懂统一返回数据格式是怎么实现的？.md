---
title: Springboot返回统一JSON数据格式是怎么实现的？
tags:
  - Spring Boot
categories: [Coding, Spring-Boot]
summary: RESTful API 返回统一数据格式，这是一种非常简单的方式，透过本文，你更能了解这种简单实现方式背后的秘密
id: spring-boot-global-return
date: 2019-08-08 21:22:23
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootjson.png
description: SpringBoot RESTful API 返回统一数据格式，这是一种非常简单的方式，透过本文，你更能了解这种简单实现方式背后的秘密
keywords: SpringBoot,统一返回,JSON,HttpMessageConverter
icons: [fas fa-star yellow]

---

关于 Spring 的全局处理，我有两方面要说:
1. 统一数据返回格式
2. 统一异常处理
为了将两个问题说明清楚，将分两个章节分别说明，本章主要说第一点

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-07-25/landscape-615429_1920.jpg)

有童鞋说，我们项目都做了这种处理，就是在每个 API 都单独工具类将返回值进行封装，但这种不够优雅；我想写最少的代码完成这件事，也许有童鞋说，加几个注解就解决问题了，说的没错，**但这篇文章主要是为了说明为什么加了几个注解就解决问题了，目的是希望大家知其所以然**。

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-08/comic-4388763_1920.png)

为了更好的说明问题，本文先说明如何实现，然后再详细剖析实现原理(这很关键)

## 为什么要做统一数据返回格式
前后端分离是当今服务形式的主流，[如何设计一个好的 RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ) ，以及如何让前端小伙伴可以处理标准的 response JSON 数据结构都至关重要，为了让前端有更好的逻辑展示与页面交互处理，每一次 RESTful 请求都应该包含以下几个信息:

| 名称  | 描述  |
|---|---|
|status   | 状态码，标识请求成功与否，如 [1:成功；-1:失败]  |
| errorCode  | 错误码，给出明确错误码，更好的应对业务异常；请求成功该值可为空  |
| errorMsg  |错误消息，与错误码相对应，更具体的描述异常信息   |
| resultBody  | 返回结果，通常是 Bean 对象对应的 JSON 数据, 通常为了应对不同返回值类型，将其声明为泛型类型 |

## 实现
### 通用返回值类定义
根据上面的描述，用 Java Bean 来体现这个结构就是这样:
```java
@Data
public final class CommonResult<T> {

	private int status = 1;

	private String errorCode = "";

	private String errorMsg = "";

	private T resultBody;

	public CommonResult() {
	}

	public CommonResult(T resultBody) {
		this.resultBody = resultBody;
	}
}
```
### 配置
没错，我们需要借助几个关键注解来完成一下相关配置:
```java
@EnableWebMvc
@Configuration
public class UnifiedReturnConfig {

	@RestControllerAdvice("com.example.unifiedreturn.api")
	static class CommonResultResponseAdvice implements ResponseBodyAdvice<Object>{
		@Override
		public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
			return true;
		}

		@Override
		public Object beforeBodyWrite(Object body, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
			if (body instanceof CommonResult){
				return body;
			}

			return new CommonResult<Object>(body);
		}
	}
}
```
到这里就结束了，我们就可以纵情的写任何 RESTful API 了，所有的返回值都会有统一的 JSON 结构
### 测试
新建 UserController，添加相应的 RESTful API，测试用例写的比较简单，只为了说明返回值的处理
```java
@RestController
@RequestMapping("/users")
public class UserController {


	@GetMapping("")
	public List<UserVo> getUserList(){
		List<UserVo> userVoList = Lists.newArrayListWithCapacity(2);
		userVoList.add(UserVo.builder().id(1L).name("日拱一兵").age(18).build());
		userVoList.add(UserVo.builder().id(2L).name("tan").age(19).build());
		return userVoList;
	}
}
```
打开浏览器输入地址测试:  *http://localhost:8080/users/* ，我们可以看到返回了 List JSON 数据

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-08/2019-08-08-20-16-04.png)


继续添加 RESTful API，根据用户 ID 查询用户信息
```java
@GetMapping("/{id}")
public UserVo getUserByName(@PathVariable Long id){
    return UserVo.builder().id(1L).name("日拱一兵").age(18).build();
}
```

打开浏览器输入地址测试:  *http://localhost:8080/users/1* ，我们可以看到返回了单个 User JSON 数据

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-08/2019-08-08-20-16-32.png)


添加一个返回值类型为 ResponseEntity 的 API
```java
@GetMapping("/testResponseEntity")
public ResponseEntity getUserByAge(){
    return new ResponseEntity(UserVo.builder().id(1L).name("日拱一兵").age(18).build(), HttpStatus.OK);
}
```

打开浏览器输入地址测试:  *http://localhost:8080/users/testResponseEntity* ，我们可以看到同样返回了单个 User JSON 数据

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-08/2019-08-08-20-17-07.png)

## 解剖实现过程
我会将关键部分一一说明清楚，断案还需小伙伴自己去案发现场(打开自己的 IDE 查看)

故事要从 `@EnableWebMvc` 这个注解说起，打开该注解看:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```
通过 `@Import` 注解引入了 `DelegatingWebMvcConfiguration.class`，那来看这个类吧:
```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
    ...
}
```
有 `@Configuration` 注解，你应该很熟悉了，该类的父类 `WebMvcConfigurationSupport` 中却隐藏着一段关键代码:
```java
@Bean
public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
    RequestMappingHandlerAdapter adapter = createRequestMappingHandlerAdapter();
    ...
    return adapter;
}
```
RequestMappingHandlerAdapter 是每一次请求处理的关键，来看该类的定义:
```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
		implements BeanFactoryAware, InitializingBean {
    ...
}
```
该类实现了 InitializingBean 接口，我在 [Spring Bean 生命周期之“我从哪里来”？](https://mp.weixin.qq.com/s/I7zgbOoCgAUnRFy4avipiQ) 这篇文章中明确说明了 Spring Bean 初始化的几个关键，其中 InitializingBean 接口的
`afterPropertiesSet` 方法就是关键之一，在 RequestMappingHandlerAdapter 类中同样重写了该方法:
```java
@Override
public void afterPropertiesSet() {
    // Do this first, it may add ResponseBody advice beans
    initControllerAdviceCache();

    if (this.argumentResolvers == null) {
        List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
        this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.initBinderArgumentResolvers == null) {
        List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
        this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.returnValueHandlers == null) {
        List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
        this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
    }
}
```
该方法内容都非常关键，但我们先来看 `initControllerAdviceCache` 方法，其他内容后续再单独说明:
```java
private void initControllerAdviceCache() {
        ...
    if (logger.isInfoEnabled()) {
        logger.info("Looking for @ControllerAdvice: " + getApplicationContext());
    }

    List<ControllerAdviceBean> beans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());
    AnnotationAwareOrderComparator.sort(beans);

    List<Object> requestResponseBodyAdviceBeans = new ArrayList<Object>();

    for (ControllerAdviceBean bean : beans) {
        ...
        if (ResponseBodyAdvice.class.isAssignableFrom(bean.getBeanType())) {
            requestResponseBodyAdviceBeans.add(bean);
        }
    }
}
```

通过 ControllerAdviceBean 静态方法扫描 `ControllerAdvice` 注解，可是我们在实现上使用的是 `@RestControllerAdvice` 注解，打开看该注解:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
```
> 该注解由 `@ControllerAdvice` 和 `@ResponseBody` 标记，就好比你熟悉的 `@RestController` 注解由 `@Controller` 和 `@ResponseBody`  标记是一样的

到这里你已经知道我们用 `@RestControllerAdvice` 标记的 Bean 是如何被加载到 Spring 上下文的，接下来就要知道是 Spring 是如何使用我们的 bean 以及对返回 body 做处理的

其实在 [HttpMessageConverter是如何转换数据的？](https://mp.weixin.qq.com/s/wWj8M46EmxRRgBWdch2Big) 这篇文章中已经说明了一部分，希望小伙伴先看这篇文章，下面的部分就会秒懂了，我们在这里做进一步的说明

在 AbstractMessageConverterMethodProcessor 的 `writeWithMessageConverters` 方法中，有一段核心代码:
```java
if (messageConverter instanceof GenericHttpMessageConverter) {
    if (((GenericHttpMessageConverter) messageConverter).canWrite(
            declaredType, valueType, selectedMediaType)) {
        outputValue = (T) getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType,
                (Class<? extends HttpMessageConverter<?>>) messageConverter.getClass(),
                inputMessage, outputMessage);
            ...
        return;
    }
}
```
可以看到通过 getAdvice() 调用了 `beforeBodyWrite` 方法，我们已经接近真相了
```java
protected RequestResponseBodyAdviceChain getAdvice() {
    return this.advice;
}
```
RequestResponseBodyAdviceChain，看名字带有 Chain，很明显用到了「责任链设计模式」，这些内容在 [不得不知的责任链设计模式](https://mp.weixin.qq.com/s/SqIkX01JwR6QXKTa6Kqp4Q) 文章中明确说明过，只不过它传递责任链以循环的方式完成:
```java
class RequestResponseBodyAdviceChain implements RequestBodyAdvice, ResponseBodyAdvice<Object> {

    @Override
	public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType contentType,
			Class<? extends HttpMessageConverter<?>> converterType,
			ServerHttpRequest request, ServerHttpResponse response) {

		return processBody(body, returnType, contentType, converterType, request, response);
	}

    @SuppressWarnings("unchecked")
	private <T> Object processBody(Object body, MethodParameter returnType, MediaType contentType,
			Class<? extends HttpMessageConverter<?>> converterType,
			ServerHttpRequest request, ServerHttpResponse response) {

		for (ResponseBodyAdvice<?> advice : getMatchingAdvice(returnType, ResponseBodyAdvice.class)) {
			if (advice.supports(returnType, converterType)) {
				body = ((ResponseBodyAdvice<T>) advice).beforeBodyWrite((T) body, returnType,
						contentType, converterType, request, response);
			}
		}
		return body;
	}
}
```
我们重写的 `beforeBodyWrite` 方法终究会被调用到，真相就是这样了!!!

其实还没完，你有没有想过，如果我们的 API 方法返回值是 `org.springframework.http.ResponseEntity<T>` 类型，我们可以指定 HTTP 返回状态码，但是这个返回值会直接放到我们的 beforeBodyWrite 方法的 body 参数中吗？如果这样做很明显是错误的，因为 ResponseEntity 包含很多我们非业务数据在里面，那 Spring 是怎么帮我们处理的呢？

在我们方法取得返回值并且在调用 `beforeBodyWrite` 方法之前，还要选择 HandlerMethodReturnValueHandler 用于处理不同的 Handler 来处理返回值

在类 HandlerMethodReturnValueHandlerComposite 中的 handleReturnValue 方法中
```java
@Override
public void handleReturnValue(Object returnValue, MethodParameter returnType,
        ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

    HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
    if (handler == null) {
        throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
    }
    handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
}
```
通过调用 selectHandler 方法来选择合适的 handler，Spring 内置了很多个 Handler，我们来看类图:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-08/2019-08-08-16-36-31.png)


HttpEntityMethodProcessor 就是其中之一，它重写了 supportsParameter 方法，支持 HttpEntity 类型，即支持 ResponseEntity 类型:
```java
@Override
public boolean supportsParameter(MethodParameter parameter) {
    return (HttpEntity.class == parameter.getParameterType() ||
            RequestEntity.class == parameter.getParameterType());
}
```
所以当我们返回的类型为 ResponseEntity 时，就要通过 HttpEntityMethodProcessor 的 handleReturnValue 方法来处理我们的结果:
```java
@Override
public void handleReturnValue(Object returnValue, MethodParameter returnType,
        ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

    ...
    if (responseEntity instanceof ResponseEntity) {
        int returnStatus = ((ResponseEntity<?>) responseEntity).getStatusCodeValue();
        outputMessage.getServletResponse().setStatus(returnStatus);
        if (returnStatus == 200) {
            if (SAFE_METHODS.contains(inputMessage.getMethod())
                    && isResourceNotModified(inputMessage, outputMessage)) {
                // Ensure headers are flushed, no body should be written.
                outputMessage.flush();
                // Skip call to converters, as they may update the body.
                return;
            }
        }
    }

    // Try even with null body. ResponseBodyAdvice could get involved.
    writeWithMessageConverters(responseEntity.getBody(), returnType, inputMessage, outputMessage);

    // Ensure headers are flushed even if no body was written.
    outputMessage.flush();
}
```
该方法提取出 responseEntity.getBody()，并传递个 MessageConverter，然后再继续调用 `beforeBodyWrite` 方法，这才是真相!!!

**这是 RESTful API 正常返回内容的情况，下一篇文章，让我们来侦查一下统一异常情况的处理以及实现原理**

## 灵魂追问
1. 返回值是非 ResponseEntity 类型时，用的是什么 handler？它支持的返回值类型是什么？看过你也许就知道为什么要用 `@ResponseBody` 注解了
2. 你有追踪过 DispatchServlet 的整个请求过程吗？

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)


--------

## 推荐阅读
+ [只会用 git pull ？有时候你可以尝试更优雅的处理方式 ](https://mp.weixin.qq.com/s/6dg3u2PkcTSQHu_3T_QYnA)
+ [双亲委派模型：大厂高频面试题，轻松搞定](https://mp.weixin.qq.com/s/Dnr1jLebvBUHnziZzSfcrA)
+ [面试还不知道BeanFactory和ApplicationContext的区别？](https://mp.weixin.qq.com/s/YBQB086ADBjHUmwrFQrWew)
+ [如何设计好的RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ)
+ [程序猿为什么要看源码？](https://mp.weixin.qq.com/s/V7h8O6pVFQ-nr_iA2SNqtw)

--------
> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享
> - 高效工具汇总 | 回复「工具」
> - 面试问题分析与解答
> - 技术资料领取 | 回复「资料」

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)
