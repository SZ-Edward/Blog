---
layout: post
title: 关于Tornado的set_secure_cookie方法报UnicodeDecodeError
description: tornado, set_secure_cookie, UnicodeDecodeError
---
昨天用`tornado`搭建新系统，做登录模块的时候，当用户登录验证成功，服务器跳转用户页面并记录用户信息，使用到了[set_secure_cookie](http://tornado.readthedocs.org/en/latest/web.html#tornado.web.RequestHandler.set_secure_cookie)这个方法。

`RequestHandler.set_secure_cookie(name, value, expires_days=30, version=None, **kwargs)`这个方法的value参数值应为**随机的字节序列**，这里由于我的误用设置了一个`dict`对象，造成程序执行到`set_secure_cookie`方法时就报`UnicodeDecodeError`。将value参数值改为`unicode`或`str`即解决此问题。

--EOF--

