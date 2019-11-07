---
title: Spring MVC 之 @initbinder
tags:
  - SpringMVC
categories: [Coding, Spring-Boot]
id: springmvc-init-binder
date: 2017-03-28 16:24:26
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgspringbootannotation.png
---

### 背景
在Web项目中，太多需要提交表单或者在请求URL中添加参数信息的操作，无论是前者还是后者，SpringMVC都将每个元素当做String来处理， 如果前台传入格式化为字符串的日期或这数值类型的时候就会报错（在SpringMVC中，bean中定义了Date，double等类型，如果没有做任何处理的话，日期以及double都无法绑定），我们手动来强制转型很是麻烦，SpringMVC 提供的@initbinder 就是解决这个问题

### @initbinder
在我的项目中是在BaseController中增加方法initBinder，并使用注解@InitBinder标注，那么spring mvc在绑定表单之前，都会先注册这些编辑器, Spring自己提供了大量的实现类，诸如CustomDateEditor ，CustomBooleanEditor，CustomNumberEditor等许多，基本上够用, 当然我们也可以自己来写这些编辑器.


<!-- more -->

#### 自定义编辑器

	import org.springframework.beans.propertyeditors.PropertiesEditor;  

	public class DoubleEditor extends PropertiesEditor {    
	    @Override    
	    public void setAsText(String text) throws IllegalArgumentException {    
	        if (text == null || text.equals("")) {    
	            text = "0";    
	        }    
	        setValue(Double.parseDouble(text));    
	    }    

	    @Override    
	    public String getAsText() {    
	        return getValue().toString();    
	    }    
	}  

类似Long、Float等都可以这样写，然后将这些自定义的编辑器或Spring自带的编辑器放到BaseController中带有@initbinder的方法中注册就好.

	   @InitBinder    
	   protected void initBinder(WebDataBinder binder) {    
	       binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"), true));    
         binder.registerCustomEditor(int.class, new CustomNumberEditor(int.class, true));    
	       binder.registerCustomEditor(int.class, new IntegerEditor());    
	       binder.registerCustomEditor(long.class, new CustomNumberEditor(long.class, true));  
	       binder.registerCustomEditor(long.class, new LongEditor());    
	       binder.registerCustomEditor(double.class, new DoubleEditor());    
	       binder.registerCustomEditor(float.class, new FloatEditor());    
	   }  

当然我们也可以将自定义的编辑器直接继承 **PropertyEditorSupport**， 因为：

	public class org.springframework.beans.propertyeditors.PropertiesEditor extends java.beans.PropertyEditorSupport {  	

### 参考
非常感谢一下两位作者贡献整理的文章：
1. [spring mvc使用@InitBinder 标签对表单数据绑定](http://blog.csdn.net/axin66ok/article/details/17938095#)
2. [@InitBind来解决字符串转日期类型](http://www.cnblogs.com/shenxiaoquan/p/5753351.html)
