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

**参考**

1. [What's the difference between 'git pull' and 'git fetch'](http://stackoverflow.com/questions/292357/whats-the-difference-between-git-pull-and-git-fetch)
2. [Visual Git Cheat Sheet ](http://ndpsoftware.com/git-cheatsheet.html)
3. [Heroku Cheat Sheet](https://na1.salesforce.com/help/doc/en/salesforce_git_developer_cheatsheet.pdf)
