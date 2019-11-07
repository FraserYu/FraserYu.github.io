---
title: Git配置多个账号（GitLab和GitHub）
tags:
  - Git
categories: [Coding, DevOps]
id: git-multiple-account
date: 2016-08-09 10:29:32
description: Git多账号管理详细教程
keywords: Git,多账号,Github,GitLab

---

### 背景
现在好多公司用GitLab作为项目管理工具，自己也会有开源项目在GitHub上，按照以往的配置全局的用户貌似在这种情形下就没有作用了（公司的项目需要用公司邮箱，自己的项目需要用自己的邮箱），此时就要做Git多账户的管理与设置。

### Git支持的协议
Git主要支持四中协议： file:// , git://,  http(s)://, ssh://, 这里主要说明一下http(s)和ssh协议的区别。

#### https 和 SSH 的区别
> 1. 前者可以随意克隆github上的项目，而不管是谁的；而后者则是你必须是你要克隆的项目的拥有者或管理员，且需要先添加 SSH key (这里就是添加公钥，取得管理员的信任)，否则无法克隆。
> 2. https url 在push的时候是需要验证用户名和密码的，而 SSH 在push的时候，是不需要输入用户名的，如果配置SSH key的时候设置了密码，则需要输入密码的，否则直接是不需要输入密码的。

<!-- more -->

### Git设置多账户
#### 检查本机是否已有SSH Key

	$ cd ~/.ssh
	$ ls

如果存在会有两个秘钥文件id_rsa(私钥)和id_rsa.pub(公钥)，如果没有则需要下一步的创建。

#### 创建SSH Key
输入如下命令：

	$ ssh-keygen -t rsa -C "your_email@example.com"

参数解析：
> + -t 指定密钥类型，默认是 rsa ，可以省略。
> + -C 设置注释文字，比如邮箱。
> + -f 指定密钥文件存储文件名。

以上代码省略了 -f 参数，因此，运行上面那条命令后会让你输入一个文件名，用于保存刚才生成的 SSH key 代码，如：

	Generating public/private rsa key pair.
	# Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter]

当然，你也可以不输入文件名，直接车，使用默认文件名（推荐），那么就会生成 id_rsa 和 id_rsa.pub 两个秘钥文件。

接着又会提示你输入两次密码（该密码是你push文件的时候要输入的密码，而不是github管理者的密码），这样别人用你的机器没有密码是不可能随便push代码的，一种安全性的体现，当然，你也可以不输入密码，直接按回车。那么push的时候就不需要输入密码，直接提交到github上了。

	Enter passphrase (empty for no passphrase):
	# Enter same passphrase again:

在此案例中，就需要输入两个email来创建个相应的SSH Key:

	$ ssh-keygen -t rsa -C "personal@example.com" (个人邮箱)
	$ ssh-keygen -t rsa -C "company@example.com" (公司邮箱)

此时在.ssh 目录下应该会有下面四个文件：

	id_rsa  id_rsa.pub  id_rsa_company  id_rsa_company.pub

将公钥id_rsa.pub和id_rsa_company.pub 分别添加到GitHub和GitLab上，怎样添加请自行查询，至此，我们已经有两个用户（可以更多），怎样指定要连接GitHub还是GitLab呢？这样就需要解决多用户的权限问题。

#### 多用户权限问题
GitHub/GitLab使用SSH与客户端连接。如果是单用户（A），生成密钥对后，将公钥保存至 GitHub/GitLab，每次连接时SSH客户端发送本地私钥（默认~/.ssh/id_rsa）到服务端验证。单用户情况下，连接的服务器上保存的公钥和发送的私钥自然是配对的。但是如果是 多用户 （A,B），我们在连接到第二个帐号时，B保存的是自己的公钥，但是SSH客户端依然发送默认私钥，即A的私钥，那么这个验证自然无法通过, 所以需要让SSH客户端发现B的私钥。

1. 查看系统SSH Key

		$ ssh-add -l
如果提示：

		$ Could not open a connection to your authentication agent.
可以输入如下命令然后重新输入：**ssh-add -l**

		$ ssh-agent bash
> **ssh-agent bash命令解释：**
> ssh-agent是专为既令人愉快又安全的处理RSA和DSA密钥而设计的特殊程序，不同于ssh，ssh-agent是个长时间持续运行的守护进程（daemon），设计它的唯一目的就是对解密的专用密钥进行**高速缓存**。ssh包含的内建支持允许它同ssh-agent通信，允许ssh不必每次新连接时都提示您要密码才能获取解密的专用密钥。对于ssh-agent，您只要使用ssh-add把专用密钥添加到ssh-agent的高速缓存中。这是个一次性过程；用过ssh-add之后，ssh将从ssh-agent获取您的专用密钥，而不会提示要密码短语来烦您了。
如果出现如下提示信息说明没有SSH Key在代理中

		$ The agent has no identities.
下面可以手动加入SSH Key 到代理中：

		$ ssh-add ~/.ssh/id_rsa_company
可以使用如下命令remove SSH Key

		$ ssh-add -D

2. 配置config 路由文件
config文件即绑定私钥和远程服务器的关系，以我的config文件为例：

		# 该文件用于配置私钥对应的服务器
		# 默认自己GitHub的配置
		Host github
		 HostName github.com
		 User FraserYu
		 PreferredAuthentications publickey
		 IdentityFile ~/.ssh/id_rsa

		# 公司所用GitLab的配置
		Host gitlab
		 HostName gitlab.com #(可以是域名/IP地址)
		 User Fraser
		 PreferredAuthentications publickey
		 IdentityFile ~/.ssh/id_rsa_company		 

3. 测试
输入如下命令进行GitLab和GitHub的测试(注意命令中用了Host的别名)：

		$ ssh -T git@github
		$ ssh -T git@gitlab
		$ git clone git@github:username/project.git

4. 用户名邮箱配置
Git全局配置和单个仓库的用户名邮箱配置，学习git的时候, 大家刚开始使用之前都配置了一个全局的用户名和邮箱

		$ git config --global user.name "github's Name"
		$ git config --global user.email "github@xx.com"
		$ git config --list
如果你公司的项目是放在自建的gitlab上面, 如果你不进行配置用户名和邮箱的话, 则会使用全局的, 这个时候是错误的, 正确的做法是针对公司的项目, 在项目根目录下进行单独配置

		$ git config user.name "gitlab's Name"
		$ git config user.email "gitlab@xx.com"
		$ git config --list
 git config --list查看当前配置, 在当前项目下面查看的配置是全局配置+当前项目的配置, 使用的时候会优先使用当前项目的配置

5. 除了默认的id_rsa是被git默认读取，我们每次打开终端都需要执行 **ssh-add ～/.ssh/id_rsa_company**， 这样十分麻烦，以MacOs为🌰，我们可以将其添加到 KeyChain Access， 执行命令 **ssh-add -K [path/to/your/ssh-key]**，  如：ssh-add -K ～/.ssh/id_rsa_company 即可
