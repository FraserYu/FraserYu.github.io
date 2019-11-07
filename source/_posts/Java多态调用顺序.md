---
title: Java多态调用顺序
tags:
  - 多态
categories: [Coding, Java]
id: java-polymorphism-order
date: 2017-05-05 20:21:20
description: Java多态调用顺序透彻了解
keywords: Java多态,调用顺序,继承,构造函数

---

### Java多态调用该案例
大家先看自行测试如下程序的输出结果是什么？

```java
	public class A {  
	    public String show(D obj) {  
	        return ("A and D");  
	    }  

	    public String show(A obj) {  
	        return ("A and A");  
	    }   

	}  

	public class B extends A{  
	    public String show(B obj){  
	        return ("B and B");  
	    }  

	    public String show(A obj){  
	        return ("B and A");  
	    }   
	}  

	public class C extends B{  

	}  

	public class D extends B{  

	}  

	public class Test {  
	    public static void main(String[] args) {  
	        A a1 = new A();  
	        A a2 = new B();  
	        B b = new B();  
	        C c = new C();  
	        D d = new D();  

	        System.out.println("1--" + a1.show(b));  
	        System.out.println("2--" + a1.show(c));  
	        System.out.println("3--" + a1.show(d));  
	        System.out.println("4--" + a2.show(b));  
	        System.out.println("5--" + a2.show(c));  
	        System.out.println("6--" + a2.show(d));  
	        System.out.println("7--" + b.show(b));  
	        System.out.println("8--" + b.show(c));  
	        System.out.println("9--" + b.show(d));        
	    }  
	}  
```

<!-- more -->

仔细思考哦................
来看答案，有没有意外发生？

	1--A and A  
	2--A and A  
	3--A and D  
	4--B and A  
	5--B and A  
	6--A and D  
	7--B and B  
	8--B and B  
	9--A and D  

接下来主要针对第四个输出结果做出解释，其他的输出结果按照此规矩自然就能得出，请记住：
> **当超类对象引用变量引用子类对象时，被引用对象的类型而不是引用变量的类型决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法，但是它仍然要根据继承链中方法调用的优先级来确认方法，该优先级为：this.show(O)、super.show(O)、this.show((super)O)、super.show((super)O)**

所以，这里a2是引用变量，为A类型，它引用的是B对象，因此按照上面那句话的意思是说有B来决定调用谁的方法,所以a2.show(b)应该要调用B中的show(B obj)，产生的结果应该是“B and B”，但是为什么会与前面的运行结果产生差异呢？这里我们忽略了后面那句话“但是这儿被调用的方法必须是在超类中定义过的”，那么show(B obj)在A类中存在吗？根本就不存在！所以这句话在这里不适用？那么难道是这句话错误了？非也！其实这句话还隐含这这句话：它仍然要按照继承链中调用方法的优先级来确认。所以它才会在A类中找到show(A obj)，同时由于B重写了该方法所以才会调用B类中的方法，否则就会调用A类中的方法。

### 感谢
感谢[作者原文](http://blog.csdn.net/chenssy/article/details/12786385#)
