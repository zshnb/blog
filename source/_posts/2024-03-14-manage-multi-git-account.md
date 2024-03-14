---
title: 多Git用户管理 - 根据目录自动切换Git用户
date: 2024-03-14 17:43:03
tags:
  - Git
---

由于我的个人项目和公司项目在同一台电脑上开发，有个问题一直在困扰我，当我在不同项目上开发时需要检查当前项目对应的Git用户是否正确。
因为公司用的Git用户邮箱和个人Git邮箱不一样，如果不小心用了个人Git用户提交了公司代码会很麻烦，所以一直以来我都是默认设置成公司Git用户。
这样又有个问题，总是会不小心用公司Git用户提交了个人项目，导致经常需要重置提交后再覆盖提交信息。正好最近找到一个非常方便的解决方案，分享一下。
<!--more-->

1. 编辑~/.gitconfig，填入以下内容

```text
[includeIf "gitdir:**/Workbench/dior/**"]
path = ~/.diorconfig
[includeIf "gitdir:**/Workbench/**"]
path = ~/.githubconfig
```

2. 分别编辑.diorconfig和.githubconfig，填入不同的name和email

```text
[user]
  name = zshnb
  email = xxx@gmail.com
```

这样当你进入不同项目目录时，对应的git配置文件就会生效，效果如下

| 个人项目                | 公司项目                |
|---------------------|---------------------|
| ![个人项目目录](img1.png) | ![公司项目目录](img2.png) |


