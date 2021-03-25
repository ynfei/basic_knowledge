# git 修改某次提交

[TOC]

## 1. 修改最近一次提交

有两种方式

### 1.1 git commit -amend

这种方法不仅可以修改commit message，也可以修改提交的内容。这种方式在还没有推送到远端的情况下可以比较方便的保持原有的change id，若已经推送到远端，change id则会修改掉

```
git add XXX #这一步如果只修改commit messge就不需要
git commit --amend
git push origin master -f 
```

### 1.2 reset后修改

这种方法与上面方法一致，也可以修改提交内容和commit message。这种方式在还没有推动到远端的情况下也可以比较方便的保持原有的change-id，如果推动到远端，change-id也会修改。

```
git reset HEAD^  #这里用的是默认--mixed，工作区保存了上一次的更改
git add .  #这一步如果只修改commit messge就不需要
git commit
git push origin master -f
```

##  2. 修改历史上某次提交

```
#找到需要修改的提交的上一次提交
git rebase -i "commit"^
#把pick修改为edit,保存退出
#进行修改
git add XXX
git commit --amend
git rebase --continue
git push origin master -f
```

## 3. 提交到错误的分支

- 方式一：

  ```
  git reset --soft HEAD^ #仅撤销git commit, git add内容还保留
  git stash #保存当前更改
  git chcekout name-of-the-correct-branch
  git stash pop
  git add .
  git commit 
  ```

- 方式二：

  ```
  git checkout name-of-the-correct-branch
  git cherry-pick master #把主分支最新提交摘过来
  git chcekout master 
  git reset --hard HEAD^ 删除master分支上的最新提交
  ```

  