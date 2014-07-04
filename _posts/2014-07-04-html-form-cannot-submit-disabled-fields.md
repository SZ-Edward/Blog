---
layout: post
title: HTML表单无法提交disabled字段内容
description: html, form, submit, disabled 
---
今天表单中一个`input`有`disabled`属性，其字段值总是无法被服务器接收到，其实这个`input`的作用是不能被用户编辑但可以让服务器接收的，应该用`readonly`属性而非`disabled`属性，改过来后就正常啦，又涨姿势了～
