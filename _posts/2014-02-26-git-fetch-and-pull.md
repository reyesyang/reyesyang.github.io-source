---
title: git fetch 和 git pull
layout: post
---

###git fetch
获取远程仓库更新到本地仓库

`git fetch` 不加任何参数时，如果当前分支没有设置跟踪分支(tracking branch)，则默认获取 origin 远程仓库的更新，否则获取跟踪分支对应远程仓库的更新。

`git fetch --all` 则会获取所有远程仓库的更新。

###git pull
获取远程仓库更新到本地仓库，并且合并当前所在分支所对应的跟踪分支(tracking branch)代码。等价于下面的两步操作：

```bash
git fetch
git merge FETCH_HEAD
```

`git pull --rebase` 命令也经常会使用到，该命令会在合并操作之前先进行一次衍合(rebase)操作。
假设当前在 master 分支，并且 master 的跟踪分支为 origin/master，则 `git pull --rebase` 等价于下面的三步操作：

```bash
git pull
git rebase origin/master
git merge origin/master
```
  
由于进行了衍合(rebase)操作，会改变历史提交记录，所以在公共分支上使用时一定要慎重。

###参考
1. [git-fetch(1) Manual Page](http://git-scm.com/docs/git-fetch)
2. [git-pull(1) Manual Page](http://git-scm.com/docs/git-pull)
