---
layout: post
title: 关于Tornado的get_argument函数我所犯的低级错误
description: Tornado, get_argument, understood mistaken
---
Tornado的`get_argument`函数用法是：

    RequestHandler.get_argument(name, default=, []strip=True)[source]
	
	Returns the value of the argument with the given name.

	If default is not provided, the argument is considered to be required, and we raise a MissingArgumentError if it is missing.

	If the argument appears in the url more than once, we return the last value.

	The returned value is always unicode.

简单地说，我所犯地错误就是用`RequestHandler.get_argument('greeting', 'hello')`的用法误以为请求的url参数greeting的值为空，则默认greeting参数值为hello，这样的理解是错误的。只有请求的url中没有greeting这个参数时，才会有一个greeting参数值为hello。
