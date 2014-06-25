---
layout: post
title: 在Mysql中复制数据
description: Mysql, copy data
---
一天，唐师傅对悟空说道：“徒儿，为师这里有一个屋子叫mysql，里面堆着一些大大小小的木箱子tables。我想把这些箱子整理一下，先把所有值钱的东西放在一个大箱子里，你来给我帮帮忙吧。”

悟空说了一句：“好的，师傅。看我七十二变～”，接着念起咒语来：

    INSERT INTO table(w, x, y, z)
    SELECT column1, column2, column3, 'expensive'
    FROM table1, table2, table3...

弹指间，只见所有值钱的东西全都飞进了一个大箱子。

