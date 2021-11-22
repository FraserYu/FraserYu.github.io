---
title: 保持清洁的Git提交记录，三招就够了
date: 2021-11-21 10:29:52
tags:
  - Git
categories: [Coding, DevOps]
id: three-ways-to-keep-git-log-clear
description: 保持Git log清洁，是程序员的基本素养
keywords: Git, amend，rebase
---

## 背景

大家都有学习如何规范简洁的**编写代码**，但却很少学习如何规范简洁的**提交代码**。现在大家基本上都用 Git 作为源码管理的工具，Git 提供了极大的灵活性，我们按照各种 workflow 来提交/合并 code，这种灵活性把控不好，也会带来很多问题



最常见的问题就是乱成一团的 git log history，那真的是老太太的裹脚布, 又臭又长, 个人极其不喜欢这种 log 

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20211120204638.png)



造成这个问题的根本原因就是随意提交代码。

> 代码都提交了，那还有什么办法拯救吗？三个锦囊，就可以完美解决了

## 善用 git commit --amend

这个命令的帮助文档是这样描述的：

```shell
--amend               amend previous commit
```

也就是说，它可以帮助我们修改**最后一次提交**

> 既可以修改我们提交的 message，又可以修改我们提交的文件，最后还会替换最后一个 commit-id



我们可能会在某次提交的时候遗漏了某个文件，当我们再次提交就可能会多处一个无用的 commit-id，大家都这样做，git log 慢慢就会乱的无法追踪完整功能了

假设我们有这样一段 log 信息

```shell
* 98a75af (HEAD -> feature/JIRA123-amend-test) feat: [JIRA123] add feature 1.2
* 119f86e feat: [JIRA123] add feature 1.1
* 5dd0ad3 feat: [JIRA123] add feature 1
* c69f53d (origin/main, origin/feature/JIRA123-amend-test, origin/HEAD, main) Initial commit
```

假设我们要修改最后一个 log message，就可以使用下面命令：

```shell
git commit --amend -m "feat: [JIRA123] add feature 1.2 and 1.3"
```

我们再来看一下 log 信息, 可以发现，我们用新的 commit-id `5e354d1` 替换了旧的 commit-id `98a75af`, 修改了 message，并没有增加节点

```shell
* 5e354d1 (HEAD -> feature/JIRA123-amend-test) feat: [JIRA123] add feature 1.2 and 1.3
* 119f86e feat: [JIRA123] add feature 1.1
* 5dd0ad3 feat: [JIRA123] add feature 1
* c69f53d (origin/main, origin/feature/JIRA123-amend-test, origin/HEAD, main) Initial commit
```

现在我们的 repo 中文件是这样的：

```shell
.
├── README.md
└── feat1.txt

0 directories, 2 files
```

假设我们提交 `feature 1.3` 的时候，忘记了一个配置文件 `config.yaml`, 不想修改 log，不想添加新的 commit-id，那下面的这个命令就非常好用了

```shell
echo "feature 1.3 config info" > config.yaml
git add .
git commit --amend --no-edit
```

`git commit --amend --no-edit` 就是灵魂所在了，来看一下当前的 repo 文件：

```shell
.
├── README.md
├── config.yaml
└── feat1.txt

0 directories, 3 files
```

再来看一下 git log

```shell
* 247572e (HEAD -> feature/JIRA123-amend-test) feat: [JIRA123] add feature 1.2 and 1.3
* 119f86e feat: [JIRA123] add feature 1.1
* 5dd0ad3 feat: [JIRA123] add feature 1
* c69f53d (origin/main, origin/feature/JIRA123-amend-test, origin/HEAD, main) Initial commit
```

知道这个技巧，就可以确保我们的每次提交都包含有效的信息了。一张图描述这个过程就是这个样子了：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20211120212936.png)

有了 `--no-edit` 的 buff 加成，威力更大一些

## 善用 git rebase -i

可以看着，上面的 log 都是在开发 feature1，我们在把 feature 分支 merge 到 main 分支之前，还是应该继续合并 log commit 节点的，这就用到了 

