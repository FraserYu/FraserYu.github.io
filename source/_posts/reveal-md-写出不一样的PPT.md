---
title: reveal-md 写出不一样的PPT
date: 2020-09-07 18:30:25
tags:
    - 总结
categories: [Coding, Tools]
id: reveal-md-ppt
description: 通常程序员瞧不上写PPT抑或不愿意写PPT，其实这是一个不容被忽视的软技能，通过PPT的演示可以锻炼一个人的综合能力
keywords: reveal-md,ppt,reveal.js
music:
    enable: true
    server: netease
    type: song
    id: 33419765
---

听说程序员看不上写 PPT ？

<img src="https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200905221047.png" style="zoom:25%;" />



程序员是实干派，面对写 PPT 这事通常都是嗤之以鼻。头可断，万行代码不能乱；血可流，写PPT是真犯愁。写 PPT 着实是程序员工作中的三座大山之一

<img src="https://rgyb.sunluomeng.top/20200830215554.png" style="zoom:15%;" />



有人做了三分，可以通过 PPT 说出七分，有人做了七分，却说不出三分，程序员的内心活动正如去年很火的新东方年会改编 「沙漠骆驼」的歌词一样：

![](https://rgyb.sunluomeng.top/20200830220950.png)

为了不再经历相似的年底考评悲剧。所以，作为老实人，如果做了七分，底线至少要说出七分，没有商量的余地



可是当程序员面对 **PPT** 或 **KeyNote** 总是无从下手，这要怎么办？

> 那咱就以程序员熟悉的方式写  PPT 呗

这里要给大家打个预防针，以这种方式写 PPT 的门槛低到吓人，只需会基本的 Markdown 语法即可



## reveal-md

> reveal-md 可以将 Markdown 文件渲染出非常漂亮的演示文稿

**Github 地址：** https://github.com/webpro/reveal-md



### 安装

安装很简单，安装好 NodeJS，一条 npm 命令（全局安装）即可：

```javascript
npm install -g reveal-md
```

## 

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200830202837.png)



安装完，你需要做的就只是写 Markdown 文件了



### 写 Markdown 文件

在一个 Markdown 文件中，用 `\n---\n` 作为分隔多个 PPT 页面的标识符，如果你用 Typora，其实就是输入 `---`，然后回车就搞定了，然后在里面按照 Markdown 语法填充内容就 OK 了

```markdown
# 标题

* 程序员如何看待写 PPT？
* 如何以技术人员的方式写 PPT？


---

## 程序员如何看待写 PPT？

> 世界最牛逼的语言是 「PPT」

```

这就写完了 2 页 PPT，不用调整格式，不用添加过渡效果，只需要run reveal-md 服务渲染一下你的 Markdown 文件就好了



### 渲染 Markdown 文件

因为我们是全局安装的 `reveal-md`， 所以只需要按照下面命令通过路径找到你的 MD 文件即可

```shell
reveal-md path/to/yourSlide.md
```

你也可以 cd 到 Markdown file 目录直接运行：

