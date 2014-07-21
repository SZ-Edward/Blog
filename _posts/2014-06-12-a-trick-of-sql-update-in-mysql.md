---
layout: post
title: 关于Mysql的UPDATE中WHERE SELECT也是同一张表的bug和trick
description: Mysql, UPDATE, WHERE SELECT, bug, trick
---
今天在Mysql中对A表的数据进行**UPDATE**时发现了一个bug，我的**UPDATE**语句是：
    `UPDATE A SET x=1 WHERE y IN (SELECT t.y FROM A t WHERE t.z != 0)`

这是Mysql报错为：
    `You can't specify target table A for update in FROM clause.`

[Stackoverflow](http://stackoverflow.com/)了一下，[You can't specify target table for update in FROM clause](http://stackoverflow.com/questions/4429319/you-cant-specify-target-table-for-update-in-from-clause)，问题出在Mysql不允许在inner query做**事务性的操作**。

那么怎么解决这个问题呢？Trick来了，在UPDATE的WHERE里面用临时表来替换一下就可以了，于是修改下**UPDATE**语句：
    `UPDATE A SET x=1 WHERE y IN (SELECT t.y FROM (SELECT * FROM A) t WHERE t.z != 0)`
就可以执行成功了。

--EOF--
