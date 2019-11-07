---
title: SpringBoot统一异常处理
tags:
  - Exception
categories: [Coding, Spring-Boot]
summary: 学会统一处理异常，让你的代码更清秀，了解其背后原理，加深理解，做到知其然也要知其所以然
id: spring-boot-global-exception
date: 2019-08-09 15:13:46
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootexception.png
description: SpringBoot RESTful API 统一处理异常，让你的代码更清秀，了解其背后原理，加深理解，做到知其然也要知其所以然
keywords: SpringBoot,统一异常,统一异常处理,Exception
---


## 话说异常
`「欲渡黄河冰塞川，将登太行雪满天」`，无论生活还是计算机世界难免发生异常，上一篇文章[RESTful API 返回统一JSON数据格式](https://fraseryu.github.io/2019/08/08/mei-tian-yong-springboot-huan-bu-dong-tong-yi-fan-hui-shu-ju-ge-shi-shi-zen-me-shi-xian-de/#toc-heading-2) 说明了统一返回的处理，这是请求一切正常的情形；**这篇文章将说明如何统一处理异常，以及其背后的实现原理**，老套路，先实现，后说明原理，有了上一篇文章的铺底，相信，理解这篇文章就驾轻就熟了

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-09/sunset-3325080_1920.jpg)

## 实现
### 新建业务异常
新建 BusinessException.class 类表示业务异常，**注意这是一个 Runtime 异常**
```java
@Data
@AllArgsConstructor
public final class BusinessException extends RuntimeException {

	private String errorCode;

	private String errorMsg;

}
```

### 添加统一异常处理静态方法
在 CommonResult 类中添加静态方法 errorResult 用于接收异常码和异常消息:
```java
public static <T> CommonResult<T> errorResult(String errorCode, String errorMsg){
    CommonResult<T> commonResult = new CommonResult<>();
    commonResult.errorCode = errorCode;
    commonResult.errorMsg = errorMsg;
    commonResult.status = -1;
    return commonResult;
}
```

### 配置
同样要用到 `@RestControllerAdvice` 注解，将统一异常添加到配置中:
```java
@RestControllerAdvice("com.example.unifiedreturn.api")
static class UnifiedExceptionHandler{

    @ExceptionHandler(BusinessException.class)
    public CommonResult<Void> handleBusinessException(BusinessException be){
        return CommonResult.errorResult(be.getErrorCode(), be.getErrorMsg());
    }
}
```
三部搞定，到这里无论是 Controller 还是 Service 中，只要抛出 BusinessException, 我们都会返回给前端一个统一数据格式

## 测试
将 UserController 中的方法进行改造，直接抛出异常:
```java
@GetMapping("/{id}")
public UserVo getUserById(@PathVariable Long id){
    throw new BusinessException("1001", "根据ID查询用户异常");
}
```
浏览器中输入:  *http://localhost:8080/users/1*


![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-09/2019-08-09-14-29-07.png)


在 Service 中抛出异常:
```java
@Service
public class UserServiceImpl implements UserService {

	/**
	 * 根据用户ID查询用户
	 *
	 * @param id
	 * @return
	 */
	@Override
	public UserVo getUserById(Long id) {
		throw new BusinessException("1001", "根据ID查询用户异常");
	}
}
```
运行是得到同样的结果，所以我们尽可能的抛出异常吧 (作为一个程序猿这种心理很可拍)