```shell
reveal-md yourSlide.md
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200902111947.png)

在运行上述命令后会自动打开你默认使用的浏览器，默认是高大上的黑色主题，如果你开启浏览器全屏，就是这个效果了 （听说，属于你的演示来要开始了？）

![](https://rgyb.sunluomeng.top/20200902105626.png)

点击右下角的`下一页`图标, 会有默认翻滚的过渡效果，就到了下一页（这个效果值几毛？）

![](https://rgyb.sunluomeng.top/2020-09-06at20.09.58.gif)




这种简约又简单的方式爱了没？



我们都知道，PPT 或者 KeyNote 对代码的展示不是很友好，reveal-md 就不一样了（毕竟是 Markdown 嘛）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img9150e4e5ly1fkq92f0ughg206o06ot9b.gif)

我们再通过 `---` 新建一页，按照 Markdown 语法添加代码，重新 run 上述命令，可以看到，展示出来的代码**有滚动条**，**有代码高亮**，一应俱全

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200902111803.png)



以上我们都是使用 reveal-md 的默认值，它当然也支持一些个性化设置



### 个性化设置

#### 主题

上面使用的是默认主题 black （黑色主题），我们可以通过 `--theme xxxTheme` 来变更主题

```shell
reveal-md myTest.md --theme solarized
```

效果就是这个样子了（好像有点阮一峰老师博客的赶脚）

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903212518.png)

去哪里找这些主题呢？访问 https://github.com/hakimel/reveal.js/tree/master/css/theme/source 就可以查看 reveal-md 默认支持的所有主题，你只需要在运行命令时指定主题名字就好



如果你想玩点更高级的自定义样式，只需要指定自定义的主题文件就可以了

```shell
reveal-md slides.md --theme theme/my-custom.css
```

如果你不知道怎么写自定义主题 CSS 文件，那就参照 https://rawgit.com/puzzle/pitc-revealjs-theme/master/theme/puzzle.css 更改一些参数值就是你自定义的了



#### 代码高亮主题

Markdown 本身就对代码有很好的支持，上面使用的代码默认高亮主题是 `zenburn`，当然也可以使用 `--highlight-theme xxxTheme` 选择其他高亮主题，像这样：

```shell
reveal-md slides.md --highlight-theme atom-one-dark
```

reveal-md 同样支持很多高亮主题，查看 https://github.com/highlightjs/highlight.js/tree/master/src/styles 同样是指定名字就好。这里更换为 atom-one-dark 高亮主题后，就是这个效果了

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903213345.png)



#### 添加背景图片

在上面演示过渡效果时你也许就注意到了，我在某一页上添加了背景图片，添加背景图片也很（四声）简单，只需要在每页 PPT 的开头添加这段代码指定图片就可以

```markdown
<!-- .slide: data-background="./lighthouse-5439227_1920.jpg" -->
```

担心你在 Markdown 文件里写上述代码有些困惑，这里做个截图说明

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903213901.png)

重新运行，就是你刚刚看到的效果

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903214037.png)



能添加图片，当然也能添加视频



#### 添加背景视频

和添加背景图片类似，同样在每一页的开头添加下面代码：

```markdown
<!-- .slide: data-background-video="./all.mov" -->
```

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903214514.png)



> 背景视频是 2020 Mac 屏保，还是非常炫酷低，有兴趣的可以访问：*https://mp.weixin.qq.com/s/TgbF7gRWEu7DeUQdYYWYmg*  获取相应视频和安装文件就好了



#### 演讲人预览下一页视角

通常在讲演 PPT 时，会有一块小屏幕做下一页提醒，reveal-md 同样支持，只需要在 Markdown 文件开头单独页添加（别写错）：

**Note:.**

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903215220.png)

重新运行，在打开的界面按键盘 `s` 按键, 就会弹出另外一个浏览器窗口为你提供预览下一页的界面，这时我们可以将全凭浏览器页拖到公共大屏，自己查看小屏幕

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200903215336.png)



#### 自动刷新改动

在写演示稿如果有些改动就重新运行还挺麻烦的，所以可以通过 **-w** 参数来自动刷新页面内容

```shell
reveal-md myTest.md --theme solarized --highlight-theme atom-one-dark -w
```

这时我们就不用每次重新启动服务了，你来试试看？万一演示的时候发现问一些小问题，可以神不知鬼不觉的更改



#### 导出 PDF

精彩的演示稿，别人想拿过去学习一番，你可以直接将 Markdown 文件导出为 PDF 文件

```shell
reveal-md myTest.md --print slides.pdf
```



#### 自定义模版

上面演示的这些都是在默认模版下渲染的，我们可以通过 --template xxxTemplate 来自定义自己的模版

```shell
reveal-md myTest.md --template my-reveal-template.html
```

同样，如果你不知道怎么写模版，你完全可以参考这个默认的模版内容做一些值的更改：*https://github.com/webpro/reveal-md/blob/master/lib/template/reveal.html*



#### 双列模式

之前刚接触 reveal-md.js  时只是使用单列模式，其实在有些情况双列展示可以非常友好的展示一些对比性内容， **刚好组内万能大神找到了双列解决方案，我就私下取经, 得到了解决方案**

我们可以写个自己的 CSS 文件，比如 `doubleCol.css` 然后在里面添加这点样式就行了：

```css
#left {
	margin: 10px 0 15px 20px;
	text-align: left;
	float: left;
	z-index:-10;
	width:48%;
	font-size: 0.85em;
	line-height: 1.5; 
}

#right {
	margin: 10px 0 15px 0;
	float: right;
	text-align: left;
	z-index:-10;
	width:48%;
	font-size: 0.85em;
	line-height: 1.5; 
}
```

然后我们在 Markdown 文件里，像下面这样添加 Markdown 语法内容就好了

```html
<div id="left">

## Left column
- Bullet 1
- Bullet 2
- Bullet 3 
- Even [links](https://www.google.com)

</div>

<div id="right">

## Right colum
1. List
2. List
3. ![Icon](https://cdn3.iconfinder.com/data/icons/ballicons-free/128/graph.png)

</div>
```



通过 --css 引入咱们自定义的 CSS 文件即可，就想这样：

```java
reveal-md myTest.md  --css theme/doubleCol.css
```

![](https://rgyb.sunluomeng.top/20200906205645.png)



到这里，这些设置基本上可以满足基本需求了，如果你听过 [reveal.js](https://github.com/hakimel/reveal.js) ，那你很容易就看出来了， reveal-md 只不过在上层做了 MarkDown 的渲染，这样只需要用基本的 Markdown 语法就可以代替写整个 HTML，大大简化整个过程，不建议书写大量 HTML，这与 Markdown 的设计思想时违背的，如果你有兴趣完全可以尝试一下了



## 总结

reveal-md 只是写演示文稿的工具，不要被其过渡捆绑；另外写 PPT 也是一个不容忽视的软技能，不要产生排斥心理，它可以锻炼自己归纳总结的能力，梳理内容的逻辑思维，以及演讲表达的技巧...... 作为程序员，拓宽自己的能力总是没错的

![](https://rgyb.sunluomeng.top/20200906211702.png)











