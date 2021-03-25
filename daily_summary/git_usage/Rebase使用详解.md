# Rebase使用详解

git rebase的主要使用场景有两个：①多人合作在同一个分支上共同开发；②合并不同的分支

#### 场景一：本地与远端同一分支提交历史不一致

git pull之后会出现分支现象，当本地分支与远端分支不一样时，git进行了自动合并，生成了一个新的提交历史。

使用git rebase就可以解决这个问题

git pull之后，使用git rebase，则不会生成新的提交历史，不会有分支；或者使用git pull --rebase

#### 场景二：不同分支之间的合并

当合并两个分支时，例如将dev分支合并到master分支，如果有冲突合并之后也会出现分支。

解决办法：在dev分支上执行git rebase master如果有冲突产生，解决冲突；重点是合并完冲突之后，要继续rebase, git rebase --continue完成rebase操作。然后切回master分支进行merge,不会有分支产生。



#### git merge与git rebase的区别

使用git merge，git的操作如下：

1. 找出dev分支和master分支的最近共同祖先commit点
2. 将dev最新一次commit（C5）和master最新一次commit（C6）合并后生成一个新的commit（C7），有冲突的话需要解决冲突
3. 将以上两个分支dev和master上的所有提交点（从C2以后的）按照**提交时间的先后顺序**进行依次放到master分支上

![img](img/git_merge_rebase)

使用git rebase，git的操作如下：

1. rebase之前需要经master分支拉到最新
2. 切换分支到需要rebase的分支，这里是dev分支
3. 执行git rebase master，有冲突就解决冲突，解决后直接git add . 再git rebase --continue即可
4. 切换到master分支，执行git merge dev

可以发现其一并没有多出一次commit，其二dev后面几次提交的commit hash值已经变了，包括C3，C4，C5

发现采用rebase的方式进行分支合并，整个master分支并没有多出一个新的commit，原来dev分支上的那几次（C3，C4，C5）commit在rebase之后其hash值发生了变化，不在是当初在dev分支上提交的时候的hash值了，但是提交的内容被全部复制保留了，并且整个master分支的commit记录呈线性记录

![img](img/git_merge_rebase1)

#### 总结

- git merge 操作合并分支会让两个分支的每一次提交都按照提交时间（并不是push时间）排序，并且会将两个分支的最新一次commit点进行合并成一个新的commit，最终的分支树呈现非整条线性直线的形式

- git rebase操作实际上是将当前执行rebase分支的所有基于原分支提交点之后的commit打散成一个一个的patch，并重新生成一个新的commit hash值，再次基于原分支目前最新的commit点上进行提交，并不根据两个分支上实际的每次提交的时间点排序，rebase完成后，切到基分支进行合并另一个分支时也不会生成一个新的commit点，可以保持整个分支树的完美线性

  

  

  

  