---
title: Manjaro谷歌浏览器设置代理
date: 2023-10-23 12:00:27
tags:
  - Linux
  - Chrome
---

最近安装了最新版本的google chrome浏览器后，运行后发现设置里的proxy选项没有了，于是就导致无法登录google账号，且无法科学上网。经过搜索后发现，在Arch Linux | Manjaro下，想要设置google chrome的代理，需要通过`/opt/google/chrome/chrome --proxy-server="[http://127.0.0.1:7890](http://127.0.0.1:7890/)"` 启动后，才能正常科学上网。MacOS下的chrome目前没有这个问题。

![设置](img1.png)