---
layout: post
title: mysql拒绝root用户访问之【ERROR 1045 (28000)】解决方法 
description: mysql, error 1045, solution
---
有一段日子没用本机的mysql数据库了，平时开发一直连的测试机的数据库，今天心血来潮用本机的数据库，输入命令

    mysql -u root -p

结果返回

    ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

估计是密码错误，想了半天也没想起来密码，那么干脆改密码吧！


使用命令

    mysqladmin -u root password 'new-password'

结果返回

    mysqladmin: connect to server at 'localhost' failed 
      
    error: 'Access denied for user 'root'@'localhost' (using password: YES)'

于是，使用命令

    netstat -anop | grep 3306

因为3306是mysql运行默认端口，所以执行上述命令查看mysqld是否运行。如果正在运行，执行命令

    /usr/sbin/mysqld stop

本机mysql改过配置，一般执行`/etc/init.d/mysqld stop`即可。或者推荐用`service mysql stop`，不行就用暴力方式`kill -9 mysqld 进程号`。

接着执行命令

    mysqld_safe --user=mysql --skip-grant-tables --skip-networking & 

此时终端输出`mysqld_safe A mysqld process already exists`，也就是mysql server端没关掉还在运行，于是我用暴力方式`kill -9 mysqld 进程号`关掉了mysqld，然后输入命令

    cat /etc/mysql/debian.cnf

把【client】部分的user和password记下来，使用命令

    mysql -u user -p

然后在终端提示输入密码（Enter password: ） 后把password输入、回车，就进入mysql了！

    Welcome to the MySQL monitor.  Commands end with ; or \g.

    Your MySQL connection id is 39

    Server version: 5.5.32-0ubuntu0.13.04.1 (Ubuntu)



    Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.



    Oracle is a registered trademark of Oracle Corporation and/or its affiliates. Other names may be trademarks of their respective owners.



    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.



    mysql>

  
到了此处恭喜你，接近成功了。如果你的mysql中建立过数据库，可以使用下列命令

    mysql> UPDATE user SET Password=PASSWORD(’new password’) where USER=’root’; 

    mysql> FLUSH PRIVILEGES; 

    mysql> quit;

来改root密码。如果你的mysql跟我的一样啥数据库都没有，则推荐使用命令

    mysql> GRANT ALL PRIVILEGES ON *.* TO root@localhost IDENTIFIED BY "new password"; 

    mysql> FLUSH PRIVILEGES; 

    mysql> quit;

来改root密码。最后，重启mysql服务，使用命令

    /etc/init.d/mysqld restart

或

    service mysql restart

就这样，你敲入命令`mysql -u root -p`，在提示密码后输入密码就可以使用mysql了！


--EOF--