## 解剖实现过程
解剖这个过程是相当纠结的，为了更好的说(yin)明(wei)问(wo)题(lan)，我要说重中之重了，真心希望看该文章的童鞋自己去案发现场发现线索
还是在 WebMvcConfigurationSupport 类中实例化了 HandlerExceptionResolver Bean
```java
@Bean
public HandlerExceptionResolver handlerExceptionResolver() {
    List<HandlerExceptionResolver> exceptionResolvers = new ArrayList<>();
    configureHandlerExceptionResolvers(exceptionResolvers);
    if (exceptionResolvers.isEmpty()) {
        addDefaultHandlerExceptionResolvers(exceptionResolvers);
    }
    extendHandlerExceptionResolvers(exceptionResolvers);
    HandlerExceptionResolverComposite composite = new HandlerExceptionResolverComposite();
    composite.setOrder(0);
    composite.setExceptionResolvers(exceptionResolvers);
    return composite;
}
```
和上一篇文章一毛一样的套路，ExceptionHandlerExceptionResolver 实现了 InitializingBean 接口，重写了 afterPropertiesSet 方法:
```java
@Override
public void afterPropertiesSet() {
    // Do this first, it may add ResponseBodyAdvice beans
    initExceptionHandlerAdviceCache();
    ...
}

private void initExceptionHandlerAdviceCache() {
    if (getApplicationContext() == null) {
        return;
    }

    List<ControllerAdviceBean> adviceBeans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());
    AnnotationAwareOrderComparator.sort(adviceBeans);

    for (ControllerAdviceBean adviceBean : adviceBeans) {
        Class<?> beanType = adviceBean.getBeanType();
        if (beanType == null) {
            throw new IllegalStateException("Unresolvable type for ControllerAdviceBean: " + adviceBean);
        }
        // 重点看这个构造方法
        ExceptionHandlerMethodResolver resolver = new ExceptionHandlerMethodResolver(beanType);
        if (resolver.hasExceptionMappings()) {
            this.exceptionHandlerAdviceCache.put(adviceBean, resolver);
        }
        if (ResponseBodyAdvice.class.isAssignableFrom(beanType)) {
            this.responseBodyAdvice.add(adviceBean);
        }
    }
}
```
重点看上面我用注释标记的构造方法，代码很好懂，仔细看看吧，其实就是筛选出我们用 @ExceptionHandler 注解标记的方法并放到集合当中，用于后续全局异常捕获的匹配
```java
/**
 * A constructor that finds {@link ExceptionHandler} methods in the given type.
 * @param handlerType the type to introspect
 */
public ExceptionHandlerMethodResolver(Class<?> handlerType) {
    for (Method method : MethodIntrospector.selectMethods(handlerType, EXCEPTION_HANDLER_METHODS)) {
        for (Class<? extends Throwable> exceptionType : detectExceptionMappings(method)) {
            addExceptionMapping(exceptionType, method);
        }
    }
}


/**
 * Extract exception mappings from the {@code @ExceptionHandler} annotation first,
 * and then as a fallback from the method signature itself.
 */
@SuppressWarnings("unchecked")
private List<Class<? extends Throwable>> detectExceptionMappings(Method method) {
    List<Class<? extends Throwable>> result = new ArrayList<>();
    detectAnnotationExceptionMappings(method, result);
    if (result.isEmpty()) {
        for (Class<?> paramType : method.getParameterTypes()) {
            if (Throwable.class.isAssignableFrom(paramType)) {
                result.add((Class<? extends Throwable>) paramType);
            }
        }
    }
    if (result.isEmpty()) {
        throw new IllegalStateException("No exception types mapped to " + method);
    }
    return result;
}

private void detectAnnotationExceptionMappings(Method method, List<Class<? extends Throwable>> result) {
    ExceptionHandler ann = AnnotatedElementUtils.findMergedAnnotation(method, ExceptionHandler.class);
    Assert.state(ann != null, "No ExceptionHandler annotation");
    result.addAll(Arrays.asList(ann.value()));
}
```
到这里，我们用 `@RestControllerAdvice` 和 `@ExceptionHandler` 注解就会被 Spring 扫描到上下文，供我们使用

让我们回到你最熟悉的调用的入口 DispatcherServlet 类的  doDispatch 方法:
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ...

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            ...
            // 当请求发生异常，该方法会通过 catch 捕获异常
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
        ...

        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 调用该方法分析捕获的异常
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    ...
}
```
接下来，我们来看 processDispatchResult 方法，这里只要展示调用栈你就会眼前一亮了，又是为了返回统一格式数据:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-09/2019-08-09-15-01-27.png)


## 总结
上一篇文章的返回统一数据格式是基础，当异常情况发生时，只不过需要将异常信息提取出来。本文主要为了说明问题，剖析原理，好多地方设计方式是不可取，比如我们最好将异常封装在一个 Enum 类，通过 enum 对象抛出异常等，如果你用到这些，去完善你的设计方案吧

回复 「demo」，打开链接，查看文件夹 「unifiedreturn」下内容，获取完整代码

## 附加说明
之前看到的一本书对异常的分类让我印象深刻，在此摘录一小段分享给大家:
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-09/Untitled%20Diagram.png)

结合出国旅行的例子说明异常分类:
> - 机场地震，属于不可抗力，对应异常分类中的 `Error`，在制订出行计划时，根本不需要把这个部分的异常考虑进去
> - 堵车属于 `checked` 异常，应对这种异常，我们可以提前出发，或者改签机票。而飞机延误异常,虽然也需要 check，但我们无能为力，只能持续关注航班动态
> - 没有带护照，明显属于可`提前预测的异常`，只要出发前检查即可避免；去机场路上车子抛锚，这个异常是突发的，虽然难以预料，但是必须处理，属于`需要捕捉的异常`，可以通过更换交通工具；应对检票机器故障属于 `可透出异常`，交由航空公司处理,我们无须关心

## 灵魂追问
1. 这两篇文章，你学到了哪些设计模式？
2. 你能熟练的使用反射吗？当看源码是会看到很多反射的应用
3. 你了解 Spring CGLIB 吗？它的工作原理是什么？

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)

### JSON-Viewer

JSON-Viewer  是 Chrome 浏览器的插件，用于快速解析及格式化 json 内容，在 Chrome omnibox（多功能输入框）输入`json-viewer + TAB` ，将 json 内容拷贝进去，然后输入回车键，将看到结构清晰的 json 数据，同时可以自定义主题

![](http://ww1.sinaimg.cn/large/8dc363e6ly1g39u1u0er6j21v80tyadz.jpg)

另外，前端人员打开开发者工具，双击请求链接，会自动将 response 中的 json 数据解析出来，非常方便
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
