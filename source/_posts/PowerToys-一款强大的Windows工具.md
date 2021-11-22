---
title: 'PowerToys,一款强大的Windows工具'
date: 2020-11-29 21:15:19
tags:
    - 工具
categories: [Coding, Tools]
id: powertoys-a-powerful-tool-in-Windows
description: Windows 下强有力的多合一工具，颜色提取器，文件重命名，多窗口管理，快捷键管理，文件快速查找等功能一应俱全
keywords: PowerToys, Color Picker, FancyZones
---

上次写了个 [cheat.sh 在手，天下我有](https://dayarch.top/p/cheat.sh-usage.html)，小伙伴们热情高涨，觉得这是一个没有杂质的好工具；也有小伙伴抱怨说对 Windows 用户不是特别友好 （其实用 curl API 是没啥问题的）。为了「雨露均沾」，今天就介绍一款对 Windows 超级 * N （N 是几，大家读完文章自己定） 友好的工具

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129212948.png)



我甚至有些嫉妒，因为 Windows 的这款工具需要我在 Mac 结合好几个工具才能达到与之相媲美的效果 

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129192100.png)



今天的主角就是~~~~

## PowerToys

[PowerToys](https://github.com/microsoft/PowerToys "PowerToys") 就是这么一款有 Power 的工具，使用的前置要求很简单，**Windows 10 操作系统**，截至写本文时，PowerToys release 的版本为 [v0.25.0](https://github.com/microsoft/PowerToys/releases/tag/v0.25.0)，打开 release 页面，直接拉到页面底部，下载 exe 文件即可

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129192934.png)

安装没有什么顾虑，选定一个安装目录，无脑下一步，直至安装结束

<img src="https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129193311.png" style="zoom:25%;" />



安装完打开 PowerToys，在左侧边栏看到的就是 PowerToys 的全部功能

<img src="https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129193836.png" style="zoom:50%;" />

我们来逐一看看这些强大的工具

### Color Picker

直译过来就是**颜色提取器**（*在 Mac 中我用 Sip*），这应该是前端工程师的标配，后端工程师建博客以及画图，如果需要好的配色，都是很需要它滴



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129194218.png)



使用快捷键 `Win + Shift + C` 就可以滑动鼠标取色了，如果一个地方的颜色分布比较细腻，还可以通过滚动鼠标滚轮进行放大提取，支持两种颜色值（RGB ｜ HEX）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129194637.png)



### FancyZones

程序员从前怼到后，干一个活同时开很多个应用都是很正常的现象，合理快捷的利用屏幕区域显得尤为关键。比如打开一个 Terminal， 它只需要在一个角落里让我们动态地看到 log 就行。按照以往，我们可能会选择用鼠标缩放某个应用到指定大小，然后用鼠标拖拽的合适的位置上，两个字「**麻烦**」

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129195313.png)



FancyZones 就是一个非常有效的窗口管理器 （*Mac 中我用 SizeUp*)， 你可以按照模版布局进行设置：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129195714.png)



也可以自定义你喜欢的的布局：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129195917.png)



然后就是按照你喜欢的方式拖拽应用到相应的窗口就可以了，还可以通过快捷键 `Win+Left/Right Arrow` 快速在多个区域之间移动你的应用 （**如果快捷键有冲突，可以自行设置**），FancyZones 还有很多高级设置，大家根据自己的喜好一次配置到位就好了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129200430.png)



### File Explorer Preview

Mac Finder 自带文件预览功能，但是却没有**渲染文件并预览**的功能，File Explorer Preview 就实现了这个完整的功能，使用这个功能的前提是把 Preview Pane(预览面板)打开，就是选中这个：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129201144.png)

File Explorer Preview 目前支持三种文件类型的渲染和预览：

- MarkDown
- SVG
- HTML 

如果你打开 SVG 文件并编辑颜色区域，`CTRL + S`  保存，还可以实时预览出新的变化，爽歪歪

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgdemo.gif)



### Image Resizer

处理图片大小，日常用 Mac 的Preview 或者找一些在线的 Resize 图片工具。但 Image Resizer 缺带来了不一样的赶脚

- 单个图片 Resize
- 批量多个图片 Resize
- 自定义 Resize 大小
- 批量生成 Resize 之后的图片文件名

直接一张图片自己体会吧

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgresizeSettings.gif)



### Keyboard Manager

直译键盘管理器，其实就是重定义快捷键的。当多个应用快捷键冲突的时候，在此集中管理那是一点都不混乱，不但可以 remap 全局的快捷键，还可以 remap 某个应用的快捷键

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129202836.png)

不知道打游戏的快捷键会不会被 remap，大家可以试试



### PowerRename

如其名，这是一个重命名的功能，到底 Power 在哪里呢？还是看动图说明吧

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgPowerRenameDemo.gif)



配合下面这些 Options 的使用，真是强大而灵活，这就是 Power

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129203234.png)

文件重命名还动态支持一些变量，这个大家自行查看说明就好了



### PowerToys Run

当我们查找一个应用，我们要找桌面上的图片，常用的还好一下可以定位到位置，万一不常用可使要多找上一会；如果要找个文件或文件夹更是麻烦。PowerToys Run 就为解决这个痛点而生 （*Mac 下我用 Alfred*），我知道有些插件比如 UTools 也有类似的功能，但是 PowerToys Run 和 Windows 系统融合得更加流畅

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgFeatures.gif)

日常做个加减乘除，也不用再打开计算器了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgFeaturesCalculator.gif)

这一切，只需要按一下快捷键 `Alt+Space` 即可



### Shortcut Guide

这是一个显示使用 Windows 键的常见键盘快捷键，使用方式很简单，按住 Windows 键大概 1秒钟，就会显示出配合 Windows 键使用的快捷键了，甚至任务栏的应用都有快捷键，直接一键跳到相应的应用上，不用再 tab~tab 来回切换了 (Mac 下我要用 cheatsheet 和 PopClip 结合使用)

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129204234.png)

看看到上图右侧，一定有你不知道的快捷键功能，是不是方便了许多呢？



### Video Conference Mute (Experimental)

这是一个 Pre-release 的功能，受疫情影响，目前还有很多小伙伴在家远程办公，视频会议成为了日常的沟通手段。在办公室喊一嗓子就可以解决的问题都要跑到线上来说明，难免会有一些多个任务并行的时刻，都开视频语音肯定会互相干扰，所以灵活的切换或静音就是一个非常给力的功能了，这个自行设置快捷键就可以了：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201129205407.png)



## 总结

Github上有伙伴说，PowerToys 工具的这些功能在未来甚至可能直接成为 Windows 的内置功能，这个到底能否成真不可知，至少现在我们也可以享受这些功能带来的便利，这个多合一的工具，偶尔让我觉得 Mac 也不是很香，有 Windows 10 系统的小伙伴们都可以实验起来了



在介绍 PowerToys 工具时，我也在文中附上了 Mac 的工具，这样的均沾您觉得还可以吗？
