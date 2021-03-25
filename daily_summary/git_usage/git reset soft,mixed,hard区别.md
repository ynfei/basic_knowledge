# git reset soft,mixed,hard区别

先解释几个术语

- HEAD:当前分支版本顶端的别名，也就是当前分支最近的一个提交
- Index:也被称为staging area，指一整套即将被下一个提交的文件集合。
- Working tree:代表你正在工作的那个文件集

soft:

![img](img/git_reset_soft.png)

mixed:

![img](img/git_reset_mixed.png)

hard:

![img](img/git_reset_hard.png)

简单总结一下，其实就是--soft  、--mixed以及--hard是三个恢复等级。使用--soft就仅仅将头指针恢复，已经add的缓存以及工作空间的所有东西都不变。如果使用--mixed，就将头恢复掉，已经add的缓存也会丢失掉，工作空间的代码什么的是不变的。如果使用--hard，那么一切就全都恢复了，头变，aad的缓存消失，代码什么的也恢复到以前状态。