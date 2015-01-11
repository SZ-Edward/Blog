---
layout: post
title: 关于git stash命令 
description: git, git stash
---
今天完成了一个新功能准备提交到服务器的git仓库上，发现有同事更新了项目，但是我的本地git仓库的项目代码不是最新版本，为了避免出现版本冲突，所以[Google](www.google.com)了一下解决方案，见文末的`参考资料`。
主要是用到了[git stash](http://git-scm.com/book/zh/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89)命令，此命令的应用场景是：本地代码发生了改动，但是不想提交到代码仓库里，于是把这部分改动**暂存**起来。
于是我执行了
> $ git stash
`Saved working directory and index state  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file (To restore them type "git stash apply")`

这时本地项目**恢复**到了改动之前的版本，执行
> $ git pull
`Merge made by the 'recursive' strategy.
......
11 files changed, 249 insertions(+), 281 deletions(-)`

合并了服务器上最新的改动，接着执行
> $ git stash pop
`......
Dropped refs/stash@{0} (....)`

将**暂存**的内容恢复并在堆栈中删除它。如果你不想删除，就不要执行`git stash pop`命令，而执行`git stash apply`命令。此时你可以使用`git diff -w +文件名`来确认代码自动合并的情况，然后查看`git status`，该`git add`、`git commit`或`git push`就执行吧。

**参考资料**：
1. [Git:代码冲突常见解决方法](http://blog.csdn.net/iefreer/article/details/7679631)
2. [关于git pull的问题，如何在不commit的前提下pull回来？](http://www.zhihu.com/question/20180787)


--EOF--
