---
layout: post
title: 32bit ubuntu安装PIL失败的解决办法 
description: 32bit, ubuntu, PIL library
---
1. 安装所有需要的开发库

    sudo apt-get install libjpeg-dev libjpeg62 libjpeg62-dev zlib1g-dev libfreetype6 libfreetype6-dev

2. 由于我的项目跑在virtualenv中，所以用命令安装PIL和pillow两个库

    pip install PIL

    pip install pillow

###如果还是不行，执行下面的命令（64位系统看下面的参考链接）

    sudo ln -s /usr/lib/i386-linux-gnu/libz.so /usr/lib/

    sudo ln -s /usr/lib/i386-linux-gnu/libfreetype.so.6/usr/lib/

    sudo ln -s /usr/lib/i386-linux-gnu/libjpeg.so /usr/lib/

    pip install -I PIL


参考链接：[Python Image Library fails with message “decoder JPEG not available” - PIL](http://stackoverflow.com/questions/8915296/python-image-library-fails-with-message-decoder-jpeg-not-available-pil)


--EOF--
