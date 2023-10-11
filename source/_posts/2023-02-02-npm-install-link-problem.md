---
title: npm9.x安装本地文件依赖踩坑
date: 2023-02-02 15:21
tags:
- NPM
- Javascript
---

# 背景
公司项目下有几个submodule的依赖，通过file协议安装，最近在开发新功能时发现本地更改submodule的代码，主项目无法引用到最新的代码，
同时webstorm的代码跳转会进入node_module目录下同名文件夹，而同事表示他们本地没有这种情况。
<!--more-->
# 探寻
经过对比发现我跟同事电脑开发环境只有npm版本不同，我安装的是最新9.2，他们的是8.11，于是我们开始了一番实验。
- 同事升级npm到9，出现跟我一样的行为
- 我降低npm到8，问题消失

于是确认是npm9导致的问题，我们又对比了8和9执行`npm install`命令所生成的package-lock.json不同处
![](package-lock-diff.png)
可以看到左边的`link: true`消失了，取而代之的是`file:xxx`，这个不同意味着8安装的本地依赖为link类型的文件夹，node_module
中的依赖指向了外部源码文件夹，而9则变成了node_module中的依赖为独立文件夹，外部的更改无法同步，只能通过re-install同步。
然后我在npm的官方仓库里找到了这个行为的变更记录
![](npm-changelog.jpeg)
可以看到第4条，更详细的还可以去看看对应的[PR](https://github.com/npm/cli/pull/5458)

# 解决
问题都查清楚了，那要怎么解决这个问题呢
1. 降级8
2. npm install --install-links=false
3. .npmrc文件中添加install-links=false