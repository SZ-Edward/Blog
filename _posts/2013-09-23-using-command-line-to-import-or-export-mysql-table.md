---
layout: post
title: 命令行导出/导入mysql表结构和数据 
description: command line, import, export, table
---
在命令行下mysql的数据导出有个很好用命令`mysqldump`，它的参数有一大把，常用它来导入或导出表，如：

    mysqldump -u root -p anything database example table1 table2 > example.sql

这样就可以将数据库example的表table1和table2以sql形式导入到example.sql文件中，其中-u参数表示访问数据库的用户名，如果有密码还需要加上-p参数。

用此命令给mysql数据库导入数据也是非常便捷，如：

    mysql -u root database foo < foo.sql

这样就可以将foo.sql文件的数据全部导入数据库foo。


--EOF--
