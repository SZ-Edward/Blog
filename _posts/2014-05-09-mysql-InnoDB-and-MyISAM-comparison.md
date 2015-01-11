---
layout: post
title: 关于Mysql的存储引擎InnoDB和MyISAM的对比
description: mysql, InnoDB, MyISAM
---
上午在重构代码的时候，想起了在Tornado里面使用MySQLdb处理事务的一个教训：之前操作数据库表的时候，A表用的InnoDB引擎，B表用的是MyISAM引擎。然后往这两张表插入数据，A表中数据保存正常，B表中没有数据进去。查看Log日志，A、B两张表的insert语句执行正常，不知道是什么原因出现这样诡异的现象，我和我的小伙伴都无语了。。。

仔细查看代码，发现sql语句执行完毕后，没有自动提交事务，于是加上事务提交的代码，问题解决。

过后回想这个问题，打开数据库查看A、B两张表的差异，发现它们的存储引擎不同。Google这两个存储引擎的差异，发现还真是跟事务有关，因此也不奇怪发生`没有手动提交事务，但是A表中数据保存正常，B表中没有数据进去`的现象了。

————————————我是卖萌的分割线————————————

InnoDB给MySQL提供了具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。InnoDB 提供了行锁(locking on row level)，提供与 Oracle 类型一致的不加锁读取(non-locking read in SELECTs)。这些特性均提高了多用户并发操作的性能表现。在InnoDB表中不需要扩大锁定(lock escalation)，因为 InnoDB 的列锁定(row level locks)适宜非常小的空间。InnoDB是MySQL上第一个提供外键约束(FOREIGN KEY constraints)的表引擎。

MyISAM是MySQL缺省存贮引擎 。每张MyISAM 表被存放在三个文件 。frm文件存放表格定义。数据文件是MYD (MYData) 。索引文件是MYI (MYIndex) 引伸。因为MyISAM相对简单所以在效率上要优于InnoDB，小型应用使用MyISAM是不错的选择。另外，MyISAM表保存成文件的形式，在跨平台的数据转移中使用MyISAM存储会省去不少的麻烦。



以下是一些细节和具体实现的差别：

  1. InnoDB不支持FULLTEXT类型的索引。

  2. InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。

  3. 对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。

  4. DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。

  5. LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用。

另外，InnoDB表的行锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表，例如update table set num=1 where name like “%aaa%”

参考：[mysql 存储引擎 InnoDB和myisam存储引擎的区别 /（自己小结）](http://blog.csdn.net/fbd2011/article/details/6898572)


--EOF--
