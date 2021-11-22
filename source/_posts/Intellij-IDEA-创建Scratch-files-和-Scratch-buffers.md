---
title: Intellij IDEA 创建Scratch files 和 Scratch buffers
date: 2020-10-31 21:18:28
tags:
  - IntelliJ IDEA
id: scratch-files-and-scratch-bufferss
categories: [Coding, Java]
description: 有时候我们需要在项目之外创建一些临时文件或一些实验性代码，创建在项目中可能一不小心 Git 误提交，不创建项目里又需要切换到其他应用上来回拷贝，对于专注 coding 的我们来说，总显得不够流畅，IDEA 其实早已为我们解决了这个痛点，借助 Scratch files 和 Scratch buffers 就可以解决
keywords: Scratch files, Scratch buffers, Intellij IDEA
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200207104108.png
---

多数人对于 Intellij IDEA 可能始于其「颜值」，终于其「才华」，外加各种插件 buff 的加成，coding 的节奏分分钟要暴走

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201030214621.png)</fancybox>



抛开自己安装的插件，IDEA 其实也内秀的很，在 [IntelliJ IDEA HTTP Client高级使用详解](https://dayarch.top/p/http-client-advanced-usage.html) 中详细的介绍了开发小组内放弃 Postman 的理由，用过的小伙伴后台留言直呼爽。



但今天要介绍的是另外一个秀的有些含蓄的小功能，她那么显眼的站在你面前，你却选择忽视她

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201030215504.png)</fancybox>



有时候我们需要在项目之外创建一些临时文件或一些实验性代码，创建在项目中可能一不小心 Git 误提交，不创建项目里又需要切换到其他应用上来回拷贝，对于专注 coding 的我们来说，总显得不够流畅



IDEA 其实早已为我们解决了这个痛点，借助 Scratch files 和 Scratch buffers 就可以解决



## Scratch files ｜ Scratch buffers

IDEA 会在项目平行目录中自动生成下面的目录结构，这就是今天主角的位置，你有正视过她们吗？

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201030220217.png)</fancybox>



Scratch files 和 Scratch buffers 二者还是有很大差别的：



### Scratch files

Scratch files 是一种功能完整的、可运行的和可调试的文件，支持语法高亮显示、代码补全和相应文件类型的所有其他特性 (说白了很像Jupyter Notebooks)



Scratch file 的功能，就可以满足我们在 Coding 中的各种想法，用于快速记录。创建好的这个文件并不存储在我们的项目目录中（避免了意外 git push 的尴尬），甚至在 IDEA 中切换到其他项目中也可以看到你刚刚创建的这个文件，进一步说白一点，这是凌驾在项目之上的一个全局功能，如下图，切换到了其他项目中，Scratch files 依旧存在

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201030221641.png)</fancybox>



创建一个 Scratch file 很简单，使用快捷键 `⇧⌘N` ，支持关键字搜索，直接创建相应类型的文件就可以，比如这里创建一个 java 文件

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201030222023.png)</fancybox>

默认会创建名为 Scratch.java 的文件，并写好 main 函数，就像这样：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031193921.png)</fancybox>

在这里和你正常在项目中写 java 代码没什么区别



其实我很常用的是创建一个 scratch.sql 文件，存放一些日常 SQL 语句，同样的创建方式，搜索 `sql` 默认会创建一个 scratch.sql 的文件，文件创建之后要配置数据源

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031194800.png)</fancybox>

配置好后，就可以尽情的书写你日常用到的 SQL 了

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031195807.png)</fancybox>

光标放到 SQL 位置，使用快捷键 `⌘⏎` ，选择相应的 Session （会话） 就可以 run 这条 SQL 语句了

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031200512.png)</fancybox>

执行后，就看到你熟悉的画面了，因为这里的画面就是 IDEA 默认的数据库管理工具，这篇 [IntelliJ IDEA的数据库管理工具实在太方便了](https://mp.weixin.qq.com/s/SmjBdZAMynTndU_oTITd8Q) 文章中有过详细说明

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031200604.png)</fancybox>



创建 Scratch files 可选择的类型非常多，总有一个适合你的一些需要



### Scratch buffers

和 Scratch files 类似，只不过 Scratch buffers 就是一个简单的文本文件，没有任何编码辅助功能 （说白了，可以将它理解成一个记事本），创建 Scratch buffers 没有直接的快捷键，需要用通用快捷键 `⇧⌘A` ，并输入关键字（比如 buffer）：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031201105.png)</fancybox>

敲击回车键，就会创建好一个名为 scratch.txt 的文本文件。**反复创建 Scratch buffers，你会发现，最多只允许创建 5 个**：

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031201657.png)</fancybox>

因为这个操作不频繁，所以也就没有默认快捷键，如果你是个快捷键强迫症患者，那就在 KeyMap 处添加相应的快捷键就可以了

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031201344.png)</fancybox>

这里要说明一个**注意事项**：

> 如果你在 buffer1.txt 文件中记录了一些内容，当你创建第 6 次 scratch buffer 文件时，buffer1.txt 的内容就会被清空



减少应用切换，尽量保持专注，可以借助预览模式(ctrl+opt+v) 和 快捷键 （cmd+e）切换最近常用文件，戴上耳机，快告诉我，时速多少迈？

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201031202406.png)</fancybox>



## 总结

这是一个很小的功能，但是在日常开发中真的可以带来很大的帮助：

- 跨越项目的访问
- 不被 Git 所跟踪，防止误提交
- 可以临时测试各种语言的代码

最后配合预览模式的使用，减少应用之间的切换，一切显得都很流畅

> 当然，保持 Code Clean，减少猜测和回忆时间，我们最好给我们创建的 Scratch files 和 Scratch buffers 更友好的文件名称

