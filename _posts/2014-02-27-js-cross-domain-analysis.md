---
layout: post
title: js跨域问题的分析
description: JavaScript, cross domain
---
今天新网站（www.example.com）上线，主页面需要用ajax访问sms.example.com的短信发送服务，造成了js跨域的问题。

> ####什么叫js跨域呢？
1、不同域名下的js访问
2、同一域名、不同端口的js访问
3、同一域名、不同协议的js访问
4、域名和对应的ip地址的js访问
5、主域相同、子域不同的js访问

笔者遇到的就是第5种情况，那么，如何解决呢？

按照正规路数，代理的方法最好了，详情请自行google，笔者使用了iframe，把需要请求sms.example.com的短信发送服务的html代码放到sms.example.com上面了，这样就避免了js跨域访问的问题，至此问题解决。

参考链接：[JavaScript跨域总结与解决办法](http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html#m0)


--EOF--
