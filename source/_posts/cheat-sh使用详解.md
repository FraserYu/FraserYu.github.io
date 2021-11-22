---
title: cheat.sh使用详解
date: 2020-11-26 22:23:57
tags:
    - 工具
categories: [Coding, Tools]
id: cheat.sh-usage
description: 作为程序员需要了解的东西有很多，日常编码和写脚本脱离不开各式语言与 Linux 命令。为了记住一些杂乱的或不被经常使用的知识点，我们迫切需要一个“小抄”/备忘录
keywords: cheat.sh
---

## 前言

作为程序员需要了解的东西有很多，日常编码和写脚本脱离不开各式语言与 Linux 命令。为了记住一些杂乱的或不被经常使用的知识点，我们迫切需要一个“小抄”/备忘录，小抄内容多了自然繁杂，所以我们希望这个小抄要：

1. 简洁：只包含你想要的内容，没有其他「花边」内容
2. 快速：可以立即使用
3. 全面：能基本包含你所有问题的答案
4. 通用：它应该在任何地方、任何时间都可用，不需要任何准备
5. 不唐突：它不应该让你从主要任务上分心（比如减少应用切换）
6. 辅导：它应该帮助你学习这个科目（在答案基础上扩展知识）
7. 不显眼：应该可以在完全不被注意的情况下使用（就好比划词翻译，鼠标轻点就有答案）



老gong，你是想介绍哆啦A梦吗？

<img src="https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115205225.png" style="zoom:25%;" />



非也，其实是 `cheat.sh`



## cheat.sh 介绍

[cheat.sh](https://github.com/chubin/cheat.sh "cheat.sh") 就是一个可以满足上述愿望的小哆啦，目前在 Github 的形式是这样滴：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115210247.png)

Commit 也非常活跃，就是这么一个哆啦

- 它提供一个简单的 curl/浏览器接口方便我们查询
- 目前覆盖 58 种编程语言，多种 DBMS以及超过 1000 个UNIX/Linux 常用命令
- 提供对世界上最好的社区驱动的备忘单存储库的访问，与StackOverflow持平（绝对是高质量的内容）
- 提供命令行客户端
- 可以嵌套在代码编辑器中使用，比如 Intellij IDEA 和 VS-Code
- 支持一个特殊的隐身模式，可以完全隐形的使用它 （感觉挺神秘的呢）



先来认识一下，打开命令行终端，使用 curl 命令输入：

```shell
curl cht.sh
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115210940.png)



### 如何使用 cheat.sh

先拿几个常用的 UNIX/Linux 命令练练手：

```shell
curl cht.sh/tar
```

瞧这整理的规范和简洁不？

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115211502.png)



```shell
curl cht.sh/tr
```

答案依旧整洁规范， 同时还**高亮显示**，友好的很啊

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115211649.png)



如果你不知道某个命令，还可以使用 `～Keyword` 的形式来查询，比如你想查看如何建立快照

```shell
curl cht.sh/~snapshot
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115213039.png)



上面说过， cheat.sh 包含 1000 多个常用的 UNIX/Linux 命令，当需要的时候，按照语法 `curl cht.sh/<you-cmd>` 尽情查询吧

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img6af89bc8gw1f8qfau787rg203k034glk.gif)



除了 Linux 命令，我们还说支持 58 种语言，当写代码时某个 API 不会用或需要完成某些操作，cheat.sh 依旧可以帮上忙，比如我总是记不住 Java Lambda 的 group 操作

```shell
curl cht.sh/java/lambda+group
```

记住下面的标准格式，搜索的结果都是和  StackOverflow 一样的高质量

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115215811.png)



如果这个答案还不是你想要的，你就可以**添加数字进行翻页**获取其他结果

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201115220446.png)



另外你觉得结果中的注释很碍眼的话，可以在每次查询的后面加上 `\?Q `，就像这样：

