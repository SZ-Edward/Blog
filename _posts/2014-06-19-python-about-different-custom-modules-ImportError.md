---
layout: post
title: 关于Python的自定义模块之互相引用的ImportError
description: Python, module, ImportError
---
本文的场景是：有一个名为`Python Project`的项目，包含有两个自定义的module分别是`package_a`和`package_b`，其项目路径如下所示：

--Python Project
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
    --...

今天在Mysql中对A表的数据进行**UPDATE**时发现了一个bug，我的**UPDATE**语句是：
    `UPDATE A SET x=1 WHERE y IN (SELECT t.y FROM A t WHERE t.z != 0)`

这是Mysql报错为：
    `You can't specify target table A for update in FROM clause.`

