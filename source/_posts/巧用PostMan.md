---
title: 巧用PostMan
tags:
  - PostMan
category: [Coding, Others]
id: postman-advanced-usage
date: 2016-10-27 17:06:12
---

现在越发的认识到巧用工具提高工作效率的重要性，接下来看一下PostMan带来的便利
### 业务情景
在实际工作中，当App或其他渠道来调用Java后台接口的时候，我们通常会用到PostMan这个工具来模拟调用，当需要token来作为验证的时候，当每次请求的token发生变化的时候我们同时要改变其他请求URL中的token内容，这样很麻烦，降低工作效率，PostMan的环境变量解决了这个问题

<!-- more -->

### 配置环境变量
![添加环境变量](http://7xkyc7.com1.z0.glb.clouddn.com/postman_manage_env.png)
and
![增加](http://7xkyc7.com1.z0.glb.clouddn.com/postman_add_get.png)
and
![添加变量](http://7xkyc7.com1.z0.glb.clouddn.com/postman_add_post.png)
and
![token_url](http://7xkyc7.com1.z0.glb.clouddn.com/postman_token_url.png)
and
![script](http://7xkyc7.com1.z0.glb.clouddn.com/postman_token_script.png)
and
![get token](http://7xkyc7.com1.z0.glb.clouddn.com/postman_get_token.png)
and
![user token](http://7xkyc7.com1.z0.glb.clouddn.com/postman_use_token.png)

至此设置环境变量结束，其他功能会陆续补充.
