---
title: Java后端的我在学Node.js 你敢信？
date: 2020-02-27 08:40:40
  - IntelliJ IDEA
id: start-to-learn-nodejs
categories: [Coding, Node.js]
description: 作为一名纯java后端的人员，由于项目需要，我要开始学Node.js 了，对于要接触陌生的东西，内心其实是抗拒的，但是我提醒自己要跳出舒适区，多学一些东西，本文就是记录我是如何开始接触和学习Node.js 的
keywords: Node.js, Node.js思维导图, 学习方法, 极客时间, 慕课网
thumbnail: https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200227085626.png
---



项目需要，我需要储备一些Node.js 相关的知识了，整体感觉是一件好事



## 背景

从前，我也写过前端，不过那会最多用到HTML5 +jQuery相关技术。也没有准确的时间点来划分，我就忽忽悠悠的专注于Java后端了

![img](https://i03piccdn.sogoucdn.com/8c2c3c7fdfc3b299)

像现在的大前端Angular、VUE、REACT这些大前端框架我只是略有了解，完全没有用过，用Node作为中间服务器也是相对陌生状态，项目中需要，我是一定不会放过这个可以掉头发的良机（想法很简答， 跳出自己的舒适区）

![img](https://i01piccdn.sogoucdn.com/21c3784942552f15)



毕竟要在实际项目中写Node代码的，这么严峻的问题我是要认真对待的，要不然会让队友消耗太多 WTFs/min 能量（这是什么含义？请看【[读《Clean Code 代码整洁之道》之感悟](https://mp.weixin.qq.com/s/K3fHhXbSNAMVEQF_2NHg9g)】），作为一个小白，通过学习一小段时间还是有所收获的，下面主要说一下我的学习过程，**请有经验的大佬多指正，让俺少走点弯路多留几根头发**



## Node.js 学习

和大家一样，我也是各种上网查阅资料，这里将目前的学习过程做个简单的记录

### [Node.js 官网](https://nodejs.org/en/)

学习一样东西，第一步自然是先打开官网，这里根正苗红，与众不同



优点：很快可以有一个相对直观的了解，文档与API齐全

缺点：这里的苗太正了，【事件驱动、构建在V8引擎】等陌生词汇直接让我眼冒金星，这类词语就好比小时候父母对我们说的词汇，只有长大之后才会明白



面对这些陌生的内容，我并不是很担心（内心懵的一P），相信长大后会明白的，随手毫不留情把网站关掉



### tutorialspoint

[tutorialspoint](https://www.tutorialspoint.com/index.htm) 是我非常喜欢的学习网站，上面有很多技术教程，先来个截图瞧瞧

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200225080943.png)



优点：教程内容简单明了，循序渐进，案例代码齐全

缺点：这是一个英文网站



（**Tips**: 看英文文档应该成为我们的必备技能之一）



不要被英文网站吓到，其实没什么陌生词汇，按照[Node.js 教程](https://www.tutorialspoint.com/nodejs/index.htm) 从头到尾撸了一遍，有了相对全面的了解，知道了基本骨架内容, 一边撸一遍记笔记（忽略着潦草的字）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgWechatIMG197.jpeg)



中途还是有很多不理解的内容，都用黄色的笔圈了出来，放在后面慢慢查阅，**紧跟主线，以防自己跑偏**



### 极客时间/慕课网

教学视频是前辈多年浓缩的精华，有了一些基础概念，我会选择站在巨人的肩膀上整体看一下。日常主要通过【极客时间】和【慕课网】这两个网站搜索视频资料，其中还在极客时间购买了下面的这门课

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgWechatIMG199.jpeg)



这门课的基础知识讲的不算多，前序章节以石头剪刀布的简单游戏来说明Node.js一些特性，后面实战实现极客时间【详情/列表/评论】等页面



杨浩老师讲解的很清晰，中间穿插着很多冷幽默，但对于小白的我来说，这门课看早了，里面讲的很多内容我还不能理解，但是里面说的一些构建思想，比如CommonJS，这些内容还是十分受用的，视频课程目前看了超过2/3，我选择按下暂停键，觉得有必要写一些Demo来使用上面学到的一些知识点了，我相信当我再回头过来看这个视频的收获一定是不一样的





### Github

欢迎来到世界最大的同性交友网站，想不到要写一个什么样的Demo，于是来这里搜索一下，还真找到了自建博客的学习性项目[N-Blog](https://github.com/nswbmw/N-blog) （你有什么需求完全可以先来上面找找轮子的）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200225092053.png)



找项目有几看：

1. commits的活跃度
2. 星标/fork数量
3. README/wiki编写质量
4. Issues处理情况



按照上述几个标准确认过眼神，这是我要找的姑娘



该项目主要应用下面几项技术：

- Node.js: 8.9.1
- MongoDB: 3.4.10
- Express: 4.16.2



看技术栈，就可以认定，这是接近真实项目的存在，于是，按部就班的**敲代码，敲代码，敲代码**（不是复制粘贴），因为敲代码就可能会出现错误，这样也在学习中逐步学会了调试

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200225092721.png)



通过两天时间也终于做出了最终的效果

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200225093129.png)



麻雀虽小，五脏俱全，在实现过程遇到了太多问题，比如：

- npm全局安装权限问题
- Homebrew 安装MongoDB不支持services指令问题
- 陌生的API使用问题
- ......



在学习的过程中我都有做记录，同时结合项目内容做了下面这张思维导图，只有全部点亮这些叶子，才能算是做完了这个Demo（更多细节内容隐藏在了节点notes里面）



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgNode1.png)







这个思维导图肯定不是标准的Node.js 学习内容，只不过是应对我本次基础的学习，后续的学习会不断对这个思维导图进行修正的



## 总结

你以为我们就可以彻底抛弃官网了吗？在调试与编写代码的过程中，遇到某个知识点，就要果断回到官网查看，那里有最权威的解释。另外，带有目的性的查看官网总比一头扎进去舒服的多，你觉得呢



与其说这是我这几天学习Node.js 方式，不如说这是我学习新东西的整体方法和路线，写这篇文章也想借此当个话题和大家聊聊，欢迎大家留言或[进群](https://mp.weixin.qq.com/s/G7BXuZh0Qh1-mE6ts4LJqQ)分享彼此的学习方法/读书笔记/技术交流等，共同进步



隔三差五，我也会输出一些Node.js 相关的内容，你以为我喜新厌旧了？在我心中，Java依旧是我怡红院的头牌（我必须宠她）



最后，**不要永远呆在自己的舒适区，stay hungry， stay foolish**



























