---
title: Spring AOP
tags:
  - Spring
  - AOP
categories: [Coding, Spring]
id: spring-aop
date: 2017-02-23 20:14:54
---

### 前言
Spring AOP 的道理懂得，但是很少在项目中使用，刚巧这个项目中有用到，将AOP的理解以及使用中遇到的问题加以总结记录

### 使用场景
设想一下，项目已接近尾声，经理说为了项目验收，更好的监控程序调用日志，需要在每个方法调用前或调用后加上日志，天啊，这是多大的工作量，项目中至少有几百个方法，我们要做这种无脑的工作了吗？ 如果不懂得Spring AOP面向切面编程的道理，那这个体力活看来是费力不讨好的事情，接下来让我们慢慢剖析Spring AOP 的道理.
### Java 动态代理
Java23中设计模式之一代理模式，Spring AOP充分利用了Java 动态代理功能来实现（对动态代理有所理解的可以跳过该段内容），即便没有Spring AOP 我们也能够通过Java  动态代理解决上述问题.  在Java 动态代理中有两个重要的接口或类：
>	1. **InvocationHandler** （Interface）
>	2. **Proxy** （Class）

<!-- more -->

这一个接口和类是实现我们动态代理所必须用到的，其实这种设计模式可以联想到我们现实生活中的代购或代理商:
![enter image description here](http://7xkyc7.com1.z0.glb.clouddn.com/Spring_springaop.png)

这是什么，怎么好像和我们说的项目中需要不沾边际，项目快要结束了，满项目鞋子和衣服方法已经写好，突然项目经理说要给鞋子和衣服加上包装（日志），刚好代理商可以轻松做到这个，每次客户调用鞋子和衣服方法时，只需要调用代理商暴露出来的鞋子和方法即可, 还没懂？ 没关系，程序猿看代码还是最实际的.

#### InvocationHandler
看一下官方定义：
> *InvocationHandler is the interface implemented by the invocation handler of a proxy instance. Each proxy instance has an associated invocation handler. When a method is invoked on a proxy instance, the method invocation is encoded and dispatched to the invoke method of its invocation handler.* —— InvocationHandler 是一个接口，它被一个代理类实例的handler所实现，每一个代理类实例都有其对应的handler, 当一个代理实例的方法被调用，方法的调用就交给了它对应的handler来处理

InvocationHandler 接口只定义了一个方法：

	Object invoke(Object proxy, Method method, Object[] args) throws Throwable;

1. proxy : 代理类对象
2. method : 我们调用真实对象的某个方法
3. args : 我们调用真是对象的某个方法接受的参数

#### Proxy
看一下官方定义：
> Proxy provides static methods for creating dynamic proxy classes and instances, and it is also the superclass of all dynamic proxy classes created by those methods.  —— Proxy提供一个静态方法用来创建动态代理类实例，并且是所有代理类的父类

Proxy 类中最常用的方法是：

	/**
	*Returns an instance of a proxy class for the specified interfaces that dispatches method invocations to the specified invocation handler
	*/

	public static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h) {
        try {
            Class e = getProxyClass(loader, interfaces);
            return e.getConstructor(new Class[]{InvocationHandler.class}).newInstance(new Object[]{h});
        } catch (RuntimeException var4) {
            throw var4;
        } catch (Exception var5) {
            throw new CodeGenerationException(var5);
        }
    }

1. loader :  一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载
2. interfaces : 一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了
3. h : 一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上

### 代码示例
#### 定义Subject 接口：

	public interface Subject
	{
	    public void shoes();

	    public void clothes(String str);
	}

#### 定义接口真实实现类

	public class RealSubject implements Subject
	{
	    @Override
	    public void shoes()
	    {
	        System.out.println("This is shoes method without packaging");
	    }

	    @Override
	    public void clothes(String str)
	    {
	        System.out.println("hello: " + str);
	    }
	}

#### 定义动态代理类，并实现InvocationHandler 接口

	public class DynamicProxy implements InvocationHandler
	{
	    //这个就是我们要代理的真实对象
	    private Object subject;

	    //构造方法，给我们要代理的真实对象赋初值
	    public DynamicProxy(Object subject)
	    {
	        this.subject = subject;
	    }

	    @Override
	    public Object invoke(Object object, Method method, Object[] args)
	            throws Throwable
	    {
	        //在代理真实对象前我们可以添加一些自己的操作，比如给鞋子和衣服做包装
	        System.out.println("before packaging");

	        System.out.println("Method:" + method);

	        //当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
	        method.invoke(subject, args);

	        //在代理真实对象后我们也可以添加一些自己的操作，加完包装之后的一些售后服务
	        System.out.println("after packaging");

	        return null;
	    }

	}


#### 看看客户端类的定义

	public class Client
	{
	    public static void main(String[] args)
	    {
	        //我们要代理的真实对象
	        Subject realSubject = new RealSubject();

	        //我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
	        InvocationHandler handler = new DynamicProxy(realSubject);

	        /*
	         * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
	         * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
	         * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
	         * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
	         */
	        Subject subject = (Subject)Proxy.newProxyInstance(handler.getClass().getClassLoader(), realSubject
	                .getClass().getInterfaces(), handler);

	        System.out.println(subject.getClass().getName());
	        subject.shoes();
	        subject.clothes("clothes, it's without packaging");
	    }
	}

