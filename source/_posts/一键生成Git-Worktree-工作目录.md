---
title: 一键生成Git Worktree 工作目录
date: 2021-12-01 15:27:06
tags:
  - Git
categories: [Coding, DevOps]
id: git-worktree-with-script
description: 一键生成git worktree 工作目录
keywords: Git, worktree
---

## 前言
上一篇文章[Git Worktree 高级使用](https://dayarch.top/p/git-worktree-in-advance.html) 整体反应不错，这完全是日常开发中可以用到的奇淫技巧。微服务环境下，通常我们都会有多个 repo，高级用法好归好，但每个 repo 都按照高级用法进行配置，还是比较麻烦的，你看这不就有同学发声了嘛

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20211201153240.png)

说者有心，听者有意，那就写个脚本吧

## Git Worktree 脚本
个人不是很擅长写 bash script，磕磕绊绊写了一个 `worktree.sh`，完全执行上一篇文章的整个过程

```bash
#!/bin/bash -e

repo=$1
dir="${repo##*/}"
dir="${dir%.*}"
echo $dir
branch=$2
defaultBranch="${branch:-main}"

mkdir -p $dir
cd $dir

git clone --bare $repo .bare
echo "gitdir: ./.bare" > .git
echo "    fetch = +refs/heads/*:refs/remotes/origin/*" >> ./.bare/config

git worktree add $defaultBranch
```

这个 script 接收两个参数

1. 第一个参数是 **repo**，`https | ssh` 两种方式都可以
2. 第二个参数是 **branch**，不同的 repo 默认的主分支命名可能不一样，Github 现在将主分支命从 master 改为 `main`，所以这里默认值就是 `main`

> 该脚本默认创建 repo 同名文件夹

将 worktree.sh 保存在磁盘目录的某个位置，并授权(最大权限)

```shell
chmod -R 777 worktree.sh
```

接下来就测试一下效果

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img2021-12-01at16.00.47.gif)

假如默认主分支名是 `develop`, 我们只需要添加一个参数就可以了：

```shell
../worktree.sh git@github.com:FraserYu/amend-crash-demo.git develop
```

每次找可执行文件的目录很显然不是一个好的方式，我们需要起个别名，配知道环境变量中，这样方便我们全局使用，根据自己电脑情况打开 `.zshrc` 或 `.bashprofile` 文件（我用的前者）

在里面创建一个别名 `gwt`(感觉这个名字好高端)：

```shell
alias gwt='/Users/rgyb/Documents/projects/personal/worktree.sh'
```

然后刷新一下该文件

```shell
source ～/.zshrc
```

再来看一下效果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img2021-12-01at16.10.42.gif)

到这里，使用 worktree 应该非常简单了吧



## 总结

如果你有多分支切换的各种痛点，学会使用 git worktree，那些问题将不复存在。服务提供全套，脚本放在了

https://github.com/FraserYu/script.git， 有什么问题欢迎留言

