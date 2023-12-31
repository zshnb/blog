---
title: Git系列（1）完美的提交
date: 2022-09-04 21:10
tags:
- Git
---

# 如何创建一个完美的提交

1. 添加正确，合适的更改内容到暂存区
2. 编写易读的提交信息

下面来具体说明。
<!--more-->
很多时候我们经常有的一个问题是，更改了许多文件，并且文件之间彼此更改的目的不同，最后提交的时候，一股脑`git add . && git commit -m "update"`，这种方式虽然方便快捷，可带来的后果是无法回溯，在未来的时间看到这段提交记录无法得知当时改动的目的。更好的做法是每一次提交所包含的更改是属于同一个主题的，且通过简洁清晰的提交信息记录。下面来看操作步骤。

1. 有如下3个文件的更改，分别属于3个topic，首先需要提交topic1的信息，注意，在change2文件中同时存在topic1和其他topic的更改

![commit](img1.png)

可以看到在change2文件中存在2个修改块，分别属于2个topic，但第一个提交信息我们只需要上面的更改块，于是我们可以这样操作 ![stage add](img4.png)

通过执行`git add -p file`选择此次add的更改块，可以看到change2文件既存在工作区，又存在暂存区。

![commit](img2.png)

然后我们继续添加change1进入暂存区，此时topic1的所有改动都已经添加完毕，是时候准备提交了。事实上我们完全可以简单一句`git commit -m "change1 topic"`就完事，但这不符合我们今天的主题，一个完美的提交应该由**主题（简介清晰的概述此次提交）和说明（此次提交修改了什么内容;为什么要修改;此次提交需要注意的点）**组成。下面就是第一个提交的信息，注意第一行下面由一行空行，作用是用来分割主题和说明用的。

![commit](img3.png)

最后通过`git log`可以看到此次的提交信息，非常清晰地说明了当时提交的原因，更改内容，为之后的回溯提供非常大的便利。

![log](img5.png)
