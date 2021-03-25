# git 撤销某次提交

[TOC]

## 1. 删除最近一次提交

这种情况最简单，有以下两种方式供选择：

### 1.1 git reset

```
git reset --hard HEAD^
git push origin master -f
```

这种方式直接丢弃了最新的一次更改，没有保留记录。

### 1.2 git revert

```
git revert HEAD
git push origin master
```

这种方式放弃指定提交的修改，但是会生成一次新的提交，以前的提交历史都在。

## 2. 删除历史某次提交

### 2.1 git rebase

```
git log
git rebase -i "cmmmit id"^
git push origin master -f
```

第一步操作找出需要删除提交的`commit id`，第二步`rebase`需要提交的`commit id`的上一个提交，会弹出如下所示的框：

![image-20200521150314827](img/git_rebase_i.png)

修改为：

![image-20200521150426436](img/git_rebase_drop.png)

退出后，则丢弃掉了该commit，推送到远程即可。

