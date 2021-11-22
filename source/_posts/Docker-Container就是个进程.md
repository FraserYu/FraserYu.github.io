---
title: Docker Container就是个进程
date: 2020-12-22 17:16:18
categories: [Coding, Docker]
tags:
  - Docker
id: docker-container-is-a-process
description: 学习新内容有门槛，降低门槛的办法就是贴近我们已有知识，Docker Container 其实就是一个进程，进程是可以获取操作系统资源的，Container 亦是如此，只不过具体的方式在上层做了封装略有不同
keywords: Container, Docker, Process
---

大家对 Docker 都应该有了或多或少的认识了，相信大家都是从这两张图来粗旷的理解 Docker 及容器概念的

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201221212233.png)



那我们如何更轻松的理解容器 `Container` 呢？说白了



## Container 就是一个进程

比如我们 `run` 一个 `mongo` 的镜像 `image`

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img2020-12-21at21.39.19.gif)

然后我们通过下面命令列举出正在运行的容器 (以下两个命令等同)

```shell
# 旧命令
docker ps
# 新命令
docker container ls
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201221214553.png)



个人建议使用新命令

> 如果你对上述等同命令有所疑惑，或者好奇动图中的命令自动补全是怎么实现的，以及为什么建议使用新命令，请看 [Docker 命令自动补全](https://dayarch.top/p/docker-install-command-completion.html)，在不熟悉命令之前，建议充分利用 TAB 键来查看每个命令的含义，然后结合实际使用场景，慢慢记忆，这样才根深蒂固



我们 run 下面命令：

```shell
# top      -- Display the running processes of a container （这是 TAB 补全给的命令提示说明）
docker container top mongo
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201222160406.png)

从上图中可以看到，PID 为 2292，command 为 `mongod`。既然我们说 container 是一个进程，那我们就应该在 Host 中找得到，执行下面命令

```shell
ps aux | grep mongod
```

查看执行结果：

```shell
rgyb           49927   0.0  0.0  4277516    708 s000  S+    4:06PM   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mongod
```

关于 mongod 只有我们刚刚执行的 `grep mongod` 的操作，并没有上面说的 container，这是为什么？

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201222161238.png)



细心的朋友可能已经从动图中发现我是用 Mac 做的测试，Docker Platform 原生运行在 Linux 上（在 Linux 操作系统中就不会有上述问题，大家可以自行尝试）。我是用 Docker for Mac，其实使用的是在macOS上一个特殊的 xhyve VM中运行的小型(定制)Alpine Linux，所以如果想看到这个进程，我们需要进入到 Mac 的这个 Docker VM



执行下述命令：

```shell
docker run -it --rm --privileged --pid=host justincormack/nsenter1
```

（这里暂不展开说明，有兴趣的可以看看这个 [justincormack/nsenter1 image](https://github.com/justincormack/nsenter1) 到底做了什么，Docker for Windows 也可以用这种方式进入 Docker VM）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201222162637.png)



到这，Container 是个进程算是证明完了，但是老gong，你证明这个有什么用呢？

> 程序员都对进程有基本了解，证明 Container 是个进程，只不过是将一个“新”东西剥开看看本质，并挂靠到你熟悉的内容上

进程就是它可以获取操作系统的哪些资源（网络/磁盘/文件等），当停止进程，也就会自动退出，释放相应资源。所以，接下来只要慢慢探索，一个 Container 中使用了哪些资源，是如何获取资源的。了解了这些，也就慢慢了解了 Docker



大家可以通过下面两个命令了解 Container 的更多详情

- `docker container inspect mongo` 查看Container 的详细信息（JSON 的数据形式）

```shell
Usage:	docker container inspect [OPTIONS] CONTAINER [CONTAINER...]

Display detailed information on one or more containers

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes
------------------------------
docker container inspect mongo
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20201222164416.png)



细节内容非常多，可以简单的看一看（一定有你一眼就能看明白的信息），暂不用深究



-  `docker container stats mongo` 查看资源是用情况（动态统计）

```shell
Usage:	docker container stats [OPTIONS] [CONTAINER...]

Display a live stream of container(s) resource usage statistics

Options:
  -a, --all             Show all containers (default shows just running)
      --format string   Pretty-print images using a Go template
      --no-stream       Disable streaming stats and only pull the first result
      --no-trunc        Do not truncate output
----------------------------      
docker container stats mongo
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img2020-12-22at16.48.57.gif)



## 总结

这里没有上来就和大家死背命令，现在不会，将来也不会。而是通过实际目的，结合命令补全自行查看的方式逐步认识与了解。强烈建议大家安装命令补全，可以尽情使用 TAB，也可以在每个命令后面添加 `--help` 的方式随时查看使用方式

证明 Container 就是一个进程，这样贴近我们已有知识后，学习门槛至少矮了半截吧。最后做个游戏吧，你记住了本文的多少个命令？

## 灵魂追问
1. 为什么资源动态统计 Mem LIMIT 是 1.941GB，这个是在哪里设置的？