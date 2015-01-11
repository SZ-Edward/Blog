---
layout: post
title: 关于MySQL的1064错误 
description: mysql, error, 1064
---
MySQL的1064错误是SQL语句写的有问题时出现的，即SQL的语法错误。笔者常常使用MySQL-python这个库来对MySQL进行操作，代码中报这个错误的一般是cursor.execute(sql, param)这一行。

这种参数式执行SQL语句的用法可以有效防止SQL注入的安全问题，但是为什么MySQL会报错呢？如果你确认SQL写的没问题，检查一下SQL语句中是否使用了引号。

在使用cursor.execute(sql, param)时，MySQL-python库会自动转义含有%s的字符串，所以不要画蛇添足在SQL语句中给%s加引号了，会报1064的错误滴！

另外也有许多人使用有SQL注入隐患的cursor.execute(sql % param)这种用法，这样是可以给%s加引号的。

但是安全问题孰重孰轻，相信各位自有判断。


--EOF--
