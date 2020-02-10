---
title: 小小TODO也有大道理
date: 2020-01-09 10:50:07
categories: [Coding, 安全-协议-规范]
tags:
  - IntelliJ IDEA
id: how-to-use-todo-feature
description: 有时，我们需要标记部分代码以供将来参考，比如:优化，改进，可能的更改，要讨论的问题等。通常我们会在代码中加入TODO作为标识，但是在团队中协同工作，TODO的背后也蕴藏着一些道理
keywords: TODO, Intellij IDEA, IDEA, 注释, comment
---

## 前言
有时，我们需要标记部分代码以供将来参考，比如: 优化，改进，可能的更改，要讨论的问题等。 通常我们会在代码中加入如下的标记表示待办:

```java
//TODO 我将要在这里做 xxx
```

你这样做，别人也会这样做。一时间，项目中可能会存在大量的 `TODO`，当你搜寻你的 `TODO` 时也就变得非常麻烦，如同石沉大海，也就失去了这个标记的意义。

IntelliJ IDEA允许我们添加特殊类型的注释，使得这些注释在编辑器中突出显示，它们被索引，并在 `TODO 工具窗口` 中列出。这样，我们就容易追踪自己的 `TODO` 了。

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgTODOexample.png)</fancybox>

## 默认的 `TODO`

默认情况下，IntelliJ IDEA识别两种模式：小写和大写的 `TODO` 和 `FIXME` 这些模式可在任何受支持文件类型的行注释和块注释内部使用。我们可以根据需要修改默认模式或添加自己的模式

如上图，我们可以创建多行的 `TODO` (类似 Spring Boot 中的 YAML 配置多个值)，需要缩进第一行之后的注释行。如果没有缩进，则将行视为常规注释行

要禁用多行 `TODO` 项目，使用快捷键 `⌘ + ,` 打开 Preferences， 搜索 `TODO` (Editor | TODO), 你会看到如下界面

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-01-09_09-06-39.jpg)</fancybox>

要查看系统中的所有 `TODO`，请打开 TODO 工具窗口 (快捷键 `⌘ + 6` )。切换选项查看 `TODO` 范围:
- 从当前项目中的所有文件
- 仅基于当前文件的范围
- 指定范围的文件
- 活动的变更列表

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-01-09_09-23-45.jpg)</fancybox>

到这里 Intellij IDEA 默认提供的 `TODO` 就介绍完了，为了能更快的找到我们自己的 TODO，我们就需要进行自定义

## 自定义 TODO
重新打开 `TODO` 位置，新增 TODO item，这里新增 `optimize`，用于标识待优化内容

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-01-09_09-46-03.jpg)</fancybox>

添加个过滤器，用于 `TODO` 的分组
<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-01-09_09-50-35.jpg)</fancybox>

随便添加一个优化备注，通过以上介绍的功能，快速定位到我们自己的 `TODO`

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgXnip2020-01-09_09-54-37.jpg)</fancybox>

如果你的待办事项通常是相对固定的描述，你也可以配合 Live Template 快速生成 `TODO` 内容

## 总结
当团队规模很大，你又同时有很多待办的时候，`TODO` 特性可以帮助我们做标识，自定义 `TODO` 可以帮我们快速定位，我们可以充分利用这个特性，但是
> #### 定期清理 TODO
> <fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img2020-01-09-10-45-53@2x.png)</fancybox>



## 灵魂追问
1. 你觉得项目中代码有哪些不规范/不够整洁的地方？(欢迎到博客下方留言讨论)

- - - - - 