#### 控制台的输出结果

	$Proxy0

	before packaging
	Method:public abstract void com.xiaoluo.dynamicproxy.Subject.shoes()
	This is shoes method without packaging
	after packaging

	before packaging
	Method:public abstract void com.xiaoluo.dynamicproxy.Subject.clothes(java.lang.String)
	hello: clothes, it's without packaging
	after packaging

Proxy.newProxyInstance 生成代理对象时所需要的invocationHandler 参数更像是代理商和供应商之间的物流，当代理商需要供应商的鞋子时候，需要告诉这个物流去取鞋子.  一个代理商当然也代理多个品牌，这就是为什么参数中interfaces 是数组形式，至此关于Java 动态代理就告一段落了，我们来看看Spring AOP , 道理都是相通的.

### Spring AOP
#### 术语解释

| 术语名称      |     对应解释 |   官方术语描述   |
| :-------- | :--------| :------ |
| Aspect    |   它是一个包含一系列API的模块，同时提供横切的需求，比如日志记录.|  A module which has a set of APIs providing cross-cutting requirements. For example, a logging module would be called AOP aspect for logging. An application can have any number of aspects depending on the requirement.  |
| Join point    |  应用程序真正会发生通知行为的一个点 |  This represents a point in your application where you can plug-in AOP aspect. You can also say, it is the actual place in the application where an action will be taken using Spring AOP framework. |
| Advice    |   可以理解为通知，在目标程序执行前通知还是执行后通知|  This is the actual action to be taken either before or after the method execution. This is actual piece of code that is invoked during program execution by Spring AOP framework.  |
| Pointcut    |   Join Point的集合，程序中所有想切入的点的集合 |  This is a set of one or more joinpoints where an advice should be executed. You can specify pointcuts using expressions or patterns as we will see in our AOP examples.  |

#### Advice 通知

| 术语名称     |    对应解释  |   官方术语描述  |
| :-------- | :--------| :------ |
| before   |   在调用目标方法之前通知 |  Run advice before the a method execution  |
| after|   在调用目标方法之后通知，不管目标方法是否执行成功 |  Run advice after the a method execution regardless of its outcome.  |
| after-returning   |   当目标方法成功执行没有任何异常之后 |  Run advice after the a method execution only if method completes successfully  |
| after-throwing   |   after 或 after-returning 运行之后捕获异常 |  Run advice after the a method execution only if method exits by throwing an exception.  |
| around   |   运行目标方法前后环绕 |  Run advice before and after the advised method is invoked  |


	 try{
	    //前置通知
	         //环绕通知
	            //调用目标对象方法
	         //环绕通知
	    //后置通知
	  }catch(){
	    //异常通知
	  }finally{
	    //终止通知
	  }


#### Spring XML配置

	<context:annotation-config/>
		<aop:config proxy-target-class="true" expose-proxy="true">
			<aop:aspect ref="capitalModification">
				<aop:pointcut id="handleOrder" expression="execution(* com.honor.facades.app.order.HonorOrderFacade.handleCapital(..))
					|| execution(* com.honor.facades.serviceOrder.HonorServiceOrderFacade.handleCapital(..))
					|| execution(* com.honor.facades.aftersales.HonorReturnFormFacades.handleCapital(..))"/>
				<aop:after method="capitalAndRecordModification"  pointcut-ref="handleOrder"/>
				<aop:after-throwing throwing="ex" method="capitalAndRecordModificationExceptionHandle"  pointcut-ref="handleOrder"/>
			</aop:aspect>
		</aop:config>

	<bean id="capitalModification" class="com.honor.core.customerCapital.impl.DefaultCapitalModificationStrategy">
			<property name="customerCapitalService" ref="customerCapitalService"/>
			<property name="honorSuperOrderService" ref="honorSuperOrderService"/>
			<property name="honorBillService" ref="honorBillService"/>
		</bean>

多个pointcut 可以用 || 或 or 来实现.

### 项目使用中遇到的问题
关于AOP无法切入同类调用方法的问题
Controller 层方法A调用了Facade层方法B，可是B内调用了方法C, 我们想使用AOP织入方法C, 当程序运行后， 始终找不到代理方法，意味着织入C没有成功,

	public class MyController{

		public void methodA(){
			myFacade.methodB();
		}
	}

	public class MyController{

		public void methodB(){
			myFacade.methodC();
		}

		public void methodC(){
			System.out.println("bingo");
		}
	}

	public class Aspect {  

		 @AfterReturning(execution(* Facade.methodB(..)))  
		 public void after() {  
			 Logger.info(after call and do something);  
		}  
	}  

我们发现在 AopContext.class 中：

	public static Object currentProxy() throws IllegalStateException {
	        Object proxy = currentProxy.get();
	        if(proxy == null) {
	            throw new IllegalStateException("Cannot find current proxy: Set \'exposeProxy\' property on Advised to \'true\' to make it available.");
	        } else {
	            return proxy;
	        }
	    }

解决办法：
1. 设置proxy-target-class="true"expose-proxy="true"

		<aop:aspectj-autoproxy proxy-target-class="true"expose-proxy="true"/>  

2. 手动指定代理调用方法

		((Facade) AopContext.currentProxy()).methodB();  



### 参考链接
非常感谢以下几位博主，在你们认真的博客上学到了许多.

1. http://blog.csdn.net/lirui0822/article/details/8555691
2. http://www.cnblogs.com/xiaoluo501395377/p/3383130.html

同时修改AOP函数返回值及获取函数参数请参考以下文章：
1. http://blog.csdn.net/cleverutd/article/details/8607491
2. https://my.oschina.net/itblog/blog/211693
