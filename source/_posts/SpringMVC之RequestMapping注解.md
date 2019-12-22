---
title: SpringMVC之RequestMapping注解
tags:
	- 注解
	- annotation
categories: [Coding, Spring-Boot]
id: springmvc-requestmapping-annotation
date: 2016-01-26 10:49:59
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootannotation.png
description: RequestMapping注解详解，属性详解
keywords: RequestMapping,注解,SpringMVC
---
# SpringMVC之RequestMapping注解
-----------------------------------------------
### 什么是RequestMapping注解
在SpringMVC的API中我们发现对@interface RequestMapping 有如下的定义：
> *Annotation for mapping web requests onto specific handler classes and/or handler methods. Provides a consistent style between Servlet and Portlet environments, with the semantics adapting to the concrete environment. *

翻译过来的意思就是通过在在类和方法上的限制来匹配request

<!-- more -->

### RequestMapping的几大属性
在Spring4.2.4的API中发现RequestMapping有八大属性来mappingURL请求
1. **value**
>  *In a Servlet environment this is an alias for path(). For example @RequestMapping("/foo") is equivalent to @RequestMapping(path="/foo"). *
> 在Servlet中value就是表示path，表示请求的路径的mapping, 支持通配符形式

2. **method**
> *The HTTP request methods to map to, narrowing the primary mapping: **GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE. ***
> Http请求中有以上几种方式，我们可以通过加入method来限制mapping范围

3. **params**
> *Same format for any environment: a sequence of "myParam=myValue" style expressions, with a request only mapped if each such parameter is found to have the given value. Expressions can be negated by using the "!=" operator, as in "myParam!=myValue". "myParam" style expressions are also supported, with such parameters having to be present in the request (allowed to have any value). Finally, "!myParam" style expressions indicate that the specified parameter is not supposed to be present in the request. *
> 请求参数中必须有key**等于**某个value或者key必须**不等于**某个value，多个条件是**与**的关系。

4. **headers**
> *Same format for any environment: a sequence of "My-Header=myValue" style expressions, with a request only mapped if each such header is found to have the given value. Expressions can be negated by using the "!=" operator, as in "My-Header!=myValue". "My-Header" style expressions are also supported, with such headers having to be present in the request (allowed to have any value). Finally, "!My-Header" style expressions indicate that the specified header is not supposed to be present in the request.*
***Also supports media type wildcards (*), for headers such as Accept and Content-Type. For instance,will match requests with a Content-Type of "text/html", "text/plain", etc. ***
和params属性类似，支持通配符匹配。
```java
@RequestMapping(value = "/something", headers = "content-type=text/*")
```

5. **name**
> *Assign a name to this mapping. Supported at the type level as well as at the method level! When used on both levels, a combined name is derived by concatenation with "#" as separator.*
> 给mapping命名，支持在类和方法上使用该属性，联合起来的name用#来分隔

6. **consumes**
> *The consumable media types of the mapped request, narrowing the primary mapping. The format is a single media type or a sequence of media types, with a request only mapped if the Content-Type matches one of these media types. Examples:

		consumes = "text/plain"
		consumes = {"text/plain", "application/*"}
> *Expressions can be negated by using the "!" operator, as in "!text/plain", which matches all requests with a Content-Type other than "text/plain". *
> consumes数组里面可以是一个或多个值，只要匹配到他们其中一个即可。

7. **produces**
> *he producible media types of the mapped request, narrowing the primary mapping. The format is a single media type or a sequence of media types, with a request only mapped if the Accept matches one of these media types. Examples:*

		 produces = "text/plain"
		 produces = {"text/plain", "application/*"}
		 produces = "application/json; charset=UTF-8"
> *It affects the actual content type written, for example to produce a JSON response with UTF-8 encoding, "application/json; charset=UTF-8" should be used. Expressions can be negated by using the "!" operator, as in "!text/plain", which matches all requests with a Accept other than "text/plain". *
> 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

8. **path**
> *In a Servlet environment only: the path mapping URIs (e.g. "/myPath.do"). Ant-style path patterns are also supported (e.g. "/myPath/*.do"). At the method level, relative paths (e.g. "edit.do") are supported within the primary mapping expressed at the type level. Path mapping URIs may contain placeholders (e.g. "/${connect}") *

以上所有属性都是来限制mapping要求的，但是常用的是**params、method、consumes**.

