---
title: Docker命令自动补全
date: 2020-12-04 22:23:44
categories: [Coding, Docker]
tags:
  - Docker
id: docker-install-command-completion
description: 安装Docker命令补全，加速学习
keywords: Docker, Docker命令补全, Docker命令自动补全
---

## 前言

不知道这个小伙伴有多久没用过 Docker 了， 突然对我说 Docker 命令怎么发生变化了

```shell
docker run ...
#变成了
docker container run ...
```

他说，本来对 Docker 命令就不熟悉，这下感觉更加混乱了。其实个人看来，这么变化还使得命令看着更加规整



当在命令行直接输入 `docker` 然后回车：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201203215735.png)

从图中可以看出，Docker 将命令结构化的划分了两大类，Management Commands 和 Commands，其实前者就是一级命令，后者就是子命令 （这是自 Docker 1.13 开始的改动），所以以后使用命令就是这样滴：

```shell
docker <Management Command> <Sub-Command <Opts/Args>>
```

这样以后我们使用命令只需要先关注 Management Commands 就可以了，那后续的子命令还是不知道怎么用，还要一点点查询嘛？



## Docker 命令自动补全

为了解决这个问题，Docker 也提供了非常完善的命令自动补全功能，也就是把一切交给 Tab 键



### Mac 安装Docker命令自动补全

逐条键入下面命令：

```shell
brew install bash-completion

sudo curl -L https://raw.githubusercontent.com/docker/compose/1.27.4/contrib/completion/bash/docker-compose -o /usr/local/etc/bash_completion.d/docker-compose
```

打开 `~/.bash_profile` 文件，将下面内容粘贴进去：

```shell
if [ -f $(brew --prefix)/etc/bash_completion ]; then
 . $(brew --prefix)/etc/bash_completion
 fi
```

然后刷新使之生效

```shell
source ~/.bash_profile
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img但是.gif)





我觉得 Zsh 更好，为什么？答案请看这篇：[这篇 iTerm2 + Oh My Zsh 教程手把手让你成为这条街最靓的仔](https://juejin.cn/post/6844904178075058189)



### Zsh安装Docker命令自动补全

如果没有安装 Oh-My-Zsh shell，第一步则是要先安装它，逐条键入下面命令：

```shell
mkdir -p ~/.zsh/completion

curl -L https://raw.githubusercontent.com/docker/compose/1.27.4/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose
```

打开 `~/.zshrc` 文件，将下面内容粘贴进去：

```shell
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit -i
```

比如我的 `~/.zshrc` 文件内容：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201204220141.png)

搜索该文件插件位置，更新插件内容：

```shell
plugins=(... docker docker-compose
)
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201204220531.png)



顺便说一下，强烈建议使用 git 插件

最后刷新一下使之生效：

```shell
source ~/.zshrc
```



## 总结

自动补全功能就可以疯狂利用你的 Tab 键，这比查阅文档要更加快捷，来看看效果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img2020-12-04at22.12.36.gif)



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img2020-12-04at22.20.51.gif)


