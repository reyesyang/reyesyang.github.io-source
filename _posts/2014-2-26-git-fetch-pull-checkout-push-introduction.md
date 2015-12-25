---
title: Git fetch, pull, checkout 和 push 简介
tags: [Git]
---

#### git fetch
获取远程仓库更新到本地仓库

`git fetch` 不加任何参数时，如果当前分支没有设置上游(upstream)分支，则默认获取 origin 远程仓库的更新，否则获取跟踪分支对应远程仓库的更新。

`git fetch --all` 则会获取所有远程仓库的更新。

#### git pull
获取远程仓库更新到本地仓库，并且合并当前所在分支所对应的上游(upstream)分支代码。等价于下面的两步操作：

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

#### git checkout
该命令有三种作用：

1.  切换本地分支
2.  如果指定的跟地分支不存在，但是本地仓库存在同名的远程分支，则会创建同名的本地分支作为跟踪分支

    ```bash
    git checkout branch_name
    # 等价于
    git checkout -b branch_name --track remote/branch_name
    ```

3.  取消当前工作目录(working directory)的修改

    ```bash
    git checkout filename
    # 等价于
    git checkout -- filename
    ```

需要注意的是最后一种情况仅当没有和 filename 同名的分支名时才生效。

#### git push
推送本地分支代码到远程仓库

在本地分支直接运行 `git push` 经常会遇到的一种错误是：

> fatal: The current branch branch_name has no upstream branch.

这是因为当前分支没有设置上游(upstream)分支，git 不能确定当前分支应该推送到远程仓库的哪个分支导致的。解决方法：

```bash
git branch --set-upstream remote/branch_name
```

上游(upstream)分支设置完成后，运行 `git push` 则等价于：

```bash
git push remote branch_name:branch_name
```

也可以使用下面语句：

```bash
git push remote branch_name:other_branch_name
```

来将当前本地分支代码推送到其他远程分支上去。

#### 参考

1. [git-fetch(1) Manual Page](http://git-scm.com/docs/git-fetch)
2. [git-pull(1) Manual Page](http://git-scm.com/docs/git-pull)
2. [git-checkout(1) Manual Page](http://git-scm.com/docs/git-checkout)
2. [git-push(1) Manual Page](http://git-scm.com/docs/git-push)
