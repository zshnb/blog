---
layout: post
title: Git系列（3）合并冲突
date: 2022-09-18 21:10
author: zsh
catalog: true
tags:
- Git
---

# 合并冲突如何发生

看到**合并冲突**这4个字，很多人觉得肯定只有合并的时候才会发生。事实上只要你尝试把来自几个不同源的修改同时应用到同一个文件上，都可能发生冲突，这里的应用包括以下几种操作

- merge
- rebase
- cherry-pick
- git stash pop
<!--more-->
那么为什么会有冲突呢，简单来说就是2个对同一个文件的同一处修改不一致了，Git不知道该用哪一段修改，于是就标记出冲突行，让用户自行解决。

![merge branch](img1.png)

首先在feature2分支删除change1文件，然后在master分支修改change1文件，最后合并feature2分支，此时git便会提示存在merge conflict，通过git status也能看到发生冲突的文件。git顺便提示了如何解决冲突，对于这个场景来说，解决冲突就意味着是留下还是删除文件，对应的git add/git rm命令。

![merge branch](img2.png)

第2种情况是2个分支的提交同时修改同一个文件的同一处，此时git status提示冲突所在文件，打开文件可以看到git标注的冲突块

![merge branch](img3.png)

以===分割，<<<表示的是当前提交处的修改，>>>表示的是导致冲突的其他操作的修改。这里解决冲突只需要删除git添加的冲突提示，然后正常git add . git commit即可。

如果出于某种理由不想解决冲突了怎么办？每种操作都可以撤销，比如merge对应的是git merge --abort，表示取消合并，其余几个命令同样。