```shell
git rebase -i HEAD~n
```

其中 n 代表最后几个提交，上面我们针对 feature 1 有三个提交，所以就可以使用：

```shell
git rebase -i HEAD~3
```

运行后，会显示一个 vim 编辑器，内容如下：

```shell
  1 pick 5dd0ad3 feat: [JIRA123] add feature 1
  2 pick 119f86e feat: [JIRA123] add feature 1.1
  3 pick 247572e feat: [JIRA123] add feature 1.2 and 1.3
  4
  5 # Rebase c69f53d..247572e onto c69f53d (3 commands)
  6 #
  7 # Commands:
  8 # p, pick <commit> = use commit
  9 # r, reword <commit> = use commit, but edit the commit message
 10 # e, edit <commit> = use commit, but stop for amending
 11 # s, squash <commit> = use commit, but meld into previous commit
 12 # f, fixup <commit> = like "squash", but discard this commit's log message
 13 # x, exec <command> = run command (the rest of the line) using shell
 14 # d, drop <commit> = remove commit
 15 # l, label <label> = label current HEAD with a name
 16 # t, reset <label> = reset HEAD to a label
 17 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 18 # .       create a merge commit using the original merge commit's
 19 # .       message (or the oneline, if no original merge commit was
 20 # .       specified). Use -c <commit> to reword the commit message.
 21 #
 22 # These lines can be re-ordered; they are executed from top to bottom.
 23 #
 24 # If you remove a line here THAT COMMIT WILL BE LOST.
 25 #
 26 #   However, if you remove everything, the rebase will be aborted.
 27 #
 28 #
 29 # Note that empty commits are commented out
```

合并 commit-id 最常用的是 `squash` 和 `fixup`, 前者包含 commit message，后者不包含，这里使用 fixup, 然后 `:wq` 退出

```shell
  1 pick 5dd0ad3 feat: [JIRA123] add feature 1
  2 fixup 119f86e feat: [JIRA123] add feature 1.1
  3 fixup 247572e feat: [JIRA123] add feature 1.2 and 1.3
```

我们再来看一下 log, 这就非常清晰了

```shell
* 41cd711 (HEAD -> feature/JIRA123-amend-test) feat: [JIRA123] add feature 1
* c69f53d (origin/main, origin/feature/JIRA123-amend-test, origin/HEAD, main) Initial commit
```

## 善用 rebase

上面的 feature1 已经完整的开发完了，main 分支也有了其他人的更新，在将 feature merge 回 main 分支之前，以防代码有冲突，需要先将 main 分支的内容合并到 feature 中，如果用 merge 命令，就会多处一个 merge 节点，log history 中也会出现拐点，并不是线性的，所以这里我们可以在 feature 分支上使用 rebase 命令

```shell
git pull origin main --rebase
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20211120215234.png)

pull 命令的背后是自动帮我们做 merge 的，但是这里以 rebase 的形式，再来看一下 log

```shell
* d40daa6 (HEAD -> feature/JIRA123-amend-test) feat: [JIRA123] add feature 1
* 446f463 (origin/main, origin/HEAD) Create main.properties
* c69f53d (origin/feature/JIRA123-amend-test, main) Initial commit
```

我们的 feature1 功能 `on top of` main 的提交节点，还是保持线性，接下来就可以 push 代码，然后提 PR，将你的 feature merge 到 main 分支了

简单描述 merge 和 rebase 的区别就是这样的：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-imggit1.gif)

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-imggit2.gif)

我这里使用 `git pull origin main --rebase` 省略了切换 main 并拉取最新内容再切回来的过程，一步到位，背后的原理都是上图展示的这样



使用 rebase 是要遵守一个黄金法则的，这个之前有说过，就不再是赘述了



## 总结

有了这三个锦囊，相信大家的 git log 都无比的清晰，如果你还不知道，完全可以用起来，如果你的组内成员不知道，你完全可以推广起来，这样的 repo 看起来才更健康

接下来会介绍一个多分支切换互不影响的锦囊