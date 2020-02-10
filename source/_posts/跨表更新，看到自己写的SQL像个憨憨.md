---
title: 跨表更新，看到自己写的SQL像个憨憨
date: 2020-01-22 10:09:02
tags:
    - MySQL
categories: [Coding, 数据库-持久层-SQL]
id: mysql-cross-table-update
description: 跨表查询我们经常食用，其实类似的语法也可以用到跨表更新上面，如果你有类似从一张表的数据更新到另一张表的数据需求时，了解这个知识点还是非常有用的
keywords: 跨表更新, sql, mysql, update inner join, update left join

---

有点 SQL 基础的朋友肯定听过 「跨表查询」，那啥是跨表更新啊？

## 背景

项目新导入了一批人员数据，这些人的有的部门名称发生了变化，有的联系方式发生了变化，暂且称该表为 

`t_dept_members`, 系统中有另外一张表 `t_user_info` 记录了人员信息。要求将 `t_dept_members` 中有变化的信息更新到 `t_user` 表中，这个需求就是「跨表更新」啦



## 憨B SQL 直接被秒杀

不带脑子出门的就写出了下面的 SQL 

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200121143958.png)





看到身后 DBA 小段总在修仙，想着让他帮润色一下😜，于是发给了他，然后甩手回来就是这个样子：

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200121144308.png)

​	

看到这个 SQL 语句我都惊呆了，还能这样写，在无情的嘲笑下，一声 KO 我直接倒下。死也得死的明白，咱得查查这是咋回事啊



## Mysql Update Join

我们经常使用 `join` 查询表中具有（在 `INNER JOIN` 情况下）或可能没有（在 `LEFT JOIN` 情况下）另一个表中匹配行的表中的行。



同样，在 MySQL 中, 我们也可以在 UPDATE 语句中使用 JOIN 子句执行跨表更新，语法就是这样：

```sql
UPDATE T1, T2,
[INNER JOIN | LEFT JOIN] T1 ON T1.C1 = T2. C1
SET T1.C2 = T2.C2,
    T2.C3 = expr
WHERE condition
```



我们还是详细的说明一下上面的语法：

- 首先，在 UPDATE 子句之后，指定主表（T1）和希望主表联接到的表（T2）。请注意，必须在UPDATE 子句之后至少指定一个表
- 接下来，指定你要使用的联接类型，即 INNER JOIN 或 LEFT JOIN 以及联接谓词。 JOIN子句必须出现在 UPDATE 子句之后（这个大家都是知道的哈）
- 然后，将新值分配给要更新的 T1或 T2 表中的列
- 最后，在 WHERE 子句中指定一个条件以将行限制为要更新的行



如果你遵循 update 语法，你会发现有另外一种语法也可以完成跨表更新

```sql
UPDATE T1, T2
SET T1.c2 = T2.c2,
      T2.c3 = expr
WHERE T1.c1 = T2.c1 AND condition
```



上面的语法其实隐式使用了 inner join 关键字，完全等同于下面的样子：

```sql
UPDATE T1,T2
INNER JOIN T2 ON T1.C1 = T2.C1
SET T1.C2 = T2.C2,
      T2.C3 = expr
WHERE condition
```



*个人建议还是加上 `inner join` 关键字吧，这样可读性更好，尽享丝滑，你觉得呢？*



> 我摸鱼看到的，觉得是灵魂翻译
>
> 谈太廉，秀你码  （Talk is cheap，show me the code）



## Update Join 例子

年底了，又到了评绩效的时候了，就是那个叫 KPI 的东东（你们有吗），听说要根据 KPI 调工资了。有两张表

第一张表「employees-员工表」

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200121165912.png)



建表语句如下：

```sql
create table employees
(
	employee_id bigint auto_increment comment '员工ID，主键',
	employee_name varchar(50) null comment '员工名称',
	performance int(4) null comment '绩效分数 1，2，3，4，5',
	salary float null comment '员工薪水',
	constraint employees_pk
		primary key (employee_id)
)
comment '员工表';
```



第二张表「merits-绩效字典表」

![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200121170053.png)



建表语句如下：

```sql
create table merits
(
	performance int(4) null,
	percentage float null
)
comment '绩效字典表';
```



先生成一些模拟数据

```sql
-- 绩效字典初始化数据
INSERT INTO merits(performance, percentage)
VALUES (1, 0),
       (2, 0.01),
       (3, 0.03),
       (4, 0.05),
       (5, 0.08);


-- 员工表初始化数据
INSERT INTO employees(employee_name, performance, salary)
VALUES ('拱哥', 1, 1000),
       ('小段总', 3, 20000),
       ('大人', 4, 18000),
       ('司令', 5, 28000),
       ('老六', 2, 10000),
       ('罗蒙', 3, 20000);
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200122091333.png)



> 调薪规则：
>
> 原有薪资 + （原有薪资 * 当前绩效对应的调薪百分比）



按照调薪规则写 update 语句：

```sql
UPDATE employees
    INNER JOIN
    merits ON employees.performance = merits.performance
SET salary = salary + salary * percentage;
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200122091505.png)



拱哥绩效不好，没给涨工资......



> 三横一竖一咕嘎，四个小猪🐷来吃zha，咕嘎咕嘎又来俩



临近年底，公司又来了两位新同事, 但是公司年度绩效已经评完，所以新员工绩效为 NULL



```sql
INSERT INTO employees(employee_name, performance, salary)
VALUES ('馮大', NULL, 8000),
       ('馮二', NULL, 5000);
```



新员工工作干的不错，也要 `1.5%`  涨点工资的。如果我们还是用 `UPDATE INNER JOIN`，按照上面的更新语句是不可能完成的，因为条件等式不成立，这是我们就要用到 `UPDATE LEFT JOIN` 了



```sql
UPDATE employees
    LEFT JOIN
    merits ON employees.performance = merits.performance
SET salary = salary + salary * 0.015
WHERE merits.percentage IS NULL;
```



![](https://cdn.jsdelivr.net/gh/FraserYu/img-host/blog-img20200122094147.png)



到这里，新员工的涨薪工作也做完，拱哥由于知识点了解不透彻，灰溜溜的回家过年
