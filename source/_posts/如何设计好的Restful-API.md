---
title: 如何设计好的Restful API
categories: [Coding, 安全-协议-规范]
tags:
  - RESTful
  - HTTP
id: restful-api-design
date: 2019-05-16 14:48:27
icons: [fas fa-star yellow]
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgjava.png
description: 微服务与前后端分离的当下，如何设计好的 RESTful API至关重要
keywords: RESTful,RESTful API,JSON,Http
---



## 现状

现阶段的开发模式多以前后端分离形式存在，前后端开发人员需要通过大量 API 来进行数据交互，如果在交互过程中前后端人员经常遭遇如下问题：

+ 前端人员不能快速理解接口字段含义及接口字段变化
+ 后端人员想复用某些接口，但是不能快速从接口 URL 的定义中明确该接口的含义，需要进一步读代码确认
+ URL中的英文单词使用五花八门，搜索某个接口不知道具体的关键字
+ 请求方法动词如 POST GET 随意使用
+ 完成当前业务接口对接，前端人员经常会询问下一步业务流程的接口定义在哪里，对接形式是什么样的

以上只是前后端人员通过接口交互的一小部分问题，这些问题就好比"牙痛"，不致命，但是在整个软件开发的生命周期内，天天"牙痛”是很要命的, 需要解决上述的问题，需要前后端人员都能认识与了解接口设计规范的重要性。

## 什么是REST

