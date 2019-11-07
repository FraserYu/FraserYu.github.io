---
title: Git Ignoring Files
tags:
  - Git
categories: [Coding, DevOps]
id: git-ignore
date: 2016-10-15 09:32:43
description: 配置好Git ignore文件是保持远程代码清晰的关键
keywords: Git,ignore,ignore file

---
### 前言
以下内容根据[Git Help](https://help.github.com/articles/ignoring-files/)以及[Git Docs](https://git-scm.com/docs/gitignore) 整理翻译
### Ignoring files(忽略文件)
有时候会有一些我们不想Git 提交一些文件到GitHub/GitLab，下面有几种方式让我们告诉Git来忽略哪些文件.

<!-- more -->

#### Create a local .gitignore（创建局部/本地的.gitignore文件）
>如果在local repository（仓库）中创建一个名为**.gitignore**的文件，在你commit（提交）动作之前，Git 将会读取***.gitignore**文件去判断哪些文件或者目录需要忽略掉. 同时这个文件应该push到remote repository, 这样clone该repository的用户都会共享ignore的规则. 当然GitHub上维护了一个官方的、针对许多流行的操作系统、环境、语言的**.gitignore** [文件样例](https://github.com/github/gitignore)

在终端，定位到Git repository所在位置，输入以下命令来创建**.gitignore**文件（当然在windows环境可以直接在仓库位置鼠标右键创建名为.gitignore的文件即可）

	touch .gitignore

**注意**：如果某个文件已经被提交，此时你想让git 忽略它，这是你把这个文件添加到**.gitignore**文件中，git也不会忽略它，此时我们必须先untrack这个文件，这样Git才会忽略这个文件

	git rm --cached FILENAME

#### Create a global .gitignore（创建全局.gitignore文件）
>通常我们会有多个repository，如果多个仓库需要的规则类似，我们还需要为每一个repository都新建一个**.gitignore** 文件吗？显然我们此刻需要一个全局的方式来管控所有的repository. 有两种方式来创建该全局文件：

1. 当安装了Git之后，在git 主目录(Windows环境C:\Users\Fraser\)下会有一个**.gitconfig**文件，里面指定了如下内容：

		[user]
		[core]
			excludesfile = D:\\Users\\fraser\\Documents\\gitignore_global.txt
		[user]
		[user]
			name = XXXXX
			email = XXXXX@qq.com
也就是说我们把规则定义在**gitignore_global.txt**文件中就好. 或者指定其他文件， 这个文件是windows自动生成的

2. 通过命令指定**.gitignore_global**的文件

		git config --global core.excludesfile ~/.gitignore_global
通常.gitignore_global文件也是放在主目录下.

#### Explicit repository excludes(明确仓库忽略的文件)
> 如果你不想创建一个**.gitignore**文件并且share给其他人，你可以创建一个不需要提交到repository的规则，比如我使用IntelliJ IDEA编辑器，编辑器会生成一些相应文件，但是同事使用Eclipse，也就是说IntelliJ IDEA编辑器生成的文件规则并不适用Eclipse，所以我们需要用另外方法来处理这种情况.

用编辑器打开**.git/info/exclude**，将规则加载这里即可,这个git是不会提交的.

### Ignore 规则说明
gitignore - Specifies intentionally untracked files to ignore

#### 描述
上面我们说到了我们可以采用多种方式来让git ignore文件，同时配置了这些文件，优先级是什么呢？
1. **Level1**
命令行执行的命令(很少用)
2. **Level2**
仓库中的**.gitignore** 文件
3. **Level3**
$GIT_DIR/info/exclude
4. **Level4**
全局的配置：core.excludesFile

#### PATTERN FORMAT规则说明
1. 空行，没有任何实际ignore意义，只是作为可读性的分隔符
2. "#"号开头代表该行是注释，如果在前面加上“\”，则转义成以“#”开头
3. “!” 是重新让git check in 某文件的意思，但是该文件的父目录被exclude，re-include 该文件Git是做不到的. 同样可以加上"\"来转义，**!important!.txt**和**\!important!.txt**
4. 以“/”结束，如**foo/** 会匹配foo以及foo下面的子文件夹，但是不会匹配foo/下面的任何文件.
5. \* 是通配符，如*.c，会匹配所有.c为后缀的文件
6. \** 的含义非常多：
+ A leading "\**" followed by a slash means match in all directories. For example, "**/foo" matches file or directory "foo" anywhere, the same as pattern "foo". "**/foo/bar" matches file or directory "bar" anywhere that is directly under directory "foo".
+ A trailing "/\**" matches everything inside. For example, "abc/**" matches all files inside directory "abc", relative to the location of the .gitignore file, with infinite depth.
+ A slash followed by two consecutive asterisks then a slash matches zero or more directories. For example, "a/**/b" matches "a/b", "a/x/b", "a/x/y/b" and so on.
