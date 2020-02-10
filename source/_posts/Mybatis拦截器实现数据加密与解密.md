---
title: Mybatis拦截器实现数据加密与解密
tags:
  - MyBatis
categories: [Coding, 数据库-持久层-SQL]
id: mybatis-interceptor-encrypt-decrypt
date: 2019-09-02 08:40:51
description: Mybatis拦截器实现数据加密与解密，全局处理需要在数据库中敏感数据
keywords: Mybatis,拦截器,Interceptor,加密,解密,全局处理,敏感数据
---


## 拦截器介绍
`Mybatis Interceptor`  在 Mybatis 中被当作 Plugin(插件)，不知道为什么，但确实是在 `org.apache.ibatis.plugin` 包下面

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/_image/2019-06-04/sunset-3689760.jpg)


既然是拦截器，可以拦截哪些内容呢？试想一下...... 当程序写到持久层时，Mybatis 会 **执行** 指定 **SQL 语句**，并处理 **请求参数** 和 **返回值**。没错，Mybatis 拦截器可以帮助我们处理上述内容，请看[官网](http://www.mybatis.org/mybatis-3/zh/configuration.html#plugins)的 Plugins 的片段, 内容不多
```java
// 执行
Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
// 请求参数处理
ParameterHandler (getParameterObject, setParameters)
// 返回结果集处理
ResultSetHandler (handleResultSets, handleOutputParameters)
// SQL语句构建
StatementHandler (prepare, parameterize, batch, update, query)
```

## 拦截器的使用
如果需要实现自定义的拦截器，只需要实现 `org.apache.ibatis.plugin.Interceptor` 接口，该接口有三个方法：
```java
Object intercept(Invocation invocation) throws Throwable;

Object plugin(Object target);

void setProperties(Properties properties);
```

我们要实现数据加密，进入数据库的字段不能是真实的数据，但是返回来的数据要真实可用，所以我们需要针对 Parameter 和 ResultSet 两种类型处理，同时为了更灵活的使用，我们需要自定义注解

### 自定义注解
类注解，将注解放在实体类上
```java
/**
 * 需要加解密的类注解
 */
@Documented
@Inherited
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface EncryptDecryptClass {
}
```
字段注解，将注解放在实体字段上
```java
/**
 * 加密字段注解
 */
@Documented
@Inherited
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface EncryptDecryptField {

}
```
有了这两个注解，我们可以在我们可以标记我们要处理的实体和实体中的字段

### 自定义参数处理拦截器
参考官网，通过 `@Intercepts` 和 `@Signature` 的联合使用，指定 `ParameterHandler.class` 类型，同时通过 `@Component` 注解注入到容器中，即可在设置参数的时候进行拦截，通过自定义接口 `IEncryptDecrypt`, 根据 Field 的各种类型自定义加密解密算法
```java
@Intercepts({
		@Signature(type = ParameterHandler.class, method = "setParameters", args = PreparedStatement.class),
})
@ConditionalOnProperty(value = "domain.encrypt", havingValue = "true")
@Component
@Slf4j
public class ParammeterInterceptor  implements Interceptor {

	@Autowired
	private IEncryptDecrypt encryptDecrypt;

	@Override
	public Object intercept(Invocation invocation) throws Throwable {

		log.info("拦截器ParamInterceptor");
		//拦截 ParameterHandler 的 setParameters 方法 动态设置参数
		if (invocation.getTarget() instanceof ParameterHandler) {
			ParameterHandler parameterHandler = (ParameterHandler) invocation.getTarget();
			PreparedStatement ps = (PreparedStatement) invocation.getArgs()[0];

			// 反射获取 BoundSql 对象，此对象包含生成的sql和sql的参数map映射
			/*Field boundSqlField = parameterHandler.getClass().getDeclaredField("boundSql");
			boundSqlField.setAccessible(true);
			BoundSql boundSql = (BoundSql) boundSqlField.get(parameterHandler);*/

			// 反射获取 参数对像
			Field parameterField =
					parameterHandler.getClass().getDeclaredField("parameterObject");
			parameterField.setAccessible(true);
			Object parameterObject = parameterField.get(parameterHandler);
			if (Objects.nonNull(parameterObject)){
				Class<?> parameterObjectClass = parameterObject.getClass();
				EncryptDecryptClass encryptDecryptClass = AnnotationUtils.findAnnotation(parameterObjectClass, EncryptDecryptClass.class);
				if (Objects.nonNull(encryptDecryptClass)){
					Field[] declaredFields = parameterObjectClass.getDeclaredFields();

					final Object encrypt = encryptDecrypt.encrypt(declaredFields, parameterObject);
				}
			}
		}
		return invocation.proceed();
	}

	@Override
	public Object plugin(Object o) {
		return Plugin.wrap(o, this);
	}

	@Override
	public void setProperties(Properties properties) {

	}
}
```
同样新建结果集拦截器
### 结果集拦截器
与参数拦截器基本一样, 只不过类型指定为 `ResultSetHandler.class`
```java
@Intercepts({
		@Signature(type = ResultSetHandler.class, method = "handleResultSets", args={Statement.class})
})
@ConditionalOnProperty(value = "domain.decrypt", havingValue = "true")
@Component
@Slf4j
public class ResultInterceptor implements Interceptor {

	@Autowired
	private IEncryptDecrypt encryptDecrypt;

	@Override
	public Object intercept(Invocation invocation) throws Throwable {
		Object result = invocation.proceed();
		if (Objects.isNull(result)){
			return null;
		}

		if (result instanceof ArrayList) {
			ArrayList resultList = (ArrayList) result;
			if (CollectionUtils.isNotEmpty(resultList) && needToDecrypt(resultList.get(0))){
				for (int i = 0; i < resultList.size(); i++) {
					encryptDecrypt.decrypt(resultList.get(i));
				}
			}
		}else {
			if (needToDecrypt(result)){
				encryptDecrypt.decrypt(result);
			}
		}
		return result;
	}

	public boolean needToDecrypt(Object object){
		Class<?> objectClass = object.getClass();
		EncryptDecryptClass encryptDecryptClass = AnnotationUtils.findAnnotation(objectClass, EncryptDecryptClass.class);
		if (Objects.nonNull(encryptDecryptClass)){
			return true;
		}
		return false;
	}

	@Override
	public Object plugin(Object target) {
		return Plugin.wrap(target, this);
	}

	@Override
	public void setProperties(Properties properties) {

	}
}
```
### 加密解密
`IEncryptDecrypt` 接口定义了 加密和解密两个方法，
```java
public interface IEncryptDecrypt {

	/**
	 * 加密方法
	 * @param declaredFields 反射bean成员变量
	 * @param parameterObject Mybatis入参
	 * @param <T>
	 * @return
	 */
	public <T> T encrypt(Field[] declaredFields, T parameterObject) throws IllegalAccessException;


	/**
	 * 解密方法
	 * @param result Mybatis 返回值，需要判断是否是ArrayList类型
	 * @param <T>
	 * @return
	 */
	public <T> T decrypt(T result) throws IllegalAccessException;
}
```

两个拦截器通过在 YAML 中配置属性，按条件注入，外加自定义加密解密算法，完成全局灵活的配置。
核心代码已上传至 [Github Demo](https://github.com/FraserYu/learnings/tree/master/mybatis-interceptor/interceptor)

## 问题彩蛋
也许应对当前的业务，看了该文章满足了当下需求，我们目前只看到了什么是 Mybatis 拦截器，怎样简单使用，拦截器的其他用法以及其他很多为什么都没有解决，关注公众号，回复“人迹罕至” 读完文章 「程序猿为什么要看源码」后 ，我不会满足眼前的这些基本应用，我会有诸多疑问，

1. 我们日常写 CRUD 的业务，为什么 Executor 中只有 R(query) 和 U(update), 那么C(insert) 和 D(delete) 怎样处理的？
2. 自定义拦截器是以什么方式被执行的，执行顺序是什么？
3. 分页也是 Mybatis 拦截器的一种，带有分页的框架是怎样使用拦截器的呢？如 `Mybatis Plus`, `PageHelper`
4. 虽然重写了 Inteceptor 接口的 `public void setProperties(Properties properties)` 方法，但是并没有写什么业务逻辑，这个方法能怎样使用？
5. ......
后续文章也会通过读源码的方式逐步解析这些问题，当然你有相关问题也可以留言交流讨论

## 提高效率工具
依旧推荐在写文章时用到的高效工具，后续相关工具也会在文章中陆续更新，请持续关注
### MyBatis Log Plugin
MyBatis Log Plugin 是 Intelligj IDEA 的一个插件，用来从 Mybatis 输出的 log 中提取出当前调用的 SQL 语句，并将参数封装在 SQL 语句中组成完整的 SQL，这样，当我们调试的时候更加清晰方便，可以轻松定位是否 SQL 又问题

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/_image/2019-06-04/MyBatisLogPlugin.gif)


## 推荐阅读
+ [每天用SpringBoot，还不懂RESTful API返回统一数据格式是怎么实现的？ ](https://mp.weixin.qq.com/s/q5cyK14WGiMm-R6NIFBcTg)
+ [双亲委派模型：大厂高频面试题，轻松搞定](https://mp.weixin.qq.com/s/Dnr1jLebvBUHnziZzSfcrA)
+ [面试还不知道BeanFactory和ApplicationContext的区别？](https://mp.weixin.qq.com/s/YBQB086ADBjHUmwrFQrWew)
+ [如何设计好的RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ)
+ [红黑树，超强动静图详解，简单易懂](https://mp.weixin.qq.com/s/ilND8u_8HGSTSrJiMB4X8g)

--------
> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享
> - 高效工具汇总 | 回复「工具」
> - 面试问题分析与解答
> - 技术资料领取 | 回复「资料」

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)
