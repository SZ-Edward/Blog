---
layout: post
title: Tornado模板使用的一个小细节
description: tornado, template
---
今天在HTML代码中使用了Tornado模板，有一个`input`的代码是这样的：

    <input type="text" name="uid" value={{ "{{ uid " }}}} placeholder="请输入用户编号">

当template渲染时，{{ "{{ uid " }}}}的值不为空的页面展示正常，一旦{{ "{{ uid " }}}}的值为空，这个input框中就自动将placeholder的内容赋值了。出现这个问题的原因是，`给属性赋值应该加上引号`，在上面的代码里，给input的value属性赋值方法就不正确。改过来后，一切正常了。

    <input type="text" name="uid" value="{{ "{{ uid " }}}}" placeholder="请输入用户编号">

为了完成本文，还遇到了`jekyll template`转义的蛋疼问题（由`jekyll`使用的`liquid template`引起），`to escape {{ "{{ this " }}}}`的解决方案详见[How to escape liquid template tags?](http://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags)

--EOF--
