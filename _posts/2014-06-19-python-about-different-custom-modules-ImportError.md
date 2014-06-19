---
layout: post
title: 关于Python的自定义模块之互相引用的ImportError: No module named XXX的bug
description: Python, module, ImportError
---
本文的场景是：有一个名为`Python Project`的项目，包含有两个自定义的module分别是`package_a`和`package_b`，其项目路径如下所示：
    `--Python Project
      |
       --package_a
        |
         --__init__.py
         --a.py
         --...
       --package_b
	|
	 --__init__.py
	 --b.py
	 --...`

今天在Mysql中对A表的数据进行**UPDATE**时发现了一个bug，我的**UPDATE**语句是：
    `UPDATE A SET x=1 WHERE y IN (SELECT t.y FROM A t WHERE t.z != 0)`

这是Mysql报错为：
    `You can't specify target table A for update in FROM clause.`

[Stackoverflow](http://stackoverflow.com/)了一下，[You can't specify target table for update in FROM clause](http://stackoverflow.com/questions/4429319/you-cant-specify-target-table-for-update-in-from-clause)，问题出在Mysql不允许在inner query做**事务性的操作**。

那么怎么解决这个问题呢？Trick来了，在UPDATE的WHERE里面用临时表来替换一下就可以了，于是修改下**UPDATE**语句：
    `UPDATE A SET x=1 WHERE y IN (SELECT t.y FROM (SELECT * FROM A) t WHERE t.z != 0)`
就可以执行成功了。
