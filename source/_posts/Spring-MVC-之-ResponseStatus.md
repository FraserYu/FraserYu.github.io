---
title: Spring MVC 之 @ResponseStatus
tags:
  - Spring
id: springmvc-response-status
categories: [Coding, Spring-Boot]
date: 2017-03-28 17:09:03
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootannotation.png
---

### @ResponseStatus
Restful webservice请求会用到@ResponseStatus 注解，该注解可用于类级别上，也可以应用在方法级别上，代表请求响应的状态，通常就是返回HttpStatus的状态码，具体可查询每个状态码代，这里简单罗列一些：

```sql
	    CONTINUE(100, "Continue"),
	    SWITCHING_PROTOCOLS(101, "Switching Protocols"),
	    PROCESSING(102, "Processing"),
	    CHECKPOINT(103, "Checkpoint"),
	    OK(200, "OK"),
	    CREATED(201, "Created"),
	    ACCEPTED(202, "Accepted"),
	    NON_AUTHORITATIVE_INFORMATION(203, "Non-Authoritative Information"),
	    NO_CONTENT(204, "No Content"),
	    RESET_CONTENT(205, "Reset Content"),
	    PARTIAL_CONTENT(206, "Partial Content"),
	    MULTI_STATUS(207, "Multi-Status"),
	    ALREADY_REPORTED(208, "Already Reported"),
	    IM_USED(226, "IM Used"),
	    MULTIPLE_CHOICES(300, "Multiple Choices"),
	    MOVED_PERMANENTLY(301, "Moved Permanently"),
	    FOUND(302, "Found"),
```

<!-- more -->

#### 应用在类级别
创建一个异常类，用该注解标注
```java
package com.zj.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(value=HttpStatus.FORBIDDEN,reason="用户不匹配")
public class UserNotMatchException extends RuntimeException{

}
```

写一个目标方法来抛出异常
```java
@RequestMapping("/testResponseStatus")
public String testResponseStatus(int i){
	if(i==0)
		throw new UserNotMatchException();
	return "hello";
}
```

这样当我们请求该方法，如果出现异常，会将用户不匹配的信息返回给浏览器，让异常信息更加明确，而不是一堆异常信息代码

#### 应用在方法级别
```java
@ResponseStatus(value=HttpStatus.FORBIDDEN,reason="用户名不匹配")
@RequestMapping("/testResponseStatus")
public String testResponseStatus(int i){
	if(i==0)
		throw new UserNotMatchException();
	return "hello";
}
```

**ResponseStatus修饰目标方法，无论它执行方法过程中有没有异常产生，用户都会得到异常的界面。而目标方法正常执行**

这个情况要看项目的使用情况，通常会制定一个标准的异常信息显示，可以灵活使用.