### 验证属性
1. **value**
value支持通配符的表现形式,  **?**表示匹配文件名中的一个字符, * 表示匹配任意一个或多个字符, ** 表示匹配多层路径的一个或多个字符, @PathVariable算是URL参数的占位符。

		@RequestMapping(value="/?/testValue1/{id}")
		public String testValue1(@PathVariable("id") String id){
			System.out.println("value测试" + id);
			return SUCCESS;
		}
> **URL**:  http://localhost:8080/springMVC1/demo/a/testValue1/28
> **Result**:  value测试28
>
现在我们遇到一种情况，如果我们的id是1.2.0007.9 这种类型的字符，如果在还向上面那样来传参数只会的到1.0这样的结果，于是我们需要用到正则表达式：

		@RequestMapping(value="/?/testValue1/{id：.*}")
				public String testValue1(@PathVariable("id") String id){
					System.out.println("value测试" + id);
					return SUCCESS;
				}

	写成{id:.*} 或 {id:.+} 都可以。

2. **method**
jsp部分代码如下：

		<dl>
			<dt>Test method</dt>
			<dd>
				<a href="demo/testMethod_GET">Method_GET</a>
			</dd>
			<dd>
				<form action="demo/testMethod_POST" method="post">
					<input type="submit" value="Method_POST"/>
				</form>
			</dd>
			<dd>
				<form action="demo/testMethod_DELETE" method="post">
					<input type="hidden" name="_method" value="DELETE"/>
					<input type="submit" value="Method_DELETE"/>
				</form>
			</dd>
			<dd>
				<form action="demo/testMethod_PUT" method="post">
					<input type="hidden" name="_method" value="PUT"/>
					<input type="submit" value="Method_PUT"/>
				</form>
			</dd>
		</dl>
GET和POST方法很好理解，可是在DELETE方法和PUT方法中我们却加入了一个hidden类型的input，值分别为DELETE和PUT，为什么要加这个隐藏域，我们先来看一下Spring中org.springframework.web.filter.HiddenHttpMethodFilter类的一段源码：

		public static final String DEFAULT_METHOD_PARAM = "_method";
		private String methodParam = DEFAULT_METHOD_PARAM;

		@Override
		protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)throws ServletException, IOException {

			String paramValue = request.getParameter(this.methodParam);
			if ("POST".equals(request.getMethod()) && StringUtils.hasLength(paramValue)) {
				String method = paramValue.toUpperCase(Locale.ENGLISH);
				HttpServletRequest wrapper = new HttpMethodRequestWrapper(request, method);
				filterChain.doFilter(wrapper, response);
			}
			else {
				filterChain.doFilter(request, response);
			}
		}
这段代码的主要意思就是如果_method中存在值，并且是post方法，那我们的servlet对这个方法进行处理，为了完成这个功能我们需要进一步在web.xml中配置这个org.springframework.web.filter.HiddenHttpMethodFilter，如下：

		<!-- org.springframework.web.filter.HiddenHttpMethodFilter 可以将post请求转为PUT和DELETE请求 -->
		<filter>
			<filter-name>HiddenHttpMethodFilter</filter-name>
			<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
		</filter>

		<filter-mapping>
			<filter-name>HiddenHttpMethodFilter</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
现在我们来看JAVA后台controller中的代码，如下：

		@RequestMapping(value="/testMethod_GET", method=RequestMethod.GET)
		public String testMethod_GET(){
			System.out.println("This is GET method");
			return SUCCESS;
		}

		@RequestMapping(value="/testMethod_POST",method=RequestMethod.POST)
		public String testMethod_POST(){
			System.out.println("This is POST method");
			return SUCCESS;
		}

		@RequestMapping(value="/testMethod_DELETE", method=RequestMethod.DELETE)
		public String testMethod_DELETE(){
			System.out.println("This is DELETE method");
			return SUCCESS;
		}

		@RequestMapping(value="/testMethod_PUT", method=RequestMethod.PUT)
		public String testMethod_PUT(){
			System.out.println("This is PUT method");
			return SUCCESS;
		}		

3. **params**
jsp部分代码如下：

		<dl>
			<dt>Test params</dt>
			<dd>
				<a href="demo/testParams?username&age=10">testParams</a>
			</dd>
		</dl>
Java的Controller代码如下：

		@RequestMapping(value="/testParams", params={"username","age!=0"})
		public String testParams(){
			System.out.println("testParams");
			return SUCCESS;
		}
params={"username","age!=0"}代表请求参数中**必须**有username和age这两个参数，同时age不能等于0.

4. **headers**
headers属性同params类似，都是可以以数组的形式来进行约束。需要什么样的请求参数可以通过开发者工具来查看请求中都有什么样的参数。
