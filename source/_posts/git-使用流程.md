---
title: git 使用流程
tags:
  - Git
categories: [Coding, DevOps]
id: git-workflow
date: 2016-06-24 09:44:01
description: Git的基本使用，这几个基本命令会满足日常的Git使用
keywords: Git, git命令, git跟踪

---

### Git分支管理策略
首先share一下Ruan老师的博客内容：[Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html) ，读过之后，理解Master、Develop、临时分支（feature、release、fixbug）的基本意思，这里写的就更加透彻了：[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)， 项目中如果按照这种严格规范来执行版本的管理，应该是会取得成功的结果.  仔细阅读这个workflow深刻的体会：
![Git Branch Model](http://7xkyc7.com1.z0.glb.clouddn.com/git_workflow.png)

### Git 使用实战情景说明
为什么不罗列出来Git的命令列表，因为罗列在这貌似也很难深刻理解命令的含义，也不知道怎样去使用，所以做一些情景案例来说明git命令

<!-- more -->

#### 跟踪GitHub上开源项目
当在Github中遨游一阵，发现了喜欢的项目想在本地研究，同时还想跟踪了解源项目的最新动态，我们需要如下的一些步骤：
1. Fork (分叉)
在跟踪的开源项目上点击Fork，会发现分叉到你的Repo中，此时我们应用如下命令将项目clone到local Repo, https或ssh两种都可以

		$ git clone git@github.com:YOUR-USERNAME/REPO-NAME
		$ git clone https://github.com/YOUR-USERNAME/REPO-NAME
此时可以通过如下命令来查看remote的版本，即你自己的remote Repo的信息

		$ git remote -v
得到如下结果：

		https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
		https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
2. 添加 fork 源仓库的地址
我们的目的不但是要在本地修改自己想要的代码，还要了解最新的源的动态，所以我们需要用如下命令：

		$ git remote add originProject https://github.com/ORIGINAL_OWNER/ORIGINAL_REPO.git
同样应用 **git remote -v**命令会的到如下结果：

		origin https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
		origin https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
		originProject https://github.com/ORIGINAL_OWNER/ORIGINAL_REPO.git(fetch)
		originProject https://github.com/ORIGINAL_OWNER/ORIGINAL_REPO.git (push)
此处的originProject 可以是你取得任意的名字, 就是给远程主机取个名字

		$ git remote add <主机名> <网址>

3. Fetch源
当你要跟踪源动态，就要fetch一份到本地Repo

		$ git fetch originProject
通过如下命令查看分支：

		$ git branch

4. 合并源到本地
首先切换到本地master分支

		$ git checkout master
之后merge源到本地master

		$ git merge originProject/master
这个地方可能会有冲突，需要解决冲突完成merge，最后将merge好的代码push到自己remote

		git push origin master

#### 本地上传代码到GitHub
通常会在本地写点程序，然后放到github来管理，此时我们怎样把本地的代码上传到github呢
1. git初始化
新建一个文件夹，初始化

		$ git init
你的项目code应该在这个文件夹中
2. 添加项目文件到本地Repo
我们需要将本地的code添加到本地的Repo中，add . 代表添加全部文件

		$ git add .
3. 提交文件到本地Repo
添加文件之后，我们需要commit文件到本地Repo，并且标记注释

		$ git commit -m "注释语句"

4. 在github上新建Repo
在github上新建Repo，并且copy Repo的https或ssh URL

		$ git remote add origin https://github.com/YOUR_USERNAME/YOUR_FORK.git
5. Pull远程代码
在push之前我们需要pull remote 代码

		$ git pull origin master
6. Push本地代码到remote
我们已经提交代码到本地Repo，此时我们需要push到remote

		$ git push -u origin master

#### 更改远程URL
在实际场景中我们会遇到Git服务器迁移了，这样我们.git仓库存储的remote repository的地址信息就是错误的，所以需要更改这个URL. 请参考Git Help之[Changing a remote's URL](https://help.github.com/articles/changing-a-remote-s-url/)


如果你觉得你合并后的状态是一团乱麻，想把当前的修改都放弃，你可以用下面的命令回到合并之前的状态：

		$ git reset --hard HEAD

或者你已经把合并后的代码提交，但还是想把它们撒销：

		$ git reset --hard ORIG_HEAD

但是刚才这条命令在某些情况会很危险，如果你把一个已经被另一个分支合并的分支给删了，那么 以后在合并相关的分支时会出错。

### Git远程操作命令详解
有了基本的了解，我们还是在这里引用一下Ruan老师的博客内容，慢慢理解与体会 [Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html) 和 [Git常用命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html?utm_source=tool.lu), 同时还要把廖雪峰老师的Git教程简介：[Git远程操作详解](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)放在这里，都是最好的学习资料，后续我还会继续补充. 有疑问可以留言，加入git实用情景中.