在 2000 年，Roy Fielding 提出 [Representational State Transfer (REST)](https://zjcqoo.github.io/-----https://en.wikipedia.org/wiki/Representational_state_transfer) 的概念，中文翻译过来"表述性状态传递"，感兴趣的朋友可以去维基百科看看原始概念，乍一看是一个挺抽象的概念，但其实，这个概念就像交通灯规则一样简单，就看如何看待相关规范. 当我们谈及 RESTful 设计规范，多数人能了解设计的大原则，但是不了解小细节，而对这些细节的了解与否，是能否治好"牙痛病"的关键

## REST术语介绍

现实世界交通灯有红绿黄，REST相关的概念也是三个：资源，集合，URL

### 资源

资源是某种东西的对象或表示，它具有一些与之相关的数据，并且可以有一组方法对其进行操作。 例如, 动物，学校和员工是资源;  删除，添加，更新是对这些资源执行的相关操作

### 集合

集合是资源集合，例如，公司是公司资源的集合

### URL

URL（统一资源定位符）是可以通过其定位资源的路径，并且可以对其执行某些操作

了解到以上内容， 那REST 世界的"交通灯"规则是什么样的？我们来了解一下

## 如何设计和开发一个高可用的 REST APIs

> 网上一直有关于"最好的Restful API的设计"争论，何为最好，至今没有一个官方的指导。本文总结 RESTful 的设计细节，介绍如何设计出易于理解和使用的 API。在 Restful API 设计标准之上，我们可以为我们的设计增加一些弹性（团队都认可的方式），每个项目的情况不同，最重要的是项目组成员达成一致的Restful API 设计规则，达到高可用即可

### URL 设计

*学英语，名词（car/animal/teacher）都很好记忆，但是如何用动词和这些名词组合来准确的表达特定的含义却很困难，庆幸的是在 REST 的世界，动词寥寥无几，并且含义单一* ，RESTful 的核心思想也是通过这些**动词 + 名词**完成对资源的操作与访问，但我们经常看到这样的动词与名词的 URL 组合：

+ /getAllUsers
+ /createNewCompany
+ /updateUserInfo
+ /deleteUser?name=zhangsan

这些 URL 的设计会导致文章开头所说的很多问题，我们进一步来了解如何应用所谓的**动词 + 名词**

#### 动词

动词通常就是 5 种 HTTP 方法，对应我们常见的 CRUD 操作：

+ POST：新建（Create）
+ GET：读取（Read）
+ PUT：更新（Update）
+ PATCH：更新（Update），通常不分更新，也很少用到
+ DELETE：删除（Delete）

根据 HTTP 规范，动词一律大写，另外根据RESTful 幂等性（多次调用是否会对资源产生影响）原则，我们不能乱用动词，GET/PUT/DELETE 是幂等的，POST/PATCH 不是幂等的

有些客户端只能使用`GET`和`POST`这两种方法。服务器必须接受`POST`模拟其他三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上`X-HTTP-Method-Override`属性，告诉服务器应该使用哪一个动词，覆盖`POST`方法。

```http
POST /users/12 HTTP/1.1
X-HTTP-Method-Override: PUT
```

上面代码中，`X-HTTP-Method-Override`指定本次请求的方法是`PUT`，而不是`POST`

#### 名词

名词就是表示一个资源或者服务，如 /users，/teachers，这里看到我用名词复数的形式描述某一资源，至于用单数还是复数每个人都有自己的见解，我在这里推荐使用复数，因为在现实世界中，资源多数是以集合的形式存在的

#### 动词 + 名词

| 资源      | POST（Create）      | GET（Read）  | PUT（Update） | DELETE （Delete） |
| --------- | ------------------- | ------------ | ------------- | ----------------- |
| /users    | 创建新用户          | 查询所有用户 | 批量更新用户  | 删除所有用户      |
| /users/12 | 方法不被允许（405） | 查询指定用户 | 更新指定用户  | 删除指定用户      |

```http
POST /users
GET /users
PUT /users
DELETE /users
GET /users/12
PUT /users/12
DELETE /users/12
```

上述**动词 + 名词**的组合是不是清晰多了，没有杂乱的动词在 URL 中，大家的理解含义相同

#### URL 层级

现实中哪有这么简单的 CRUD，资源的相互关联与嵌套很常见，*查找 id 是 12 的用户的所有帖子*， 如何设计这个 URL，下面两种设计也会有争论：

```http
GET /users/12/posts
GET /posts?userId=12
```

第一种出现两个名词主题（users/posts），会让人有几秒钟的猜想，这到底请求的是用户资源还是帖子资源，当存在更深浅套的时候也不容易扩展，所以我推荐第二种方式，主体名词 posts 资源明显，其他过滤条件也更容易扩展，比如 /posts?userName=zhangsan，我们可以复用同样的接口

#### 版本

我们看到过很多如下 URL 设计，用来区分 API 版本：

```http
POST /v2/users
GET /V1/users/12
```

我们都指向同样的资源 users，URL 中为什么要加版本号呢？ 针对这个问题，答案依旧没有统一标准，如果多个版本的API版本返回数据结果结构一样，那没必要区分版本，如果结构已经发生变化，而且要向下兼容，那版本号是很好的区分方式，而且通过 URL 加版本的方式可以更好的发现资源

#### 过滤/分页/排序

实际的业务场景中会经常对请求资源做条件筛选，分页显示，以及排序，我们不要为这些业务要求创建不同步的 API，我们应该尽量保持 URL 的信息简单，只需添加查询条件参数来实现上述功能，同时符合"望 URL 知意"的原则

##### 过滤

```http
GET /users/12/posts?state=published
GET /users/12/posts?published=true
```

上述两种方式都可以实现资源的过滤

##### 分页

```http
GET /users?pageNo=1&pageSize=20
```

以分页方式查询用户列表，显示第 2 页内容，每页显示 20 条信息

##### 排序

```http
GET /users?sort=score_desc
```

按照学生分数降序进行排序

上述所有的方式我们都可以做到"望 URL 知意"，这就是好的设计

### 返回结果

RESTful API 的返回结果也是设计环节中重要的一环

#### 响应数据格式

API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的`Content-Type`属性要设为`application/json`。同时客户端也应作出相应的配合，客户端请求时，也要明确告诉服务器，可以接受 JSON 格式，即请求的 HTTP 头的`ACCEPT`属性也要设成`application/json`，多渠道调用可能会存在相同资源需要有不同的 producer 类型的情况存在

#### 响应状态码

很多后端开发人员可能受开发框架所限，或者返回数据封装形式不够好，经常会给前端人员不是很友好的 HTTP 状态码，比如 response 有 error，却给出 `200 HTTP.OK` 的状态码 (明明吃了三碗粉，却给两碗粉的钱)

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status": "-1",
    "result": {
        "error": "分数应小于150"
    }
}
```

有人说，我已经标记返回内容 status 是 -1（表示failure）, 用 200 作为状态码也无妨吧？这是一个很错误的观念，RESTful 的设计理念之一是简单直观，试想一下，前端开发人员打开开发者工具，所有请求都是200的状态码，但是页面数据就是没有显示出来，难道前端开发人员还要每个接口调用点开看一看，是哪个 status 是 -1 导致的吗？ 很显然我们不希望这样的情况发生，正确的做法应该类似这样的：

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "status": "-1",
    "result": {
        "error": "分数应小于150"
    }
}
```

下面页列举出来常用的状态码以及表示的含义：

##### 2xx 状态码

   200表示成功，同时我们可以表示的更加精确

   > + `GET: 200 OK` 请求成功
   > + `POST: 201 Created` 创建成功
   > + `PUT: 200 OK` 更新成功
   > + `DELETE: 204 No Content` 找不到要删除的内容

   使用状态码 202 有时候会比 使用状态啊吗 201 是更好的选择，状态码 202 的意思是：服务端已接收到了请求，但是还没有创建任何资源，但结果一切正常。 比如：

   > + 异步操作：服务器已接收到请求，但是还未处理，但是会在未来处理
   > + 资源已经存在，没有创建新的资源 （有些业务可能会返回错误信息"您创建的数据已存在"，所以这种情景没有明确的规定，符合自己的业务需求即可）

