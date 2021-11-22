---
title: Jenkins 动态使用分支名称
date: 2021-03-28 20:33:56
tags:
    - Jenkins
categories: [Coding, DevOps]
id: jenkins-dynamic-using-git-branch
music:
    enable: true
    server: netease
    type: song
    id: 403710591
---

## 前言

在上一篇 [Jenkins 使用环境变量](https://dayarch.top/p/jenkins-environment-variables.html) 中，帮助大家使用一条 Docker 命令就可以快速玩转 Jenkins，同时用最简单的方式解释了 Jenkins 中让人混乱的环境变量，本文还是接着变量说点事情



一般成熟的项目流程都会通过 Jenkins Pipeline 来做 CI 部分，在默认 Jenkins 环境配置中，Jenkins Pipeline 分为两种：

1. Pipeline （单分支 Pipeline）
2. Multibranch Pipeline （多分支 Pipeline）

如下图：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210317100822.png)



如果使用了`多分支 Pipeline`，就不会存在动态使用分支名称的问题了。如果你想使用单分支 Pipeline，又想动态使用分支，那本文就派上用场了



## Jenkins 中动态使用分支名称

新建单分支 Pipeline后，可以在界面中看到 `This project is parameterized` , 勾选上，然后添加 String 类型的参数，如下图所示，String 类型的参数名称为 `BranchName`, 默认值是 `master` 分支



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210317101629.png)



向下滚动，来配置 Jenkins Pipeline，其中在指定 Jenkins Pipeline 分支的位置，就可以使用上面刚刚创建的变量 `BranchName` 了，如下图所示，配置成

```groovy
*/${BranchName}
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210317102111.png)



> 注意：如果勾选 Lightweight checkout 可能会出现下面的 Error

```shell
stderr: fatal: Couldn't find remote ref refs/heads/${BranchName}
```



以这种方式，Jenkins Pipeline 会动态根据分支名称来拉取代码，同样也可以在 Jenkinsfile 中动态使用刚刚创建好的 `BranchName`

```groovy
pipeline {
	...
	
	parameters {
		string(name: 'BranchName', defaultValue: 'master', description: null)
	}
	
	stages {
		stage ('Test Branch Name') {
			steps {
				echo "${env.BranchName}"
			}
		}
	}
}
```

当点击左侧 Build with Parameters 后，我们就可以动态输入分支名称来运行 job 了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host@master/blog-img20210328203212.png)

## 总结

在 Jenkins 中，其实这是一种很常见的动态使用参数的方式，config 的其他参数也可以动态引用变量，大大增加灵活性，如果你要维护 JenkinsPipelie 相关的内容，你大概率会遇到这种需求，这个小技巧收入囊中吧
