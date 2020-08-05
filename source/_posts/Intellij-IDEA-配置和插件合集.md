---
title: Intellij IDEA 配置和插件合集
date: 2020-05-03 09:43:47
tags:
  - IntelliJ IDEA
id: intellij-idea-configuration-and-plugins
categories: [Coding, Java]
description: 高效的使用 IDEA 进行开发离不开好的配置和插件，本文会陆续更新和整理那些可以大大提高编码效率的配置
keywords: IntelliJ IDEA，IDEA， 插件, plugins
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200207104108.png
---

最近在陆续写 [Java 并发编程系列](https://dayarch.top/categories/Coding/Java-Concurrency/)，好多朋私信问我的不是并发内容本身，而是我的 IDEA 主题配置。我就姑且认为好的主题配置可以写出更好的并发程序吧

即便这种可能性只有万分之一，我也要把我的 IDEA 相关值得配置的内容/插件和大家分享出来


先来一张我的 IDE 截图，有你看中的地方吗？



<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200429091925.png)</fancybox>



## 插件篇

好用的插件总是让人：

神清气爽，精神抖擞 ，丰神异彩，炯炯有神，神采奕奕，气贯长虹 ，英姿飒爽，精神焕发 



**下面所有插件都可以按照文中标注的名称在 IDEA 插件市场中直接搜索并安装**



### Material Theme UI 

Material Theme UI  在主题下载量排行榜中高居第一。安装主题后（在页面底部就会有进入主题的快捷入口），选择自己喜欢的主题进行微调就可以啦，如果懒得做配置，按照下图勾选相应设置就和我的一样了：

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at10.46.17.gif)</fancybox>


### Rainbow Brackets

翻译过来叫【彩虹括号】，该插件除了可以实现多彩的括号匹配外，我使用更多的是其【区域代码高亮】功能 ，这样可以清晰定位区域代码内容

> Mac 快捷键：`cmd + 鼠标右键`; 
>
> Windows 快捷键：`ctrl + 鼠标右键` 

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at11.31.30.gif)</fancybox>



你也可以使用非选中部分暗淡效果

> 快捷键：`alt + 鼠标右键`

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at11.35.02.gif)</fancybox>

相比之下，我更喜欢第一种高亮效果




### Codota

Codota 总结起来有三大功能：

#### 1. 智能自动补全让编码速度更快

当编写代码时，Codota 可以快速的完成智能补全以及高频用法提示

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at21.18.45.gif)</fancybox>

#### 2. 从经过测试或证明过的程序中获得编码建议

如果我们觉得给出的提示不够清晰，可以使用快捷键： `ctrl + shift + o` ， 快速查询相关使用案例，同时可以通过添加关键字进行过滤，查找到更加精确的代码样例

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at21.23.10.gif)</fancybox>

#### 3. 不脱离IDE发现并利用更多开源代码

当你不知道某个类如何使用时，可以直接使用快捷键：`ctrl + shift + y` ， 然后输入关键字，会查询到很多【开源框架】中使用该类的经典案例。不用脱离 IDE，没有广告，没有废话，只有经典的代码样例，你说爽不爽？

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-01at21.25.26.gif)</fancybox>

如果你用 Eclipse ，Codota 也是完美支持的
<fancybox>![](https://rgyb.sunluomeng.top/20200502193044.png)</fancybox>


### Key Promoter X

高效的编码脱离不开快捷键的使用，但是 IDEA 中的快捷键实在太多了，不是很方便记忆，Key Promoter X 会根据你使用的功能提示你设置快捷键

<fancybox>![](https://rgyb.sunluomeng.top/2020-05-03at10.24.10.gif)</fancybox>


设置完后，如果你忘记了该快捷键，再次通过鼠标点击该功能时，设置好的快捷键就会相应的提示出来，真是深知码农苦啊

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-02at19.49.37.gif)</fancybox>

### Alibaba Java Code Guideline

安装该插件后，你编写的代码就会按照 Alibaba 编码规约规范我们编写的代码（谁说我们不能写出和大厂一样规范的代码？）

比如通过Executors 快速创建一个线程池是不被允许的，具体原因已经在[我会手动创建线程，为什么要使用线程池?](https://dayarch.top/p/why-we-need-to-use-threadpool.html) 中说明，当检测出之后，鼠标悬浮在上面会给出提示，按快捷键 `cmd + F1` 会更完整的告诉你为什么不可以，同时还给出你应该怎样编写的样例 （暖暖的，很贴心）

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-02at20.10.38.gif)</fancybox>


### CodeGlance

英文直译【代码一瞥】，细心的朋友已经在开篇的图中看到了这个设置，安装该插件后，IDE右侧会出现一个mini 视图，比如看 ConcurrentHashMap 源码，那么长的内容，可以通过该插件快速的拖动到大概位置，方便很多

<fancybox>![](https://rgyb.sunluomeng.top/20200502202154.png)</fancybox>


### Restful Tookit

这个插件之前我有介绍过，编写 RESTful 接口必不可少的插件，编写完接口当然还需要调试，搭配HTTP Client 一起使用才更香 [IntelliJ IDEA HTTP Client高级使用详解](https://dayarch.top/p/http-client-advanced-usage.html)，相信你学会使用这个，不单单是你个人，你们整个小组都会因此受益




### CamelCase

编码离不开字符串的使用，安装该插件后，可以通过快捷键 `shift + alt + U` 快速的切换字符串格式，当然如果你只是单纯的切换大小写，使用 `shift + cmd + U` 更便捷一些

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-02at20.37.56.gif)</fancybox>



## 设置篇
### 方法分割线以及字符缩进

通过方法分割线可以更清晰明了的区分方法的边界，通过字符缩进也可以让代码的层次感更加明显，先来看整体效果

<fancybox>![](https://rgyb.sunluomeng.top/20200502205359.png)</fancybox>


只需要按照下图勾选相应选项就可以了 （现在是不是很清晰？）

<fancybox>![](https://rgyb.sunluomeng.top/20200502205434.png)</fancybox>


### Editor 边栏位置设置

如果你不能熟悉的使用 `cmd + E` 快捷键（一不小心给了提示）打开你最近常用的文件，顶部位置可显示的打开的类名称少之又少，所以通过挪动 Editor 的显示位置，就可以解决这个痛点问题，现在是不是极度舒适了呢？

<fancybox>![](https://rgyb.sunluomeng.top/Kapture2020-05-02at20.46.49.gif)</fancybox>

---

当你觉得需要 IDEA 帮助你更高效的工作时，不妨去插件市场搜搜看，没准就有意想不到的惊喜。另外，这是一个 IDEA 设置篇的合集，我会陆续将更多设置和好用的插件整合到这一篇文章当中，为了不错过更多精彩内容，你可以直接选择收藏啦

最后，我也整理了很多可以帮你高效工作的工具，公众号回复【工具】即可



