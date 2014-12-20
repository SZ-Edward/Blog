---
layout: post
title: 更新brew操作中发生的问题
description: Homebrew, update, git client vulnerability
---
昨天在[围脖的timeline](http://www.anquan.org/news/660)上看到github发布了一则[Git客户端漏洞公告](https://github.com/blog/1938-git-client-vulnerability-announced)，使用命令

    which git

显示的路径是`/usr/bin/git`，估计git是Xcode安装时自带的，查看git的版本号

    git --version

显示`git version 1.8.5.2 (Apple Git-48)`，正好需要升级一下，改用Homebrew管理更方便，于是使用命令

    brew install git

安装后的git版本号居然还是1.8，网上的git最新版本号已经是2.2.1了，应该是Homebrew的版本问题，于是更新Homebrew，使用命令

    brew update

接着使用命令

    brew outdated ＃检查所有安装包的最新版本

把git也更新到最新版本，使用命令

    brew upgrade git

这时查看git版本号就是2.2.1了，最后清理不需要的版本和安装包缓存，使用命令

    brew cleanup

但是!!仅仅是brew的git是最新版本，使用命令

    which git

显示的路径仍然是`/usr/bin/git`。必须把之前安装的git管理包转到brew下面，执行命令

    sudo mv /usr/bin/git /usr/bin/git-apple

然后再执行命令

    which git

显示的路径就是`/usr/local/bin/git`了，查看git版本号，使用命令

    git --version

这时显示的就是brew下面的git版本号2.2.1。


--EOF--

