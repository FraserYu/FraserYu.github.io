---
title: MySQL group_concat 函数详解
date: 2020-02-03 21:03:27
tags:
    - MySQL
categories: [Coding, 数据库-持久层-SQL]
id: mysql-group-concat-function-usage
description: 在SQL语句中字符串拼接我们都知道，但是如何分组拼接字符串呢？MySQL提供了内置的函数group_concat来解决日常的很多业务问题，熟练使用该函数，将会轻松解决许多所谓的难题
keywords: group_concat, group_ws, sql, mysql, group by, 
---

上一篇文章 [[跨表更新，看到自己写的SQL像个憨憨](https://dayarch.top/p/mysql-cross-table-update.html)](https://dayarch.top/p/mysql-cross-table-update.html) 写了关于跨表个更新的内容。一年过的很快，文中后来的两位员工 `馮大` 和 `馮二` 也要面对无情的 KPI 考核了，他们工作干的很不错，performance 分别是 4 和 5



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203190431.png)



新需求来了，静悄悄的来了！！！ 领导想要查看每个 performance 下都有谁，同时要求将这些人的名称要逗号拼接成一个字符串，也就是说要得到下面的结果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203190752.png)



要将结果集中某个指定的列进行字符串拼接，这要怎么做呢？主角闪亮✨登场



## GROUP_CONCAT(expr)

在 [Mysql 官方文档](https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html#function_group-concat) 中，该函数被放在聚合函数章节，如果你要按照指定字段分组拼接，就要配合关键字 `GROUP BY` 来使用的

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-imgimage-20200203192855823.png)





### 定义

> 该函数返回一个字符串结果，该字符串结果是通过分组串联的`非NULL值`。如果没有非NULL值，则返回NULL。完整语法如下：



```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])
```



What?  这个语法看着太复杂了吧，别着急，下面会用例子慢慢说明逐一验证滴



### 使用案例



先完成文章开头的需求：

```sql
SELECT performance, GROUP_CONCAT(employee_name) AS employees
FROM employees
GROUP BY performance;
```



zou是这个结果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203190644.png)



到这里，领导给过来的需求就完成了😜



**客官请留步，您点的菜还没上完呢......**



我们是国际化的团队，我们的家乡遍布五湖四海

<fancybox>![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203200846.png)</fancybox>



领导想关怀一下员工，要查看公司全部员工的家乡都有哪些地方。员工们可能来自同一个地方，所以要将结果集去重复，`DISTINCT` 关键字就派上用场了



```sql
SELECT GROUP_CONCAT(DISTINCT home_town)
FROM employees;
```



来看结果：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203201356.png)



领导的关怀遍布五湖四海啊......



文案要改了，领导的关怀是遍布`四海五湖`的, 那么 `ORDER BY` 关键字就派上用场了

```sql
SELECT GROUP_CONCAT(DISTINCT home_town ORDER BY home_town DESC) AS '领导关怀地区'
FROM employees;

-- 没我这么起变量的哈，还是汉语，我看你是疯了
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203201904.png)



这里你看到 `GROUP_CONCAT` 函数拼接字符串默认的分隔符是逗号 `,`, 领导不开心，逗号么的感情，要用❕才能体现出关怀的强烈， `SEPARATOR` 关键字就派上用场了



> 分组拼接的值之间默认分隔符是逗号（，）。要明确指定分隔符，需要使用 `SEPARATOR` 关键字，紧跟其后的是你想设置的分隔符。要完全消除分隔符，就在 `SEPARATOR` 关键字后面写  '' 就好了



```sql
SELECT GROUP_CONCAT(DISTINCT home_town ORDER BY home_town DESC SEPARATOR '!') AS '领导关怀地区'
FROM employees;
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203202350.png)



```sql
SELECT GROUP_CONCAT(DISTINCT home_town SEPARATOR '') AS '领导关怀地区'
FROM employees;
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203203120.png)



这关怀到位了吧，你品，你细品！！！



领导的关怀能力也有限，**拼接的字符串默认的最大长度是1024个字符**，可以通过下面语句查看当前限制是多少：



```sql
show variables like 'group_concat_max_len';
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200203204525.png)



领导的能力可是飘忽不定的，所以我们可以灵活的设置这个值



```sql
SET [GLOBAL | SESSION] group_concat_max_len = val;
```



- SESSION: 在当前对话中生效
- GLOBAL：全局都生效

该语句在执行后，MySQL重启之前一直有作用，一旦重启 MySQL，则会恢复默认值



有时候 ``GROUP_CONCAT()``  还要搭配 `CONCAT_WS() ` 发挥出一点点威力，举个简单的例子



> 将消费者的名和姓用逗号进行分隔，然后再用 `;` 进行分隔



```sql
SELECT
    GROUP_CONCAT(
       CONCAT_WS(', ', contactLastName, contactFirstName)
       SEPARATOR ';')
FROM
    customers;
```



这里是 [CONCAT_WS()函数用法](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_concat-ws), 很简单，请自行查看吧......



## 注意⚠️

GROUP_CONCAT（）函数返回单个字符串，而不是值列表。这意味着我们不能在 IN 运算符中使用GROUP_CONCAT（）函数的结果，例如，在子查询中， 像这样：



```sql
SELECT
    id, name
FROM
    table_name
WHERE
    id IN GROUP_CONCAT(id);
```



## 总结

在许多情况下，我们都可以应用GROUP_CONCAT（）函数产生出有用的结果，同时也可以结合其他函数发挥出更大的威力. 单招学会了，就要学会连招 combo 了



- 如果你也像我一样刚知道这个知识点，还请点个「在看」
- 如果你早都知道这个小儿科内容，还请留言送上「嘘声」


