---
layout: post
title: Git Q&A
---

**fetch 和 pull 的区别**

**新建分支**

```bash
git branch new-branchname
git checkout new-branchname
# 等价与
git checkout -b new-branchname
```

**暂存修改过的文件**  

```bash
git add filename
# 或者
git add .

git rm filename  # 删除工作目录和暂存区中的文件

git rm --cache filename  # 删除暂存区中的文件，保留工作目录中的代码

git mv filename new-filename
```

**提交修改**  
提交暂存区文件

```bash
git commit -m 'commit comment'
```

提交所有修改过的文件

```bash
git commit -am 'commit comment'
```

```bash
git commit --amend
```

**合并分支**

**删除分支**  
 删除合并过的分支

```bash
git branch -d branchname
```

删除未合并的分支

```bash
git branch -d -f branchname
# 等价于
git branch -D branchname
```

**--no-ff 的作用**

**何时使用 rebase**

**fetch 别人的分支 git branch 看不到？**

```bash
git fetch
git branch -a
git checkout new-branchname
```

**5. 如何删除分支？**  
删除本地分支

```bash
git branch -d branchname
```
删除远程分支

```bash
git push origin :branchname
```

**HEAD 是那个区域**  

**git checkout -- filename, 其中 -- 的作用**

**设置 git clone 时创建的默认分支**  
Git clone additionally creates a remote called ‘origin’ for the repo cloned from, sets up a local branch based on the remote’s active branch (generally master)

```bash
git symbolic-ref HEAD refs/heads/mybranch
```

**git reset**  

```
git reset
git reset --soft
git reset --hard
```

**参考**

* [A Visual Git Reference](http://marklodato.github.io/visual-git-guide/index-en.html?no-svg)
* [What's the difference between 'git pull' and 'git fetch'](http://stackoverflow.com/questions/292357/whats-the-difference-between-git-pull-and-git-fetch)
* [Visual Git Cheat Sheet ](http://ndpsoftware.com/git-cheatsheet.html)
* [Heroku Cheat Sheet](https://na1.salesforce.com/help/doc/en/salesforce_git_developer_cheatsheet.pdf)
* [Setting the default git branch in a bare repository](http://feeding.cloud.geek.nz/posts/setting-default-git-branch-in-bare/)
* [THE DIFFERENCE BETWEEN GIT PULL, GIT FETCH AND GIT CLONE (AND GIT REBASE)](http://blog.mikepearce.net/2010/05/18/the-difference-between-git-pull-git-fetch-and-git-clone-and-git-rebase/)
