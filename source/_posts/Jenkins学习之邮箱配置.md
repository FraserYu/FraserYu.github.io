---
title: Jenkins学习之邮箱配置
tags:
  - Jenkins
categories: [Coding, DevOps]
id: jenkins-config-mail
date: 2017-01-05 15:05:18
---

### 使用场景
开发人员build project 之后，build结果无论是成功还是失败，都要及时的通知组内其他成员了解最新情况，邮件通知这时候就派上用场，恰巧 Jenkins 提供了这么一个功能，不过该功能还是过于单一，如不能编写email template 来格式化邮件内容，但**Extended E-mail Notification**（Jenkins 邮件插件）实现了更高级的功能，接下来逐步看一下 Jenkins 邮件功能的配置

### 配置邮件服务器
以管理员身份登录，在 Jenkins 首页click **Manage Jenkins**, 然后 click **Configure System**, 下拉到页面的最底部 **E-mail Notification** 处：
<!-- more -->
![E-mail Notification](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_mail1.png)

此处我用了网易**yeah.net**的邮箱，User Name(邮箱账号)，Password(邮箱密码)，在Test e-mail recipient(接收邮件)，不过配置了这些仅仅是配置邮件服务器地址、账号和密码，但是jenkins不知道采用哪个邮箱去发送，所以我们需要在该处配置发送的邮箱，并且要和管理员的邮箱一致

![admin](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_mail2.png)

到此点击一下**Test configuration** button 看一下测试结果，如果测试不成功要检查一下管理员邮箱是否有开通 SMTP服务，登录你设置的邮箱开通SMTP

![smtp](http://7xkyc7.com1.z0.glb.clouddn.com/mail3.png)

设置完之后测试结果为**成功**

当我们在project 中配置build之后的行为时，可以添加邮件通知，这样在接收人列表中的人都可以接受到build的结果：

![build](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_mail4.png)

build 一下项目，看一下有没有收到邮件通知吧.

### 安装Email插件
Jenkins 首页 —— Manage Jenkins —— Manage Plugins，安装该插件
![plugin](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_mail5.png)

重新启动Jenkins, 可以在刚刚配置邮件的上方看到已安装好的email 插件
![mail plugin](http://7xkyc7.com1.z0.glb.clouddn.com/jenkins_mail6.png)

这样编辑**Default Content** 字段来编写email template, 支持 HTML 标签.  这样在项目构建后行为中选择**Editable Email Notificatioin** 来发布build的结果

**NOTE:** 大家看不懂的字段多多点开字段右侧的问号，里面的说明还是非常清晰的. 邮件插件的其他小功能也请自行研究.
