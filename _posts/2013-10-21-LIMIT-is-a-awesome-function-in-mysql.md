---
layout: post
title: mysql中很赞的函数limit
description: mysql, limit
---
在使用mysql-workbench进行查询的过程中，经常看到查询语句的最后加上了一个LIMIT，今天了解了一下，发现这是一个配合做分页插件的绝佳功能，非常赞，下面就简单介绍一下它的基本用法。

> SELECT * FROM table  LIMIT [offset,] rows | rows OFFSET offset

LIMIT 子句可以被用于强制 SELECT 语句返回指定的记录数。LIMIT 接受一个或两个数字参数。参数必须是一个整数常量。如果给定两个参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目。初始记录行的偏移量是 0(而不是 1)，为了与 PostgreSQL 兼容，MySQL 也支持句法： LIMIT # OFFSET #。
> SELECT * FROM table LIMIT 5,10;  // 检索记录行 6-15

为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：      
> SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.     

如果只给定一个参数，它表示返回最大的记录行数目：      
> SELECT * FROM table LIMIT 5;     //检索前 5 个记录行     

//换句话说，LIMIT n 等价于 LIMIT 0,n。

如果做一个分页插件，使用LIMIT来做查询会相当方便。例如：

页条数、当前页数、记录总数、最大页数分别用pageRows、pageIndex、totalRows、pageMax来表示。

那么除了pageRows和pageIndex是用户自定义的，totalRows为SELECT COUNT(*) FROM table，整除情况下pageMax为totalRows/pageRows，否则pageMax为totalRows/pageRows + 1，因此当用户请求某一页的数据，我们发的sql为

> SELECT * FROM TABLE WHERE ... LIMIT pageRows*(pageIndex-1), pageRows;    

是不是很赞呢~ ：D


--EOF--
