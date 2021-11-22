---
title: IntelliJ IDEA HTTP Client高级使用详解
date: 2020-02-07 10:30:15
tags:
  - IntelliJ IDEA
id: http-client-advanced-usage
categories: [Coding, Java]
description: 后端程序员经常要编写RESTful API，一个好用的测试工具可以大大提高工作效率，甚至整个项目组的效率，IntelliJ IDEA 本身提供了 HTTP Client 这个工具，但是鲜有人知，本文要说明该工具的高级用法，一定能解决项目中的调试痛点
keywords: HTTP Client, RESTful, 调试接口工具
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200207104108.png
---
> - 你有一个思想，我有一个思想，我们交换后，一个人就有两个思想
>
> - If you can NOT explain it simply, you do NOT understand it well enough



陆续将Demo代码和技术文章整理在一起 [Github实践精选](https://github.com/FraserYu/learnings)，方便大家阅读查看，觉得不错，还请Star🌟🌟🌟🌟🌟🌟




![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgfenge.gif)



抗击疫情，在家办公。工作照常干，领导需要几个新功能接口开发。以前在公司办公，通常开发完的功能没什么问题，暴露出 Swagger 接口文档，直接找旁边的前端大人联调测试了



当下，君在长江头，我在长江尾，夜夜思君不敢出门见君啊，一切测试全交给自己吧，



<img src="https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206102942.png" style="zoom:50%;" />

虽然想念前端，但是在家办公也绝对是和在公司办公一样一样的，高效不（qu）摸鱼

## 插播背景

在多个产品线上来回穿切换着开发功能，以前用Postman的场景是这样的:

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206110455.png)</fancybox>

实际远远不止这几个文件夹来归类多个产品线的接口，Postman的功能非常强大，但是面对以下这些状况时，我觉得调试一个接口太麻烦了 （这里不讨论工具的好坏，工具是帮助我们提高效率的，每个人的需求也不一样，我只说明我个人遇到的一些情况，不喜请勿喷）

- 查找配置多数要通过鼠标点来点去, 与习惯文本和快捷键操作的便捷方式违背
- 调试别人接口要导入他们的一些数据，比较麻烦
- 多个产品线环境变量查看不直观
- 写完接口要来回切换应用进行测试，比如（IDEA <——> Postman）
- 快速定位接口比较麻烦
- ......



无意间发现 IntelliJ IDEA 的 `HTTP Client` 工具刚好能解决我上面提到的一些问题，简单的说就是能直接在 IDEA 的代码编辑器中 `创建，编辑，执行` HTTP请求，就像这样(如果你心动了，请继续向下看吧)：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img97483310-4556-42b8-ac61-9b127c9aaf05.gif)</fancybox>



于是，去官网查看一番作出如下整理：



## 走进 Http Client 

HTTP Client 是 IDEA 默认绑定好并启用的插件，如果你那里没有启用，按照下图启用就好

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206115605.png)</fancybox>



点击菜单：`Tools` — `HTTP Client` — `Test RESTful Web Service`



<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206112908.png)</fancybox>



接下来进入下面的界面：



<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206125806.png)</fancybox>



上图已给出提示，REST Client 是被弃用的，点击右侧的 `Convert request to new format` , 进到下面界面：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206132014.png)</fancybox>



默认会创建一个名为 `rest-api.http` 的文件，该文件被存储在 Scratches 文件夹下，为了突出主角光环，关于 [Scratch Files](https://www.jetbrains.com/help/idea/scratches.html) 请官网自行查看 （**继续向下看不影响理解的**），黄色框线的功能也非常有用，继续向下看



## 创建 HTTP request 文件



刚刚提到的 `rest-api.http` 就是 HTTP request 文件，可以通过两种方式创建：

1. 通过快捷键  `⇧⌘N`  然后选择 **HTTP Request**. （文件存放在Scratches 文件夹）
2. 通过菜单操作 File—New—HTTP Request （文件存放在我们指定的目录下，就和我们平时创建class/package是一样一样滴）



如果在项目中使用，这里推荐使用第二种方式，因为它可以作为项目文件，通过 Git 提交到仓库，大家共享文件，共同维护接口请求数据，自然就不会出现调试别人接口还要导入他人数据的情况啦



## 编辑 HTTP request 文件

我们模拟实际项目中场景来编辑文件

1. 用户登录，成功后获取 Token，通常是 POST 请求
2. 用户后续访问行为都要在请求头中携带登录成功返回的 Token



通过点击 Add Request，选择相应的方法就可以编写啦

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206140641.png)</fancybox>



都知道，通常写一个完整的请求需要写好多内容，贴心的 IDEA 给我们提供了模版，我们只需要在 Examples 中找模版就可以啦，比如找 POST 请求的模版，选取合适的拷贝过去就可以，so easy～～～

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206142611.png)</fancybox>



到这里，就可以发送基本的请求了，但是，一个项目中接口众多，如何快速生成参数？如何快速切换端口？如何让登录之后的每个请求自动携带成功返回的 Token？我们需要更高级的玩法



## HTTP Client 进阶玩法

### 使用环境变量

在编写HTTP请求时，可以使用变量对其元素进行参数化。变量可以保存请求的host、port和path、查询参数或值、请求头值或请求体值等.



使用变量的方式非常简单，就用两个大括号包围定义好的变量就可以了，就像这样：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206145933.png)</fancybox>



