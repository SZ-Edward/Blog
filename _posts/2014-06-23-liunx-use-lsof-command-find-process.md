---
layout: post
title: 在Linux/Unix系统中使用lsof命令找到进程信息
description: Linux, Unix, lsof
---
今天修复一个系统bug的时候，看到有一个API调用的配置是http://127.0.0.1:9999，想去看看这个系统的文件，**突然想不起它的文件路径了**，呵呵，别捉急，让`lsof`命令来帮你！

根据端口号使用`lsof`命令：

    lsof -i:port number

系统打印的信息是：

    COMMAND     PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
    python2.7 18800 root   12u  IPv4 657136855      0t0  TCP *:9999 (LISTEN)

根据进程ID（PID）找到文件路径：

    lsof -p PID

更多用法猛戳[10 lsof command usages with example – Unix/Linux](http://crybit.com/lsof-command-usages/)

--EOF--

