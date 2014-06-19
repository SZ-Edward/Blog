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

其中`package_a`中的`a.py`引用了`package_b`的`b.py`的函数，引用语句如：

    from package_b.b import *

然后在`Python Project`项目路径下运行命令：

    python package_a/a.py

报错为`ImportError: No module named package_b.b`，检查了`package_a`和`package_b`下的`__init__.py`均正常，预估是文件路径的问题。于是将`a.py`中的引用语句改为：
    
    from package_b import b

亦不行，又把引用语句换为相对路径的引用（不推荐用相对引用）如：

    from .package_b import b
    from ..package_b import b
    from ...package_b import b

还是报错，不过报错内容变为：`ValueError: Attempted relative import in non-package`，然后在`package_a`路径下运行命令：

    python a.py

一样报错。

回到问题的原点，报错的内容是`ImportError: No module named package_b.b`，也就是说python加载依赖包的路径列表里没有`package_b`这个模块，于是在`a.py`中添加当前项目文件路径，语句如：

    import sys, os
    sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

接着在`package_a`路径下运行命令：

    python a.py

依然还是报错：`ImportError: No module named package_b.b`。难道，python的自定义模块间真的没有办法互相引用？还是我们的思路错了？

再次回到问题的原点，仔细分析两条报错语句`ImportError: No module named package_b.b`和`ValueError: Attempted relative import in non-package`，我在[stackoverflow](http://stackoverflow.com/)上找到了答案，详见[Attempted relative import in non-package even with __init__.py](http://stackoverflow.com/questions/11536764/attempted-relative-import-in-non-package-even-with-init-py)，原来问题出在###我们没有用模块的方式来运行命令###。接着在`Python Project`项目路径下运行命令：

    python -m package_a.a

或`package_a`路径下运行命令：

    python -m a

It works well啦！

