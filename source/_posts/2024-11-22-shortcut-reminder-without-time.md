---
title: iOS快捷指令创建不带时间的提醒事项
date: 2024-11-22 15:10:11
tags:
  - 快捷指令
---

最近在开发一个快捷指令，核心功能是从Notion获取数据，创建提醒事项，在开发过程中发现一个问题，当我使用下面这个Action创建提醒事项时，`dueDate`变量在没有时间部分下，创建出来的提醒事项会自动带上12:00，即使设置了format也没用。
<!--more-->
![](1.webp)
于是尝试搜索了一下，在Reddit里找到一篇[帖子](https://www.reddit.com/r/shortcuts/comments/i9n8e0/create_a_reminder_with_a_date_but_no_time/)，解决办法是在dueDate变量后面手动填上`at no time` ，问题就解决了。

![](2.webp)