##### 4xx 状态码

   4xx 状态码表示客户端的错误，主要有以下几种：

   > + `400 Bad Request`：服务器不理解客户端的请求，未做任何处理
   > + `401 Unauthorized`：用户未提供身份验证凭据，或者没有通过身份验证
   > + `403 Forbidden`：用户通过了身份验证，但是不具有访问资源所需的权限
   > + `404 Not Found`：所请求的资源不存在，或不可用
   > + `415 Unsupported Media Type`：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式

   这里要注意状态码 401 和 403 的区别

##### 5xx 状态码

   5xx 状态码表示服务端错误，通常只会用到两个：

   > + `500 Internal Server Error`：客户端请求有效，服务器处理时发生了意外
   > + `503 Service Unavailable`：服务器无法处理请求，一般用于网站维护状态

#### 无状态

过去开发人员通常会将活动的用户信息存储在服务端的 session 中， 这种形式很显然不适用于现在分布式微服务架构的模式，我们可以使用 JWT (JSON Web Token) 如 OAuth2 来实现，这样每次在 Httpheader 中添加 token 来做验证即可

#### API 文档

Swagger是一种广泛使用的工具来用来记录与呈现 REST API，它提供了一种探索特定 API 使用的方法，因此允许开发人员理解底层的语义行为。 这是一种使用注释添加文档的声明性方法，它进一步生成描述 API 及其用法的 JSON，可以实时应对 API 的更新，具体请参考 [Swagger 官网](https://swagger.io/) , 同时使用 Spring Boot 的小伙伴也可以很轻松的集成 Swagger，只需引入[Swagger Starter](https://github.com/SpringForAll/spring-boot-starter-swagger)

```xml
<dependency>
	<groupId>com.spring4all</groupId>
	<artifactId>swagger-spring-boot-starter</artifactId>
	<version>1.9.0.RELEASE</version>
</dependency>
```

#### HATEOAS

HATEOAS (Hypermedia As Transfer Engine Of Application State), API 的使用者未必知道，URL 是怎么设计的。一个解决方法就是，在回应中，给出相关链接，便于下一步操作。这样的话，用户只要记住一个 URL，就可以发现其他的 URL。这种方法叫做 HATEOAS , 举个例子，列表页数据通常会有查看操作，这样我们在返回列表页的数据的时候同样返回如何操作查看具体数据详情的 API 接口：

```json
{
    "status": "-1",
    "result": [{
        "id": 1,
      	"name": "zhangsan",
      	"links":[
          {
            "href": "http://localhost:8080/user/{id}"
          }
        ]
    }]
}
```

使用 Spring 框架的小伙伴可以快速的体验一下这种方式，Spring 官网项目 [Spring HATEOAS](https://spring.io/projects/spring-hateoas) , 会快速的将参数都做替换, 将查看 API URL 中的 id 直接替换成 1。

```json
{
    "status": "-1",
    "result": [{
        "id": 1,
      	"name": "zhangsan",
      	"links":[
          {
            "href": "http://localhost:8080/user/1"
          }
        ]
    }]
}
```



## 提升效率工具

### RestfulToolkit

后端开发人员可以安装 IntellJ idea 插件 [RestfulToolkit](https://plugins.jetbrains.com/plugin/10292-restfultoolkit), Mac 环境使用快捷键 `CMD + \` 输入关键字快速定位到 API 位置

![官网截图](https://plugins.jetbrains.com/files/10292/screenshot_17681.png)

同时在右侧工具栏打开 API，会自动生成 demoData 请求参数，实现快速调用测试：

![参数](https://plugins.jetbrains.com/files/10292/screenshot_17679.png)



### JSON-Viewer

JSON-Viewer  是 Chrome 浏览器的插件，用于快速解析及格式化 json 内容，在 Chrome omnibox（多功能输入框）输入`json-viewer + TAB` ，将 json 内容拷贝进去，然后输入回车键，将看到结构清晰的 json 数据，同时可以自定义主题

![](http://ww1.sinaimg.cn/large/8dc363e6ly1g39u1u0er6j21v80tyadz.jpg)

另外，前端人员打开开发者工具，双击请求链接，会自动将 response 中的 json 数据解析出来，非常方便

### Postman

Postman 功能十分强大， 搜索 `Postman 自定义环境变量`，会打开新世界的大门

## 写在最后

如何设计出最好的 RESTful API 永远不会有结论，设计出高可用，团队认可，简单清晰明了的 RESTful API 就是好的。

欢迎交流你们在团队中是如何设计 RESTful API 的，遇到了哪些问题，是如何解决和规范的
