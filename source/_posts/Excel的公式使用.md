---
title: Excel的公式使用
tags:
  - Excel
categories: [Coding, Others]
id: excel-formula
date: 2019-05-10 10:15:39
description: Excel强大的办公工具，学习基本的公式增加工作效率
keywords: Excel,公式,Office

---

### 前言

最近在做基础数据的整理，数据存放在 Word 表格中，要将大量的数据整理好并导入到数据库中着实需要费些力气，于是需要笨方法和巧方法一起用

### 整理过程

首先应用笨方法，将 Word 中的数据表格拷贝到 Excel 中，貌似这是必须的，由于拷贝过来的表格有合并单元格的情况，所以我们要将 Excel 中的表格标准化

### Excel通过公式拼接字符串换行
我们通常都会通过Excel公式**=CONCATENATE(Text1,[Text2])**来拼接字符串，当我们需要拼接的字符串中需要换行时我们可以使用**CHAR(10)**来完成拼接换行符，请看如下例子

	=CONCATENATE("hello",CHAR(10),"world")

输出结果如下：

    hello
    world

**NOTE:** 使用拼接字符串公式的单元格请设置为**常规**，不要是自动换行.




1. 取消合并单元格

   将 Excel 全部选中，然后点击取消单元格

   <img itemprop="url image" src="/uploads/May-16-2019 11-17-29.gif" />

2. 将空单元格进行字段的填充

   我们看到拆分后的单元格好多都是空的，我们需要将空的单元格填充为它上面一行的数据

   + 选中待处理的单元格
   + 按F5
   + 在弹出框的位置选择空值选项
   + 键盘输入等于号"="，然后敲击向上方向键
   + Ctrl + Enter 组合完成

   <img itemprop="url image" src="/uploads/May-16-2019 11-20-29.gif" />

3. 修改 Excel 的默认语言

  我用的Mac 英文版，打开 Excel 没有类似 LENB() 这样的函数，所以我需要将语言就改为中文

  ```shell
  # 修改 Excel 语言
  defaults write com.microsoft.Excel AppleLanguages '("zh-tw")'

  # 修改 Word 语言
  defaults write com.microsoft.Word AppleLanguages '("zh-tw")'
  ```

4. 数字与汉字的拆分

  拷贝过来的数据，需要做K-V 的映射，将一段文字中的数字提取出来作为 key， 将文本中的汉字提取出来作为 Value，所以需要用到两组公式：

  + LEFT(单元格, length) / RIGHT(单元格, length)

    > LEFT: 从单元格内容开始位置截取指定长度内容；
    >
    > RIGHT: 从单元格内容末尾位置截取指定长度内容；

  + LEN(单元格) / LENB(单元格)

    > LEN是返回字符串的**字符**数;
    >
    > LENB是返回字符串的**字节**数;
    >
    > *数字、字母、英文、标点符号（半角状态）都是按1计算的，汉字、全角状态下的标点符号，每个字符按2计算*

  <img itemprop="url image" src="/uploads/May-16-2019 11-27-17.gif" />

5. 阿拉伯数字转罗马数字

   只需要应用公式：**= ROMAN(A16)**

   <img itemprop="url image" src="/uploads/May-16-2019 11-48-53.gif" />

6. 罗马数字转阿拉伯数字

   同样应用公式：**=MATCH(B16,INDEX(ROMAN(ROW(INDIRECT("1:4000"))),0),0)**

   <img itemprop="url image" src="/uploads/May-16-2019 11-52-57.gif" />

以上就是这次数据提取用到的一些公式

### 附录
在此处附上 office 的各种语言字典码
<img itemprop="url image" src="/uploads/Xnip2019-05-13_16-19-16.jpg" />