```shell
curl cht.sh/java/lambda+group\?Q
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201119215347.png)



当然每种语言都默认支持 :list 和 :help 两种查询方式，作为帮助指令，大家可以自行尝试了，比如 go 语言

```shell
curl cht.sh/go/:list
curl cht.sh/go/:help
```



相信到这里，你已经可以掌握 cheat.sh 的基本使用方式了



但是，这种 curl 方式总是显得不是很方便，比如空格要用 `+`  替代，日常工作语言比如只有 Java，每次都要输入 **curl cht.sh/java/xxxxxxx** 这样就会显得很麻烦， 为了解决这些问题，cheat.sh 很贴心，也提供了命令行客户端，大大简化了搜索操作



### Cheat.sh 命令行客户端

#### 安装

安装 CLI Client （Command Line Client）非常简单，只需要依次执行下面的命令即可：

```shell
# 注意你的环境变量 PATH 已经 import 了 ~/bin 下的内容
mkdir -p ~/bin/
curl https://cht.sh/:cht.sh > ~/bin/cht.sh
chmod +x ~/bin/cht.sh
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201122203135.png)

如果要保证 shell 模式可用，还需要安装一个依赖 `rlwrap`, 下面两种安装方式都可以（我直接用brew安装的）

```shell
brew install rlwrap
# 或者
sudo apt install rlwrap
```

#### 使用

有了 CLI Client 之后，来看一看搜索上的变化：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201122204101.png)



这个 CLI Client 还提供了一个更加便利的 shell 模式：

```shell
cht.sh --shell
```

如下图，每次直接按照语言搜索相关内容就可以了：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201124214729.png)

通常我们编程在一段时间内会用一种语言，我们可以进一步简化搜索过程，cd 到某个语言目录下：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201124214952.png)

如果进入 shell 模式，同时想一次性进入某个语言目录，也可以通过一条命令搞定：

```shell
cht.sh --shell java
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201124215150.png)



#### 隐身模式

相信很多小伙伴都配有划词工具，比如某个单词不会了，选中相应的单词，就会出来解释，cheat.sh 也有类似的模式，进入某个语言目录下之后，输入 `stealth Q` 就可以进入这个模式了：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201124215809.png)用鼠标选中文本后，用起来的效果就是这样滴：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgstealth.gif)

> 不过这里建议，**搜索的单词不要超过 5 个**



以上这些使用方式，默认都会调用它自己的服务，为了更快速的响应，我们可以搭建自己的服务，前提是要更改 CLI Client 的 server URL：

打开或新建 `~/.cht.sh/cht.sh.conf` ，添加

```shell
CHTSH_URL=https://cht.sh            # URL of the cheat.sh server
```

然后就可以 run 自己的服务 

```shell
git clone https://github.com/chubin/cheat.sh.git
cd cheat.sh
docker-compose up
```

最后访问服务： [http://localhost:8002](http://localhost:8002/) 



### 集成主流编辑器

cheat.sh 同样和主流编辑器有很好的集成：

|Feature            |Emacs|Sublime|Vim|VSCode|IDEA|QtCreator|
|-------------------|-----|-------|---|------|----|---------|
|Command queries    |✓    |✓      |✓  |✓     |✓   |✓        |
|Queries from buffer|     |       |✓  |✓     |    |✓        |
|Toggle comments    |     |       |✓  |✓     |✓   |✓        |
|Prev/next answer   |     |       |✓  |✓     |✓   |✓        |
|Multiple answers   |     |✓      |   |      |✓   |         |
|Warnings as queries|     |       |✓  |      |    |         |
|Queries history    |     |       |✓  |✓     |    |         |
|Session id         |     |       |✓  |      |    |         |
|Configurable server|✓    |       |✓  |✓     |    |✓        |

Vim 的集成度是最高的，大家可以根据 [cheat.sh-vim](https://github.com/dbeniamine/cheat.sh-vim) 自行配置

VSCode 和 IDEA 是大家高频使用的两个 IDE，和他们集成就很简单了，只需要安装相应的插件：



#### VSCode 插件

安装 [vscode-snippet](https://github.com/mre/vscode-snippet) 就可以在 VSCode 中快速使用这个功能了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgvs.gif)



#### IDEA 插件

安装 [idea-cheatsh-plugin](https://github.com/szymonprz/idea-cheatsh-plugin) 这个插件就可以在 Intellij IDEA 中使用这个功能了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgidea.gif)



## 总结

至于支持的 58 种语言都是什么，请大家自行参考 README 文档，关于 cheat.sh, 了解这些基本的使用就已经够了，还是那句话，好的工具是用来提高工作效率的，不要被工具过度捆绑，公众号回复「工具」获取更多内容


