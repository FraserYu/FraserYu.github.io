---
title: Hexo在github上搭建自己的博客
tags: 
    - GitHub Pages
categories: [Coding, Others]
id: build-github-pages-blog
date: 2015-08-09 20:01:29
description: 通过Hexo搭建属于自己的博客，静态发布博客，不再受网络限制
keywords: Hexo,搭建博客,教程
icons: [fas fa-fire accent]
---


引言
-------
长时间在某平台上写博客记录些什么，但是近期该平台陆陆续续出现一些小问题，让写博客这么愉快的事情总是充斥着各种插曲，稍稍不爽，于是乎开始准备搭建自己的的博客，通过查阅资料用了两天时间初步搭建起来，收获颇丰，所以趁热打铁记录整个过程，搭建过程参考了非常多前辈们的blog，先真诚的道一声**“感谢”**，文章末尾会给出相关的参考链接。可搭建博客的框架很多，参考[静态博客框架对比](http://www.chinaz.com/special/static-blog/index.html?qq-pf-to=pcqq.c2c)，选择你看上眼的，本文选择[Hexo](https://hexo.io/docs/)，为什么选择它，我只能说对比之后喜欢.



为什么选择github平台？
-----------------------------------
为什么选择**github**平台，我们通过下面的两组问题就可以得到答案了.
> 博友们是否在编写博客的时候遇到过以下问题：

> 1. 博客平台升级维护
> 2. 没有网络（比博客升级和维护还惨）
> 3. 不够定制化

-----------------------
 > github上搭建自己的博客当然是有很多好处的：

> 1. 学着用github，享受github的便利，上面有很多大牛，眼界会开阔很多
> 2. 顺便看看github工作原理，最好的团队协作流程
> 3. 自我认为开源是一种趋势，github上有良好的开源氛围与精神
> 4. 定制化的博客，可以按照自己的喜好来做更改
> 5. 即便离线我也可以按照Markdown语法来编写博客，有网络时上传即可
> 6. 一份使用 Markdown 格式撰写的文件应该可以直接以纯文本发布，并且看起来不会像是由许多标签或是格式指令所构成，可读性强

搭建过程
-------------
按照下面的过程逐步进行，细心理解领会，你会发现搭建过程很简单, 下文都是在Windows 环境进行搭建。

### 前期准备工作
安装Hexo很简单，但是需要下面两样东东的支持：

+ Node.js
+ Git

在Windows 环境安装真的是太简单了，基本上是无脑**下一步**

<!-- more -->

### Hexo安装
在某个磁盘目录下新建一个文件夹，然后鼠标右键单击**【Git Bash】**, 进入下图：
![单击Git Bash](http://7xkyc7.com1.z0.glb.clouddn.com/blog_1_文件夹下单击git_bash.png)

执行下面命令安装Hexo：
> $ npm install -g hexo-cli

查看一下安装的Hexo版本（本人安装的是3.1.1）
![查看hexo版本](http://7xkyc7.com1.z0.glb.clouddn.com/blog_1_查看hexo版本.png)

hexo的命令要怎么用？必须help命令来帮忙
![hexo help命令](http://7xkyc7.com1.z0.glb.clouddn.com/blog_1_hexo%20help命令.png)

安装好后就需要初始化Hexo来创建项目了
> $ hexo init

该文件夹下会生成以下文件：

> + scaffolds 脚手架，也就是一个工具模板
> + scripts 写文件的js，扩展hexo的功能
> + source 存放博客正文内容
> + source/_drafts 草稿箱
> + source/_posts 文件箱
> + themes 存放皮肤的目录
> + themes/landscape 默认的皮肤
> + _config.yml 全局的配置文件
> + db.json 静态常量

项目创建完毕，就要启动了
> $ hexo server

这个地方可能会遇到错误， 因为server在Hexo 3的版本被分离出来，需要[单独安装](https://hexo.io/docs/server.html)：
> $ npm install hexo-server --save

这样重新启动就好了.

这时端口4000被打开了，我们能过浏览器打址，http://localhost:4000/
![hexo默认主题](http://7xkyc7.com1.z0.glb.clouddn.com/blog_1_hexo默认主题.png)

我们可以使用下面命令来创建新page来写博客了
> $ hexo new myNewPage

创建好myNewPage.md文件会自动放在**source/_posts**目录下，生成的md后缀的文件需要采用[Markdown](http://www.appinn.com/markdown/#philosophy)进行编写，然后刷新页面，你就会发现你的新文章了，是不是有点小激动，还有更好的，往下看。

### 更改Hexo主题

安装好的Hexo我们都是一样的在**source/themes**默认主题**landscape**, 是不是觉得不够爽， 那就[选择主题](https://github.com/hexojs/hexo/wiki/Themes), 这里面应该有你想要的那一款吧。

执行下面命令将选好的主题clone到自己的hexo空间
> $ git clone https://github.com/A-limon/pacman.git themes/pacman

这样你会发现pacman这个主题就在你的themes文件下了，想让自己博客指定主题，只需要修改hexo的主题文件_config.yml 中的theme属性即可
> theme: pacman

重新启动，刷新你的页面，主题变了吧。

### 配置和使用Github

我们如何让本地git项目与远程的github建立联系呢？用**SSH keys**
开始菜单处单击【Git Bash】**

首先我们需要检查你电脑上现有的ssh key：
> $ cd ~/.ssh

如果提示：No such file or directory 说明你是第一次使用git， 接下来需要生成**SSH keys**
> $ ssh-keygen -t rsa -C "邮件地址@youremail.com"
> Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>

**注意:** 此处的邮箱地址，你可以输入自己的邮箱地址；注意2: 此处的「-C」的是大写的「C」

系统要求输入密码和确认密码：
> \$ Enter passphrase (empty for no passphrase):<输入加密串>
> \$ Enter same passphrase again:<再次输入加密串>

最后看到下面界面就生成了**SSH keys**
![生成sshkey](http://7xkyc7.com1.z0.glb.clouddn.com/blog_1_生成sshkey.png)

按照log提示找到id_rsa.pub文件，复制里面的内容，然后将**SSH keys**添加到github上，登陆github系统。点击右上角的 Account Settings—->SSH Public keys —-> add another public keys

可以通过下面命令来进行测试是否成功
> $ ssh -T git@github.com

现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。

Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。
> \$ git config --global user.name "your name"
> \$ git config --global user.email  "username@gmail.com"


至此**SSH keys**设置成功

### 发布博客到Github

我们的终极目标是将博客发布到github上，我们还是需要有一些准备工作要做。在github上新建username.github.io的repository，至于为什么请参考[github pages](https://pages.github.com/)

本地新建一个文件夹，该文件夹下右键单击**【Git Bash】** 执行命令
> $ git clone git@github.com:username/username.github.io.git

这样就在本地生成了自己的repository

怎样发布到github上去，我们同样要修改hexo下的_config.yml文件，在最后deployment处增加这样两行

> deploy:
> type: github
> repo: git@github.com:username/username.github.io.git

然后执行命令进行静态化，将所有文件静态化到public文件夹下
> $ hexo generate

然后进行部署
> $ hexo deploy

这个地方可能又遇到问题了，提示【hexo ERROR Deployer not found: github】，同样是3以上版本的问题，按照下面操作即可：

> $ npm install hexo-deployer-git --save

然后将上面说到的部署文件的type由**github**改为**git**
> deploy:
> type: git
> repo: git@github.com:username/username.github.io.git

重新生成静态文件，然后发布，这样在浏览器中输入username.github.io就可以到你在github上的博客了， 有没有小激动？ 最后小伙伴们哪里看不懂或哪里有问题欢迎留言，我进行补充或回答, 更多问题可以参考官网文档[Docs](https://hexo.io/docs/) .

最后真的感激前辈们的博文指点，下面提供我参考过的一些链接：

1. [如何搭建一个独立博客——简明GithubPages与Hexo教程](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/)
2. [Hexo在github上构建免费的Web应用](http://blog.fens.me/hexo-blog-github/)
3. [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
4. [用Jekyll在github上写博客——《搭建一个免费的，无限流量的Blog》的注脚](http://www.cnblogs.com/fstang/p/3308964.html)