当然我们也要有地方定义变量



### 定义环境变量

环境变量需要定义在环境文件中，环境文件有两种：

1. 创建名为 `rest-client.env.json` 或者 `http-client.env.json` 的环境文件(其实里面就是保存 JSON 数据)，该文件里可以定义用在整个项目上的所有常规变量
2. 创建名为`rest-client.private.env.json` 或者 `http-client.private.env.json`， 看文件名你应该也猜到这是保存敏感数据的，比如密码，token等，该文件默认是被加入到 VCS 的 ignore文件中的，同时优先级高于其他环境文件， 也就是说，该文件的变量会覆盖其他环境文件中的变量值



里面的文件内容就像这样

```json
{
  "dev": {
    "host": "localhost",
    "port": 8081,
    "identifier": "tanrgyb",
    "password": "iloveu"
  },
  "prod": {
    "host": "dayarch.top",
    "port": 8080,
    "identifier": "admin",
    "password": "admin"
  }
}
```



运行一下我们编写的请求吧：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206153120.png)</fancybox>



IDEA自动识别多个环境，这样就可以轻而易举的切换环境，使用不同的变量值了（这皮鞋，你说亮不亮，还有更亮的）



### 巧用 response handler 脚本

上面提到，我们要让登录成功后的所有请求都自动携带成功返回的 Token，这样不用我们每次都手动将其添加到header中，同样有两种方式将脚本插入到请求中

- 内嵌方式

```http
GET host/api/test

> {%
response 脚本
%}
```

- 外部文件方式（就是将内嵌的脚本抽离出到文件中）

```http
GET host/api/test

> scripts/my-script.js
```



以登录返回获取的token设置到变量中为例，看代码：

```http
POST http://{{host}}:{{port}}/login
Content-Type: application/json
Accept: application/json

> {%
client.global.set("auth_token", response.body.result.token);
 %}
```



> #### 注意
>
> `response.body.result.token` 是我按照我登录返回的数据结构写的，不同结构不一样，你也可以是这样的 `response.body.token` , response.body 之后根据你的数据结构发挥吧



我还是不放心，把我的登录返回结构（项目中怎样设计这种结构，可以参考之前写的[Springboot返回统一JSON数据格式是怎么实现的？](https://dayarch.top/p/spring-boot-global-return.html) )粘贴在此处吧，这回理解了吧？

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206155455.png)</fancybox>



接下来我们就可以愉快的在其他请求上携带这个 Token 了

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206161327.png)</fancybox>



> #### 注意
>
> 这里的Authorization 类型，大家根据自己的实际情况做修改，比如：Authorization:  Bearer {{auth_token}}



以上这些已经满足我的日常使用，没有进一步了解更多，更多关于 Response 脚本的用法请大家查看官网 [HTTP Response reference](https://www.jetbrains.com/help/idea/http-response-reference.html)吧 



你以为到这里结束了（OMG），还有香料需要和大家分享，搭配上面功能使用更棒哦



## 辅助功能说明

###  RestfulToolkit

RestfulToolkit 同样是个插件，在插件市场搜索安装即可

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206163158.png)</fancybox>



安装了这个插件后，打开侧边栏，项目的所有接口信息都会展现在此处：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206163719.png)</fancybox>



我常用的功能就是把指定接口生成的JSON数据拷贝到 HTTP request 文件中，免去手写的麻烦了，你说方便不？



除此之外，使用快捷键 `cmd+\`, 可以根据关键字快速找到接口，回车迅速到达代码接口位置，这也是带来了极大的便利

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206164118.png)</fancybox>





### Live Template

项目中请求内容各有不同，IDEA标准提供的GET POST 请求案例可能还不能满足我们的需求，这时我们就可以利用 Live Template 定制自己的模版，迅速生成request 内容，像这样：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgb9bb5b55-737f-40f2-aedc-24096909ff26.gif)</fancybox>



### JSON Viewer

JSON Viewer是一款 Chrome浏览器插件，在浏览器 Omini-box 中输入 `json-viewer` + `Tab`, 粘贴json在此处，就可以对json数据进行格式化了

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200206164811.png)</fancybox>



打开开发者工具，在Network下双击某个HTTP请求，会自动在 new tab下格式化返回的json数据，免去了粘贴数据然后格式化的烦恼



关于自测接口的干货我抖的差不多了，抖抖更健康



## 总结

再次重申，不做工具党，也没有任何批判之意，工具只是为了让我们更高效的工作，选择适合自己的。从上面的介绍中来看，IDEA HTTP client 搭配我说的几个辅助功能很好的解决了文章开头说明的几个问题，对我个人情况来说，足矣！！





**截图码字不易**

- 如果你认为本文对你有帮助，还请「在看/转发/赞」（这就是送我的大火箭🚀，大跑车🚗，大灰机✈️）

- 如果你还发现了更好的功能还请补充在留言区，我回头继续补充这块内容，万分感谢

- 公众号回复「工具」，还有更精彩的等着你

  

## 灵魂追问

1. Kibana的Dev Tools用来调试ES接口，可以延伸了解一下
2. 你在项目中如何高效测试接口与联调的呢？还请大方赐教
3. 在家办公和公司办公对你有什么影响？



## 参考

- [Testing RESTful web services﻿](https://www.jetbrains.com/help/idea/testing-restful-web-services.html)



