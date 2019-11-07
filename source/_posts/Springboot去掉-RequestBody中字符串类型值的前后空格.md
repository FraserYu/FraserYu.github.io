---
title: Springboot去掉@RequestBody中字符串类型值的前后空格
tags:
  - Springboot
  - Json
categories: [Coding, Spring-Boot]
id: spring-boot-request-body-space
date: 2018-07-12 16:06:17
---


<blockquote class="blockquote-center">改造MappingJackson2HttpMessageConverter</blockquote>

### 前言
项目组提出要求，Springboot项目中，@ResquestBody标记的bean传入的字符串类型的值要去掉前后空格

<!-- more -->


### 实现
因为项目默认使用 **MappingJackson2HttpMessageConverter** 作为 Json转换器， 于是乎重写一些方法做文章，具体实现如下：

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

             /**
             * 重写read方法，然后做去掉前后空格处理，重新转换成object
             * @param type
             * @param contextClass
             * @param inputMessage
             * @return
             * @throws IOException
             * @throws HttpMessageNotReadableException
             */
             @Override
             public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
                Object object = super.read(type, contextClass, inputMessage);
                String json = JSONObject.toJSONString(object);
                Map<String,Object> map = (Map<String, Object>) JSONObject.parse(json);

                Set<Map.Entry<String, Object>> entrySet = map.entrySet();

                Map<String, Object> removeSpaceMap = Maps.newHashMap();
                for (Map.Entry<String, Object> entry : entrySet){
                    if (entry.getValue() instanceof String){
                        removeSpaceMap.put(entry.getKey(), ((String) entry.getValue()).trim());
                    }else {
                        removeSpaceMap.put(entry.getKey(),entry.getValue());
                    }
                }
                Gson gson = new Gson();

                String removeSpaceJson = JSON.toJSONString(removeSpaceMap);
                return gson.fromJson(removeSpaceJson, type);
            }
         });
    }
  }


```

但是上面的代码经过各种测试发现一个问题，Gson在处理Integer，Long非浮点型的数值类型时候，自动转换为Double来处理，这导致后续代码做一些数据转换的时候会出现问题，因为Gson内部在处理数据类型都当成Number类型，可以读一下源码，这样需要做如下的改动：

```Java
/**
 *
 * @param type
 * @param contextClass
 * @param inputMessage
 * @return
 * @throws IOException
 * @throws HttpMessageNotReadableException
 */
@Override
public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
    Object object = super.read(type, contextClass, inputMessage);
    String json = JSONObject.toJSONString(object);
    Map<String,Object> map = (Map<String, Object>) JSONObject.parse(json);

    Set<Map.Entry<String, Object>> entrySet = map.entrySet();

    Map<String, Object> removeSpaceMap = Maps.newHashMap();
    for (Map.Entry<String, Object> entry : entrySet){
        if (entry.getValue() instanceof String){
            removeSpaceMap.put(entry.getKey(), ((String) entry.getValue()).trim());
        }else {
            removeSpaceMap.put(entry.getKey(),entry.getValue());
        }
    }
    String removeSpaceJson = JSON.toJSONString(removeSpaceMap);
    return JSONObject.parseObject(removeSpaceJson, type);

    //注释掉Gson，转换默认将Integer，Long等转换成Double，内部处理看作Number
    /*Gson gson = new Gson();
    return gson.fromJson(removeSpaceJson, type);*/
}

```

以为这样就结束了吗？哈哈，没有... 😒

以上我们传入的都是正常的JsonObject，比如这种格式的数据

```Json
{
    "commonName":"test2",
    "ver":1,
    "certificateRootType":"Z200",
    "expiredDate":"2118-01-01",
    "certificateName":"税务登记证 。",
    "commonCode":"suibian",
    "updateTime":"2018-07-12 13:09:14",
    "perpetual":2,
    "certificateNo":"11111111",
    "certificateTypeName":"税务登记证",
    "imgInfoVOList":[
        {
            "url":"https://www.a.b.c/a.jpg"
        },
        {
            "url":"https://www.a.b.c/b.jpg"
        }
    ],
    "producerType":1,
    "createTime":"2018-07-12 13:09:14",
    "supplier":1043,
    "id":5016,
    "startDate":"2018-01-01",
    "certificateType":"Z202"
}
```

但是要有这种格式的数据


```Json
[
    {
        "roleId":212,
        "name":"上帝视角"
    },
    {
        "roleId":212,
        "name":"上帝视角"
    },
    {
        "roleId":212,
        "name":"上帝视角"
    }
]
```

如果是下面这种格式的，我们解析的时候就会报转换错误，所以针对下面JsonArray类型的数据，我们需要做特殊处理：

```Java
/**
 *
 * @param type
 * @param contextClass
 * @param inputMessage
 * @return
 * @throws IOException
 * @throws HttpMessageNotReadableException
 */
@SuppressWarnings("unchecked")
@Override
public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
    Object object = super.read(type, contextClass, inputMessage);
    String json = JSONObject.toJSONString(object);
    String jsonResult = null;
    if (json.startsWith("[")){
        JSONArray jsonArray = JSONArray.parseArray(json);
        if (jsonArray != null && !jsonArray.isEmpty()){
            List<Map<String, Object>> result = Lists.newArrayList();
            for (int i=0; i<jsonArray.size(); i++){
                Map<String, Object> map = (Map<String, Object>) jsonArray.get(i);
                Map<String, Object> removeSpaceMap = removeSpace(map);
                result.add(removeSpaceMap);
            }
         jsonResult = JSON.toJSONString(result);
        }
    }else {
        Map<String,Object> map = (Map<String, Object>) JSONObject.parse(json);

        Map<String, Object> removeSpaceMap = removeSpace(map);
        jsonResult = JSON.toJSONString(removeSpaceMap);
    }


    //注释掉Gson，转换默认将Integer，Long等转换成Double，内部处理看作Number
    /*Gson gson = new Gson();
    return gson.fromJson(removeSpaceJson, type);*/
    return JSONObject.parseObject(jsonResult, type);
}


private Map<String, Object> removeSpace(Map<String, Object> map){
  Set<Map.Entry<String, Object>> entrySet = map.entrySet();

  Map<String, Object> removeSpaceMap = Maps.newHashMap();
  for (Map.Entry<String, Object> entry : entrySet){
      if (entry.getValue() instanceof String){
          removeSpaceMap.put(entry.getKey(), ((String) entry.getValue()).trim());
      }else {
          removeSpaceMap.put(entry.getKey(),entry.getValue());
      }
  }
  return removeSpaceMap;
}

```
这样就应对大多数问题了

<img itemprop="url image" src="/uploads/spring/httpmessageconverter.png" class="full-image" />

接下来需要具体了解一下，构建过程，如上图，在HttpInputMessage 通过 converter 转换成Object对象之前做处理，这样在Controller接收的时候就可以拿到处理后的对象了
