---
title: Springboot Json格式数据不返回null值属性
tags:
  - Spring Boot
  - Json
categories: [Coding, Spring-Boot]
id: spring-boot-json-return-non-null
date: 2018-07-02 13:41:37
description: Springboot Json格式数据不返回null值属性, 让数据更清晰
keywords: SpringBoot,null,null值过滤,MessageConverter
---

<blockquote class="blockquote-center">Spring ResponseBody without null field</blockquote>

> 通常Restful返回的Json格式数据，里面包含很多为空值null的字段，我们没必要将这些字段返回给前端，应该给出更多的有效字段，所以我们需要在Converter Message的时候将这些null值字段给过滤掉

#### 方法一
添加 **@JsonInclude(JsonInclude.Include.NON_NULL)** 注解
该方法很简单，但是我们有太多的返回类型，一一添加也是很大的工作量，所以我们需要在转换的源头进行控制

<!-- more -->


#### 方法二
实现 **WebMvcConfigurer** 接口，添加 **@EnableWebMvc** 和 **@Configuration** 注解，然后重写 **MappingJackson2HttpMessageConverter 中 getObjectMapper() 方法**

```java
  @EnableWebMvc
  @Configuration
  public class WebConfig implements WebMvcConfigurer{
    //... 省略重写的其他方法

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
     super.configureMessageConverters(converters);
     //对应无法直接返回String类型
     converters.add(0, new MappingJackson2HttpMessageConverter(){
             @Override
             public ObjectMapper getObjectMapper() {
                 super.getObjectMapper().setSerializationInclusion(JsonInclude.Include.NON_NULL);
                 return super.getObjectMapper();
             }
         });
    }
  }


```
这样所有的null值都会被过滤掉，注意这里使用的是 **JsonInclude.Include.NON_NULL**， Json中的某个集合如果为空还是会返回空数组，我们希望把这样的数据也过滤掉，只需要将**JsonInclude.Include.NON_NULL** 修改为 **JsonInclude.Include.NON_EMPTY**

这样就好了，返回给前端清晰有效的数据.
